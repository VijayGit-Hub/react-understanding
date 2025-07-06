# Container/Presentational Pattern: Separation of Concerns

The Container/Presentational pattern (also known as Smart/Dumb components) separates components into two categories: those that handle data and logic (containers) and those that handle UI rendering (presentational).

---

## What is the Container/Presentational Pattern?

This pattern divides React components into two distinct responsibilities:

**Container Components (Smart):**
- Handle data fetching, state management, and business logic
- Pass data and callbacks to presentational components
- Rarely contain UI markup
- Often use lifecycle methods and state

**Presentational Components (Dumb):**
- Focus purely on UI rendering
- Receive data and callbacks via props
- Don't manage state or side effects
- Highly reusable and testable

---

## Example: User List Component

### Presentational Component (Dumb)
```jsx
// UserList.js - Presentational Component
import React from 'react';

function UserList({ users, isLoading, onUserClick, onDeleteUser }) {
  if (isLoading) {
    return <div>Loading users...</div>;
  }
  
  if (!users || users.length === 0) {
    return <div>No users found.</div>;
  }
  
  return (
    <div className="user-list">
      <h2>Users</h2>
      {users.map(user => (
        <div key={user.id} className="user-item">
          <span onClick={() => onUserClick(user)}>
            {user.name} - {user.email}
          </span>
          <button onClick={() => onDeleteUser(user.id)}>
            Delete
          </button>
        </div>
      ))}
    </div>
  );
}

export default UserList;
```

### Container Component (Smart)
```jsx
// UserListContainer.js - Container Component
import React, { useState, useEffect } from 'react';
import UserList from './UserList';

function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  const fetchUsers = async () => {
    try {
      setIsLoading(true);
      // Simulate API call
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  const handleUserClick = (user) => {
    console.log('User clicked:', user);
    // Navigate to user detail page
  };
  
  const handleDeleteUser = async (userId) => {
    try {
      await fetch(`/api/users/${userId}`, { method: 'DELETE' });
      setUsers(users.filter(user => user.id !== userId));
    } catch (err) {
      console.error('Failed to delete user:', err);
    }
  };
  
  if (error) {
    return <div>Error: {error}</div>;
  }
  
  return (
    <UserList
      users={users}
      isLoading={isLoading}
      onUserClick={handleUserClick}
      onDeleteUser={handleDeleteUser}
    />
  );
}

export default UserListContainer;
```

---

## Example: Search Component

### Presentational Component
```jsx
// SearchBox.js - Presentational Component
import React from 'react';

function SearchBox({ 
  searchTerm, 
  onSearchChange, 
  onSearchSubmit, 
  placeholder = "Search..." 
}) {
  const handleSubmit = (e) => {
    e.preventDefault();
    onSearchSubmit(searchTerm);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => onSearchChange(e.target.value)}
        placeholder={placeholder}
      />
      <button type="submit">Search</button>
    </form>
  );
}

export default SearchBox;
```

### Container Component
```jsx
// SearchContainer.js - Container Component
import React, { useState, useCallback } from 'react';
import SearchBox from './SearchBox';

function SearchContainer({ onSearch }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [searchHistory, setSearchHistory] = useState([]);
  
  const handleSearchChange = useCallback((value) => {
    setSearchTerm(value);
  }, []);
  
  const handleSearchSubmit = useCallback((term) => {
    // Add to search history
    setSearchHistory(prev => [...prev, term]);
    
    // Call parent's search function
    onSearch(term);
  }, [onSearch]);
  
  return (
    <div>
      <SearchBox
        searchTerm={searchTerm}
        onSearchChange={handleSearchChange}
        onSearchSubmit={handleSearchSubmit}
        placeholder="Search for anything..."
      />
      {searchHistory.length > 0 && (
        <div>
          <h4>Recent Searches:</h4>
          <ul>
            {searchHistory.slice(-5).map((term, index) => (
              <li key={index}>{term}</li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}

export default SearchContainer;
```

---

## Benefits of Container/Presentational Pattern

1. **Separation of Concerns**: Clear distinction between logic and UI
2. **Reusability**: Presentational components can be reused with different data sources
3. **Testability**: Easy to test presentational components in isolation
4. **Maintainability**: Changes to logic don't affect UI and vice versa
5. **Team Collaboration**: Different team members can work on containers vs presentational components

---

## When to Use This Pattern

- **Complex Applications**: When you have significant business logic
- **Large Teams**: When different people work on logic vs UI
- **Reusable UI Components**: When you want to create a component library
- **Testing Focus**: When you need to test UI and logic separately

---

## Best Practices

1. **Single Responsibility**: Each component should have one clear purpose
2. **Props Interface**: Define clear prop interfaces for presentational components
3. **No Business Logic in Presentational**: Keep UI components pure
4. **Consistent Naming**: Use clear naming conventions (e.g., `UserList` vs `UserListContainer`)
5. **Composition**: Use composition to combine containers and presentational components

---

## Example: Form Pattern

### Presentational Form
```jsx
// UserForm.js - Presentational Component
import React from 'react';

function UserForm({ 
  user, 
  onSubmit, 
  onChange, 
  errors = {}, 
  isSubmitting = false 
}) {
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(user);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          type="text"
          value={user.name || ''}
          onChange={(e) => onChange({ ...user, name: e.target.value })}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          value={user.email || ''}
          onChange={(e) => onChange({ ...user, email: e.target.value })}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Saving...' : 'Save User'}
      </button>
    </form>
  );
}

export default UserForm;
```

### Container Form
```jsx
// UserFormContainer.js - Container Component
import React, { useState } from 'react';
import UserForm from './UserForm';

function UserFormContainer({ initialUser = {}, onSave }) {
  const [user, setUser] = useState(initialUser);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const validateUser = (userData) => {
    const newErrors = {};
    
    if (!userData.name?.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!userData.email?.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(userData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    return newErrors;
  };
  
  const handleSubmit = async (userData) => {
    const validationErrors = validateUser(userData);
    
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    setIsSubmitting(true);
    try {
      await onSave(userData);
      setErrors({});
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <UserForm
      user={user}
      onSubmit={handleSubmit}
      onChange={setUser}
      errors={errors}
      isSubmitting={isSubmitting}
    />
  );
}

export default UserFormContainer;
```

---

## Comparison with Other Patterns

| Pattern | Use Case | Complexity | Reusability |
|---------|----------|------------|-------------|
| **Container/Presentational** | Logic/UI separation | Medium | High |
| **HOCs** | Cross-cutting concerns | High | Medium |
| **Render Props** | Dynamic rendering | Medium | High |
| **Hooks** | Stateful logic | Low | High |

---

## Modern Approach: Hooks + Presentational

With hooks, you can often replace containers with custom hooks:

```jsx
// Custom hook (replaces container logic)
function useUsers() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  const fetchUsers = async () => {
    // ... fetch logic
  };
  
  return { users, isLoading, refetch: fetchUsers };
}

// Component (presentational + hook)
function UserList() {
  const { users, isLoading } = useUsers();
  
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

**Reference:** [Patterns.dev: Container/Presentational Pattern](https://www.patterns.dev/react/container-presentational-pattern) 