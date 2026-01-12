# Controlled vs. Uncontrolled Components

## Concept Explanation

In React, the distinction between **Controlled** and **Uncontrolled** components revolves around the **Source of Truth** for the component's state (typically form data or UI state like "is open").

*   **Uncontrolled Components:** The DOM handles the state internally. You use a `ref` to "pull" the value from the DOM when you need it (e.g., on submit). It is closer to traditional HTML.
*   **Controlled Components:** React state (`useState` or `useReducer`) handles the state. React "pushes" the value into the component via props and updates it via callbacks.

### The Problem it Solves
- **Synchronization:** Controlled components make it easy to sync the value of an input with other UI elements (e.g., a live preview).
- **Validation:** Controlled components allow for real-time validation and input masking (e.g., preventing a user from typing numbers).
- **Predictability:** Uncontrolled components can be simpler to write for basic forms but harder to debug in complex, state-driven UIs.

---

## Mental Models & Analogies

### The "Remote Control" vs. "Manual Knob"
- **Controlled (Remote Control):** You have a remote. When you press "Volume Up," the remote sends a signal to the TV (React State), the TV updates its internal logic, and then the screen displays the new volume. The remote is the only way to change the state.
- **Uncontrolled (Manual Knob):** You walk up to the TV and turn a physical knob. The TV changes volume immediately. If you want to know what the volume is, you have to go look at where the knob is pointing (The DOM).

---

## Which is Preferred?

There is no "better" choice, only "right for the use case," though **Controlled components are the standard for most React applications.**

| Feature | Controlled | Uncontrolled |
| :--- | :--- | :--- |
| **Source of Truth** | React State | DOM |
| **Complexity** | Higher (Requires more boilerplate) | Lower (Requires `refs`) |
| **Performance** | Can cause re-renders on every keystroke | Highly performant (No re-renders) |
| **Validation** | Easy (Real-time) | Difficult (Usually on Submit) |
| **Best For** | Complex forms, dynamic UI, Design Systems | Simple forms, non-React integration |

---

## Design Patterns & Code Examples

### 1. Uncontrolled Component (The "Ref" Pattern)
The component keeps track of its own state. We only ask for it when we need it.

```tsx
import { useRef } from 'react';

export const UncontrolledInput = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    // We "pull" the value from the DOM via the ref
    alert(`Input Value: ${inputRef.current?.value}`);
  };

  return (
    <>
      <input type="text" ref={inputRef} defaultValue="Initial Value" />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
};
```

### 2. Controlled Component (The "State" Pattern)
React is in full control. The input cannot change unless the state updates.

```tsx
import { useState } from 'react';

export const ControlledInput = () => {
  const [value, setValue] = useState('');

  return (
    <input 
      type="text" 
      value={value} 
      onChange={(e) => setValue(e.target.value)} 
      placeholder="Type something..."
    />
  );
};
```

### 3. Advanced: The Hybrid Modal (Controlled & Uncontrolled)
In Design Systems, you often want a component that can manage its own state (Uncontrolled) but can also be overridden by the parent (Controlled).

```tsx
interface ModalProps {
  isOpen?: boolean; // If provided, the component is "Controlled"
  onClose?: () => void;
}

export const Modal = ({ isOpen: controlledIsOpen, onClose }: ModalProps) => {
  // Internal state for "Uncontrolled" usage
  const [internalIsOpen, setInternalIsOpen] = useState(false);

  // Determine source of truth
  const isControlled = controlledIsOpen !== undefined;
  const isOpen = isControlled ? controlledIsOpen : internalIsOpen;

  const handleClose = () => {
    if (isControlled) {
      onClose?.();
    } else {
      setInternalIsOpen(false);
    }
  };

  if (!isOpen) return null;

  return (
    <div className="modal">
      <button onClick={handleClose}>Close</button>
      <p>Modal Content</p>
    </div>
  );
};
```

---

## Performance Considerations

### The "Keystroke Lag"
In Controlled components, every keystroke triggers a re-render of the component and potentially its entire sub-tree. 
- **Optimization:** If a form is massive (50+ inputs), use **Uncontrolled components** or libraries like `react-hook-form` which use refs internally to avoid global re-renders.
- **Optimization:** Debounce the `onChange` handler if you are performing expensive operations (like API calls) based on the input.

---

## Real-World Use Cases

1.  **Search Bars with Suggestions:** Must be **Controlled** to filter the list in real-time as the user types.
2.  **Multi-step Wizards:** Usually **Controlled** to ensure data is preserved and validated before moving to the next step.
3.  **Legacy Integration:** Use **Uncontrolled** components when wrapping a jQuery plugin or a non-React library that manages its own DOM state.
4.  **Large Data-Entry Forms:** Use **Uncontrolled** (via `react-hook-form`) to maintain 60fps performance during typing.

---

## Common Mistakes

- **Mixing `value` and `defaultValue`:** 
  - `value` makes an input Controlled. 
  - `defaultValue` is for Uncontrolled inputs (initial value only).
  - *Error:* Providing both or switching between them (e.g., starting with `undefined` and then setting a string) will trigger a React warning: *"A component is changing an uncontrolled input to be controlled."*
- **Forgetting `onChange`:** If you provide a `value` but no `onChange`, the input becomes **read-only** and the user will be unable to type.

---

## Summary & Key Takeaways

- **Controlled:** React state is the boss. Best for validation and complex logic.
- **Uncontrolled:** DOM is the boss. Best for performance and simple "one-off" forms.
- **Source of Truth:** Always identify who "owns" the data before writing the component.
- **Interview Tip:** If asked which to use, say: *"I prefer Controlled components for the predictability and ease of testing, but I opt for Uncontrolled components when building high-performance forms with many fields to avoid unnecessary re-renders."*

## Form Design Patterns (Controlled vs. Uncontrolled)

In React, form management is the most common arena where the "Controlled vs. Uncontrolled" debate takes place. It determines how form data is collected, validated, and submitted.

*   **Uncontrolled Forms:** Data is handled by the DOM itself. We "pull" the values out of the form elements using `refs` or the native `FormData` API, typically only when the user clicks "Submit."
*   **Controlled Forms:** Data is handled by React state. Every keystroke updates the state, and the state drives the value of the inputs.

### The Problem it Solves
- **Complexity Management:** How do we handle 2 inputs vs. 20 inputs?
- **Real-time Feedback:** How do we show "Password too short" *while* the user types?
- **Data Integrity:** Ensuring the data sent to the API matches exactly what is shown in the UI.

---

## Mental Models & Analogies

### The "Snapshot" vs. "Live Stream"
- **Uncontrolled Form (Snapshot):** Like taking a photo of a room. You don't know what happened in the room all day; you only see the state of things the exact moment you press the shutter button (Submit).
- **Controlled Form (Live Stream):** Like a security camera feed. You see every movement (keystroke), every change, and every interaction in real-time as it happens.

---

## 1. Uncontrolled Forms

In an uncontrolled form, you don't write an event handler for every typing action. Instead, you use a `ref` to access the form or use the native `onSubmit` event to grab all data at once.

### Code Example: The Modern Native Way
Instead of creating 10 different `useRef` hooks, we use the `FormData` API. This is the cleanest "Staff-level" way to handle uncontrolled forms.

```tsx
export const UncontrolledForm = () => {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    // Use the native browser API to collect all data
    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData.entries());
    
    console.log('Form Submitted:', data);
    // Result: { name: 'John', age: '25' }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" type="text" placeholder="Name" />
      <input name="age" type="number" placeholder="Age" />
      <button type="submit">Submit</button>
    </form>
  );
};
```

### When to use:
- Simple forms with no real-time validation.
- Large forms where performance is critical (avoiding re-renders on every character).
- "Quick and dirty" internal tools.

---

## 2. Controlled Forms

In a controlled form, React state is the **Single Source of Truth**. The input values are "bound" to the state.

### Code Example: The Object State Pattern
Instead of a separate `useState` for every input, we use a single object state to keep the code dry.

```tsx
import { useState } from 'react';

export const ControlledForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    age: '',
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value // Dynamic key update
    }));
  };

  const isInvalid = formData.name.length < 3;

  return (
    <form>
      <input 
        name="name" 
        value={formData.name} 
        onChange={handleChange} 
      />
      {isInvalid && <span style={{ color: 'red' }}>Name too short</span>}
      
      <input 
        name="age" 
        value={formData.age} 
        onChange={handleChange} 
      />
      
      <button disabled={isInvalid} type="submit">Submit</button>
    </form>
  );
};
```

### When to use:
- Real-time validation (e.g., strength meters, availability checks).
- Conditionally disabling the submit button.
- Dynamic inputs (e.g., a field that appears only if the previous field is filled).
- Enforcing specific input formats (e.g., credit card spacing).

---

## Performance Considerations

### The "Re-render" Trap
In a **Controlled Form**, every time a user types:
1. State updates.
2. The entire form component re-renders.
3. If the form is huge or has heavy child components, the UI will feel "laggy."

**Staff Optimization:** If performance becomes an issue but you still need validation, use a library like **React Hook Form**. It uses `uncontrolled` inputs internally for performance but provides a `controlled` API for validation.

---

## Real-World Use Cases

1.  **Search Inputs:** Almost always **Controlled** so the results list can filter immediately.
2.  **Checkout Flows:** Usually **Controlled** because shipping options depend on the address entered.
3.  **Login Forms:** Can be **Uncontrolled**. You usually don't need to validate a password while the user is typing it; you just need it when they click "Login."

---

## Common Mistakes

- **Initializing with `undefined`:** If you initialize state as `useState()` (empty), and later set a string, React will complain that you are switching an uncontrolled component to controlled. **Always initialize with an empty string `''` or a default object.**
- **Manual DOM Manipulation:** Trying to change an input value using `document.getElementById('my-input').value = '...'` in a Controlled form. React will immediately overwrite it back to what is in the state.
- **Missing `name` attribute:** In the Object State pattern, if you forget the `name="email"` attribute on your input, the `handleChange` function won't know which property to update.

---

## Summary & Key Takeaways

- **Uncontrolled Forms** are easier to write and more performant but offer less control. Use `FormData` to extract values.
- **Controlled Forms** provide maximum control and real-time feedback but require more boilerplate and can cause performance issues if not optimized.
- **Standard Practice:** Start with **Uncontrolled** for simple needs; move to **Controlled** as soon as you need UI reactivity (like disabling buttons or showing errors).
- **Interview Tip:** "I prefer **Controlled Forms** for complex validation logic because it keeps the UI and State in perfect sync, but for high-performance data-entry forms, I leverage the **Uncontrolled** approach (often via `react-hook-form`) to minimize the render count."

## Controlled Modals

A **Controlled Modal** is a design pattern where the visibility of the modal is managed entirely by its parent component via props. Unlike an "Uncontrolled Modal," which might maintain its own `isOpen` state internally, a Controlled Modal follows the React philosophy of **Lifting State Up**.

### What the concept is
The parent passes a boolean prop (e.g., `shouldShow` or `isOpen`) and a callback function (e.g., `onRequestClose`) to the modal. The modal "requests" to be closed (when the user clicks the overlay or the "X" button), but the parent ultimately decides whether to update the state and hide it.

### Why it exists
In production apps, the decision to close a modal often depends on external factors. For example:
- You may want to prevent a modal from closing if a form inside is "dirty" (unsaved changes).
- You may need to wait for an API call to finish before the modal disappears.
- You might need to open the modal from a completely different part of the UI (like a header button) while the modal lives inside a layout component.

---

## Mental Models & Analogies

### The "Gatekeeper" Metaphor
Think of an Uncontrolled Modal like a **Bathroom Door**: You go in, you lock it (state changes), and you unlock it to leave. No one outside knows or cares about the lock.

Think of a Controlled Modal like a **High-Security Gate**: You walk up to the gate (User action), but you don't open it yourself. You talk to the Gatekeeper (The Parent Component) and ask, "Can I come in?" or "Can I leave?" The Gatekeeper checks the rules (Logic/State) and pushes the button to move the gate.

---

## Design Patterns & Best Practices

### 1. The "Request Close" Pattern
Avoid naming callbacks `onClose`. Use `onRequestClose` to signal that the modal is *asking* to close, but it isn't closed yet.

### 2. State-Driven Content
Since the parent controls the state, you can easily switch the modal's content based on which button was clicked or which item in a list was selected.

### 3. Separation of Visibility and Content
Keep the logic for *showing* the modal separate from the *logic inside* the modal. This keeps your parent components clean.

---

## Code Examples

### Implementation: The Controlled Modal Component

```tsx
import { createPortal } from 'react-dom';
import styled from 'styled-components';

const Overlay = styled.div`
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex; justify-content: center; align-items: center;
`;

const Content = styled.div`
  background: white; padding: 2rem; border-radius: 8px;
`;

interface ControlledModalProps {
  shouldShow: boolean;
  onRequestClose: () => void;
  children: React.ReactNode;
}

export const ControlledModal = ({ 
  shouldShow, 
  onRequestClose, 
  children 
}: ControlledModalProps) => {
  if (!shouldShow) return null;

  return createPortal(
    <Overlay onClick={onRequestClose}>
      <Content onClick={(e) => e.stopPropagation()}>
        <button onClick={onRequestClose}>X</button>
        {children}
      </Content>
    </Overlay>,
    document.body
  );
};
```

### Implementation: The Parent (The Controller)

```tsx
export const App = () => {
  const [shouldShowModal, setShouldShowModal] = useState(false);

  const handleClose = () => {
    // We can add logic here: "Are you sure you want to exit?"
    const confirm = window.confirm("Discard changes?");
    if (confirm) setShouldShowModal(false);
  };

  return (
    <>
      <button onClick={() => setShouldShowModal(true)}>Open Modal</button>
      
      <ControlledModal 
        shouldShow={shouldShowModal} 
        onRequestClose={handleClose}
      >
        <h1>Controlled Content</h1>
        <p>This modal only closes if the parent allows it.</p>
      </ControlledModal>
    </>
  );
};
```

---

## Performance Considerations

### 1. Unmounting vs. CSS Hiding
Always prefer `if (!shouldShow) return null`. This ensures that any logic inside the modal (like data fetching or timers) stops when the modal is closed. It also keeps the DOM tree shallow.

### 2. Backdrop Re-renders
If the parent component re-renders frequently, use `React.memo` on the Modal component. Since the modal is a "Portal" and often covers the whole screen, visual lag during its opening/closing animation is very noticeable to users.

---

## Real-World Use Cases

1.  **Form Dirty Checks:** Preventing the modal from closing if the user has typed into a form and hasn't saved.
2.  **Multi-Step Modals:** The parent tracks `stepIndex`. The modal displays Step 1, Step 2, or Step 3. The "Close" button might actually trigger a "Go Back" action instead of closing, depending on parent state.
3.  **Route-Based Modals:** Using the URL (e.g., `/photos/123`) to control the `shouldShow` state. If the user hits the browser "Back" button, the parent detects the URL change and closes the modal automatically.

---

## Common Mistakes

- **Forgetting `e.stopPropagation()`:** Clicking inside the modal triggers the background click handler and closes the modal unexpectedly.
- **Syncing State via `useEffect`:** Trying to keep a local `isOpen` state in the modal that syncs with the prop. This creates a "Double Source of Truth" and leads to bugs. Use the prop directly.
- **Prop Drilling:** Passing

## Multi-Step Flows (Wizards)

A **Flow** (often called a **Wizard**) is a design pattern used to break down a complex task into a series of sequential steps. Instead of overwhelming a user with one massive form, you "flow" them through logical chunks of information gathering.

In React, the architectural challenge is managing two things:
1.  **Navigation State:** Which step is currently active?
2.  **Data State:** How is the data collected across steps and where does it live?

### The Problem it Solves
- **Cognitive Load:** Reduces user anxiety by showing one screen at a time.
- **Data Validation:** Allows for validating small chunks of data before the user moves forward.
- **Complex Branching:** Enables "Conditional Paths" (e.g., if the user selects "Business" in Step 1, show Step 2b instead of 2a).

---

## 1. Uncontrolled Flows

An **Uncontrolled Flow** is a component that manages its own "current step" and "accumulated data" internally. The parent component simply renders the flow and provides a final callback (e.g., `onFinish`).

### Why use it?
- It is a "black box" solution. You can drop it anywhere in the app without worrying about state management.
- Ideal for simple, linear onboarding or setup wizards that don't need to be synced with a URL or an external database mid-process.

### Mental Model: The "Escalator"
Once you step on an escalator (the flow), it handles the movement. You don't control the speed or the steps from the outside; you just wait until you arrive at the top (the final result).

---

## 2. Controlled Flows

A **Controlled Flow** lifts the navigation and data state to the parent component. The parent tells the flow which step to display and stores the data as it comes in.

### Why use it?
- **URL Syncing:** You want the user to be able to hit "Refresh" or "Back" in the browser and stay on the same step (e.g., `/onboarding/step-2`).
- **Persistence:** You want to save the data to the API after *every* step so the user can finish later.
- **External Triggers:** You need a button in the Header or Sidebar to trigger a step jump.

### Mental Model: The "Remote Control"
The parent holds the remote. It decides when to change the "channel" (step) and records what happened on each channel.

---

## 3. The "Collecting Data" Pattern

In both patterns, the core technical hurdle is: **How does the Flow component gather data from children it knows nothing about?**

We use the **Prop Injection Pattern** (via `React.cloneElement`). The Flow component passes a "next" function to its children. When the child calls `onNext(data)`, the Flow component saves that data and increments the step.

---

## Code Examples

### The Uncontrolled Flow Implementation

```tsx
import React, { useState, ReactElement } from 'react';

export const UncontrolledFlow = ({ children, onFinish }: { children: React.ReactNode[], onFinish: (data: any) => void }) => {
  const [data, setData] = useState({});
  const [currentStepIndex, setCurrentStepIndex] = useState(0);

  const onNext = (stepData: any) => {
    const nextIndex = currentStepIndex + 1;
    const updatedData = { ...data, ...stepData };

    if (nextIndex < children.length) {
      setCurrentStepIndex(nextIndex);
    } else {
      onFinish(updatedData);
    }

    setData(updatedData);
  };

  const currentChild = React.Children.toArray(children)[currentStepIndex];

  if (React.isValidElement(currentChild)) {
    // Inject the 'onNext' handler into the current step component
    return React.cloneElement(currentChild as ReactElement, { onNext });
  }

  return currentChild;
};
```

### Usage (The Steps)

```tsx
const StepOne = ({ onNext }: any) => (
  <button onClick={() => onNext({ name: 'John Doe' })}>Next (Name)</button>
);

const StepTwo = ({ onNext }: any) => (
  <button onClick={() => onNext({ age: 30 })}>Finish (Age)</button>
);

// App Usage
<UncontrolledFlow onFinish={data => console.log('Final Data:', data)}>
  <StepOne />
  <StepTwo />
</UncontrolledFlow>
```

---

## Performance Considerations

### 1. Re-mounting Children
Because the Flow component usually renders only one child at a time (`children[currentStepIndex]`), every step change causes the previous child to **unmount** and the new one to **mount**. 
- **Staff Tip:** Ensure that "Step" components do not have heavy `useEffect` side effects that run on every mount unless necessary.

### 2. State Accumulation
As the `data` object grows, every update triggers a re-render of the Flow parent. For very large flows (20+ steps), consider using a `useReducer` or a specialized state library to manage the "accumulator" more efficiently.

---

## Real-World Use Cases

1.  **SaaS Onboarding:** Collecting user name, company size, and inviting team members.
2.  **Checkout Pipelines:** Shipping address -> Payment Info -> Review -> Confirmation.
3.  **Complex Surveys:** Branching logic where Step 2 changes based on the answer in Step 1. (Controlled Flows are better here).

---

## Common Mistakes

- **Deep Prop Drilling:** Trying to pass the `data` object through every step. Steps should only know about their *own* data. The Flow component should be the one to merge them.
- **Hardcoding Step Numbers:** Writing `if (step === 1) return <StepOne />`. 
  - *Fix:* Use `React.Children` to keep the Flow component generic and reusable for any sequence of components.
- **Ignoring "Back" Logic:** Developers often forget the `onBack` handler. A professional flow should allow users to return to previous steps to correct mistakes without losing the data they already typed.

---

## Summary & Key Takeaways

- **Flows** are higher-order components that manage a sequence of UI states.
- **Uncontrolled Flows** are self-contained; **Controlled Flows** are driven by the parent.
- **Data Collection** is best handled by injecting a "completion" callback (`onNext`) into children using `React.cloneElement`.
- **Interview Tip:** If asked about Wizards/Flows, highlight the **Data Accumulation Strategy**. Explain that the container should act as the "Single Source of Truth" for the cumulative data, while the children remain "Dumb Components" that only worry about their specific slice of the form.