# Code Splitting & Lazy Loading in React

## What is Code Splitting?

Code splitting is a technique to split your JavaScript bundle into smaller chunks, so users only download the code they need for the current page or feature. This improves initial load time and overall performance, especially in large applications.

---

## Why is Code Splitting Important?
- Reduces initial bundle size, leading to faster first paint and better user experience.
- Allows users to load only the code required for the current route or feature.
- Enables better caching and parallel loading of resources.

---

## Types of Code Splitting in React

### 1. Route-Based Code Splitting
Load different parts of your app only when the user navigates to a specific route.

```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Dashboard />} />
          <Route path="/analytics" element={<Analytics />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```
- **Best Practice:** Always wrap lazy-loaded components in `<Suspense>` with a meaningful fallback UI.

---

### 2. Component-Based Code Splitting
Load heavy or rarely-used components only when needed (e.g., after a button click).

```jsx
import React, { useState, Suspense, lazy } from 'react';

const HeavyChart = lazy(() => import('./components/HeavyChart'));
const DataTable = lazy(() => import('./components/DataTable'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showTable, setShowTable] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Load Chart</button>
      <button onClick={() => setShowTable(true)}>Load Table</button>
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
      {showTable && (
        <Suspense fallback={<div>Loading table...</div>}>
          <DataTable />
        </Suspense>
      )}
    </div>
  );
}
```
- **Best Practice:** Use component-based splitting for modals, charts, or admin panels that are not always visible.

---

### 3. Vendor Splitting (Webpack Example)
Split third-party libraries (like React, Lodash, etc.) into separate chunks for better caching and parallel loading.

```js
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react',
          chunks: 'all',
        },
      },
    },
  },
};
```
- **Best Practice:** Use your build tool's documentation to fine-tune vendor splitting for your app's needs.

---

## Lazy Loading in React

Lazy loading is the practice of loading resources only when they are needed. In React, this is typically done with `React.lazy` and `Suspense`.

- **React.lazy:** Dynamically imports a component.
- **Suspense:** Provides a fallback UI while the component is loading.

**Example:**
```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function MyPage() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

---

## Advanced: Preloading and Prefetching

- **Preloading:** Load code for a route/component before the user navigates there (e.g., on hover).
- **Prefetching:** Hint the browser to fetch resources in advance for likely future navigation.

**Example:**
```jsx
const preloadAnalytics = () => {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = '/static/js/analytics.chunk.js';
  document.head.appendChild(link);
};

// Usage in navigation
<a href="/analytics" onMouseEnter={preloadAnalytics}>Analytics</a>
```

---

## Key Takeaways
- Use code splitting and lazy loading to improve performance and user experience.
- Always provide a fallback UI for lazy-loaded components.
- Use route-based splitting for main pages, component-based for heavy/rarely-used features, and vendor splitting for third-party libraries.
- Consider preloading/prefetching for anticipated navigation. 