# 3. Check if Array is Sorted

## Problem Statement
You are given an array of integers. Your task is to determine whether the array is sorted in **non-decreasing order** (i.e., each element is greater than or equal to the one before it).

## Function Signature
```cpp
int isSorted(int n, vector<int> a);
```
- `n`: Size of the array
- `a`: The array of integers

## Code
```cpp
int isSorted(int n, vector<int> a) {
    // Write your code here.

    for(int i=1; i<n; i++) {
        if(a[i] >= a[i-1]) {

        }
        else {
            return 0;
        }
    }
    return 1;
}
```

## How it Works
- We loop through the array starting from index `1` to `n-1`.
- At each step, we compare the current element `a[i]` with the previous element `a[i-1]`.
- If we find any case where `a[i] < a[i-1]`, it means the array is **not** in non-decreasing order, so we return `0`.
- If the loop completes without finding such a case, the array is sorted, and we return `1`.

## Time and Space Complexity
- **Time Complexity**: `O(n)` — We traverse the array once.
- **Space Complexity**: `O(1)` — We use a constant amount of space.

## Example
### Input
```cpp
n = 5
a = {1, 2, 2, 4, 5}
```
### Output
```cpp
1
```
### Explanation
The array is in non-decreasing order, so the function returns `1`.

### Input
```cpp
n = 5
a = {1, 3, 2, 4, 5}
```
### Output
```cpp
0
```
### Explanation
The element `2` is less than `3`, which breaks the sorting rule.

---

✅ This function is helpful in many real-world scenarios, such as validating if a list of timestamps, prices, or scores is sorted.

---

*Feel free to use this `README.md`-style content for your GitHub repository!*
