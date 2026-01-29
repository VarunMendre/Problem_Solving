# Sum of Subarray Minimums – In‑Depth Explanation

This document explains the given **C++ solution for the “Sum of Subarray Minimums” problem** using **monotonic stacks**. The explanation is broken down into:

1. **Intuition / Approach**
2. **Code Walkthrough (Function‑wise & Logic‑wise)**
3. **Time & Space Complexity Analysis**

---

## 1. Intuition / Approach

### 🔍 Problem Restatement
You are given an array `arr`. For **every possible subarray**, find the **minimum element**, and return the **sum of all those minimums**.

A brute‑force approach would:
- Generate all subarrays → **O(N²)**
- Find minimum for each → **O(N)**
- Total → **O(N³)** ❌ (too slow)

---

### 💡 Optimized Idea (Key Insight)
Instead of iterating over subarrays, **flip the thinking**:

> ❝ For each element `arr[i]`, count how many subarrays exist where `arr[i]` is the **minimum** ❞

If we can calculate:
- How many subarrays consider `arr[i]` as the minimum
- Then multiply that count by `arr[i]`

We can sum all contributions efficiently.

---

### 📐 How do we count subarrays where `arr[i]` is minimum?

We find:

- **Previous Smaller or Equal Element (PSEE)** → boundary on the **left**
- **Next Smaller Element (NSE)** → boundary on the **right**

These boundaries tell us **how far `arr[i]` can expand** while still remaining the minimum.

```
Left choices  = i - PSEE[i]
Right choices = NSE[i] - i

Total subarrays where arr[i] is minimum = left * right
```

---

### 🧠 Why Monotonic Stack?

Monotonic stacks help us find **nearest smaller elements in O(N)** time:
- Traverse once
- Each element is pushed & popped at most once

Perfect fit for this problem.

---

## 2. Code Explanation (Deep Dive)

### 🔹 Function 1: `nextSmallerElement`

```cpp
vector<int> nextSmallerElement(vector<int> &arr,int &n)
```

#### 🎯 Purpose
Find the **next element to the right** of each index that is **strictly smaller**.

If no such element exists, store `n`.

#### 🧱 Stack Behavior
- Stack stores **indices**
- Maintains **increasing order** from top to bottom

#### 🔁 Traversal
- Loop from **right to left**
- Pop elements **>= current** to ensure strict smaller

#### 📌 Result
`NSE[i]` = index of next smaller element

---

### 🔹 Function 2: `previousSmallerEqualElement`

```cpp
vector<int> previousSmallerEqualElement(vector<int> &arr,int &n)
```

#### 🎯 Purpose
Find the **previous element to the left** that is **smaller or equal**.

If none exists, store `-1`.

#### ⚠️ Why "Smaller or Equal" here?
This avoids **double counting** when duplicate elements exist.

#### 🔁 Traversal
- Loop from **left to right**
- Pop elements **> current** (allow equal elements)

#### 📌 Result
`PSEE[i]` = index of previous smaller or equal element

---

### 🔹 Main Function: `sumSubarrayMins`

```cpp
int sumSubarrayMins(vector<int>& arr)
```

#### 🧩 Steps Breakdown

1. Get array size
2. Compute:
   - `NSE` using nextSmallerElement
   - `PSEE` using previousSmallerEqualElement

3. For each index `i`:

```cpp
int left = i - PSEE[i];
int right = NSE[i] - i;
```

These represent:
- Number of valid subarrays extending **left**
- Number of valid subarrays extending **right**

4. Contribution calculation:

```cpp
freq = left * right
value = freq * arr[i]
```

5. Add contribution to total using modulo

```cpp
total = (total + value) % mod;
```

---

### 🧮 Why Modulo?

The number of subarrays can be very large.
To avoid **integer overflow**, we use:

```
mod = 1e9 + 7
```

---

## 3. Time & Space Complexity

### ⏱️ Time Complexity

| Operation | Complexity |
|---------|------------|
| Next Smaller Element | O(N) |
| Previous Smaller / Equal | O(N) |
| Final Loop | O(N) |

**Total:**
```
O(N + N + N) = O(3N) ≈ O(N)
```

(Your notation O(4N) is correct but simplifies to O(N))

---

### 🧠 Space Complexity

| Structure | Space |
|---------|-------|
| NSE array | O(N) |
| PSEE array | O(N) |
| Stack usage | O(N) |

**Total:**
```
O(3N) ≈ O(N)
```

---

## ✅ Final Takeaway

- This solution avoids brute force by **counting contributions**
- Monotonic stacks help find boundaries efficiently
- Each element contributes independently
- Overall solution is **optimal and interview‑ready** 🚀

---

If you want:
- Dry run with example
- Visualization of stacks
- Similar problems (Max Subarray Min/Max)

Just say the word 👌

