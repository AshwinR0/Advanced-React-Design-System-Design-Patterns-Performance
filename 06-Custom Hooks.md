# Custom Hooks

## Concept Explanation

### What is a Custom Hook?
A **Custom Hook** is a JavaScript function whose name starts with `use` and that can call other hooks (like `useState`, `useEffect`, `useContext`). It is a mechanism to extract component logic into reusable functions.

### Why is it Necessary?
Before hooks, sharing stateful logic between components required complex patterns like **Higher-Order Components (HOCs)** or **Render Props**. Custom hooks solve several architectural pain points:
1.  **Logic Reusability:** Share the same stateful logic (e.g., a subscription, a form handler, a fetch request) across multiple components without duplicating code.
2.  **Separation of Concerns:** Keep components "lean" by moving complex business logic out of the UI layer.
3.  **Readability:** Break down a massive component into smaller, specialized functions that handle specific parts of the state.
4.  **Testability:** Since custom hooks are just functions, they are significantly easier to unit test in isolation than logic buried inside a React component.

---

## Mental Models & Analogies

### The "Subroutine" for State
In traditional programming, when you have a block of code you use often, you extract it into a **function**. A Custom Hook is exactly that, but for **React State**. 

Think of a component as a **Smartphone**. The UI is the screen, but the logic (GPS, Battery Management, Connectivity) are **Custom Hooks**. Multiple different phone models (components) can use the exact same GPS logic (hook) without needing to know how the satellites work.

---

## Naming Conventions & Rules

### The `use` Prefix
The most critical convention is that a custom hook **must** start with the word `use` (e.g., `useAuth`, `useFetch`, `useLocalStorage`).

**Why?**
-   **Linter Integration:** React uses the `eslint-plugin-react-hooks` to ensure the "Rules of Hooks" are followed. The linter relies on the `use` prefix to identify which functions are hooks.
-   **Developer Intent:** It signals to other engineers that this function follows the specific rules of React (e.g., it can only be called at the top level, not inside loops or conditions).

---

## Design Patterns & Best Practices

### 1. Returning Data: Objects vs. Arrays
-   **Arrays (like `useState`):** Use when the user will likely want to rename the returned values (e.g., `const [val, setVal] = useMyHook()`).
-   **Objects (Most common for Custom Hooks):** Use when returning multiple values or functions. This makes the API more discoverable and prevents errors if more properties are added later.

### 2. Composition of Hooks
A great custom hook often calls other custom hooks. This "layered" approach allows you to build highly complex logic from small, simple primitives.

---

## Code Examples

### Before: Logic Tangled in UI
```tsx
const UserProfile = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(user => {
      setData(user);
      setLoading(false);
    });
  }, []);

  if (loading) return <p>Loading...</p>;
  return <div>{data.name}</div>;
};
```

### After: Logic Extracted into Custom Hook
```tsx
// useUser.ts
import { useState, useEffect } from 'react';

export const useUser = (userId: string) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/user/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setIsLoading(false);
      });
  }, [userId]);

  return { user, isLoading };
};

// UserProfile.tsx
const UserProfile = ({ id }) => {
  const { user, isLoading } = useUser(id); // Clean, readable UI
  
  if (isLoading) return <p>Loading...</p>;
  return <div>{user.name}</div>;
};
```

---

## Performance Considerations

### 1. Referential Integrity
If your hook returns a function, wrap that function in `useCallback`. If it returns a derived object, wrap it in `useMemo`. This prevents components using your hook from re-rendering unnecessarily when the hook's parent re-renders.

### 2. Dependency Management
Always be explicit with your `useEffect` or `useMemo` dependencies inside the hook. A custom hook that misses a dependency can lead to "stale closures" where the hook uses old data.

---

## Real-World Use Cases

1.  **Form Management:** `useForm` to handle inputs, validation, and submission state.
2.  **Authentication:** `useAuth` to provide the current user object and `login/logout` methods.
3.  **Media Queries:** `useBreakpoint` to return a boolean based on the window width for responsive JS logic.
4.  **Browser APIs:** `useLocalStorage`, `useClipboard`, or `useEventListener`.

---

## Common Mistakes (Anti-patterns)

-   **Conditional Calling:** Never call a custom hook inside an `if` statement. Hooks must be called in the same order every time a component renders.
-   **Too Many Responsibilities:** A hook called `useUserAndProductsAndOrders` is a red flag. Split them into `useUser`, `useProducts`, and `useOrders`.
-   **State Synching:** Using a custom hook to copy a prop into state and keep it in sync. This is usually unnecessary and leads to "Single Source of Truth" bugs.

---

## Summary & Key Takeaways

-   **Custom Hooks** extract stateful logic for reuse.
-   **Naming:** Must start with `use`.
-   **Benefits:** Better testing, cleaner UI components, and logic composition.
-   **Interview Tip:** "I use custom hooks to encapsulate business logic. This makes my components purely presentational and allows me to test the logic independently using tools like `@testing-library/react-hooks`."

# Progressive Data Fetching with Custom Hooks

## Concept Explanation
In modern React, **Data Fetching Hooks** are the standard for managing asynchronous state. While HOCs and Container Components manipulate the *structure* of the component tree, Custom Hooks encapsulate the *logic* of the request lifecycle (Idle → Loading → Success/Error).

### The Evolution of Fetching Hooks
1.  **Domain Hooks (`useUser`):** Hardcoded for specific business entities.
2.  **Resource Hooks (`useResource`):** URL-driven and entity-agnostic.
3.  **DataSource Hooks (`useDataSource`):** Protocol-agnostic (the "Staff-Level" abstraction).

---

## 1. Domain-Specific Hook: `useUser`
This hook is tailored to a single user. It is easy to use but tightly coupled to a specific API endpoint.

### Code Example
```tsx
import { useState, useEffect } from 'react';

export const useUser = (userId: string) => {
  const [user, setUser] = useState<any>(null);

  useEffect(() => {
    (async () => {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data);
    })();
  }, [userId]);

  return user;
};

// Usage
// const user = useUser('123');
```

---

## 2. Collection Hook: `useUsers`
When dealing with lists, the logic remains similar, but the hook is optimized for handling arrays and potentially filtering/pagination logic.

### Code Example
```tsx
export const useUsers = () => {
  const [users, setUsers] = useState<any[]>([]);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(setUsers);
  }, []);

  return users;
};

// Usage
// const users = useUsers();
```

---

## 3. The Generic Hook: `useResource`
To follow **DRY (Don't Repeat Yourself)** principles, we create a hook that accepts a URL. This single hook can now replace `useUser`, `useProduct`, and `useComments`.

### Code Example
```tsx
export const useResource = (resourceUrl: string) => {
  const [resource, setResource] = useState<any>(null);

  useEffect(() => {
    (async () => {
      const response = await fetch(resourceUrl);
      const data = await response.json();
      setResource(data);
    })();
  }, [resourceUrl]);

  return resource;
};

// Usage
// const user = useResource('/api/users/1');
// const products = useResource('/api/products');
```

---

## 4. The Staff-Level Abstraction: `useDataSource`
The most generic version of this pattern. It doesn't even assume you are using `fetch`. It accepts a function that returns data. This allows the data to come from **LocalStorage**, **Firebase**, **GraphQL**, or a **Mock JSON**.

### Code Example
```tsx
export const useDataSource = <T,>(getDataFunc: () => Promise<T>) => {
  const [resource, setResource] = useState<T | null>(null);

  useEffect(() => {
    (async () => {
      const result = await getDataFunc();
      setResource(result);
    })();
  }, [getDataFunc]);

  return resource;
};

// Usage
const fetchUser = async () => {
    const response = await fetch('/api/users/123');
    return response.json();
};

const user = useDataSource(fetchUser);
```

---

## Mental Models & Analogies

### The "Faucet" Metaphor
- **`useUser`** is a specialized faucet that only dispenses filtered drinking water.
- **`useResource`** is a standard faucet. You attach a hose (URL) and it dispenses whatever is in the pipe.
- **`useDataSource`** is the **Plumbing System** itself. It doesn't care if the water comes from a well, a city pipe, or a tank (the `getDataFunc`). It just ensures the water reaches the tap.

---

## Performance Considerations

### 1. Referential Integrity (The "Infinite Loop" Bug)
In the `useDataSource` example, `getDataFunc` is a dependency of `useEffect`. If the parent component defines that function inline, a new function reference is created on every render, triggering an infinite fetch loop.
*   **Fix:** Wrap the fetching function in `useCallback` at the parent level or define it outside the component.

### 2. Loading & Error States
A professional hook shouldn't just return data; it should return the status.
```tsx
return { data, isLoading, error };
```

### 3. Cleanup & Race Conditions
If a user clicks "User 1" then immediately "User 2," the request for User 1 might finish *after* User 2, overwriting the state with wrong data.
*   **Fix:** Use an `AbortController` or a local "ignore" flag inside `useEffect`.

---

## Real-World Use Cases

1.  **Feature Toggles:** A `useDataSource` hook that pulls configuration from a remote dashboard.
2.  **Multilingual Apps:** `useResource('/locales/en.json')` to fetch translation files.
3.  **Offline Support:** A hook that attempts to fetch from a URL, but falls back to a LocalStorage function if the fetch fails.

---

## Summary & Key Takeaways

- **Progressive Abstraction:** Start specific (`useUser`), but abstract when you see patterns (`useResource`).
- **Decoupling:** `useDataSource` is the gold standard because it decouples your UI logic from your network protocol.
- **Rules of Hooks:** Always call these at the top level and ensure the `use` prefix is present.
- **Interview Tip:** "I prefer building generic hooks like `useDataSource` because they are protocol-agnostic. This makes the UI easier to test with mock data and allows us to switch from REST to GraphQL or Firebase in the future without changing a single line of component code."