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

- The `key` prop helps React identify which items have changed, are added, or are removed.
- Using array indices as keys can cause bugs and performance issues, especially when items are reordered or removed.

```jsx
// Bad: Using index as key
{items.map((item, i) => <li key={i}>{item.name}</li>)}

// Good: Use a unique, stable id
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

---

## 4. Batch State Updates

- React batches multiple state updates into a single render for better performance.
- In event handlers, updates are batched automatically. In async code (like setTimeout or Promises), use `ReactDOM.flushSync` or `startTransition` (React 18+) for batching.

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

---

## 5. Avoid Deeply Nested State

- Deeply nested state objects are harder to update efficiently and can lead to unnecessary re-renders.
- Normalize state shape (especially in Redux or large apps).

---

## 6. Use Production Builds

- Development builds of React include extra checks and warnings, making them slower.
- Always use production builds for deployment (`npm run build`).

---

## 7. Profile and Measure

- Use React DevTools Profiler to identify slow components and unnecessary renders.
- Optimize only where you see real performance issues.

---

## Key Takeaways
- Use PureComponent/shouldComponentUpdate for class components.
- Avoid anonymous functions in render.
- Use stable, unique keys in lists.
- Batch updates and avoid deeply nested state.
- Always profile before optimizing. 