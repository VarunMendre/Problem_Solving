# 6. Next Permutation

## 1. Problem Statement

Given an array of integers `nums`, find the next lexicographically greater permutation of numbers.
If such an arrangement is not possible, rearrange it as the lowest possible order (i.e., sorted in ascending order).

You must do this **in-place** and use only **constant extra memory**.

---

## 🔹 Brute Force Approach

### 2. Intuition
Try out all possible permutations and look for the one that comes next after the given array.

### 3. Algorithm
1. Generate all permutations of the array.
2. Sort all permutations to get them in lexicographical order.
3. Traverse the list of permutations and find the given array.
4. Return the next permutation in the list.

### 4. Dry Run
Input: [1, 2, 3]

All permutations:  
[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]

Input is [1, 2, 3] → Next permutation is [1, 3, 2]

### 5. Code

```cpp
// Not implemented explicitly to avoid TLE, but conceptually:
// vector<vector<int>> perms = all_permutations(nums);
// sort(perms.begin(), perms.end());
// find the given permutation and return the next one
```

### 6. Time & Space Complexity
- **Time**: O(N! * N) — Generating all permutations and comparing
- **Space**: O(N! * N) — Storing all permutations

---

## 🔹 Better Approach (Using C++ STL)

### 2. Intuition
Use C++ STL's built-in function `next_permutation()` that does exactly what the problem asks.

### 3. Algorithm
1. Call `next_permutation(arr, arr + n)`.
2. It rearranges the array in-place to the next lexicographical permutation.

### 4. Dry Run
Input: [1, 2, 3]

Using STL: `next_permutation(arr, arr+3)` → arr becomes [1, 3, 2]

### 5. Code

```cpp
#include <algorithm>
#include <iostream>
using namespace std;

int main() {
    int arr[] = {1, 2, 3};
    next_permutation(arr, arr + 3);
    for (int i = 0; i < 3; i++) cout << arr[i] << " ";
    return 0;
}
```

### 6. Time & Space Complexity
- **Time**: O(2N)
- **Space**: O(1)

---

## 🔹 Optimal Approach (In-place Algorithm)

### 2. Intuition
To find the next permutation:
- Find the first decreasing number from the end.
- Swap it with the just larger number to its right.
- Reverse the suffix to make it the smallest possible.

### 3. Algorithm
1. Traverse from right to left, find the first `i` such that `nums[i] < nums[i+1]`.
2. If no such `i` exists, reverse the entire array.
3. Otherwise, find the largest index `j > i` such that `nums[j] > nums[i]`.
4. Swap `nums[i]` and `nums[j]`.
5. Reverse the subarray from `i+1` to the end.

### 4. Dry Run
Input: [1, 2, 3]

- i = 1, nums[1] = 2 < nums[2] = 3 → Found pivot at index 1
- j = 2, nums[2] = 3 > nums[1] = 2 → Swap → [1, 3, 2]
- Reverse from index 2 to end → Already sorted

Output: [1, 3, 2]

### 5. Code

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int n = nums.size();
        int index = -1;

        for(int i = n - 2; i >= 0; i--) {
            if(nums[i] < nums[i + 1]) {
                index = i;
                break;
            }
        }

        if(index == -1) {
            reverse(nums.begin(), nums.end());
            return;
        }

        for(int i = n - 1; i > index; i--) {
            if(nums[i] > nums[index]) {
                swap(nums[i], nums[index]);
                break;
            }
        }

        reverse(nums.begin() + index + 1, nums.end());
    }
};
```

### 6. Time & Space Complexity
- **Time**: O(3N) — Linear scan and reverse
- **Space**: O(1) — In-place operation