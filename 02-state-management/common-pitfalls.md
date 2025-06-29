# Common Pitfalls in React State Management

Avoiding anti-patterns and common mistakes is essential for building maintainable, high-performance React apps. This section highlights frequent pitfalls and how to avoid them.

## Key Interview Questions & Answers

### 1. What are the most common anti-patterns in Redux and Context API usage?

**Answer:**

#### Redux Anti-Patterns
- **Mutating State Directly:**
  - *Why it happens:* Developers sometimes mutate state directly in reducers (e.g., `state.value = 1`), which breaks Redux's immutability contract and can cause unpredictable bugs.
  - *Example (bad):*
    ```js
    // ❌ Bad: Direct mutation
    function counterReducer(state, action) {
      switch (action.type) {
        case 'INCREMENT':
          state.value += 1;
          return state;
        default:
          return state;
      }
    }
    ```
  - *Best Practice:* Always return a new state object. Use Redux Toolkit's `createSlice`, which uses Immer under the hood to allow "mutative" code that is actually safe.
    ```js
    // ✅ Good: Redux Toolkit createSlice
    const counterSlice = createSlice({
      name: 'counter',
      initialState: { value: 0 },
      reducers: {
        increment: (state) => { state.value += 1; }, // Safe with Immer
      },
    });
    ```

- **Putting Non-Serializable Data in State:**
  - *Why it happens:* Storing functions, class instances, or Promises in Redux state can break time-travel debugging and persistence.
  - *Best Practice:* Only store serializable data (plain objects, arrays, primitives). Use middleware like `redux-immutable-state-invariant` in dev.

- **Too Much Global State:**
  - *Why it happens:* Developers put all state in Redux, even UI state (e.g., modal open/close), leading to unnecessary complexity and re-renders.
  - *Best Practice:* Use Redux for global, cross-cutting state. Keep local UI state (e.g., form inputs, toggles) in component state with `useState`.

#### Context API Anti-Patterns
- **Overusing Context for High-Frequency State:**
  - *Why it happens:* Using Context for rapidly changing state (e.g., input values) causes all consumers to re-render on every change, hurting performance.
  - *Best Practice:* Use Context for static or low-frequency state (e.g., theme, auth). For dynamic state, combine Context with `useReducer` or use Redux.

- **Large Context Value Objects:**
  - *Why it happens:* Passing a large object as context value causes all consumers to re-render when any property changes.
  - *Best Practice:* Split context into smaller providers, or memoize context values with `useMemo`.

---

### 2. How do you avoid deeply nested or over-normalized state?

**Answer:**

- **Deeply Nested State:**
  - *Problem:* Deeply nested objects/arrays are hard to update immutably and cause unnecessary re-renders.
  - *Example (bad):*
    ```js
    // ❌ Bad: Deep nesting
    const state = {
      user: {
        profile: {
          address: {
            city: 'NYC',
            zip: '10001',
          },
        },
      },
    };
    ```
  - *Best Practice:* Flatten state as much as possible. In Redux, use normalization (e.g., with `createEntityAdapter` or `normalizr`).
    ```js
    // ✅ Good: Normalized state
    const state = {
      users: { byId: { 1: { id: 1, name: 'Alice' } }, allIds: [1] },
      addresses: { byId: { 1: { id: 1, city: 'NYC', zip: '10001' } }, allIds: [1] },
    };
    ```

- **Over-Normalization:**
  - *Problem:* Over-normalizing tiny or rarely-updated data adds complexity without benefit.
  - *Best Practice:* Normalize only large, relational, or frequently updated collections. For small, static data, keep it simple.

---

### 3. What are the risks of using array indices as keys in lists?

**Answer:**

- **Why it's a problem:**
  - React uses the `key` prop to track list items for efficient updates. Using array indices as keys can cause bugs when items are reordered, inserted, or deleted, leading to UI glitches or state leaks between items.
- **Example (bad):**
  ```jsx
  // ❌ Bad: Using index as key
  {items.map((item, i) => <li key={i}>{item.name}</li>)}
  ```
- **Best Practice:**
  - Use a unique, stable identifier (e.g., `item.id`) as the key.
  ```jsx
  // ✅ Good: Using unique id
  {items.map(item => <li key={item.id}>{item.name}</li>)}
  ```
- **Tools:**
  - Use [eslint-plugin-react](https://github.com/jsx-eslint/eslint-plugin-react) to warn about unsafe keys.
  - Use [why-did-you-render](https://github.com/welldone-software/why-did-you-render) to detect unnecessary re-renders.

---

### 4. How do you prevent memory leaks in event-driven state management?

**Answer:**

- **Problem:**
  - When using event buses, subscriptions, or listeners (e.g., in micro-frontends or custom hooks), failing to clean up listeners can cause memory leaks and unexpected behavior.
- **Example (bad):**
  ```js
  useEffect(() => {
    eventBus.on('update', handler);
    // ❌ No cleanup: handler stays after component unmounts
  }, []);
  ```
- **Best Practice:**
  - Always return a cleanup function in `useEffect` to remove listeners.
  ```js
  useEffect(() => {
    eventBus.on('update', handler);
    return () => eventBus.off('update', handler);
  }, []);
  ```
  - For custom event buses, implement an `off` method to remove listeners.
  - In Redux, use middleware for side effects and avoid manual event subscriptions when possible.

---

### 5. What are the dangers of mixing local and global state without clear boundaries?

**Answer:**

- **Problem:**
  - Mixing local (`useState`) and global (Redux/Context) state for the same data can cause bugs, race conditions, and hard-to-debug UI inconsistencies.
- **Example:**
  - A form input is controlled by both local state and Redux, leading to out-of-sync values.
- **Best Practice:**
  - Define clear ownership: UI state (inputs, toggles) should be local; cross-component or app-wide state should be global.
  - Never duplicate the same piece of state in both local and global stores.
  - Use custom hooks to encapsulate state logic and expose a single source of truth.

---

### 6. How do you handle error states and rollback in async flows?

**Answer:**

- **Problem:**
  - Failing to handle errors in async actions (e.g., API calls) can leave the UI in an inconsistent state or confuse users.
- **Best Practice:**
  - In Redux Toolkit, use `createAsyncThunk` for async actions. Handle `pending`, `fulfilled`, and `rejected` states in your slice.
  - Roll back optimistic updates if the async action fails.
  - Always show user-friendly error messages and allow retry.
- **Example:**
  ```js
  // Redux Toolkit slice with async thunk
  const fetchUser = createAsyncThunk('user/fetch', async (id) => {
    const response = await api.getUser(id);
    return response.data;
  });

  const userSlice = createSlice({
    name: 'user',
    initialState: { data: null, loading: false, error: null },
    extraReducers: (builder) => {
      builder
        .addCase(fetchUser.pending, (state) => { state.loading = true; state.error = null; })
        .addCase(fetchUser.fulfilled, (state, action) => { state.loading = false; state.data = action.payload; })
        .addCase(fetchUser.rejected, (state, action) => { state.loading = false; state.error = action.error.message; });
    },
  });
  ```
  - For optimistic updates, store a backup of the previous state and restore it if the async action fails.

---

**Summary Table: Common Pitfalls & Solutions**

| Pitfall | Why it Happens | How to Avoid |
|---------|----------------|--------------|
| Mutating state directly | Lack of immutability awareness | Use Redux Toolkit, always return new state |
| Non-serializable state | Storing functions/classes | Only store plain data |
| Too much global state | Overusing Redux | Keep UI state local |
| Overusing Context | Using for high-frequency state | Use for static/low-frequency state |
| Deeply nested state | Modeling after backend | Normalize/flatten state |
| Array indices as keys | No unique id | Use stable unique keys |
| Memory leaks | Not cleaning up listeners | Always clean up in useEffect |
| Mixed local/global state | No clear boundaries | Define ownership, avoid duplication |
| Poor async error handling | Not handling rejected promises | Use async thunks, handle all states |

_Answers, code examples, and real-world scenarios will be added below each question._ 