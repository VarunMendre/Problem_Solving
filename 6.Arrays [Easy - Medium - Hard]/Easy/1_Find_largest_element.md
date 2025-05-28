# 💪 1. Find the Largest Element in an Array

This file provides two approaches to solve the problem of finding the **largest element in an array** — a **Brute Force** method using sorting and an **Optimal Solution** using a max variable.

---

## 🥶 Brute Force Approach: Sorting

### ✨ Algorithm / Intuition:
Sort the array in **ascending order**, and the **last element** of the array will be the **largest element**.

### ✍️ Approach:
1. Sort the array using any sorting algorithm.
2. Return the element at index `size - 1`.

### 📊 Dry Run:
```cpp
Before sorting: arr[] = {2, 5, 1, 3, 0};
After sorting:  arr[] = {0, 1, 2, 3, 5};
Answer: arr[5-1] = 5
```

### 👨‍💻 Code:
```cpp
int sortArr(vector<int>& arr) {
    sort(arr.begin(), arr.end());
    return arr[arr.size() - 1];
}
```

### ⏱ Time Complexity: `O(N log N)`
> Due to the sorting step.

### 📂 Space Complexity: `O(1)`
> If using in-place sorting like Quick Sort.

---

## ✨ Optimal Solution: Max Variable

### ✨ Algorithm / Intuition:
Keep track of the **maximum value** seen so far by comparing each element in the array.

### ✍️ Approach:
1. Initialize a variable `largestEle` with the first element of the array.
2. Traverse the array from start to end.
3. If the current element is greater than `largestEle`, update it.
4. Return `largestEle` after the loop.

### 👨‍💻 Code:
```cpp
#include <bits/stdc++.h> 
int largestElement(vector<int> &arr, int n) {
    int largestEle = arr[0];

    for(int i = 0; i < n; i++) {
        if(arr[i] > largestEle) {
            largestEle = arr[i];
        }
    }
    return largestEle;
}
```

### ⏱ Time Complexity: `O(N)`
> Only a single traversal of the array.

### 📂 Space Complexity: `O(1)`
> No extra space used.

---

## ✅ Summary
| Approach       | Time Complexity | Space Complexity | Notes                          |
|----------------|------------------|-------------------|---------------------------------|
| Brute Force    | O(N log N)       | O(1)              | Uses sorting                   |
| Optimal        | O(N)             | O(1)              | Efficient for large arrays     |

> 🚀 Always prefer the optimal solution unless sorting is necessary for other reasons.
