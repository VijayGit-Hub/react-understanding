# Render Props Pattern: Dynamic Component Logic

The Render Props pattern allows you to share component logic by passing a function as a prop that returns React elements. This pattern provides great flexibility and reusability for complex component interactions.

---

## What is the Render Props Pattern?

A render prop is a function prop that a component uses to know what to render. Instead of hardcoding what a component renders, you pass a function that returns JSX, giving you complete control over the rendering logic.

**Key Concept:** The component provides data/logic, and the render function determines how to display it.

---

## Basic Render Props Structure

```jsx
// Component with render prop
function DataProvider({ render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Fetch data logic
    fetchData().then(setData).finally(() => setLoading(false));
  }, []);
  
  // Call the render function with data
  return render({ data, loading });
}

// Usage
function App() {
  return (
    <DataProvider
      render={({ data, loading }) => {
        if (loading) return <div>Loading...</div>;
        return <div>{data?.message}</div>;
      }}
    />
  );
}
```

---

## Example: Mouse Tracker with Render Props

```jsx
import React, { useState, useEffect } from 'react';

// Component that tracks mouse position
function MouseTracker({ render }) {
  const [mousePosition, setMousePosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (event) => {
      setMousePosition({
        x: event.clientX,
        y: event.clientY
      });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);
  
  // Pass mouse position to render function
  return render(mousePosition);
}

// Usage: Different ways to use the mouse data
function App() {
  return (
    <div>
      {/* Display coordinates */}
      <MouseTracker
        render={(mouse) => (
          <div>
            Mouse position: {mouse.x}, {mouse.y}
          </div>
        )}
      />
      
      {/* Create a following element */}
      <MouseTracker
        render={(mouse) => (
          <div
            style={{
              position: 'fixed',
              left: mouse.x,
              top: mouse.y,
              width: '20px',
              height: '20px',
              backgroundColor: 'red',
              borderRadius: '50%',
              pointerEvents: 'none'
            }}
          />
        )}
      />
    </div>
  );
}
```

---

## Example: Data Fetching with Render Props

```jsx
import React, { useState, useEffect } from 'react';

// Generic data fetcher component
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
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
  
  return render({ data, loading, error });
}

// Usage: Different ways to display the data
function App() {
  return (
    <div>
      {/* Display as list */}
      <DataFetcher
        url="/api/users"
        render={({ data, loading, error }) => {
          if (loading) return <div>Loading users...</div>;
          if (error) return <div>Error: {error}</div>;
          
          return (
            <ul>
              {data?.map(user => (
                <li key={user.id}>{user.name}</li>
              ))}
            </ul>
          );
        }}
      />
      
      {/* Display as table */}
      <DataFetcher
        url="/api/users"
        render={({ data, loading, error }) => {
          if (loading) return <div>Loading...</div>;
          if (error) return <div>Error: {error}</div>;
          
          return (
            <table>
              <thead>
                <tr>
                  <th>Name</th>
                  <th>Email</th>
                </tr>
              </thead>
              <tbody>
                {data?.map(user => (
                  <tr key={user.id}>
                    <td>{user.name}</td>
                    <td>{user.email}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          );
        }}
      />
    </div>
  );
}
```

---

## Example: Form Validation with Render Props

```jsx
import React, { useState } from 'react';

// Form validator component
function FormValidator({ initialValues, validationRules, render }) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const validate = (fieldValues = values) => {
    const newErrors = {};
    
    Object.keys(validationRules).forEach(field => {
      const value = fieldValues[field];
      const rules = validationRules[field];
      
      if (rules.required && !value) {
        newErrors[field] = `${field} is required`;
      } else if (rules.pattern && !rules.pattern.test(value)) {
        newErrors[field] = rules.message || `${field} is invalid`;
      }
    });
    
    return newErrors;
  };
  
  const handleChange = (field, value) => {
    const newValues = { ...values, [field]: value };
    setValues(newValues);
    
    if (touched[field]) {
      const newErrors = validate(newValues);
      setErrors(newErrors);
    }
  };
  
  const handleBlur = (field) => {
    setTouched({ ...touched, [field]: true });
    const newErrors = validate();
    setErrors(newErrors);
  };
  
  const handleSubmit = (onSubmit) => {
    const newErrors = validate();
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      onSubmit(values);
    }
  };
  
  return render({
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit
  });
}

// Usage
function LoginForm() {
  const validationRules = {
    email: {
      required: true,
      pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      message: 'Please enter a valid email'
    },
    password: {
      required: true,
      pattern: /.{6,}/,
      message: 'Password must be at least 6 characters'
    }
  };
  
  return (
    <FormValidator
      initialValues={{ email: '', password: '' }}
      validationRules={validationRules}
      render={({ values, errors, touched, handleChange, handleBlur, handleSubmit }) => (
        <form onSubmit={(e) => {
          e.preventDefault();
          handleSubmit((formData) => {
            console.log('Form submitted:', formData);
          });
        }}>
          <div>
            <label>Email:</label>
            <input
              type="email"
              value={values.email}
              onChange={(e) => handleChange('email', e.target.value)}
              onBlur={() => handleBlur('email')}
            />
            {touched.email && errors.email && (
              <span className="error">{errors.email}</span>
            )}
          </div>
          
          <div>
            <label>Password:</label>
            <input
              type="password"
              value={values.password}
              onChange={(e) => handleChange('password', e.target.value)}
              onBlur={() => handleBlur('password')}
            />
            {touched.password && errors.password && (
              <span className="error">{errors.password}</span>
            )}
          </div>
          
          <button type="submit">Login</button>
        </form>
      )}
    />
  );
}
```

---

## Benefits of Render Props Pattern

1. **Flexibility**: Complete control over how data is rendered
2. **Reusability**: Logic can be shared across different UI implementations
3. **Composability**: Easy to combine multiple render prop components
4. **Type Safety**: Better TypeScript support than HOCs
5. **No Wrapper Hell**: Avoids the nesting issues of HOCs

---

## Common Render Props Patterns

### 1. Children as Function
```jsx
function MouseTracker({ children }) {
  const [mouse, setMouse] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => setMouse({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return children(mouse);
}

// Usage
<MouseTracker>
  {(mouse) => <div>Mouse: {mouse.x}, {mouse.y}</div>}
</MouseTracker>
```

### 2. Render Prop with Multiple Parameters
```jsx
function DataProvider({ render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  return render({ data, loading, error, refetch: () => {/* refetch logic */} });
}
```

### 3. Conditional Rendering
```jsx
function ConditionalRenderer({ condition, render }) {
  if (!condition) return null;
  return render();
}
```

---

## Best Practices

1. **Clear Naming**: Use descriptive names for render props (e.g., `render`, `children`, `renderItem`)
2. **Type Safety**: Define proper TypeScript interfaces for render functions
3. **Performance**: Memoize render functions if they're expensive
4. **Composition**: Combine render props with other patterns when needed
5. **Documentation**: Clearly document what data the render function receives

---

## Render Props vs Other Patterns

| Pattern | Pros | Cons | Best For |
|---------|------|------|----------|
| **Render Props** | Flexible, composable, type-safe | Can be verbose | Dynamic rendering, complex logic |
| **HOCs** | Familiar, good for cross-cutting concerns | Wrapper hell, prop collisions | Simple enhancements |
| **Hooks** | Simple, no wrapper components | Less flexible for complex rendering | Stateful logic, simple cases |

---

## Example: Combining Render Props with Hooks

```jsx
// Custom hook for mouse tracking
function useMouse() {
  const [mouse, setMouse] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => setMouse({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return mouse;
}

// Component using both hook and render prop
function MouseTracker({ render }) {
  const mouse = useMouse();
  return render(mouse);
}

// Usage
function App() {
  return (
    <MouseTracker
      render={(mouse) => (
        <div>
          Mouse position: {mouse.x}, {mouse.y}
        </div>
      )}
    />
  );
}
```

---

**Reference:** [Patterns.dev: Render Props Pattern](https://www.patterns.dev/react/render-props-pattern) 