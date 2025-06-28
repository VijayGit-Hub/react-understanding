# General React Performance Optimization Tips

This guide covers additional performance best practices in React beyond memoization, code splitting, and virtual scrolling.

---

## 1. Use React.PureComponent and shouldComponentUpdate (Class Components)

- **React.PureComponent** automatically implements a shallow prop and state comparison in class components, preventing unnecessary re-renders.
- **shouldComponentUpdate** allows you to manually control when a component should update.

```jsx
class MyComponent extends React.PureComponent {
  render() {
    // Will only re-render if props or state change (shallow comparison)
    return <div>{this.props.value}</div>;
  }
}

class CustomComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Only update if value prop changes
    return nextProps.value !== this.props.value;
  }
  render() {
    return <div>{this.props.value}</div>;
  }
}
```
- **Best Practice:** Use PureComponent for simple, pure class components. Use shouldComponentUpdate for custom logic.

---

## 2. Avoid Creating Anonymous Functions in Render

- Creating new functions inside render (or JSX) causes new references on every render, which can trigger unnecessary re-renders in child components.

```jsx
// Bad: New function created on every render
<MyButton onClick={() => doSomething(id)} />

// Good: Use useCallback or class method
const handleClick = useCallback(() => doSomething(id), [id]);
<MyButton onClick={handleClick} />
```

---

## 3. Use Stable and Unique Keys in Lists

- The `key` prop helps React identify which items have changed, are added, or are removed. This is crucial for efficient reconciliation and DOM updates.
- **Why not use array indices?** Using array indices as keys can cause subtle bugs and performance issues, especially when items are reordered, inserted, or removed. React may reuse DOM nodes incorrectly, leading to UI glitches or even state leaks between list items.
- **Best Practice:** Always use a unique, stable identifier (like a database ID or UUID) as the key.

**Example:**
```jsx
// Bad: Using index as key (can cause bugs if list changes)
{items.map((item, i) => <li key={i}>{item.name}</li>)}

// Good: Use a unique, stable id
{items.map(item => <li key={item.id}>{item.name}</li>)}
```
- **Tools:**
  - [eslint-plugin-react](https://github.com/jsx-eslint/eslint-plugin-react) can warn you about missing or unsafe keys in lists.
  - [why-did-you-render](https://github.com/welldone-software/why-did-you-render) can help you spot unnecessary re-renders caused by unstable keys.

---

## 4. Batch State Updates

- React batches multiple state updates into a single render for better performance. This reduces the number of re-renders and DOM updates, making your app more efficient.
- **Automatic Batching:** In event handlers and lifecycle methods, React automatically batches updates.
- **Async Batching:** In async code (like setTimeout, Promises, or async/await), React 18+ supports automatic batching. In older versions, updates in async code may not be batched, leading to extra renders.
- **Manual Batching:**
  - Use `ReactDOM.flushSync` to force synchronous updates (rarely needed, use with caution).
  - Use `startTransition` (React 18+) for non-urgent updates to keep the UI responsive.

**Example:**
```jsx
// React 18+ startTransition for non-urgent updates
import { startTransition } from 'react';

function handleInputChange(e) {
  const value = e.target.value;
  // Mark as non-urgent
  startTransition(() => {
    setSearchQuery(value);
  });
}
```
- **Real-World Scenario:**
  - When updating both UI state (like form input) and triggering a heavy computation or network request, use `startTransition` for the non-urgent part to keep the UI snappy.
- **Tools:**
  - [React Profiler](https://react.dev/learn/profile-performance-with-the-react-devtools-profiler) can help you see how many renders are triggered by state updates.
  - [why-did-you-render](https://github.com/welldone-software/why-did-you-render) can help you spot unnecessary renders due to unbatched updates.

---

## 5. Avoid Deeply Nested State

- **Why is deeply nested state a problem?**
  - Updating deeply nested objects or arrays in React is cumbersome and error-prone, as you must create new copies at every level to maintain immutability.
  - Passing deeply nested objects as props can cause unnecessary re-renders, because even a small change deep in the tree changes the top-level reference.
  - Shallow comparison (used by React.memo, PureComponent, etc.) only checks the top-level reference, so nested changes can be missed or cause unnecessary re-renders.

### Example: Deeply Nested State vs. Normalized State

#### ❌ Deeply Nested State
```jsx
const [state, setState] = useState({
  user: {
    name: 'Alice',
    address: {
      city: 'New York',
      zip: '10001'
    }
  },
  posts: [
    { id: 1, title: 'Hello', comments: [{ id: 1, text: 'Nice!' }] },
    { id: 2, title: 'World', comments: [] }
  ]
});

// Updating a comment's text immutably is verbose and error-prone:
setState(prev => ({
  ...prev,
  posts: prev.posts.map(post =>
    post.id === 1
      ? {
          ...post,
          comments: post.comments.map(comment =>
            comment.id === 1 ? { ...comment, text: 'Awesome!' } : comment
          )
        }
      : post
  )
}));
```
- This is hard to read, easy to get wrong, and can cause unnecessary re-renders if large objects are passed as props.

#### ✅ Normalized (Flat) State (Redux-style)
```js
const state = {
  users: {
    1: { id: 1, name: 'Alice', address: { city: 'New York', zip: '10001' } }
  },
  posts: {
    1: { id: 1, title: 'Hello', comments: [1] },
    2: { id: 2, title: 'World', comments: [] }
  },
  comments: {
    1: { id: 1, text: 'Nice!', postId: 1 }
  }
};
// Now, updating a comment is a simple object update:
const updatedState = {
  ...state,
  comments: {
    ...state.comments,
    1: { ...state.comments[1], text: 'Awesome!' }
  }
};
```
- This is much easier to update, reason about, and optimize for performance.
- **Redux and large apps** use this pattern to avoid deeply nested state and make updates predictable and efficient.

- **Tools & Techniques:**
  - [normalizr](https://github.com/paularmstrong/normalizr) for normalizing nested API responses.
  - Redux Toolkit's `createEntityAdapter` for managing normalized collections.
  - Use selectors (with [reselect](https://github.com/reduxjs/reselect)) to efficiently compute derived data from normalized state.

- **Summary:**
  - Keep state as flat as possible.
  - Normalize nested data for easier updates and better performance.
  - Use libraries and patterns that encourage immutability and normalization.

---

## 6. Use Production Builds

- Development builds of React include extra checks and warnings, making them slower.
- Always use production builds for deployment (`npm run build`).

---

## 7. Profile and Measure

- **Why profile?** Optimizing without measurement can lead to wasted effort or even worse performance. Always measure first!
- **React DevTools Profiler:**
  - Use the [React DevTools Profiler](https://react.dev/learn/profile-performance-with-the-react-devtools-profiler) to record and analyze component render times, flamegraphs, and wasted renders.
  - Identify which components re-render most often and why.
- **why-did-you-render:**
  - Add [why-did-you-render](https://github.com/welldone-software/why-did-you-render) to your project to get console warnings when unnecessary re-renders happen (great for development and debugging!).
- **Browser Performance Tools:**
  - Use Chrome DevTools Performance tab to analyze scripting, rendering, and painting bottlenecks.
- **Other Tools:**
  - [React Profiler API](https://react.dev/reference/react/Profiler) for custom profiling in code.
  - [Web Vitals](https://web.dev/vitals/) for measuring real user performance (FCP, LCP, CLS, etc.).
- **Best Practice:**
  - Profile before and after making optimizations to ensure your changes actually improve performance.
  - Focus on optimizing the slowest components or the most frequent re-renders.

---

## Key Takeaways
- Use PureComponent/shouldComponentUpdate for class components.
- Avoid anonymous functions in render.
- Use stable, unique keys in lists.
- Batch updates and avoid deeply nested state.
- Always profile before optimizing. 