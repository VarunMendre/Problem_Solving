# Spiral Matrix Traversal

## 1. Problem Statement

Given an `m x n` matrix, return **all elements** of the matrix in **spiral order**.

### Example:

Input:
```
matrix = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]
```

Output:
```
[1, 2, 3, 6, 9, 8, 7, 4, 5]
```

---

## 2. Intuition

The idea is to simulate the process of going in a spiral: start from the top-left corner of the matrix, move right until the end, then move down the last column, then move left on the bottom row, then move up on the first column — and repeat this pattern inward until all elements are visited.

You maintain boundaries (`top`, `bottom`, `left`, and `right`) and iterate within them in the correct spiral order.

---

## 3. Approach

- Initialize four boundaries: `top`, `bottom`, `left`, and `right`.
- Traverse in the spiral direction:
  - Left to Right on the top row.
  - Top to Bottom on the rightmost column.
  - Right to Left on the bottom row.
  - Bottom to Top on the leftmost column.
- After completing each direction, shrink the respective boundary.
- Continue while `top <= bottom` and `left <= right`.

This ensures that we move layer by layer, inward.

---

## 4. Algorithm

1. Initialize `top = 0`, `bottom = n - 1`, `left = 0`, `right = m - 1`.
2. While `top <= bottom` and `left <= right`:
    - Traverse from `left` to `right` along `top` row. Increment `top`.
    - Traverse from `top` to `bottom` along `right` column. Decrement `right`.
    - If `top <= bottom`, traverse from `right` to `left` along `bottom` row. Decrement `bottom`.
    - If `left <= right`, traverse from `bottom` to `top` along `left` column. Increment `left`.
3. Repeat until all elements are added.

---

## 5. Dry Run

### Input Matrix:
```
[[1, 2, 3],
 [4, 5, 6],
 [7, 8, 9]]
```

### Step-by-Step:

- **Initial Boundaries**: `top=0`, `bottom=2`, `left=0`, `right=2`

1. **Left to Right (top row 0)**: 1, 2, 3 → increment `top = 1`
2. **Top to Bottom (right col 2)**: 6, 9 → decrement `right = 1`
3. **Right to Left (bottom row 2)**: 8, 7 → decrement `bottom = 1`
4. **Bottom to Top (left col 0)**: 4 → increment `left = 1`
5. **Left to Right (top row 1)**: 5 → increment `top = 2`

### Final Result: `[1, 2, 3, 6, 9, 8, 7, 4, 5]`

---

## 6. Code

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        int top = 0, bottom = n - 1;
        int left = 0, right = m - 1;

        vector<int> ans;
       
        while (left <= right && top <= bottom) {
            // Move right
            for (int i = left; i <= right; i++) {
                ans.push_back(matrix[top][i]);
            }
            top++;

            // Move down
            for (int i = top; i <= bottom; i++) {
                ans.push_back(matrix[i][right]);
            }
            right--;

            // Move left
            if (top <= bottom) {
                for (int i = right; i >= left; i--) {
                    ans.push_back(matrix[bottom][i]);
                }
                bottom--;
            }

            // Move up
            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    ans.push_back(matrix[i][left]);
                }
                left++;
            }    
        }
        return ans;
    }
};
```

---

## 7. Time and Space Complexity

### Time Complexity:
- `O(m * n)` where `m` is the number of rows and `n` is the number of columns.
- Every element is visited exactly once.

### Space Complexity:
- `O(m * n)` for storing the result in a separate list `ans`.
- If output storage isn’t counted, then `O(1)` extra space is used.