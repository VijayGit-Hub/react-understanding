# React.memo Deep Dive

## What is React.memo?

`React.memo` is a higher-order component that memoizes a functional component. It prevents unnecessary re-renders by only re-rendering the component if its props have changed (using shallow comparison by default).

---

## Why is React.memo Needed?

### Default React Behavior
- When a parent component re-renders, **all its child components re-render by default**, even if their props have not changed.
- This is because React cannot know if a child depends on something other than props (like context or state), so it re-renders the entire subtree for safety.

### Problem
- In large apps, parent components may re-render frequently (due to state or context changes), causing expensive child components to re-render unnecessarily.
- This can lead to performance bottlenecks, especially for components with heavy computation or large DOM trees.

### Solution: React.memo
- `React.memo` tells React: "If the props to this component are the same as last time, skip re-rendering this component."
- It does a **shallow comparison** of the props by default.

---

## How Does React.memo Work?

```jsx
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data, onUpdate }) {
  // This will only log if props actually change
  console.log('ExpensiveComponent rendered');
  return (
    <div>
      {data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      <button onClick={onUpdate}>Update</button>
    </div>
  );
});
```

- If the parent re-renders but `data` and `onUpdate` are unchanged, `ExpensiveComponent` will **not** re-render.

---

## Custom Comparison Function

By default, `React.memo` does a shallow comparison. For complex props (like arrays or objects), you can provide a custom comparison function:

```jsx
const areEqual = (prevProps, nextProps) => {
  // Only re-render if the 'id' field of the data prop changes
  return prevProps.data.id === nextProps.data.id;
};
const MemoizedComponent = React.memo(MyComponent, areEqual);
```

---

## Practical Scenario

Suppose you have a parent component that updates its own state (like a counter), but passes the same data prop to a child:

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const data = useMemo(() => [/* ...large array... */], []);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <ExpensiveChild data={data} />
    </>
  );
}
```
- Every time you click the button, `Parent` re-renders.
- By default, `ExpensiveChild` will also re-render, **even though `data` hasn't changed**.
- If you wrap `ExpensiveChild` with `React.memo`, it will **not re-render** unless `data` changes.

---

## Summary Table

| Without React.memo         | With React.memo                |
|---------------------------|-------------------------------|
| Child re-renders whenever parent re-renders, regardless of prop changes | Child only re-renders if props actually change (by shallow or custom comparison) |

---

## Key Takeaways
- `React.memo` is not about "re-render on prop change" (which is default), but about **skipping re-renders when props have NOT changed**, even if the parent re-renders.
- This is a key optimization for expensive or deeply nested components.
- Use custom comparison for complex props if needed. 