# Unique Paths II (With Obstacles)

---

## 1. Problem Statement

Same as Unique Paths, but now the grid contains obstacles. A cell with value `1` is an obstacle — the robot cannot pass through it. Find the number of unique paths from top-left to bottom-right.

```
obstacleGrid:
0  0  0
0  1  0
0  0  0

Blocked path: cannot go through (1,1)
Valid paths:
  R→D→D→R (right, down, down, right): skips (1,1) ✅
  D→D→R→R: passes through... wait:
    (0,0)→(1,0)→(2,0)→(2,1)→(2,2) ✅
    (0,0)→(0,1)→(0,2)→(1,2)→(2,2) ✅

Answer: 2
```

```
obstacleGrid:
0  1
0  0

Blocked: (0,1) is obstacle
Only valid path: D→R → (0,0)→(1,0)→(1,1)
Answer: 1
```

---

## 2. How Unique Paths I Extends to Unique Paths II

The core recurrence is identical:
```
paths(i,j) = paths(i-1,j) + paths(i,j-1)
```

The only addition: **if a cell is an obstacle (`obstacleGrid[i][j] == 1`), set `paths(i,j) = 0`.**

An obstacle cell has 0 ways to pass through — it contributes nothing to any successor cell. This naturally propagates: any cell that could ONLY be reached via the obstacle also gets 0.

---

## 3. The Obstacle Effect — Why Setting to 0 Works

When we set `temp[j] = 0` for an obstacle cell, all downstream cells that read from this cell via `up = prevRow[j]` or `left = temp[j-1]` get `0` from it. If all their predecessors are blocked, they too become 0. The blockage cascades correctly through the grid.

```
Grid:
0  0  0
0  1  0
0  0  0

Without obstacle:          With obstacle at (1,1):
1  1  1                    1  1  1
1  2  3                    1  0  1
1  3  6                    1  1  2

(1,1)=0 blocks the paths:
  dp[1][2] = up(dp[0][2]=1) + left(dp[1][1]=0) = 1
  dp[2][1] = up(dp[1][1]=0) + left(dp[2][0]=1) = 1
  dp[2][2] = up(dp[1][2]=1) + left(dp[2][1]=1) = 2 ✅
```

---

## 4. Critical Edge Cases

### Obstacle at Start (0,0)

```cpp
if(obstacleGrid[i][j] == 1) {
    temp[j] = 0;
    continue;
}
if(i == 0 && j == 0)
    temp[j] = 1;
```

The obstacle check comes BEFORE the base case check. If `(0,0)` has an obstacle, `temp[0] = 0` and we skip — correctly returning 0 paths from a blocked start. If we set `temp[0] = 1` first and THEN checked for obstacle, we'd incorrectly count paths starting from a blocked cell.

### Obstacle at Destination (m-1, n-1)

If the destination has an obstacle, `temp[n-1] = 0` — correctly returns 0. The robot can never arrive at a blocked destination.

### Entire First Row/Column Blocked Midway

```
0  0  1  0  0
```

dp: `1  1  0  0  0` ← once we hit the obstacle at column 2, all right-ward cells become 0 (only predecessor is left, which is 0).

---

## 5. Dry Run

```
obstacleGrid:
0  0  0
0  1  0
0  0  0

m=3, n=3
prevRow = [0,0,0]
```

**Row i=0:**
```
j=0: obstacle?no, i==0&&j==0 → temp[0]=1
j=1: obstacle?no, up=prevRow[1]=0, left=temp[0]=1 → temp[1]=1
j=2: obstacle?no, up=0, left=temp[1]=1 → temp[2]=1

prevRow = [1,1,1]
```

**Row i=1:**
```
j=0: obstacle?no, up=prevRow[0]=1, left(j>0? no)=0 → temp[0]=1
j=1: obstacleGrid[1][1]=1 → temp[1]=0, continue
j=2: obstacle?no, up=prevRow[2]=1, left=temp[1]=0 → temp[2]=1

prevRow = [1,0,1]
```

**Row i=2:**
```
j=0: obstacle?no, up=prevRow[0]=1, left=0 → temp[0]=1
j=1: obstacle?no, up=prevRow[1]=0, left=temp[0]=1 → temp[1]=1
j=2: obstacle?no, up=prevRow[2]=1, left=temp[1]=1 → temp[2]=2

prevRow = [1,1,2]
```

**Return prevRow[2] = 2 ✅**

---

## 6. Code

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();

        vector<int> prevRow(n, 0);

        for(int i = 0; i < m; i++) {
            vector<int> temp(n, 0);

            for(int j = 0; j < n; j++) {
                // OBSTACLE CHECK FIRST — before base case
                if(obstacleGrid[i][j] == 1) {
                    temp[j] = 0;   // can't pass through obstacle
                    continue;
                }

                // Base case: starting cell
                if(i == 0 && j == 0) {
                    temp[j] = 1;
                } else {
                    int up   = (i > 0) ? prevRow[j]  : 0;   // from row above
                    int left = (j > 0) ? temp[j - 1] : 0;    // from left in current row

                    temp[j] = up + left;
                }
            }

            prevRow = temp;
        }

        return prevRow[n-1];
    }
};
```

---

## 7. Unique Paths I vs Unique Paths II

| | Unique Paths I | Unique Paths II |
|---|---|---|
| **Grid values** | All 0 (no obstacles) | 0 = open, 1 = obstacle |
| **Extra check** | None | `if(obstacle) → set 0, continue` |
| **Recurrence** | Same | Same |
| **Base case** | `dp[0][0] = 1` | `dp[0][0] = 1` IF not obstacle |
| **Code lines added** | — | 3 lines (check + continue) |

The obstacle variant is literally a 3-line addition to Unique Paths I. This is a hallmark of well-structured DP — variations are incremental modifications, not rewrites.

---

## 8. Complexity Analysis

### Time Complexity — `O(m × n)`

| Step | Cost |
|---|---|
| Fill every cell once | `O(m × n)` |
| Obstacle cells | `O(1)` — just set 0 and continue |

**Total: `O(m × n)`**

---

### Space Complexity — `O(n)`

| Structure | Size | Reason |
|---|---|---|
| `prevRow` | `O(n)` | Previous row |
| `temp` | `O(n)` | Current row being filled |

**Total: `O(2n)` = `O(n)`**

> Down from `O(m × n)` in the 2D tabulation version — same space optimization as Unique Paths I.

---

## 9. The `i >= 0 && j >= 0` Redundancy

```cpp
if(i >= 0 && j >= 0 && obstacleGrid[i][j] == 1)
```

Since the loop bounds are `i ∈ [0,m)` and `j ∈ [0,n)`, both `i >= 0` and `j >= 0` are ALWAYS true inside the loop. These checks are redundant — a minor code smell left from copy-paste. The correct minimal condition is simply:

```cpp
if(obstacleGrid[i][j] == 1)
```
