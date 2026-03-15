
# 9. Intersection of Two Sorted Arrays

### 🔍 Problem Statement

Given two sorted arrays, return their intersection. Each element in the result must appear as many times as it shows in both arrays, and the result should also be in sorted order.

---

## ✅ Approaches

---

### 1. Brute Force

#### 🔸 Algorithm (Step-by-Step):
1. Initialize an empty result vector `ans`.
2. Create a `visit[]` array of size `m` (length of second array), initialized with `0` to mark visited elements.
3. Traverse the first array using loop `i = 0 to n-1`.
4. For each element in `arr1[i]`, loop over `arr2[j]` and:
   - If `arr1[i] == arr2[j]` and `visit[j] == 0`, add it to result and mark `visit[j] = 1`.
   - Break if `arr2[j] > arr1[i]` because both arrays are sorted.
5. Return the result vector `ans`.

#### 💻 Code:
```cpp
#include <bits/stdc++.h>
vector<int> findArrayIntersection(vector<int> &arr1, int n, vector<int> &arr2, int m)
{
    vector<int> ans;
    int visit[m] = {0};

    for(int i = 0; i < n; i++) {
        for(int j = 0; j < m; j++) {
            if(arr1[i] == arr2[j] && visit[j] == 0) {
                ans.push_back(arr1[i]);
                visit[j] = 1;
                break;
            }
            if(arr2[j] > arr1[i]) break;
        }
    }

    return ans;
}
```

#### ⏱️ Time Complexity:
- **O(n1 * n2)** — where `n1` and `n2` are the sizes of the two arrays.
  - Worst-case: For each element in `arr1`, loop through all elements in `arr2`.

#### 🧠 Space Complexity:
- **O(n2)** — extra space for the `visit[]` array.

---

### 2. Optimal Solution (Two Pointer Technique)

#### 🔸 Algorithm (Step-by-Step):
1. Initialize two pointers `i = 0`, `j = 0`.
2. Create an empty result vector `ans`.
3. Loop while `i < n` and `j < m`:
   - If `arr1[i] < arr2[j]`, increment `i`.
   - Else if `arr2[j] < arr1[i]`, increment `j`.
   - Else, elements match:
     - Add `arr1[i]` to result.
     - Increment both `i` and `j`.
4. Return the result vector.

#### 💻 Code:
```cpp
#include <bits/stdc++.h>
vector<int> findArrayIntersection(vector<int> &arr1, int n, vector<int> &arr2, int m)
{
    int i = 0, j = 0;
    vector<int> ans;

    while(i < n && j < m) {
        if(arr1[i] < arr2[j]) {
            i++;
        }
        else if(arr2[j] < arr1[i]) {
            j++;
        }
        else {
            ans.push_back(arr1[i]);
            i++;
            j++;
        }
    }

    return ans;
}
```

#### ⏱️ Time Complexity:
- **O(n1 + n2)** — each pointer moves at most `n1` or `n2` times.

#### 🧠 Space Complexity:
- **O(1)** — no extra space used apart from output.

---

## 📌 Conclusion

| Approach         | Time Complexity | Space Complexity | Notes                                      |
|------------------|------------------|------------------|--------------------------------------------|
| Brute Force      | O(n1 * n2)       | O(n2)            | Simple but uses extra space & nested loops |
| Optimal (2 Ptr)  | O(n1 + n2)       | O(1)             | Most efficient for sorted arrays           |

---
