# 🚨 Why Using Array Indices as Keys in React Lists is a Pitfall

## The Problem: Index Keys Can Break Your UI

Using array indices as keys in React lists can cause subtle, hard-to-debug bugs—especially when your list is dynamic (items can be added, removed, or reordered). Here's a simple example that shows exactly why.

---

## 📝 Example: The Checkbox List Bug

Suppose you have a list of tasks, each with a checkbox to mark it as done:

```jsx
function TaskList({ tasks }) {
  return (
    <ul>
      {tasks.map((task, i) => (
        <TaskItem key={i} text={task.text} />
      ))}
    </ul>
  );
}

function TaskItem({ text }) {
  const [checked, setChecked] = React.useState(false);
  return (
    <li>
      <input
        type="checkbox"
        checked={checked}
        onChange={() => setChecked(c => !c)}
      />
      {text}
    </li>
  );
}
```

### 1. **Initial List**
| Index | Text        | Checkbox State |
|-------|-------------|---------------|
| 0     | Buy milk    | ⬜️ (unchecked) |
| 1     | Walk dog    | ⬜️ (unchecked) |
| 2     | Read book   | ⬜️ (unchecked) |

Suppose you check the box for "Walk dog" (index 1):

| Index | Text        | Checkbox State |
|-------|-------------|---------------|
| 0     | Buy milk    | ⬜️            |
| 1     | Walk dog    | ☑️            |
| 2     | Read book   | ⬜️            |

---

### 2. **Now, Remove the First Item ("Buy milk")**

The new list is:
| Index | Text        | Checkbox State (expected) |
|-------|-------------|--------------------------|
| 0     | Walk dog    | ☑️ (should be checked)   |
| 1     | Read book   | ⬜️                      |

But what actually happens with `key={i}`:
- React sees index 0 is still present, so it **reuses the DOM node and state** from the old index 0 ("Buy milk").
- The checked state from "Buy milk" (which was unchecked) is now applied to "Walk dog".

| Index | Text        | Checkbox State (actual)   |
|-------|-------------|--------------------------|
| 0     | Walk dog    | ⬜️ (incorrect!)          |
| 1     | Read book   | ☑️ or ⬜️ (could be wrong) |

**Result:**
- The checkbox for "Walk dog" is now unchecked, even though you checked it before!
- The checked state is "shifted" to the wrong item, causing a confusing bug.

---

### 3. **How to Fix It?**

Use a unique, stable key (like an ID):

```jsx
{tasks.map(task => (
  <TaskItem key={task.id} text={task.text} />
))}
```

Now, when you remove "Buy milk", React knows "Walk dog" is still the same task (by ID), so it keeps the correct checkbox state.

---

## ✅ Summary
- **With index as key:** State and DOM nodes can get "shifted" to the wrong item when the list changes.
- **With unique ID as key:** State and DOM nodes always match the correct data, no matter how the list changes.

**This is why using array indices as keys is dangerous for dynamic lists!** 