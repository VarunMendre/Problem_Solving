
# 6. Left Rotate Array by D Places

## Problem Statement

Given an array `arr` with `n` elements, the task is to rotate the array to the **left** by `k` steps, where `k` is non-negative.

### Example:

```
Input:
arr = [1, 2, 3, 4, 5]
k = 2

Output:
[3, 4, 5, 1, 2]
```

## Brute Force Approach

### Approach:
1. Copy the first `k` elements to a temporary array.
2. Shift the remaining `n-k` elements to the left.
3. Place the copied elements from temp at the end of the array.

### Why It Works:
- By preserving the first `k` elements and moving the rest forward, we simulate the left rotation.
- Simple and easy to implement for small `k`.

### Code:
```cpp
vector<int> rotateArray(vector<int> arr, int k) {
    int n = arr.size();
    k = k % n;
    vector<int> temp(k);

    for (int i = 0; i < k; i++) {
        temp[i] = arr[i];
    }

    for (int i = k; i < n; i++) {
        arr[i - k] = arr[i];
    }

    for (int i = n - k; i < n; i++) {
        arr[i] = temp[i - (n - k)];
    }

    return arr;
}
```

### Time Complexity:
- O(k) for copying
- O(n-k) for shifting
- O(k) for inserting
- **Total: O(n + k)**

### Space Complexity:
- **O(k)** for the temporary array

---

## Optimal Approach

### Approach:
1. Reverse the first `k` elements.
2. Reverse the remaining `n-k` elements.
3. Reverse the entire array.

### Why It Works:
- Reversing subarrays allows us to rotate elements in-place.
- The three-step reversal transforms the array into the correct rotated state.

### Code360 Code (Left Rotation):
```cpp
#include<bits/stdc++.h>
#include<iostream>
using namespace std;

vector<int> rotateArray(vector<int> arr, int k) {
    int n = arr.size();
    k = k % n;

    reverse(arr.begin(), arr.begin() + k);
    reverse(arr.begin() + k, arr.end());
    reverse(arr.begin(), arr.end());

    return arr;
}
```

### Leetcode Code (Right Rotation):
```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k = k % n;

        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};
```

### External Reverse Function (If Asked in Interview):
```cpp
void reverseArray(vector<int>& nums, int start, int end) {
    while (start < end) {
        swap(nums[start], nums[end]);
        start++;
        end--;
    }
}
```

### Time Complexity:
- O(n) – Each reverse operation takes linear time

### Space Complexity:
- O(1) – In-place operations, no extra space used

---

## Summary

| Approach      | Time Complexity | Space Complexity | Notes                        |
|---------------|------------------|------------------|------------------------------|
| Brute Force   | O(n + k)         | O(k)             | Good for small `k`           |
| Optimal       | O(n)             | O(1)             | Efficient for all cases      |
