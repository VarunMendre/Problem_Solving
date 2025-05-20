# Selection Sort Documentation

## Problem Statement

Given an array of N integers, write a program to implement the Selection sorting algorithm.

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

## Approach: Selection Sort

Selection sort is a simple comparison-based sorting algorithm. The idea is to divide the array into two parts:
1. The subarray which is already sorted.
2. The remaining subarray which is unsorted.

In every iteration, we find the minimum element from the unsorted subarray and move it to the end of the sorted subarray.

### Time Complexity
- Best Case: O(N^2)
- Average Case: O(N^2)
- Worst Case: O(N^2)
- Space Complexity: O(1) — in-place sorting

---

## Dry Run

Let's dry run the code for `arr[] = {13, 46, 24, 52, 20, 9}`

### Iteration 1 (i=0):
- Minimum from index 0 to 5 is `9` at index 5
- Swap `arr[0]` and `arr[5]` → `{9, 46, 24, 52, 20, 13}`

### Iteration 2 (i=1):
- Minimum from index 1 to 5 is `13` at index 5
- Swap `arr[1]` and `arr[5]` → `{9, 13, 24, 52, 20, 46}`

### Iteration 3 (i=2):
- Minimum from index 2 to 5 is `20` at index 4
- Swap `arr[2]` and `arr[4]` → `{9, 13, 20, 52, 24, 46}`

### Iteration 4 (i=3):
- Minimum from index 3 to 5 is `24` at index 4
- Swap `arr[3]` and `arr[4]` → `{9, 13, 20, 24, 52, 46}`

### Iteration 5 (i=4):
- Minimum from index 4 to 5 is `46` at index 5
- Swap `arr[4]` and `arr[5]` → `{9, 13, 20, 24, 46, 52}`

Final Sorted Array: `{9, 13, 20, 24, 46, 52}`

---

## Code

```cpp
#include <bits/stdc++.h> 
void selectionSort(vector<int>& arr, int n)
{   
    // Write your code here.
    for(int i=0;i<=n-2;i++){
        int mini = i;

        for(int j=i;j<=n-1;j++){
            if(arr[j] < arr[mini]){
                mini = j;
            }
        }
        swap(arr[mini], arr[i]);
    }
}
```