# Naming Conventions in React

## Concept Explanation

In a large-scale React ecosystem, naming conventions are more than just aesthetic choices; they are a form of **architectural documentation**. Consistent naming reduces cognitive load, makes the codebase "grep-able" (searchable), and ensures that a team of 50+ engineers can navigate a repository as if it were written by a single person.

### The Problem it Solves
- **Searchability:** Being able to find `UserAvatar.tsx` instantly without guessing if it's named `user-avatar`, `avatar`, or `User_avatar`.
- **Conflict Resolution:** Preventing naming collisions between local variables, hooks, and components.
- **Onboarding:** Reducing the time it takes for new engineers to understand the role of a file based solely on its name and location.

---

## Mental Models & Analogies

### The "Library Catalog" Metaphor
Think of your codebase as a massive library. 
- **PascalCase** is the **Title of a Book** (A discrete, standalone entity/Component).
- **camelCase** is the **Content inside the book** (Variables, logic, functions).
- **kebab-case** is the **Shelf Label** (Folders or non-component assets).

---

## Design Patterns & Best Practices

### 1. Component Naming (PascalCase)
Components represent entities. They should always use `PascalCase`.

*   **Standard:** `ComponentName.tsx`
*   **Pattern:** `[Domain][Entity][Extension]`
*   **Example:** `AuthLoginForm.tsx`, `ProductCard.tsx`

### 2. File Naming vs. Component Naming
While some environments prefer `kebab-case` for all files, the React industry standard leans toward **matching the file name to the primary export.**

| File Type | Convention | Example |
| :--- | :--- | :--- |
| **Component** | `PascalCase` | `SubmitButton.tsx` |
| **Hook** | `camelCase` (prefixed with `use`) | `useLocalStorage.ts` |
| **Utility/Helper** | `camelCase` or `kebab-case` | `formatCurrency.ts` |
| **Styles** | Match the component | `SubmitButton.module.scss` |
| **Tests** | Match the component + `.test` | `SubmitButton.test.tsx` |

### 3. Folder Structuring: The "Index" Pattern
In design systems, we often use folders to group a component with its styles and tests.

```text
src/components/
└── Button/
    ├── Button.tsx          // The component
    ├── Button.styles.ts    // Styled-components or CSS Modules
    ├── Button.test.tsx     // Unit tests
    ├── Button.types.ts     // TypeScript interfaces
    └── index.ts            // The "Public API" (re-exports Button)
```

**Why?** This allows you to import from `components/Button` instead of `components/Button/Button`.

---

## Code Examples

### Boolean Prop Naming
Avoid ambiguous names. Use prefixes like `is`, `has`, or `should`.

```tsx
// ❌ Bad: Ambiguous
<User visible={true} logic={false} />

// ✅ Good: Clear intent
<User preservesState={true} isVisible={false} hasPermission={true} />
```

### Event Handler Naming
Follow the `on[Event]` vs `handle[Event]` pattern.
- **Props:** Use the `on` prefix (e.g., `onClick`).
- **Internal Functions:** Use the `handle` prefix (e.g., `handleClick`).

```tsx
// Implementation
const UserProfile = ({ onUpdate }) => {
  const handleUpdateClick = () => {
    // Logic here
    onUpdate();
  };

  return <button onClick={handleUpdateClick}>Update</button>;
};
```

---

## Performance & Tooling Considerations

### 1. Case Sensitivity Pitfalls
Windows and macOS (by default) are case-insensitive, but Linux (CI/CD environments) is case-sensitive. 
- **The Bug:** You rename `Userprofile.tsx` to `UserProfile.tsx`. Your local machine works. Your GitHub Action/Build fails because it still looks for the old casing.
- **Fix:** Always use a consistent convention from day one to avoid "Ghost Files" in Git.

### 2. Tree Shaking & Barrel Files (`index.ts`)
While `index.ts` files make imports cleaner, be careful in massive libraries. If not configured correctly (e.g., missing `"sideEffects": false` in `package.json`), importing one component from a barrel file might pull in the entire library into the bundle.

---

## Real-World Use Cases

### Design Systems (e.g., Shopify Polaris, Ant Design)
In professional design systems, naming follows a strict **Modifier Pattern**:
- `Button` (Base)
- `ButtonPrimary` (Variant)
- `ButtonLoading` (State)

### Large Scale Feature Folders
In "Bulletproof React" or "Feature-driven" architectures:
- `features/auth/api/login.ts`
- `features/auth/components/LoginForm.tsx`
- `features/auth/hooks/useAuth.ts`

---

## Common Mistakes & Anti-patterns

1.  **Generic Names:** Naming a file `Styles.ts` or `Helpers.ts`. In a large project, you'll end up with 50 tabs open all named `Styles.ts`. Use `Button.styles.ts`.
2.  **Abbreviations:** `UsrProfImg.tsx` vs `UserProfileImage.tsx`. Screen real estate is cheap; developer time spent deciphering acronyms is expensive.
3.  **Redundant Naming:** `const [userState, setUserState] = useState()`. Since it's a state hook, `user` and `setUser` are sufficient.

---

## Summary & Key Takeaways

- **Components:** `PascalCase` (e.g., `Header.tsx`).
- **Non-Components:** `camelCase` (e.g., `formatter.ts`).
- **Props:** Use `is/has/should` for booleans; `onEvent` for callbacks.
- **Consistency > Preference:** It doesn't matter if you prefer `kebab-case` or `PascalCase` for files, as long as the **entire team** uses the same one.
- **Interview Tip:** If asked about naming, mention **"Grep-ability"** and **"Predictability"**. A senior engineer cares about how easy it is to find a bug at 3 AM based on file names.