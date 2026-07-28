# Shortest Path in a DAG (Weighted)

---

## 1. Problem Statement

Given a **Directed Acyclic Graph (DAG)** with `V` vertices, `E` edges, and edge weights, find the **shortest path from source node `0` to all other nodes**.

Return a `dist[]` array where `dist[i]` = shortest distance from node `0` to node `i`.
If node `i` is unreachable, return `-1` for that node.

```
V=6, edges with weights:
0→1 (2), 0→4 (1)
1→2 (3)
4→2 (2), 4→5 (4)
2→3 (6)
5→3 (1)

Shortest distances from 0:
dist = [0, 2, 3, 6, 1, 5]
          0  1  2  3  4  5
```

---

## 2. Intuition / Approach

### Why Not Dijkstra or BFS?

- **BFS** only works for unweighted graphs (all edges cost 1)
- **Dijkstra** works for weighted graphs but uses a priority queue → `O((V+E) log V)`
- For a **DAG**, we can do better: `O(V + E)` using topological sort

### The Core Insight — DAG Has a Natural Processing Order

In a DAG, topological order gives us a sequence where **every node comes before all nodes it points to**. This is the perfect property for shortest path computation:

> If we process nodes in topological order, whenever we process node `u`, we're guaranteed that **all nodes that could contribute to `u`'s distance have already been processed**.

In other words: by the time we process `u`, `dist[u]` is already finalized — we can safely use it to relax all of `u`'s outgoing edges.

This is why we don't need a priority queue. Topological order gives us the "right" processing order for free.

---

### Why Dijkstra Works but is Suboptimal for DAGs?

Dijkstra uses a min-heap to always pick the node with the currently smallest tentative distance — this ensures that when a node is popped, its distance is finalized.

In a DAG, topological order provides this same guarantee without any heap overhead. Topo order → each node processed once → all its incoming paths already resolved.

---

### The Two-Phase Algorithm

```
Phase 1: Topological Sort (DFS-based)
  Run DFS on all nodes, push to stack in post-order
  Stack gives topo order (top = first in topo order)

Phase 2: Relax Edges in Topo Order
  Initialize dist[0]=0, dist[all others]=INF
  Pop each node from stack in topo order:
    If dist[node] == INF → unreachable → mark -1, skip relaxation
    For each outgoing edge (node → v, weight w):
      If dist[node] + w < dist[v]:
        dist[v] = dist[node] + w   ← relaxation
```

---

### Why `dist[node] == INT_MAX → -1` and Skip?

If `dist[node]` is still `INT_MAX` when we pop it from the stack, this node is **unreachable from source `0`**. We:
1. Set `dist[node] = -1` (required output format)
2. Skip relaxing its neighbors — there's no useful distance to propagate

If we didn't skip, `INT_MAX + weight` would overflow and give nonsensical results.

---

### The Adjacency List Format

```cpp
vector<vector<pair<int,int>>> adjLs(V);
// adjLs[u] = [(v1, w1), (v2, w2), ...]
// directed weighted edge: u → v with weight w
```

Each entry is a `{neighbor, weight}` pair.

---

## 3. Dry Run

```
V=6
Edges: [0,1,2], [0,4,1], [1,2,3], [4,2,2], [4,5,4], [2,3,6], [5,3,1]

adjLs:
0 → [(1,2), (4,1)]
1 → [(2,3)]
2 → [(3,6)]
3 → []
4 → [(2,2), (5,4)]
5 → [(3,1)]
```

---

### Phase 1 — Topological Sort (DFS)

```
vis = [0,0,0,0,0,0], stack = []

i=0: vis[0]=0 → topoSort(0):
  vis[0]=1
  neighbor 1: vis[1]=0 → topoSort(1):
    vis[1]=1
    neighbor 2: vis[2]=0 → topoSort(2):
      vis[2]=1
      neighbor 3: vis[3]=0 → topoSort(3):
        vis[3]=1
        no neighbors
        push 3 → stack=[3]
      push 2 → stack=[3,2]
    push 1 → stack=[3,2,1]
  neighbor 4: vis[4]=0 → topoSort(4):
    vis[4]=1
    neighbor 2: vis[2]=1 → skip
    neighbor 5: vis[5]=0 → topoSort(5):
      vis[5]=1
      neighbor 3: vis[3]=1 → skip
      push 5 → stack=[3,2,1,5]
    push 4 → stack=[3,2,1,5,4]
  push 0 → stack=[3,2,1,5,4,0]

Stack (top→bottom): [0, 4, 5, 1, 2, 3]
Topo order: 0 → 4 → 5 → 1 → 2 → 3
```

---

### Phase 2 — Relax in Topo Order

```
dist = [0, INF, INF, INF, INF, INF]
       [0,  1,   2,   3,   4,   5 ]
```

**Pop 0 (dist=0):**
```
dist[0]=0 ≠ INF → process
  neighbor 1, wt=2: 0+2=2 < INF → dist[1]=2
  neighbor 4, wt=1: 0+1=1 < INF → dist[4]=1

dist = [0, 2, INF, INF, 1, INF]
```

**Pop 4 (dist=1):**
```
  neighbor 2, wt=2: 1+2=3 < INF → dist[2]=3
  neighbor 5, wt=4: 1+4=5 < INF → dist[5]=5

dist = [0, 2, 3, INF, 1, 5]
```

**Pop 5 (dist=5):**
```
  neighbor 3, wt=1: 5+1=6 < INF → dist[3]=6

dist = [0, 2, 3, 6, 1, 5]
```

**Pop 1 (dist=2):**
```
  neighbor 2, wt=3: 2+3=5 < dist[2]=3? NO → skip

dist = [0, 2, 3, 6, 1, 5]
```

**Pop 2 (dist=3):**
```
  neighbor 3, wt=6: 3+6=9 < dist[3]=6? NO → skip

dist = [0, 2, 3, 6, 1, 5]
```

**Pop 3 (dist=6):**
```
  no neighbors → nothing

Final dist = [0, 2, 3, 6, 1, 5] ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Topological order = guaranteed processing order for DAG shortest path"**

In topological order, when you process node `u`, all nodes that have paths LEADING TO `u` have already been processed. So `dist[u]` is already at its minimum value when you process it. No need to re-visit or use a priority queue. This is the fundamental reason this approach is `O(V+E)` instead of Dijkstra's `O((V+E) log V)`.

---

**Story Point 2 — "Skip unreachable nodes — but why not just propagate INF?"**

If `dist[node] == INT_MAX` and we compute `INT_MAX + weight`, we get integer overflow → undefined behavior or a very small negative number → incorrect relaxation.

Setting it to `-1` and using `continue` is both correct (output format) and safe (prevents overflow). Nodes that depend on this unreachable node will also stay `INT_MAX` until they're popped, when they too get set to `-1`.

---

**Story Point 3 — "Relaxation step is the same as Dijkstra"**

```cpp
if(dist[node] + wt < dist[v])
    dist[v] = dist[node] + wt;
```

This is exactly Dijkstra's relaxation. The difference is in HOW we pick the processing order:
- Dijkstra: priority queue picks node with smallest current `dist` → `O(log V)` per pick
- DAG: topo order picks nodes in the right order → `O(1)` per pick

Same relaxation, smarter order exploitation.

---

**Story Point 4 — "This does NOT work for graphs with cycles"**

Topological sort is only defined for DAGs. If the graph had a cycle, nodes in the cycle would have no valid topological order, and some nodes might be processed before their shortest-distance contributor is known. This is why Dijkstra (or Bellman-Ford for negative weights) is needed for general graphs.

---

**Story Point 5 — "Source node is hardcoded as node 0"**

`dist[0] = 0` — the source is always node 0. If you wanted a different source, you'd change this initialization. The topological sort itself is source-independent — it's only Phase 2 (relaxation) that depends on the source.

---

## 5. Code

```cpp
class Solution {
private:
    void topoSort(int node, vector<int>& vis, stack<int>& st,
                  vector<vector<pair<int,int>>>& adjLs) {
        vis[node] = 1;

        for(auto& it : adjLs[node]) {
            int neighbour = it.first;
            if(!vis[neighbour])
                topoSort(neighbour, vis, st, adjLs);
        }

        st.push(node);   // push AFTER exploring all neighbors (post-order)
    }

public:
    vector<int> shortestPath(int V, int E, vector<vector<int>>& edges) {
        // Step 1: Build directed weighted adjacency list
        vector<vector<pair<int,int>>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back({it[1], it[2]});   // {neighbor, weight}
        }

        // Step 2: Topological sort via DFS
        stack<int> st;
        vector<int> vis(V, 0);
        for(int i = 0; i < V; i++) {
            if(!vis[i])
                topoSort(i, vis, st, adjLs);
        }

        // Step 3: Initialize distances
        vector<int> dist(V, INT_MAX);
        dist[0] = 0;   // source = node 0

        // Step 4: Relax edges in topological order
        while(!st.empty()) {
            int node = st.top();
            st.pop();

            if(dist[node] == INT_MAX) {
                // Unreachable node → mark -1, skip to avoid overflow
                dist[node] = -1;
                continue;
            }

            for(auto& it : adjLs[node]) {
                int v  = it.first;
                int wt = it.second;

                // Relaxation: update if shorter path found
                if(dist[node] + wt < dist[v])
                    dist[v] = dist[node] + wt;
            }
        }

        return dist;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists + E directed weighted edges |
| Topological sort (DFS) | `O(V + E)` | Each node and edge visited once |
| Relaxation loop | `O(V + E)` | Each node popped once + each edge relaxed once |

**Total: `O(V + E)`**

> This is optimal — you can't find shortest paths without at least examining every edge once.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + E)` | V lists + E `{neighbor, weight}` pairs |
| `vis[]` | `O(V)` | One per node |
| `stack` | `O(V)` | Holds all V nodes |
| `dist[]` | `O(V)` | One distance per node |
| Recursion stack (topo sort) | `O(V)` worst case | Linear chain depth |

**Total: `O(V + E)`**

---

### Comparison — Shortest Path Algorithms

| Algorithm | Graph Type | TC | SC | Notes |
|---|---|---|---|---|
| **BFS** | Unweighted | `O(V+E)` | `O(V+E)` | All weights = 1 |
| **DAG + Topo Sort** | Weighted DAG | `O(V+E)` | `O(V+E)` | No cycles allowed |
| **Dijkstra** | Weighted, no negative | `O((V+E) log V)` | `O(V+E)` | General weighted graphs |
| **Bellman-Ford** | Weighted, negative OK | `O(V×E)` | `O(V)` | Detects negative cycles |

> For DAGs: always prefer Topo Sort approach — same correctness as Dijkstra but `O(V+E)` vs `O((V+E) log V)`.
