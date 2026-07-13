# Rotting Oranges

---

## 1. Intuition / Approach

### Problem Understanding

A grid has three types of cells:
- `0` → empty
- `1` → fresh orange
- `2` → rotten orange

Every minute, a rotten orange spreads rot to all **4-directionally adjacent** fresh oranges simultaneously. Find the minimum time for all fresh oranges to rot. If impossible, return `-1`.

---

### Why BFS and Not DFS?

This is a classic **multi-source BFS** problem.

The rot spreads **simultaneously in all directions from all rotten oranges at the same time** — not sequentially from one rotten orange at a time. This is exactly the "ripple / level-by-level" behavior that BFS models perfectly.

DFS would explore one path to its deepest point before backtracking — it can't model simultaneous spreading.

**Multi-source** means we don't start BFS from just one node — we start from **all initially rotten oranges at the same time** (level 0). Then all their fresh neighbors rot at minute 1, then those neighbors at minute 2, and so on.

---

### The Key Observations

**1. Time = BFS level**  
The minute at which an orange rots = the BFS level it's discovered at. The answer is the maximum level reached.

**2. Count fresh oranges**  
To detect the impossible case (`-1`), we count fresh oranges upfront (`cntFresh`). After BFS, if the number of oranges we actually made rotten (`cnt`) doesn't equal `cntFresh`, some fresh orange was unreachable → return `-1`.

**3. `vis[][]` acts as both visited array and state tracker**  
We copy the grid into `vis` so we can modify it without altering the original. Marking `vis[r][c] = 2` when an orange rots serves as the "visited" marker — preventing re-processing.

**4. Direction arrays**  
```cpp
int delRow[4] = {-1, 0, +1, 0};
int delCol[4] = { 0,+1,  0,-1};
```
This is a clean trick to try all 4 directions (up, right, down, left) with a loop instead of 4 separate if-statements.

```
Index 0: (-1, 0) → up
Index 1: ( 0,+1) → right
Index 2: (+1, 0) → down
Index 3: ( 0,-1) → left
```

---

### Algorithm

```
Step 1: Scan grid
  - Copy to vis[][]
  - Push all rotten oranges into queue with time=0
  - Count total fresh oranges (cntFresh)

Step 2: Multi-source BFS
  - Pop (row, col, time) from queue
  - Update tm = max(tm, time)
  - For each of 4 neighbors:
      If in bounds AND vis == 1 (fresh):
        Mark vis = 2 (rotten)
        Push (neighbor, time+1)
        cnt++

Step 3: Check
  - If cnt != cntFresh → return -1 (some orange never rotted)
  - Else return tm (maximum time reached)
```

---

## 2. Dry Run

```
Grid:
[ 2  1  1 ]
[ 1  1  0 ]
[ 0  1  1 ]
```

**Step 1 — Initialize:**
```
vis = same as grid
cntFresh = 6 (six 1s)
queue = [((0,0), 0)]   ← only one initially rotten orange at (0,0)
```

**Step 2 — BFS:**

```
Minute 0: pop (0,0,t=0)
  tm = max(0,0) = 0
  Check 4 neighbors of (0,0):
    up:    (-1,0) → out of bounds
    right: (0,1)  → vis=1 ✅ → mark 2, push ((0,1),1), cnt=1
    down:  (1,0)  → vis=1 ✅ → mark 2, push ((1,0),1), cnt=2
    left:  (0,-1) → out of bounds

queue = [((0,1),1), ((1,0),1)]
vis:
[ 2  2  1 ]
[ 2  1  0 ]
[ 0  1  1 ]
```

```
Minute 1: pop (0,1,t=1)
  tm = max(0,1) = 1
  Neighbors of (0,1):
    up:    (-1,1) → OOB
    right: (0,2)  → vis=1 ✅ → mark 2, push ((0,2),2), cnt=3
    down:  (1,1)  → vis=1 ✅ → mark 2, push ((1,1),2), cnt=4
    left:  (0,0)  → vis=2 → skip

Minute 1: pop (1,0,t=1)
  tm = max(1,1) = 1
  Neighbors of (1,0):
    up:    (0,0)  → vis=2 → skip
    right: (1,1)  → vis=2 (just marked) → skip
    down:  (2,0)  → vis=0 → skip (empty cell)
    left:  (1,-1) → OOB

queue = [((0,2),2), ((1,1),2)]
vis:
[ 2  2  2 ]
[ 2  2  0 ]
[ 0  1  1 ]
```

```
Minute 2: pop (0,2,t=2)
  tm = max(1,2) = 2
  Neighbors of (0,2):
    right: OOB
    down:  (1,2) → vis=0 (empty) → skip
    left:  (0,1) → vis=2 → skip
    up:    OOB

Minute 2: pop (1,1,t=2)
  tm = max(2,2) = 2
  Neighbors of (1,1):
    up:    (0,1) → vis=2 → skip
    right: (1,2) → vis=0 → skip
    down:  (2,1) → vis=1 ✅ → mark 2, push ((2,1),3), cnt=5
    left:  (1,0) → vis=2 → skip

queue = [((2,1),3)]
vis:
[ 2  2  2 ]
[ 2  2  0 ]
[ 0  2  1 ]
```

```
Minute 3: pop (2,1,t=3)
  tm = max(2,3) = 3
  Neighbors of (2,1):
    up:    (1,1) → vis=2 → skip
    right: (2,2) → vis=1 ✅ → mark 2, push ((2,2),4), cnt=6
    down:  OOB
    left:  (2,0) → vis=0 → skip

queue = [((2,2),4)]
vis:
[ 2  2  2 ]
[ 2  2  0 ]
[ 0  2  2 ]
```

```
Minute 4: pop (2,2,t=4)
  tm = max(3,4) = 4
  Neighbors: all OOB or vis=2 → nothing pushed

queue = []  → BFS ends
```

**Step 3 — Check:**
```
cnt = 6 == cntFresh = 6 ✅
return tm = 4 ✅
```

---

## 3. Code

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int n = grid.size();
        int m = grid[0].size();

        // Queue stores: {{row, col}, time}
        queue<pair<pair<int,int>, int>> q;

        // vis acts as visited + state (0=empty, 1=fresh, 2=rotten)
        int vis[n][m];

        int cntFresh = 0;

        // Step 1: Initialize vis, enqueue all rotten oranges, count fresh
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                vis[i][j] = grid[i][j];

                if(grid[i][j] == 2) {
                    q.push({{i, j}, 0});    // all rotten oranges start at time=0
                }
                if(grid[i][j] == 1)
                    cntFresh++;
            }
        }

        // Direction arrays for 4-directional movement
        int delRow[4] = {-1, 0, +1,  0};
        int delCol[4] = { 0,+1,  0, -1};

        int tm = 0;     // tracks maximum time reached
        int cnt = 0;    // counts how many fresh oranges we actually rotted

        // Step 2: Multi-source BFS
        while(!q.empty()) {
            int r = q.front().first.first;
            int c = q.front().first.second;
            int t = q.front().second;
            q.pop();

            tm = max(tm, t);   // update max time

            for(int i = 0; i < 4; i++) {
                int nRow = r + delRow[i];
                int nCol = c + delCol[i];

                // Valid cell + fresh orange
                if(nRow >= 0 && nRow < n &&
                   nCol >= 0 && nCol < m &&
                   vis[nRow][nCol] == 1) {

                    vis[nRow][nCol] = 2;            // mark as rotten
                    q.push({{nRow, nCol}, t + 1});  // spread rot at t+1
                    cnt++;                          // one more fresh orange rotted
                }
            }
        }

        // Step 3: If not all fresh oranges rotted → impossible
        if(cnt != cntFresh)
            return -1;

        return tm;
    }
};
```

> **Note:** The condition `vis[nRow][nCol] != 2 && vis[nRow][nCol] == 1` in the original code is redundant — `== 1` already guarantees `!= 2`. Simplified to just `== 1` above.

---

## 4. Time & Space Complexity

### Time Complexity — `O(N × M)`

| Step | Cost | Reason |
|---|---|---|
| Initial grid scan | `O(N × M)` | Visit every cell once to fill vis, queue, count fresh |
| BFS | `O(N × M)` | Every cell is enqueued and dequeued at most once |
| Per cell: check 4 directions | `O(4)` = `O(1)` | Constant directions |

**Total: `O(N × M) + O(N × M)` = `O(N × M)`**

> Each cell is processed at most once in BFS — the `vis[r][c] = 2` mark ensures it's never re-enqueued. So total BFS work = total cells = `N × M`.

---

### Space Complexity — `O(N × M)`

| Structure | Size | Reason |
|---|---|---|
| `vis[][]` array | `O(N × M)` | Copy of the entire grid |
| BFS queue | `O(N × M)` | Worst case: all cells are rotten and enqueued simultaneously |

**Total: `O(N × M)`**

---

### Impossible Case — Why `cnt != cntFresh`?

```
Grid:
[ 2  1  0  1 ]

Orange at (0,3) is blocked by empty cell at (0,2)
Rot can never reach it

cntFresh = 2
After BFS, cnt = 1   (only (0,1) got rotted)
cnt != cntFresh → return -1 ✅
```
