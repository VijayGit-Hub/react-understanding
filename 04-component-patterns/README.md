# React Component Patterns - Senior Level Interview Questions

## 1. Higher-Order Components (HOCs) vs Render Props vs Custom Hooks

**Q: Compare and contrast HOCs, Render Props, and Custom Hooks. Provide implementation examples for each pattern and explain when you would choose one over the others in a large-scale application.**

### Answer
**HOCs** wrap components to add functionality. **Render Props** pass functions as props. **Custom Hooks** extract stateful logic. Each has different use cases and trade-offs.

**Higher-Order Component Example:**
```jsx
import React from 'react';

// HOC for authentication
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = React.useState(false);
    const [user, setUser] = React.useState(null);

    React.useEffect(() => {
      // Check authentication status
      checkAuthStatus().then(authData => {
        setIsAuthenticated(authData.isAuthenticated);
        setUser(authData.user);
      });
    }, []);

    if (!isAuthenticated) {
      return <LoginPage />;
    }

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
```

**Render Props Pattern:**
```jsx
import React from 'react';

// Render prop component for data fetching
class DataFetcher extends React.Component {
  state = {
    data: null,
    loading: true,
    error: null
  };

  componentDidMount() {
    this.fetchData();
  }

  fetchData = async () => {
    try {
      const response = await fetch(this.props.url);
      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  render() {
    return this.props.children(this.state);
  }
}

// Usage
function UserList() {
  return (
    <DataFetcher url="/api/users">
      {({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        return (
          <ul>
            {data.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        );
      }}
    </DataFetcher>
  );
}
```

**Custom Hook Example:**
```jsx
import { useState, useEffect } from 'react';

// Custom hook for data fetching
function useDataFetcher(url) {
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
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data, loading, error } = useDataFetcher('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**When to Use Each Pattern:**
- **HOCs**: Cross-cutting concerns, component composition
- **Render Props**: Dynamic rendering logic, complex state sharing
- **Custom Hooks**: Reusable stateful logic, simpler than HOCs

---

## 2. Compound Components Pattern

**Q: Implement a compound components pattern for a complex form system. Explain the benefits of this pattern and demonstrate how to handle state management and validation across multiple related components.**

### Answer
Compound components share implicit state and provide flexible composition. They're great for complex UI components that need to work together.

**Form System Implementation:**
```jsx
import React, { createContext, useContext, useState } from 'react';

const FormContext = createContext();

// Main form component
function Form({ children, onSubmit, initialValues = {} }) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const setValue = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const setError = (name, error) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  };

  const setFieldTouched = (name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const validate = () => {
    const newErrors = {};
    // Validation logic here
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      onSubmit(values);
    }
  };

  const contextValue = {
    values,
    errors,
    touched,
    setValue,
    setError,
    setFieldTouched
  };

  return (
    <FormContext.Provider value={contextValue}>
      <form onSubmit={handleSubmit}>
        {children}
      </form>
    </FormContext.Provider>
  );
}

// Form field component
function FormField({ name, children }) {
  const { values, errors, touched, setValue, setError, setFieldTouched } = useContext(FormContext);
  
  const value = values[name] || '';
  const error = errors[name];
  const isTouched = touched[name];

  const handleChange = (e) => {
    setValue(name, e.target.value);
    if (error) {
      setError(name, null);
    }
  };

  const handleBlur = () => {
    setFieldTouched(name);
  };

  return children({
    value,
    error: isTouched ? error : null,
    onChange: handleChange,
    onBlur: handleBlur
  });
}

// Input component
function Input({ name, ...props }) {
  return (
    <FormField name={name}>
      {({ value, error, onChange, onBlur }) => (
        <div>
          <input
            {...props}
            value={value}
            onChange={onChange}
            onBlur={onBlur}
            className={error ? 'error' : ''}
          />
          {error && <span className="error-message">{error}</span>}
        </div>
      )}
    </FormField>
  );
}

// Submit button component
function SubmitButton({ children }) {
  const { errors } = useContext(FormContext);
  const hasErrors = Object.keys(errors).length > 0;

  return (
    <button type="submit" disabled={hasErrors}>
      {children}
    </button>
  );
}

// Usage
function UserForm() {
  const handleSubmit = (values) => {
    console.log('Form submitted:', values);
  };

  return (
    <Form onSubmit={handleSubmit} initialValues={{ name: '', email: '' }}>
      <Input name="name" placeholder="Name" />
      <Input name="email" type="email" placeholder="Email" />
      <SubmitButton>Submit</SubmitButton>
    </Form>
  );
}
```

**Benefits of Compound Components:**
- Flexible composition
- Implicit state sharing
- Better API design
- Reusable components

---

## 3. Advanced Component Composition: Render Props with TypeScript

**Q: Design a flexible component composition system using render props and TypeScript. Implement a solution that provides type safety while maintaining flexibility for different use cases.**

### Answer
Use TypeScript generics and render props to create type-safe, flexible components that can handle various data types and rendering scenarios.

**Type-Safe Render Props Implementation:**
```tsx
import React from 'react';

// Generic types for different data structures
interface User {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: string;
  title: string;
  price: number;
}

// Generic data fetcher with render props
interface DataFetcherProps<T> {
  url: string;
  children: (state: DataState<T>) => React.ReactNode;
  transform?: (data: any) => T;
}

interface DataState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function DataFetcher<T>({ url, children, transform }: DataFetcherProps<T>) {
  const [state, setState] = React.useState<DataState<T>>({
    data: null,
    loading: true,
    error: null,
    refetch: () => {}
  });

  const fetchData = React.useCallback(async () => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      const response = await fetch(url);
      const rawData = await response.json();
      const data = transform ? transform(rawData) : rawData;
      setState(prev => ({ ...prev, data, loading: false }));
    } catch (error) {
      setState(prev => ({ 
        ...prev, 
        error: error instanceof Error ? error : new Error('Unknown error'),
        loading: false 
      }));
    }
  }, [url, transform]);

  React.useEffect(() => {
    fetchData();
  }, [fetchData]);

  const refetch = React.useCallback(() => {
    fetchData();
  }, [fetchData]);

  return <>{children({ ...state, refetch })}</>;
}

// Type-safe list component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyMessage?: string;
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage }: ListProps<T>) {
  if (items.length === 0) {
    return <div>{emptyMessage || 'No items found'}</div>;
  }

  return (
    <div>
      {items.map((item, index) => (
        <div key={keyExtractor(item)}>
          {renderItem(item, index)}
        </div>
      ))}
    </div>
  );
}

// Usage with type safety
function UserManagement() {
  return (
    <DataFetcher<User[]>
      url="/api/users"
      transform={(data: any[]) => data.map(user => ({
        id: user.id,
        name: user.name,
        email: user.email
      }))}
    >
      {({ data, loading, error, refetch }) => {
        if (loading) return <div>Loading users...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No data</div>;

        return (
          <div>
            <button onClick={refetch}>Refresh</button>
            <List
              items={data}
              keyExtractor={(user: User) => user.id}
              renderItem={(user: User) => (
                <div>
                  <h3>{user.name}</h3>
                  <p>{user.email}</p>
                </div>
              )}
            />
          </div>
        );
      }}
    </DataFetcher>
  );
}

// Product management with different data type
function ProductManagement() {
  return (
    <DataFetcher<Product[]>
      url="/api/products"
      transform={(data: any[]) => data.map(product => ({
        id: product.id,
        title: product.title,
        price: product.price
      }))}
    >
      {({ data, loading, error }) => {
        if (loading) return <div>Loading products...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No data</div>;

        return (
          <List
            items={data}
            keyExtractor={(product: Product) => product.id}
            renderItem={(product: Product) => (
              <div>
                <h3>{product.title}</h3>
                <p>${product.price}</p>
              </div>
            )}
          />
        );
      }}
    </DataFetcher>
  );
}
```

**Advanced Composition with Higher-Order Components:**
```tsx
// HOC that adds loading states
function withLoading<T extends object>(
  WrappedComponent: React.ComponentType<T>
) {
  return function WithLoadingComponent(props: T & { loading?: boolean }) {
    const { loading, ...restProps } = props;
    
    if (loading) {
      return <div>Loading...</div>;
    }
    
    return <WrappedComponent {...(restProps as T)} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);
```

**Type Safety Benefits:**
- Compile-time error checking
- IntelliSense support
- Refactoring safety
- Better developer experience 