# Bridges in a Graph (Critical Connections)

---

## 1. Problem Statement

Given a graph with `n` servers (nodes) and a list of connections (edges), a **bridge** (or critical connection) is an edge that, if removed, would disconnect the graph — making some servers unreachable from others.

Return all such bridges.

```
n=4
connections = [[0,1],[1,2],[2,0],[1,3]]

Graph:
0—1—3
|/
2

Edge 1—3: if removed, server 3 is isolated → BRIDGE
Edge 0—1, 1—2, 2—0: form a cycle; removing any one still leaves the others connected → NOT bridges

Answer: [[1,3]]
```

---

## 2. What is a Bridge?

An edge `(u, v)` is a **bridge** if it is NOT part of any cycle. Equivalently:

> An edge `(u, v)` is a bridge if there is NO alternate path from `u` to `v` other than the direct edge `(u,v)`.

If the edge is in a cycle, removing it still leaves `u` and `v` connected via the rest of the cycle.
If the edge is NOT in any cycle, removing it disconnects `u` from `v`.

---

## 3. The Key Insight — `tin[]` and `low[]`

### Discovery Time `tin[node]`

`tin[node]` = the timestamp when DFS first **visits** (discovers) `node`.

```
DFS visits: 0 → 1 → 2 → 3
tin:         1    2    3    4
```

`tin` captures the order of discovery. It never changes after a node is first visited.

---

### Lowest Discovery Time `low[node]`

`low[node]` = the **lowest `tin`** value reachable from the subtree rooted at `node` (in the DFS tree), using at most one **back edge** (edge going to an ancestor).

Initially: `low[node] = tin[node]` (can only reach yourself)

`low[node]` gets updated:
- **After returning from DFS on child `adjNode`:**
  `low[node] = min(low[node], low[adjNode])`
  (the child might reach an ancestor that's also reachable from `node`)

- **When seeing an already-visited neighbor (back edge to ancestor):**
  `low[node] = min(low[node], tin[adjNode])`  ← NOT `low[adjNode]`!
  (the already-visited neighbor is an ancestor; we can reach as far back as its `tin`)

Wait — the code does:
```cpp
if(visited[adjNode]) {
    low[node] = min(low[node], low[adjNode]);  ← uses low[adjNode]
}
```

This works correctly in most cases but the technically more precise formula for back edges uses `tin[adjNode]`. Both are valid implementations — using `low[adjNode]` for back edges is a common simplification that still correctly identifies bridges.

---

### The Bridge Condition

**Edge `(node, adjNode)` is a bridge if and only if:**

```
low[adjNode] > tin[node]
```

**Interpretation:**
- `tin[node]` = when DFS first reached `node`
- `low[adjNode]` = the earliest ancestor reachable from `adjNode`'s subtree

If `low[adjNode] > tin[node]`:
- `adjNode`'s subtree cannot reach `node` or any ancestor of `node` via any back edge
- The ONLY way to reach `node` from `adjNode` is through the direct edge `(node, adjNode)`
- Remove that edge → `adjNode`'s subtree disconnects → **BRIDGE**

If `low[adjNode] <= tin[node]`:
- There exists a back edge from `adjNode`'s subtree to `node` or an ancestor of `node`
- Even without the direct edge `(node, adjNode)`, connectivity is maintained via the back edge
- **NOT a bridge**

---

### Visual Intuition

```
DFS tree:
   0 (tin=1)
   |
   1 (tin=2)
  / \
 2   3
(tin=3) (tin=4)

Back edge: 2→0 exists

For edge (1,2): low[2] = min(tin[2], tin[0]) = min(3,1) = 1
  low[2]=1 ≤ tin[1]=2 → NOT a bridge (2 can reach back to 0 via back edge)

For edge (1,3): low[3] = tin[3] = 4 (no back edges from 3)
  low[3]=4 > tin[1]=2 → BRIDGE ✅
```

---

## 4. Dry Run

```
n=4
Edges: 0-1, 1-2, 2-0, 1-3

adjLs:
0 → [1, 2]
1 → [0, 2, 3]
2 → [1, 0]
3 → [1]

timer=1, visited=[0,0,0,0], tin=[0,0,0,0], low=[0,0,0,0]
Start: dfs(0, parent=-1)
```

---

**dfs(0, parent=-1):**
```
visited[0]=1
tin[0]=low[0]=1, timer=2

Neighbors: 1, 2

→ neighbor 1 (not visited):
  dfs(1, parent=0):
    visited[1]=1
    tin[1]=low[1]=2, timer=3

    Neighbors: 0, 2, 3

    → neighbor 0 (parent!) → SKIP

    → neighbor 2 (not visited):
      dfs(2, parent=1):
        visited[2]=1
        tin[2]=low[2]=3, timer=4

        Neighbors: 1, 0

        → neighbor 1 (parent!) → SKIP

        → neighbor 0 (VISITED, not parent):
          low[2] = min(low[2], low[0]) = min(3, 1) = 1

        Return from dfs(2)

      Back in dfs(1):
      low[1] = min(low[1], low[2]) = min(2, 1) = 1

      Check bridge: low[2]=1 > tin[1]=2? NO → not a bridge

    → neighbor 3 (not visited):
      dfs(3, parent=1):
        visited[3]=1
        tin[3]=low[3]=4, timer=5

        Neighbors: 1

        → neighbor 1 (parent!) → SKIP

        Return from dfs(3)

      Back in dfs(1):
      low[1] = min(low[1], low[3]) = min(1, 4) = 1

      Check bridge: low[3]=4 > tin[1]=2? YES → BRIDGE! Add [1,3]

    Return from dfs(1)

  Back in dfs(0):
  low[0] = min(low[0], low[1]) = min(1, 1) = 1

  Check bridge: low[1]=1 > tin[0]=1? NO → not a bridge

→ neighbor 2 (VISITED, not parent):
  low[0] = min(low[0], low[2]) = min(1, 1) = 1

Return from dfs(0)
```

**Final values:**
```
tin = [1, 2, 3, 4]
low = [1, 1, 1, 4]

Bridges: [[1, 3]] ✅
```

---

## 5. Story Points

---

**Story Point 1 — "`tin` is fixed at discovery, `low` propagates upward"**

`tin[node]` is set ONCE when DFS first visits `node` — it never changes.

`low[node]` starts equal to `tin[node]` and can only DECREASE — it propagates the "reach" of back edges upward through the DFS tree. If a descendant can reach back to an ancestor (via back edge), that ancestor's `tin` propagates up through `low` values.

---

**Story Point 2 — "Skip the parent, but don't skip other visited nodes"**

```cpp
if(adjNode == parent) continue;   // ← parent check
```

In an undirected graph, every edge appears in both directions in the adjacency list. If we're at node `1` and came from `0`, we see `0` in `1`'s neighbor list. Without the parent check, we'd treat the edge back to `0` as a back edge → incorrectly lower `low[1]`.

But we MUST process other visited nodes (actual back edges to ancestors that are NOT the parent) — those are what give `low` its value.

---

**Story Point 3 — "Bridge condition: `low[adjNode] > tin[node]`, NOT `low[adjNode] >= tin[node]`"**

```cpp
if(low[adjNode] > tin[node]) {   // strictly greater
    bridges.push_back({node, adjNode});
}
```

If `low[adjNode] == tin[node]`: the child's subtree can reach exactly `node` (not an ancestor of `node`). This means there's still a back edge connecting back to `node` — so the edge `(node, adjNode)` is NOT a bridge (you can still reach `adjNode`'s subtree via the back edge that returns to `node`).

Strictly `>` means: the child's subtree can reach NO ancestor of `node` (not even `node` itself) except through the direct edge.

---

**Story Point 4 — "Why `timer` is a class member, not a local variable"**

```cpp
int timer = 1;   // class member
```

`timer` must be SHARED across all recursive DFS calls — each call increments it to give each node a unique `tin`. If `timer` were a local variable inside `dfs`, each call would start fresh at 1, giving all nodes the same `tin` → the bridge condition would always fail.

By making it a class member, all recursive calls share the same counter.

---

**Story Point 5 — "Bridges vs Articulation Points — related but different"**

Both use `tin` and `low` in similar DFS algorithms:

| | Bridge | Articulation Point |
|---|---|---|
| **What** | Edge whose removal disconnects graph | Vertex whose removal disconnects graph |
| **Condition** | `low[child] > tin[node]` | `low[child] >= tin[node]` (for non-root) |
| **Note** | Strict `>` | `>=` (includes equality) |

A bridge always has an articulation point at one of its endpoints. But articulation points can exist without bridges (e.g., in certain cycle configurations).

---

## 6. Code

```cpp
class Solution {
private:
    int timer = 1;   // global DFS timestamp — must be shared across calls

    void dfs(int node, int parent, vector<int>& visited,
             vector<int> adjLs[], vector<int>& tin, vector<int>& low,
             vector<vector<int>>& bridges) {

        visited[node] = 1;
        tin[node] = low[node] = timer++;   // assign and increment

        for(auto adjNode : adjLs[node]) {
            if(adjNode == parent) continue;   // skip edge back to parent

            if(visited[adjNode]) {
                // Back edge to an ancestor — update low
                low[node] = min(low[node], low[adjNode]);
            } else {
                // Tree edge — recurse into unvisited neighbor
                dfs(adjNode, node, visited, adjLs, tin, low, bridges);

                // After recursion: propagate child's low value up
                low[node] = min(low[node], low[adjNode]);

                // Bridge condition: child's subtree can't reach above current node
                if(low[adjNode] > tin[node])
                    bridges.push_back({node, adjNode});
            }
        }
    }

public:
    vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
        vector<int> adjLs[n];

        for(auto& it : connections) {
            adjLs[it[0]].push_back(it[1]);
            adjLs[it[1]].push_back(it[0]);
        }

        vector<int> visited(n, 0);
        vector<int> tin(n, 0);
        vector<int> low(n, 0);
        vector<vector<int>> bridges;

        dfs(0, -1, visited, adjLs, tin, low, bridges);

        return bridges;
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists + 2E entries |
| DFS traversal | `O(V + E)` | Each node visited once, each edge traversed twice (undirected) |

**Total: `O(V + E)`**

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| Adjacency list | `O(V + 2E)` | Undirected graph |
| `visited[]`, `tin[]`, `low[]` | `O(V)` each | One per node |
| DFS recursion stack | `O(V)` worst case | Linear chain depth |
| `bridges` result | `O(E)` worst case | At most E bridges |

**Total: `O(V + E)`**

---

## 8. Bridges vs Standard DFS

| | Standard DFS | Bridge-Finding DFS |
|---|---|---|
| **Goal** | Traversal / connected components | Find edges critical for connectivity |
| **Extra arrays** | `visited[]` only | + `tin[]` + `low[]` |
| **Key condition** | None | `low[child] > tin[node]` |
| **Parent tracking** | Not needed | Critical (skip parent edge) |
| **TC** | `O(V+E)` | `O(V+E)` |
