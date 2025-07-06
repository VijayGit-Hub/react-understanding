
# 🧮 Two Sum Problem - Solution Comparison & Analysis

**Problem Statement:**

Given an array of integers `nums` and an integer `target`, return **indices of the two numbers** such that they add up to `target`.

You may assume that each input would have **exactly one solution**, and you may not use the same element twice.

---

## 📘 Input Example

```js
const nums = [2, 7, 11, 15, 3, 6, 8];
const target = 14;
```

---

## 1. 🔨 Brute Force Solution (O(n²))

```js
function twoSumBruteForce(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
}
```

### 🔁 Iteration Table (Brute Force):

| i | j | nums[i] | nums[j] | sum        | Match? |
|---|---|---------|---------|------------|--------|
| 0 | 1 |   2     |   7     | 2 + 7 = 9  | ❌     |
| 0 | 2 |   2     |  11     | 2 + 11 = 13| ❌     |
| 0 | 3 |   2     |  15     | 2 + 15 = 17| ❌     |
| 0 | 4 |   2     |   3     | 2 + 3 = 5  | ❌     |
| 0 | 5 |   2     |   6     | 2 + 6 = 8  | ❌     |
| 0 | 6 |   2     |   8     | 2 + 8 = 10 | ❌     |
| 1 | 2 |   7     |  11     | 7 + 11 = 18| ❌     |
| 1 | 3 |   7     |  15     | 7 + 15 = 22| ❌     |
| 1 | 4 |   7     |   3     | 7 + 3 = 10 | ❌     |
| 1 | 5 |   7     |   6     | 7 + 6 = 13 | ❌     |
| 1 | 6 |   7     |   8     | 7 + 8 = 15 | ❌     |
| 2 | 3 |  11     |  15     | 11 + 15 = 26| ❌    |
| 2 | 4 |  11     |   3     | 11 + 3 = 14| ✅     |

➡️ **Result: [2, 4]**  
➡️ **Total Iterations: 13**

---

## 2. ⚡ Optimized Hash Map Solution (O(n))

```js
function twoSumMap(nums, target) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(nums[i], i);
  }
}
```

### 🧠 Iteration Breakdown (Map):

| i | nums[i] | complement | map before lookup        | Found? | Action         |
|---|---------|------------|---------------------------|--------|----------------|
| 0 | 2       | 12         | {}                        | ❌     | store 2 → 0    |
| 1 | 7       | 7          | {2: 0}                    | ❌     | store 7 → 1    |
| 2 | 11      | 3          | {2: 0, 7: 1}              | ❌     | store 11 → 2   |
| 3 | 15      | -1         | {2: 0, 7: 1, 11: 2}       | ❌     | store 15 → 3   |
| 4 | 3       | 11         | {2: 0, 7: 1, 11: 2, 15: 3}| ✅     | return [2, 4]  |

➡️ **Result: [2, 4]**  
➡️ **Total Iterations: 5**

---

## 3. ❌ Using `indexOf()` (Slower and Buggy)

```js
function twoSumIndexOf(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    let complement = target - nums[i];
    let j = nums.indexOf(complement);
    if (j !== -1 && j !== i) {
      return [i, j];
    }
  }
}
```

### ❌ Why This Can Fail:

1. `indexOf()` returns the **first match**, which could be `i` itself.
2. It performs a **linear scan** each time — O(n) inside a loop — O(n²) overall.
3. Can fail for **duplicate values**.

#### Example:

```js
nums = [3, 3], target = 6

i = 0, complement = 3
indexOf(3) → 0 → same as i → skipped

Fails to find correct [0, 1]
```

---

## ✅ Summary Table

| Method         | Correct? | Time Complexity | Notes                          |
|----------------|----------|------------------|--------------------------------|
| Brute Force    | ✅       | O(n²)           | Simple but slow                |
| Hash Map       | ✅       | O(n)            | Fast, elegant, preferred       |
| `indexOf` Trick| ⚠️ Sometimes | O(n²)      | Fails with duplicates, slower  |

---

## ✅ Final Recommendation

Use the **Hash Map solution** for optimal performance and reliability.  
It handles duplicates safely, and scales well for large input sizes.

```js
// Best version
function twoSum(nums, target) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(nums[i], i);
  }
}
```
