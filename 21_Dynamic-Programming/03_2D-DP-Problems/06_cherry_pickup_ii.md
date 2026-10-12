# Cherry Pickup II

---

## 1. Problem Statement

Two robots start at `(0,0)` and `(0, cols-1)` (top-left and top-right of a grid). Both move downward simultaneously — each step, a robot moves to `(i+1, j-1)`, `(i+1, j)`, or `(i+1, j+1)`. A robot collects all cherries in the cell it visits. If both robots land on the same cell, only one collects.

Return the **maximum total cherries** both robots can collect.

```
grid = [[3,1,1],
         [2,5,1],
         [1,5,5],
         [2,1,1]]

Robot1: (0,0)→(1,0)→(2,1)→(3,0) = 3+2+5+2 = 12
Robot2: (0,2)→(1,1)→(2,2)→(3,2)... wait:
  Actually Robot2: (0,2)→(1,2)→(2,2)→(3,2) = 1+1+5+1 = 8?
  Or: (0,2)→(1,1)→(2,1)→(3,1) = 1+5+5+1 = 12!

Best: Robot1=12 (left path), Robot2=12 (right convergence)
Total = 24 ✅
```

---

## 2. The Core Insight — 3D State

### Why Not Simulate Each Robot Independently?

If we ran each robot's optimal path independently, they might **share cells** — and cherries are collected only once. The robots interact: Robot1's choices affect what Robot2 can collect.

We must solve BOTH robots' paths **simultaneously**.

---

### The 3D State: `(i, j1, j2)`

Since both robots are always on the same row (they move down one row per step simultaneously), we can track:

```
state = (row i, col of Robot1 = j1, col of Robot2 = j2)
```

At each step, BOTH robots move. Robot1 has 3 choices (dj1 ∈ {-1,0,+1}), Robot2 has 3 choices (dj2 ∈ {-1,0,+1}) → **9 combinations** per state.

Total states: `n × m × m`

This is the leap from 2D DP (one agent) to **3D DP (two simultaneous agents)**.

---

### The Same-Cell Rule

```cpp
if(j1 == j2)
    cherries = grid[i][j1];        // only count once
else
    cherries = grid[i][j1] + grid[i][j2];   // count both
```

When both robots land on the same cell, they collect only once (the cell is empty after first collection). This single condition handles the overlap.

---

### Why `-1e8` for Out-of-Bounds?

We're maximizing. An invalid state (robot moves off-grid) should never be selected. Returning `-1e8` (a very large negative number) ensures `max(...)` never picks an out-of-bounds transition.

The base case initializes `maxi = -1e9` to allow proper updating from -1e8 values when ALL three directions for one robot might be out of bounds (edge columns).

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int i, int j1, int j2, int n, int m, vector<vector<int>>& grid) {
        // Either robot moved out of bounds
        if(j1 < 0 || j1 >= m || j2 < 0 || j2 >= m) return -1e8;

        // Base case: last row
        if(i == n-1) {
            if(j1 == j2) return grid[i][j1];
            return grid[i][j1] + grid[i][j2];
        }

        int maxi = -1e9;

        // Try all 9 combinations: 3 moves for Robot1 × 3 moves for Robot2
        for(int dj1 = -1; dj1 <= 1; dj1++) {
            for(int dj2 = -1; dj2 <= 1; dj2++) {
                int value;
                if(j1 == j2)
                    value = grid[i][j1];
                else
                    value = grid[i][j1] + grid[i][j2];

                value += fn(i+1, j1+dj1, j2+dj2, n, m, grid);
                maxi = max(maxi, value);
            }
        }

        return maxi;
    }

public:
    int cherryPickup(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        return fn(0, 0, m-1, n, m, grid);
    }
};
```

### Recursion Tree

```
fn(0, 0, m-1)
  ↓ 9 branches (all dj1,dj2 combinations)
  fn(1, j1', j2')
    ↓ 9 branches each
    fn(2, ...)
      ...

Total calls ≈ 9ⁿ (exponential in rows)
Many (i, j1, j2) states repeated across branches → overlapping subproblems
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(m² × 9ⁿ)` | m² starting states, 9ⁿ branches each |
| **Space** | `O(n)` | Recursion stack depth |

---

## 4. Approach 2 — Memoization (Top-Down DP)

Add a 3D dp cache:

```cpp
class Solution {
private:
    int fn(int i, int j1, int j2, int n, int m,
           vector<vector<vector<int>>>& dp, vector<vector<int>>& grid) {
        if(j1 < 0 || j1 >= m || j2 < 0 || j2 >= m) return -1e8;

        if(dp[i][j1][j2] != -1)
            return dp[i][j1][j2];   // cache hit

        if(i == n-1) {
            if(j1 == j2) return dp[i][j1][j2] = grid[i][j1];
            return dp[i][j1][j2] = grid[i][j1] + grid[i][j2];
        }

        int maxi = -1e9;

        for(int dj1 = -1; dj1 <= 1; dj1++) {
            for(int dj2 = -1; dj2 <= 1; dj2++) {
                int value = (j1 == j2) ? grid[i][j1] : grid[i][j1] + grid[i][j2];
                value += fn(i+1, j1+dj1, j2+dj2, n, m, dp, grid);
                maxi = max(maxi, value);
            }
        }

        return dp[i][j1][j2] = maxi;
    }

public:
    int cherryPickup(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<vector<int>>> dp(n, vector<vector<int>>(m, vector<int>(m, -1)));
        return fn(0, 0, m-1, n, m, dp, grid);
    }
};
```

### dp Dimensions: `n × m × m`

- `dp[i][j1][j2]` = max cherries both robots can collect from row `i` to row `n-1`, given Robot1 is at col `j1` and Robot2 is at col `j2` on row `i`
- Total states: `n × m × m`
- Work per state: 9 transitions = `O(1)`

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m² × 9)` = `O(nm²)` | n×m² states × 9 transitions |
| **Space** | `O(nm²) + O(n)` | dp table + recursion stack |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

Fill from the last row upward:

```cpp
class Solution {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<vector<int>>> dp(n, vector<vector<int>>(m, vector<int>(m, -1)));

        // Base case: last row
        for(int j1 = 0; j1 < m; j1++) {
            for(int j2 = 0; j2 < m; j2++) {
                dp[n-1][j1][j2] = (j1 == j2) ? grid[n-1][j1]
                                               : grid[n-1][j1] + grid[n-1][j2];
            }
        }

        // Fill rows n-2 to 0
        for(int i = n-2; i >= 0; i--) {
            for(int j1 = 0; j1 < m; j1++) {
                for(int j2 = 0; j2 < m; j2++) {
                    int maxi = -1e9;

                    for(int dj1 = -1; dj1 <= 1; dj1++) {
                        for(int dj2 = -1; dj2 <= 1; dj2++) {
                            int value = (j1 == j2) ? grid[i][j1]
                                                   : grid[i][j1] + grid[i][j2];

                            int nj1 = j1+dj1, nj2 = j2+dj2;
                            if(nj1 >= 0 && nj1 < m && nj2 >= 0 && nj2 < m)
                                value += dp[i+1][nj1][nj2];
                            else
                                value += (int)(-1e8);

                            maxi = max(maxi, value);
                        }
                    }

                    dp[i][j1][j2] = maxi;
                }
            }
        }

        return dp[0][0][m-1];   // robots start at (0,0) and (0,m-1)
    }
};
```

### Tabulation Trace (grid = [[3,1,1],[2,5,1],[1,5,5],[2,1,1]])

```
n=4, m=3

Base case (row 3):
  dp[3][j1][j2]:
    (0,0)=grid[3][0]=2, (1,1)=grid[3][1]=1, (2,2)=grid[3][2]=1
    (0,1)=2+1=3, (0,2)=2+1=3
    (1,0)=1+2=3, (1,2)=1+1=2
    (2,0)=1+2=3, (2,1)=1+1=2

Row 2 (i=2): [1,5,5]
  For (j1=0, j2=2): value=grid[2][0]+grid[2][2]=1+5=6
    Try all 9 dj1,dj2:
      (dj1=0,dj2=0): 6+dp[3][0][2]=6+3=9
      (dj1=0,dj2=-1): 6+dp[3][0][1]=6+3=9
      (dj1=1,dj2=-1): 6+dp[3][1][1]=6+1=7
      (dj1=1,dj2=0): 6+dp[3][1][2]=6+2=8
      ... best = 9
    dp[2][0][2] = 9

Row 1 (i=1): [2,5,1]
  For (j1=0, j2=2): 2+1=3
    (dj1=0,dj2=-1): 3+dp[2][0][1]=3+? ...
    (dj1=1,dj2=-1): 3+dp[2][1][1]=3+?  
    ... continue to find best

Row 0 (i=0): [3,1,1]
  dp[0][0][2] = answer

return dp[0][0][m-1] = dp[0][0][2]
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m² × 9)` = `O(nm²)` | 4 nested loops |
| **Space** | `O(nm²)` | 3D dp array |

---

## 6. Approach 4 — Space Optimization

`dp[i]` only needs `dp[i+1]` → keep two 2D layers:

```cpp
class Solution {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();

        // prev = dp[i+1], cur = dp[i]
        vector<vector<int>> prev(m, vector<int>(m, -1));
        vector<vector<int>> cur(m, vector<int>(m, -1));

        // Base case: last row
        for(int j1 = 0; j1 < m; j1++)
            for(int j2 = 0; j2 < m; j2++)
                prev[j1][j2] = (j1 == j2) ? grid[n-1][j1]
                                           : grid[n-1][j1] + grid[n-1][j2];

        for(int i = n-2; i >= 0; i--) {
            for(int j1 = 0; j1 < m; j1++) {
                for(int j2 = 0; j2 < m; j2++) {
                    int maxi = -1e9;

                    for(int dj1 = -1; dj1 <= 1; dj1++) {
                        for(int dj2 = -1; dj2 <= 1; dj2++) {
                            int value = (j1 == j2) ? grid[i][j1]
                                                   : grid[i][j1] + grid[i][j2];

                            int nj1 = j1+dj1, nj2 = j2+dj2;
                            if(nj1 >= 0 && nj1 < m && nj2 >= 0 && nj2 < m)
                                value += prev[nj1][nj2];
                            else
                                value += (int)(-1e8);

                            maxi = max(maxi, value);
                        }
                    }

                    cur[j1][j2] = maxi;
                }
            }
            prev = cur;   // slide upward
        }

        return prev[0][m-1];
    }
};
```

### Why Two 2D Arrays Instead of One Row?

In 1D problems (Frog Jump, House Robber), one row depends on at most 2 previous values → use 2 variables.

Here `dp[i][j1][j2]` depends on `dp[i+1][j1+dj1][j2+dj2]` for all 9 combinations of `dj1,dj2` → we need the ENTIRE previous 2D slice. We can't compress further without overwriting values we still need.

**Minimum storage: two 2D slices = `O(2m²)` = `O(m²)`.**

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(nm²)` | Same loops |
| **Space** | `O(2m²)` = `O(m²)` | Two 2D arrays of size m×m |

---

## 7. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(m × 9ⁿ)` | `O(n)` | Try all 9 move combinations per row |
| **Memoization** | `O(nm²)` | `O(nm²)` | Cache 3D state `dp[i][j1][j2]` |
| **Tabulation** | `O(nm²)` | `O(nm²)` | Fill 3D table bottom-up |
| **Space Optimized** | `O(nm²)` | `O(m²)` | Slide two 2D layers upward |

---

## 8. Story Points

---

**Story Point 1 — "Two robots, one row index — that's the 3D state insight"**

Both robots are always on the same row (they move down simultaneously). So the state `(row, j1, j2)` fully describes the situation. If robots moved at different speeds, we'd need 4D state `(row1, j1, row2, j2)`. The simultaneous movement constraint collapses this to 3D.

---

**Story Point 2 — "The same-cell rule is one `if` statement"**

The entire overlap constraint — "if both robots are on the same cell, count cherries only once" — is:

```cpp
int value = (j1 == j2) ? grid[i][j1] : grid[i][j1] + grid[i][j2];
```

No special case needed beyond this. Elegant.

---

**Story Point 3 — "9 inner loop iterations = 3 choices for Robot1 × 3 choices for Robot2"**

The inner double loop `dj1 ∈ {-1,0,+1}` and `dj2 ∈ {-1,0,+1}` generates all 9 possible joint moves. This is the key DP transition. Since both robots move simultaneously, we must consider all pairs — not just each robot independently.

---

**Story Point 4 — "Space optimization stops at O(m²) — cannot go further"**

For 1D DP (House Robber): `O(n)` → `O(1)` (2 variables)
For 2D DP (Unique Paths): `O(nm)` → `O(m)` (1 row)
For 3D DP (Cherry Pickup II): `O(nm²)` → `O(m²)` (2 layers of 2D)

Each dimension reduction saves one factor. Since the third dimension (n, rows) is the one we can eliminate with a sliding window, we go from `O(nm²)` to `O(m²)`. The `m²` cannot be further reduced because we need ALL 9 neighbors of the current `(j1,j2)` from the previous layer.

---

**Story Point 5 — "This is the template for ALL multi-agent simultaneous grid DP"**

Any problem where multiple agents move simultaneously through a grid follows this pattern:
1. State = `(row, col1, col2, ..., colK)` for K agents
2. Transitions = all combinations of each agent's moves
3. Collision/overlap handling = check if any agents share a position
4. Space optimization = slide one layer at a time

Cherry Pickup II is the canonical 2-agent version. Extend to K agents → `(K+1)D` DP.

---

## 9. Complexity Summary

| | Value | Reason |
|---|---|---|
| **States** | `n × m × m` | row × Robot1 col × Robot2 col |
| **Transitions per state** | 9 | 3 Robot1 moves × 3 Robot2 moves |
| **Total work** | `O(9nm²)` = `O(nm²)` | |
| **Optimal space** | `O(m²)` | Two 2D slices, each `m × m` |
