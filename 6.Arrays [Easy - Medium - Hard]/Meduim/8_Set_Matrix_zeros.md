# Set Matrix Zeroes

## Problem Statement

Given an `m x n` integer matrix, if an element is `0`, set its entire row and column to `0` in-place.

---

## 🧠 Brute Force Approach

### Intuition
The brute force approach marks rows and columns when a zero is encountered using a temporary value (-1), which will later be converted to 0.

### Algorithm & Approach
1. Traverse through the matrix.
2. If `matrix[i][j] == 0`, call `markRows()` and `markCols()` to mark those rows and columns with `-1` (excluding the original 0s).
3. Traverse again and convert all `-1` values to 0.

### Dry Run

Input:  
```
1 1 1
1 0 1
1 1 1
```

After marking with -1:  
```
1 -1 1
-1 0 -1
1 -1 1
```

Final matrix:  
```
1 0 1
0 0 0
1 0 1
```

### Code
```cpp
class Solution {
public:
    void markRows(vector<vector<int>>& matrix , int m, int n, int i) {
        for(int j=0;j<m;j++) {
            if(matrix[i][j] != 0) {
                matrix[i][j] = -1;
            }
        }
    }
    void markCols(vector<vector<int>>& matrix , int m, int n, int j) {
        for(int i = 0; i < n; i++ ) {
            if(matrix[i][j] != 0) {
                matrix[i][j] = -1;
            }
        }
    }
    void setZeroes(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        for(int i = 0; i < n; i++) { 
            for(int j = 0; j < m; j++) {
                if(matrix[i][j] == 0) {
                    markRows(matrix, m, n, i);
                    markCols(matrix, m, n, j);
                }
            }
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (matrix[i][j] == -1) {
                    matrix[i][j] = 0;
                }
            }
        }
    }
};
```

### Time & Space Complexity

- Time: `O(N*M*(N+M))` ≈ `O(N³)` due to nested traversal and marking rows/cols
- Space: `O(1)` (no extra space apart from constants)

---

## 💡 Better Approach

### Intuition
Store the row and column indices separately when a zero is found. In the second pass, zero out the cells that belong to those rows or columns.

### Algorithm & Approach
1. Use two arrays `row[]` and `col[]` to mark rows and columns to be zeroed.
2. First pass: mark `row[i] = 1` and `col[j] = 1` where `matrix[i][j] == 0`.
3. Second pass: if either `row[i]` or `col[j]` is `1`, set `matrix[i][j] = 0`.

### Dry Run

Input:  
```
1 1 1
1 0 1
1 1 1
```

Row and col arrays after first pass:  
```
row = [0, 1, 0]
col = [0, 1, 0]
```

Final matrix:  
```
1 0 1
0 0 0
1 0 1
```

### Code
```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        vector<int> col(m, 0);
        vector<int> row(n, 0);

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(matrix[i][j] == 0) {
                    row[i] = 1;
                    col[j] = 1;
                }
            }
        }

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(row[i] || col[j]) {
                    matrix[i][j] = 0;
                }
            }
        }
    }
};
```

### Time & Space Complexity

- Time: `O(N*M)` for both passes
- Space: `O(N + M)` for row and col arrays

---

## 🚀 Optimal Approach

### Intuition
Instead of using extra space, we can use the first row and column of the matrix itself to mark the zeroing information.

### Algorithm & Approach
1. Use `matrix[0][j]` and `matrix[i][0]` to mark zeros for rows and columns.
2. Use a variable `col0` to track if the first column needs to be zeroed.
3. First pass: fill markers.
4. Second pass: update matrix based on markers.
5. Third pass: update the first row and column if needed.

### Dry Run

Input:  
```
1 1 1
1 0 1
1 1 1
```

After marking:  
```
1 1 1
0 0 1
1 1 1
```

Final matrix:  
```
1 0 1
0 0 0
1 0 1
```

### Code
```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        int col0 = 1;
        for(int i = 0; i < n; i++ ) {
            for(int j = 0;j < m; j++) {
                if(matrix[i][j] == 0) {
                    matrix[i][0] = 0;
                    if(j != 0)
                        matrix[0][j] = 0;
                    else 
                        col0 = 0;
                }
            }
        }

        for(int i = 1; i < n; i++) {
            for(int j = 1; j < m; j++) {
                if(matrix[i][j] != 0) {
                    if(matrix[0][j] == 0 || matrix[i][0] == 0) {
                        matrix[i][j] = 0;
                    }
                }
            }
        }

        if(matrix[0][0] == 0) {
            for(int j = 0; j < m ; j++) matrix[0][j] = 0;
        }
        if(col0 == 0) {
            for(int i = 0; i < n ; i++) matrix[i][0] = 0;
        }
    }
};
```

### Time & Space Complexity

- Time: `O(N*M)` (single pass to mark + single pass to update)
- Space: `O(1)` (in-place using matrix itself)