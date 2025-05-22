# Insertion Sort Documentation

## Problem Statement

Given an array of `N` integers, write a program to implement the Insertion Sort algorithm to sort the array in ascending order.

### Examples

**Example 1:**
- Input: `N = 6`, `array[] = {13, 46, 24, 52, 20, 9}`
- Output: `9, 13, 20, 24, 46, 52`
- Explanation: After sorting, the array becomes: `9, 13, 20, 24, 46, 52`

**Example 2:**
- Input: `N = 5`, `array[] = {5, 4, 3, 2, 1}`
- Output: `1, 2, 3, 4, 5`
- Explanation: After sorting, the array becomes: `1, 2, 3, 4, 5`

---

## Approach: Insertion Sort

Insertion Sort works similarly to how we sort playing cards in our hands. The array is virtually split into a sorted and an unsorted part. Elements from the unsorted part are picked and placed at the correct position in the sorted part.

### Steps:
1. Start from index 1 (second element), and assume the first element is already sorted.
2. Compare the current element with the previous elements.
3. Shift all larger elements one position to the right.
4. Insert the current element at the correct position.
5. Repeat the above steps until the whole array is sorted.

---

## Time Complexity

- **Best Case (Already Sorted):** O(N)
  - Only one comparison per element, no swaps.
- **Average Case:** O(N²)
  - Each element is compared with half of the sorted array on average.
- **Worst Case (Reversed Array):** O(N²)
  - Every element has to be compared and moved to the beginning.
- **Space Complexity:** O(1)
  - In-place sorting (no extra space used).

---

## Dry Run

Let's dry run the code for `arr[] = {13, 46, 24, 52, 20, 9}`

### Iteration 1 (i = 1):
- `j = 1` → `arr[0] = 13`, `arr[1] = 46` → no swap → `{13, 46, 24, 52, 20, 9}`

### Iteration 2 (i = 2):
- `j = 2` → `arr[1] = 46`, `arr[2] = 24` → swap → `{13, 24, 46, 52, 20, 9}`

### Iteration 3 (i = 3):
- `j = 3` → `arr[2] = 46`, `arr[3] = 52` → no swap → `{13, 24, 46, 52, 20, 9}`

### Iteration 4 (i = 4):
- `j = 4` → `arr[3] = 52`, `arr[4] = 20` → swap → `{13, 24, 46, 20, 52, 9}`
- `j = 3` → `arr[2] = 46`, `arr[3] = 20` → swap → `{13, 24, 20, 46, 52, 9}`
- `j = 2` → `arr[1] = 24`, `arr[2] = 20` → swap → `{13, 20, 24, 46, 52, 9}`
- `j = 1` → `arr[0] = 13`, `arr[1] = 20` → no swap

### Iteration 5 (i = 5):
- `j = 5` → `arr[4] = 52`, `arr[5] = 9` → swap → `{13, 20, 24, 46, 9, 52}`
- `j = 4` → `arr[3] = 46`, `arr[4] = 9` → swap → `{13, 20, 24, 9, 46, 52}`
- `j = 3` → `arr[2] = 24`, `arr[3] = 9` → swap → `{13, 20, 9, 24, 46, 52}`
- `j = 2` → `arr[1] = 20`, `arr[2] = 9` → swap → `{13, 9, 20, 24, 46, 52}`
- `j = 1` → `arr[0] = 13`, `arr[1] = 9` → swap → `{9, 13, 20, 24, 46, 52}`

Final Sorted Array: `{9, 13, 20, 24, 46, 52}`

---

## Code

```cpp
#include <bits/stdc++.h> 
void insertionSort(int n, vector<int> &arr){
    for(int i = 0; i <= n-1; i++){
        int j = i;
        while(j > 0 && arr[j-1] > arr[j]){
            swap(arr[j-1], arr[j]);
            j--;
        }
    }
}
```

---

## Problem Link

You can refer to this problem on [Coding Ninjas (Naukri)](https://www.naukri.com/code360/problems/insertion-sort_3155179?leftPanelTabValue=PROBLEM)
