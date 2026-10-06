# Minimum Falling Path Sum

A detailed breakdown of four approaches to solve the **Minimum Falling Path Sum** problem:
1. Pure Recursion
2. Top-Down Dynamic Programming (Memoization)
3. Bottom-Up Dynamic Programming (Tabulation)
4. Space-Optimized Dynamic Programming

---

## 1. Recursion

### 1. Approach / Intuition
The core idea is to explore every possible valid falling path starting from any element in the first row ($row = 0$) down to the last row ($row = n-1$). At each cell $(row, col)$, we can fall down to three potential destination cells in the next row:
- Directly down-left: $(row + 1, col - 1)$
- Directly down: $(row + 1, col)$
- Directly down-right: $(row + 1, col + 1)$

The path sum from $(row, col)$ is the value of the current cell `matrix[row][col]` plus the minimum of the paths resulting from those 3 branches.

### 2. Why We Think About This
Since we need to try all possible choices at each step to find the minimum sum path, recursion naturally fits as a starting point. It enables us to model the decision tree of choosing between left-down, straight-down, and right-down paths.

### 3. Dry-Run
Consider a simple $3 \times 3$ matrix:
```text
[ [2, 1, 3],
  [6, 5, 4],
  [7, 8, 9] ]
```

Starting from `(0, 1)` with value `1`:
1. `solve(0, 1)` computes `matrix[0][1] + min(solve(1, 0), solve(1, 1), solve(1, 2))`
2. `solve(1, 0)` computes `matrix[1][0] + min(solve(2, -1), solve(2, 0), solve(2, 1))`
   - `solve(2, -1)` returns `INT_MAX` (Out of Bounds).
   - `solve(2, 0)` reaches base row ($row = 2$), returns `matrix[2][0] = 7`.
   - `solve(2, 1)` reaches base row ($row = 2$), returns `matrix[2][1] = 8`.
   - Minimum is $\min(\infty, 7, 8) = 7$. So `solve(1, 0) = 6 + 7 = 13`.
3. Similar recursive steps are performed for `solve(1, 1)` and `solve(1, 2)`.
4. Finally, `solve(0, 1)` combines the optimal sub-results and returns $1 + 12 = 13$.

### 4. Story Points
- **The Starting Point:** Try every starting position in row 0.
- **The Journey:** From row $r$, step into $r+1$ choosing left-diagonal, directly down, or right-diagonal.
- **Out-of-Bounds Penalty:** Stepping off the grid returns `INT_MAX` to render that invalid path infinitely expensive.
- **Destination:** Reaching row $n-1$ returns the cell's own value.

### 5. Code
```cpp
class Solution {
private:
    int solve(int row, int col, int n, vector<vector<int>>& matrix) {
        // Base Case 1: Out of bounds check
        if (col < 0 || col >= n || row < 0 || row >= n)
            return INT_MAX;
        
        // Base Case 2: Reached the last row
        if (row == n - 1)
            return matrix[row][col];
        
        // Explore 3 paths in the next row
        int left = solve(row + 1, col - 1, n, matrix);
        int down = solve(row + 1, col, n, matrix);
        int right = solve(row + 1, col + 1, n, matrix);
        
        // Return current cell value + minimum of valid sub-paths
        return matrix[row][col] + min({left, down, right});
    }
public:
    int minFallingPathSum(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int ans = INT_MAX;
        
        // Try starting from each column in the first row
        for (int col = 0; col < n; col++) {
            ans = min(ans, solve(0, col, n, matrix));
        }
        return ans;
    }
};
```

### 6. Time & Space Complexity
- **Time Complexity:** $\mathcal{O}(n \cdot 3^n)$ — There are $n$ starting columns, and for each column, we branch 3 times at every row level (depth $n$).
- **Space Complexity:** $\mathcal{O}(n)$ — Recursion stack depth goes up to $n$ rows.

---

## 2. Memoization (Top-Down DP)

### 1. Approach / Intuition
In pure recursion, many subproblems are recalculated multiple times. Memoization stores the calculated minimum path sum for any cell `(row, col)` in a 2D `dp` table. If we encounter `(row, col)` again, we reuse the precomputed answer in $\mathcal{O}(1)$ time.

### 2. Why We Think About This
The recursive execution tree contains **overlapping subproblems**. For instance, `solve(0, 0)` and `solve(0, 2)` may both end up calling `solve(1, 1)`. Storing previously computed values avoids redundant computations.

### 3. Dry-Run
Matrix:
```text
[ [2, 1, 3],
  [6, 5, 4],
  [7, 8, 9] ]
```
1. Initialize `dp` table of size $n \times n$ filled with `INT_MAX`.
2. When evaluating `solve(0, 0)`, we compute `solve(1, 0)` and `solve(1, 1)`.
3. `solve(1, 1)` calculates its answer (which is $5 + \min(7, 8, 9) = 12$) and stores `dp[1][1] = 12`.
4. Later, when evaluating `solve(0, 2)`, it asks for `solve(1, 1)`.
5. Since `dp[1][1] != INT_MAX`, it immediately returns $12$ without re-evaluating sub-trees.

### 4. Story Points
- **Memory Check:** Before computing, ask: *"Have I solved this cell before?"*
- **Save Result:** Once computed, record the result in `dp[row][col]`.
- **Reuse:** Future branches visiting the same cell fetch the stored answer directly.

### 5. Code
```cpp
class Solution {
private:
    int solve(int row, int col, int n, vector<vector<int>>& dp,
              vector<vector<int>>& matrix) {
        // Base Case 1: Out of bounds check
        if (col < 0 || col >= n || row < 0 || row >= n)
            return INT_MAX;
        
        // Base Case 2: Reached the last row
        if (row == n - 1)
            return matrix[row][col];
        
        // Return saved state if already computed
        if (dp[row][col] != INT_MAX)
            return dp[row][col];
        
        // Explore 3 paths in the next row
        int left = solve(row + 1, col - 1, n, dp, matrix);
        int down = solve(row + 1, col, n, dp, matrix);
        int right = solve(row + 1, col + 1, n, dp, matrix);
        
        // Memoize and return
        return dp[row][col] = matrix[row][col] + min({left, down, right});
    }
public:
    int minFallingPathSum(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int ans = INT_MAX;
        
        // DP table initialized to INT_MAX
        vector<vector<int>> dp(n, vector<int>(n, INT_MAX));
        
        for (int col = 0; col < n; col++) {
            ans = min(ans, solve(0, col, n, dp, matrix));
        }
        return ans;
    }
};
```

### 6. Time & Space Complexity
- **Time Complexity:** $\mathcal{O}(n^2)$ — Each cell `(row, col)` is solved at most once.
- **Space Complexity:** $\mathcal{O}(n^2) + \mathcal{O}(n) = \mathcal{O}(n^2)$ — $\mathcal{O}(n^2)$ for the 2D DP array plus $\mathcal{O}(n)$ recursion call stack depth.

---

## 3. Tabulation (Bottom-Up DP)

### 1. Approach / Intuition
Instead of computing top-down from row $0$ to row $n-1$ using recursion, we build the solution from row $0$ down to row $n-1$ iteratively. 
For any cell `dp[row][col]`, its minimum falling path sum depends on the row above it:
- `dp[row - 1][col - 1]`
- `dp[row - 1][col]`
- `dp[row - 1][col + 1]`

### 2. Why We Think About This
Tabulation eliminates the recursion call stack overhead altogether. Since dependencies move row-by-row top-down (or bottom-up), an iterative nested loop over rows and columns is straightforward and fast.

### 3. Dry-Run
Matrix:
```text
[ [2, 1, 3],
  [6, 5, 4],
  [7, 8, 9] ]
```

1. Initialize `dp[0]` with `matrix[0]`: `[2, 1, 3]`.
2. Process `row = 1`:
   - `col = 0`: `matrix[1][0] + min(dp[0][0], dp[0][1])` $\rightarrow 6 + \min(2, 1) = 7$.
   - `col = 1`: `matrix[1][1] + min(dp[0][0], dp[0][1], dp[0][2])` $\rightarrow 5 + \min(2, 1, 3) = 6$.
   - `col = 2`: `matrix[1][2] + min(dp[0][1], dp[0][2])` $\rightarrow 4 + \min(1, 3) = 5$.
   - `dp[1]` becomes `[7, 6, 5]`.
3. Process `row = 2`:
   - `col = 0`: $7 + \min(7, 6) = 13$.
   - `col = 1`: $8 + \min(7, 6, 5) = 13$.
   - `col = 2`: $9 + \min(6, 5) = 14$.
   - `dp[2]` becomes `[13, 13, 14]`.
4. Answer is $\min(13, 13, 14) = 13$.

### 4. Story Points
- **Base Row Setup:** Fill row 0 of DP with matrix's row 0 values.
- **Iterative Fill:** Move row by row from top to bottom.
- **Parent Lookup:** Check valid parents in the row above (left-diagonal, straight, right-diagonal).
- **Final Result:** Find the minimum entry in the last row.

### 5. Code
```cpp
class Solution {
public:
    int minFallingPathSum(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<vector<int>> dp(n, vector<int>(n, INT_MAX));
        
        // Initialize the base row
        dp[0] = matrix[0];
        
        // Relative offsets to access previous row parents
        const int dbRow[3] = {-1, -1, -1};
        const int dbCol[3] = {-1, 0, 1};
        
        for (int row = 1; row < n; row++) {
            for (int col = 0; col < n; col++) {
                int baseParent = INT_MAX;
                for (int d = 0; d < 3; d++) {
                    int newRow = row + dbRow[d];
                    int newCol = col + dbCol[d];
                    if (newRow >= 0 && newRow < n && newCol >= 0 &&
                        newCol < n) {
                        baseParent = min(baseParent, dp[newRow][newCol]);
                    }
                }
                dp[row][col] = matrix[row][col] + baseParent;
            }
        }
        
        // Minimum value in the last row is our answer
        int ans = INT_MAX;
        for (int col = 0; col < n; col++) {
            ans = min(ans, dp[n - 1][col]);
        }
        return ans;
    }
};
```

### 6. Time & Space Complexity
- **Time Complexity:** $\mathcal{O}(n^2)$ — Iterating through an $n \times n$ matrix with constant inner work ($3$ direction checks).
- **Space Complexity:** $\mathcal{O}(n^2)$ — To store the 2D DP matrix.

---

## 4. Space Optimization

### 1. Approach / Intuition
When filling out `dp[row][col]` in Tabulation, we only ever inspect values from `dp[row - 1]`. The values from `dp[row - 2]` or higher are never needed again. Thus, we can store only the previous row in a 1D array (`prevRow`) of size $n$, alongside a temporary array (`temp`) to store values for the current row.

### 2. Why We Think About This
Storing the entire $n \times n$ grid is unnecessary memory overhead when only the immediate previous row determines current calculations.

### 3. Dry-Run
Matrix:
```text
[ [2, 1, 3],
  [6, 5, 4],
  [7, 8, 9] ]
```

1. `prevRow = [2, 1, 3]`.
2. For `row = 1`:
   - Compute `temp[0] = 6 + min(2, 1) = 7`
   - Compute `temp[1] = 5 + min(2, 1, 3) = 6`
   - Compute `temp[2] = 4 + min(1, 3) = 5`
   - Set `prevRow = [7, 6, 5]`.
3. For `row = 2`:
   - Compute `temp[0] = 7 + min(7, 6) = 13`
   - Compute `temp[1] = 8 + min(7, 6, 5) = 13`
   - Compute `temp[2] = 9 + min(6, 5) = 14`
   - Set `prevRow = [13, 13, 14]`.
4. Answer is $\min(13, 13, 14) = 13$.

### 4. Story Points
- **Single Row Buffer:** Only remember the immediately preceding row.
- **Compute and Swap:** Fill a `temp` array for the current row, then update `prevRow = temp`.
- **Optimal Space:** Reduces spatial requirement from 2D matrix down to two 1D vectors.

### 5. Code
```cpp
class Solution {
public:
    int minFallingPathSum(vector<vector<int>>& matrix) {
        int n = matrix.size();
        
        // Vector storing values of the previous row
        vector<int> prevRow = matrix[0];
        const int dbCol[3] = {-1, 0, 1};
        
        for (int row = 1; row < n; row++) {
            vector<int> temp(n, INT_MAX);
            for (int col = 0; col < n; col++) {
                int baseParent = INT_MAX;
                for (int d = 0; d < 3; d++) {
                    int newCol = col + dbCol[d];
                    if (newCol >= 0 && newCol < n) {
                        baseParent = min(baseParent, prevRow[newCol]);
                    }
                }
                temp[col] = matrix[row][col] + baseParent;
            }
            // Move temp row to prevRow
            prevRow = temp;
        }
        
        // Answer is the minimum element in the last evaluated row
        int ans = INT_MAX;
        for (int col = 0; col < n; col++) {
            ans = min(ans, prevRow[col]);
        }
        return ans;
    }
};
```

### 6. Time & Space Complexity
- **Time Complexity:** $\mathcal{O}(n^2)$ — Iterates through all $n \times n$ cells once.
- **Space Complexity:** $\mathcal{O}(n)$ — Only stores two 1D arrays (`prevRow` and `temp`) of size $n$.