# Number of Islands II (Dynamic Island Addition)

---

## 1. Problem Statement

You are given an `n × m` grid initially filled with water (`0`). You receive a list of `operators` where `operators[i] = [row, col]` turns the cell `(row, col)` into land.

After each operation, return the **current number of islands**.

An island is a group of connected land cells (4-directionally adjacent). Return a list of island counts after each operation.

```
n=4, m=5
operators = [[1,1],[0,1],[3,3],[3,4]]

After op [1,1]: land at (1,1)
  Islands: 1

After op [0,1]: land at (0,1) — adjacent to (1,1)
  Islands: 1 (merged with existing island)

After op [3,3]: land at (3,3) — isolated
  Islands: 2

After op [3,4]: land at (3,4) — adjacent to (3,3)
  Islands: 2 (merged)

Output: [1, 1, 2, 2]
```

---

## 2. Why DSU? — The Dynamic Nature of the Problem

### Why Not BFS/DFS?

BFS/DFS on a static grid computes island count in `O(n×m)`. But here, after **each operation**, we'd need to rerun BFS/DFS from scratch → `O(ops × n×m)`.

For `ops = 10,000` and a `1000×1000` grid: `10,000 × 1,000,000 = 10^10` operations — impossibly slow.

### Why DSU is Perfect Here

This is a **dynamic connectivity** problem — the graph grows one node at a time, and we need connectivity information after each addition. DSU handles this in near `O(1)` per addition:

- Adding a land cell = adding a node to DSU
- Checking if two adjacent land cells are connected = `findUParent`
- Merging two islands = `unionBySize`
- Island count tracked incrementally

Total: `O(ops × α(n×m))` ≈ `O(ops)` — orders of magnitude faster.

---

## 3. The 2D → 1D Node Mapping

The DSU is 1D (indices 0 to n×m-1). Grid cell `(row, col)` maps to:

```
nodeIndex = row × m + col
```

This flattens the 2D grid into a 1D array — the standard technique for grid-based DSU problems.

```
n=3, m=4 grid:
(0,0)=0  (0,1)=1  (0,2)=2  (0,3)=3
(1,0)=4  (1,1)=5  (1,2)=6  (1,3)=7
(2,0)=8  (2,1)=9  (2,2)=10 (2,3)=11
```

---

## 4. The Core Algorithm

For each operation `(row, col)`:

```
Step 1: If cell already visited → push current count, skip
        (duplicate operation — island count unchanged)

Step 2: Mark cell as visited, increment islandCnt
        (new land cell = new potential island)

Step 3: Check all 4 neighbors:
          If neighbor is valid AND already visited (is land):
            If in different component:
              union(currentNode, neighborNode)
              islandCnt--   ← two islands merged into one

Step 4: Push current islandCnt to result
```

**The increment-then-decrement pattern:**
- Every new land cell starts as its own island (+1)
- For each adjacent land cell in a different component → merge → (-1)

```
Example: adding cell (1,1) with 3 adjacent land neighbors in 2 different components

islandCnt++ = existing + 1 (new island)
  merge with component A → islandCnt-- (merged into A)
  neighbor B is also in A → no merge (same component)
  merge with component C → islandCnt-- (C absorbed)

Net change: +1 - 2 = -1 island (two separate islands became one)
```

---

## 5. The Key: Union the ROOTS, Not the Nodes

```cpp
int currentParent  = ds.findUParent(currentNode);
int neighborParent = ds.findUParent(neighborNode);

if(currentParent != neighborParent) {
    ds.unionBySize(currentParent, neighborParent);
    isLandCnt--;
}
```

Why call `findUParent` before `unionBySize`?

`unionBySize` internally calls `findUParent` anyway. But here we explicitly check roots BEFORE calling union — the `if(currentParent != neighborParent)` check requires knowing both roots first to decide whether to decrement `isLandCnt`.

An alternative (and slightly cleaner) pattern:
```cpp
if(ds.unionBySize(currentNode, neighborNode)) {
    isLandCnt--;
}
```
Since `unionBySize` returns `false` when roots are the same (no merge), this skips the decrement correctly.

---

## 6. Duplicate Operation Handling

```cpp
if(visited[row][col] == 1) {
    result.push_back(isLandCnt);   // push unchanged count
    continue;                       // skip processing
}
```

If the same cell is added twice, the second addition does nothing — cell is already land. Island count unchanged → just push the current count.

This is handled BEFORE incrementing `isLandCnt`, so the count is never incorrectly increased for duplicate operations.

---

## 7. Dry Run

```
n=3, m=3, operators = [[0,0],[0,1],[1,1],[1,0]]
DSU: 9 nodes (0..8), all isolated
visited: all 0s
isLandCnt = 0
```

---

**Op [0,0]:**
```
visited[0][0]=0 → new land
visited[0][0]=1, isLandCnt=1
currentNode = 0×3+0 = 0

Check 4 neighbors:
  (-1,0): invalid
  (+1,0): (1,0) visited=0 → skip
  (0,+1): (0,1) visited=0 → skip
  (0,-1): invalid

No merges.
result=[1]

Grid:
L . .
. . .
. . .
```

---

**Op [0,1]:**
```
visited[0][1]=0 → new land
visited[0][1]=1, isLandCnt=2
currentNode = 0×3+1 = 1

Check 4 neighbors:
  (-1,1): invalid
  (+1,1): (1,1) visited=0 → skip
  (0,2):  (0,2) visited=0 → skip
  (0,0):  (0,0) visited=1 ✅ → neighborNode=0
    findUParent(1)=1, findUParent(0)=0
    1 ≠ 0 → unionBySize(0,1) → isLandCnt-- = 1

result=[1,1]

Grid:
L L .
. . .
. . .

DSU: parent[1]=0, size[0]=2
```

---

**Op [1,1]:**
```
visited[1][1]=0 → new land
visited[1][1]=1, isLandCnt=2
currentNode = 1×3+1 = 4

Check 4 neighbors:
  (0,1): visited=1 ✅ → neighborNode=1
    findUParent(4)=4, findUParent(1)=0
    4 ≠ 0 → unionBySize(4,0) → isLandCnt-- = 1
    DSU: parent[4]=0, size[0]=3

  (2,1): visited=0 → skip
  (1,2): visited=0 → skip
  (1,0): visited=0 → skip

result=[1,1,1]

Grid:
L L .
. L .
. . .
```

---

**Op [1,0]:**
```
visited[1][0]=0 → new land
visited[1][0]=1, isLandCnt=2
currentNode = 1×3+0 = 3

Check 4 neighbors:
  (0,0): visited=1 ✅ → neighborNode=0
    findUParent(3)=3, findUParent(0)=0
    3 ≠ 0 → unionBySize(3,0) → isLandCnt-- = 1

  (2,0): visited=0 → skip
  (1,1): visited=1 ✅ → neighborNode=4
    findUParent(3)=0 (just merged!), findUParent(4)=0
    0 == 0 → same component → no merge (no decrement)

  (1,-1): invalid

result=[1,1,1,1]

Grid:
L L .
L L .
. . .

All 4 land cells in one island ✅
```

---

## 8. Story Points

---

**Story Point 1 — "Increment first, then decrement — net change = adjacency effect"**

Every new land cell is initially a new island (+1). Then for each adjacent land cell in a different component, a merge happens (-1). The final count correctly reflects: net islands = previous + 1 - merges.

For a cell surrounded by 3 different islands: +1 - 3 = -2 (4 islands become 1, net change -3, but +1 new cell makes it -2 from previous).

---

**Story Point 2 — "Check roots before union to decide whether to decrement"**

The `if(currentParent != neighborParent)` check is crucial. Without it, you'd decrement `isLandCnt` even when two adjacent cells are ALREADY in the same component (connected via another path). DSU handles this via the same-root check — no merge needed, no decrement.

---

**Story Point 3 — "`visited[]` is separate from DSU `size` — both serve different purposes"**

- `visited[row][col]` = "has this cell been turned into land by an operator?" — used to check duplicate operations and skip water cells during neighbor checks.
- DSU `size` = "how many nodes are in this component?" — used for union by size optimization.

They're not interchangeable. A cell can be visited (land) but in a component of size 1 (isolated island).

---

**Story Point 4 — "DSU initialized for `n×m` nodes upfront — even unvisited cells"**

`DisjointSet ds(n×m)` creates DSU for ALL grid cells. Most start as isolated nodes (water). As cells become land, they're connected via unions. The DSU is "pre-allocated" for efficiency — no dynamic resizing needed.

---

**Story Point 5 — "This is the dynamic version of Number of Islands I"**

Number of Islands I: static grid, one BFS/DFS pass → `O(n×m)`.
Number of Islands II: dynamic additions → DSU handles each addition in `O(α(n×m))`.

The DSU approach is ONLY better for the dynamic version. For the static problem, BFS is simpler and equally fast.

---

## 9. Code

```cpp
class Solution {
private:
    bool isValid(int row, int col, int n, int m) {
        return row >= 0 && row < n && col >= 0 && col < m;
    }

public:
    vector<int> numOfIslands(int n, int m, vector<vector<int>>& operators) {
        vector<vector<int>> visited(n, vector<int>(m, 0));

        DisjointSet ds(n * m);   // one DSU node per grid cell

        vector<int> result;
        int isLandCnt = 0;

        const int dRow[4] = {-1, +1,  0,  0};
        const int dCol[4] = { 0,  0, +1, -1};

        for(const auto& operation : operators) {
            int row = operation[0];
            int col = operation[1];

            // Duplicate operation: cell already land → count unchanged
            if(visited[row][col] == 1) {
                result.push_back(isLandCnt);
                continue;
            }

            // New land cell: starts as its own island
            visited[row][col] = 1;
            isLandCnt++;

            int currentNode = row * m + col;   // 2D → 1D mapping

            // Check all 4 adjacent cells
            for(int direction = 0; direction < 4; direction++) {
                int neighborRow = row + dRow[direction];
                int neighborCol = col + dCol[direction];

                if(!isValid(neighborRow, neighborCol, n, m)) continue;
                if(!visited[neighborRow][neighborCol]) continue;   // water → skip

                int neighborNode = neighborRow * m + neighborCol;

                int currentParent  = ds.findUParent(currentNode);
                int neighborParent = ds.findUParent(neighborNode);

                // Merge only if in different components
                if(currentParent != neighborParent) {
                    ds.unionBySize(currentParent, neighborParent);
                    isLandCnt--;   // two islands merged into one
                }
            }

            result.push_back(isLandCnt);
        }

        return result;
    }
};
```

---

## 10. Complexity Analysis

### Time Complexity — `O(Q × α(n×m))`

Where `Q` = number of operations.

| Step | Cost | Reason |
|---|---|---|
| Initialize DSU | `O(n×m)` | n×m nodes |
| Per operation: neighbor checks | `O(4 × α(n×m))` | 4 directions, each `findUParent`/`unionBySize` = `O(α)` |
| Q operations total | `O(Q × α(n×m))` | |

**Total: `O(n×m + Q × α(n×m))` ≈ `O(n×m + Q)`**

---

### Space Complexity — `O(n×m)`

| Structure | Size | Reason |
|---|---|---|
| `visited[][]` | `O(n×m)` | Grid-sized boolean array |
| DSU `parent[]` + `size[]` | `O(n×m)` | One per grid cell |
| `result` | `O(Q)` | One answer per operation |

**Total: `O(n×m + Q)`**

---

### BFS/DFS vs DSU Comparison

| | BFS/DFS (rerun each time) | DSU (incremental) |
|---|---|---|
| **TC** | `O(Q × n×m)` | `O(Q × α(n×m))` |
| **Handles additions** | ❌ Needs full rerun | ✅ Incremental `O(1)` per addition |
| **Handles deletions** | ❌ | ❌ (DSU doesn't support deletions) |
| **Code complexity** | Simple | Moderate (DSU needed) |
| **For Q=10K, n×m=10^6** | 10^10 ops ❌ | ~10^7 ops ✅ |
