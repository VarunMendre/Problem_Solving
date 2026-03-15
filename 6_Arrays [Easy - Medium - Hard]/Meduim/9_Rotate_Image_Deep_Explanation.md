
# Rotate Image - Deep Dive

---

## 🔁 Brute Force Code Explanation


### 1. Problem Statement
You're given a square `n x n` 2D matrix. You must rotate it **90 degrees clockwise**. Return the new rotated matrix.

### 2. Intuition
In a 90° clockwise rotation, the element at position `(i, j)` in the original matrix goes to `(j, n-1-i)` in the rotated matrix.

### 3. Approach / Thought Process
- We need to **preserve the original matrix**.
- So, we create a **new matrix** of the same dimensions to store the rotated values.
- We scan the original matrix and copy each element into its rotated position in the new matrix using the formula:  `ans[j][n-1-i] = matrix[i][j]`.

### 4. Algorithm
1. Create a new matrix `ans` of size `n x n` initialized with 0.
2. Loop through all indices `(i, j)` of the input matrix.
3. Set `ans[j][n - 1 - i] = matrix[i][j]`.
4. Return `ans`.

### 5. Dry Run
**Input:**
```
1 2 3
4 5 6
7 8 9
```
**Resulting Matrix:**
```
7 4 1
8 5 2
9 6 3
```

### 6. Code
```cpp
class Solution {
public:
    vector<vector<int>> rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<vector<int>> ans(n, vector<int>(n, 0));

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n; j++) {
                ans[j][n - 1 - i] = matrix[i][j];
            }
        }
        return ans;
    }
};
```

### 7. Time & Space Complexity
- **Time Complexity:** `O(N^2)`
- **Space Complexity:** `O(N^2)`

---

## ✅ Optimal In-Place Code Explanation


### 1. Problem Statement
Rotate a given `n x n` matrix by 90° clockwise. But now, do it **in-place** (i.e., without using extra space).

### 2. Intuition
We observe that a clockwise rotation can be achieved in two steps:
- **Transpose the matrix** (convert rows to columns).
- **Reverse each row** of the transposed matrix.

### 3. Approach / Thought Process
- **Transpose Step:** Swap every element at `(i, j)` with `(j, i)` if `j > i`.
- **Reverse Step:** After transposing, reverse each row of the matrix.

### 4. Algorithm
1. **Transpose:**
   - For every `i` from `0` to `n-1`:
     - For every `j` from `i+1` to `n-1`:
       - Swap `matrix[i][j]` with `matrix[j][i]`
2. **Reverse Rows:**
   - For each row in matrix, reverse it.

### 5. Dry Run
**Input:**
```
1 2 3
4 5 6
7 8 9
```
**Step 1: Transpose**
```
1 4 7
2 5 8
3 6 9
```
**Step 2: Reverse each row**
```
7 4 1
8 5 2
9 6 3
```

### 6. Code
```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Transpose
        for(int i = 0; i < n - 1; i++) {
            for(int j = i + 1; j < n; j++) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }

        // Reverse rows
        for(int i = 0; i < n; i++) {
            reverse(matrix[i].begin(), matrix[i].end());
        }
    }
};
```

### 7. Time & Space Complexity
- **Time Complexity:** `O(N^2)`
- **Space Complexity:** `O(1)`
