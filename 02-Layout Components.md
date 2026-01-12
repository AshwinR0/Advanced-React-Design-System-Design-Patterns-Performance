# Layout Components Pattern

## Concept Explanation

**Layout Components** are a structural design pattern in React where specific components are tasked exclusively with the **positioning, alignment, and distribution of space** between other components. 

In a traditional React component, developers often mix logic (data fetching, state) with layout (divs, flexbox, margins). Layout components decouple these concerns. They treat other components as "data" or "children" and focus entirely on the visual skeleton of the application.

### The Problem it Solves
- **Rigid Styling:** Prevents "margin-leaking," where a component defines its own outer spacing, making it hard to reuse in different contexts.
- **Code Duplication:** Eliminates repetitive flexbox/grid configurations across multiple screens.
- **Maintenance:** Centralizes the "look and feel" of the app’s structure. Changing a site-wide gutter becomes a single-file change.

---

## Mental Models & Analogies

### The "Picture Frame" Metaphor
Think of a **Feature Component** as the artwork and a **Layout Component** as the frame. 
- The artwork (e.g., a `UserList`) shouldn't care if it's hanging in a gallery, a hallway, or a small bedroom. 
- The frame (e.g., a `Split` layout) doesn't care if it's holding an oil painting or a photograph; it only cares about the dimensions, the border, and how it sits on the wall.

---

## Design Patterns & Best Practices

### 1. The "Slot" Pattern (Composition)
Instead of passing data, you pass fully formed React elements as props. This allows the Layout component to place them in specific "slots."

### 2. The "Stack" and "Split" Patterns
These are the two pillars of layout-driven design systems:
- **Stack:** Handles vertical spacing between elements.
- **Split:** Handles horizontal distribution (e.g., a sidebar and a main content area).

### 3. Avoiding "Margin Leaking"
**Rule:** A reusable component should never have outer margins. It should be "space-neutral." The Layout component (the parent) is responsible for providing the space between its children.

---

## Code Examples

### Before: The "Hardcoded" Approach (Anti-pattern)
The `UserCard` is hard to reuse because it has a fixed width and margin.

```tsx
// ❌ Problem: This card dictates its own position
const UserCard = () => (
  <div style={{ marginBottom: '20px', width: '300px' }}>
    {/* Content */}
  </div>
);
```

### After: The Layout Component Pattern
We create a generic `Split` component to handle the arrangement.

```tsx
// ✅ Better: Generic Layout Component
interface SplitProps {
  children: [React.ReactNode, React.ReactNode];
  leftWeight?: number;
  rightWeight?: number;
  gutter?: number;
}

export const Split = ({ 
  children, 
  leftWeight = 1, 
  rightWeight = 1, 
  gutter = 16 
}: SplitProps) => {
  const [Left, Right] = children;

  return (
    <div style={{ display: 'flex', gap: `${gutter}px` }}>
      <div style={{ flex: leftWeight }}>{Left}</div>
      <div style={{ flex: rightWeight }}>{Right}</div>
    </div>
  );
};

// Usage in a Page
const Dashboard = () => {
  return (
    <Split leftWeight={1} rightWeight={3}>
      <Sidebar />
      <MainContent />
    </Split>
  );
};
```

---

## Performance Considerations

### 1. Component Over-nesting
Layout components often introduce extra `div` layers. In extremely deep trees, this can impact DOM performance. Use `React.Fragment` or design your layout components to accept an `as` prop (e.g., `as="section"`) to keep the HTML semantic and flat.

### 2. Prop Stability
Since layout components often wrap the entire application or large sections, ensure that the props passed to them are stable. If you pass a new `style` object on every render to a Layout component, it might trigger unnecessary re-renders of the entire sub-tree.

---

## Real-World Use Cases

1.  **Dashboard Shells:** An `AppLayout` component that defines the Sidebar, TopNav, and ContentArea.
2.  **Modals/Drawers:** A `ModalLayout` that manages the header, sticky footer, and scrollable body.
3.  **Design Systems:** Systems like **Braid** or **Chakra UI** rely almost entirely on Layout components (e.g., `<Box>`, `<Stack>`, `<Inline>`) rather than writing raw CSS for positioning.

---

## Common Mistakes

- **Passing Data Props:** Sending a `user` object to a `SidebarLayout`. A layout should only receive `children` or `slots`. It shouldn't know what a "user" is.
- **Conditional Logic:** Putting complex `if/else` logic inside the layout. The layout should be "dumb." Logic should live in the Page or Feature component that *uses* the layout.
- **Over-abstraction:** Creating a layout component for a one-off structural need. Only abstract layouts that are reused or define a consistent rhythm in your UI.

---

## Summary & Key Takeaways

- **Purpose:** Layout components manage **space and position**, not business logic.
- **Separation:** Feature components define **content**; Layout components define **context**.
- **The "Gutter" Strategy:** Use a `gap` or `gutter` prop in layouts instead of `margin` inside components.
- **Interview Tip:** If asked about "Component Composition," bring up Layout Components. It’s the ultimate example of using `props.children` or "slots" to build a flexible, scalable UI.

## 1. The Split Screen Pattern

The **Split Screen Pattern** is a structural layout pattern used to divide a view into two or more distinct areas (usually horizontal sections). While it sounds simple, the architectural implementation determines how flexible and reusable your UI becomes. 

In React, there are two primary ways to implement this:
1.  **Component as Props (The "Slot" Pattern)**
2.  **Component as Children (The "Composition" Pattern)**

### The Problem it Solves
- **Responsive Symmetry:** Managing how two side-by-side containers behave when the viewport shrinks.
- **Prop Drilling for Layout:** Avoiding passing layout-specific styling (like `flex-basis`) down into business-logic components.
- **Composition vs. Configuration:** Providing a choice between a rigid, predictable layout (Props) and a flexible, dynamic one (Children).

---

## Mental Models & Analogies

### The "Stage & Spotlight" Metaphor
- **The Props Approach:** The director (Parent) assigns specific actors to specific "marked spots" on the stage (Left Slot, Right Slot). The actors cannot move from their assigned marks.
- **The Children Approach:** The director provides a "stage area" (the Container) and tells the actors to "get in line." The order they stand in defines their position.

---

## Design Patterns & Implementation

### 1. The Props Pattern (Explicit Slots)
In this pattern, the component explicitly defines props for each side. This is often preferred in **Design Systems** where you want to enforce that a layout *must* have exactly a left and a right side.

**Pros:**
- Highly readable: `<SplitScreen left={<Nav />} right={<Content />} />`
- Easy to enforce specific types for each slot.
- Can easily change the order of rendering in code without changing the usage.

**Cons:**
- Can feel verbose if passing many components.
- Harder to extend to "Three-way splits" without adding more props.

### 2. The Children Pattern (Implicit Composition)
This pattern uses the standard `children` prop. The layout component usually assumes the first child is the "Left" and the second is the "Right."

**Pros:**
- More "React-native" feel.
- Extremely flexible; easy to wrap children in extra providers or fragments.

**Cons:**
- Brittle: If a developer adds a third child by mistake, the layout might break.
- Less explicit: It’s not immediately clear which child is intended for which "slot" without looking at the implementation.

---

## Code Examples (TypeScript)

### Implementation A: Components as Props

```tsx
interface SplitScreenProps {
  left: React.ReactNode;
  right: React.ReactNode;
  leftWeight?: number;
  rightWeight?: number;
}

export const SplitScreenPropsBased = ({
  left,
  right,
  leftWeight = 1,
  rightWeight = 1,
}: SplitScreenProps) => {
  return (
    <div style={{ display: 'flex', width: '100%' }}>
      <div style={{ flex: leftWeight }}>{left}</div>
      <div style={{ flex: rightWeight }}>{right}</div>
    </div>
  );
};

// Usage
// <SplitScreenPropsBased left={<Sidebar />} right={<MainContent />} />
```

### Implementation B: Components as Children

```tsx
interface SplitChildrenProps {
  children: [React.ReactNode, React.ReactNode]; // Enforce exactly two children
  leftWeight?: number;
  rightWeight?: number;
}

export const SplitScreenChildrenBased = ({
  children,
  leftWeight = 1,
  rightWeight = 1,
}: SplitChildrenProps) => {
  const [Left, Right] = children; // Destructure the children array

  return (
    <div style={{ display: 'flex', width: '100%' }}>
      <div style={{ flex: leftWeight }}>{Left}</div>
      <div style={{ flex: rightWeight }}>{Right}</div>
    </div>
  );
};

// Usage
// <SplitScreenChildrenBased>
//    <Sidebar />
//    <MainContent />
// </SplitScreenChildrenBased>
```

---

## Performance Considerations

### 1. Array Destructuring in Children
In the "Children" approach, `children` is an opaque data structure. If you only pass one child, `const [Left, Right] = children` will throw an error or result in `undefined`. 
**Optimization:** Use `React.Children.toArray(children)` to safely handle cases where children might be conditionally rendered (e.g., `{showSide && <Side />}`).

### 2. Re-renders
Neither pattern inherently causes more re-renders than the other. However, passing components as **Props** allows you to memoize the slot content more easily if the `SplitScreen` parent re-renders frequently.

---

## Real-World Use Cases

1.  **Authentication Pages:** A split screen with a marketing image on the left and the login form on the right.
2.  **Comparison Tools:** Two products shown side-by-side for feature comparison.
3.  **File Explorers:** A navigation tree on the left and the file content on the right.

---

## Common Mistakes

- **Hardcoding Weights:** Avoid hardcoding `flex: 1` inside the component. Always provide weights as props to make the component reusable for 30/70, 50/50, or 20/80 splits.
- **Ignoring Mobile:** A "Split Screen" on a mobile device usually needs to become a "Stack." Ensure your layout component handles media queries or a `stackOnMobile` boolean prop.
- **Implicit Ordering:** In the Children pattern, developers often forget that the order in the DOM matters for accessibility (screen readers). Ensure that the visual "Left" is also the first item in the DOM.

---

## Summary & Key Takeaways

- **The Props Pattern** is best for **Strict Layouts** (e.g., Header/Footer/Body) where you want to explicitly name the areas.
- **The Children Pattern** is best for **Generic Containers** where you want to stay close to standard HTML/React composition.
- **Interview Tip:** If asked which is better, the senior answer is: *"It depends on the API surface you want to expose. Slots (Props) provide better documentation and constraint, while Composition (Children) provides better flexibility for the consumer."*

## 2. The Generic List Pattern

In React, rendering lists is a fundamental task, but at scale, simply calling `.map()` inside a component leads to tight coupling and logic duplication. The **Generic List Pattern** decouples the **Iteration Logic** (the parent) from the **Representation Logic** (the list item).

### The Problem it Solves
- **Tight Coupling:** In a standard map, the parent component must know the exact structure of the child component's props.
- **Boilerplate:** Every time you need a list, you have to write the same map logic, key handling, and empty-state checks.
- **Design Inconsistency:** Ensuring that all lists (users, products, logs) follow the same spacing and separator rules is difficult when they are hardcoded.

---

## Mental Models & Analogies

### The "Stenciling" Metaphor
Think of the **Parent Component** as a **Stencil**. The stencil defines where the shapes go and how they are spaced out on the paper. The **ListItemComponent** is the **Paint**. You can change the color or type of paint (the data and its look), but the stencil ensures the shapes remain perfectly aligned and consistent.

---

## Design Patterns & Best Practices

### 1. The "Component-as-Prop" Pattern
Instead of hardcoding a child component, the Parent accepts a component definition as a prop. This allows the parent to be "agnostic" about what it is actually rendering.

### 2. The Resource Name Mapping
To make the Parent truly generic, we pass a `resourceName` prop. This tells the Parent which prop name the child component expects for its data (e.g., `user`, `product`, or `item`).

### 3. Separation of Concerns
- **Parent (`RegularList`):** Handles the "How" (Looping, keys, layout, separators).
- **Child (`ListItem`):** Handles the "What" (Displaying the specific data object).

---

## Code Examples

### Before: The Hardcoded Approach (Low Reusability)
```tsx
// ❌ Problem: This component only works for Users. 
// If we want a Product list, we have to rewrite this entire structure.
const UserList = ({ users }) => (
  <ul>
    {users.map(user => (
      <UserListItem key={user.id} user={user} />
    ))}
  </ul>
);
```

### After: The Generic List Pattern (Staff Level)
Using TypeScript generics ensures that the data passed to the list matches the props expected by the item component.

```tsx
interface ListProps<T> {
  items: T[];
  resourceName: string;
  itemComponent: React.ComponentType<any>;
}

export const RegularList = <T,>({
  items,
  resourceName,
  itemComponent: ItemComponent,
}: ListProps<T>) => {
  return (
    <>
      {items.map((item, i) => (
        // We dynamically spread the object using the resourceName as the key
        <ItemComponent key={i} {...{ [resourceName]: item }} />
      ))}
    </>
  );
};

// --- Usage ---

const SmallUserListItem = ({ user }: { user: any }) => {
  return <p>{user.name} - {user.age}</p>;
};

// Now RegularList can be used for ANY data type and ANY component
<RegularList 
  items={users} 
  resourceName="user" 
  itemComponent={SmallUserListItem} 
/>
```

---

## Performance Considerations

### 1. The "Key" Prop
In generic lists, we often default to the index `i` for keys. **This is a performance anti-pattern** if the list is dynamic (filtered, sorted, or items added/removed). 
*   **Optimization:** Pass a `keyExtractor` function prop to the `RegularList` to allow the parent to retrieve a unique ID (e.g., `item => item.id`).

### 2. Reconciliation & Memoization
If the `RegularList` re-renders, every `ItemComponent` will re-render by default. 
*   **Optimization:** Wrap your `ListItemComponent` in `React.memo()`. Since the generic list is "dumb" and just passes props through, memoization becomes highly effective.

### 3. List Virtualization
For lists exceeding 100+ items, even a generic list will lag. At this stage, the `RegularList` should be swapped for a **Virtualized List** (using libraries like `react-window`), which only renders items currently in the viewport.

---

## Real-World Use Cases

1.  **Search Results:** A generic search page that renders `PersonResult`, `DocumentResult`, or `ImageResult` depending on the filter, all using the same `RegularList` skeleton.
2.  **Design Systems:** A `Dropdown` or `Select` component that uses a `List` parent to handle keyboard navigation (up/down arrows) while allowing the consumer to pass custom `Option` components.
3.  **Feeds:** Social media feeds where "Post," "Ad," and "Suggested Follow" components are all fed into a single list engine.

---

## Common Mistakes

- **Component Creation inside Render:** Never define the `itemComponent` inside the parent's render function (e.g., `itemComponent={() => <div />}`). This creates a new functional reference on every render, causing the entire list to unmount and remount, destroying performance and state.
- **Over-Generalization:** Don't try to make one list component handle horizontal, vertical, grid, and masonry layouts. Create `GridList`, `StackList`, and `RowList` as separate layout parents.
- **Missing Empty States:** A professional generic list should always handle the `items.length === 0` case internally or via an `EmptyStateComponent` prop.

---

## Summary & Key Takeaways

- **Generic Lists** treat the child component as a "plugin."
- **`resourceName`** is the bridge that maps generic data to specific component props.
- **Key Extraction:** Always prioritize stable IDs over array indices.
- **Interview Tip:** When discussing lists, emphasize **"Separation of Concerns."** Explain that the list engine shouldn't care about the business logic of the items it displays; it only cares about the iteration and layout.

## 3. The Modal Pattern

A **Modal** is an imperatively or declaratively triggered overlay that interrupts the user's workflow to focus their attention on a specific task or piece of information. Architecturally, a robust React Modal consists of two primary visual layers:
1.  **ModalBackground (Overlay):** The semi-transparent layer that dims the rest of the application, signaling that the UI is temporarily inactive.
2.  **ModalContent (Container):** The actual window containing the specific UI/Logic.

### The Problem it Solves
- **Focus Management:** It forces the user to interact with a specific flow before returning to the main application.
- **Context Preservation:** Allows users to perform a secondary task (e.g., adding a new category while filling out a product form) without navigating away from the current page.
- **Visual Hierarchy:** Solves "z-index wars" by using Portals to render above the main DOM tree.

---

## Mental Models & Analogies

### The "Theater Spotlight" Metaphor
Imagine a theater stage. 
- **The Stage:** Your main application content.
- **The Dimmed Lights (ModalBackground):** Everything else goes dark to eliminate distractions.
- **The Spotlight (ModalContent):** A bright, focused beam shines on a single actor (the Modal content). The audience cannot look elsewhere until the scene ends.

---

## Design Patterns & Best Practices

### 1. The Composition Pattern (Children)
A professional Modal component should be a "dumb" container. It handles the *behavior* (closing on ESC, blocking scroll), while the *content* is passed as `children`. This prevents the Modal component from becoming a "God Object" that knows about every form in your app.

### 2. React Portals
Modals should almost always be rendered via `ReactDOM.createPortal`. 
- **Why?** If a parent component has `overflow: hidden` or a specific `z-index`, the Modal might be clipped or hidden. Portals inject the Modal into a `<div>` at the end of the `<body>`, bypassing parent CSS constraints.

### 3. Body Scroll Locking
When a modal is open, the background content should not be scrollable. This is a hallmark of a high-quality user experience.

---

## Code Examples

### The "Staff-Level" Modal Implementation
This implementation uses Portals and the Composition pattern.

```tsx
import React, { ReactNode, useEffect } from 'react';
import { createPortal } from 'react-dom';
import styled from 'styled-components';

// 1. Modal Background (Overlay)
const ModalBackground = styled.div`
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
`;

// 2. Modal Content (The Box)
const ModalContent = styled.div`
  background-color: white;
  padding: 20px;
  width: 50%;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
`;

interface ModalProps {
  shouldShow: boolean;
  onRequestClose: () => void;
  children: ReactNode;
}

export const Modal = ({ shouldShow, onRequestClose, children }: ModalProps) => {
  // Prevent body scroll when modal is active
  useEffect(() => {
    if (shouldShow) {
      document.body.style.overflow = 'hidden';
    } else {
      document.body.style.overflow = 'auto';
    }
    return () => { document.body.style.overflow = 'auto'; };
  }, [shouldShow]);

  if (!shouldShow) return null;

  // Use Portal to render at the top level of the DOM
  return createPortal(
    <ModalBackground onClick={onRequestClose}>
      {/* stopPropagation prevents clicking the content from closing the modal */}
      <ModalContent onClick={(e) => e.stopPropagation()}>
        <button onClick={onRequestClose}>Close</button>
        {children}
      </ModalContent>
    </ModalBackground>,
    document.body
  );
};
```

### Usage
```tsx
const App = () => {
  const [showModal, setShowModal] = useState(false);

  return (
    <>
      <button onClick={() => setShowModal(true)}>Open Form</button>
      
      <Modal shouldShow={showModal} onRequestClose={() => setShowModal(false)}>
        <h2>Subscribe to Newsletter</h2>
        <input type="email" placeholder="Enter email" />
        <button>Submit</button>
      </Modal>
    </>
  );
};
```

---

## Performance Considerations

### 1. Conditional Rendering vs. Hidden Display
- **Unmounting (Recommended):** `if (!shouldShow) return null`. This removes the Modal and its children from the DOM, freeing up memory and preventing unnecessary background logic/renders.
- **Hiding (`display: none`):** Useful only if the modal content is extremely heavy and needs to keep its internal state (like an unsaved video edit).

### 2. Event Listener Cleanup
If you add a listener for the `Escape` key inside the Modal, ensure it is cleaned up in the `useEffect` return function to avoid memory leaks.

---

## Real-World Use Cases

1.  **Critical Confirmations:** "Are you sure you want to delete this project?"
2.  **Complex Forms:** Multi-step sign-up flows that shouldn't disrupt the background context.
3.  **Image Lightboxes:** Viewing a high-resolution version of a product image.

---

## Common Mistakes

- **Not using `e.stopPropagation()`:** If the `ModalBackground` has an `onClick={close}` handler, clicking anywhere inside the `ModalContent` will also trigger the close unless you stop the event bubble.
- **Forgetting Accessibility (A11y):** A professional modal needs `role="dialog"` and `aria-modal="true"`. It should also "trap" the focus (the user shouldn't be able to Tab out of the modal into the background).
- **Hardcoding State:** Creating the `isOpen` state *inside* the Modal component. This makes it impossible for the parent to control the modal or use it as a "Controlled Component."

---

## Summary & Key Takeaways

- **Composition:** Always pass modal content as `children`.
- **Portals:** Use `createPortal` to solve Z-index and layout clipping issues.
- **The "Three Layers":** Trigger (Parent State) -> Background (Overlay) -> Content (Container).
- **Interview Tip:** When asked about Modals, mention **"Focus Trapping"** and **"Portals."** These are the two features that distinguish a junior implementation from a staff-level implementation. Focus trapping ensures keyboard users don't get lost in the background while the modal is open.