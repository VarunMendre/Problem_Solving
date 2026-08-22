# Making a Large Island

---

## 1. Problem Statement

You are given an `n × n` binary grid. You can change **at most one `0` to `1`**. Return the size of the **largest island** possible after this change.

An island is a group of `1`s connected 4-directionally.

```
Grid:
1 0
0 1

Flip (0,1): 
1 1
0 1
→ Island of size 3 ✅ (but still 2 islands)

Flip (1,0):
1 0
1 1
→ Island of size 3 ✅

Answer: 3
```

```
Grid:
1 1
1 0

Flip (1,1):
1 1
1 1
→ Island of size 4 ✅

Answer: 4
```

---

## 2. Intuition / Approach

### Brute Force — Why We Don't Do It

For each `0` cell, flip it to `1` and run BFS/DFS to find the resulting island size.

```
O(n²) zeros × O(n²) BFS per zero = O(n⁴)

For n=500: 500⁴ = 62.5 billion operations → way too slow
```

---

### The DSU Approach — Two-Pass Strategy

**Key insight:** When we flip a `0` to `1`, the new island's size =

```
1 (the flipped cell itself)
+ size of all DISTINCT adjacent components
```

"Distinct" is critical — if two adjacent `1` neighbors are already in the same component, we count that component only once.

**Two passes:**

**Pass 1 — Build all existing land components:**
Scan the grid. For each `1` cell, union it with all adjacent `1` cells. After this pass, DSU contains all current connected components and their sizes.

**Pass 2 — Try flipping each `0`:**
For each `0` cell, look at its 4 neighbors. Collect the **set of unique component roots** among those neighbors. Sum up their sizes + 1 (for the flipped cell itself). Track the maximum.

**Step 3 — Handle all-1s grid:**
If the grid has no `0`s, Pass 2 never executes. Step 3 scans all DSU nodes and takes the maximum component size — returning the whole grid's island size.

---

### Why `set<int> uniqueComponents`?

```cpp
set<int> uniqueComponents;
// ... for each land neighbor:
uniqueComponents.insert(ds.findUParent(neighborNode));
```

Consider this case:
```
. 1 .
1 0 1
. 1 .
```
Flipping center `0`: all 4 neighbors might be in the SAME component (all connected via a ring). Without deduplication, we'd sum the same component's size 4 times → wrong.

The `set` stores only unique roots → each component counted exactly once.

---

### Node Mapping: 2D → 1D

Same as before: `nodeIndex = row × n + col`

```
n=3 grid:
(0,0)=0  (0,1)=1  (0,2)=2
(1,0)=3  (1,1)=4  (1,2)=5
(2,0)=6  (2,1)=7  (2,2)=8
```

---

### Why Access `ds.size[parent]` Directly (Not Via a Method)?

```cpp
for(int parent : uniqueComponents) {
    islandSize += ds.size[parent];
}
```

`parent` is already the root (returned by `findUParent`). The `size` of the root = size of the entire component. Accessing `ds.size[root]` directly is `O(1)` — no additional find needed.

This is why `parent` and `size` are `public` in this DSU implementation.

---

## 3. Dry Run

```
Grid (3×3):
1 1 0
0 0 1
1 0 1

n=3, DSU: 9 nodes (0..8)
```

**Node mapping:**
```
(0,0)=0  (0,1)=1  (0,2)=2
(1,0)=3  (1,1)=4  (1,2)=5
(2,0)=6  (2,1)=7  (2,2)=8
```

---

### Pass 1 — Build Components

Scan all `1` cells and union adjacent `1`s:

```
(0,0)=1: check neighbors
  right (0,1)=1 → union(0,1): parent[1]=0, size[0]=2

(0,1)=1: check neighbors
  left (0,0) already in same component
  down (1,1)=0 → skip
  right (0,2)=0 → skip

(0,2)=0 → skip

(1,0)=0 → skip
(1,1)=0 → skip

(1,2)=1: check neighbors
  up (0,2)=0 → skip
  down (2,2)=1 → union(5,8): parent[8]=5, size[5]=2
  left (1,1)=0 → skip

(2,0)=1: check neighbors
  up (1,0)=0 → skip
  right (2,1)=0 → skip

(2,1)=0 → skip

(2,2)=1: check neighbors
  up (1,2)=1 → findUParent(5)=5, findUParent(8)=5 → same → no new union
  left (2,1)=0 → skip

DSU components:
  {0,1} → root=0, size=2  (top-left island)
  {5,8} → root=5, size=2  (right island)
  {6}   → root=6, size=1  (isolated bottom-left)

parent = [0,0,2,3,4,5,6,7,5]
size   = [2,1,1,1,1,2,1,1,1]
```

---

### Pass 2 — Try Each `0` Cell

**Cell (0,2):**
```
neighbors:
  left (0,1)=1 → root=findUParent(1)=0
  down (1,2)=1 → root=findUParent(5)=5
  up, right: invalid

uniqueComponents = {0, 5}
islandSize = 1 + size[0] + size[5] = 1+2+2 = 5
largestIsland = 5
```

**Cell (1,0):**
```
neighbors:
  up   (0,0)=1 → root=findUParent(0)=0
  down (2,0)=1 → root=findUParent(6)=6
  right (1,1)=0 → skip
  left: invalid

uniqueComponents = {0, 6}
islandSize = 1+2+1 = 4
largestIsland stays 5
```

**Cell (1,1):**
```
neighbors:
  up   (0,1)=1 → root=0
  down (2,1)=0 → skip
  left (1,0)=0 → skip
  right (1,2)=1 → root=5

uniqueComponents = {0, 5}
islandSize = 1+2+2 = 5
largestIsland stays 5
```

**Cell (2,1):**
```
neighbors:
  left (2,0)=1 → root=6
  right (2,2)=1 → root=findUParent(8)=5
  up (1,1)=0 → skip
  down: invalid

uniqueComponents = {5, 6}
islandSize = 1+2+1 = 4
largestIsland stays 5
```

---

### Step 3 — Handle All-1s (here grid has 0s, so this is a safety check)

```
Max ds.size[findUParent(node)] across all nodes:
  node 0: root=0, size=2
  node 5: root=5, size=2
  node 6: root=6, size=1
  (others are non-roots, their size at root is 2 or 1)

largestIsland stays 5 (already larger from pass 2)
```

**Answer: 5** — flip `(0,2)` or `(1,1)` to get an island of size 5.

```
Flip (0,2):
1 1 1
0 0 1
1 0 1
→ {(0,0),(0,1),(0,2),(1,2),(2,2)} = 5 cells ✅
```

---

## 4. Story Points

---

**Story Point 1 — "The `set` for unique components is the crux of correctness"**

Without deduplication:
```
. 1 .
1 0 1   ← flip center
. 1 .

If all 4 neighbors are in the same component (size 4):
Without set: 1 + 4+4+4+4 = 17 ← WRONG
With set:    1 + 4       = 5  ← CORRECT
```

The `set<int>` ensures each component is counted exactly once, regardless of how many neighbors border the flipped cell.

---

**Story Point 2 — "Step 3 handles the all-1s edge case"**

If the grid is entirely `1`, there are no `0` cells. Pass 2 is never entered. Without Step 3, `largestIsland` would remain `0` — completely wrong.

Step 3 finds the largest existing component directly from DSU, covering this case.

```cpp
for(int node = 0; node < n*n; node++) {
    largestIsland = max(largestIsland, ds.size[ds.findUParent(node)]);
}
```

---

**Story Point 3 — "Accessing `ds.size[parent]` instead of `ds.size[node]`"**

DSU only guarantees that `size[root]` is accurate — the root's size reflects the full component. Non-root nodes may have stale size values.

Since `parent` (in the loop `for(int parent : uniqueComponents)`) is already the root, `ds.size[parent]` is the correct component size.

---

**Story Point 4 — "Pass 1 unions ALL adjacent land pairs — not just new ones"**

In Pass 1, scanning left-to-right, top-to-bottom, we union every `1` with its `1` neighbors. Some pairs will have the same root after earlier unions — `unionBySize` handles this with the `if(rootU == rootV) return` check. No special handling needed.

---

**Story Point 5 — "Size of island after flip = 1 + Σ(sizes of unique adjacent components)"**

This formula is the entire algorithm's foundation:
- `1` = the flipped cell itself
- Each unique adjacent component contributes its full size (all cells in that component get connected to the new land)
- Uniqueness via `set` prevents double-counting

No BFS needed — DSU gives component sizes in `O(1)`.

---

## 5. Code

```cpp
class Solution {
private:
    bool isValid(int row, int col, int n) const {
        return row >= 0 && row < n && col >= 0 && col < n;
    }

public:
    int largestIsland(vector<vector<int>>& grid) {
        int n = grid.size();
        DisjointSet ds(n * n);

        const int dRow[4] = { 1,-1, 0, 0};
        const int dCol[4] = { 0, 0,-1, 1};

        // Pass 1: Build connected components from existing land
        for(int row = 0; row < n; row++) {
            for(int col = 0; col < n; col++) {
                if(grid[row][col] == 0) continue;

                int currentNode = row * n + col;

                for(int d = 0; d < 4; d++) {
                    int nRow = row + dRow[d];
                    int nCol = col + dCol[d];

                    if(!isValid(nRow, nCol, n)) continue;
                    if(grid[nRow][nCol] == 0) continue;

                    ds.unionBySize(currentNode, nRow * n + nCol);
                }
            }
        }

        int largestIsland = 0;

        // Pass 2: Try flipping each 0 and compute resulting island size
        for(int row = 0; row < n; row++) {
            for(int col = 0; col < n; col++) {
                if(grid[row][col] == 1) continue;   // only flip 0s

                set<int> uniqueComponents;   // collect distinct adjacent roots

                for(int d = 0; d < 4; d++) {
                    int nRow = row + dRow[d];
                    int nCol = col + dCol[d];

                    if(!isValid(nRow, nCol, n)) continue;
                    if(grid[nRow][nCol] == 0) continue;   // water neighbor → skip

                    uniqueComponents.insert(ds.findUParent(nRow * n + nCol));
                }

                int islandSize = 1;   // the flipped cell itself
                for(int root : uniqueComponents)
                    islandSize += ds.size[root];   // add full component size

                largestIsland = max(largestIsland, islandSize);
            }
        }

        // Step 3: Handle all-1s grid (no 0 was flipped in pass 2)
        for(int node = 0; node < n * n; node++)
            largestIsland = max(largestIsland, ds.size[ds.findUParent(node)]);

        return largestIsland;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(n² × α(n²))`

| Step | Cost | Reason |
|---|---|---|
| Pass 1: Build components | `O(n² × α(n²))` | n² cells × 4 directions × `O(α)` union per pair |
| Pass 2: Try each 0 | `O(n² × α(n²))` | n² cells × 4 `findUParent` calls + set ops `O(log 4)` = `O(1)` |
| Step 3: All-1s check | `O(n² × α(n²))` | n² `findUParent` calls |

**Total: `O(n² × α(n²))` ≈ `O(n²)`**

---

### Space Complexity — `O(n²)`

| Structure | Size | Reason |
|---|---|---|
| DSU `parent[]` + `size[]` | `O(n²)` | One per grid cell |
| `uniqueComponents` set | `O(4)` = `O(1)` | At most 4 neighbors |

**Total: `O(n²)`**

---

### Brute Force vs DSU

| | Brute Force | DSU Approach |
|---|---|---|
| **Algorithm** | Flip each 0, run BFS | Precompute components, simulate flip |
| **TC** | `O(n⁴)` | `O(n²)` |
| **For n=500** | 62.5 billion ops ❌ | ~250K ops ✅ |
| **Space** | `O(n²)` | `O(n²)` |
