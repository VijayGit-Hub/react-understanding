# React Routing & Navigation: Practical & Advanced Patterns

This guide covers both practical, real-world routing patterns for React apps and advanced enterprise techniques. All code is self-contained, commented, and easy to follow.

---

## 1. Basic Routing with `react-router-dom`

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Simple page components
function HomePage() {
  return <h2>Home</h2>;
}
function AboutPage() {
  return <h2>About</h2>;
}
function NotFoundPage() {
  return <h2>404 - Not Found</h2>;
}

export default function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link> | <Link to="/about">About</Link>
      </nav>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/about" element={<AboutPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 2. Protected Routes (Authentication)

```jsx
import React, { createContext, useContext, useState } from 'react';
import { BrowserRouter, Routes, Route, Navigate, Link } from 'react-router-dom';

// Mock Auth Context
const AuthContext = createContext(null);
function useAuth() {
  return useContext(AuthContext);
}

function AuthProvider({ children }) {
  // For demo: user is null (logged out) or { name: 'Alice' }
  const [user, setUser] = useState(null);
  const login = () => setUser({ name: 'Alice' });
  const logout = () => setUser(null);
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// ProtectedRoute component
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  if (!user) {
    // Redirect to login if not authenticated
    return <Navigate to="/login" replace />;
  }
  return children;
}

function HomePage() {
  return <h2>Home (Public)</h2>;
}
function Dashboard() {
  return <h2>Dashboard (Protected)</h2>;
}
function LoginPage() {
  const { login } = useAuth();
  return (
    <div>
      <h2>Login</h2>
      <button onClick={login}>Log In</button>
    </div>
  );
}

export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <nav>
          <Link to="/">Home</Link> | <Link to="/dashboard">Dashboard</Link>
        </nav>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/dashboard" element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          } />
          <Route path="/login" element={<LoginPage />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

---

## 3. Role-Based Routes (Authorization)

```jsx
import React, { createContext, useContext, useState } from 'react';
import { BrowserRouter, Routes, Route, Navigate, Link } from 'react-router-dom';

// Mock Auth Context with roles
const AuthContext = createContext(null);
function useAuth() {
  return useContext(AuthContext);
}

function AuthProvider({ children }) {
  // For demo: user is null, or has a role
  const [user, setUser] = useState(null);
  const loginAsAdmin = () => setUser({ name: 'Admin', role: 'admin' });
  const loginAsUser = () => setUser({ name: 'User', role: 'user' });
  const logout = () => setUser(null);
  return (
    <AuthContext.Provider value={{ user, loginAsAdmin, loginAsUser, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// RoleProtectedRoute component
function RoleProtectedRoute({ allowedRoles, children }) {
  const { user } = useAuth();
  if (!user || !allowedRoles.includes(user.role)) {
    // Redirect to unauthorized page if not allowed
    return <Navigate to="/unauthorized" replace />;
  }
  return children;
}

function HomePage() {
  return <h2>Home (Public)</h2>;
}
function AdminPage() {
  return <h2>Admin Page (Admins Only)</h2>;
}
function UserPage() {
  return <h2>User Page (Users Only)</h2>;
}
function UnauthorizedPage() {
  return <h2>Unauthorized</h2>;
}
function LoginPage() {
  const { loginAsAdmin, loginAsUser } = useAuth();
  return (
    <div>
      <h2>Login</h2>
      <button onClick={loginAsAdmin}>Log In as Admin</button>
      <button onClick={loginAsUser}>Log In as User</button>
    </div>
  );
}

export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <nav>
          <Link to="/">Home</Link> | <Link to="/admin">Admin</Link> | <Link to="/user">User</Link>
        </nav>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/admin" element={
            <RoleProtectedRoute allowedRoles={['admin']}>
              <AdminPage />
            </RoleProtectedRoute>
          } />
          <Route path="/user" element={
            <RoleProtectedRoute allowedRoles={['user']}>
              <UserPage />
            </RoleProtectedRoute>
          } />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/unauthorized" element={<UnauthorizedPage />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

---

## 4. Advanced Patterns: Dynamic Routes, Middleware, and Guards

Below is a more advanced example, but **every function, component, and hook is defined and commented**. This is for reference in enterprise scenarios.

```jsx
import React, { createContext, useContext, useState } from 'react';
import { BrowserRouter, Routes, Route, Navigate, useLocation, useNavigate, Link } from 'react-router-dom';

// --- Auth Context with roles ---
const AuthContext = createContext(null);
function useAuth() {
  return useContext(AuthContext);
}
function AuthProvider({ children }) {
  // user: null, or { name, role }
  const [user, setUser] = useState(null);
  const loginAsAdmin = () => setUser({ name: 'Admin', role: 'admin' });
  const loginAsManager = () => setUser({ name: 'Manager', role: 'manager' });
  const logout = () => setUser(null);
  return (
    <AuthContext.Provider value={{ user, loginAsAdmin, loginAsManager, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// --- Middleware pattern ---
// Each middleware receives context and returns true/false
function logMiddleware({ user, location }) {
  console.log('Route access:', { user, path: location.pathname });
  return true;
}
function adminOnlyMiddleware({ user }) {
  return user && user.role === 'admin';
}

// --- RouteGuard component ---
function RouteGuard({ middleware = [], children }) {
  const { user } = useAuth();
  const location = useLocation();
  const navigate = useNavigate();

  // Run all middleware; if any returns false, redirect
  for (const mw of middleware) {
    if (!mw({ user, location, navigate })) {
      return <Navigate to="/unauthorized" replace />;
    }
  }
  return children;
}

// --- Pages ---
function HomePage() { return <h2>Home</h2>; }
function AdminPage() { return <h2>Admin (Admins Only)</h2>; }
function ManagerPage() { return <h2>Manager (Managers Only)</h2>; }
function UnauthorizedPage() { return <h2>Unauthorized</h2>; }
function LoginPage() {
  const { loginAsAdmin, loginAsManager } = useAuth();
  return (
    <div>
      <h2>Login</h2>
      <button onClick={loginAsAdmin}>Log In as Admin</button>
      <button onClick={loginAsManager}>Log In as Manager</button>
    </div>
  );
}

export default function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <nav>
          <Link to="/">Home</Link> | <Link to="/admin">Admin</Link> | <Link to="/manager">Manager</Link>
        </nav>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/admin" element={
            <RouteGuard middleware={[logMiddleware, adminOnlyMiddleware]}>
              <AdminPage />
            </RouteGuard>
          } />
          <Route path="/manager" element={
            <RouteGuard middleware={[logMiddleware]}>
              <ManagerPage />
            </RouteGuard>
          } />
          <Route path="/login" element={<LoginPage />} />
          <Route path="/unauthorized" element={<UnauthorizedPage />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

---

## 5. What is Middleware in React Routing? Why Use It?

**Middleware** is a pattern borrowed from backend frameworks (like Express.js) where you can run logic before a request reaches its final handler. In React routing, middleware lets you run custom logic before a route is rendered—such as checking permissions, logging, or redirecting.

### Why call it "middleware"?
- **Intention:** Middleware sits "in the middle" between a navigation event and the final page render. It can allow, block, or modify navigation.
- **Benefit:** It keeps your route logic clean and reusable. You can compose multiple checks (auth, logging, feature flags) without cluttering your page components.
- **Analogy:** Think of a security checkpoint at an airport. Before you board (see the page), you go through checks (middleware). If you pass, you continue; if not, you're redirected.

### When should you use middleware in routing?
- When you need to run logic for many routes (e.g., authentication, logging, analytics, feature flags)
- When you want to keep route/page components focused on UI, not access logic
- When you want to compose multiple checks in a clean, reusable way

### Simple Example: Logging Middleware

```jsx
// Middleware function: logs every route access
function logMiddleware({ user, location }) {
  console.log('User', user ? user.name : 'Guest', 'navigated to', location.pathname);
  return true; // Always allow navigation
}

// Usage in a RouteGuard
<RouteGuard middleware={[logMiddleware]}>
  <SomePage />
</RouteGuard>
```

### Example: Auth Middleware

```jsx
// Middleware: only allow if user is logged in
function authMiddleware({ user }) {
  return !!user; // true if user exists
}

<RouteGuard middleware={[authMiddleware]}>
  <ProtectedPage />
</RouteGuard>
```

**Summary:**
- Middleware lets you run logic before a route renders
- It's called middleware because it sits between navigation and rendering
- Use it for checks, logging, analytics, and more—without cluttering your UI code

---

**Summary Table:**

| Pattern                | Example Section | Key Points |
|------------------------|-----------------|------------|
| Basic Routing          | 1               | Simple navigation, 404 |
| Protected Route        | 2               | Auth context, redirect to login |
| Role-Based Route       | 3               | User roles, redirect to unauthorized |
| Middleware/Guards      | 4               | Custom logic, all code defined |

---

**Best Practices:**
- Keep routing logic simple unless you need advanced features
- Always define and comment any custom hooks/components
- Use context for auth and roles
- Use middleware/guards only for complex enterprise needs 

---

## 6. React Router v6 vs v5: Key Differences & Migration Guide

React Router v6 introduced several breaking changes and improvements over v5. Here are the main differences, with code snippets for each version.

### 1. Route Rendering: `component`/`render` vs `element`

**v5:**
```jsx
<Route path="/about" component={AboutPage} />
<Route path="/about" render={() => <AboutPage />} />
```
**v6:**
```jsx
<Route path="/about" element={<AboutPage />} />
```

---

### 2. Nested Routes: Children vs Nested `<Route>`

**v5:**
```jsx
<Route path="/dashboard" component={Dashboard}>
  <Route path="stats" component={Stats} />
</Route>
```
**v6:**
```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route path="stats" element={<Stats />} />
</Route>
```

---

### 3. Switch vs Routes

**v5:**
```jsx
<Switch>
  <Route path="/" component={Home} />
  <Route path="/about" component={About} />
</Switch>
```
**v6:**
```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

---

### 4. Redirects: `Redirect` vs `Navigate`

**v5:**
```jsx
<Redirect to="/" />
```
**v6:**
```jsx
<Navigate to="/" replace />
```

---

### 5. Route Matching: Exact Matching by Default

- **v5:** You had to add `exact` to prevent partial matches.
  ```jsx
  <Route exact path="/" component={Home} />
  ```
- **v6:** All routes are exact by default. No need for `exact` prop.
  ```jsx
  <Route path="/" element={<Home />} />
  ```

---

### 6. No More `withRouter`, Use Hooks

- **v5:**
  ```jsx
  import { withRouter } from 'react-router-dom';
  export default withRouter(MyComponent);
  ```
- **v6:**
  ```jsx
  import { useNavigate, useLocation, useParams } from 'react-router-dom';
  // Use these hooks inside your function component
  ```

---

### 7. Params: `match.params` vs `useParams()`

- **v5:**
  ```jsx
  function MyComponent({ match }) {
    return <div>ID: {match.params.id}</div>;
  }
  ```
- **v6:**
  ```jsx
  import { useParams } from 'react-router-dom';
  function MyComponent() {
    const { id } = useParams();
    return <div>ID: {id}</div>;
  }
  ```

---

### 8. Summary Table

| Feature                | v5 Example                        | v6 Example                       |
|------------------------|-----------------------------------|----------------------------------|
| Route Rendering        | component/render                   | element                          |
| Nested Routes          | Children prop                      | Nested <Route>                   |
| Switch/Routes          | <Switch>                           | <Routes>                         |
| Redirect               | <Redirect to="/" />               | <Navigate to="/" />             |
| Exact Matching         | exact prop needed                  | Exact by default                 |
| Access Params          | match.params                       | useParams() hook                 |
| withRouter             | HOC                                | Hooks (useNavigate, etc.)        |

---

**Best Practices:**
- Prefer v6 for new projects—simpler, more powerful, and better for TypeScript
- When migrating, update all `<Route>` usages, replace `<Switch>` with `<Routes>`, and use hooks instead of HOCs
- See the [official migration guide](https://reactrouter.com/en/main/upgrading/v5) for more details 