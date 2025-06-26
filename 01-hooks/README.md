# React Hooks - Senior Level Interview Questions

## 1. Custom Hook Architecture and Performance Optimization

**Q: Design a custom hook for managing complex form state with validation, and explain how you would optimize it for performance in a large-scale application.**

### Answer
Custom hooks should be modular, reusable, and optimized for performance. Use `useCallback` and `useMemo` to avoid unnecessary re-renders. Example:

```jsx
import { useState, useCallback, useMemo } from 'react';

function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues((prev) => ({ ...prev, [name]: value }));
    setErrors((prev) => ({ ...prev, [name]: validate(name, value) }));
  }, [validate]);

  const isValid = useMemo(() => Object.values(errors).every((e) => !e), [errors]);

  return { values, errors, handleChange, isValid };
}
```

**Performance Tips:**
- Use `useCallback` for handlers
- Use `useMemo` for derived state
- Validate only changed fields

---

## 2. useEffect Dependencies and Memory Leaks

**Q: Explain the useEffect dependency array, common pitfalls, and how to handle memory leaks in React applications. Provide a real-world example.**

### Answer
The dependency array tells React when to re-run the effect. Omitting dependencies can cause bugs; including unstable references can cause infinite loops. Always clean up side effects to avoid memory leaks.

```jsx
import { useEffect, useState } from 'react';

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let isMounted = true;
    fetch(url)
      .then((res) => res.json())
      .then((d) => { if (isMounted) setData(d); });
    return () => { isMounted = false; };
  }, [url]);
  return data;
}
```

**Pitfalls:**
- Missing dependencies: can cause stale data
- Unstable functions/objects: can cause infinite loops
- Not cleaning up: can cause memory leaks (e.g., subscriptions, timers)

---

## 3. Advanced Hook Composition and State Synchronization

**Q: Design a hook system for managing complex state synchronization across multiple components, including real-time updates and optimistic updates. Explain the architecture decisions.**

### Answer
Use context and custom hooks for shared state. For real-time and optimistic updates, use a combination of local state, context, and effects.

```jsx
import React, { createContext, useContext, useState, useCallback } from 'react';

const DataContext = createContext();

export function DataProvider({ children }) {
  const [data, setData] = useState([]);

  const updateData = useCallback((newItem) => {
    setData((prev) => [...prev, newItem]); // Optimistic update
    // Simulate server sync
    setTimeout(() => {
      // If server fails, rollback (not shown)
    }, 1000);
  }, []);

  return (
    <DataContext.Provider value={{ data, updateData }}>
      {children}
    </DataContext.Provider>
  );
}

export function useData() {
  return useContext(DataContext);
}
```

**Architecture Decisions:**
- Use Context for global state
- Use custom hooks for encapsulation
- Optimistic updates for better UX
- Rollback on server error if needed 