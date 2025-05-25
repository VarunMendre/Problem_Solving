
# Quick Sort

## 1. Information & Intuition

Quick Sort is a **divide-and-conquer** sorting algorithm that works by selecting a **pivot** element and partitioning the array such that:
- Elements smaller than the pivot go to the left
- Elements greater than the pivot go to the right

This process is then **recursively** applied to the left and right subarrays. It's widely used because of its **average-case efficiency**.

> The key idea: **Choose pivot → Partition → Recur on both sides**

Quick Sort is **not stable** but is **in-place** (no extra space used, except recursion stack).

---

## 2. Pseudocode

```
function quickSort(arr, low, high):
    if low < high:
        pIndex = partition(arr, low, high)
        quickSort(arr, low, pIndex - 1)
        quickSort(arr, pIndex + 1, high)

function partition(arr, low, high):
    pivot = arr[low]
    i = low
    j = high
    while i < j:
        while arr[i] <= pivot and i <= high - 1:
            i++
        while arr[j] > pivot and j >= low + 1:
            j--
        if i < j:
            swap(arr[i], arr[j])
    swap(arr[low], arr[j])
    return j
```

---

## 3. Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[low];
    int i = low;
    int j = high;

    while(i < j) {
        while(arr[i] <= pivot && i <= high - 1) i++;
        while(arr[j] > pivot && j >= low + 1) j--;

        if(i < j) swap(arr[i], arr[j]);
    }
    swap(arr[low], arr[j]);

    return j;
}

void q_s(vector<int>& arr, int low, int high) {
    if(low < high) {
        int pIndex = partition(arr, low, high);
        q_s(arr, low, pIndex - 1);
        q_s(arr, pIndex + 1, high);
    }
}

vector<int> quickSort(vector<int> arr) {
    q_s(arr, 0, arr.size() - 1);
    return arr;
}
```

---

## 4. Time & Space Complexity

| Case       | Time Complexity | Explanation                  |
|------------|------------------|------------------------------|
| Best Case  | O(n log n)       | Balanced partitions          |
| Average    | O(n log n)       | Random pivot gives log splits |
| Worst Case | O(n²)            | Already sorted or skewed pivot |

### Space Complexity:
- **O(1)** (excluding recursion stack)
- **O(log n)** recursive calls in average case
- **O(n)** in worst case due to stack depth

---
