# Sliding Window Maximum (LeetCode)

---

## Problem Statement

Given an integer array `nums` and an integer `k`, you need to find the **maximum**k** element in every contiguous subarray (window) of size ****\*\*\*\***.

### Example

```
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
```

Each output element represents the maximum of a sliding window of size `k` as it moves one step to the right.

---

## 1. Intuition / Approach (In Depth)

### ❌ Brute Force (Why it fails)

A naive approach would be:

- For every window of size `k`
- Scan all `k` elements to find the maximum

This results in:

```
Time Complexity = O(N * k)
```

Which is too slow for large `N`.

---

### ✅ Optimized Approach: **Monotonic Deque**

We use a **deque (double-ended queue)** that stores **indices**, not values.

The deque is maintained in such a way that:

- Elements are stored in **decreasing order of their values** (`nums[index]`)
- The **front of the deque always holds the index of the maximum element** of the current window

This structure allows us to:

- Add new elements efficiently
- Remove outdated elements
- Get the maximum in **O(1)** time per window

---

### 🔑 Key Observations

1. **Why store indices instead of values?**

   - To easily check whether an element is **outside the current window**
   - Window range is `[i - k + 1, i]`

2. **Why remove smaller elements from the back?**

   - If a new element is greater than an existing one, the smaller one can **never become a maximum** in future windows

3. **Why the front is always the maximum?**

   - Because the deque is maintained in decreasing order

---

### 🪜 Step-by-Step Flow

For each index `i`:

1. **Remove out-of-window elements**

   - If the index at the front is `<= i - k`, it no longer belongs to the window

2. **Maintain decreasing order**

   - Remove indices from the back whose values are smaller than `nums[i]`

3. **Insert current index**

4. **Record the maximum**

   - Once `i >= k - 1`, the window is complete
   - The element at `dq.front()` is the maximum

---

## 2. Code (Exact Code – No Explanation)

```cpp
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int n = nums.size();

        vector<int>ans;
        deque<int> dq;

        for(int i = 0; i < n; i++) {
            if(!dq.empty() && dq.front() <= i - k)
                dq.pop_front();

            while(!dq.empty() && nums[dq.back()] < nums[i])
                dq.pop_back();

            dq.push_back(i);

            // Once the first window is completed, add front element to result
            if(i >= k - 1) {
                ans.push_back(nums[dq.front()]);
            }   
        }   

        return ans;
    }
};
```

---

## 3. Time & Space Complexity (In Depth)

### ⏱ Time Complexity: **O(2N) → O(N)**

At first glance, the nested `while` loop might look expensive, but let’s analyze carefully.

#### 🔍 Push Operations

- Each index `i` is **pushed exactly once** into the deque

#### 🔍 Pop Operations

- Each index can be **popped at most once** from:
  - The front (out-of-window removal)
  - OR the back (maintaining decreasing order)

So overall:

```
Total push operations = N
Total pop operations  = N

Total operations = 2N
```

✅ Hence:

```
Time Complexity = O(2N) ≈ O(N)
```

This is **optimal** because every element is processed only a constant number of times.

---

### 💾 Space Complexity: **O(k) + O(N - k)**

Let’s break it down.

#### 1️⃣ Deque Space → `O(k)`

- The deque stores **indices of elements in the current window only**
- Maximum possible size of the deque is `k`

```
Deque space = O(k)
```

#### 2️⃣ Result Array → `O(N - k + 1)`

- For an array of size `N`, number of windows is:

```
N - k + 1
```

So result storage:

```
O(N - k)
```

---

### 📌 Final Space Complexity

```
Space Complexity = O(k) + O(N - k)
```

This is efficient and unavoidable since the output itself requires space.

---

##

|   |
| - |



