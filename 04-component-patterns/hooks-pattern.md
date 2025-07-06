# React Hooks Pattern: Modern State & Logic Sharing

Hooks revolutionized React by enabling state, side effects, and code sharing in function components—without classes or wrapper hell. This pattern is now the standard for building scalable, maintainable React apps.

---

## What is the Hooks Pattern?

Hooks are special functions (like `useState`, `useEffect`, `useContext`, and `useReducer`) that let you "hook into" React features from function components. They:
- Add state and lifecycle logic to functions (no classes needed)
- Enable code reuse and separation of concerns via custom hooks
- Replace patterns like HOC and render props for logic sharing

---

## The Problem with Class Components

Before hooks, class components were required for state and lifecycle logic, but they had several issues:

### 1. Verbose and Complex
```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
  }
  
  componentDidMount() {
    document.title = `Count: ${this.state.count}`;
  }
  
  componentDidUpdate() {
    document.title = `Count: ${this.state.count}`;
  }
  
  increment() {
    this.setState(prevState => ({ count: prevState.count + 1 }));
  }
  
  decrement() {
    this.setState(prevState => ({ count: prevState.count - 1 }));
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>+</button>
        <button onClick={this.decrement}>-</button>
      </div>
    );
  }
}
```

### 2. Wrapper Hell
Sharing logic between components led to deeply nested HOCs:
```jsx
const EnhancedComponent = withAuth(
  withLoading(
    withErrorBoundary(
      withAnalytics(MyComponent)
    )
  )
);
```

### 3. Tangled Logic
Unrelated concerns mixed together in lifecycle methods:
```jsx
class App extends React.Component {
  constructor() {
    super();
    this.state = {
      count: 0,
      width: window.innerWidth
    };
  }
  
  componentDidMount() {
    // Counter logic
    this.timer = setInterval(() => {
      this.setState(prev => ({ count: prev.count + 1 }));
    }, 1000);
    
    // Window resize logic
    window.addEventListener('resize', this.handleResize);
  }
  
  componentWillUnmount() {
    // Counter cleanup
    clearInterval(this.timer);
    
    // Window resize cleanup
    window.removeEventListener('resize', this.handleResize);
  }
  
  handleResize = () => {
    this.setState({ width: window.innerWidth });
  };
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <p>Width: {this.state.width}</p>
      </div>
    );
  }
}
```

---

## Hooks: The Solution

Hooks solve these problems by enabling function components to have state and side effects:

### Simple State with useState
```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

### Side Effects with useEffect
```jsx
import React, { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]); // Only run when count changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+</button>
    </div>
  );
}
```

---

## Custom Hooks: Logic Reuse

Custom hooks let you extract and share stateful logic:

### useCounter Hook
```jsx
import { useState } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}

// Usage in components
function Counter() {
  const { count, increment, decrement } = useCounter();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}

function Timer() {
  const { count, increment, reset } = useCounter(10);
  
  useEffect(() => {
    const timer = setInterval(increment, 1000);
    return () => clearInterval(timer);
  }, [increment]);
  
  return (
    <div>
      <p>Timer: {count}</p>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### useWindowSize Hook
```jsx
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Usage
function ResponsiveComponent() {
  const { width, height } = useWindowSize();
  
  return (
    <div>
      <p>Window size: {width} x {height}</p>
      {width < 768 && <p>Mobile view</p>}
    </div>
  );
}
```

---

## Solving Complex Logic with Hooks

### Separated Concerns
```jsx
import React, { useState, useEffect } from 'react';

// Custom hook for counter logic
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return { count, setCount };
}

// Custom hook for window size logic
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Clean component using both hooks
function App() {
  const { count, setCount } = useCounter();
  const { width, height } = useWindowSize();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+</button>
      <p>Window: {width} x {height}</p>
    </div>
  );
}
```

---

## Main React Hooks: Quick Guide

| Hook | Purpose | Example Use Case |
|------|---------|------------------|
| `useState` | Local state in function components | Toggle, form input, counter |
| `useEffect` | Side effects (fetch, subscriptions) | Data fetch, event listeners |
| `useContext` | Access context value | Theme, auth, i18n |
| `useReducer` | Complex state logic, like Redux | Form state, undo/redo |
| `useCallback` | Memoize functions | Prevent unnecessary re-renders |
| `useMemo` | Memoize values | Expensive calculations |
| `useRef` | Persistent mutable values | DOM access, timers |

---

## Benefits of Hooks Pattern

### 1. Less Code
**Class Component:**
```jsx
class TweetSearchResults extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      filterText: "",
      inThisLocation: false,
    };
    this.handleFilterTextChange = this.handleFilterTextChange.bind(this);
    this.handleInThisLocationChange = this.handleInThisLocationChange.bind(this);
  }

  handleFilterTextChange(filterText) {
    this.setState({ filterText });
  }

  handleInThisLocationChange(inThisLocation) {
    this.setState({ inThisLocation });
  }

  render() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inThisLocation={this.state.inThisLocation}
          onFilterTextChange={this.handleFilterTextChange}
          onInThisLocationChange={this.handleInThisLocationChange}
        />
        <TweetList
          tweets={this.props.tweets}
          filterText={this.state.filterText}
          inThisLocation={this.state.inThisLocation}
        />
      </div>
    );
  }
}
```

**Hooks Version:**
```jsx
const TweetSearchResults = ({ tweets }) => {
  const [filterText, setFilterText] = useState("");
  const [inThisLocation, setInThisLocation] = useState(false);
  
  return (
    <div>
      <SearchBar
        filterText={filterText}
        inThisLocation={inThisLocation}
        setFilterText={setFilterText}
        setInThisLocation={setInThisLocation}
      />
      <TweetList
        tweets={tweets}
        filterText={filterText}
        inThisLocation={inThisLocation}
      />
    </div>
  );
};
```

### 2. Better Logic Reuse
- Custom hooks can be shared across components
- No wrapper hell or prop drilling
- Logic is composable and testable

### 3. Easier Testing
- Test hooks in isolation
- No need to mock class instances
- Simpler unit tests

---

## Hooks vs Classes: Detailed Comparison

| Feature | Hooks (Function) | Classes |
|---------|------------------|---------|
| **State** | `useState` | `this.state` |
| **Side Effects** | `useEffect` | `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` |
| **Logic Sharing** | Custom hooks | HOC, render props |
| **Code Reuse** | Easy, composable | Hard, wrapper hell |
| **Boilerplate** | Minimal | High (constructor, binding, etc.) |
| **Learning Curve** | Rules of hooks | Classes, binding, `this` |
| **Performance** | Better optimization | Can cause unnecessary re-renders |
| **Testing** | Easier to test | More complex |
| **TypeScript** | Better support | More verbose |

---

## Best Practices

1. **Use Hooks for All New Code**: Prefer function components with hooks
2. **Extract Logic into Custom Hooks**: Keep components focused on UI
3. **Follow Rules of Hooks**: Only call hooks at top level, in functions
4. **Use Appropriate Hooks**: Choose the right hook for the job
5. **Optimize with useCallback/useMemo**: When needed for performance

---

## Common Pitfalls

1. **Breaking Rules of Hooks**: Calling hooks conditionally or in loops
2. **Overusing useCallback/useMemo**: Only use when necessary
3. **Not Cleaning Up Effects**: Always return cleanup function from useEffect
4. **Missing Dependencies**: Include all dependencies in useEffect array

---

## Example: Complete App with Hooks

```jsx
import React, { useState, useEffect } from 'react';

// Custom hook for API calls
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}

// Custom hook for form handling
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  
  const handleChange = (field, value) => {
    setValues(prev => ({ ...prev, [field]: value }));
  };
  
  const reset = () => setValues(initialValues);
  
  return { values, handleChange, reset };
}

// Main component using both hooks
function UserManagement() {
  const { data: users, loading, error } = useApi('/api/users');
  const { values: formData, handleChange, reset } = useForm({
    name: '',
    email: ''
  });
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users?.map(user => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
      
      <form>
        <input
          value={formData.name}
          onChange={(e) => handleChange('name', e.target.value)}
          placeholder="Name"
        />
        <input
          value={formData.email}
          onChange={(e) => handleChange('email', e.target.value)}
          placeholder="Email"
        />
        <button type="button" onClick={reset}>Reset</button>
      </form>
    </div>
  );
}
```

---

**Reference:** [Patterns.dev: Hooks Pattern](https://www.patterns.dev/react/hooks-pattern) 