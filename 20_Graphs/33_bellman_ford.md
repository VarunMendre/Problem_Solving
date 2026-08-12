# Bellman-Ford Algorithm

---

## 1. Problem Statement

Given a **weighted directed graph** with `V` vertices and edges (which may have **negative weights**), find the shortest distance from a source node `src` to all other nodes.

If a **negative cycle** is reachable from `src`, return `{-1}` (no valid shortest path exists).

```
V=5, src=0
Edges: 0→1(5), 0→2(4), 1→3(-3), 2→3(2), 3→4(1)

Shortest distances from 0:
0→0 = 0
0→1 = 5
0→2 = 4
0→1→3 = 5+(-3) = 2
0→2→3 = 4+2 = 6 (longer)
0→1→3→4 = 2+1 = 3

dist = [0, 5, 4, 2, 3] ✅
```

---

## 2. Why Use Bellman-Ford?

### When to Choose Bellman-Ford Over Dijkstra

| Situation | Algorithm |
|---|---|
| No negative weights | Dijkstra `O((V+E) log V)` |
| Has negative weights, no negative cycles | **Bellman-Ford** `O(V × E)` |
| Need to detect negative cycles | **Bellman-Ford** (only algorithm that does this) |
| DAG with negative weights | Topo Sort + Relaxation `O(V+E)` |

**Dijkstra fails with negative weights** because its greedy finalization (popping min-distance node = finalized) breaks when negative edges can reduce distances after finalization.

**Bellman-Ford** doesn't finalize early — it repeatedly relaxes ALL edges `V-1` times, ensuring negative edges are handled correctly.

---

## 3. Intuition / Approach

### The Core Idea — Repeated Relaxation

The shortest path between any two nodes in a graph with `V` vertices uses at most `V-1` edges (a simple path visits each node at most once → at most `V-1` edges).

**Bellman-Ford's Strategy:**

```
After 1 iteration:  dist[] contains shortest paths using at most 1 edge
After 2 iterations: dist[] contains shortest paths using at most 2 edges
...
After V-1 iterations: dist[] contains shortest paths using at most V-1 edges
                       = true shortest paths (no negative cycles)
```

Each iteration relaxes ALL edges. Relaxing an edge `u → v` means:

```
if dist[u] + weight(u,v) < dist[v]:
    dist[v] = dist[u] + weight(u,v)
```

---

### Why Exactly `V-1` Iterations?

**Formal reason:** In a graph with `V` nodes, any simple path (no repeated nodes) has at most `V-1` edges. Since shortest paths are always simple paths (in a no-negative-cycle graph), `V-1` relaxation rounds are sufficient.

**Intuitive reason:** Think of "wavefront propagation":
- After iteration 1: correctly computed shortest paths using exactly 1 edge from `src`
- After iteration 2: correctly computed shortest paths using at most 2 edges from `src`
- After iteration `k`: correctly computed shortest paths using at most `k` edges from `src`

```
Example: src=0, path 0→1→2→3→4 (4 edges)

Iteration 1: dist[1] finalized (0→1, 1 edge)
Iteration 2: dist[2] finalized (0→1→2, 2 edges)
Iteration 3: dist[3] finalized (0→1→2→3, 3 edges)
Iteration 4: dist[4] finalized (0→1→2→3→4, 4 edges)

For V=5 nodes, V-1=4 iterations. ✅
```

A path with 5 nodes needs at most 4 edges → 4 iterations → `V-1` iterations.

**If we did fewer than `V-1` iterations**, we might miss paths that require more hops. **If we did more**, it's unnecessary (and in the presence of negative cycles, distances would diverge incorrectly).

---

### Why NOT Using Dijkstra's Priority Queue?

Dijkstra relaxes edges from the node with the currently smallest distance. With negative edges:

```
0 →(2)→ 1 →(-5)→ 2
0 →(4)→ 2

Dijkstra:
Pop (0,0): relax 1 (dist=2), relax 2 (dist=4)
Pop (2,1): dist[2]=2+(-5)=-3 < dist[2]=4 → update

This actually works! But consider:
0 →(10)→ 1
0 →(2)→  2 →(-20)→ 1

Dijkstra:
Pop (0,0): dist[1]=10, dist[2]=2
Pop (2,2): 2+(-20)=-18 → dist[1]=-18 ← needs to re-open node 1!
Pop (10,1): Stale! But -18 update happened... 
            → Node 1 won't be re-processed correctly
```

Dijkstra marks nodes as "finalized" when popped. Negative edges can invalidate this finalization. Bellman-Ford avoids this by relaxing ALL edges every iteration — no premature finalization.

---

### The `dist[u] != 1e8` Check

```cpp
if(dist[u] != 1e8 && dist[u] + wt < dist[v])
```

If `dist[u]` is still "infinity" (`1e8`), node `u` is unreachable from `src`. Relaxing `u → v` with an "infinite" distance makes no sense — `1e8 + wt` would be meaningless. This guard prevents propagating infinity.

---

## 4. Dry Run

```
V=5, src=0
Edges: [0,1,5], [0,2,4], [1,3,-3], [2,3,2], [3,4,1]

dist = [0, 1e8, 1e8, 1e8, 1e8]
       [0,  1,   2,   3,   4]
```

---

**Iteration 1 (relax all edges once):**

```
Edge 0→1 (wt=5): dist[0]+5=5 < 1e8 → dist[1]=5
Edge 0→2 (wt=4): dist[0]+4=4 < 1e8 → dist[2]=4
Edge 1→3 (wt=-3): dist[1]+(-3)=2 < 1e8 → dist[3]=2
Edge 2→3 (wt=2):  dist[2]+2=6 < dist[3]=2? NO
Edge 3→4 (wt=1):  dist[3]+1=3 < 1e8 → dist[4]=3

dist = [0, 5, 4, 2, 3]
```

Already correct after 1 iteration in this case! But we don't know this yet — we must run all `V-1=4` iterations.

---

**Iteration 2 (relax all edges again):**

```
All edges: no improvement found (distances already optimal)
dist = [0, 5, 4, 2, 3]   (unchanged)
```

**Iterations 3 and 4:** No changes. Final `dist = [0, 5, 4, 2, 3]` ✅

---

**Negative Cycle Check:**

```
Run one more relaxation of all edges:
Edge 0→1: 0+5=5, dist[1]=5 → not better
Edge 0→2: 0+4=4, dist[2]=4 → not better
Edge 1→3: 5-3=2, dist[3]=2 → not better
Edge 2→3: 4+2=6, dist[3]=2 → not better
Edge 3→4: 2+1=3, dist[4]=3 → not better

No improvement → no negative cycle ✅
```

---

## 5. Negative Cycle Detection — In Depth

### What is a Negative Cycle?

A cycle in the graph where the **sum of edge weights is negative**.

```
A →(-1)→ B →(-1)→ C →(-1)→ A

Total cycle weight = -1 + (-1) + (-1) = -3 (negative cycle!)
```

Going around this cycle reduces the total path cost by 3 each time. You can go around forever, reducing the cost to `-∞`. No shortest path exists for any node in or reachable from this cycle.

---

### How Bellman-Ford Detects Negative Cycles

After `V-1` iterations, `dist[]` contains the true shortest distances (assuming no negative cycles). If there's a negative cycle, we can still improve `dist[]` — which is the detection mechanism.

**The `V-th` iteration (detection pass):**

```cpp
for(auto& it : edges) {
    int u = it[0], v = it[1], wt = it[2];
    if(dist[u] != 1e8 && dist[u] + wt < dist[v]) {
        return {-1};   // can still improve → negative cycle!
    }
}
```

**Why does this work?**

After `V-1` iterations, if NO negative cycle: distances are optimal → no more improvements possible → `V-th` iteration finds nothing.

If a NEGATIVE CYCLE exists: the cycle can always be "improved" by going around it one more time → the `V-th` iteration WILL find an improvement → we return `-1`.

---

### Concrete Negative Cycle Example

```
V=4, src=0
Edges: 0→1(1), 1→2(-3), 2→3(2), 3→1(-1)

Cycle: 1→2→3→1, weight = -3+2+(-1) = -2 (NEGATIVE CYCLE!)

dist = [0, 1e8, 1e8, 1e8]
```

**Iteration 1:**
```
0→1: dist[1]=1
1→2: dist[2]=1+(-3)=-2
2→3: dist[3]=-2+2=0
3→1: dist[1]=min(1, 0+(-1))=-1 → dist[1]=-1
```

**Iteration 2:**
```
0→1: dist[1]=0+1=1 < -1? NO
1→2: dist[2]=-1+(-3)=-4 < -2 → dist[2]=-4 (GETTING SMALLER!)
2→3: dist[3]=-4+2=-2 < 0 → dist[3]=-2
3→1: dist[1]=-2+(-1)=-3 < -1 → dist[1]=-3 (GETTING SMALLER!)
```

**Iteration 3:**
```
1→2: dist[2]=-3-3=-6 (STILL GETTING SMALLER!)
...
```

Distances keep decreasing — they'll never stabilize. After `V-1=3` iterations, run the detection pass:

```
Detection (V-th pass):
1→2: dist[1]+(-3) = -? < dist[2] → YES, can still improve → return {-1} ✅
```

**Negative cycle detected!**

---

### Visual Intuition for Negative Cycle Detection

```
Normal graph (V-1 iterations sufficient):
Iteration 1: [0, 5, 4, INF, INF]
Iteration 2: [0, 5, 4, 2, INF]
Iteration 3: [0, 5, 4, 2, 3]    ← stabilizes, no more changes

Negative cycle graph (never stabilizes):
Iteration 1: [0, 1, -2, 0, INF]
Iteration 2: [0, -1, -4, -2, INF]   ← keeps decreasing!
Iteration 3: [0, -3, -6, -4, INF]
V-th pass: still improving → negative cycle ✅
```

---

## 6. Code

```cpp
class Solution {
public:
    vector<int> bellmanFord(int V, vector<vector<int>>& edges, int src) {
        // Initialize all distances to "infinity" (1e8 = safe large value)
        vector<int> dist(V, 1e8);
        dist[src] = 0;   // source is 0 distance from itself

        // Phase 1: Relax all edges V-1 times
        // After k iterations, dist[] has shortest paths using ≤ k edges
        for(int i = 0; i < V - 1; i++) {
            for(auto& it : edges) {
                int u  = it[0];
                int v  = it[1];
                int wt = it[2];

                // Only relax if u is reachable (guard against inf + wt)
                if(dist[u] != 1e8 && dist[u] + wt < dist[v]) {
                    dist[v] = dist[u] + wt;
                }
            }
        }

        // Phase 2: Negative cycle detection
        // If any edge can still be relaxed → negative cycle exists
        for(auto& it : edges) {
            int u  = it[0];
            int v  = it[1];
            int wt = it[2];

            if(dist[u] != 1e8 && dist[u] + wt < dist[v]) {
                return {-1};   // negative cycle detected
            }
        }

        return dist;
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(V × E)`

| Step | Cost | Reason |
|---|---|---|
| Initialize `dist` | `O(V)` | V nodes |
| Relaxation loop | `O(V × E)` | `V-1` iterations × `E` edges per iteration |
| Negative cycle detection | `O(E)` | One pass over all edges |

**Total: `O(V × E)`**

> This is significantly slower than Dijkstra `O((V+E) log V)` for sparse graphs. Bellman-Ford is the necessary trade-off for handling negative weights.

---

### Space Complexity — `O(V)`

| Structure | Size |
|---|---|
| `dist[]` | `O(V)` |

**Total: `O(V)`** — very space efficient, only needs the distance array.

---

## 8. Common Questions

---

**Q: Why `1e8` instead of `INT_MAX`?**

`INT_MAX + any_positive_weight` overflows. `1e8` is large enough to represent "infinity" for typical edge weights while safely adding values without overflow. Alternatively, `LLONG_MAX` with `long long` dist array works for very large weights.

---

**Q: Does edge order matter in Bellman-Ford?**

The ORDER of edge relaxation within each iteration doesn't affect correctness (unlike Dijkstra where order determines which node is processed next). All `V-1` iterations together guarantee correctness regardless of edge processing order. However, a lucky ordering might cause convergence in fewer iterations in practice.

---

**Q: Can Bellman-Ford handle both positive and negative edges?**

Yes. It handles any mix of positive, zero, and negative edge weights, as long as there's no negative cycle reachable from the source.

---

**Q: Does Bellman-Ford work on undirected graphs with negative edges?**

**NO.** An undirected edge with negative weight creates a negative cycle of length 2 (go and return → 2 × negative_weight < 0). Bellman-Ford would always detect a negative cycle. Negative weights on undirected edges are generally meaningless/ill-defined for shortest paths.

---

## 9. Dijkstra vs Bellman-Ford — Full Comparison

| Aspect | Dijkstra | Bellman-Ford |
|---|---|---|
| **Negative weights** | ❌ Fails | ✅ Handles correctly |
| **Negative cycles** | ❌ Cannot detect | ✅ Detects |
| **Time Complexity** | `O((V+E) log V)` | `O(V × E)` |
| **Space Complexity** | `O(V + E)` (heap) | `O(V)` |
| **Algorithm type** | Greedy (finalize on pop) | Dynamic programming |
| **Edge processing** | Outgoing from min-dist node | ALL edges in each iteration |
| **DAG optimization** | Not applicable | Use topo sort instead |
| **Best for** | Positive-weight graphs | Negative weights / cycle detection |
