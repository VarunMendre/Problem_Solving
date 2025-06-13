# 📂 Problem 8: Find Union / Merge of Two Sorted Arrays

---

## 🧠 Brute Force Approach

### ✅ Approach:

The brute force method uses a `set` to store only unique values in sorted order.

1. Insert all elements of the first array `a` into a set.
2. Insert all elements of the second array `b` into the same set.
3. The set automatically removes duplicates and keeps elements sorted.
4. Convert the set into a vector and return it.

---

### 💻 Code:
```cpp
#include <bits/stdc++.h>
vector<int> sortedArray(vector<int> a, vector<int> b) {
    int n1 = a.size();
    int n2 = b.size();
    set<int> st;

    for(int i = 0; i < n1; i++) {
        st.insert(a[i]);
    }

    for(int i = 0; i < n2; i++) {
        st.insert(b[i]);
    }

    vector<int> temp;
    for(auto it : st) {
        temp.push_back(it);
    }

    return temp;
}
```

---

### ⏱️ Time Complexity:

- `O(n1 * log n1 + n2 * log n2)`: inserting into a set takes `log n` time per element.
- `O(n1 + n2)`: for copying set elements into a result vector.

**Total:** `O((n1 + n2) * log(n1 + n2))`

---

### 💾 Space Complexity:

- `O(n1 + n2)` for the set (worst case: all elements are unique)
- `O(n1 + n2)` for the result vector

**Total:** `O(n1 + n2)`

---

## 🚀 Optimal Approach (Two-Pointer Technique)

### ✅ Approach:

Since both arrays are sorted, we can use a two-pointer technique:

1. Initialize `i = 0`, `j = 0`.
2. Compare elements at `a[i]` and `b[j]`:
   - If `a[i] < b[j]`: push `a[i]` if not a duplicate and increment `i`.
   - Else if `a[i] > b[j]`: push `b[j]` if not a duplicate and increment `j`.
   - If equal: push one of them if not a duplicate and increment both `i` and `j`.
3. After loop, push remaining elements from `a` and `b` (if any), avoiding duplicates.

---

### 💻 Code:
```cpp
vector<int> sortedArray(vector<int> a, vector<int> b) {
    int n1 = a.size();
    int n2 = b.size();
    int i = 0, j = 0;

    vector<int> unionArray;

    while(i < n1 && j < n2) {
        if(a[i] <= b[j]) {
            if(unionArray.empty() || unionArray.back() != a[i]) {
                unionArray.push_back(a[i]);
            }
            i++;
        } else {
            if(unionArray.empty() || unionArray.back() != b[j]) {
                unionArray.push_back(b[j]);
            }
            j++;
        }
    }

    while(i < n1) {
        if(unionArray.empty() || unionArray.back() != a[i]) {
            unionArray.push_back(a[i]);
        }
        i++;
    }

    while(j < n2) {
        if(unionArray.empty() || unionArray.back() != b[j]) {
            unionArray.push_back(b[j]);
        }
        j++;
    }

    return unionArray;
}
```

---

### ⏱️ Time Complexity:

- `O(n1 + n2)` — We traverse each element once from both arrays.

---

### 💾 Space Complexity:

- `O(n1 + n2)` — In the worst-case, all elements are unique and included in the result vector.

> No extra data structure like `set` is used.

---

### ✅ Final Note:

- Optimal approach is more efficient for sorted arrays.
- Always check if the arrays are sorted before choosing the method.
