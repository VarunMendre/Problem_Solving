
# Merge Sort Documentation

## Problem Statement

Given an array of `N` integers, implement the **Merge Sort** algorithm to sort the array in ascending order.

### Examples:

**Example 1:**
- **Input:** `N = 5`, `arr[] = {4, 2, 1, 6, 7}`
- **Output:** `1, 2, 4, 6, 7`

**Example 2:**
- **Input:** `N = 7`, `arr[] = {3, 2, 8, 5, 1, 4, 23}`
- **Output:** `1, 2, 3, 4, 5, 8, 23`

---

## Algorithm Explanation

**Merge Sort** is a **Divide and Conquer** algorithm.

- It divides the array into two halves recursively until the subarrays have only one element.
- Then it merges the sorted subarrays to produce a fully sorted array.

### Two main functions:
1. `mergeSort()` – divides the array into two halves.
2. `merge()` – merges two sorted halves into a single sorted array.

### Step-by-step:
- **Divide** the array into halves until each part has only one element.
- **Merge** each half by comparing elements and arranging them in sorted order.

---

## Pseudocode

```
function mergeSort(arr, low, high):
    if low >= high:
        return
    mid = (low + high) // 2
    mergeSort(arr, low, mid)
    mergeSort(arr, mid + 1, high)
    merge(arr, low, mid, high)

function merge(arr, low, mid, high):
    create temp array
    left = low
    right = mid + 1

    while left <= mid and right <= high:
        if arr[left] <= arr[right]:
            temp.push(arr[left])
            left++
        else:
            temp.push(arr[right])
            right++

    while left <= mid:
        temp.push(arr[left])
        left++

    while right <= high:
        temp.push(arr[right])
        right++

    for i from low to high:
        arr[i] = temp[i - low]
```

---

## Dry Run

**Input:** `arr[] = {4, 2, 1, 6, 7}`  

### First Call:
`mergeSort(arr, 0, 4)`  
- Mid = 2 → Split into [0,2] and [3,4]

#### Left Half:
`mergeSort(arr, 0, 2)`
- Mid = 1 → [0,1] and [2,2]

`mergeSort(arr, 0, 1)`
- Mid = 0 → [0,0] and [1,1]
- Merge [4] and [2] → [2,4]

Now merge [2,4] and [1]  
- Merge [2,4] and [1] → [1,2,4]

#### Right Half:
`mergeSort(arr, 3, 4)` → Merge [6] and [7] → [6,7]

### Final Merge:
Merge [1,2,4] and [6,7] → [1,2,4,6,7]

✅ Final output: `1, 2, 4, 6, 7`

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

void merge(vector<int> &arr , int low , int mid , int high){
    vector<int> temp;

    int left = low;
    int right = mid + 1;

    while(left <= mid && right <= high){
        if(arr[left] <= arr[right]){
            temp.push_back(arr[left++]);
        }
        else{
            temp.push_back(arr[right++]);
        }
    }

    while(left <= mid){
        temp.push_back(arr[left++]);
    }

    while(right <= high){
        temp.push_back(arr[right++]);
    }

    for(int i = low; i <= high; i++){
        arr[i] = temp[i - low];
    }
}

void ms(vector<int> &arr, int low, int high){
    if(low >= high) return;

    int mid = (low + high) / 2;
    ms(arr, low, mid);
    ms(arr, mid + 1, high);
    merge(arr, low, mid, high);
}

void mergeSort(vector<int> &arr, int n) {
    ms(arr, 0, n - 1);
}
```

---

## Time & Space Complexity

| Case        | Time Complexity | Explanation                              |
|-------------|------------------|------------------------------------------|
| Best        | O(N log N)       | Array always splits into halves          |
| Average     | O(N log N)       | Recursive splitting + merging            |
| Worst       | O(N log N)       | Still follows divide and conquer         |
| Space       | O(N)             | Due to the temporary array used in merge |

---

## Problem Link

Try this problem on [Coding Ninjas](https://www.naukri.com/code360/problems/merge-sort_5846)
