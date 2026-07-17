# Number of Islands

---

## 1. Problem Statement

Given an `n × m` 2D grid of characters where:
- `'1'` represents land
- `'0'` represents water

Return the **number of islands**.

An island is a group of `'1'`s connected **4-directionally** (up, down, left, right). The grid is surrounded by water on all edges.

```
Input:
[ 1  1  1  1  0 ]
[ 1  1  0  1  0 ]
[ 1  1  0  0  0 ]
[ 0  0  0  0  0 ]

Output: 1  (all land is connected into one island)

Input:
[ 1  1  0  0  0 ]
[ 1  1  0  0  0 ]
[ 0  0  1  0  0 ]
[ 0  0  0  1  1 ]

Output: 3  (three separate islands)
```

---

## 2. Intuition / Approach

### What is an Island?

An island is simply a **connected component of `'1'` cells**. So this problem is directly asking: **how many connected components of `'1'`s exist in the grid?**

This is the **grid version of "Number of Provinces"** — same pattern, different representation.

---

### The Algorithm

1. Scan every cell in the grid
2. When you find an unvisited `'1'`:
   - It's a new island → increment `islandCnt`
   - Run BFS from it → marks all connected `'1'`s as visited
3. Future iterations skip visited cells — so each island is counted exactly once

```
Finding unvisited '1' = discovering a new island
BFS from it = mapping the entire island (marking all of it visited)
Next unvisited '1' = a different island
```

---

### Why BFS (Not DFS)?

Both work identically here. BFS is used to avoid potential recursion stack overflow on large grids (DFS could go very deep for large islands). The traversal direction doesn't affect the count.

---

### Why `vis[][]` Separate From Grid?

We could mark visited cells by changing `grid[i][j] = '0'` in-place (destroying the island). That works, but:
- It **mutates the input** — bad practice
- A separate `vis[][]` is cleaner and doesn't corrupt original data

---

### The BFS Helper is Passed the Queue by Reference

```cpp
void bfs(int n, int m, queue<pair<int,int>>& q, ...)
```

The queue is pre-seeded with the starting land cell before `bfs()` is called. Passing by reference lets `bfs()` drain that queue completely (visiting the whole island) without copying.

---

## 3. Dry Run

```
Grid:
[ 1  1  0  0  0 ]
[ 1  1  0  0  0 ]
[ 0  0  1  0  0 ]
[ 0  0  0  1  1 ]

n=4, m=5
vis = all 0s, islandCnt=0
```

---

**Scanning... i=0, j=0:**
```
grid[0][0]='1', vis=0 → NEW ISLAND → islandCnt=1
push (0,0), vis[0][0]=1
call bfs():

  Pop (0,0):
    up    (-1,0): OOB
    down  (1,0):  '1', vis=0 ✅ → push (1,0), vis[1][0]=1
    left  (0,-1): OOB
    right (0,1):  '1', vis=0 ✅ → push (0,1), vis[0][1]=1

  Pop (1,0):
    up    (0,0):  vis=1 → skip
    down  (2,0):  '0' → skip
    left  (1,-1): OOB
    right (1,1):  '1', vis=0 ✅ → push (1,1), vis[1][1]=1

  Pop (0,1):
    up    (-1,1): OOB
    down  (1,1):  vis=1 → skip
    left  (0,0):  vis=1 → skip
    right (0,2):  '0' → skip

  Pop (1,1):
    up    (0,1):  vis=1 → skip
    down  (2,1):  '0' → skip
    left  (1,0):  vis=1 → skip
    right (1,2):  '0' → skip

  Queue empty → bfs() returns

vis:
[ 1  1  0  0  0 ]
[ 1  1  0  0  0 ]
[ 0  0  0  0  0 ]
[ 0  0  0  0  0 ]
```

---

**Continue scanning... i=0,j=1 → vis=1 skip | i=0,j=2,3,4 → '0' skip**
**i=1,j=0,1 → vis=1 skip | j=2,3,4 → '0' skip**
**i=2,j=0,1 → '0' skip**

**i=2, j=2:**
```
grid[2][2]='1', vis=0 → NEW ISLAND → islandCnt=2
push (2,2), vis[2][2]=1
call bfs():

  Pop (2,2):
    up    (1,2): '0' → skip
    down  (3,2): '0' → skip
    left  (2,1): '0' → skip
    right (2,3): '0' → skip

  Queue empty → bfs() returns (island of size 1)
```

---

**i=3, j=0,1,2 → '0' skip**

**i=3, j=3:**
```
grid[3][3]='1', vis=0 → NEW ISLAND → islandCnt=3
push (3,3), vis[3][3]=1
call bfs():

  Pop (3,3):
    up    (2,3): '0' → skip
    down  (4,3): OOB
    left  (3,2): '0' → skip
    right (3,4): '1', vis=0 ✅ → push (3,4), vis[3][4]=1

  Pop (3,4):
    up    (2,4): '0' → skip
    down  (4,4): OOB
    left  (3,3): vis=1 → skip
    right (3,5): OOB

  Queue empty → bfs() returns
```

**i=3, j=4 → vis=1 → skip**

**Scan complete.**

**Return `islandCnt = 3` ✅**

---

## 4. Story Points

---

**Story Point 1 — "Island = connected component of '1's"**

Once you recognize this, the entire solution follows naturally. Number of islands = number of times we start a fresh BFS (each fresh start = undiscovered component = new island). This is identical to the "Number of Provinces" problem — just a grid instead of an adjacency matrix.

---

**Story Point 2 — "Seed the queue BEFORE calling bfs()"**

```cpp
q.push({i, j});
vis[i][j] = 1;
islandCnt++;
bfs(n, m, q, vis, grid, dRow, dCol);
```

The starting cell is pushed and marked visited in `numIslands()`, then `bfs()` takes over. This is why the queue is passed by reference — `bfs()` drains the queue that was already seeded.

Alternative: seed inside `bfs()`. Both work, but this design keeps the "discovery" logic in the outer function and the "expansion" logic in `bfs()`.

---

**Story Point 3 — "vis marked on ENQUEUE prevents duplicate processing"**

When BFS discovers a neighbor, it marks it visited AND enqueues it immediately. If another path also reaches this same cell later, `!vis[nrow][ncol]` fails → skipped. This ensures each cell is processed exactly once.

---

**Story Point 4 — "Water cells ('0') are never visited or enqueued"**

```cpp
if(... && grid[nrow][ncol] == '1' && !vis[nrow][ncol])
```

Both conditions must hold. Water acts as a natural barrier — BFS can never cross water, so it stays within the current island's land mass. This is what gives islands their "surrounded by water" property.

---

**Story Point 5 — "Single-cell islands work automatically"**

If a `'1'` cell has no `'1'` neighbors:
- We push it, call `bfs()`
- BFS pops it, finds no valid neighbors → queue empty → returns immediately
- That single cell = an island of size 1

No special handling needed.

---

## 5. Code

```cpp
class Solution {
private:
    void bfs(int n, int m, queue<pair<int,int>>& q,
             vector<vector<int>>& vis,
             vector<vector<char>>& grid,
             const int dRow[], const int dCol[]) {

        while(!q.empty()) {
            int row = q.front().first;
            int col = q.front().second;
            q.pop();

            // Explore all 4 directions
            for(int i = 0; i < 4; i++) {
                int nRow = row + dRow[i];
                int nCol = col + dCol[i];

                // In bounds + land cell + unvisited
                if(nRow >= 0 && nRow < n &&
                   nCol >= 0 && nCol < m &&
                   grid[nRow][nCol] == '1' &&
                   !vis[nRow][nCol]) {
                    vis[nRow][nCol] = 1;         // mark on enqueue
                    q.push({nRow, nCol});
                }
            }
        }
    }

public:
    int numIslands(vector<vector<char>>& grid) {
        int n = grid.size();
        int m = grid[0].size();

        vector<vector<int>> vis(n, vector<int>(m, 0));
        queue<pair<int,int>> q;

        int islandCnt = 0;

        const int dRow[4] = {-1, +1, 0,  0};
        const int dCol[4] = { 0,  0,-1, +1};

        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(grid[i][j] == '1' && !vis[i][j]) {
                    // New unvisited land → new island found
                    q.push({i, j});
                    vis[i][j] = 1;
                    islandCnt++;
                    bfs(n, m, q, vis, grid, dRow, dCol);  // map entire island
                }
            }
        }

        return islandCnt;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M)`

| Step | Cost | Reason |
|---|---|---|
| Outer grid scan | `O(N × M)` | Every cell checked once |
| All BFS calls combined | `O(N × M)` | `vis[][]` is global — each cell enqueued and dequeued at most once across ALL BFS calls |
| Per cell: 4 direction checks | `O(4)` = `O(1)` | Constant |

**Total: `O(N × M)`**

> The key insight: even though `bfs()` is called multiple times (once per island), the total work across all calls = N×M. The visited array prevents any cell from being processed twice — ever.

---

### Space Complexity — `O(N × M)`

| Structure | Size | Reason |
|---|---|---|
| `vis[][]` | `O(N × M)` | One entry per cell |
| BFS queue | `O(N × M)` | Worst case: entire grid is `'1'` (one giant island) → all cells enqueued |

**Total: `O(N × M)`**

---

### Number of Islands vs Number of Provinces

| | Number of Islands | Number of Provinces |
|---|---|---|
| **Input** | 2D char grid | Adjacency matrix / edge list |
| **Node** | Grid cell `(i,j)` | Integer node `0..V-1` |
| **Edge** | 4-directional adjacency | Explicit edge in matrix/list |
| **Component** | Island of `'1'` cells | Connected group of cities |
| **Algorithm** | Identical — BFS/DFS per component | Identical |
| **TC** | `O(N × M)` | `O(V + 2E)` |
