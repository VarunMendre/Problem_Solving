# Number of Operations to Make Network Connected

---

## 1. Problem Statement

You have `n` computers numbered `0` to `n-1` connected by ethernet cables (`connections[i] = [u, v]` means there's a cable between `u` and `v`).

You can **remove a cable from one pair of directly connected computers** and use it to connect another pair that's not directly connected.

Return the **minimum number of cable moves** to make all computers connected. If it's impossible, return `-1`.

```
n=4
connections = [[0,1],[0,2],[1,2]]

Network:
0—1
|/
2   3 (isolated!)

We have 3 cables for 4 computers.
Cable 0-1 or 1-2 is redundant (0,1,2 are already connected via multiple paths).
Remove one redundant cable → reconnect it to node 3.

Answer: 1 (one move)
```

```
n=6
connections = [[0,1],[0,2],[0,3],[1,2],[1,3]]

5 cables, 6 computers.
Need at least n-1=5 cables to connect all → but 5 cables all go to same cluster.
Nodes 4 and 5 are isolated — need 2 cables to connect them.
But we only have 0 redundant cables (exactly 5 cables, all needed for current cluster).

Answer: -1
```

---

## 2. Intuition / Approach

### The Key Insight — Two Independent Observations

**Observation 1: When is it IMPOSSIBLE?**

To connect `n` computers, you need at least `n-1` cables (minimum spanning tree = n-1 edges). If `connections.size() < n-1`, we don't have enough cables regardless of how we rearrange them → return `-1` immediately.

**Observation 2: How many moves are needed?**

After processing all connections:
- Each **redundant cable** (one that connects two already-connected computers) can be **freed** and reused
- Each **disconnected component** (beyond the first) needs exactly **one cable** to connect it to the main network

```
If we have C components total:
  → Need C-1 cables to connect them all (like edges of a tree)
  → If extraEdges >= C-1 → answer is C-1
  → If extraEdges < C-1 → impossible (already caught by upfront check)
```

---

### The Dual Role of `unionBySize`

The modified `unionBySize` returns a `bool`:
- **`true`** → merge happened → the two nodes were in DIFFERENT components → components count decreases by 1
- **`false`** → no merge → the two nodes were ALREADY in the same component → this cable is **redundant** → `extraEdges++`

```cpp
if(ds.unionBySize(u, v)) {
    componentes--;    // merged two components
} else {
    extraEdges++;     // redundant cable — can be reused
}
```

This single loop simultaneously:
1. Builds the connected components (via DSU unions)
2. Counts redundant (freeable) cables
3. Counts how many components remain

---

### Why `components - 1` Operations?

After processing all cables, if we have `C` components:

```
Component 1: {0, 1, 2}
Component 2: {3}
Component 3: {4, 5}

To merge 3 components into 1:
  Connect comp1 ↔ comp2 (1 cable)
  Connect comp1 ↔ comp3 (1 cable)
  Total: 2 = C-1 = 3-1 cables needed
```

Any `C` components need exactly `C-1` cable moves to fully connect them. This is the same as: a spanning tree of `C` nodes has `C-1` edges.

---

### The Upfront Check

```cpp
if(connections.size() < n - 1)
    return -1;
```

With `n` computers, you need at least `n-1` cables to connect them (even in the best case — a tree). If you have fewer, it's mathematically impossible regardless of arrangement.

This check makes the `-1` return in the final line (`extraEdges < requiredEdges`) technically unreachable — it's kept for safety/clarity.

---

## 3. Dry Run

```
n=6
connections = [[0,1],[0,2],[1,2],[3,4],[4,5]]

Initial:
components = 6
extraEdges = 0
DSU: {0},{1},{2},{3},{4},{5}
```

---

**Process [0,1]:**
```
unionBySize(0,1): different components → merge
  parent[1]=0, size[0]=2
  returns true → components-- = 5

DSU: {0,1},{2},{3},{4},{5}
extraEdges=0, components=5
```

**Process [0,2]:**
```
unionBySize(0,2): different → merge
  parent[2]=0, size[0]=3
  returns true → components-- = 4

DSU: {0,1,2},{3},{4},{5}
extraEdges=0, components=4
```

**Process [1,2]:**
```
findUParent(1): 1→0 → root=0
findUParent(2): 2→0 → root=0
Same component! → no merge
  returns false → extraEdges++ = 1

DSU: {0,1,2},{3},{4},{5}   (unchanged)
extraEdges=1, components=4
```

**Process [3,4]:**
```
unionBySize(3,4): different → merge
  parent[4]=3, size[3]=2
  returns true → components-- = 3

DSU: {0,1,2},{3,4},{5}
extraEdges=1, components=3
```

**Process [4,5]:**
```
findUParent(4): 4→3 → root=3
findUParent(5): root=5
Different → merge
  parent[5]=3, size[3]=3
  returns true → components-- = 2

DSU: {0,1,2},{3,4,5}
extraEdges=1, components=2
```

---

**Final Calculation:**
```
components    = 2
extraEdges    = 1
requiredEdges = components - 1 = 2 - 1 = 1

extraEdges(1) >= requiredEdges(1)? YES
return requiredEdges = 1 ✅
```

**Interpretation:** Move the redundant cable `[1,2]` to connect component `{0,1,2}` with component `{3,4,5}`. One operation. Done.

---

## 4. Story Points

---

**Story Point 1 — "`unionBySize` returning `bool` is the key modification"**

Standard `unionBySize` returns `void`. Here it returns `bool` to tell us whether a merge actually happened. This lets one loop do two things simultaneously: track components AND count redundant edges. Clean and efficient.

---

**Story Point 2 — "Upfront check makes the final `-1` branch unreachable"**

```cpp
if(connections.size() < n - 1) return -1;   // ← catches all impossible cases

// ... process connections ...

return extraEdges >= requiredEdges ? requiredEdges : -1;
// ↑ the -1 here is never reached because upfront check already handled it
```

After the upfront check, we're guaranteed `connections.size() >= n-1`. This means we always have enough cables to build a spanning tree — the question is just how many moves.

The final `-1` is defensive code — logically unreachable but safe to keep.

---

**Story Point 3 — "Components start at `n` and decrease by 1 per successful union"**

Starting with `n` isolated nodes = `n` components. Each successful `unionBySize` merges exactly two components into one → `components--`. Failed unions (redundant edges) don't change the component count.

After processing all edges: `components` = true number of connected components remaining.

---

**Story Point 4 — "This is a graph problem disguised as a networking problem"**

Translated to graph terms:
- Computers = nodes
- Cables = edges
- Redundant cables = extra edges (edges that create cycles)
- Disconnected components = groups that need to be bridged
- Answer = number of bridge operations needed

Any "connectivity" or "network" problem with this structure maps to: count components, count extra edges, check feasibility.

---

**Story Point 5 — "C components need exactly C-1 cables — same as MST"**

This is the spanning tree property: a tree on `C` nodes has `C-1` edges. To connect `C` components into one, you build a "meta-tree" where each component is a node and each cable move is an edge. This meta-tree needs `C-1` edges.

---

## 5. Code

```cpp
class DisjointSet {
    vector<int> parent, size;

public:
    DisjointSet(int n) {
        parent.resize(n, 0);
        size.resize(n, 1);
        for(int i = 0; i < n; i++)
            parent[i] = i;   // 0-indexed: each node is its own parent
    }

    int findUParent(int node) {
        if(node == parent[node]) return node;
        return parent[node] = findUParent(parent[node]);   // path compression
    }

    // Returns true if merge happened, false if already same component
    bool unionBySize(int u, int v) {
        int rootU = findUParent(u);
        int rootV = findUParent(v);

        if(rootU == rootV) return false;   // same component → redundant edge

        if(size[rootU] < size[rootV]) {
            parent[rootU] = rootV;
            size[rootV] += size[rootU];
        } else {
            parent[rootV] = rootU;
            size[rootU] += size[rootV];
        }

        return true;   // successful merge
    }
};

class Solution {
public:
    int makeConnected(int n, vector<vector<int>>& connections) {
        // Need at least n-1 cables to connect n computers
        if(connections.size() < n - 1)
            return -1;

        DisjointSet ds(n);

        int extraEdges  = 0;   // redundant cables that can be reused
        int components  = n;   // start: n isolated components

        for(auto& it : connections) {
            int u = it[0], v = it[1];

            if(ds.unionBySize(u, v)) {
                components--;    // merged two components
            } else {
                extraEdges++;    // redundant cable — can be freed
            }
        }

        int requiredEdges = components - 1;   // cables needed to bridge all components

        return extraEdges >= requiredEdges ? requiredEdges : -1;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(E × α(V))` ≈ `O(E)`

| Step | Cost | Reason |
|---|---|---|
| Upfront check | `O(1)` | Simple size comparison |
| Initialize DSU | `O(V)` | V parent/size entries |
| Process all E connections | `O(E × α(V))` | Each connection: one `findUParent` + possible `union`, both `O(α(V))` |

**Total: `O(V + E × α(V))` ≈ `O(V + E)`**

> `α(V)` ≈ constant (≤ 4) for all practical inputs.

---

### Space Complexity — `O(V)`

| Structure | Size | Reason |
|---|---|---|
| DSU `parent[]` | `O(V)` | One per node |
| DSU `size[]` | `O(V)` | One per node |

**Total: `O(V)`**

---

### Compared to BFS/DFS Approach

| | DSU Approach | BFS/DFS Approach |
|---|---|---|
| **Find components** | `O(E × α(V))` | `O(V + E)` |
| **Count redundant edges** | `O(1)` per edge | Needs extra tracking |
| **Code complexity** | Simple | Moderate |
| **Handles dynamic additions** | ✅ Naturally | ❌ Needs full rerun |

Both are `O(V + E)` in practice. DSU is preferred for its extensibility to dynamic graphs.
