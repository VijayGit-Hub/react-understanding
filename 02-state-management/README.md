# React State Management - Senior Level Interview Questions

## 1. Redux Toolkit vs Context API: Architecture Decision Making

**Q: You're building a large-scale e-commerce application. Compare Redux Toolkit and Context API, and explain when you would choose one over the other. Provide a detailed implementation example for both approaches.**

### Answer
**Redux Toolkit** is better for complex state with frequent updates, middleware needs, and debugging requirements. **Context API** is simpler for static data or simple state sharing.

**Redux Toolkit Example:**
```jsx
import { createSlice, configureStore } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [], total: 0 },
  reducers: {
    addItem: (state, action) => {
      state.items.push(action.payload);
      state.total += action.payload.price;
    },
    removeItem: (state, action) => {
      const index = state.items.findIndex(item => item.id === action.payload);
      if (index > -1) {
        state.total -= state.items[index].price;
        state.items.splice(index, 1);
      }
    }
  }
});

const store = configureStore({
  reducer: { cart: cartSlice.reducer }
});
```

**Context API Example:**
```jsx
import React, { createContext, useContext, useReducer } from 'react';

const CartContext = createContext();

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload],
        total: state.total + action.payload.price
      };
    default:
      return state;
  }
};

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [], total: 0 });
  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}
```

**Decision Criteria:**
- **Redux**: Complex state, middleware, dev tools, large team
- **Context**: Simple state, static data, small team

---

## 2. State Management Performance Optimization

**Q: Your React application has performance issues due to frequent re-renders from state updates. Explain how you would optimize state management to minimize unnecessary re-renders and improve performance.**

### Answer
Use selective subscriptions, memoization, and state normalization to prevent unnecessary re-renders.

**Selective Subscription Pattern:**
```jsx
import { useSelector } from 'react-redux';

// Bad: Component re-renders when any cart state changes
function CartBad() {
  const cart = useSelector(state => state.cart);
  return <div>{cart.items.length} items</div>;
}

// Good: Component only re-renders when item count changes
function CartGood() {
  const itemCount = useSelector(state => state.cart.items.length);
  return <div>{itemCount} items</div>;
}
```

**State Normalization:**
```jsx
// Bad: Nested state structure
const badState = {
  users: [
    { id: 1, name: 'John', posts: [{ id: 1, title: 'Post 1' }] }
  ]
};

// Good: Normalized state structure
const goodState = {
  users: {
    byId: { 1: { id: 1, name: 'John' } },
    allIds: [1]
  },
  posts: {
    byId: { 1: { id: 1, title: 'Post 1', userId: 1 } },
    allIds: [1]
  }
};
```

**Performance Techniques:**
- Use `useSelector` with equality functions
- Normalize state structure
- Implement memoization with `useMemo` and `useCallback`
- Use `React.memo` for components

---

## 3. Advanced State Management Patterns: Micro-Frontend State Coordination

**Q: Design a state management solution for a micro-frontend architecture where multiple React applications need to share state and communicate with each other. Explain the challenges and your solution.**

### Answer
Use a combination of shared state management, event-driven communication, and state synchronization patterns.

**Shared State Container:**
```jsx
// Shared state management across micro-frontends
class SharedStateManager {
  constructor() {
    this.state = {};
    this.subscribers = new Map();
  }

  subscribe(key, callback) {
    if (!this.subscribers.has(key)) {
      this.subscribers.set(key, new Set());
    }
    this.subscribers.get(key).add(callback);
  }

  setState(key, value) {
    this.state[key] = value;
    this.notify(key, value);
  }

  notify(key, value) {
    const callbacks = this.subscribers.get(key);
    if (callbacks) {
      callbacks.forEach(callback => callback(value));
    }
  }
}

// Usage in micro-frontend
const sharedState = new SharedStateManager();

function useSharedState(key, defaultValue) {
  const [value, setValue] = useState(defaultValue);

  useEffect(() => {
    sharedState.subscribe(key, setValue);
    return () => sharedState.subscribers.get(key)?.delete(setValue);
  }, [key]);

  const updateValue = useCallback((newValue) => {
    sharedState.setState(key, newValue);
  }, [key]);

  return [value, updateValue];
}
```

**Event-Driven Communication:**
```jsx
// Cross-micro-frontend communication
class EventBus {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

const eventBus = new EventBus();

// In Micro-frontend A
eventBus.emit('user-logged-in', { userId: 123 });

// In Micro-frontend B
eventBus.on('user-logged-in', (data) => {
  // Update local state based on shared event
});
```

**Challenges & Solutions:**
- **State Synchronization**: Use shared state manager
- **Event Communication**: Implement event bus pattern
- **Performance**: Selective subscriptions and memoization
- **Security**: Validate shared data and implement proper access controls 