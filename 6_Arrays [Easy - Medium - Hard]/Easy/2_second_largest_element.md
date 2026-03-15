
# 📘 Second Largest and Second Smallest Element in an Array

This document describes three different approaches to find the **second largest** and **second smallest** elements in an array:

- Brute Force
- Better Solution
- Optimal Solution

---

## ✅ Problem Statement

Given an array of `n` integers, find:
- The second largest element
- The second smallest element

Return `-1` if they don't exist (i.e., array size < 2 or all elements are the same).

---

## 🔍 1. Brute Force Approach

### ✅ **Code**
```cpp
void getElements(int arr[], int n) {
    if (n == 0 || n == 1) {
        cout << -1 << " " << -1 << endl;  // Edge case
        return;
    }

    sort(arr, arr + n);  // Sort the array

    int small = arr[1];       // Second smallest
    int large = arr[n - 2];   // Second largest

    cout << "Second smallest is " << small << endl;
    cout << "Second largest is " << large << endl;
}
```

### 🧠 **Approach**
- Sort the array in ascending order.
- Second smallest is at index `1` and second largest is at index `n - 2`.

### ⏱️ **Time Complexity**
- `O(n log n)` due to sorting.

### 📦 **Space Complexity**
- `O(1)` (in-place sorting if built-in sort is used).

### ⚠️ **Limitations**
- Doesn't handle duplicates properly. For example, `[1, 1, 1, 2]` will return `1` as second smallest.

---

## ⚙️ 2. Better Approach (Two Passes)

### ✅ **Code**
```cpp
int findSecondLargest(int arr[], int n) {
    int largest = arr[0];

    // First pass to find the largest
    for (int i = 1; i < n; i++) {
        if (arr[i] > largest) {
            largest = arr[i];
        }
    }

    int sLargest = -1;

    // Second pass to find the second largest
    for (int i = 0; i < n; i++) {
        if (arr[i] != largest && arr[i] > sLargest) {
            sLargest = arr[i];
        }
    }

    return sLargest;
}
```

> Similar logic applies for `secondSmallest` by reversing the comparison logic.

### 🧠 **Approach**
- **First Pass:** Find the largest.
- **Second Pass:** Find the largest element that is not equal to the maximum found earlier.

### ⏱️ **Time Complexity**
- `O(n)` (Two passes through the array).

### 📦 **Space Complexity**
- `O(1)`.

### ✅ **Advantages**
- Handles duplicates properly.
- Doesn't sort the array.

---

## 🚀 3. Optimal Approach (Single Pass)

### ✅ **Code**
```cpp
int secondLargest(vector<int> &a, int n) {
    int largest = a[0];
    int sLargest = -1;

    for (int i = 1; i < n; i++) {
        if (a[i] > largest) {
            sLargest = largest;
            largest = a[i];
        }
        else if (a[i] < largest && a[i] > sLargest) {
            sLargest = a[i];
        }
    }
    return sLargest;
}

int secondSmallest(vector<int> &a, int n) {
    int smallest = a[0];
    int sSmallest = INT_MAX;

    for (int i = 1; i < n; i++) {
        if (a[i] < smallest) {
            sSmallest = smallest;
            smallest = a[i];
        }
        else if (a[i] != smallest && a[i] < sSmallest) {
            sSmallest = a[i];
        }
    }
    return (sSmallest == INT_MAX) ? -1 : sSmallest;
}

vector<int> getSecondOrderElements(int n, vector<int> a) {
    int sLargest = secondLargest(a, n);
    int sSmallest = secondSmallest(a, n);
    return {sLargest, sSmallest};
}
```

### 🧠 **Approach**
- Traverse the array once while maintaining two variables:
  - `largest`, `secondLargest`
  - `smallest`, `secondSmallest`
- Update them accordingly during a **single pass**.

### ⏱️ **Time Complexity**
- `O(n)` – only one traversal.

### 📦 **Space Complexity**
- `O(1)` – no extra space used.

### ✅ **Advantages**
- Most efficient.
- Handles all edge cases.
- No sorting, only one loop.

---

## 📌 Edge Cases to Consider

- **Empty or single-element array** → Return `-1` for both.
- **All elements same** → No second largest/smallest.
- **Duplicates** → Proper handling without skipping valid second elements.

---

## 🧪 Example

```cpp
Input: [1, 2, 3, 4, 5]
Output:
Second smallest = 2
Second largest = 4

Input: [10, 10, 10]
Output:
Second smallest = -1
Second largest = -1
```

---

## 🏁 Conclusion

| Approach      | Time Complexity | Space Complexity | Stable | Handles Duplicates | Comments                  |
|---------------|------------------|------------------|--------|---------------------|----------------------------|
| Brute Force   | O(n log n)       | O(1)             | ❌      | ❌                  | Sorting-based, unreliable |
| Better        | O(n)             | O(1)             | ✅      | ✅                  | Two loops needed          |
| Optimal       | O(n)             | O(1)             | ✅      | ✅                  | Best solution             |

---
