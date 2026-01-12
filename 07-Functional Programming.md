# Functional Programming in React

## Concept Explanation

**Functional Programming (FP)** is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data. In the context of React, FP is the architectural backbone that makes the library predictable and performant.

### What the Concept Is
At its core, FP in React is about **Data In $\rightarrow$ UI Out**. React components are essentially functions that take `props` (and `state`) as input and return a description of the UI (JSX) as output.

### Why it Exists
Traditional imperative programming relies on a "step-by-step" modification of the DOM (e.g., `document.getElementById('id').innerHTML = 'new value'`). This becomes unmanageable in complex UIs. FP allows React to:
- **Predict the UI:** If the props are the same, the output is the same.
- **Optimize Rendering:** By using immutable data, React can perform "shallow equality" checks to skip unnecessary re-renders.
- **Isolate Side Effects:** Patterns like `useEffect` create explicit boundaries for "dirty" logic (API calls, DOM mutations), keeping the rest of the app "pure."

---

## Mental Models & Analogies

### The "Black Box" (Pure Function)
Think of a pure component as a high-quality juicer. You put in oranges (Props), and you always get orange juice (JSX). The juicer doesn't suddenly change the color of your kitchen walls or start making toast (Side Effects). It does exactly one thing based strictly on what you put in.

### The "Lego Brick" (Composition)
In FP, you don't build a giant "Castle" object. You build small, perfect "bricks" (Functions/Components) and snap them together. The Castle is just the result of how you **composed** the bricks.

---

## Design Patterns & Best Practices

### 1. Pure Components
A component is "pure" if it renders the same output for the same props and has no side effects.
- **Best Practice:** Keep logic out of the render body. If a calculation is expensive, move it to a helper function or wrap it in `useMemo`.

### 2. Immutability
Never modify objects or arrays directly. Always create a copy.
- **Pattern:** `const nextState = [...prevState, newItem];`
- **Why:** React uses referential equality (`prevState === nextState`) to determine if it should re-render. If you mutate the original, the reference stays the same, and React might miss the update.

### 3. Higher-Order Functions & Currying
**Currying** is a technique where a function with multiple arguments is transformed into a sequence of functions, each taking a single argument. In React, this is incredibly useful for event handlers.

---

## Code Examples

### Currying for Dynamic Event Handlers
Instead of creating multiple anonymous functions in your JSX, use currying to generate specific handlers.

```tsx
// ✅ Advanced: Using Currying to handle multiple form fields
const MyForm = () => {
  const [formState, setFormState] = useState({ name: '', email: '' });

  // The Curried Function: (fieldName) => (event) => void
  const updateField = (fieldName: string) => (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormState(prev => ({
      ...prev,
      [fieldName]: e.target.value
    }));
  };

  return (
    <form>
      {/* updateField('name') returns the actual (e) => {} function */}
      <input onChange={updateField('name')} value={formState.name} />
      <input onChange={updateField('email')} value={formState.email} />
    </form>
  );
};
```

### Declarative Data Transformation
Avoid `for` loops inside components. Use `map`, `filter`, and `reduce`.

```tsx
// ✅ Pure transformation: Abstracting logic outside the component
const formatUserData = (users: User[]) => 
  users
    .filter(user => user.isActive)
    .map(user => ({ ...user, name: user.name.toUpperCase() }));

const UserList = ({ rawUsers }: { rawUsers: User[] }) => {
  const users = useMemo(() => formatUserData(rawUsers), [rawUsers]);
  
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
};
```

---

## Performance Considerations

### 1. Referential Transparency
In FP, a function is "referentially transparent" if it can be replaced by its value without changing the program's behavior. React leverages this with **Memoization**. 
- If a component is pure, `React.memo` will skip the entire rendering process if the props haven't changed.

### 2. Allocation Costs
FP creates many new objects. In very tight loops, this can trigger Garbage Collection (GC) pauses.
- **Optimization:** Use libraries like `Immer` for complex state updates—it provides a "draft" state that looks like mutation but handles immutability under the hood efficiently.

---

## Real-World Use Cases

1.  **Redux Reducers:** The quintessential FP pattern in React. `(state, action) => nextState` is a pure function.
2.  **Design Systems:** Highly composable components (e.g., `<Stack>`, `<Box>`) that behave like utility functions.
3.  **Utility Libraries:** Using `Ramda` or `Lodash/fp` for complex data piping in large-scale dashboards (e.g., `pipe(filter, sort, groupByCategory)`).

---

## Common Mistakes

- **Impure Logic in Render:** `const id = Math.random();` inside a component. This generates a new ID on every render, breaking child component reconciliation.
- **Mutating Props:** `props.user.name = 'New Name';` Props must be treated as read-only. Modifying them causes unpredictable behavior across the component tree.
- **Side Effects in Reducers:** Putting an API call inside a reducer. Reducers must be 100% deterministic.

---

## Summary & Key Takeaways

- **FP Principle:** UI is a function of State ($UI = f(State)$).
- **Immutability:** Essential for React’s change detection. Always return new references.
- **Purity:** Pure functions make testing and debugging trivial.
- **Interview Tip:** "I embrace functional programming in React because it enforces **Referential Transparency**. This allows me to use `React.memo` and `useMemo` confidently, ensuring that the UI remains predictable and performance stays optimized even as the application scales."

# Advanced Component Patterns

## 1. Recursive Components

### Concept Explanation
A **Recursive Component** is a component that calls itself within its own definition. This pattern is essential for rendering data structures with an **unknown depth**, such as nested trees, file systems, or threaded comment sections.

*   **Problem it Solves:** Standard loops (`.map`) only work for flat arrays or fixed-depth nesting. Recursion allows the UI to automatically adapt to data nested $N$ levels deep.

### Mental Models & Analogies
**The Matryoshka (Russian Nesting) Doll:** You open a doll (Component), and inside is another version of the same doll, perhaps slightly smaller or with different attributes, until you reach the smallest doll that cannot be opened (**The Base Case**).

### Code Example: File Explorer
```tsx
interface FileNode {
  name: string;
  children?: FileNode[];
}

const RecursiveTree = ({ data }: { data: FileNode[] }) => {
  return (
    <ul style={{ paddingLeft: '20px' }}>
      {data.map((item) => (
        <li key={item.name}>
          {item.name}
          {/* The Base Case: Only recurse if children exist */}
          {item.children && item.children.length > 0 && (
            <RecursiveTree data={item.children} />
          )}
        </li>
      ))}
    </ul>
  );
};
```

### Performance Considerations
*   **Stack Depth:** Extremely deep recursion can lead to performance hits. 
*   **Optimization:** Use `React.memo` on the recursive component to prevent the entire tree from re-rendering when only one branch changes.

---

## 2. Composition Components

### Concept Explanation
**Composition** is the core philosophy of React ("Composition over Inheritance"). It involves building complex components by assembling simpler, smaller components using the `children` prop or "slots."

*   **Problem it Solves:** Prevents "Prop Drilling" and "God Components" (components with 50+ props trying to handle every possible configuration).

### Mental Models & Analogies
**The Picture Frame:** The frame (Parent) provides the structure and border, but it doesn't care if you put a photo, a painting, or a mirror (Children) inside it. The frame provides the **context**, the child provides the **content**.

### Design Patterns: The "Slot" Pattern
Instead of just one `children` prop, you can use specific props for different areas of the UI.

```tsx
interface LayoutProps {
  navigation: React.ReactNode;
  content: React.ReactNode;
  footer?: React.ReactNode;
}

const PageLayout = ({ navigation, content, footer }: LayoutProps) => (
  <div className="layout">
    <header>{navigation}</header>
    <main>{content}</main>
    {footer && <footer>{footer}</footer>}
  </div>
);

// Usage:
// <PageLayout 
//    navigation={<Navbar />} 
//    content={<DashboardContent />} 
// />
```

### Real-World Use Cases
*   **Modals:** A generic modal wrapper that accepts any form or message as children.
*   **Design Systems:** Building a `Card` component that composed of `Card.Header`, `Card.Body`, and `Card.Footer`.

---

## 3. Partial Components

### Concept Explanation
**Partial Components** (inspired by "Partial Application" in Functional Programming) are created by taking a generic component and "pre-filling" some of its props to create a more specialized version.

*   **Problem it Solves:** Reduces repetitive prop passing. If you use a `<Button variant="primary" size="large" color="blue" />` fifty times, a Partial Component centralizes those settings.

### Mental Models & Analogies
**The Custom Order:** You have a "Generic Pizza" recipe. A "Partial" is like having a "Pepperoni Pizza" template where the toppings are already decided, and the chef only needs to know the size.

### Code Example: Specializing a Component
You can implement this using a simple wrapper or a Higher-Order Function.

```tsx
// Generic Component
const Button = ({ size, color, text, onClick }: any) => (
  <button style={{ fontSize: size === 'lg' ? '20px' : '14px', color }} onClick={onClick}>
    {text}
  </button>
);

// Partial Components
export const PrimaryButton = (props: any) => (
  <Button {...props} size="lg" color="blue" />
);

export const DangerButton = (props: any) => (
  <Button {...props} size="sm" color="red" />
);

// Usage
// <PrimaryButton text="Save Changes" onClick={save} />
```

### Design Patterns & Best Practices
*   **Avoid Over-Wrapping:** Don't create a partial for every single use case. Only create them for recurring patterns in your Design System (e.g., `SuccessToast`, `ErrorModal`).
*   **Naming:** Use descriptive prefixes that indicate specialization (e.g., `SmallInput`, `BigHeader`).

---

## Performance & Optimization Summary

| Pattern | Render Impact | Bundle Size | Optimization Technique |
| :--- | :--- | :--- | :--- |
| **Recursive** | High (Multi-level) | Low | `React.memo` & Base Case checks. |
| **Composition** | Medium (Prop passing) | Low | Use `Context` for deep composition to avoid drilling. |
| **Partial** | Minimal | Low | Ensure `...props` are passed correctly to maintain event handlers. |

---

## Common Mistakes (Anti-patterns)

1.  **Infinite Recursion:** Forgetting the base case in a Recursive component, leading to a stack overflow/browser crash.
2.  **Composition Overkill:** Wrapping a component in so many layers of composition that it becomes impossible to trace where a specific style or behavior originates.
3.  **Hardcoding in Partials:** Forgetting to spread `{...props}` in a Partial component, which accidentally blocks the user from adding new props or event listeners (like `className` or `onClick`).

---

## Summary & Key Takeaways

*   **Recursive Components** are for **Tree-like Data**. They require a clear exit condition to prevent infinite loops.
*   **Composition** is for **Structural Flexibility**. Use it to keep components "dumb" and reusable by letting the parent define what lives inside.
*   **Partial Components** are for **Configuration Reuse**. They act as templates that lock in specific props to create specialized UI elements.
*   **Interview Friendly Tip:** "I utilize **Composition** to keep my UI library flat and flexible, but when dealing with hierarchical data like organization charts, I implement **Recursive Components**. To maintain a dry codebase, I use **Partial Components** to create specialized variants of generic base components."