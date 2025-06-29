# Context API: Senior-Level Interview Questions & Best Practices

React Context API is a built-in solution for sharing state across components without prop drilling. It is best suited for static or low-frequency state and small-to-medium apps.

## Key Interview Questions & Answers

### 1. When is Context API a better choice than Redux Toolkit?

**Answer:**
- **Context API is best for:**
  - Sharing static or rarely-changing data (e.g., theme, locale, authenticated user info) across the component tree.
  - Small to medium apps where global state is simple and doesn't require advanced features (middleware, devtools, async logic).
  - Avoiding extra dependencies and boilerplate for simple use cases.
- **When NOT to use Context API:**
  - For large, frequently-changing, or deeply relational state (e.g., large lists, normalized data, complex async flows).
  - When you need middleware, time-travel debugging, or advanced devtools (use Redux Toolkit).
- **Example:**
  ```jsx
  // Theme context (good use case)
  const ThemeContext = React.createContext('light');
  function App() {
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
  function Toolbar() {
    const theme = React.useContext(ThemeContext);
    return <div className={theme}>Toolbar</div>;
  }
  ```

---

### 2. What are the performance pitfalls of Context API, and how do you mitigate them?

**Answer:**
- **Pitfall:** Every time the context value changes, all consuming components re-render, even if only a small part of the value changed.
- **Why it matters:** For high-frequency or large context values, this can cause unnecessary re-renders and performance bottlenecks.
- **How to mitigate:**
  - **Split context:** Use multiple, smaller context providers for logically separate data.
  - **Memoize context value:** Use `useMemo` to avoid unnecessary value changes.
  - **Selector pattern:** Use custom hooks to select only the needed part of context (see below).
- **Example:**
  ```jsx
  // Memoizing context value
  const UserContext = React.createContext();
  function UserProvider({ children }) {
    const [user, setUser] = React.useState({ name: 'Alice', age: 30 });
    const value = React.useMemo(() => ({ user, setUser }), [user]);
    return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
  }
  ```
  - **Selector pattern:** (with use-context-selector library)
    ```jsx
    import { createContext, useContextSelector } from 'use-context-selector';
    const UserContext = createContext();
    // In consumer:
    const userName = useContextSelector(UserContext, v => v.user.name);
    ```

---

### 3. How do you structure context providers for large applications?

**Answer:**
- **Best Practice:**
  - Use multiple, focused context providers (e.g., AuthContext, ThemeContext, SettingsContext) instead of a single "global" context.
  - Compose providers at the app root or at feature boundaries.
  - Encapsulate context logic in custom hooks (e.g., `useAuth`, `useTheme`).
- **Example:**
  ```jsx
  function AppProviders({ children }) {
    return (
      <AuthProvider>
        <ThemeProvider>
          <SettingsProvider>
            {children}
          </SettingsProvider>
        </ThemeProvider>
      </AuthProvider>
    );
  }
  // Usage: <AppProviders><App /></AppProviders>
  ```

---

### 4. How do you avoid unnecessary re-renders with Context?

**Answer:**
- **Problem:** All consumers re-render when the context value changes, even if they only use part of the value.
- **Solutions:**
  - **Split context:** As above, use multiple providers for unrelated data.
  - **Memoize value:** Use `useMemo` for the context value.
  - **Selector pattern:** Use libraries like `use-context-selector` or `react-tracked` to subscribe to only the needed part of context.
  - **Local state for high-frequency data:** Keep rapidly changing state local, and only lift to context if truly needed globally.
- **Example:**
  ```jsx
  // Only re-render when 'theme' changes, not when 'user' changes
  const ThemeContext = React.createContext();
  const UserContext = React.createContext();
  ```

---

### 5. What are the limitations of Context API for global state management?

**Answer:**
- No built-in support for middleware, devtools, or time-travel debugging.
- All consumers re-render on any value change (no built-in selectors or memoized subscriptions).
- Not ideal for large, frequently-changing, or normalized data.
- No built-in async logic handling (e.g., thunks, sagas).
- Can lead to "provider hell" if overused (deeply nested providers).
- **When to use Redux Toolkit or other solutions:**
  - When you need advanced debugging, middleware, or scalable state patterns.
  - For large apps with complex state relationships.

---

### Selector Pattern with use-context-selector: Solving the Re-render Pitfall

**Problem Recap:**
- With vanilla Context API, *all* consumers re-render whenever the context value changes, even if they only use a small part of the value. This is a major performance pitfall for large or frequently-changing state.
- There is also no built-in way to "select" only a part of the context value, like you can with Redux selectors.

**Solution:**
- Libraries like [`use-context-selector`](https://github.com/dai-shi/use-context-selector) allow consumers to subscribe only to the part of the context they care about. This means only components that actually use the changed value will re-render.

**Example: Using use-context-selector for Fine-Grained Subscriptions**

```jsx
import React from 'react';
import { createContext, useContextSelector } from 'use-context-selector';

// 1. Create context
const UserContext = createContext(null);

// 2. Provider with a complex value
function UserProvider({ children }) {
  const [user, setUser] = React.useState({ name: 'Alice', age: 30 });
  const [theme, setTheme] = React.useState('light');
  const value = React.useMemo(() => ({ user, setUser, theme, setTheme }), [user, theme]);
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

// 3. Consumer that only cares about user name
function UserName() {
  // Only re-renders when user.name changes
  const name = useContextSelector(UserContext, v => v.user.name);
  return <div>User: {name}</div>;
}

// 4. Consumer that only cares about theme
function ThemeDisplay() {
  // Only re-renders when theme changes
  const theme = useContextSelector(UserContext, v => v.theme);
  return <div>Theme: {theme}</div>;
}

// 5. App usage
function App() {
  return (
    <UserProvider>
      <UserName />
      <ThemeDisplay />
    </UserProvider>
  );
}
```

**Explanation:**
- `useContextSelector` lets each consumer specify exactly which part of the context value it cares about.
- If only `user.name` changes, *only* `UserName` re-renders; `ThemeDisplay` does not.
- This pattern brings Context API closer to Redux's selector-based performance, but without middleware/devtools/time-travel.
- For large or performance-sensitive apps, this is a powerful way to avoid unnecessary re-renders while still using Context.

---

### 6. How do you test components that consume context?

**Answer:**
- **Best Practice:**
  - Wrap the component in the relevant provider with a test value.
  - Use React Testing Library or similar tools to render and assert behavior.
- **Example:**
  ```jsx
  // MyComponent.js
  export function MyComponent() {
    const theme = React.useContext(ThemeContext);
    return <div className={theme}>Hello</div>;
  }
  // MyComponent.test.js
  import { render, screen } from '@testing-library/react';
  test('renders with dark theme', () => {
    render(
      <ThemeContext.Provider value="dark">
        <MyComponent />
      </ThemeContext.Provider>
    );
    expect(screen.getByText('Hello')).toHaveClass('dark');
  });
  ```

---

### 7. How do you combine Context API with other state management solutions?

**Answer:**
- **Pattern:**
  - Use Context for cross-cutting concerns (theme, auth, feature flags).
  - Use Redux Toolkit, Zustand, or other libraries for complex, relational, or high-frequency state.
  - Context can provide access to a Redux store or custom state manager.
- **Example:**
  ```jsx
  // Provide Redux store via context
  import { Provider } from 'react-redux';
  function App() {
    return (
      <Provider store={store}>
        <ThemeProvider>
          <AppRoutes />
        </ThemeProvider>
      </Provider>
    );
  }
  ```
- **Custom hooks:**
  - Use custom hooks to abstract state logic and expose a clean API to components.

---

**Summary Table: Context API Best Practices**

| Use Case | Context API | Redux Toolkit |
|----------|-------------|---------------|
| Theme, locale, auth | ✅ | 🚫 |
| Large, relational, or normalized data | 🚫 | ✅ |
| High-frequency updates | 🚫 | ✅ |
| Middleware/devtools | 🚫 | ✅ |
| Simple, static global state | ✅ | 🚫 |
| Async logic | 🚫 | ✅ |

_Answers, code examples, and real-world scenarios will be added below each question._ 