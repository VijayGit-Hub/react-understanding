# Higher-Order Component (HOC) Pattern: Component Transformation

A Higher-Order Component (HOC) is a function that takes a component and returns a new component with enhanced functionality. HOCs are a powerful way to reuse component logic across your React application.

---

## What is a Higher-Order Component?

An HOC is a function that:
- Takes a component as an argument
- Returns a new component with additional props or behavior
- Follows the pattern: `HOC(Component) => EnhancedComponent`

**Key Concept:** While a component transforms props into UI, an HOC transforms a component into another component.

---

## Basic HOC Structure

```jsx
// HOC function
function withEnhancement(WrappedComponent) {
  // Return a new component
  return function EnhancedComponent(props) {
    // Add new logic, state, or props
    const enhancedProps = {
      ...props,
      enhanced: true,
      message: "This component is enhanced!"
    };
    
    // Render the original component with enhanced props
    return <WrappedComponent {...enhancedProps} />;
  };
}

// Usage
const EnhancedButton = withEnhancement(Button);
```

---

## Example: withLoading HOC

```jsx
import React, { useState, useEffect } from 'react';

// HOC that adds loading state to any component
function withLoading(WrappedComponent) {
  return function WithLoadingComponent(props) {
    const [loading, setLoading] = useState(true);
    const [data, setData] = useState(null);
    
    useEffect(() => {
      // Simulate API call
      const fetchData = async () => {
        setLoading(true);
        try {
          // Simulate API delay
          await new Promise(resolve => setTimeout(resolve, 2000));
          setData({ message: "Data loaded successfully!" });
        } catch (error) {
          setData({ error: "Failed to load data" });
        } finally {
          setLoading(false);
        }
      };
      
      fetchData();
    }, []);
    
    if (loading) {
      return <div>Loading...</div>;
    }
    
    return <WrappedComponent {...props} data={data} />;
  };
}

// Original component
function UserProfile({ data, name }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{data?.message || data?.error}</p>
    </div>
  );
}

// Enhanced component
const UserProfileWithLoading = withLoading(UserProfile);

// Usage
function App() {
  return <UserProfileWithLoading name="John Doe" />;
}
```

---

## Example: withAuth HOC

```jsx
import React, { useContext } from 'react';

// Mock auth context
const AuthContext = React.createContext();

function withAuth(WrappedComponent) {
  return function WithAuthComponent(props) {
    const { user, login, logout } = useContext(AuthContext);
    
    if (!user) {
      return (
        <div>
          <p>Please log in to continue</p>
          <button onClick={login}>Login</button>
        </div>
      );
    }
    
    // Pass auth-related props to the wrapped component
    const enhancedProps = {
      ...props,
      user,
      logout
    };
    
    return <WrappedComponent {...enhancedProps} />;
  };
}

// Original component
function Dashboard({ user, logout }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Enhanced component
const ProtectedDashboard = withAuth(Dashboard);
```

---

## Example: withErrorBoundary HOC

```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong!</h2>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }
    
    return this.props.children;
  }
}

function withErrorBoundary(WrappedComponent) {
  return function WithErrorBoundaryComponent(props) {
    return (
      <ErrorBoundary>
        <WrappedComponent {...props} />
      </ErrorBoundary>
    );
  };
}

// Usage
const SafeComponent = withErrorBoundary(MyComponent);
```

---

## Benefits of HOCs

1. **Code Reuse**: Share logic across multiple components
2. **Separation of Concerns**: Keep components focused on UI, HOCs handle logic
3. **Composability**: Combine multiple HOCs for complex enhancements
4. **Abstraction**: Hide complex logic from component implementations

---

## Common HOC Patterns

### 1. Props Proxy
```jsx
function withProps(WrappedComponent) {
  return function EnhancedComponent(props) {
    const newProps = {
      ...props,
      additionalProp: "value"
    };
    return <WrappedComponent {...newProps} />;
  };
}
```

### 2. Inheritance Inversion
```jsx
function withInheritance(WrappedComponent) {
  return class EnhancedComponent extends WrappedComponent {
    render() {
      return (
        <div>
          <p>Before</p>
          {super.render()}
          <p>After</p>
        </div>
      );
    }
  };
}
```

### 3. Display Name for Debugging
```jsx
function withDisplayName(WrappedComponent) {
  const EnhancedComponent = (props) => {
    return <WrappedComponent {...props} />;
  };
  
  EnhancedComponent.displayName = `withDisplayName(${WrappedComponent.displayName || WrappedComponent.name})`;
  return EnhancedComponent;
}
```

---

## Best Practices

1. **Don't Mutate the Original Component**: Always return a new component
2. **Pass Through Unrelated Props**: Use spread operator to pass all props
3. **Set Display Name**: Help with debugging in React DevTools
4. **Compose HOCs**: Use utility functions like `compose` for multiple HOCs
5. **Keep HOCs Pure**: Avoid side effects in HOC functions

---

## HOCs vs Hooks

| Aspect | HOCs | Hooks |
|--------|------|-------|
| **Learning Curve** | Steeper (classes, this) | Easier (functions) |
| **Wrapper Hell** | Can create deep nesting | Flat composition |
| **Logic Reuse** | Component-level | Function-level |
| **Performance** | Can cause unnecessary re-renders | Better optimization |
| **Testing** | More complex | Easier to test |

---

## Common Pitfalls

1. **Forgetting to Pass Props**: Always spread props to wrapped component
2. **Naming Conflicts**: Be careful with prop name collisions
3. **Static Methods**: HOCs don't automatically copy static methods
4. **Refs**: Need special handling for refs (use `forwardRef`)

---

## Example: Composing Multiple HOCs

```jsx
import { compose } from 'lodash/fp';

const EnhancedComponent = compose(
  withLoading,
  withAuth,
  withErrorBoundary
)(MyComponent);

// Equivalent to:
// const EnhancedComponent = withErrorBoundary(withAuth(withLoading(MyComponent)));
```

---

**Reference:** [Patterns.dev: HOC Pattern](https://www.patterns.dev/react/hoc-pattern) 