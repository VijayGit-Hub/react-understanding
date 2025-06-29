# Micro-Frontend Communication: Methods, Patterns, and Best Practices

Micro-frontends (MFEs) enable teams to build and deploy independent features, but communication between MFEs—and between host and MFEs—can be challenging. This guide covers the most useful, scalable methods for micro-frontend communication, with practical examples and best practices.

---

## 1. Custom Events (window/DOM)

**When to use:**
- MFEs are loaded in separate bundles, possibly with different frameworks or versions.
- You want a decoupled, browser-native solution.

**How it works:**
- Use `window.dispatchEvent` and `window.addEventListener` with `CustomEvent` to send/receive messages.

**Example:**
```js
// MFE A (emit event)
window.dispatchEvent(new CustomEvent('myapp:user-logged-in', { detail: { userId: 123 } }));

// MFE B (listen for event)
useEffect(() => {
  function handler(e) { console.log('User logged in:', e.detail); }
  window.addEventListener('myapp:user-logged-in', handler);
  return () => window.removeEventListener('myapp:user-logged-in', handler);
}, []);
```

**Pros:** Decoupled, works across bundles/contexts, no shared code needed.
**Cons:** No type safety, must namespace events, not for large/complex state.

---

## Custom Events: Bidirectional Host/MFE Communication

Custom Events are a modern, browser-native, and highly scalable way for hosts and micro-frontends (MFEs) to communicate in both directions. They are especially useful when you want to avoid tight coupling, shared code, or direct dependencies.

### Why Custom Events?
- **Bidirectional:** Both host and MFEs can emit and listen for events.
- **Decoupled:** No shared JS context or direct imports required.
- **Scalable:** Any number of MFEs or the host can listen for or emit the same event.
- **Works across bundles, frameworks, and deployment models.**

### Host to MFE Example
```js
// Host triggers a data refresh in all MFEs
window.dispatchEvent(new CustomEvent('myapp:refresh-data'));
```
```js
// MFE listens for the event
useEffect(() => {
  function handler() { /* refresh logic */ }
  window.addEventListener('myapp:refresh-data', handler);
  return () => window.removeEventListener('myapp:refresh-data', handler);
}, []);
```

### MFE to Host Example
```js
// MFE notifies host of a user login
window.dispatchEvent(new CustomEvent('myapp:user-logged-in', { detail: { userId: 123 } }));
```
```js
// Host listens for the event
window.addEventListener('myapp:user-logged-in', (e) => {
  const user = e.detail;
  // Update global state, analytics, etc.
});
```

### Best Practices
- **Namespace your events** (e.g., `myapp:user-logged-in`) to avoid collisions.
- **Document event contracts** (event names, payload shapes) for team clarity.
- **Always clean up listeners** in React's `useEffect` to prevent memory leaks.
- **Use the `detail` property** of `CustomEvent` for structured data.

### When to Use
- **Best for:** Decoupled, event-driven communication between host and MFEs, especially when you want to avoid tight coupling or shared state.
- **Not ideal for:** Large, complex, or highly reactive state (use Redux, Zustand, or shared state for that).

### Summary
Custom Events are a top choice for most micro-frontend architectures, enabling robust, scalable, and decoupled communication between host and MFEs with minimal boilerplate and maximum flexibility.

---

## 2. JavaScript Event Bus (Shared Singleton)

**When to use:**
- MFEs share a JavaScript context (e.g., loaded via Module Federation or as ES modules).
- You want pub/sub but don't need browser events.

**How it works:**
- Export a singleton event bus object with `on`, `off`, and `emit` methods.

**Example:**
```js
// eventBus.js (shared)
export const eventBus = { ... };
// MFE A
import { eventBus } from 'shared/eventBus';
eventBus.emit('user-logged-in', { userId: 123 });
// MFE B
useEffect(() => { eventBus.on('user-logged-in', handler); return () => eventBus.off('user-logged-in', handler); }, []);
```

**Pros:** Type-safe (with TS), easy to test, can add features (middleware, etc.).
**Cons:** Only works if MFEs share the same JS context (not for iframes or separately bundled apps).

---

## 3. Props/Callbacks (Module Federation, React, or Web Components)

**When to use:**
- MFEs are loaded as components by a host (e.g., via Module Federation, React.lazy, or custom elements).
- You want direct, type-safe communication.

**How it works:**
- Host passes props/callbacks to MFE; MFE calls callbacks to notify host.

**Example:**
```jsx
// Host
<MFEComponent onUserLogin={user => setUser(user)} />
// MFE
function MFEComponent({ onUserLogin }) {
  return <button onClick={() => onUserLogin({ id: 1 })}>Login</button>;
}
```

**Pros:** Type-safe, simple, works well for parent-child relationships.
**Cons:** Only works if host controls MFE instantiation; not for peer-to-peer MFE comms.

---

## 4. Shared State (Redux, Zustand, etc.)

**When to use:**
- MFEs share a JS context and want to share complex or reactive state.

**How it works:**
- Host and MFEs use the same store instance (e.g., Redux store from a shared package).

**Example:**
```js
// shared/store.js
export const store = configureStore(...);
// Host and MFEs
import { store } from 'shared/store';
// Use Redux hooks/selectors as usual
```

**Pros:** Powerful, supports selectors, middleware, devtools.
**Cons:** Only works if MFEs share context; can tightly couple MFEs if not careful.

---

## 5. URL/Query Params

**When to use:**
- MFEs are loaded via routing (e.g., in SPA shell or via iframes).
- You want to persist state in the URL (e.g., filters, selected items).

**How it works:**
- Host or MFE updates the URL/query params; others read and react to changes.

**Example:**
```js
// Host updates URL
history.pushState({}, '', '?user=123');
// MFE reads from location.search
const params = new URLSearchParams(window.location.search);
```

**Pros:** Decoupled, persists across reloads, supports deep linking.
**Cons:** Not for high-frequency or sensitive data; limited payload size.

---

## 6. LocalStorage/SessionStorage

**When to use:**
- MFEs need to share small amounts of data, even across reloads or tabs.

**How it works:**
- Write to localStorage; listen for `storage` events to react to changes.

**Example:**
```js
// MFE A
localStorage.setItem('user', JSON.stringify({ id: 1 }));
// MFE B
window.addEventListener('storage', e => {
  if (e.key === 'user') {
    const user = JSON.parse(e.newValue);
    // React to user change
  }
});
```

**Pros:** Works across tabs/windows, persists data.
**Cons:** Not reactive in the same tab, not for large or sensitive data.

---

## 7. postMessage (for iframes or cross-window)

**When to use:**
- MFEs are loaded in iframes or separate windows.

**How it works:**
- Use `window.postMessage` to send messages; listen for `message` events.

**Example:**
```js
// Host to MFE (iframe)
iframe.contentWindow.postMessage({ type: 'user-logged-in', user: { id: 1 } }, '*');
// MFE listens
window.addEventListener('message', e => {
  if (e.data.type === 'user-logged-in') { ... }
});
```

**Pros:** Works across origins, windows, iframes.
**Cons:** Must handle security (origin checks), serialization, and message contracts.

---

## Host ↔ MFE Communication Patterns

### Host to MFE
- **Props/callbacks:** If MFE is loaded as a component.
- **Custom events:** Host dispatches events on window or a DOM node.
- **Shared state:** Host updates shared store.
- **postMessage:** If MFE is in an iframe.

### MFE to Host
- **Callbacks:** MFE calls a prop function.
- **Custom events:** MFE dispatches events on window/DOM.
- **Shared state:** MFE updates shared store.
- **postMessage:** MFE sends message to parent window.

**Example: Host triggers MFE action via custom event**
```js
// Host
window.dispatchEvent(new CustomEvent('myapp:refresh-data'));
// MFE
useEffect(() => {
  function handler() { /* refresh logic */ }
  window.addEventListener('myapp:refresh-data', handler);
  return () => window.removeEventListener('myapp:refresh-data', handler);
}, []);
```

---

## Best Practices & Pitfalls
- **Document your event/message contracts** (event names, payload shapes).
- **Namespace events** to avoid collisions.
- **Always clean up listeners** to prevent memory leaks.
- **Prefer decoupled methods** (custom events, postMessage) for independent deployment.
- **Use shared state only when MFEs are tightly integrated.**
- **Validate and sanitize all cross-origin messages.**

---

## Summary Table

| Method                | Use Case                                 | Pros                        | Cons                        |
|-----------------------|------------------------------------------|-----------------------------|-----------------------------|
| Custom Events         | Decoupled, browser-native, cross-bundle  | Simple, decoupled           | No type safety, no history  |
| JS Event Bus          | Shared context, pub/sub                  | Type-safe, extensible       | Not cross-bundle            |
| Props/Callbacks       | Host loads MFE as component              | Type-safe, simple           | Not peer-to-peer            |
| Shared State          | Complex, reactive state, shared context  | Powerful, devtools          | Tight coupling              |
| URL/Query Params      | Routing, deep linking                    | Decoupled, persistent       | Limited, not reactive       |
| LocalStorage          | Small, persistent data                   | Cross-tab, persistent       | Not reactive in tab         |
| postMessage           | Iframes, cross-window                    | Cross-origin, flexible      | Security, serialization     |

--- 