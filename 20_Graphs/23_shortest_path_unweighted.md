# Shortest Path in Undirected Unweighted Graph

---

## 1. Problem Statement

Given an **undirected, unweighted graph** with `V` vertices and a list of edges, find the **shortest path** (minimum number of edges) from a source node `src` to a destination node `dest`.

Return the length of the shortest path, or `-1` if no path exists.

```
V = 9
Edges: 0-1, 0-3, 3-4, 4-5, 5-6, 1-2, 2-6, 6-7, 7-8

src = 0, dest = 8

Possible paths from 0 to 8:
0→1→2→6→7→8  (length 4)
0→3→4→5→6→7→8 (length 6)

Shortest: 0→1→2→6→7→8 → return 4
```

---

## 2. Intuition / Approach

### Why BFS for Shortest Path?

In an **unweighted graph**, every edge has equal cost (1 step). BFS explores nodes **level by level** — all nodes at distance 1 from source first, then distance 2, then distance 3, and so on.

This level-by-level property guarantees: **the first time BFS reaches any node, it's via the shortest possible path**. No shorter path can exist because all shorter distances were explored in earlier levels.

**DFS cannot give shortest paths** — it follows one path as deep as possible and might find a long path to `dest` before ever trying the short one.

---

### The `dist[]` Array

`dist[node]` = shortest distance found so far from `src` to `node`.

- Initialize `dist[src] = 0` (source is 0 steps from itself)
- Initialize `dist[all others] = INT_MAX` (infinity — not yet reached)

For each neighbor, we only update and enqueue if we found a **shorter path** than what's already recorded:

```cpp
if(dist[node] + 1 < dist[neighbour]) {
    dist[neighbour] = dist[node] + 1;
    q.push(neighbour);
}
```

---

### Early Exit Optimization

```cpp
if(neighbour == dest)
    return dist[neighbour];
```

The moment we reach `dest`, we can immediately return — BFS guarantees this is the shortest path. No need to continue exploring.

---

### Why `dist[]` Instead of Just `vis[]`?

In standard BFS (e.g., connected components), a `vis[]` array is enough — we just need to know if a node was visited.

Here we need to know the **distance** — so `dist[]` replaces `vis[]` and serves double duty:
- `dist[node] == INT_MAX` → unvisited
- `dist[node] < INT_MAX` → visited, with the shortest known distance stored

The condition `dist[node] + 1 < dist[neighbour]` prevents re-enqueuing a node that already has a shorter (or equal) recorded distance — same as the "visited" check.

---

### The Early Exit — `src == dest`

```cpp
if(src == dest) return 0;
```

Trivial case: source IS the destination → distance = 0, handled before building the graph.

---

## 3. Dry Run

```
V = 6
Edges: 0-1, 0-2, 1-3, 2-4, 3-5, 4-5

adjLs:
0 → [1, 2]
1 → [0, 3]
2 → [0, 4]
3 → [1, 5]
4 → [2, 5]
5 → [3, 4]

src = 0, dest = 5
dist = [0, INF, INF, INF, INF, INF]
queue = [0]
```

---

**Pop 0 (dist=0):**
```
Neighbors: 1, 2

neighbor 1: dist[0]+1=1 < INF → dist[1]=1, push 1
neighbor 2: dist[0]+1=1 < INF → dist[2]=1, push 2

dist  = [0, 1, 1, INF, INF, INF]
queue = [1, 2]
```

**Pop 1 (dist=1):**
```
Neighbors: 0, 3

neighbor 0: dist[1]+1=2 < dist[0]=0? NO → skip
neighbor 3: dist[1]+1=2 < INF → dist[3]=2, push 3

dist  = [0, 1, 1, 2, INF, INF]
queue = [2, 3]
```

**Pop 2 (dist=1):**
```
Neighbors: 0, 4

neighbor 0: 2 < 0? NO → skip
neighbor 4: dist[2]+1=2 < INF → dist[4]=2, push 4

dist  = [0, 1, 1, 2, 2, INF]
queue = [3, 4]
```

**Pop 3 (dist=2):**
```
Neighbors: 1, 5

neighbor 1: 3 < 1? NO → skip
neighbor 5: dist[3]+1=3 < INF → dist[5]=3
            neighbour == dest(5) → return 3 ✅
```

**Shortest path: 0 → 1 → 3 → 5 = 3 steps ✅**

(Note: 0→2→4→5 also = 3 steps — both are valid shortest paths)

---

## 4. Story Points

---

**Story Point 1 — "BFS level = distance in unweighted graph"**

BFS processes nodes in exact order of their distance from the source. Level 0 = source itself. Level 1 = all nodes 1 edge away. Level 2 = all nodes 2 edges away. Since all edges cost 1, the BFS level number IS the shortest distance. This is why BFS works for shortest path in unweighted graphs but NOT for weighted graphs (where Dijkstra is needed).

---

**Story Point 2 — "`dist[]` serves as both visited and distance tracker"**

`dist[node] == INT_MAX` → never reached (acts as `vis[node] == 0`).  
`dist[node] < INT_MAX` → reached, with the actual shortest distance stored.

The update condition `dist[node] + 1 < dist[neighbour]` is the visited check embedded in the distance comparison. If a neighbor already has a shorter recorded distance, we skip it — same as "already visited" in standard BFS.

---

**Story Point 3 — "Early exit when dest is found"**

```cpp
if(neighbour == dest)
    return dist[neighbour];
```

BFS guarantees that the FIRST time we reach `dest`, it's via the shortest path. We return immediately without finishing the BFS. This can save significant work for graphs where `dest` is close to `src`.

---

**Story Point 4 — "INT_MAX check at the end handles disconnected graphs"**

```cpp
if(ans == INT_MAX)
    return -1;
```

If `dest` is in a different connected component than `src`, BFS from `src` never reaches it. `dist[dest]` stays `INT_MAX`. The final check catches this and returns `-1` as required.

---

**Story Point 5 — "This is BFS + Dijkstra pattern — Dijkstra generalizes this"**

For unweighted graphs, this BFS = Dijkstra with all edge weights = 1.  
The update condition `dist[node] + 1 < dist[neighbour]` is exactly the **relaxation step** in Dijkstra.  
Replace the queue with a priority queue (min-heap) and the `+ 1` with actual edge weights → you have Dijkstra's algorithm.

This problem is the conceptual stepping stone to understanding Dijkstra.

---

## 5. Code

```cpp
class Solution {
public:
    int shortestPath(int V, vector<vector<int>>& edges, int src, int dest) {
        // Trivial case
        if(src == dest) return 0;

        // Build undirected adjacency list
        vector<vector<int>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back(it[1]);
            adjLs[it[1]].push_back(it[0]);
        }

        // dist[node] = shortest distance from src to node
        // INT_MAX = not yet reached (acts as unvisited)
        vector<int> dist(V, INT_MAX);
        dist[src] = 0;

        queue<int> q;
        q.push(src);

        while(!q.empty()) {
            int node = q.front();
            q.pop();

            for(auto& neighbour : adjLs[node]) {
                // Relaxation: found a shorter path to neighbour
                if(dist[node] + 1 < dist[neighbour]) {
                    dist[neighbour] = dist[node] + 1;

                    // Early exit: reached destination via shortest path
                    if(neighbour == dest)
                        return dist[neighbour];

                    q.push(neighbour);
                }
            }
        }

        // dest unreachable → return -1
        return (dist[dest] == INT_MAX) ? -1 : dist[dest];
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists + 2E entries (undirected) |
| BFS | `O(V + E)` | Each node dequeued at most once; each edge relaxed at most once from each endpoint |

**Total: `O(V + E)`**

> The `dist[node] + 1 < dist[neighbour]` condition ensures each node is enqueued at most once (once dist is set optimally, future attempts always fail the condition). So BFS = `O(V + 2E)` = `O(V + E)`.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + 2E)` | Undirected: each edge stored twice |
| `dist[]` | `O(V)` | One per node |
| BFS queue | `O(V)` | At most V nodes simultaneously |

**Total: `O(V + E)`**

---

### BFS vs Dijkstra — When to Use Which

| | BFS Shortest Path | Dijkstra |
|---|---|---|
| **Graph type** | Unweighted (all edges = 1) | Weighted (varying edge costs) |
| **Data structure** | Queue (FIFO) | Priority queue (min-heap) |
| **TC** | `O(V + E)` | `O((V + E) log V)` |
| **Relaxation step** | `dist[node] + 1` | `dist[node] + weight` |
| **First reach = shortest?** | ✅ Yes | ✅ Yes (with min-heap) |

> BFS is essentially Dijkstra with all weights = 1, using a queue instead of a priority queue. When all edges are equal, a queue is sufficient and faster.
