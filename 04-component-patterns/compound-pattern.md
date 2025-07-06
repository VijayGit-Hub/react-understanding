# Compound Pattern: Flexible Component APIs

The Compound Pattern allows you to create flexible, reusable component APIs by combining multiple components that work together. Instead of passing all props to a single component, you split functionality across multiple related components.

---

## What is the Compound Pattern?

The Compound Pattern creates a family of components that work together to provide a complete feature. The main component acts as a container, while child components handle specific parts of the functionality.

**Key Benefits:**
- More flexible than single-component APIs
- Better separation of concerns
- Easier to customize individual parts
- More intuitive for complex components

---

## Example: Tabs Component

Instead of a single `<Tabs>` component with many props, we create a family of components:

```jsx
import React, { createContext, useContext, useState } from 'react';

// Context for sharing state between tab components
const TabsContext = createContext();

// Main Tabs container
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = useState(defaultIndex);
  
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Tab list component
function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

// Individual tab component
function Tab({ index, children }) {
  const { activeIndex, setActiveIndex } = useContext(TabsContext);
  const isActive = index === activeIndex;
  
  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveIndex(index)}
    >
      {children}
    </button>
  );
}

// Tab panel component
function TabPanel({ index, children }) {
  const { activeIndex } = useContext(TabsContext);
  
  if (index !== activeIndex) return null;
  return <div className="tab-panel">{children}</div>;
}

// Usage
function App() {
  return (
    <Tabs defaultIndex={0}>
      <TabList>
        <Tab index={0}>Profile</Tab>
        <Tab index={1}>Settings</Tab>
        <Tab index={2}>Messages</Tab>
      </TabList>
      
      <TabPanel index={0}>
        <h2>Profile Content</h2>
        <p>This is the profile tab content.</p>
      </TabPanel>
      
      <TabPanel index={1}>
        <h2>Settings Content</h2>
        <p>This is the settings tab content.</p>
      </TabPanel>
      
      <TabPanel index={2}>
        <h2>Messages Content</h2>
        <p>This is the messages tab content.</p>
      </TabPanel>
    </Tabs>
  );
}
```

---

## Example: Modal Component

```jsx
import React, { createContext, useContext, useState } from 'react';

const ModalContext = createContext();

// Main Modal container
function Modal({ children, isOpen, onClose }) {
  if (!isOpen) return null;
  
  return (
    <ModalContext.Provider value={{ onClose }}>
      <div className="modal-overlay">
        <div className="modal">{children}</div>
      </div>
    </ModalContext.Provider>
  );
}

// Modal header
function ModalHeader({ children }) {
  const { onClose } = useContext(ModalContext);
  
  return (
    <div className="modal-header">
      <h2>{children}</h2>
      <button onClick={onClose}>&times;</button>
    </div>
  );
}

// Modal body
function ModalBody({ children }) {
  return <div className="modal-body">{children}</div>;
}

// Modal footer
function ModalFooter({ children }) {
  return <div className="modal-footer">{children}</div>;
}

// Usage
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
      
      <Modal isOpen={isModalOpen} onClose={() => setIsModalOpen(false)}>
        <ModalHeader>Confirm Action</ModalHeader>
        <ModalBody>
          <p>Are you sure you want to proceed?</p>
        </ModalBody>
        <ModalFooter>
          <button onClick={() => setIsModalOpen(false)}>Cancel</button>
          <button onClick={() => setIsModalOpen(false)}>Confirm</button>
        </ModalFooter>
      </Modal>
    </div>
  );
}
```

---

## Benefits of Compound Pattern

1. **Flexibility**: Users can customize individual parts without affecting others
2. **Composability**: Components can be arranged in different ways
3. **Separation of Concerns**: Each component has a single responsibility
4. **Better API**: More intuitive than passing many props to one component
5. **Reusability**: Individual components can be reused in different contexts

---

## When to Use Compound Pattern

- **Complex Components**: When a component has multiple logical parts
- **Flexible APIs**: When you want to give users control over component structure
- **Reusable Parts**: When individual parts might be used elsewhere
- **Better Developer Experience**: When you want a more intuitive API

---

## Best Practices

1. **Use Context**: Share state between related components using React Context
2. **Keep Components Focused**: Each component should have a single responsibility
3. **Provide Defaults**: Give sensible defaults for common use cases
4. **Document Usage**: Show clear examples of how components work together
5. **Validate Props**: Ensure required components are present

---

## Comparison: Single Component vs Compound Pattern

**Single Component (Less Flexible):**
```jsx
<Tabs
  tabs={[
    { label: 'Profile', content: <ProfileContent /> },
    { label: 'Settings', content: <SettingsContent /> }
  ]}
  defaultIndex={0}
  onTabChange={handleTabChange}
/>
```

**Compound Pattern (More Flexible):**
```jsx
<Tabs defaultIndex={0}>
  <TabList>
    <Tab index={0}>Profile</Tab>
    <Tab index={1}>Settings</Tab>
  </TabList>
  <TabPanel index={0}><ProfileContent /></TabPanel>
  <TabPanel index={1}><SettingsContent /></TabPanel>
</Tabs>
```

---

**Reference:** [Patterns.dev: Compound Pattern](https://www.patterns.dev/react/compound-pattern) 