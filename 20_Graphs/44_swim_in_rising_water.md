# Swim in Rising Water

---

## 1. Problem Statement

You are given an `n × n` grid where `grid[i][j]` represents the elevation of cell `(i,j)`. Rain causes the water level to rise — at time `t`, you can swim in any cell whose elevation is **≤ t**.

You start at `(0,0)` and want to reach `(n-1, n-1)`. You can swim to adjacent cells (4-directional) if the water level covers both cells.

Return the **minimum time** `t` such that there exists a path from `(0,0)` to `(n-1, n-1)`.

```
Grid:
0  2
1  3

Time 0: only (0,0) reachable
Time 1: (0,0) and (1,0) reachable
Time 2: (0,0),(1,0),(0,1) reachable
Time 3: all cells reachable
  Path: (0,0)→(1,0)→... not connected yet at t=2
  At t=3: (0,0)→(0,1)→(1,1) works? Need t≥grid[0][1]=2 AND t≥grid[1][1]=3 → t=3

Answer: 3
```

---

## 2. Intuition / Approach

### The Core Insight — Minimize the Maximum Cell Value

The "time" to traverse a path is determined by the **highest elevation cell on that path** — you must wait until the water level reaches that height before you can enter that cell.

So the time to travel any path = `max(grid[i][j])` over all cells `(i,j)` on the path.

We want: **the path from `(0,0)` to `(n-1,n-1)` that minimizes the maximum elevation along it**.

This is the exact same "minimize maximum" problem as **Path with Minimum Effort** — but instead of height differences, the cost is the cell's raw value.

---

### Connection to Minimum Effort Path

| | Minimum Effort | Swim in Rising Water |
|---|---|---|
| **Edge weight** | `\|height[u] - height[v]\|` | `max(elevation[u], elevation[v])` |
| **Path cost** | Max difference along path | Max elevation along path |
| **Goal** | Minimize max difference | Minimize max elevation |
| **Algorithm** | Modified Dijkstra | Modified Dijkstra |
| **Relaxation** | `max(diff, currTime)` | `max(elevation, currTime)` |

Same pattern: **Dijkstra where "distance" = running maximum instead of sum**.

---

### Why Dijkstra Works Here

Dijkstra's greedy finalization works when the "cost" function is monotonically non-decreasing as we extend a path. Here:

```
currTime = max elevation seen so far on path
When we extend to a new cell: max(currTime, grid[newCell]) ≥ currTime
```

The maximum only stays the same or increases → monotone non-decreasing → Dijkstra's finalization is valid → when we pop a cell, its `dist` is the true minimum time to reach it.

---

### `dist[row][col]` Meaning

`dist[row][col]` = minimum `t` (maximum elevation) required to reach `(row,col)` from `(0,0)` via any path.

Initialization:
- `dist[0][0] = grid[0][0]` — you must wait at least until `t = grid[0][0]` to start
- `dist[all others] = INT_MAX`

---

### The Relaxation Step

```cpp
int maxTime = max(currTime, grid[newRow][newCol]);
if(maxTime < dist[newRow][newCol]) {
    dist[newRow][newCol] = maxTime;
    pq.push({maxTime, newRow, newCol});
}
```

`maxTime` = the time needed to reach `(newRow,newCol)` via the current path:
- `currTime` = max elevation seen so far (to reach current cell)
- `grid[newRow][newCol]` = elevation of the new cell (must wait until this too)
- `max(...)` = the binding constraint — whichever is higher

Update `dist` and push only if this is a shorter (lower max) path.

---

### The Tuple `TIII = tuple<int,int,int>`

The heap stores `{maxTime, row, col}` as a 3-element tuple. Using `tuple` avoids nested pairs:

```cpp
// Without tuple (nested pair):
pair<int, pair<int,int>>

// With tuple (cleaner):
tuple<int, int, int>
auto [currTime, row, col] = pq.top();   // structured binding
```

Min-heap orders by first element (maxTime) → smallest maxTime popped first → greedy minimum.

---

## 3. Dry Run

```
Grid:
0  2
1  3

n=2
dist = [[0, INF], [INF, INF]]
pq   = [{0, 0, 0}]
```

**Pop {0, (0,0)}:**
```
currTime=0, dist[0][0]=0 → 0>0? NO → process
(0,0) ≠ destination

Neighbors:
  down  (1,0): maxTime=max(0, grid[1][0])=max(0,1)=1 < INF → dist[1][0]=1, push {1,1,0}
  right (0,1): maxTime=max(0, grid[0][1])=max(0,2)=2 < INF → dist[0][1]=2, push {2,0,1}

dist = [[0,2],[1,INF]]
pq   = [{1,1,0}, {2,0,1}]
```

**Pop {1, (1,0)}:**
```
currTime=1, dist[1][0]=1 → 1>1? NO → process
(1,0) ≠ destination

Neighbors:
  up    (0,0): maxTime=max(1,0)=1, dist[0][0]=0 → 1<0? NO
  right (1,1): maxTime=max(1, grid[1][1])=max(1,3)=3 < INF → dist[1][1]=3, push {3,1,1}
  down  (2,0): invalid
  left  (1,-1): invalid

dist = [[0,2],[1,3]]
pq   = [{2,0,1}, {3,1,1}]
```

**Pop {2, (0,1)}:**
```
currTime=2, dist[0][1]=2 → 2>2? NO → process
(0,1) ≠ destination

Neighbors:
  left  (0,0): max(2,0)=2, dist[0][0]=0 → 2<0? NO
  down  (1,1): max(2,3)=3, dist[1][1]=3 → 3<3? NO
  up, right: invalid

No updates.
pq   = [{3,1,1}]
```

**Pop {3, (1,1)}:**
```
currTime=3, dist[1][1]=3 → 3>3? NO → process
(1,1) == destination (n-1=1, m-1=1) → return 3 ✅
```

**Answer: 3**

Interpretation: Must wait until time `t=3` when all cells `{0,1,2,3}` are underwater, then swim `(0,0)→(0,1)→(1,1)` or `(0,0)→(1,0)→(1,1)`.

---

## 4. Dry Run with Larger Grid

```
Grid:
0  1  2  3  4
24 23 22 21 5
12 13 14 15 16  ← wait what? Let me use a simpler one
```

```
Grid:
3  2
0  1

Start at (0,0)=3. Must wait t≥3 to even start.

pq = [{3,0,0}], dist[0][0]=3

Pop {3,(0,0)}: destination? no
  right (0,1): max(3,2)=3, dist[0][1]=3, push {3,0,1}
  down  (1,0): max(3,0)=3, dist[1][0]=3, push {3,1,0}

Pop {3,(0,1)}:
  down (1,1): max(3,1)=3, dist[1][1]=3, push {3,1,1}

Pop {3,(1,0)}:
  right (1,1): max(3,1)=3 → 3<3? NO (already 3)

Pop {3,(1,1)}: destination! return 3 ✅
```

---

## 5. Story Points

---

**Story Point 1 — "dist[row][col] = minimum MAXIMUM elevation to reach that cell"**

This is not a sum like standard Dijkstra. It's the **bottleneck** — the highest-elevation cell you're forced to cross to reach `(row,col)` via the best path. Minimizing this bottleneck = finding the path that avoids the highest peaks as much as possible.

---

**Story Point 2 — "Same pattern as Minimum Effort Path — just different cost function"**

```
Minimum Effort:     maxTime = max(currTime, |grid[row][col] - grid[newRow][newCol]|)
Swim in Water:      maxTime = max(currTime, grid[newRow][newCol])
```

The ONLY difference is what we take the max with. The rest of the algorithm — Dijkstra structure, stale check, early exit — is identical. Recognizing this pattern lets you solve both problems from one template.

---

**Story Point 3 — "Why not BFS? All 'edges' are not weight 1"**

Going from cell A to cell B costs `max(elevation[A], elevation[B])` — this varies. BFS only works when all edges cost the same. Here we need priority-ordered processing (cheapest path first) → min-heap → Dijkstra.

---

**Story Point 4 — "`dist[0][0] = grid[0][0]` not 0"**

Even at the starting cell, you must wait for the water to rise to `grid[0][0]`. If `grid[0][0] = 5`, you can't even begin until `t=5`. Initializing with `grid[0][0]` correctly captures this mandatory waiting time.

---

**Story Point 5 — "Stale entry check prevents incorrect early returns"**

```cpp
if(currTime > dist[row][col]) continue;
```

If a better path to `(row,col)` was found after this entry was pushed, this entry is stale. Without the check, a stale entry might trigger the early return `if(row==n-1 && col==n-1) return currTime` with a non-optimal time. The check must come BEFORE the destination check.

---

## 6. Code

```cpp
class Solution {
private:
    bool isValid(int row, int col, int n) {
        return row >= 0 && row < n && col >= 0 && col < n;
    }

public:
    using TIII = tuple<int,int,int>;   // {maxTime, row, col}

    int swimInWater(vector<vector<int>>& grid) {
        int n = grid.size();

        // dist[r][c] = minimum max-elevation to reach (r,c) from (0,0)
        vector<vector<int>> dist(n, vector<int>(n, INT_MAX));

        // Min-heap ordered by maxTime
        priority_queue<TIII, vector<TIII>, greater<TIII>> pq;

        dist[0][0] = grid[0][0];   // must wait until water covers starting cell
        pq.push({grid[0][0], 0, 0});

        const int dRow[4] = {-1,+1, 0, 0};
        const int dCol[4] = { 0, 0,-1,+1};

        while(!pq.empty()) {
            auto [currTime, row, col] = pq.top();
            pq.pop();

            // Stale entry check — must come before destination check
            if(currTime > dist[row][col]) continue;

            // Destination reached with minimum time
            if(row == n-1 && col == n-1)
                return currTime;

            for(int i = 0; i < 4; i++) {
                int newRow = row + dRow[i];
                int newCol = col + dCol[i];

                if(!isValid(newRow, newCol, n)) continue;

                // Time to enter new cell = max(current path max, new cell's elevation)
                int maxTime = max(currTime, grid[newRow][newCol]);

                // Relax: update if this path has lower bottleneck time
                if(maxTime < dist[newRow][newCol]) {
                    dist[newRow][newCol] = maxTime;
                    pq.push({maxTime, newRow, newCol});
                }
            }
        }

        return -1;   // unreachable (shouldn't happen for valid input)
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(n² log n)`

| Step | Cost | Reason |
|---|---|---|
| Initialize `dist` | `O(n²)` | n×n grid |
| Each cell: one heap push (at most) | `O(n² × log n²)` = `O(n² log n)` | n² cells, each push `O(log n²)` = `O(log n)` |
| Each cell: 4 direction checks | `O(4)` per cell = `O(1)` | Constant |

**Total: `O(n² log n)`**

> Heap size bounded by `O(n²)` entries → each operation `O(log n²)` = `O(2 log n)` = `O(log n)`.

---

### Space Complexity — `O(n²)`

| Structure | Size | Reason |
|---|---|---|
| `dist[][]` | `O(n²)` | n×n distance matrix |
| Priority queue | `O(n²)` worst case | At most n² entries |

**Total: `O(n²)`**

---

## 8. Algorithm Selector — "Minimize Maximum" Problems

This pattern appears frequently:

| Problem | What we minimize | Edge weight |
|---|---|---|
| Swim in Rising Water | Max cell elevation | `grid[newCell]` |
| Path with Min Effort | Max height difference | `\|grid[u]-grid[v]\|` |
| Min Bottleneck Path | Max edge weight | `edgeWeight` |

All three use the same Dijkstra template with:
```cpp
newCost = max(currCost, relevantWeight);
if(newCost < dist[next]) { ... }
```

The only variation is what `relevantWeight` represents.
