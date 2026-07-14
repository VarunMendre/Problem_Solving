# 01 Matrix — Distance to Nearest Zero

---

## 1. Problem Statement

Given an `n × m` binary matrix `mat` containing only `0`s and `1`s, return a matrix `dist` of the same size where `dist[i][j]` = **distance of the nearest `0` from cell `(i, j)`**.

Distance is measured in terms of **steps** — each step moves to an adjacent cell (up, right, down, left). Diagonal movement is not allowed.

```
Input:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  1  1 ]

Output:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  2  1 ]
```

For every `0` cell → distance is `0` (it's already at a zero).  
For every `1` cell → distance = minimum steps to reach any `0`.

---

## 2. Intuition / Approach

### The Wrong Way — BFS From Every `1`

The naive approach: for each `1` cell, run a BFS to find the nearest `0`.

- N×M cells, each BFS costs O(N×M) → **Total: O((N×M)²)**
- For a 1000×1000 grid, that's 10¹² operations. Completely infeasible.

---

### The Right Way — Reverse Multi-Source BFS From All `0`s

**Flip the perspective entirely.**

Instead of asking *"from this `1`, how far is the nearest `0`?"*, ask *"from ALL `0`s simultaneously, how far can we reach?"*

Think of it like this: imagine all `0` cells are **water sources** and you pour water into all of them at the same time. The water spreads outward in all 4 directions simultaneously. The first time the water reaches any `1` cell — that's the shortest distance from that `1` to any `0`.

This is **multi-source BFS**:
- All `0` cells start at **level 0** (distance = 0)
- Their neighbors (if `1`) get **level 1** (distance = 1)
- Those neighbors' unvisited neighbors get **level 2** (distance = 2)
- And so on...

Since BFS expands level by level, the **first time** we reach any `1` cell is guaranteed to be via the shortest possible path from the nearest `0`.

---

### Why Multi-Source BFS Gives the Correct Answer

BFS processes cells in **non-decreasing order of distance from the sources**.

```
All 0-cells (distance=0) are processed first.
Then all cells at distance=1 from any 0-cell.
Then distance=2.
...
```

The first time a cell is visited = the shortest distance from ANY zero source to that cell. Because once visited, it's marked and never revisited — we never overwrite a shorter distance with a longer one.

This is the same guarantee as single-source BFS, just extended to multiple sources simultaneously.

---

### The `vis[][]` Array

We need to track which cells have already been assigned a distance (to avoid re-processing). When a cell is added to the queue, we immediately mark `vis[r][c] = 1`.

**Why mark on enqueue, not on dequeue?**

If we marked on dequeue, the same cell could be enqueued multiple times from different neighbors before it's dequeued — causing incorrect distance values and wasted work.

Marking on enqueue means: *"this cell is claimed — don't touch it again."*

---

### Direction Arrays

```cpp
const int dRows[4] = {-1, +1,  0,  0};
const int dCols[4] = { 0,  0, -1, +1};
```

| Index | dRow | dCol | Direction |
|---|---|---|---|
| 0 | -1 | 0 | Up |
| 1 | +1 | 0 | Down |
| 2 | 0 | -1 | Left |
| 3 | 0 | +1 | Right |

---

## 3. Dry Run

```
mat:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  1  1 ]

n=3, m=3
```

**Step 1 — Initialize:**
```
Scan for 0s and push them into queue with vis=1:
(0,0), (0,1), (0,2), (1,0), (1,2) → all are 0s

queue = [(0,0),(0,1),(0,2),(1,0),(1,2)]

vis:
[ 1  1  1 ]
[ 1  0  1 ]
[ 0  0  0 ]

dist:
[ 0  0  0 ]
[ 0  0  0 ]
[ 0  0  0 ]
```

---

**BFS Processing:**

**Pop (0,0), dist=0:**
```
Neighbors: up(-1,0)OOB, down(1,0)vis=1 skip, left(0,-1)OOB, right(0,1)vis=1 skip
Nothing new enqueued.
```

**Pop (0,1), dist=0:**
```
Neighbors: up OOB, down(1,1)vis=0 ✅, left(0,0)vis=1 skip, right(0,2)vis=1 skip
  (1,1): vis=1, dist=0+1=1, push (1,1)

vis:
[ 1  1  1 ]
[ 1  1  1 ]
[ 0  0  0 ]

dist:
[ 0  0  0 ]
[ 0  1  0 ]
[ 0  0  0 ]
```

**Pop (0,2), dist=0:**
```
Neighbors: up OOB, down(1,2)vis=1 skip, left(0,1)vis=1 skip, right OOB
Nothing new.
```

**Pop (1,0), dist=0:**
```
Neighbors: up(0,0)vis=1 skip, down(2,0)vis=0 ✅, left OOB, right(1,1)vis=1 skip
  (2,0): vis=1, dist=0+1=1, push (2,0)

dist:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  0  0 ]
```

**Pop (1,2), dist=0:**
```
Neighbors: up(0,2)vis=1 skip, down(2,2)vis=0 ✅, left(1,1)vis=1 skip, right OOB
  (2,2): vis=1, dist=0+1=1, push (2,2)

dist:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  0  1 ]
```

**Pop (1,1), dist=1:**
```
Neighbors: up(0,1)vis=1 skip, down(2,1)vis=0 ✅, left(1,0)vis=1 skip, right(1,2)vis=1 skip
  (2,1): vis=1, dist=1+1=2, push (2,1)

dist:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  2  1 ]
```

**Pop (2,0), dist=1):** All neighbors visited → nothing pushed.  
**Pop (2,2), dist=1:** All neighbors visited → nothing pushed.  
**Pop (2,1), dist=2:** All neighbors visited → nothing pushed.

**Queue empty → BFS ends.**

```
Final dist:
[ 0  0  0 ]
[ 0  1  0 ]
[ 1  2  1 ]  ✅
```

---

## 4. Story Points

> These are the key "aha moments" — things that if you miss, you'll get the wrong answer or write an inefficient solution.

---

**Story Point 1 — "Don't BFS from each 1, BFS FROM the 0s"**

The most important insight. Running BFS from each `1` separately is `O((NM)²)` — way too slow. The reversal — starting from all `0`s simultaneously — collapses it to `O(NM)`. This single insight is the entire algorithm.

---

**Story Point 2 — "Mark visited on ENQUEUE, not dequeue"**

If you mark visited on dequeue, the same cell can be added to the queue multiple times (from multiple neighbors), each time potentially with a different (incorrect, larger) distance. Marking on enqueue ensures a cell is claimed the moment the shortest distance to it is found.

```
Wrong: mark on dequeue
  Cell (2,1) enqueued twice — once with dist=2, once with dist=3
  Gets processed with dist=3 first? → wrong answer

Correct: mark on enqueue
  First enqueue of (2,1) marks it → second attempt finds vis=1 → skipped
```

---

**Story Point 3 — "The `dist` array's initial value of `0` is correct"**

When we initialize `dist(n, vector<int>(m, 0))`, all `0` cells already have `dist=0` — which is correct (they're at zero distance from themselves). The `1` cells also start at `0`, but they get correctly updated when BFS reaches them. There's no collision because `0` cells are already in the queue (marked visited) and `1` cells start unvisited.

---

**Story Point 4 — "Multi-source BFS guarantees shortest distance"**

Because BFS processes cells in non-decreasing order of distance (level by level), the **first time** any cell is reached guarantees the shortest path. We never need to compare or update — just set and move on.

---

**Story Point 5 — "Why not Dijkstra?"**

All edges here have equal weight (every step costs 1). BFS on unweighted graphs gives shortest distances in `O(V+E)`. Dijkstra is needed only when edges have varying weights. Using Dijkstra here would be overkill.

---

## 5. Code

```cpp
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int n = mat.size();
        int m = mat[0].size();

        // vis tracks which cells have been processed
        vector<vector<int>> vis(n, vector<int>(m, 0));

        // dist stores the answer — distance to nearest 0 for each cell
        vector<vector<int>> dist(n, vector<int>(m, 0));

        queue<pair<int,int>> q;

        // Step 1: Multi-source BFS — seed all 0 cells simultaneously
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(mat[i][j] == 0) {
                    q.push({i, j});       // 0-cells start BFS at level 0
                    vis[i][j] = 1;        // mark visited on enqueue
                }
            }
        }

        // Direction arrays: up, down, left, right
        const int dRows[4] = {-1, +1,  0,  0};
        const int dCols[4] = { 0,  0, -1, +1};

        // Step 2: BFS — spread distances outward from all 0 sources
        while(!q.empty()) {
            int row = q.front().first;
            int col = q.front().second;
            int currentDist = dist[row][col];   // distance of current cell
            q.pop();

            for(int i = 0; i < 4; i++) {
                int newRow = row + dRows[i];
                int newCol = col + dCols[i];
                int newDistance = currentDist + 1;

                if(newRow >= 0 && newRow < n &&
                   newCol >= 0 && newCol < m &&
                   !vis[newRow][newCol]) {          // in bounds + unvisited

                    vis[newRow][newCol] = 1;         // mark visited on enqueue
                    q.push({newRow, newCol});
                    dist[newRow][newCol] = newDistance; // set shortest distance
                }
            }
        }

        return dist;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M)`

| Step | Cost | Reason |
|---|---|---|
| Initial grid scan | `O(N × M)` | Visit every cell to find 0s |
| BFS | `O(N × M)` | Each cell enqueued and dequeued at most once |
| Per cell: 4 direction checks | `O(4)` = `O(1)` | Constant |

**Total: `O(N × M)`**

> Each cell is enqueued exactly once (marked visited on enqueue prevents re-entry). So total BFS work = total cells = `N × M`.

---

### Space Complexity — `O(N × M)`

| Structure | Size | Reason |
|---|---|---|
| `vis[][]` | `O(N × M)` | One entry per cell |
| `dist[][]` | `O(N × M)` | One entry per cell (output) |
| BFS queue | `O(N × M)` | Worst case: all cells are 0 — all enqueued simultaneously |

**Total: `O(3 × N × M)` = `O(N × M)`**

---

### Naive vs Optimal

| Approach | Time | Space |
|---|---|---|
| BFS from each `1` separately | `O((NM)²)` | `O(NM)` |
| **Multi-source BFS from all `0`s** | **`O(NM)`** | **`O(NM)`** |

> For a 1000×1000 grid: naive = 10¹² operations, optimal = 10⁶ operations. The reversal of perspective gives a **1,000,000× speedup**.
