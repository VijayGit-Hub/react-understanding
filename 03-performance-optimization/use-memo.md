# useMemo Deep Dive

## What is useMemo?

`useMemo` is a React Hook that memoizes the result of an expensive calculation or object creation. The value is recomputed only if its dependencies change.

---

## Why is useMemo Needed?

### Default React Behavior
- Every time a component re-renders, **all code inside the component function runs again**.
- This means expensive calculations or object/array creations will be re-executed on every render, even if the inputs haven't changed.

### Problem
- If you have expensive computations or create new objects/arrays on every render, it can lead to performance issues and unnecessary re-renders in child components (if those objects are passed as props).

### Solution: useMemo
- `useMemo` tells React: "Only recompute this value if its dependencies change. Otherwise, reuse the previous value."

---

## How Does useMemo Work?

```jsx
const processedData = useMemo(() => {
  // Expensive calculation here
  return data.filter(item => item.active).sort((a, b) => a.value - b.value);
}, [data]);
```
- `processedData` is only recalculated if `data` changes.
- On other renders, the previous value is reused.

---

## Practical Scenario

Suppose you have a filtered and sorted list that is expensive to compute:

```jsx
function DataTable({ data, filters, sortBy }) {
  // Only recompute processedData if data, filters, or sortBy change.
  const processedData = useMemo(() => {
    // This log will only appear when dependencies change.
    console.log('Processing data...');
    return data
      .filter(item => filters.every(filter => filter(item)))
      .sort((a, b) => {
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        if (sortBy === 'date') return new Date(b.date) - new Date(a.date);
        return 0;
      });
  }, [data, filters, sortBy]);

  return (
    <table>
      {processedData.map(item => (
        <tr key={item.id}>
          <td>{item.name}</td>
          <td>{item.date}</td>
        </tr>
      ))}
    </table>
  );
}
```
- Without `useMemo`, `processedData` would be recalculated on every render, even if `data`, `filters`, and `sortBy` haven't changed.
- With `useMemo`, recalculation only happens when dependencies change, saving CPU and avoiding unnecessary work.

---

## useMemo vs useCallback

- **useMemo** is for memoizing **values** (results of calculations, objects, arrays, etc.).
- **useCallback** is for memoizing **functions** (so their reference stays stable between renders).
- Both help prevent unnecessary work and re-renders, but they serve different purposes:

| Hook        | Use for...                | Example Use Case                        |
|-------------|--------------------------|-----------------------------------------|
| useMemo     | Expensive values         | Filtered/sorted lists, derived objects  |
| useCallback | Stable function reference | Passing handlers to memoized children   |

> **Tip:** If you find yourself using `useMemo` to memoize a function, you should use `useCallback` instead.

---

## Key Takeaways
- Use `useMemo` for expensive calculations or to memoize objects/arrays passed to memoized children.
- Use `useCallback` for memoizing functions.
- Avoids unnecessary recalculations and re-renders, especially in large or complex UIs.
- Only use when you have a real performance bottleneck—overusing these hooks can add complexity. 