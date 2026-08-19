# Remove Stones to Minimize the Count

---

## 1. Problem Statement

You are given a list of `stones` where `stones[i] = [row, col]` represents a stone on a 2D grid. A stone can be removed if there is **another stone in the same row OR same column**.

Return the **largest number of stones that can be removed**.

```
stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]

Visual grid (● = stone):
     col0 col1 col2
row0:  ●    ●
row1:  ●         ●
row2:       ●    ●

All 6 stones share rows/cols with other stones.
Maximum removable = 5. One stone must always remain.
```

---

## 2. The Core Mathematical Insight

### Thinking in Connected Components

Two stones are "connected" if they share a row OR a column. Transitivity applies — if stone A shares a row with stone B, and stone B shares a column with stone C, then A, B, C are all in the same connected component (you can chain removals).

Now think about what happens within one component:

**From one component with `x` stones:**
- You can always remove `x-1` stones (keep removing a stone that shares a row/col with another, until only 1 remains)
- The last remaining stone has no partner in its row or column — it cannot be removed
- So from a component of `x` stones: **remove exactly `x-1`**

---

### The Mathematical Derivation

Say we have components `C1, C2, ..., Ck` with `x1, x2, ..., xk` stones respectively.

Total removable stones:
```
= (x1 - 1) + (x2 - 1) + ... + (xk - 1)
```

The `-1` per component = the one stone that must stay.

Expand:
```
= (x1 + x2 + ... + xk) - (1 + 1 + ... + 1)
           ↑                      ↑
    = n (total stones)       = k (number of components)

= n - k
```

**Answer = n - (number of connected components)**

> This is elegant — regardless of component sizes, the answer only depends on the total stone count and total component count.

---

### Why Only Components with `size > 1`?

A single isolated stone (size = 1) is its own component. It contributes `x-1 = 1-1 = 0` to the answer — you can't remove it. Including it in `k` would SUBTRACT it from the answer even though it contributes nothing.

Wait — let's reconsider. If we include isolated stones in `k`:

```
n=3, components: {stone A, stone B} (size 2) and {stone C} (size 1)
Correct answer: (2-1) + (1-1) = 1 + 0 = 1

Using formula n - k:
k = 2 (two components)
answer = 3 - 2 = 1 ✅
```

Actually including isolated stones IS correct in the formula — they subtract 1 from k but also contribute 1 fewer to the removable count (0 stones removed from them). So `n - k` works whether we count isolated stones or not... 

But the code counts `cnt = number of components with size > 1` and returns `n - cnt`. This is also correct because:

```
Isolated stone contributes: (1-1) = 0 to removable count
Including it in k would give: n - k (same as formula)
Excluding it from cnt: n - cnt where cnt < k
  → n - cnt > n - k

So excluding isolated stones from the count gives a LARGER answer...
which would be wrong!
```

**Re-examining the code's logic:**

```cpp
for(int i = 0; i < totalNodes; i++) {
    if(ds.parent[i] == i && ds.size[i] > 1)
        cnt++;
}
return n - cnt;
```

`totalNodes` includes both row-nodes AND col-nodes. Many of these nodes represent rows or columns that don't even have stones — they're "phantom" DSU nodes created just for the coordinate system.

The `size > 1` filter specifically excludes:
1. Row/col nodes that have no stones (they're isolated with size=1)
2. Any truly isolated stone (if somehow a stone's row and col node both point only to that stone)

The `size > 1` filter means: **count only DSU roots that represent actual multi-stone components** — i.e., groups where stones were actually connected via shared rows/cols.

**The correct count of "real components" = roots with size > 1** because every real component (group of stones sharing rows/cols) has at least 2 nodes merged into it (the row node + the col node of the first stone at minimum).

---

## 3. The Key Trick — Treating Rows AND Columns as Nodes

### Why a Standard Stone-to-Stone DSU Fails

If we tried to directly union stones with each other:
- Stone `(0,0)` and `(0,1)` share row 0 → union them
- Stone `(0,1)` and `(1,1)` share col 1 → union them
- For `n` stones, checking all pairs = `O(n²)` comparisons

For large inputs (n up to 10,000), this is too slow.

---

### The Row-Column Node Trick

**Insight:** Instead of connecting stones to stones, connect each stone's **row** to its **column**. Two stones in the same row share a row-node → they're in the same component. Two stones in the same column share a col-node → same component.

**Node numbering:**
- Rows `0` to `maxRow` → nodes `0` to `maxRow`
- Columns `0` to `maxCol` → nodes `maxRow+1` to `maxRow+maxCol+1`

```
maxRow = 2, maxCol = 2 → totalNodes = 2+2+2 = 6

Row nodes: 0→node 0, 1→node 1, 2→node 2
Col nodes: 0→node 3, 1→node 4, 2→node 5

Stone at (0,0): union(row 0, col 0) = union(node 0, node 3)
Stone at (0,1): union(row 0, col 1) = union(node 0, node 4)
Stone at (1,0): union(row 1, col 0) = union(node 1, node 3)
Stone at (1,2): union(row 1, col 2) = union(node 1, node 5)
Stone at (2,1): union(row 2, col 1) = union(node 2, node 4)
Stone at (2,2): union(row 2, col 2) = union(node 2, node 5)
```

Now:
- Stone `(0,0)` and `(0,1)` both union with row-node 0 → connected ✅
- Stone `(1,0)` and `(0,0)` both union with col-node 3 → connected ✅
- All 6 stones end up in one component → answer = 6 - 1 = 5 ✅

Each stone requires exactly **ONE union operation** → `O(n)` total, not `O(n²)`.

---

### Why Offset Columns by `maxRow + 1`?

```cpp
int nodeCol = it[1] + maxRow + 1;
```

Row `0` is node `0`. Column `0` must NOT also be node `0` (that would conflate row 0 with column 0 — wrong). By offsetting columns by `maxRow + 1`, row and column node indices are guaranteed to be distinct.

```
If maxRow = 4:
  Row indices:  0, 1, 2, 3, 4        → nodes 0,1,2,3,4
  Col indices:  0+5=5, 1+5=6, 2+5=7  → nodes 5,6,7,...
  
  No overlap! Row 0 ≠ Col 0 in DSU ✅
```

---

## 4. Dry Run

```
stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]
n = 6

maxRow = 2, maxCol = 2
totalNodes = 2+2+2 = 6

Node mapping:
  Row 0 → node 0
  Row 1 → node 1
  Row 2 → node 2
  Col 0 → node 0+2+1 = 3
  Col 1 → node 1+2+1 = 4
  Col 2 → node 2+2+1 = 5

DSU Initial:
  parent = [0,1,2,3,4,5]
  size   = [1,1,1,1,1,1]
```

---

**Stone (0,0): union(node 0, node 3) — row 0 and col 0**
```
Different roots → merge: parent[3]=0, size[0]=2
parent=[0,1,2,0,4,5], size=[2,1,1,1,1,1]
```

**Stone (0,1): union(node 0, node 4) — row 0 and col 1**
```
root(0)=0, root(4)=4 → merge: parent[4]=0, size[0]=3
parent=[0,1,2,0,0,5], size=[3,1,1,1,1,1]
```

**Stone (1,0): union(node 1, node 3) — row 1 and col 0**
```
root(1)=1, root(3)=0 (via parent[3]=0) → size[0]=3>size[1]=1
parent[1]=0, size[0]=4
parent=[0,0,2,0,0,5], size=[4,1,1,1,1,1]
```

**Stone (1,2): union(node 1, node 5) — row 1 and col 2**
```
root(1)=0 (via parent[1]=0), root(5)=5
size[0]=4>size[5]=1 → parent[5]=0, size[0]=5
parent=[0,0,2,0,0,0], size=[5,1,1,1,1,1]
```

**Stone (2,1): union(node 2, node 4) — row 2 and col 1**
```
root(2)=2, root(4)=0 (via parent[4]=0)
size[0]=5>size[2]=1 → parent[2]=0, size[0]=6
parent=[0,0,0,0,0,0], size=[6,1,1,1,1,1]
```

**Stone (2,2): union(node 2, node 5) — row 2 and col 2**
```
root(2)=0, root(5)=0 → SAME root → no merge (already connected)
```

---

**Count valid components:**
```
Check all nodes 0..5 where parent[i]==i AND size[i]>1:
  node 0: parent[0]=0 ✅, size[0]=6>1 ✅ → cnt=1
  node 1: parent[1]=0 ≠ 1 → skip
  node 2: parent[2]=0 ≠ 2 → skip
  node 3: parent[3]=0 ≠ 3 → skip
  node 4: parent[4]=0 ≠ 4 → skip
  node 5: parent[5]=0 ≠ 5 → skip

cnt = 1 (one real component)
```

**Answer: `n - cnt = 6 - 1 = 5` ✅**

5 stones can be removed, 1 must remain.

---

## 5. Story Points

---

**Story Point 1 — "The derivation: (x1-1)+(x2-1)+...= n - k"**

This is the entire algorithm's foundation. Each component of `x` stones contributes `x-1` removals. Summing across all `k` components:
```
Σ(xi - 1) = Σxi - k = n - k
```
The answer collapses to just two numbers: total stones `n` and total components `k`. You don't need to know how big each component is.

---

**Story Point 2 — "Row-col DSU is O(n), stone-stone DSU would be O(n²)"**

Direct stone-to-stone union requires comparing every pair to find shared rows/cols → `O(n²)`. The row-col trick: each stone does exactly one union (its row node with its col node) → `O(n)` unions total. Brilliant reduction.

---

**Story Point 3 — "`size > 1` filters phantom DSU nodes"**

`totalNodes = maxRow + maxCol + 2` creates DSU nodes for ALL rows and columns, not just ones with stones. Empty rows/columns are isolated singleton nodes (size=1). The `size > 1` check ensures we only count actual stone-connected components, not these phantom nodes.

---

**Story Point 4 — "The column offset `maxRow + 1` prevents row-col ID collision"**

Without the offset:
```
Row 0 = DSU node 0
Col 0 = DSU node 0  ← SAME NODE! Wrong!
Stone (0,0) would union(0,0) — self-union → nothing happens
```

The offset `maxRow + 1` shifts all column IDs above all row IDs, ensuring no overlap.

---

**Story Point 5 — "One stone ALWAYS remains per component — guaranteed by the problem's removal rule"**

A stone can only be removed if another stone shares its row OR column. The last stone in a component has no such partner (all rows and columns it touches now only have this one stone). It literally cannot be removed. So each component always loses exactly `size-1` stones, never all of them.

---

## 6. Code

```cpp
class DisjointSet {
public:
    vector<int> parent, size;

    DisjointSet(int n) {
        parent.resize(n, 0);
        size.resize(n, 1);
        for(int i = 0; i < n; i++)
            parent[i] = i;
    }

    int findUParent(int node) {
        if(node == parent[node]) return node;
        return parent[node] = findUParent(parent[node]);   // path compression
    }

    bool unionBySize(int u, int v) {
        int rootU = findUParent(u);
        int rootV = findUParent(v);
        if(rootU == rootV) return false;

        if(size[rootU] < size[rootV]) {
            parent[rootU] = rootV;
            size[rootV] += size[rootU];
        } else {
            parent[rootV] = rootU;
            size[rootU] += size[rootV];
        }
        return true;
    }
};

class Solution {
public:
    int removeStones(vector<vector<int>>& stones) {
        int n = stones.size();

        // Find bounds to determine DSU node count
        int maxRow = 0, maxCol = 0;
        for(auto& it : stones) {
            maxRow = max(maxRow, it[0]);
            maxCol = max(maxCol, it[1]);
        }

        // Total DSU nodes = row nodes (0..maxRow) + col nodes (maxRow+1..maxRow+maxCol+1)
        int totalNodes = maxRow + maxCol + 2;
        DisjointSet ds(totalNodes);

        // For each stone: union its row node with its col node
        for(auto& it : stones) {
            int nodeRow = it[0];
            int nodeCol = it[1] + maxRow + 1;   // offset cols to avoid collision with rows

            ds.unionBySize(nodeRow, nodeCol);
        }

        // Count DSU roots that represent actual stone-connected components
        // (size > 1 filters out phantom row/col nodes with no stones)
        int cnt = 0;
        for(int i = 0; i < totalNodes; i++) {
            if(ds.parent[i] == i && ds.size[i] > 1)
                cnt++;
        }

        // Answer = n - (number of components)
        // Each component of x stones contributes x-1 removals
        // Total = Σ(xi-1) = n - cnt
        return n - cnt;
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(n × α(n))`

| Step | Cost | Reason |
|---|---|---|
| Find maxRow, maxCol | `O(n)` | One pass over n stones |
| Initialize DSU | `O(maxRow + maxCol)` | Initialize `totalNodes` entries |
| Process n stones (n unions) | `O(n × α(n))` | Each `unionBySize` = `O(α(n))` |
| Count valid components | `O(maxRow + maxCol)` | Scan all DSU nodes |

**Total: `O(n × α(n))` ≈ `O(n)`**

---

### Space Complexity — `O(maxRow + maxCol)`

| Structure | Size | Reason |
|---|---|---|
| DSU `parent[]` + `size[]` | `O(maxRow + maxCol)` | `totalNodes` entries each |

**Total: `O(maxRow + maxCol)`**

> For typical constraints (rows/cols up to 10,000), this is `O(20,000)` = `O(1)` effectively constant space.
