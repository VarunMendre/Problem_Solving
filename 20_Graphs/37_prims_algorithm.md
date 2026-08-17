# Prim's Algorithm — Minimum Spanning Tree

---

## 1. Problem Statement

Given a **weighted undirected connected graph** with `V` vertices and a list of edges, find the **weight of the Minimum Spanning Tree (MST)**.

A **Spanning Tree** of a graph:
- Contains all `V` vertices
- Is connected (every vertex reachable from every other)
- Has exactly `V-1` edges (minimum to keep it connected)
- Has NO cycles

A **Minimum Spanning Tree** is the spanning tree with the **smallest total edge weight**.

```
V=5
Edges: 0-1(2), 0-3(6), 1-2(3), 1-3(8), 1-4(5), 2-4(7), 3-4(9)

MST edges: 0-1(2), 1-2(3), 1-4(5), 0-3(6)
MST weight = 2+3+5+6 = 16
```

---

## 2. What is a Minimum Spanning Tree?

### Why "Tree"?

A spanning tree of `V` nodes has exactly `V-1` edges with no cycles. Add one more edge → creates a cycle. Remove one edge → disconnects the tree.

### Why "Minimum"?

Among all possible spanning trees of the graph, the MST has the **smallest sum of edge weights**. Multiple MSTs can exist if some edge weights are equal.

### Real-World Applications

| Application | MST Meaning |
|---|---|
| Network cables | Minimum wire to connect all computers |
| Road construction | Minimum road cost to connect all cities |
| Power grids | Minimum cable to supply all houses |
| Cluster analysis | Find natural groupings in data |

---

## 3. Intuition — Prim's Algorithm

### The Greedy Strategy

Prim's builds the MST **one vertex at a time**, always adding the cheapest edge that connects a new vertex to the growing MST.

**Analogy:** You're building a network incrementally. Start with one office. Expand by always connecting the next closest office (cheapest cable from any already-connected office). Never revisit an already-connected office.

---

### The Algorithm Step by Step

```
1. Start from any node (we use node 0)
2. Push {weight=0, node=0} into min-heap
3. While heap is not empty:
     Pop the minimum weight {wt, node}
     If node is already in MST → skip (avoid cycles)
     Else → add node to MST, add wt to total
     Push all unvisited neighbors with their edge weights
4. Return total weight
```

The min-heap always gives us the cheapest edge connecting an unvisited node to the MST.

---

### Why Min-Heap?

We always want the **cheapest available edge** to the next unvisited node. Min-heap gives the smallest element in `O(log E)`.

**Why NOT a regular queue?** A regular queue processes edges in insertion order — not by weight. We'd pick arbitrary (likely suboptimal) edges. Greedy minimum selection requires the heap.

---

### Why `visited[]` Instead of `dist[]`?

Unlike Dijkstra (which minimizes path distance from source), Prim's minimizes **individual edge weights** added to the MST. A node is "in the MST" or "not in the MST" — binary, not a distance.

`visited[node] = 1` means "node already incorporated into MST — any edge to it would create a cycle."

The check:
```cpp
if(vist[node]) continue;   // already in MST, skip
```

This prevents cycles — if we try to add a node already in the MST via another edge, we skip it.

---

### The Key Timing — Add to MST When POPPED, Not Pushed

```cpp
auto [wt, node] = pq.top();
pq.pop();

if(vist[node]) continue;

vist[node] = 1;   // ← added to MST HERE (on pop)
sum += wt;        // ← edge weight added HERE
```

When multiple edges to the same unvisited node exist in the heap, the min-heap pops the cheapest one FIRST. The `continue` check discards all later (more expensive) edges to that same node.

**This is the greedy selection:** for each node, only the cheapest edge to it gets used.

---

## 4. Dry Run

```
V=5
Edges: 0-1(2), 0-3(6), 1-2(3), 1-3(8), 1-4(5), 2-4(7), 3-4(9)

adjLs:
0 → [(1,2), (3,6)]
1 → [(0,2), (2,3), (3,8), (4,5)]
2 → [(1,3), (4,7)]
3 → [(0,6), (1,8), (4,9)]
4 → [(1,5), (2,7), (3,9)]

vist = [0,0,0,0,0], sum=0
pq   = [{0,0}]
```

---

**Pop {0, node=0}:**
```
vist[0]=0 → add to MST
vist[0]=1, sum += 0 = 0

Push unvisited neighbors:
  (1, wt=2): push {2,1}
  (3, wt=6): push {6,3}

pq   = [{2,1}, {6,3}]
vist = [1,0,0,0,0]
```

**Pop {2, node=1}:**
```
vist[1]=0 → add to MST
vist[1]=1, sum += 2 = 2  ← edge 0-1 added to MST

Push unvisited neighbors:
  (0): vist[0]=1 → skip
  (2, wt=3): push {3,2}
  (3, wt=8): push {8,3}
  (4, wt=5): push {5,4}

pq   = [{3,2}, {5,4}, {6,3}, {8,3}]
vist = [1,1,0,0,0]
```

**Pop {3, node=2}:**
```
vist[2]=0 → add to MST
vist[2]=1, sum += 3 = 5  ← edge 1-2 added to MST

Push unvisited neighbors:
  (1): vist[1]=1 → skip
  (4, wt=7): push {7,4}

pq   = [{5,4}, {6,3}, {7,4}, {8,3}]
vist = [1,1,1,0,0]
```

**Pop {5, node=4}:**
```
vist[4]=0 → add to MST
vist[4]=1, sum += 5 = 10  ← edge 1-4 added to MST

Push unvisited neighbors:
  (1): vist[1]=1 → skip
  (2): vist[2]=1 → skip
  (3, wt=9): push {9,3}

pq   = [{6,3}, {7,4}, {8,3}, {9,3}]
vist = [1,1,1,0,1]
```

**Pop {6, node=3}:**
```
vist[3]=0 → add to MST
vist[3]=1, sum += 6 = 16  ← edge 0-3 added to MST

Push unvisited neighbors:
  (0): vist[0]=1 → skip
  (1): vist[1]=1 → skip
  (4): vist[4]=1 → skip

pq   = [{7,4}, {8,3}, {9,3}]
vist = [1,1,1,1,1]
```

**Pop {7, node=4}:**
```
vist[4]=1 → SKIP (already in MST)
```

**Pop {8, node=3}:**
```
vist[3]=1 → SKIP
```

**Pop {9, node=3}:**
```
vist[3]=1 → SKIP
```

**pq empty → return sum = 16 ✅**

**MST edges selected:** 0-1(2), 1-2(3), 1-4(5), 0-3(6) = 16

---

## 5. Story Points

---

**Story Point 1 — "Add to MST on POP, not on PUSH — the greedy moment"**

When a node has multiple paths from the MST, ALL of them get pushed to the heap (with different weights). The min-heap pops the cheapest one first → that node is added to MST via the cheapest edge. All subsequent pops of the same node fail the `vist[node]` check → discarded.

This is the **greedy selection**: among all edges connecting unvisited nodes to the MST, pick the minimum.

---

**Story Point 2 — "Prim's is Dijkstra with different semantics"**

| | Dijkstra | Prim's |
|---|---|---|
| **Heap stores** | `{dist_from_src, node}` | `{edge_weight, node}` |
| **What we minimize** | Total path distance | Individual edge weight |
| **`sum` accumulates** | N/A (returns dist array) | Sum of MST edge weights |
| **`visited` meaning** | Not needed (stale check) | In MST or not |
| **Goal** | Shortest path from source | Minimum weight tree |

The code structure is nearly identical — only the semantics differ.

---

**Story Point 3 — "Starting node doesn't matter — MST is unique (for distinct weights)"**

We start with `pq.push({0, 0})` — starting from node `0` with weight `0`. The `0` weight means "no cost to start". You could start from any node and get the same MST total weight. The MST is a property of the graph, not the starting node.

---

**Story Point 4 — "Stale entries in heap — same pattern as Dijkstra"**

When node `4` is first pushed with weight `5` (from node `1`), then later with weight `7` (from node `2`), BOTH entries sit in the heap. The first pop (`{5,4}`) adds it to MST. The second pop (`{7,4}`) hits `vist[4]=1` → discarded.

This is the same stale-entry pattern from Dijkstra — no need to remove old entries, just check on pop.

---

**Story Point 5 — "For dense graphs, Prim's with adjacency matrix + array (no heap) is O(V²)"**

The heap-based Prim's is `O(E log E)` — good for sparse graphs. For dense graphs (`E ≈ V²`), a simple array scan to find the minimum unvisited node costs `O(V)` per step × `V` steps = `O(V²)` — faster than `O(V² log V)` with a heap. This is the "naive Prim's" without a heap.

---

## 6. Code

```cpp
class Solution {
public:
    using PII = pair<int, int>;

    int spanningTree(int V, vector<vector<int>>& edges) {
        // Build undirected weighted adjacency list
        vector<vector<PII>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back({it[1], it[2]});
            adjLs[it[1]].push_back({it[0], it[2]});
        }

        // Min-heap: {edge_weight, node}
        priority_queue<PII, vector<PII>, greater<PII>> pq;
        vector<int> vist(V, 0);

        pq.push({0, 0});   // start from node 0 with 0 cost
        int sum = 0;

        while(!pq.empty()) {
            auto [wt, node] = pq.top();
            pq.pop();

            // Skip if already in MST (stale or redundant entry)
            if(vist[node]) continue;

            // Add this node and its connecting edge to MST
            vist[node] = 1;
            sum += wt;

            // Push all unvisited neighbors into heap
            for(auto& [adjNode, adjWt] : adjLs[node]) {
                if(!vist[adjNode])
                    pq.push({adjWt, adjNode});
            }
        }

        return sum;
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(E log E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists + 2E entries |
| While loop (E iterations) | `O(E)` | Each edge can be pushed once |
| Each pop + log | `O(log E)` | Heap of size up to E |
| Each push into heap | `O(log E)` | Heap size up to E |

**Total: `O(E log E)`**

> Each undirected edge can be pushed at most twice (once from each endpoint) → heap size ≤ 2E → `log(2E)` = `O(log E)`.
> Since `E ≤ V²`, `log E ≤ 2 log V` → `O(E log E)` = `O(E log V)` asymptotically.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + 2E)` | Adjacency list |
| `vist[]` | `O(V)` | One per vertex |
| Priority queue | `O(E)` | At most 2E entries |

**Total: `O(V + E)`**

---

## 8. Prim's vs Kruskal's

Both find the MST — different approaches:

| | Prim's | Kruskal's |
|---|---|---|
| **Approach** | Grow MST one vertex at a time | Sort edges, add if no cycle |
| **Data structure** | Min-heap + visited array | Union-Find (DSU) |
| **Starting point** | Single source vertex | All edges globally sorted |
| **TC (heap-based)** | `O(E log E)` | `O(E log E)` (sort dominates) |
| **TC (dense graph)** | `O(V²)` with array | `O(V² log V)` |
| **Best for** | Dense graphs | Sparse graphs |
| **Edge ordering** | Lazy (push as needed) | Upfront global sort |
| **Cycle detection** | `visited[]` | Union-Find |
