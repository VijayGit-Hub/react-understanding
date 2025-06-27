# Virtual Scrolling in React

## What is Virtual Scrolling?

Virtual scrolling (or windowing) is a technique for efficiently rendering large lists by only rendering the items that are visible in the viewport, plus a small buffer. This dramatically improves performance and memory usage for lists with thousands (or more) items.

---

## Why is Virtual Scrolling Needed?
- Rendering thousands of DOM nodes at once is slow and memory-intensive.
- Most users only see a small portion of a large list at any time.
- Virtual scrolling ensures only visible items are rendered, making scrolling smooth and fast.

---

## When to Use Virtual Scrolling
- Use when you have large lists (hundreds or thousands of items) that need to be displayed in a scrollable container.
- Not needed for small lists (tens of items) where the performance impact is negligible.

---

## Production-Ready Libraries

### react-window (Recommended)
- Lightweight, fast, and widely used for virtualized lists and grids.
- Supports fixed and variable item sizes.

```jsx
import { FixedSizeList as List } from 'react-window';

function MyVirtualList({ items }) {
  return (
    <List
      height={400} // Height of the scrollable container
      itemCount={items.length}
      itemSize={50} // Height of each item (fixed)
      width={300}
    >
      {({ index, style }) => (
        <div style={style} key={index}>
          {items[index].name}
        </div>
      )}
    </List>
  );
}
```

### react-virtualized
- More features (tables, grids, infinite loader) but heavier than react-window.
- Use for advanced scenarios (multi-column grids, infinite loading, etc.).

---

## Custom Implementation (For Interview/Learning)

```jsx
import React, { useState, useRef } from 'react';

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

---

## Best Practices & Performance Notes
- **Prefer libraries** for production: They handle edge cases, dynamic heights, and are well-tested.
- **Custom implementations** are good for interviews or learning, but rarely used in real apps.
- Use `requestAnimationFrame` for smooth scroll updates if building your own.
- Debounce scroll events to avoid excessive renders.
- For dynamic heights, use libraries or advanced techniques.

---

## Key Takeaways
- Virtual scrolling is essential for large lists in React.
- Use `react-window` or `react-virtualized` for most real-world projects.
- Only render what the user can see for best performance. 