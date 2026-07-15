# Number of Enclaves

---

## 1. Problem Statement

You are given an `n × m` binary grid where:
- `0` → sea cell
- `1` → land cell

A **land cell** can "walk off" the boundary if there exists a 4-directional path from it to any border cell.

Return the **count of land cells** from which you **cannot** walk off the boundary of the grid — these are called **enclaves**.

```
Input:
[ 0  0  0  0 ]
[ 1  0  1  0 ]
[ 0  1  1  0 ]
[ 0  0  0  0 ]

Output: 3
```

The `1` at `(1,0)` touches the border → can walk off → not an enclave.  
The `1`s at `(1,2)`, `(2,1)`, `(2,2)` are fully enclosed → **3 enclaves**.

---

## 2. Intuition / Approach

### The Core Insight — Same Pattern as Surrounded Regions

This problem is the **numerical cousin** of "Surrounded Regions". The logic is identical:

- In Surrounded Regions → flip surrounded `'O'`s to `'X'`
- In Number of Enclaves → **count** surrounded `1`s

The approach is exactly the same reversal trick:

> **Don't find which land cells are enclosed. Find which land cells can escape (border-connected). Count everything else.**

---

### Why Multi-Source BFS From the Border?

Any land cell that touches the border can "walk off". But a land cell deep inside the grid can ALSO walk off — if it has a chain of `1`s leading to the border.

Multi-source BFS from all border `1`s naturally propagates the "can escape" marking inward through connected land cells. Every `1` that BFS reaches = not an enclave.

```
Border 1s → BFS marks all connected 1s → remaining 1s are enclaves
```

---

### Why BFS Instead of DFS Here?

Both work. BFS is chosen here (vs DFS in Surrounded Regions) to show that the pattern works with either traversal. BFS avoids recursion stack overflow risk on large grids.

---

### The Three Steps

```
Step 1: Seed BFS with all land cells (grid==1) on the border
        Mark them vis=1 immediately

Step 2: BFS expands to all 1-cells connected to the border
        Any reachable 1-cell gets vis=1 (= "can escape")

Step 3: Count cells where grid==1 AND vis==0
        These are the enclaves — land cells that cannot escape
```

---

## 3. Dry Run

```
Grid (4×4):
[ 0  0  0  0 ]
[ 1  0  1  0 ]
[ 0  1  1  0 ]
[ 0  0  0  0 ]

n=4, m=4
vis = all 0s
```

---

**Step 1 — Scan borders for land cells:**

```
i=0 (first row):  all 0s → nothing
i=3 (last row):   all 0s → nothing
j=0 (first col):  (0,0)=0, (1,0)=1 ✅, (2,0)=0, (3,0)=0
j=3 (last col):   all 0s → nothing

Seed: (1,0) → push to queue, vis[1][0]=1

vis:
[ 0  0  0  0 ]
[ 1  0  0  0 ]
[ 0  0  0  0 ]
[ 0  0  0  0 ]

queue = [(1,0)]
```

---

**Step 2 — BFS:**

**Pop (1,0):**
```
Neighbors:
  up    (0,0): grid=0 → skip
  right (1,1): grid=0 → skip
  down  (2,0): grid=0 → skip
  left  (1,-1): OOB → skip

Nothing enqueued.
queue = []
BFS ends.
```

```
vis after BFS:
[ 0  0  0  0 ]
[ 1  0  0  0 ]
[ 0  0  0  0 ]
[ 0  0  0  0 ]
```

---

**Step 3 — Count enclaves:**

```
Scan all cells where grid==1 AND vis==0:
  (1,2): grid=1, vis=0 ✅ → landCnt=1
  (2,1): grid=1, vis=0 ✅ → landCnt=2
  (2,2): grid=1, vis=0 ✅ → landCnt=3

return 3 ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Scan borders in one pass vs four passes"**

The code uses a single nested loop with a border condition:

```cpp
if(i == 0 || i == n-1 || j == 0 || j == m-1)
```

This is more compact than scanning first row, last row, first col, last col separately (like in Surrounded Regions). Both are equivalent — single pass is slightly cleaner.

---

**Story Point 2 — "The border condition avoids double-counting corners"**

Corner cells like `(0,0)` satisfy both `i==0` AND `j==0`. The condition:

```cpp
if(grid[i][j] == 1) {
    q.push({i,j});
    vis[i][j] = 1;
}
```

The `vis[i][j]=1` on push ensures even if a corner is considered twice (from row scan and col scan within the same loop), it's only pushed once. Actually, in the single-loop approach here, corners are hit exactly once — no issue at all.

---

**Story Point 3 — "BFS marks ALL land connected to border, not just border-adjacent"**

```
Border 1 at (1,0) —— connects to nothing here.
But consider:

[ 0  0  0  0 ]
[ 1  1  1  0 ]
[ 0  0  1  0 ]
[ 0  0  0  0 ]

BFS from (1,0) reaches (1,1) → (1,2) → (2,2)
All four 1s are marked visited → ALL escape → 0 enclaves

The chain reaction through BFS is what handles inland cells.
```

---

**Story Point 4 — "This is the counting version of Surrounded Regions"**

| | Surrounded Regions | Number of Enclaves |
|---|---|---|
| Data type | `char` (`O`/`X`) | `int` (`0`/`1`) |
| Traversal | DFS | BFS |
| Final step | Flip unvisited `O` → `X` | Count unvisited `1`s |
| Output | Modified board | Integer count |
| Core logic | Identical | Identical |

Same pattern, different output. Recognizing this lets you solve one immediately after the other.

---

**Story Point 5 — "Why not just count all 1s and subtract border-connected 1s?"**

You could! Count total `1`s upfront, then subtract the count of `1`s marked `vis=1` after BFS. Same answer. The current approach (counting unvisited `1`s at the end) is equivalent and slightly more direct.

---

## 5. Code

```cpp
class Solution {
public:
    int numEnclaves(vector<vector<int>>& grid) {
        int n = grid.size();
        int m = grid[0].size();

        vector<vector<int>> vis(n, vector<int>(m, 0));
        queue<pair<int,int>> q;

        // Step 1: Seed BFS with all border land cells
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                // Check if cell is on the border
                if(i == 0 || i == n-1 || j == 0 || j == m-1) {
                    if(grid[i][j] == 1) {
                        q.push({i, j});
                        vis[i][j] = 1;   // mark on enqueue
                    }
                }
            }
        }

        int dRow[4] = {-1,  0, +1, 0};
        int dCol[4] = { 0, +1,  0,-1};

        // Step 2: BFS — mark all border-connected land cells as "can escape"
        while(!q.empty()) {
            int row = q.front().first;
            int col = q.front().second;
            q.pop();

            for(int i = 0; i < 4; i++) {
                int nRow = row + dRow[i];
                int nCol = col + dCol[i];

                // In bounds + land + unvisited
                if(nRow >= 0 && nRow < n &&
                   nCol >= 0 && nCol < m &&
                   grid[nRow][nCol] == 1 &&
                   vis[nRow][nCol] == 0) {
                    q.push({nRow, nCol});
                    vis[nRow][nCol] = 1;
                }
            }
        }

        // Step 3: Count land cells that couldn't escape (unvisited 1s = enclaves)
        int landCnt = 0;
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(vis[i][j] == 0 && grid[i][j] == 1)
                    landCnt++;
            }
        }

        return landCnt;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M)`

| Step | Cost | Reason |
|---|---|---|
| Border scan | `O(N × M)` | Single loop over all cells with border check |
| BFS | `O(N × M)` | Each cell enqueued and dequeued at most once |
| Final count scan | `O(N × M)` | Full grid scan |

**Total: `O(N × M)`**

---

### Space Complexity — `O(N × M)`

| Structure | Size | Reason |
|---|---|---|
| `vis[][]` | `O(N × M)` | One entry per cell |
| BFS queue | `O(N × M)` | Worst case: entire border is `1` → huge initial queue, plus chain reactions inward |

**Total: `O(N × M)`**
