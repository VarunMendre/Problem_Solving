# Unique Paths

---

## 1. Problem Statement

A robot is located at the top-left corner of an `m × n` grid. It can only move **right** or **down** at each step. Find the number of **unique paths** to reach the bottom-right corner.

```
m=3, n=7 grid:

S . . . . . .
. . . . . . .
. . . . . . E

Answer: 28
```

```
m=3, n=2:

S .
. .
. E

Paths: R→D→D, D→R→D, D→D→R
Answer: 3
```

---

## 2. Intuition — Where Can I Come From?

To reach cell `(i, j)`, the robot could have come from:
- `(i-1, j)` — moved **down** into `(i,j)`
- `(i, j-1)` — moved **right** into `(i,j)`

So:
```
paths(i, j) = paths(i-1, j) + paths(i, j-1)
```

**Base cases:**
- `(0, 0)`: start position — 1 way to be here (just start)
- Any cell where `i < 0` or `j < 0`: impossible — 0 ways

This is a classic **counting DP on a grid** — the same "sum of paths" structure as Pascal's Triangle.

---

### Why This is Similar to Climbing Stairs

```
Climbing Stairs: f(n) = f(n-1) + f(n-2)
Unique Paths:    f(i,j) = f(i-1,j) + f(i,j-1)
```

Both are "how many ways to reach here" — sum of ways to reach all predecessors. The grid version just has 2D predecessors instead of 1D.

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int m, int n) {
        if(m == 0 && n == 0) return 1;   // reached start → 1 valid path
        if(m < 0 || n < 0)  return 0;    // out of bounds → 0 paths

        int left = fn(m, n-1);    // came from the left
        int up   = fn(m-1, n);    // came from above

        return left + up;
    }

public:
    int uniquePaths(int m, int n) {
        return fn(m-1, n-1);   // call for bottom-right cell
    }
};
```

### Why `fn(m-1, n-1)` and Base Case `(0,0)`?

The function is called with 0-indexed coordinates. Cell `(m-1, n-1)` is the bottom-right. Base case `(0,0)` is the top-left — exactly 1 way to be at the start.

### Recursion Tree for m=3, n=3

```
                f(2,2)
              /        \
         f(2,1)          f(1,2)
        /      \         /     \
    f(2,0)   f(1,1)  f(1,1)  f(0,2)
                ↑       ↑
           REPEATED   REPEATED
```

`f(1,1)` computed twice. For larger grids, repetition is exponential.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2^(m+n))` | Binary branching tree of depth m+n |
| **Space** | `O(m+n)` | Recursion stack depth |

---

## 4. Approach 2 — Memoization (Top-Down DP)

```cpp
class Solution {
private:
    int fn(int m, int n, vector<vector<int>>& dp) {
        if(m == 0 && n == 0) return 1;
        if(m < 0 || n < 0)  return 0;

        if(dp[m][n] != -1)
            return dp[m][n];   // cache hit

        int left = fn(m, n-1, dp);
        int up   = fn(m-1, n, dp);

        return dp[m][n] = left + up;   // store before return
    }

public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n, -1));
        return fn(m-1, n-1, dp);
    }
};
```

### Total Unique States

`dp[i][j]` for `i ∈ [0, m-1]` and `j ∈ [0, n-1]` → `m × n` states. Each computed once.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(m × n)` | Each of m×n cells computed once |
| **Space** | `O(m × n) + O(m+n)` | dp table + recursion stack |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

Fill the grid from top-left to bottom-right:

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n, 0));

        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                if(i == 0 && j == 0) {
                    dp[0][0] = 1;   // start cell
                    continue;
                }

                int up = 0, left = 0;

                if(i > 0) up   = dp[i-1][j];   // came from above
                if(j > 0) left = dp[i][j-1];    // came from left

                dp[i][j] = up + left;
            }
        }

        return dp[m-1][n-1];
    }
};
```

### Tabulation for m=3, n=3

```
Fill row by row:

dp[0][0]=1  (base)
dp[0][1]=1  (only from left: dp[0][0])
dp[0][2]=1  (only from left: dp[0][1])

dp[1][0]=1  (only from above: dp[0][0])
dp[1][1]=2  (dp[0][1] + dp[1][0] = 1+1)
dp[1][2]=3  (dp[0][2] + dp[1][1] = 1+2)

dp[2][0]=1  (only from above: dp[1][0])
dp[2][1]=3  (dp[1][1] + dp[2][0] = 2+1)
dp[2][2]=6  (dp[1][2] + dp[2][1] = 3+3)

Answer: dp[2][2] = 6 ✅
```

### Visual: Pascal's Triangle Connection

```
dp grid:
1  1  1  1  1
1  2  3  4  5
1  3  6  10 15
1  4  10 20 35
```

Each cell = sum of cell above + cell to the left. This is exactly **Pascal's Triangle** rotated! The number of unique paths is `C(m+n-2, m-1)` — a combinatorial formula.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(m × n)` | Fill every cell once |
| **Space** | `O(m × n)` | 2D dp array |

---

## 6. Approach 4 — Space Optimization

`dp[i][j]` only needs `dp[i-1][j]` (row above) and `dp[i][j-1]` (same row, previous column). Store only the previous row:

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> prevRow(n, 0);

        for(int i = 0; i < m; i++) {
            vector<int> temp(n, 0);

            for(int j = 0; j < n; j++) {
                if(i == 0 && j == 0)
                    temp[j] = 1;
                else {
                    int up   = (i > 0) ? prevRow[j]   : 0;  // from row above
                    int left = (j > 0) ? temp[j-1]    : 0;  // from current row, left
                    temp[j] = up + left;
                }
            }

            prevRow = temp;   // slide: current row becomes previous
        }

        return prevRow[n-1];
    }
};
```

### Why `prevRow[j]` for "up" and `temp[j-1]` for "left"?

- **`up = prevRow[j]`:** The cell directly above `(i,j)` is `(i-1,j)` — in the PREVIOUS row, same column `j`.
- **`left = temp[j-1]`:** The cell to the left is `(i,j-1)` — in the CURRENT row being filled, column `j-1` (already computed since we fill left to right).

This is why we need TWO arrays (`prevRow` and `temp`) — `prevRow` holds the completed previous row, while `temp` holds the current row being built.

### Space Optimization Trace for m=3, n=3

```
i=0: temp = [1, 1, 1]   → prevRow = [1,1,1]
i=1: temp[0]=1, temp[1]=prevRow[1]+temp[0]=1+1=2, temp[2]=prevRow[2]+temp[1]=1+2=3
     prevRow = [1,2,3]
i=2: temp[0]=1, temp[1]=prevRow[1]+temp[0]=2+1=3, temp[2]=prevRow[2]+temp[1]=3+3=6
     prevRow = [1,3,6]

return prevRow[2] = 6 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(m × n)` | Same nested loop |
| **Space** | `O(2n)` = `O(n)` | Two rows of size n (`prevRow` + `temp`) |

> Can reduce to `O(n)` by using one array with careful in-place update (right-to-left), but the two-array version is cleaner.

---

## 7. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(2^(m+n))` | `O(m+n)` | Try all paths via recursion |
| **Memoization** | `O(m×n)` | `O(m×n + m+n)` | Cache each (i,j) result |
| **Tabulation** | `O(m×n)` | `O(m×n)` | Fill grid bottom-up |
| **Space Optimized** | `O(m×n)` | `O(n)` | Keep only one row |

---

## 8. Mathematical Shortcut — Combinatorics

To reach `(m-1, n-1)` from `(0,0)`:
- Total steps = `(m-1) + (n-1) = m+n-2`
- Must take exactly `m-1` DOWN steps (the rest are RIGHT)
- Number of ways to arrange these steps:

```
C(m+n-2, m-1) = (m+n-2)! / ((m-1)! × (n-1)!)
```

For m=3, n=7: `C(8,2) = 28` ✅

This `O(m+n)` formula is faster than `O(m×n)` DP for large grids where m and n are small but the product is large. However, it requires careful handling of large factorials (use modular arithmetic for competitive programming).
