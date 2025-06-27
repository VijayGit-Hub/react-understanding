# useCallback Deep Dive

## What is useCallback?

`useCallback` is a React Hook that memoizes a function reference. It returns a memoized version of the callback that only changes if its dependencies change.

---

## Why is useCallback Needed?

### Default React Behavior
- Every time a component re-renders, **all functions defined inside the component are recreated** (new reference in memory).
- If you pass these functions as props to child components, it can cause those children to re-render, even if the logic is the same.
- **You can observe this by adding a `console.log` inside your function or by logging the function reference:**

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    console.log('handleClick created');
  };
  console.log('handleClick reference:', handleClick);
  // ...
}
```
- On every render, you'll see a new function reference in the console unless you use `useCallback`.

### Problem
- Passing new function references to memoized children (e.g., with `React.memo`) will cause them to re-render, defeating the purpose of memoization.

### Solution: useCallback
- `useCallback` tells React: "Only recreate this function if its dependencies change. Otherwise, reuse the previous function reference."

---

## How Does useCallback Work?

```jsx
const handleAddItem = useCallback((newItem) => {
  setItems(prev => [...prev, newItem]);
}, []); // No dependencies, so reference is stable
```
- `handleAddItem` will have the same reference across renders unless dependencies change.

---

## Practical Scenario

Suppose you have a parent component that passes handlers to a memoized child:

```jsx
function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  // handleAddItem will have a stable reference unless dependencies change.
  const handleAddItem = useCallback((newItem) => {
    setItems(prev => [...prev, newItem]);
  }, []);

  // handleRemoveItem is also stable.
  const handleRemoveItem = useCallback((itemId) => {
    setItems(prev => prev.filter(item => item.id !== itemId));
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      {/* ChildComponent will not re-render unless items or handlers change */}
      <ChildComponent 
        items={items} 
        onAdd={handleAddItem} 
        onRemove={handleRemoveItem} 
      />
    </div>
  );
}
```
- Without `useCallback`, new function references would be created on every render, causing `ChildComponent` to re-render even if `items` didn't change.
- With `useCallback`, the handlers are stable, so `ChildComponent` only re-renders when relevant data changes.

---

## Drawbacks and Best Practices

### Drawbacks of Overusing useCallback
- **Memory and CPU Overhead:** Each use of `useCallback` adds a little memory and CPU overhead, as React must track dependencies and store the memoized function.
- **Code Complexity:** Wrapping every function in `useCallback` can make your code harder to read and maintain, especially if the dependencies are complex.
- **No Benefit for Inline or Non-Prop Functions:** If a function is not passed to a memoized child or is not expensive to recreate, using `useCallback` provides no real benefit.
- **False Sense of Optimization:** Overusing `useCallback` can actually hurt performance in some cases, due to the extra work React does to track and compare dependencies.

### When Should You Use useCallback?
- **Use it when:**
  - You pass a function as a prop to a memoized child (e.g., a component wrapped in `React.memo`).
  - The function is expensive to recreate or triggers expensive side effects.
- **Avoid it when:**
  - The function is only used inside the component and not passed down.
  - The function is trivial and cheap to recreate.
  - You are not seeing real performance issues.

**Rule of Thumb:**
> Use `useCallback` (and `useMemo`) as a performance optimization tool, not as a default for every function. Profile your app and optimize only where necessary.

---

## Key Takeaways
- Use `useCallback` to memoize functions passed to memoized children.
- Prevents unnecessary re-renders in child components, especially in large or deeply nested trees.
- Only use when you have a real performance bottleneck—overusing `useCallback` can add complexity and even hurt performance. 