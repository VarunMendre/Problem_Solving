# 🍎 Fruit Into Baskets (LeetCode)

This document explains **three approaches** to solve the *Fruit Into Baskets* problem, gradually improving from brute force to an optimal sliding window solution. Each approach is explained with **intuition, code, and complexity analysis**, just like we do daily for revision and interviews.

---

## 🧪 Brute Force Approach

### 1️⃣ Approach / Intuition
The brute force idea is to consider **every possible subarray** starting from index `i` and try to extend it as far as possible while keeping **at most two distinct fruit types**.

- For each starting index `i`, we iterate forward using `j`.
- A `set` is used to track distinct fruit types.
- If the set size exceeds `2`, we stop expanding the subarray.
- Track the maximum length of valid subarrays.

This works because the basket constraint allows **only two fruit types at a time**.

---

### 2️⃣ Code

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int n = fruits.size();
        int maxLen = 0;
        
        for(int i = 0; i < n; i++) {
            set<int> st;
            for(int j = i; j < n; j++) {
                st.insert(fruits[j]);
                if(st.size() <= 2) {
                    maxLen = max(maxLen, j - i + 1);
                } else {
                    break;
                }
            }
        }
        return maxLen;
    }
};
```

---

### 3️⃣ Time & Space Complexity
- **Time Complexity:** `O(N²)`  
  Even though `set` insertion is `O(log 3)`, it is constant because the set holds at most 3 elements.
- **Space Complexity:** `O(1)` (constant space for the set)

---

## ⚡ Better Approach (Sliding Window with Shrinking Loop)

### 1️⃣ Approach / Intuition
Instead of checking all subarrays, we use a **sliding window**:

- Two pointers `l` and `r` define a window.
- A `map` stores fruit frequencies in the current window.
- Expand the window using `r`.
- If fruit types exceed 2, shrink the window from `l` until valid.
- Update maximum window length.

This avoids recalculating overlapping subarrays.

---

### 2️⃣ Code

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int n = fruits.size();
        int l = 0, r = 0, maxLen = 0;
        map<int, int> mpp;

        while(r < n) {
            mpp[fruits[r]]++;

            while(mpp.size() > 2) {
                mpp[fruits[l]]--;
                if(mpp[fruits[l]] == 0)
                    mpp.erase(fruits[l]);
                l++;
            }

            maxLen = max(maxLen, r - l + 1);
            r++;
        }
        return maxLen;
    }
};
```

---

### 3️⃣ Time & Space Complexity
- **Time Complexity:** `O(2N)` ≈ `O(N)`  
  Each pointer moves at most `N` times.
- **Space Complexity:** `O(1)` (map stores max 2–3 keys)

---

## 🚀 Optimal Approach (Sliding Window without Inner Loop)

### 1️⃣ Approach / Intuition
This is a **refined sliding window** approach:

- Expand `r` and add fruits to the map.
- If fruit types exceed 2, **move `l` only once** instead of looping.
- This works because we only need to remove **one extra fruit type** at a time.

The window automatically stabilizes over iterations.

---

### 2️⃣ Code

```cpp
class Solution {
public:
    int totalFruit(vector<int>& fruits) {
        int n = fruits.size();
        int l = 0, r = 0, maxLen = 0;
        map<int, int> mpp;

        while(r < n) {
            mpp[fruits[r]]++;

            if(mpp.size() > 2) {
                mpp[fruits[l]]--;
                if(mpp[fruits[l]] == 0)
                    mpp.erase(fruits[l]);
                l++;
            }

            maxLen = max(maxLen, r - l + 1);
            r++;
        }
        return maxLen;
    }
};
```

---

### 3️⃣ Time & Space Complexity
- **Time Complexity:** `O(N)`  
  Single pass with constant work per element.
- **Space Complexity:** `O(1)` (map size ≤ 3)

---

## 🧠 Key Takeaway

> "Sliding window problems are about **maintaining constraints**, not recomputing subarrays."

This pattern directly applies to:
- Longest substring with at most K distinct characters
- Max Consecutive Ones III
- Subarrays with at most K zeros

---

✅ **Interview-ready**  
✅ **Revision-friendly**  
✅ **Production-grade logic**

