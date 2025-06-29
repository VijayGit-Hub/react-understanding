# Performance Patterns in State Management

Optimizing state management is critical for large React apps. This section covers memoization, selectors, normalization, and other advanced performance techniques.

## Key Interview Questions & Answers

### 1. How do you use memoized selectors (e.g., Reselect) to optimize Redux performance?

**Answer:**
- **Problem:** In Redux, components often use `useSelector` to read state. If you compute derived data (e.g., filtering, mapping) inside a selector, it will recompute on every render unless memoized, causing unnecessary re-renders and wasted CPU.
- **Solution:** Use [Reselect](https://github.com/reduxjs/reselect) or Redux Toolkit's `createSelector` to memoize selectors. Memoized selectors only recompute when their inputs change.
- **Example:**
  ```js
  import { createSelector } from '@reduxjs/toolkit';
  // State shape: { users: { byId: {...}, allIds: [...] } }
  const selectUsers = state => state.users.allIds.map(id => state.users.byId[id]);
  // Memoized selector for filtered users
  const selectActiveUsers = createSelector(
    [selectUsers],
    users => users.filter(u => u.active)
  );
  // In component:
  const activeUsers = useSelector(selectActiveUsers);
  ```
- **Best Practices:**
  - Use memoized selectors for any derived or computed data.
  - Use selectors for normalization/denormalization, filtering, sorting, or aggregating state.
  - Avoid creating selectors inside components (define them outside to preserve memoization).
- **Pitfall:** If you pass new objects/arrays as selector arguments on every render, memoization breaks. Always use stable references.

---

### 2. What is state colocation, and why does it matter?

**Answer:**
- **Definition:** State colocation means keeping state as close as possible to where it is used. Local UI state (e.g., input values, toggles) should live in the component; only global, cross-cutting state should be in Redux or Context.
- **Why it matters:**
  - Reduces unnecessary re-renders (local state changes don't affect the whole app).
  - Makes components easier to reason about and test.
  - Avoids "global state bloat" and accidental coupling.
- **Example:**
  ```jsx
  // Good: Local state for UI
  function SearchBox() {
    const [query, setQuery] = useState('');
    return <input value={query} onChange={e => setQuery(e.target.value)} />;
  }
  // Only lift to Redux if query needs to be shared across many components.
  ```
- **Best Practices:**
  - Start with local state; lift to global only when truly needed.
  - Use custom hooks to encapsulate reusable state logic.
- **Pitfall:** Overusing global state leads to unnecessary complexity and performance issues.

---

### 3. How do you handle deeply nested state updates efficiently?

**Answer:**
- **Problem:** Deeply nested state (e.g., objects within objects) is hard to update immutably and can cause unnecessary re-renders if not managed carefully.
- **Solution:**
  - **Normalize state:** Use flat structures (e.g., `{ byId, allIds }`) for collections.
  - **Use helper libraries:** Redux Toolkit's `createEntityAdapter` or [normalizr](https://github.com/paularmstrong/normalizr) for normalization.
- **Example:**
  ```js
  // Normalized state
  const state = {
    users: { byId: { 1: { id: 1, name: 'Alice' } }, allIds: [1] },
    posts: { byId: { 10: { id: 10, userId: 1, title: 'Hello' } }, allIds: [10] }
  };
  // Updating a user's name
  const newState = {
    ...state,
    users: {
      ...state.users,
      byId: {
        ...state.users.byId,
        1: { ...state.users.byId[1], name: 'Bob' }
      }
    }
  };
  ```
- **Best Practices:**
  - Keep state as flat as possible.
  - Use selectors to denormalize for UI.
  - Use Redux Toolkit's `createEntityAdapter` for CRUD operations on collections.
- **Pitfall:** Passing large, deeply nested objects as props can cause unnecessary re-renders. Use selectors to pass only what's needed.

---

### 4. How do you avoid unnecessary re-renders in large component trees?

**Answer:**
- **Problem:** In React, parent re-renders can cause all children to re-render, even if their props/state haven't changed.
- **Solutions:**
  - **React.memo:** Wrap pure function components to prevent re-renders if props are shallowly equal.
  - **useMemo/useCallback:** Memoize expensive computations and stable function references passed as props.
  - **Selector optimization:** Use memoized selectors (see above) to avoid passing new objects/arrays as props.
  - **Split context:** For Context API, use multiple providers or selector libraries to avoid global re-renders.
- **Example:**
  ```jsx
  const ExpensiveList = React.memo(function ExpensiveList({ items }) {
    // ...
  });
  // In parent:
  const stableItems = useMemo(() => computeItems(data), [data]);
  <ExpensiveList items={stableItems} />
  ```
- **Best Practices:**
  - Use React DevTools Profiler to find unnecessary re-renders.
  - Use [why-did-you-render](https://github.com/welldone-software/why-did-you-render) to detect wasted renders in dev.
- **Pitfall:** Overusing memoization can add complexity and sometimes hurt performance if dependencies are not managed carefully.

---

### 5. What are the trade-offs between local and global state for performance?

**Answer:**
- **Local state (useState/useReducer):**
  - Fast, isolated, and causes minimal re-renders.
  - Best for UI state, form inputs, toggles, etc.
- **Global state (Redux/Context):**
  - Needed for cross-cutting, shared, or persistent state.
  - Can cause more re-renders if not managed with selectors/memoization.
- **Trade-offs:**
  - Too much global state = unnecessary re-renders, complexity, and coupling.
  - Too much local state = hard to coordinate or share data between components.
- **Best Practices:**
  - Start local, lift to global only when needed.
  - Use selectors and memoization for global state.
- **Pitfall:** Duplicating the same state in both local and global stores can cause bugs and out-of-sync UI.

---

### 6. How do you profile and measure state-related performance bottlenecks?

**Answer:**
- **Tools:**
  - **React DevTools Profiler:** Visualize component render times, flamegraphs, and wasted renders.
  - **why-did-you-render:** Warns in dev when unnecessary re-renders happen.
  - **Redux DevTools:** Inspect state changes, action history, and time-travel debugging.
  - **Browser Performance tab:** Analyze scripting, rendering, and painting bottlenecks.
- **Best Practices:**
  - Always measure before optimizing—don't guess!
  - Profile before and after changes to verify improvement.
  - Focus on optimizing the slowest or most frequently re-rendered components.
- **Pitfall:** Premature optimization can waste time and add complexity. Always start with measurement.

---

_Answers, code examples, and real-world scenarios will be added below each question._ 