# Technical Documentation: Scalable Project Architecture

## Concept Explanation

A **Scalable Project Architecture** is a structural framework designed to manage complexity as a codebase grows from 10 to 1,000+ files. The primary goal is to maintain **Low Coupling** (modules don't depend on each other's internals) and **High Cohesion** (related logic is grouped together).

### The Problem it Solves
- **Decision Fatigue:** Developers don't have to wonder where a file belongs.
- **Circular Dependencies:** A clear hierarchy prevents Component A from importing B while B imports A.
- **Merge Conflicts:** By isolating features, developers are less likely to touch the same files simultaneously.
- **Cognitive Load:** New engineers can navigate the "City Map" of the codebase intuitively.

---

## Mental Models & Analogies

### The "City Planning" Metaphor
- **Infrastructure (`/api`, `/config`, `/styles`):** The power lines and water pipes. They serve the whole city but don't define the buildings.
- **Neighborhoods (`/features` or `/views`):** Self-contained areas where people live and work. A neighborhood has its own grocery store (local hooks) and park (local components).
- **Public Works (`/components`, `/helpers`):** Shared resources like libraries or police stations that any neighborhood can use.

---

## Directory Breakdown (The Core Domains)

A professional React architecture is typically divided into **Global Shared** and **Feature-Specific** directories.

| Directory | Purpose | Best Practice |
| :--- | :--- | :--- |
| **`api/`** | API clients (Axios/Fetch instances), interceptors, and global endpoints. | Keep this layer "thin." Don't put business logic here. |
| **`assets/`** | Static files: SVG icons, images, fonts, and global JSON. | Use a specialized tool for SVG-to-Component conversion. |
| **`config/`** | Environment variables, third-party keys, and global feature flags. | Validate `process.env` here using a schema (e.g., Zod). |
| **`constants/`** | Immutable values: URL paths, regex, error messages, and magic numbers. | Use `const` or `enum` (if strictly necessary). |
| **`context/`** | Global UI state (Theme, Auth, Toast). | Keep these small; don't make a "God Context." |
| **`helpers/`** | **Pure** utility functions (date formatting, currency, string manipulation). | Ensure these are "side-effect free" for easy unit testing. |
| **`hooks/`** | Global, reusable logic (useLocalStorage, useWindowSize). | If a hook is only used in one feature, move it to that feature's folder. |
| **`intl/`** | Localization files (i18n), translation keys, and locale config. | Use nested JSON structures to group by view or domain. |
| **`layout/`** | Structural components (AppShell, Sidebar, Footer). | Layouts should only define "where" things go, not "what" things are. |
| **`services/`** | Complex business logic or third-party SDK wrappers (Firebase, Auth0). | Use this layer to decouple your app from specific vendors. |
| **`store/`** | Global state management (Zustand, Redux, Jotai). | Organize by "slice" or "domain." |
| **`styles/`** | Design tokens, CSS variables, global reset, and themes. | Centralize your "Source of Truth" for spacing and colors here. |
| **`types/`** | Shared TypeScript interfaces, types, and enums. | Prefer `types/` for entities used across 3+ domains. |
| **`views/`** | High-level page components (Login, Dashboard, Profile). | Views "compose" components and features; they rarely contain raw CSS. |

---

## Strategic Patterns: The "Vertical Slice"

The modern "Staff-level" recommendation is to move away from purely horizontal architecture (where all hooks are in one giant folder) toward **Feature-Driven Design**.

### The Feature Folder Structure
Inside `src/features/search/`:
- `components/`: SearchBar, SearchResults
- `hooks/`: useSearchHistory
- `api/`: searchEndpoints
- `types/`: SearchResultInterface
- `index.ts`: The "Public API" (re-exports only what others are allowed to see).

---

## Code Examples: The "Public API" Pattern

To prevent "Import Spaghetti," each folder (especially features) should have an `index.ts` (Barrel File).

```tsx
// ❌ Bad: Importing from deep internals
import { SearchBar } from '@/features/search/components/SearchBar';

// ✅ Good: Features expose a Public API
// features/search/index.ts
export * from './components/SearchBar';
export * from './hooks/useSearch';
export type { SearchResult } from './types';

// Usage
import { SearchBar, useSearch } from '@/features/search';
```

---

## Performance Considerations

### 1. Tree Shaking
Barrel files (`index.ts`) can accidentally break tree shaking if not managed correctly. Ensure your `package.json` has `"sideEffects": false` or specific exclusions to help bundlers (Webpack/Vite) remove unused code.

### 2. Code Splitting (Lazy Loading)
Architecture directly impacts bundle size. By isolating `views/`, you can easily use `React.lazy()` to load pages only when needed.

```tsx
const Dashboard = lazy(() => import('@/views/Dashboard'));
```

---

## Real-World Use Cases

1.  **Large Scale Dashboards:** Using the `services/` layer to wrap a legacy API, allowing the rest of the app to interact with clean, modern data shapes.
2.  **White-label Apps:** Using the `styles/` and `config/` folders to inject different themes/branding for different clients while keeping the `features/` logic identical.

---

## Common Mistakes

- **Circular Dependencies:** Component A in Feature 1 imports from Feature 2, which imports back from Feature 1. 
  - *Fix:* If two features need each other, extract the shared logic to `helpers/`, `hooks/`, or a third feature.
- **Deep Nesting:** Folders inside folders inside folders. 
  - *Fix:* Aim for a "flat as possible" hierarchy. If a folder is 4+ levels deep, it’s a sign that your features aren't granular enough.
- **Business Logic in Views:** Putting complex data transformations inside a view. 
  - *Fix:* Move it to a `service/` or a custom `hook/`.

---

## Summary & Key Takeaways

- **Horizontal Folders (`/hooks`, `/helpers`):** For global, generic utilities.
- **Vertical Slices (`/features`):** For domain-specific business logic.
- **Separation of Concerns:** `api/` fetches, `services/` transforms, `components/` renders.
- **Barrel Files:** Use `index.ts` to define what is public and what is private.
- **Interview Tip:** "I prioritize **Feature-Driven Architecture** over Type-based folders. This allows us to scale by adding 'Vertical Slices' rather than bloating 'Horizontal Folders.' It improves maintainability, makes code-splitting easier, and prevents circular dependencies by enforcing a strict 'Public API' for every feature."

## Route Components

**Route Components** (often called **Page Components** or **Views**) are the top-level components rendered by a router when the browser's URL matches a specific path. They act as the entry point for a specific feature or screen in your application.

In modern React (especially with React Router v6.4+), Route Components have evolved from being simple UI containers into **Orchestrators**. They tie together:
1.  **URL State:** (Params like `:id`, Query strings like `?search=`).
2.  **Data Requirements:** (Loaders/API calls).
3.  **Layouts:** (Shared UI like sidebars or navbars).

### The Problem it Solves
- **URL-Driven UI:** Ensures the UI is always in sync with the address bar.
- **Entry Point Isolation:** Provides a clear boundary where a feature starts, making code-splitting and analytics tracking (page views) trivial.
- **Contextual Rendering:** Allows for nested UI structures where only the "inner" part of a page changes while the "outer" layout remains static.

---

## Mental Models & Analogies

### The "Stage & Set" Metaphor
- **The Router** is the **Stage Manager**. It watches the script (The URL) to see which scene comes next.
- **The Layout Route** is the **Set**. It’s the background, the walls, and the floor that stay the same throughout multiple scenes.
- **The Route Component** is the **Actor/Scene**. It’s the specific content that enters the stage to perform. When the scene changes, the set stays, but the actors swap out.

---

## Design Patterns & Best Practices

### 1. The "View" vs. "Feature" Distinction
Route components should live in a `views/` or `pages/` directory. They should be "Skinny."
- **Wrong:** Putting 500 lines of JSX and 3 `useEffect` hooks in `UserPage.tsx`.
- **Right:** The `UserPage` component imports 2-3 "Feature Components" (e.g., `UserProfile`, `UserOrderHistory`) and composes them.

### 2. Layout Routes & `<Outlet />`
Use Layout Routes to prevent re-rendering shared UI elements like Navbars. The `<Outlet />` is a placeholder where the child route component will be injected.

### 3. Index Routes
An Index Route is the "default" child. It renders when the parent's path matches exactly, but no sub-path is provided.

---

## Code Examples

### Standard Nested Route Architecture

```tsx
// App.tsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />, // Shared Nav/Footer
    children: [
      {
        index: true, // Matches "/"
        element: <HomePage />,
      },
      {
        path: "dashboard",
        element: <DashboardLayout />, // Sub-layout for dashboard
        children: [
          {
            index: true, // Matches "/dashboard"
            element: <DashboardStats />,
          },
          {
            path: "settings", // Matches "/dashboard/settings"
            element: <UserSettings />,
          },
        ],
      },
    ],
  },
]);
```

### The "Skinny" Route Component
This component focuses on gathering URL data and passing it to feature components.

```tsx
// views/UserDetailView.tsx
import { useParams } from 'react-router-dom';
import { UserProfileFeature } from '@/features/users';

export const UserDetailView = () => {
  const { userId } = useParams<{ userId: string }>();

  if (!userId) return <ErrorState />;

  return (
    <main className="page-container">
      <h1>User Management</h1>
      {/* Route component coordinates the feature */}
      <UserProfileFeature id={userId} />
    </main>
  );
};
```

---

## Performance Considerations

### 1. Code Splitting (Lazy Loading)
Since users only visit one route at a time, there is no need to load the code for the "Settings" page while they are on the "Login" page.

```tsx
const AdminPage = React.lazy(() => import('./views/AdminPage'));

// In router config
{
  path: "admin",
  element: (
    <Suspense fallback={<Loader />}>
      <AdminPage />
    </Suspense>
  ),
}
```

### 2. Preventing Layout Re-mounts
By using the `children` pattern in React Router, the parent Layout component **does not unmount** when navigating between sibling child routes. This preserves the internal state of the Sidebar (like scroll position or open menus).

---

## Real-World Use Cases

1.  **Dashboard Shells:** A persistent layout with a sidebar that navigates through various "widgets" or data views.
2.  **Auth Guards:** Wrapping a branch of routes in a `ProtectedRoute` component that checks for a token before rendering the `<Outlet />`.
3.  **Breadcrumbs:** Using the route hierarchy to automatically generate a breadcrumb trail (`Home > Dashboard > Settings`).

---

## Common Mistakes

- **Prop Drilling from Layouts:** Trying to pass data from a `RootLayout` to a deeply nested child via props.
    - *Fix:* Use `useOutletContext()` or a dedicated State Management store (Zustand/Context).
- **Too Much Logic in Routes:** Handling API data transformation inside the route file.
    - *Fix:* Move data logic to `loaders` or custom `hooks`.
- **Absolute Paths Everywhere:** Hardcoding `<Link to="/dashboard/settings/profile">`.
    - *Fix:* Use relative paths `<Link to="profile">` when navigating within a sub-hierarchy to make your routing more modular.

---

## Summary & Key Takeaways

- **Orchestration:** Route components should coordinate between the URL and the features.
- **Composition:** Use `<Outlet />` for nested layouts to maximize UI reuse and performance.
- **Lazy Loading:** Always lazy load route-level components to minimize the initial bundle size.
- **Stability:** Layout routes prevent the "flicker" of shared UI elements during navigation.
- **Interview Tip:** "I treat Route Components as the **glue** between the application's URL and its business features. I keep them 'skinny' by delegating UI to feature components and data logic to loaders, ensuring that my page transitions are handled efficiently through code-splitting and nested routing."

## Encapsulation in React

**Encapsulation** in React is the practice of bundling data (state), behavior (logic), and presentation (UI) into self-contained, independent units. A well-encapsulated component acts as a **Black Box**: the consumer interacts with a clear, minimal interface (Props), while the internal implementation details remain hidden and protected.

### What the concept is
- **Logic Encapsulation:** Moving complex business logic, data transformations, or API calls into custom hooks.
- **UI Encapsulation:** Ensuring styles and DOM structure are contained so they don't "leak" or conflict with other parts of the app.
- **Relationship Encapsulation:** Using patterns like Compound Components to manage how related components communicate without exposing that complexity to the parent.

### The Problem it Solves
- **Leaky Abstractions:** When a change in a component's internal logic forces you to update five other files.
- **Cognitive Overload:** Developers shouldn't need to understand *how* a `DatePicker` works to use it; they only need to know how to provide a `date` and an `onChange`.
- **Untestable Code:** Tightly coupled logic is nearly impossible to unit test.

---

## Mental Models & Analogies

### The "Automobile" Metaphor
When you drive a car, you interact with the **Steering Wheel, Pedals, and Gear Shift** (The Props/API). You do not manually manage the air-to-fuel ratio in the engine or the hydraulic pressure in the brakes (Internal Logic). 
- If the manufacturer switches the car from a gasoline engine to an electric motor (Internal Implementation), your "interface" (the pedals) stays the same. You don't have to "re-learn" how to drive.

---

## Design Patterns & Best Practices

### 1. Logic Encapsulation (The Hook Pattern)
Never leave complex state transitions or side effects inside a UI component. If a component has more than 2-3 `useEffect` or `useState` calls, it is a candidate for a custom hook.

### 2. The "Headless" Pattern (Behavioral Encapsulation)
Encapsulate the **logic** of a UI element (like a modal or a combobox) but leave the **rendering** to the consumer. This is the ultimate form of encapsulation used by libraries like **TanStack Table** or **Radix UI**.

### 3. Prop Hiding
If a component needs 10 props but 7 of them are always used together, encapsulate them into a configuration object or a specialized "Partial" component.

---

## Code Examples

### Before: The "Tangled" Component
This component is hard to test and reuse because its logic (fetching) is welded to its UI.

```tsx
// ❌ Problem: UI and Logic are tightly coupled
const UserProfile = ({ id }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/user/${id}`).then(res => res.json()).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, [id]);

  if (loading) return <Spinner />;
  return <div>{user.name}</div>;
};
```

### After: Encapsulated Logic & UI
We separate the "How it works" from "How it looks."

```tsx
// ✅ Encapsulated Logic (The Hook)
const useUser = (id: string) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Logic hidden here...
    fetchUser(id).then(data => {
      setUser(data);
      setIsLoading(false);
    });
  }, [id]);

  return { user, isLoading };
};

// ✅ Encapsulated UI (The Black Box)
const UserProfile = ({ id }) => {
  const { user, isLoading } = useUser(id);

  if (isLoading) return <Spinner />;
  return <UserCard name={user.name} />;
};
```

---

## Performance Considerations

### 1. Selective Re-renders
By encapsulating state into a custom hook or a smaller sub-component, you prevent the entire parent tree from re-rendering when that specific piece of state changes.

### 2. Memoization Boundaries
Encapsulation creates natural boundaries for `React.memo`. When a component is a self-contained unit with a stable API, React can easily determine if it needs to skip rendering.

---

## Real-World Use Cases

1.  **Design Systems:** Encapsulating complex ARIA attributes and keyboard navigation logic inside a `Dropdown` component so developers don't have to manually manage `aria-expanded` or `tabIndex`.
2.  **Form Libraries:** Encapsulating the "Dirty," "Touched," and "Error" states of an input inside a `FormField` component.
3.  **Analytics:** Encapsulating tracking logic inside a wrapper component that sends an event to a server every time its child is clicked.

---

## Common Mistakes

- **Hardcoding External Dependencies:** A component that imports a global `Store` directly is not well-encapsulated.
    - *Fix:* Pass the store data in as props or via a specialized hook.
- **Prop Drilling:** Passing props through five layers of components.
    - *Fix:* Use **Composition** (passing components as children) or **Context** to encapsulate the data flow.
- **Leaky Styling:** Using global CSS classes that might affect other components.
    - *Fix:* Use **CSS Modules** or **Styled-Components** to encapsulate styles to the component scope.

---

## Summary & Key Takeaways

- **Black Box Principle:** The consumer should know *what* a component does, never *how* it does it.
- **Custom Hooks:** The primary tool for encapsulating stateful logic.
- **Separation of Concerns:** Keep your "Fetching" logic, "Transformation" logic, and "Rendering" logic in separate boxes.
- **Maintainability:** Encapsulated code is easier to refactor because the "blast radius" of a change is limited to a single file.
- **Interview Tip:** "I prioritize encapsulation by ensuring my components have a **narrow API surface**. By extracting complex logic into custom hooks and using composition to handle UI structure, I create 'pluggable' components that are easy to test in isolation and resilient to changes in the broader application state."