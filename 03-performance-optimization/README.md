# React Performance Optimization - Senior Level Interview Questions

## 1. React.memo, useMemo, and useCallback: When and How to Use

**Q: Explain the differences between React.memo, useMemo, and useCallback. Provide real-world scenarios where each should be used, and demonstrate how to implement them effectively in a complex component tree.**

### Answer
**React.memo** prevents re-renders when props haven't changed. **useMemo** memoizes expensive calculations. **useCallback** memoizes functions to prevent child re-renders.

**React.memo Example:**
```jsx
import React from 'react';

const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
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

// Custom comparison function for complex props
const areEqual = (prevProps, nextProps) => {
  return prevProps.data.length === nextProps.data.length &&
         prevProps.data.every((item, index) => item.id === nextProps.data[index].id);
};

const OptimizedComponent = React.memo(ExpensiveComponent, areEqual);
```

**useMemo Example:**
```jsx
import React, { useMemo } from 'react';

function DataTable({ data, filters, sortBy }) {
  const processedData = useMemo(() => {
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

**useCallback Example:**
```jsx
import React, { useCallback, useState } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  const handleAddItem = useCallback((newItem) => {
    setItems(prev => [...prev, newItem]);
  }, []); // Empty dependency array since it doesn't depend on any values

  const handleRemoveItem = useCallback((itemId) => {
    setItems(prev => prev.filter(item => item.id !== itemId));
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <ChildComponent 
        items={items} 
        onAdd={handleAddItem} 
        onRemove={handleRemoveItem} 
      />
    </div>
  );
}
```

**When to Use:**
- **React.memo**: Pure components with expensive renders
- **useMemo**: Expensive calculations or object creation
- **useCallback**: Functions passed as props to memoized children

---

## 2. Code Splitting and Lazy Loading Strategies

**Q: Design a comprehensive code splitting strategy for a large React application. Explain different approaches (route-based, component-based, vendor splitting) and implement a solution that optimizes both initial load time and runtime performance.**

### Answer
Use dynamic imports, React.lazy, and Suspense for route-based splitting. Implement vendor splitting for third-party libraries and component-based splitting for heavy components.

**Route-Based Code Splitting:**
```jsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
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

**Component-Based Code Splitting:**
```jsx
import React, { useState, Suspense } from 'react';

// Lazy load heavy components
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

**Webpack Configuration for Vendor Splitting:**
```javascript
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

**Preloading Strategy:**
```jsx
// Preload critical routes
const preloadAnalytics = () => {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = '/static/js/analytics.chunk.js';
  document.head.appendChild(link);
};

// Trigger preload on hover
function Navigation() {
  return (
    <nav>
      <a href="/analytics" onMouseEnter={preloadAnalytics}>
        Analytics
      </a>
    </nav>
  );
}
```

---

## 3. Virtual Scrolling and Large List Optimization

**Q: Implement a virtual scrolling solution for rendering a list of 100,000 items efficiently. Explain the concept of virtualization, handle dynamic heights, and optimize for smooth scrolling performance.**

### Answer
Virtual scrolling renders only visible items, dramatically improving performance for large lists. Use intersection observers and dynamic height calculations.

**Basic Virtual Scrolling Implementation:**
```jsx
import React, { useState, useEffect, useRef } from 'react';

function VirtualList({ items, itemHeight = 50, containerHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const visibleItemCount = Math.ceil(containerHeight / itemHeight);
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleItemCount + 1, items.length);
  
  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;

  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };

  const visibleItems = items.slice(startIndex, endIndex);

  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <div
              key={startIndex + index}
              style={{ height: itemHeight, borderBottom: '1px solid #eee' }}
            >
              {item.name}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

**Dynamic Height Virtual Scrolling:**
```jsx
import React, { useState, useRef, useCallback } from 'react';

function DynamicVirtualList({ items, containerHeight = 400 }) {
  const [itemHeights, setItemHeights] = useState(new Map());
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const getItemHeight = useCallback((index) => {
    return itemHeights.get(index) || 50; // Default height
  }, [itemHeights]);

  const getTotalHeight = useCallback(() => {
    let total = 0;
    for (let i = 0; i < items.length; i++) {
      total += getItemHeight(i);
    }
    return total;
  }, [items.length, getItemHeight]);

  const getVisibleRange = useCallback(() => {
    let currentHeight = 0;
    let startIndex = 0;
    let endIndex = 0;

    // Find start index
    for (let i = 0; i < items.length; i++) {
      const itemHeight = getItemHeight(i);
      if (currentHeight + itemHeight > scrollTop) {
        startIndex = i;
        break;
      }
      currentHeight += itemHeight;
    }

    // Find end index
    for (let i = startIndex; i < items.length; i++) {
      const itemHeight = getItemHeight(i);
      currentHeight += itemHeight;
      if (currentHeight > scrollTop + containerHeight) {
        endIndex = i;
        break;
      }
    }

    return { startIndex, endIndex: endIndex || items.length - 1 };
  }, [scrollTop, containerHeight, items.length, getItemHeight]);

  const getOffsetY = useCallback((index) => {
    let offset = 0;
    for (let i = 0; i < index; i++) {
      offset += getItemHeight(i);
    }
    return offset;
  }, [getItemHeight]);

  const handleItemResize = useCallback((index, height) => {
    setItemHeights(prev => new Map(prev).set(index, height));
  }, []);

  const { startIndex, endIndex } = getVisibleRange();
  const visibleItems = items.slice(startIndex, endIndex + 1);
  const offsetY = getOffsetY(startIndex);

  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: getTotalHeight(), position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <DynamicListItem
              key={startIndex + index}
              item={item}
              onResize={(height) => handleItemResize(startIndex + index, height)}
            />
          ))}
        </div>
      </div>
    </div>
  );
}

function DynamicListItem({ item, onResize }) {
  const itemRef = useRef(null);

  useEffect(() => {
    if (itemRef.current) {
      const resizeObserver = new ResizeObserver((entries) => {
        for (const entry of entries) {
          onResize(entry.contentRect.height);
        }
      });
      
      resizeObserver.observe(itemRef.current);
      return () => resizeObserver.disconnect();
    }
  }, [onResize]);

  return (
    <div ref={itemRef} style={{ borderBottom: '1px solid #eee' }}>
      <h3>{item.name}</h3>
      <p>{item.description}</p>
      {/* Dynamic content that may change height */}
    </div>
  );
}
```

**Performance Optimizations:**
- Use `requestAnimationFrame` for smooth scrolling
- Implement debouncing for scroll events
- Use `ResizeObserver` for dynamic heights
- Cache calculated positions
- Implement recycling for DOM nodes 