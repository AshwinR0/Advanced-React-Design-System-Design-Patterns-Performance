# Higher-Order Components (HOCs)

## Concept Explanation

### What is an HOC?
A **Higher-Order Component (HOC)** is an advanced pattern in React for reusing component logic. Unlike a standard component which transforms props into UI, an HOC is a **function** that takes a component and returns a new, "enhanced" component.

Mathematically, it looks like this:  
`const EnhancedComponent = higherOrderComponent(OriginalComponent);`

### Why do we create an HOC?
The primary goal of an HOC is **Cross-Cutting Concerns**. It allows you to abstract logic that is shared across many components into a single, reusable wrapper.

1.  **Code Reuse:** Avoid duplicating logic for things like authentication checks, data fetching, or event logging.
2.  **Logic Abstraction:** Separate the *behavior* of a component from its *representation*.
3.  **Prop Manipulation:** Add, remove, or transform props before they reach the wrapped component.
4.  **State Management:** High-level state can be managed in the HOC and passed down to "dumb" presentational components.

---

## Mental Models & Analogies

### The "Security Badge" Analogy
Imagine a generic office building (your component). Any person can walk in. Now, imagine a **Security Gate (HOC)**. 
- You don't rebuild the office building to include security logic. 
- Instead, you wrap the entrance with the Security Gate. 
- The gate checks the person's ID (Logic) and then either lets them in with a "Visitor Badge" (New Prop) or turns them away (Conditional Rendering).

The office building remains "pure"—it only cares about what happens inside. The gate handles the "cross-cutting" concern of security.

---

## Design Patterns & Best Practices

### 1. The "With" Convention
HOCs should always be prefixed with `with` to signal their nature (e.g., `withAuth`, `withLoading`, `withTheme`).

### 2. Pass Through Unrelated Props
An HOC should not interfere with the props that are intended for the original component. Use the spread operator to pass them through.

### 3. Static Methods
When you wrap a component, the new component does not have the static methods of the original. You must manually copy them over (using libraries like `hoist-non-react-statics`).

---

## Code Examples

### The Problem: Redundant Logic
Imagine two components that both need to fetch data from an API and show a loading spinner.

```tsx
// ❌ Bad: Logic duplicated in UserProfile and ProductList
const UserProfile = () => {
  const [data, setData] = useState(null);
  if (!data) return <Loading />;
  return <div>{data.name}</div>;
};
```

### The Solution: An HOC for Data Fetching
We create `withData` to handle the fetching and loading state.

```tsx
import React, { ComponentType, useState, useEffect } from 'react';

// The HOC Function
export const withData = <P extends object>(
  WrappedComponent: ComponentType<P>,
  dataSource: string
) => {
  return (props: P) => {
    const [data, setData] = useState(null);

    useEffect(() => {
      fetch(dataSource)
        .then(res => res.json())
        .then(setData);
    }, []);

    if (!data) return <div>Loading...</div>;

    // We pass the fetched data as a prop and pass through existing props
    return <WrappedComponent {...props} data={data} />;
  };
};

// Usage
const UserInfo = ({ data }: { data: any }) => <div>{data.name}</div>;
const UserInfoWithData = withData(UserInfo, '/api/user/1');
```

---

## HOCs vs. Custom Hooks

Since React 16.8, Custom Hooks have replaced HOCs for most logic-sharing needs. However, HOCs still have a place in specific scenarios:

| Feature | Higher-Order Components (HOCs) | Custom Hooks |
| :--- | :--- | :--- |
| **Primary Use** | Enhancing a component's structure/props. | Sharing stateful logic. |
| **Composition** | Happens at the export level (Static). | Happens inside the function (Dynamic). |
| **UI Manipulation** | Can wrap components in specific UI (e.g., Error Boundaries). | Cannot return JSX elements. |
| **Nesting** | Can lead to "Wrapper Hell." | Flat and easy to read. |

---

## Performance Considerations

### 1. Don't use HOCs inside the render method
React's reconciliation process uses component identity to determine whether it should update the existing subtree or throw it away and mount a new one. If you create an HOC inside `render`, the component identity changes on every render, causing a full unmount/remount of the entire subtree.

```tsx
// ❌ Anti-pattern
render() {
  const Enhanced = withAuth(MyComponent); // New reference every time!
  return <Enhanced />;
}
```

### 2. Memoization
If the HOC injects props that change frequently, ensure you use `React.memo` inside the HOC to prevent unnecessary re-renders of the wrapped component.

---

## Real-World Use Cases

1.  **Authentication/Authorization:** `withAdmin(Dashboard)` ensures only admins can see the component; otherwise, it redirects.
2.  **Theming:** Injecting theme objects into legacy class components that can't use `useContext`.
3.  **Redux `connect`:** The most famous HOC in the ecosystem, mapping store state to component props.
4.  **Relay/GraphQL:** Injecting data fragments into components via `createFragmentContainer`.

---

## Common Mistakes

- **Prop Collisions:** The HOC injects a prop called `data`, but the original component already had a prop called `data`. The HOC will overwrite it unless carefully managed.
- **Deep Nesting:** Wrapping a component in 5 HOCs (`withAuth(withTheme(withLogger(withRouter(MyComp))))`) makes it extremely difficult to trace where a specific prop is coming from.
- **Incompatibility with Refs:** By default, refs don't pass through HOCs. You must use `React.forwardRef` inside the HOC to ensure `ref={myRef}` works on the wrapped component.

---

## Summary & Key Takeaways

- **HOCs are functions** that return a new component with added "powers."
- **Use them for structural concerns** (e.g., Error Boundaries, Auth Redirects, Portals).
- **Use Hooks for logic concerns** (e.g., Form handling, Fetching data, Timers).
- **Interview Tip:** "I use HOCs when I need to enhance a component without modifying its implementation, particularly for cross-cutting concerns that involve wrapping the component in specific UI or logic that must exist outside the component's render cycle."

## Advanced Higher-Order Component (HOC) Patterns

A **Higher-Order Component (HOC)** is a functional programming pattern applied to React components. At its core, it is a **decorator**. While hooks handle internal logic sharing, HOCs excel at **structural augmentation** and **prop-driven orchestration**.

In a production environment, HOCs are used to create "Super-Components" by wrapping a base component and injecting new capabilities, data, or restrictions.

---

## 1. Checking Props with HOC (The "Guard" Pattern)
This pattern is used to intercept props before they reach the wrapped component. If certain conditions aren't met (e.g., missing data, unauthorized role), the HOC can modify the props, redirect the user, or render a fallback UI.

**Real-world use case:** Restricting sensitive UI elements to "Admin" users only.

```tsx
// withAdmin.tsx
export const withAdmin = (WrappedComponent: React.ComponentType) => {
  return (props: any) => {
    const { userRole } = props;

    if (userRole !== 'admin') {
      return <div className="error">Access Denied: Admins Only</div>;
    }

    return <WrappedComponent {...props} />;
  };
};

// Usage
// const AdminDashboard = withAdmin(Dashboard);
```

---

## 2. Data Loading with HOC
This pattern decouples "Data Fetching" from "Data Display." The HOC manages the lifecycle (componentDidMount/useEffect), fetches the resource, and passes the result as a prop.

```tsx
// withUser.tsx
export const withUser = (WrappedComponent: React.ComponentType, userId: string) => {
  return (props: any) => {
    const [user, setUser] = useState(null);

    useEffect(() => {
      fetch(`/api/users/${userId}`)
        .then(res => res.json())
        .then(setUser);
    }, []);

    return <WrappedComponent {...props} user={user} />;
  };
};
```

---

## 3. Updating Data with HOC
Beyond just loading, HOCs can provide **Mutation Logic**. By passing down functions like `onUpdateUser`, the HOC allows the child to change data without knowing how the API works.

```tsx
// withEditableUser.tsx
export const withEditableUser = (WrappedComponent: React.ComponentType, userId: string) => {
  return (props: any) => {
    const [user, setUser] = useState(null);

    const onUpdateUser = async (updates: any) => {
      const response = await fetch(`/api/users/${userId}`, {
        method: 'POST',
        body: JSON.stringify(updates),
      });
      const data = await response.json();
      setUser(data);
    };

    return <WrappedComponent {...props} user={user} onUpdateUser={onUpdateUser} />;
  };
};
```

---

## 4. Building Forms with HOC
This is a classic "Controller" pattern. The HOC manages the form state (`values`, `errors`, `touched`) and provides handlers (`handleChange`, `handleSubmit`). This turns the child component into a purely presentational collection of inputs.

```tsx
// withForm.tsx
export const withForm = (WrappedComponent: React.ComponentType, initialValues: any) => {
  return (props: any) => {
    const [values, setValues] = useState(initialValues);

    const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
      setValues({ ...values, [e.target.name]: e.target.value });
    };

    const onSubmit = () => {
      console.log('Submitting values:', values);
    };

    return (
      <WrappedComponent 
        {...props} 
        formValues={values} 
        handleFormChange={onChange} 
        handleFormSubmit={onSubmit} 
      />
    );
  };
};
```

---

## 5. Enhancing the HOC Pattern (Staff-Level Best Practices)

To build HOCs that work at scale in large design systems, you must address three critical technical areas:

### A. Forwarding Refs
By default, refs don't pass through HOCs. If you wrap a `Button` in an HOC, `ref` will point to the HOC wrapper, not the `Button` DOM node. 
**Solution:** Use `React.forwardRef`.

### B. Display Name for Debugging
React DevTools will show `Anonymous` for components wrapped in HOCs.
**Solution:** Set the `displayName`.

### C. TypeScript Generics
To prevent HOCs from destroying your type safety, use Generics to ensure the HOC doesn't interfere with the wrapped component's existing props.

#### The "Staff-Level" Implementation Template:
```tsx
import React, { ComponentType, forwardRef, Ref } from 'react';

// 1. Use Generics <P> to preserve props
export function withLogger<P extends object>(WrappedComponent: ComponentType<P>) {
  
  const hoc = (props: P, ref: Ref<any>) => {
    useEffect(() => {
      console.log('Component mounted:', WrappedComponent.displayName || WrappedComponent.name);
    }, []);

    // 2. Forward the ref to the underlying component
    return <WrappedComponent {...props} ref={ref} />;
  };

  // 3. Provide a clear Display Name
  const wrappedComponentName = WrappedComponent.displayName || WrappedComponent.name || 'Component';
  hoc.displayName = `withLogger(${wrappedComponentName})`;

  return forwardRef(hoc);
}
```

---

## Performance Considerations

1.  **Avoid Prop Overwriting:** Always spread props (`{...props}`) **before** your HOC's injected props if you want to allow the consumer to override HOC values, or **after** if the HOC must enforce its values.
2.  **Referential Integrity:** If your HOC passes a function (like `onUpdate`), wrap it in `useCallback` inside the HOC. Otherwise, the child component will re-render every time the HOC re-renders, even if nothing changed.
3.  **Static Hoisting:** HOCs don't automatically copy static methods from the wrapped component. Use `hoist-non-react-statics` to fix this.

---

## Real-World Use Cases

- **Analytics/Logging:** A `withTracking` HOC that reports a "view" event when a component enters the viewport.
- **Theming:** Injecting theme variables into legacy class components.
- **Feature Flags:** A `withFeatureFlag('new-header')` HOC that swaps between two components based on a remote config toggle.

---

## Summary & Key Takeaways

- **Prop Checking:** Use HOCs as "Guards" for conditional rendering and validation.
- **Data Orchestration:** HOCs can bridge the gap between APIs and UI, handling both Loading and Updating logic.
- **Forms:** Abstract complex form state into an HOC to keep UI components "dumb."
- **Professional Standard:** Always **forward refs**, set **display names**, and use **TS Generics**.
- **Interview Friendly Answer:** *"HOCs are a powerful tool for structural composition. While hooks are great for sharing logic inside a component, HOCs are better when you need to wrap components to provide them with external data or context, especially when building highly reusable UI libraries or enforcing cross-cutting concerns like security."*