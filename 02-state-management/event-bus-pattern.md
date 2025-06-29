# Event Bus Pattern in React State Management

The event bus is a pattern for decoupled communication between micro-frontends or isolated React apps. It enables cross-app events without tight coupling or direct dependencies.

## Key Interview Questions & Answers

### 1. What is an event bus, and when should you use it in React apps?

**Answer:**
- **Event Bus Definition:**
  - An event bus is a simple pub/sub (publish/subscribe) system that allows different parts of an app (or different micro-frontends) to communicate by emitting and listening for named events.
  - It decouples the sender and receiver: the sender emits an event, and any interested receiver can listen for it.
- **When to Use:**
  - When you need to communicate between isolated modules, micro-frontends, or apps that don't share a direct parent-child relationship.
  - For cross-app notifications, global events (e.g., user-logged-in, theme-changed), or integration with legacy code.
- **When NOT to Use:**
  - For tightly-coupled, predictable state (prefer Context or Redux for app state).
  - When you need time-travel debugging, middleware, or strict state flow.

---

### 2. How do you implement an event bus for cross-micro-frontend communication?

**Answer:**
- **Simple Implementation:**
  - The event bus is usually a singleton object with `on`, `off`, and `emit` methods.
- **Example:**
  ```js
  // eventBus.js
  export const eventBus = {
    events: {},
    on(event, handler) {
      if (!this.events[event]) this.events[event] = new Set();
      this.events[event].add(handler);
    },
    off(event, handler) {
      if (this.events[event]) this.events[event].delete(handler);
    },
    emit(event, data) {
      if (this.events[event]) {
        this.events[event].forEach(handler => handler(data));
      }
    },
  };
  ```
- **Usage in React (function components):**
  ```jsx
  import { useEffect } from 'react';
  import { eventBus } from './eventBus';

  function Notifications() {
    useEffect(() => {
      const handler = (msg) => alert('Notification: ' + msg);
      eventBus.on('notify', handler);
      return () => eventBus.off('notify', handler); // Prevent memory leaks
    }, []);
    return null;
  }

  function SomeButton() {
    return <button onClick={() => eventBus.emit('notify', 'Hello!')}>Notify</button>;
  }
  ```
- **Micro-Frontend Use:**
  - Each micro-frontend can import the same event bus (from a shared package or window/global) and communicate without direct dependencies.

---

### 3. What are the pros and cons of the event bus pattern versus shared state containers?

**Answer:**
- **Pros:**
  - Decouples modules/micro-frontends (no direct imports or prop drilling)
  - Simple to implement and use
  - Good for cross-cutting events, notifications, or integration with non-React code
- **Cons:**
  - No state history, time-travel, or devtools
  - Harder to debug: event flow is implicit, not explicit
  - Risk of memory leaks if listeners aren't cleaned up
  - No built-in support for async flows, middleware, or selectors
  - Can lead to "spaghetti" event flows in large apps
- **When to prefer shared state (Redux/Context):**
  - For predictable, testable, and debuggable app state
  - When you need selectors, middleware, or time-travel

---

### 4. How do you ensure type safety and avoid memory leaks with event buses?

**Answer:**
- **Type Safety (TypeScript):**
  - Define event types and payloads with TypeScript interfaces or enums.
  - Example:
    ```ts
    type EventMap = {
      'notify': string;
      'user-logged-in': { userId: number };
    };
    class TypedEventBus {
      private events: { [K in keyof EventMap]?: Set<(data: EventMap[K]) => void> } = {};
      on<K extends keyof EventMap>(event: K, handler: (data: EventMap[K]) => void) { ... }
      off<K extends keyof EventMap>(event: K, handler: (data: EventMap[K]) => void) { ... }
      emit<K extends keyof EventMap>(event: K, data: EventMap[K]) { ... }
    }
    ```
- **Avoiding Memory Leaks:**
  - Always clean up listeners in `useEffect` (return a cleanup function that calls `off`).
  - For long-lived apps, periodically audit and remove unused listeners.
  - Consider using WeakRef or similar patterns for advanced scenarios.

---

### 5. How do you test event-driven communication in micro-frontends?

**Answer:**
- **Unit Test the Event Bus:**
  - Test that `on`, `off`, and `emit` work as expected.
- **Integration Test Components:**
  - Render components, emit events, and assert that listeners respond.
  - Example (Jest + React Testing Library):
    ```js
    import { render } from '@testing-library/react';
    import { eventBus } from './eventBus';
    import Notifications from './Notifications';

    test('notifies on event', () => {
      const alertMock = jest.spyOn(window, 'alert').mockImplementation(() => {});
      render(<Notifications />);
      eventBus.emit('notify', 'Test!');
      expect(alertMock).toHaveBeenCalledWith('Notification: Test!');
      alertMock.mockRestore();
    });
    ```
- **Best Practice:**
  - Keep event bus logic simple and well-tested.
  - For complex flows, prefer explicit state management (Redux/Context) for easier testing and debugging.

--- 