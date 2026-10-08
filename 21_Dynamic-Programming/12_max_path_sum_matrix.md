# Maximum Path Sum in a Matrix (Gold Mine Problem)

---

## 1. Problem Statement

Given an `n × m` integer matrix, find the **maximum path sum**. A path:
- Starts at any cell in the **first row**
- At each step moves to the **next row**, choosing from three directions: down-left, straight down, or down-right
- Ends at the **last row**

Return the maximum sum across all valid paths.

```
mat:
1  2  3
4  5  6
7  8  9

Starting from col 1 (value=2):
  Down (5) → Down (8): 2+5+8 = 15
  Down (5) → Right(9): 2+5+9 = 16 ✅

Starting from col 2 (value=3):
  Down (6) → Left (8): 3+6+8 = 17 ← actually (wait, from col2 in row1 = 6, left of that in row2 = col1 = 8)
  Down (6) → Down (9): 3+6+9 = 18 ✅

Answer: 18
```

---

## 2. Intuition

This is the **maximum** version of the minimum falling path sum pattern. The only change: `min` → `max`.

At each cell `(row, col)`, you can come from three cells in the row above (for top-down) or go to three cells in the row below (for top-up recursion):

```
dp[row][col] = mat[row][col] + max(
    dp[row+1][col-1],   // down-left
    dp[row+1][col],     // straight down
    dp[row+1][col+1]    // down-right
)
```

**Key differences from Min Falling Path Sum:**
- We maximize instead of minimize
- Start from ANY column in row 0 (same)
- Out-of-bounds returns `0` instead of `INT_MAX` (since we're maximizing, 0 is a neutral floor — a non-existent path contributes nothing)

---

### Why Out-of-Bounds Returns `0` (Not `INT_MIN`)

```cpp
if(col < 0 || col >= m)
    return 0;
```

For maximization, an invalid direction should NOT be chosen. Returning `0` means "this direction contributes nothing" — it will only be chosen if all three directions are non-positive, which is the correct behavior.

Returning `INT_MIN` would also work conceptually but risks overflow when doing `mat[row][col] + INT_MIN`. Returning `0` is safe.

For minimization (falling path sum), we returned `INT_MAX` — "this direction costs infinity, never pick it." The neutral element flips based on the optimization direction.

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int row, int col, int n, int m, vector<vector<int>>& mat) {
        // Out of bounds column → no path
        if(col < 0 || col >= m) return 0;
        // Out of bounds row (shouldn't happen but safety guard)
        if(row < 0 || row >= n) return 0;

        // Base case: last row → return this cell's value
        if(row == n-1) return mat[row][col];

        // Try all 3 downward directions
        int left  = mat[row][col] + fn(row+1, col-1, n, m, mat);
        int down  = mat[row][col] + fn(row+1, col,   n, m, mat);
        int right = mat[row][col] + fn(row+1, col+1, n, m, mat);

        return max({left, down, right});
    }

public:
    int maximumPath(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        int ans = INT_MIN;

        for(int col = 0; col < m; col++)
            ans = max(ans, fn(0, col, n, m, mat));

        return ans;
    }
};
```

### Recursion Tree (3 branches per row)

```
fn(0, col)
├── fn(1, col-1)
│   ├── fn(2, col-2)
│   ├── fn(2, col-1)
│   └── fn(2, col)     ← overlaps with branch below!
├── fn(1, col)
│   ├── fn(2, col-1)   ← REPEATED
│   ├── fn(2, col)     ← REPEATED
│   └── fn(2, col+1)
└── fn(1, col+1)
    ...
```

Many sub-calls computed multiple times → exponential waste.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(m × 3ⁿ)` | m starting cols × 3ⁿ calls per col (branching factor 3, depth n) |
| **Space** | `O(n)` | Recursion stack depth = n rows |

---

## 4. Approach 2 — Memoization (Top-Down DP)

```cpp
class Solution {
private:
    int fn(int row, int col, int n, int m,
           vector<vector<int>>& dp, vector<vector<int>>& mat) {
        if(col < 0 || col >= m) return 0;

        if(row == n-1)
            return dp[row][col] = mat[row][col];

        if(dp[row][col] != -1)
            return dp[row][col];   // cache hit

        int left  = mat[row][col] + fn(row+1, col-1, n, m, dp, mat);
        int down  = mat[row][col] + fn(row+1, col,   n, m, dp, mat);
        int right = mat[row][col] + fn(row+1, col+1, n, m, dp, mat);

        return dp[row][col] = max({left, down, right});
    }

public:
    int maximumPath(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        vector<vector<int>> dp(n, vector<int>(m, -1));
        int ans = INT_MIN;

        for(int col = 0; col < m; col++)
            ans = max(ans, fn(0, col, n, m, dp, mat));

        return ans;
    }
};
```

### Why `-1` as Sentinel (Not `INT_MAX`)?

Matrix values can be any integer, but sums of matrix values along paths are unlikely to be `-1` if all values are positive. However for a fully general solution, `-1` as sentinel could conflict with negative matrix values. The safe sentinel here depends on problem constraints.

For competitive programming (typical values ≥ 0), `-1` works. For negative values, use a separate `computed[][]` boolean array or a value outside the valid range.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | n×m unique states, O(1) work each |
| **Space** | `O(nm) + O(n)` | dp table + recursion stack |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

Process from the **last row upward** — no recursion:

```cpp
class Solution {
public:
    int maximumPath(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        vector<vector<int>> dp(n, vector<int>(m, 0));

        // Base case: last row
        dp[n-1] = mat[n-1];

        for(int row = n-2; row >= 0; row--) {
            for(int col = 0; col < m; col++) {
                // Straight down — always valid
                int down = mat[row][col] + dp[row+1][col];

                // Diagonal left — only if col > 0
                int left  = (col > 0)   ? mat[row][col] + dp[row+1][col-1] : 0;

                // Diagonal right — only if col+1 < m
                int right = (col+1 < m) ? mat[row][col] + dp[row+1][col+1] : 0;

                dp[row][col] = max({left, down, right});
            }
        }

        // Answer: max value in first row of dp
        return *max_element(dp[0].begin(), dp[0].end());
    }
};
```

### Why Process from `n-2` Down to `0`?

We fill `dp[row]` using `dp[row+1]`. Starting from `row = n-2` means the row below (`row+1 = n-1`) is already filled (it's our base case = last row of matrix). We work upward until we fill `dp[0]`.

### Tabulation Trace (mat = [[1,2,3],[4,5,6],[7,8,9]])

```
Base case (row 2):
dp[2] = [7, 8, 9]

Row 1 (row = 1):
  col=0: down=4+dp[2][0]=4+7=11, left=0(OOB), right=4+dp[2][1]=4+8=12 → dp[1][0]=12
  col=1: down=5+8=13, left=5+7=12, right=5+9=14 → dp[1][1]=14
  col=2: down=6+9=15, left=6+8=14, right=0(OOB) → dp[1][2]=15

dp[1] = [12, 14, 15]

Row 0 (row = 0):
  col=0: down=1+12=13, left=0(OOB), right=1+14=15 → dp[0][0]=15
  col=1: down=2+14=16, left=2+12=14, right=2+15=17 → dp[0][1]=17
  col=2: down=3+15=18, left=3+14=17, right=0(OOB) → dp[0][2]=18

dp[0] = [15, 17, 18]

Answer: max(15,17,18) = 18 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(nm) + O(m)` = `O(nm)` | Fill n×m cells + scan m cells for answer |
| **Space** | `O(nm)` | 2D dp table |

---

## 6. Approach 4 — Space Optimization

Keep only one row at a time:

```cpp
class Solution {
public:
    int maximumPath(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();

        vector<int> prevRow = mat[n-1];   // start with last row

        for(int row = n-2; row >= 0; row--) {
            vector<int> temp(m, 0);

            for(int col = 0; col < m; col++) {
                int down  = mat[row][col] + prevRow[col];
                int left  = (col > 0)   ? mat[row][col] + prevRow[col-1] : 0;
                int right = (col+1 < m) ? mat[row][col] + prevRow[col+1] : 0;

                temp[col] = max({left, down, right});
            }

            prevRow = temp;   // slide upward
        }

        return *max_element(prevRow.begin(), prevRow.end());
    }
};
```

### Why `prevRow[col-1]` and `prevRow[col+1]` Are Safe Here (No Overwrite Issue)

Unlike some space-optimized DPs where writing to the current position corrupts reads for later cells, here we use a **separate `temp` array** for writes and only read from `prevRow`. So `prevRow[col-1]` and `prevRow[col+1]` are always from the (already completed) row below — never corrupted by current row writes.

### Space Optimization Trace (same matrix)

```
prevRow = [7, 8, 9]   (last row)

row=1:
  col=0: down=4+7=11, left=0, right=4+8=12 → temp[0]=12
  col=1: down=5+8=13, left=5+7=12, right=5+9=14 → temp[1]=14
  col=2: down=6+9=15, left=6+8=14, right=0 → temp[2]=15
prevRow = [12, 14, 15]

row=0:
  col=0: down=1+12=13, left=0, right=1+14=15 → temp[0]=15
  col=1: down=2+14=16, left=2+12=14, right=2+15=17 → temp[1]=17
  col=2: down=3+15=18, left=3+14=17, right=0 → temp[2]=18
prevRow = [15, 17, 18]

return max(15,17,18) = 18 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(nm) + O(m)` = `O(nm)` | Same nested loops |
| **Space** | `O(m) + O(m)` = `O(m)` | `prevRow` + `temp` |

---

## 7. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(m × 3ⁿ)` | `O(n)` | Branch 3 ways per row, take max |
| **Memoization** | `O(nm)` | `O(nm + n)` | Cache `dp[row][col]` |
| **Tabulation** | `O(nm)` | `O(nm)` | Fill bottom-up from last row |
| **Space Optimized** | `O(nm)` | `O(m)` | Slide single row upward |

---

## 8. Min Falling Path Sum vs Max Path Sum

| | Min Falling Path Sum | Max Path Sum (Gold Mine) |
|---|---|---|
| **Goal** | Minimize | Maximize |
| **Out-of-bounds returns** | `INT_MAX` (can't use this path) | `0` (this direction contributes nothing) |
| **Operation** | `min(left, down, right)` | `max(left, down, right)` |
| **Traversal direction** | Top→Bottom (recursion) / Top-fill (tabulation) | Same |
| **Final answer** | `min` across last row | `max` across first row (if bottom-up) |
| **Direction arrays** | Same | Same |

The two problems share identical structure — swapping `min` ↔ `max` and `INT_MAX` ↔ `0` (out-of-bounds neutral) is the complete transformation.
