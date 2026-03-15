
# 5. Left Rotate Array by One

## Problem Statement

Given an array `arr` containing `n` elements, rotate this array left **once** and return it.

Rotating the array left by one means shifting all elements by one place to the left and moving the first element to the last position in the array.

---

## Example

### Input:
```
a = 5  
arr = [1, 2, 3, 4, 5]
```

### Output:
```
[2, 3, 4, 5, 1]
```

### Explanation:  
We moved the 2nd element to the 1st position, the 3rd to the 2nd, the 4th to the 3rd, the 5th to the 4th, and the 1st element to the 5th position.

---

## Sample Input 1:
```
4  
5 7 3 2
```

### Sample Output 1:
```
7 3 2 5
```

### Explanation:
Move the first element to the last and shift the rest of the elements one step to the left.

---

## Sample Input 2:
```
5  
4 0 3 2 5
```

### Sample Output 2:
```
0 3 2 5 4
```

### Explanation:
Same as above — shift all elements left and move the first element to the last.

---

## Expected Time Complexity:
- **O(n)**, where `n` is the size of the input array `arr`.

---

## Constraints:
- `1 <= n <= 10^5`  
- `1 <= arr[i] <= 10^9`  
- Time Limit: **1 sec**

---

## Solution (C++)

```cpp
#include <bits/stdc++.h> 
using namespace std;

vector<int> rotateArray(vector<int>& arr, int n) {
    int temp = arr[0]; // Store the first element
    for(int i = 1; i < n; i++) {
        arr[i - 1] = arr[i]; // Shift elements to the left
    }
    arr[n - 1] = temp; // Place the first element at the end
    return arr;
}
```
