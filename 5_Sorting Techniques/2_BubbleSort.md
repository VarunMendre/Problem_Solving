# Bubble Sort Documentation

## Problem Statement

Given an array of N integers, write a program to implement the Bubble sorting algorithm.

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

## Approach: Bubble Sort

Bubble Sort is a simple sorting algorithm that repeatedly steps through the list, compares adjacent elements and swaps them if they are in the wrong order. The pass through the list is repeated until the list is sorted.

### Time Complexity
- Best Case: O(N) — when the array is already sorted
- Average Case: O(N²)
- Worst Case: O(N²)
- Space Complexity: O(1) — in-place sorting

---

## Dry Run

Let's dry run the code for `arr[] = {13, 46, 24, 52, 20, 9}`

### Iteration 1 (i=5):
- Compare `13` and `46` → no swap
- Compare `46` and `24` → swap → `{13, 24, 46, 52, 20, 9}`
- Compare `46` and `52` → no swap
- Compare `52` and `20` → swap → `{13, 24, 46, 20, 52, 9}`
- Compare `52` and `9` → swap → `{13, 24, 46, 20, 9, 52}`

### Iteration 2 (i=4):
- Compare `13` and `24` → no swap
- Compare `24` and `46` → no swap
- Compare `46` and `20` → swap → `{13, 24, 20, 46, 9, 52}`
- Compare `46` and `9` → swap → `{13, 24, 20, 9, 46, 52}`

### Iteration 3 (i=3):
- Compare `13` and `24` → no swap
- Compare `24` and `20` → swap → `{13, 20, 24, 9, 46, 52}`
- Compare `24` and `9` → swap → `{13, 20, 9, 24, 46, 52}`

### Iteration 4 (i=2):
- Compare `13` and `20` → no swap
- Compare `20` and `9` → swap → `{13, 9, 20, 24, 46, 52}`

### Iteration 5 (i=1):
- Compare `13` and `9` → swap → `{9, 13, 20, 24, 46, 52}`

Final Sorted Array: `{9, 13, 20, 24, 46, 52}`

---

## Code

```cpp
#include <bits/stdc++.h> 
void bubbleSort(vector<int>& arr, int n)
{   
    // Write your code here.
    for(int i=n-1;i>=1;i--){
        for(int j=0;j<=i-1;j++){
            if(arr[j] > arr[j+1]){
                swap(arr[j],arr[j+1]);
            }
        }
    }
}
```

---

## Problem Link

You can refer to this problem on [Coding Ninjas (Naukri)](https://www.naukri.com/code360/problems/bubble-sort_980524?leftPanelTabValue=SUBMISSION)
