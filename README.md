# FocusBoard

A focused task management dashboard built with React and Vite — designed to help users plan, track, and complete daily work efficiently.

---

## Project Overview & Objective

**FocusBoard** is a single-page productivity application that lets users create, read, update, and delete tasks from a clean, intuitive dashboard. The objective is to demonstrate practical React skills including component architecture, local state management, derived data, filtering, persistent storage via `localStorage`, and responsive UI design — all without any external state library.

---

## Technology Stack

| Layer | Technology |
|---|---|
| UI Library | React 18 (with Hooks) |
| Build Tool | Vite |
| Styling | Plain CSS (component-scoped CSS files) |
| Icons | lucide-react |
| Persistence | Browser `localStorage` (custom hook) |
| Language | JavaScript (JSX) |
| Package Manager | npm |

---

## Key Features Implemented

- **Task CRUD** — Create, edit, delete tasks via a modal form with validation.
- **Status Management** — Inline status dropdown on each task card (`To do` / `In progress` / `Completed`).
- **Search** — Live search filtering tasks by title.
- **Filter Tabs** — Filter tasks by status (All / To do / In progress / Completed).
- **Priority Filter** — Dropdown filter for High / Medium / Low priority.
- **Reset Filters** — Single button clears all active filters and search.
- **Overview Statistics** — Live-calculated counts for Total, In Progress, and Completed tasks.
- **FocusNotes Carousel** — Motivational card carousel with previous/next navigation and dot indicators.
- **Persistent Storage** — All tasks are saved to `localStorage` and survive page refresh.
- **Responsive Design** — Layout adapts for desktop, tablet (768 px), and mobile (480 px).
- **Accessible UI** — ARIA labels, `aria-pressed`, `aria-label`, `role="group"` throughout.

---

## Project Folder Structure

```
focusboard/
├── public/
│   ├── favicon.svg
│   └── icons.svg
├── src/
│   ├── App.jsx                  # Root component — state hub, filter logic
│   ├── App.css                  # (reserved / empty)
│   ├── index.css                # Global reset, CSS variables, layout
│   ├── main.jsx                 # React DOM entry point
│   ├── assets/
│   │   └── hero.png
│   ├── components/
│   │   ├── FilterTabs/
│   │   │   ├── FilterTabs.jsx   # Status tabs + priority dropdown + reset
│   │   │   └── FilterTabs.css
│   │   ├── FocusNotes/
│   │   │   ├── FocusNotes.jsx   # Motivational carousel with slide state
│   │   │   └── FocusNotes.css
│   │   ├── Header/
│   │   │   ├── Header.jsx       # Top navigation bar with breadcrumb + profile
│   │   │   └── Header.css
│   │   ├── Overview/
│   │   │   ├── Overview.jsx     # Statistics section — derives counts from tasks
│   │   │   └── Overview.css
│   │   ├── SearchBar/
│   │   │   ├── SearchBar.jsx    # Controlled search input with clear button
│   │   │   └── SearchBar.css
│   │   ├── Sidebar/
│   │   │   ├── Sidebar.jsx      # Fixed left navigation
│   │   │   └── Sidebar.css
│   │   ├── StatCard/
│   │   │   ├── StatCard.jsx     # Reusable statistic display card
│   │   │   └── StatCard.css
│   │   ├── TaskCard/
│   │   │   ├── TaskCard.jsx     # Individual task row with inline status select
│   │   │   └── TaskCard.css
│   │   ├── TaskForm/
│   │   │   ├── TaskForm.jsx     # Create / Edit modal form with validation
│   │   │   └── TaskForm.css
│   │   └── TaskList/
│   │       ├── TaskList.jsx     # Task section header + renders TaskCard list
│   │       └── TaskList.css
│   ├── data/
│   │   └── sampleTasks.js       # Initial seed data loaded on first visit
│   └── hooks/
│       └── useLocalStorage.js   # Custom hook syncing state ↔ localStorage
├── index.html
├── package.json
├── vite.config.js
└── README.md
```

---

## Install Dependencies

```bash
# Navigate to the project directory
cd focusboard

# Install all dependencies
npm install
```

---

## Run the Project Locally

```bash
# Start the development server
npm run dev
```

The app will be available at **http://localhost:5173** (Vite may choose another port if 5173 is busy — check the terminal output).

```bash
# Build for production
npm run build

# Preview the production build locally
npm run preview
```

---

## State Management

### Approach

FocusBoard uses **React's built-in `useState` hook** (via a custom `useLocalStorage` wrapper) as its sole state management solution. No external library (Redux, Zustand, MobX, etc.) is used. This was a deliberate choice — the application's complexity is well within what React's local state can handle cleanly.

### How Task Data Is Maintained

All task data lives in a single state variable `tasks` in `App.jsx`:

```js
const [tasks, setTasks] = useLocalStorage("focusboard_tasks", sampleTasks);
```

`App.jsx` acts as the **single source of truth**. Child components receive tasks (or derived subsets) as props and communicate changes back via callback props (`onAddTask`, `onEditTask`, `onDeleteTask`, `onStatusChange`).

### How CRUD Operations Update State

| Operation | Handler | State update pattern |
|---|---|---|
| **Create** | `handleAddTask` / `handleSaveTask` | `setTasks(prev => [...prev, newTask])` |
| **Update** | `handleSaveTask` (with `editingTask`) | `setTasks(prev => prev.map(t => t.id === id ? updated : t))` |
| **Delete** | `handleDeleteTask` | `setTasks(prev => prev.filter(t => t.id !== id))` |
| **Status change** | `handleStatusChange` | `setTasks(prev => prev.map(t => t.id === id ? {...t, status} : t))` |

Each operation creates a **new array** (immutable update pattern), ensuring React detects the change and re-renders.

### How Derived Statistics and Filters Are Calculated

Statistics and filtered lists are **computed directly from the `tasks` array on every render** — no separate state is stored for them:

```js
// Derived statistics in Overview.jsx
const totalTasks     = tasks.length;
const inProgressTasks = tasks.filter(t => t.status === "In progress").length;
const completedTasks  = tasks.filter(t => t.status === "Completed").length;

// Filtered task list in App.jsx
const filteredTasks = tasks.filter(task => {
  const matchesSearch   = task.title.toLowerCase().includes(searchTerm.toLowerCase());
  const matchesStatus   = statusFilter === "All" || task.status === statusFilter;
  const matchesPriority = priorityFilter === "All" || task.priority === priorityFilter;
  return matchesSearch && matchesStatus && matchesPriority;
});
```

Filter state (`searchTerm`, `statusFilter`, `priorityFilter`) is managed with separate `useState` calls in `App.jsx` and passed down as props.

### How localStorage Is Synchronised with Task State

A **custom `useLocalStorage` hook** (`src/hooks/useLocalStorage.js`) transparently wraps `useState`:

- On **initialisation**: reads the stored JSON value from `localStorage`; falls back to the provided default (sample tasks) if nothing is stored yet.
- On every **state update**: a `useEffect` inside the hook writes the new value back to `localStorage` as a serialised JSON string.

This means every task creation, edit, delete, or status change is automatically persisted without any extra calls in the components themselves. Refreshing the page restores the exact task list the user left.

### How Carousel State Is Maintained

The `FocusNotes` component manages its own **local state**:

```js
const [activeSlide, setActiveSlide] = useState(0);
```

This state is entirely self-contained — it does not need to be shared with any other component. Navigation buttons call `previousSlide` / `nextSlide` which update `activeSlide` with wrap-around arithmetic (`% slides.length`), and dot indicators allow direct jump to any slide. This is a good example of **co-locating state with the component that owns it**.

---

## localStorage Implementation

### Custom Hook: `useLocalStorage`

```js
// src/hooks/useLocalStorage.js
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}
```

**Key design decisions:**
- The initialiser is a **lazy function** passed to `useState` so `localStorage` is only read once on mount, not on every render.
- Both `get` and `set` are wrapped in `try/catch` to gracefully handle private browsing mode or storage quota errors.
- The hook returns a `[value, setter]` tuple — a drop-in replacement for `useState` — so usage in `App.jsx` is identical to standard React state.

---

## Hosted Application URL

> 🔗 https://focusboard-xi-ebon.vercel.app

---

## GitHub Repository URL

> 🔗 https://github.com/Sricharan990/focusboard.git

---

## Known Limitations / Pending Work

| Item | Detail |
|---|---|
| No user authentication | Tasks are stored per-browser; no multi-user or cloud sync support. |
| No drag-and-drop reordering | Tasks are displayed in insertion order; reordering requires manual edit. |
| Single board only | No concept of multiple projects or boards. |
| No due-date reminders | Due dates are stored and displayed but no notification system exists. |
| No dark mode | Only a light theme is implemented. |
| No unit / integration tests | Test coverage is not included in this submission. |
| Sidebar is static on mobile | On narrow screens the sidebar collapses but has no hamburger toggle. |
