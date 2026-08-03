# Shortest Path in Binary Matrix

---

## 1. Problem Statement

Given an `n × n` binary matrix `grid`, return the length of the **shortest clear path** from the top-left cell `(0,0)` to the bottom-right cell `(n-1, n-1)`.

A **clear path** is a path where:
- Every cell on the path has value `0`
- Movement is allowed in **8 directions** (horizontal, vertical, diagonal)
- Path length = number of cells visited (including start and end)

Return `-1` if no such path exists.

```
Grid:
[ 0  0  0 ]
[ 1  1  0 ]
[ 1  1  0 ]

Shortest clear path:
(0,0)→(0,1)→(0,2)→(1,2)→(2,2) = length 5? 
No: (0,0)→(0,1)→(1,2)→(2,2) diagonal move allowed
Actually: (0,0)→(0,1)→(0,2)→(1,2)→(2,2) = 5
Or: (0,0) diagonal (0,0)→(0,1)→(1,2)→(2,2) = 4

Output: 4
```

---

## 2. Intuition / Approach

### Why This is Different From Standard BFS Shortest Path

In the standard unweighted BFS shortest path (previous problem), every step costs `1` and BFS naturally finds the shortest path level by level.

**This problem seems identical — so why use Dijkstra (priority queue)?**

Actually, for this specific problem where all edges have equal weight `1`, BFS works perfectly and is `O(N²)`. The Dijkstra approach here is **slightly overkill** but correct — and it generalizes naturally to weighted grid problems.

The reason this code uses Dijkstra: it was written as a generalizable template. If cells had different costs (weighted grid), this exact code would still work. BFS would not.

---

### The 8-Direction Movement

Unlike most grid problems that use 4-directional movement, this problem allows **8 directions** (including diagonals):

```
dr[] = {-1, -1,  0, +1, +1, +1,  0, -1}
dc[] = { 0, +1, +1, +1,  0, -1, -1, -1}
```

| Index | (dr, dc) | Direction |
|---|---|---|
| 0 | (-1, 0) | Up |
| 1 | (-1,+1) | Up-Right |
| 2 | (0, +1) | Right |
| 3 | (+1,+1) | Down-Right |
| 4 | (+1, 0) | Down |
| 5 | (+1,-1) | Down-Left |
| 6 | (0, -1) | Left |
| 7 | (-1,-1) | Up-Left |

All 8 directions from any cell.

---

### Early Exit Checks

```cpp
if(grid[0][0] == 1 || grid[n-1][n-1] == 1)
    return -1;
```

If the source or destination itself is blocked (`1`), immediately return `-1` — no path can exist.

---

### `dist[0][0] = 1` — Why 1, Not 0?

The path length counts **number of cells visited**, not edges traversed. So starting at `(0,0)` means we've visited 1 cell → `dist[0][0] = 1`.

When we reach `(n-1, n-1)`, `dist[n-1][n-1]` = total cells visited = path length as required.

This differs from standard Dijkstra where `dist[src] = 0` (edge count). Here it's `1` because the problem counts cells, not edges.

---

### The Node Type

```cpp
using Node = pair<int, pair<int, int>>;
// {distance, {row, col}}
```

Instead of a flat pair, we need to store 3 values: distance, row, col. The nested pair achieves this. The outer pair's first element (distance) is what the min-heap orders by.

---

### The Condition `grid[newRow][newCol] == 0`

We can only travel through cells with value `0` (clear cells). Value `1` means blocked. This single condition acts as both the "is this a valid path cell?" and "is this cell reachable?" check.

---

## 3. Dry Run

```
Grid (4×4):
[ 0  0  0  1 ]
[ 0  1  0  0 ]
[ 0  0  0  0 ]
[ 1  0  1  0 ]

n=4, grid[0][0]=0, grid[3][3]=0 → proceed

dist = all INT_MAX, dist[0][0]=1
pq   = [{1, {0,0}}]
```

---

**Pop {1, (0,0)}:**
```
1 > dist[0][0]=1? NO → process
row=n-1=3? NO → not destination

Check all 8 directions from (0,0):
  (-1,0): OOB
  (-1,1): OOB
  (0,1):  grid=0 ✅ → newDist=2 < INF → dist[0][1]=2, push {2,(0,1)}
  (1,1):  grid=1 ✗
  (1,0):  grid=0 ✅ → dist[1][0]=2, push {2,(1,0)}
  (1,-1): OOB
  (0,-1): OOB
  (-1,-1):OOB

dist:
[ 1  2  INF  INF ]
[ 2  INF INF  INF ]
[ INF INF INF  INF ]
[ INF INF INF  INF ]

pq = [{2,(0,1)}, {2,(1,0)}]
```

**Pop {2, (0,1)}:**
```
Check 8 dirs from (0,1):
  (-1,0): OOB
  (-1,2): OOB
  (0,2):  grid=0 ✅ → dist[0][2]=3, push {3,(0,2)}
  (1,2):  grid=0 ✅ → dist[1][2]=3, push {3,(1,2)}
  (1,1):  grid=1 ✗
  (1,0):  dist[1][0]=2, newDist=3 < 2? NO
  (0,0):  dist[0][0]=1, 3<1? NO
  (-1,0): OOB

pq = [{2,(1,0)}, {3,(0,2)}, {3,(1,2)}]
```

**Pop {2, (1,0)}:**
```
Check 8 dirs from (1,0):
  (0,0): 3<1? NO
  (0,1): 3<2? NO
  (1,1): grid=1 ✗
  (2,1): grid=0 ✅ → dist[2][1]=3, push {3,(2,1)}
  (2,0): grid=0 ✅ → dist[2][0]=3, push {3,(2,0)}
  (2,-1): OOB
  (1,-1): OOB
  (0,-1): OOB
```

**Pop {3, (0,2)}:**
```
8 dirs from (0,2):
  (0,3): grid=1 ✗
  (1,3): grid=0 ✅ → dist[1][3]=4, push {4,(1,3)}
  (1,2): dist=3, newDist=4<3? NO
  (1,1): grid=1 ✗
  others: visited or OOB

pq = [{3,(1,2)},{3,(2,1)},{3,(2,0)},{4,(1,3)}]
```

**Pop {3, (1,2)}:**
```
(2,2): grid=0 ✅ → dist[2][2]=4, push {4,(2,2)}
(2,3): grid=0 ✅ → dist[2][3]=4, push {4,(2,3)}
(1,3): dist=4, newDist=4<4? NO
others: visited
```

**Pop {3, (2,1)}, {3,(2,0)}:**
```
Additional relaxations but no improvements over existing distances
```

**Pop {4, (1,3)}:**
```
(2,3): dist[2][3]=4, newDist=5<4? NO
(2,2): dist=4, 5<4? NO
```

**Pop {4, (2,2)}:**
```
(3,3): grid=0 ✅ → dist[3][3]=5, push {5,(3,3)}
(3,1): grid=0 ✅ → dist[3][1]=5, push {5,(3,1)}
(3,2): grid=1 ✗
(2,3): dist=4, 5<4? NO
```

**Pop {4, (2,3)}:**
```
(3,3): dist=5, newDist=5<5? NO
(3,2): grid=1 ✗
```

**Pop {5, (3,3)}:**
```
row==n-1=3, col==n-1=3 → DESTINATION REACHED → return 5 ✅
```

---

## 4. Story Points

---

**Story Point 1 — "8 directions, not 4"**

The standard direction array for 4-dir problems has 4 entries. Here we have 8. The diagonal moves (`(-1,+1)`, `(+1,+1)`, `(+1,-1)`, `(-1,-1)`) are the additions. This means from any cell, we can reach all 8 surrounding cells. Missing this and using only 4 directions would give wrong answers for paths that require diagonal movement.

---

**Story Point 2 — "`dist[0][0] = 1` because path length = cells, not edges"**

This is the most common mistake. Standard Dijkstra initializes `dist[src] = 0` (0 edges traversed). Here, the problem asks for the number of cells in the path. Cell `(0,0)` itself is the 1st cell. So `dist[0][0] = 1`. Every step adds 1 cell, so `newDistance = distance + 1`. When we reach destination, `dist[n-1][n-1]` directly gives the answer in cells.

---

**Story Point 3 — "Early destination check inside the loop, not after"**

```cpp
if(row == n-1 && col == n-1)
    return distance;
```

This check happens when we POP a cell from the priority queue — at that moment, `distance` is guaranteed to be the shortest path to that cell (Dijkstra's finalization guarantee). We return immediately without processing all edges, saving work.

If the check were done OUTSIDE the loop (after BFS/Dijkstra finishes), it would still give the correct `dist[n-1][n-1]` — just with more unnecessary work done.

---

**Story Point 4 — "BFS would also work here — Dijkstra is a generalization"**

Since all edges cost 1 (each step = 1 cell), BFS would find the shortest path just as correctly in `O(N²)`. The priority queue here is `O(N² log N)` — slightly worse.

The advantage: if the problem were **weighted** (e.g., some cells cost 2 to enter), this Dijkstra code handles it with minimal changes. BFS would need to become Dijkstra anyway. This code is written for extensibility.

---

**Story Point 5 — "`grid[newRow][newCol] == 0` must be checked BEFORE accessing `dist`"**

```cpp
if(grid[newRow][newCol] == 0 && newDistance < dist[newRow][newCol])
```

We check `grid == 0` FIRST. If the cell is blocked (`1`), we skip it regardless of distance. The order matters — if we checked `dist` first and the cell was blocked, we might still push blocked cells into the heap if their `dist` happened to be `INT_MAX`.

---

## 5. Code

```cpp
class Solution {
public:
    using Node = pair<int, pair<int, int>>;  // {distance, {row, col}}

    int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
        int n = grid.size();

        // Early exit: source or destination is blocked
        if(grid[0][0] == 1 || grid[n-1][n-1] == 1)
            return -1;

        // dist[r][c] = min cells in path from (0,0) to (r,c)
        vector<vector<int>> dist(n, vector<int>(n, INT_MAX));
        dist[0][0] = 1;   // source counts as 1 cell

        // Min-heap: {distance, {row, col}}
        priority_queue<Node, vector<Node>, greater<Node>> pq;
        pq.push({1, {0, 0}});

        // 8-directional movement (including diagonals)
        const int dr[] = {-1,-1, 0,+1,+1,+1, 0,-1};
        const int dc[] = { 0,+1,+1,+1, 0,-1,-1,-1};

        while(!pq.empty()) {
            auto [distance, pos] = pq.top();
            pq.pop();
            auto [row, col] = pos;

            // Stale entry: we already found a shorter path
            if(distance > dist[row][col]) continue;

            // Destination reached with guaranteed shortest distance
            if(row == n-1 && col == n-1)
                return distance;

            // Explore all 8 neighbors
            for(int i = 0; i < 8; i++) {
                int newRow = row + dr[i];
                int newCol = col + dc[i];

                // Bounds check
                if(newRow < 0 || newRow >= n || newCol < 0 || newCol >= n)
                    continue;

                int newDistance = distance + 1;

                // Must be clear (0) AND a shorter path
                if(grid[newRow][newCol] == 0 &&
                   newDistance < dist[newRow][newCol]) {
                    dist[newRow][newCol] = newDistance;
                    pq.push({newDistance, {newRow, newCol}});
                }
            }
        }

        return -1;   // destination unreachable
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N² log N²)` = `O(N² log N)`

| Step | Cost | Reason |
|---|---|---|
| Initialize `dist` | `O(N²)` | N×N grid |
| Each cell enqueued/dequeued | `O(N²)` cells × `O(log N²)` per op | At most N² cells, each pushed once |
| 8 direction checks per cell | `O(8)` = `O(1)` | Constant |

**Total: `O(N² log N²)` = `O(N² × 2 log N)` = `O(N² log N)`**

> Since `log N²` = `2 log N` = `O(log N)`, the `2` drops in asymptotic notation.

---

### Space Complexity — `O(N²)`

| Structure | Size | Reason |
|---|---|---|
| `dist[][]` | `O(N²)` | N×N distance matrix |
| Priority queue | `O(N²)` worst case | At most N² entries (one per cell) |

**Total: `O(N²)`**

---

### BFS vs Dijkstra for This Problem

| | BFS | Dijkstra (this code) |
|---|---|---|
| **Applicable when** | All edges weight = 1 (this problem) | Any edge weights |
| **TC** | `O(N²)` | `O(N² log N)` |
| **SC** | `O(N²)` | `O(N²)` |
| **Generalizability** | Only unweighted grids | Weighted grids too |
| **Code change for weighted** | Needs rewrite → Dijkstra | Just change `+1` to `+weight` |

> For this specific problem: BFS is faster. Use Dijkstra when the grid is weighted (e.g., LeetCode 1368 — Minimum Cost to Make at Least One Valid Path in a Grid).
