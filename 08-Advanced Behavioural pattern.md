# Advanced Behavioral Patterns
## Compound Components & Observer Pattern

This documentation covers two powerful behavioral patterns used to manage complex component communication and state synchronization in large-scale React applications.

---

## 1. Compound Components Pattern

### Concept Explanation
**Compound Components** are a set of components that work together to form a single cohesive unit, sharing implicit state without requiring the consumer to pass props to every individual sub-component. 

*   **What it is:** A parent component that holds state and provides it to its children via Context.
*   **Why it exists:** It solves the "Prop Drilling" and "API Bloat" problems. Instead of a single component with 20 props (`<Select options={...} isOpen={...} onSelect={...} />`), you provide a set of expressive sub-components.
*   **Problem solved:** It gives the consumer **control over the UI structure** while the parent handles the **behavioral logic**.

### Mental Models & Analogies
**The Restaurant Menu:** A "Menu" is the parent. You have "Items," "Headers," and "Sections." You don't tell the Menu component about every item in a big list; you place the items inside the Menu. The Menu knows which item is highlighted or selected, but you decide exactly where the "Header" or the "Footer" of that menu is placed.

### Code Example: A Tab System
```tsx
import React, { useState, createContext, useContext, ReactNode } from 'react';

// 1. Create the Context
const TabContext = createContext<{
  activeTab: string;
  setActiveTab: (id: string) => void;
} | null>(null);

// 2. Parent Component
export const Tabs = ({ children, defaultTab }: { children: ReactNode; defaultTab: string }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs-container">{children}</div>
    </TabContext.Provider>
  );
};

// 3. Child Components (Sub-components)
Tabs.Tab = ({ id, children }: { id: string; children: ReactNode }) => {
  const context = useContext(TabContext);
  if (!context) throw new Error("Tab must be used within Tabs");
  
  return (
    <button 
      onClick={() => context.setActiveTab(id)}
      style={{ fontWeight: context.activeTab === id ? 'bold' : 'normal' }}
    >
      {children}
    </button>
  );
};

Tabs.Panel = ({ id, children }: { id: string; children: ReactNode }) => {
  const context = useContext(TabContext);
  if (!context) throw new Error("Panel must be used within Tabs");
  
  return context.activeTab === id ? <div>{children}</div> : null;
};

// --- Usage ---
// <Tabs defaultTab="home">
//   <Tabs.Tab id="home">Home</Tabs.Tab>
//   <Tabs.Tab id="settings">Settings</Tabs.Tab>
//   <Tabs.Panel id="home">Welcome home!</Tabs.Panel>
//   <Tabs.Panel id="settings">Configuration here.</Tabs.Panel>
// </Tabs>
```

### Real-World Use Cases
-   **UI Libraries:** (e.g., Radix UI, Headless UI, Ant Design).
-   **Forms:** Form, Form.Field, Form.Error, Form.Submit.
-   **Modals:** Modal.Header, Modal.Body, Modal.Footer.

---

## 2. Observer / Subscriber Pattern

### Concept Explanation
The **Observer Pattern** is a behavioral pattern where an object (the **Subject/Emitter**) maintains a list of dependents (**Observers**) and notifies them automatically of any state changes, usually by calling one of their methods.

*   **In React:** This is used to handle "Cross-Component" communication that doesn't follow the parent-child hierarchy.
*   **Why it exists:** React's "lifting state up" becomes a bottleneck if two components at opposite ends of the tree need to sync (e.g., a "Theme Toggle" in the settings page and a "Chart" in the dashboard).

### Mental Models & Analogies
**The Radio Station:** The station (Subject) broadcasts a signal. It doesn't know who is listening. Any radio (Observer) that is tuned to that specific frequency (Subscribed) will play the music. When the station changes the song, all tuned-in radios update simultaneously.

### Code Example: Modern Observer with `useSyncExternalStore`
In modern React (18+), we use `useSyncExternalStore` to subscribe to external data sources safely without tearing.

```tsx
// 1. The External Store (Vanilla JS)
class EventEmitter {
  private observers: Set<Function> = new Set();

  subscribe(callback: Function) {
    this.observers.add(callback);
    return () => this.observers.delete(callback); // Unsubscribe
  }

  emit(data: any) {
    this.observers.forEach((callback) => callback(data));
  }
}

const globalEmitter = new EventEmitter();
let globalState = { count: 0 };

const store = {
  increment: () => {
    globalState = { count: globalState.count + 1 };
    globalEmitter.emit(globalState);
  },
  subscribe: (callback: () => void) => globalEmitter.subscribe(callback),
  getSnapshot: () => globalState,
};

// 2. The Hook / Component
import { useSyncExternalStore } from 'react';

export const CounterDisplay = () => {
  const state = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <h1>Count: {state.count}</h1>;
};
```

---

## Performance Considerations

### Compound Components
-   **Context Over-rendering:** Every time the parent state changes, all consumers of that Context re-render. 
-   **Optimization:** Split context into `StateContext` and `DispatchContext` to prevent components that only trigger actions (like a Close Button) from re-rendering when the state changes.

### Observer Pattern
-   **Memory Leaks:** Always return a cleanup function in `useEffect` (or the subscription function) to unsubscribe when the component unmounts.
-   **Tearing:** In concurrent React, external stores can "tear" (show different values for the same state in different parts of the UI). Use `useSyncExternalStore` to prevent this.

---

## Common Mistakes

1.  **Compound Components - Direct Children Only:** Using `React.Children.map` instead of `Context`. If you use `Children.map`, the pattern breaks if you wrap a sub-component in a `div`. **Always use Context.**
2.  **Observer Pattern - Stale Closures:** Subscribing inside `useEffect` without properly handling dependencies can lead to the observer using old variable values.
3.  **Hiding the Pattern:** Not naming sub-components as properties of the parent (e.g., `Tabs.Tab`). This makes it harder for developers to discover the associated components.

---

## Summary & Key Takeaways

-   **Compound Components** manage **Implicit State**. They are the gold standard for flexible UI components in Design Systems.
-   **Observer Pattern** manages **Global Events**. It is best for decoupled logic (e.g., event buses, global notifications, or non-React state).
-   **API Design:** Use Compound Components when components are **visually related**; use Observers when components are **logically related but visually distant**.
-   **Interview Tip:** "I use **Compound Components** to create 'Headless' UI logic that allows consumers to define their own DOM structure while I handle the state. For global, non-hierarchical updates, I implement an **Observer pattern** using `useSyncExternalStore` to ensure thread-safe state synchronization across the app."