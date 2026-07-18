# Directed Graph Cycle Detection

---

## 1. Problem Statement

Given a **directed graph** with `V` vertices and a list of directed edges, determine whether the graph contains a **cycle**.

In a directed graph, a cycle means there exists a path that starts from some node and, by following directed edges, eventually returns to the same node.

```
Cyclic:                    Acyclic (DAG):
0 → 1 → 2                  0 → 1 → 2
        ↓                          ↓
        3 → 1  ← cycle!            3
```

---

## 2. Why Undirected Cycle Detection Doesn't Work Here

In undirected graphs, we tracked `parent` to avoid false positives (going back on the same edge). That approach fails for directed graphs.

```
Directed graph:
0 → 1
0 → 2
1 → 3
2 → 3

Shared destination (3) but NO cycle — it's a DAG.
```

Here node `3` is reached twice — from `1` and from `2`. An undirected cycle-detection approach would incorrectly flag this as a cycle (visited node found that's not parent).

**In directed graphs, "visited" is not enough — we need to know if the visited node is on the CURRENT DFS PATH.**

---

## 3. The Core Insight — Path Visibility

A cycle in a directed graph exists if and only if during DFS, we reach a node that is **already on the current recursion path** (i.e., we're walking in a circle right now).

This requires distinguishing between two types of "visited":
- **Globally visited** — seen in some DFS call (past or current)
- **Path visited** — currently on the active DFS call stack

Finding a globally-visited node doesn't mean cycle. Finding a path-visited node DOES mean cycle (we're looping back into our own current path).

---

## 4. Approach 1 — Two Arrays (`vis` + `pathVis`)

### The Two Arrays

| Array | Value `0` | Value `1` | Meaning |
|---|---|---|---|
| `vis[node]` | Unvisited | Visited | Has this node EVER been processed? |
| `pathVis[node]` | Not on current path | On current path | Is this node on the ACTIVE DFS stack right now? |

### The Key Line — `pathVis[node] = 0` on Backtrack

```cpp
pathVis[node] = 0;
```

When DFS **finishes** with a node (all its neighbors explored, no cycle found), we remove it from the current path. This is the backtracking step — the node is no longer "active" in the current DFS traversal.

This is crucial: if node `A` is fully explored from one path and then revisited from another path, its `pathVis` is `0` — correctly indicating it's not currently in the active path, so no cycle via this visit.

### The Three Cases

When visiting `neighbor` from `node`:

| Case | Condition | Action |
|---|---|---|
| Unvisited | `!vis[neighbor]` | Recurse into it |
| Visited + on current path | `vis[neighbor] && pathVis[neighbor]` | **CYCLE → return true** |
| Visited + NOT on current path | `vis[neighbor] && !pathVis[neighbor]` | Safely skip (already fully explored, no cycle) |

---

### Dry Run (Approach 1)

```
Graph: 3 nodes
0 → 1 → 2 → 1  (cycle: 1→2→1)

adjLs:
0 → [1]
1 → [2]
2 → [1]

vis = [0,0,0], pathVis = [0,0,0]
```

**Outer loop i=0:**

```
dfs(0, vis, pathVis):
  vis[0]=1, pathVis[0]=1
  neighbor 1: !vis[1] → dfs(1):
    vis[1]=1, pathVis[1]=1
    neighbor 2: !vis[2] → dfs(2):
      vis[2]=1, pathVis[2]=1
      neighbor 1: vis[1]=1 AND pathVis[1]=1 → CYCLE → return true ✅

vis=[1,1,1], pathVis=[1,1,1] at point of detection
```

---

**Dry Run — No Cycle (DAG):**

```
Graph: 0→1, 0→2, 1→3, 2→3

adjLs: 0→[1,2], 1→[3], 2→[3], 3→[]

dfs(0):
  vis[0]=1, pathVis[0]=1
  neighbor 1: dfs(1):
    vis[1]=1, pathVis[1]=1
    neighbor 3: dfs(3):
      vis[3]=1, pathVis[3]=1
      no neighbors
      pathVis[3]=0 ← backtrack
      return false
    pathVis[1]=0 ← backtrack
    return false
  neighbor 2: dfs(2):
    vis[2]=1, pathVis[2]=1
    neighbor 3: vis[3]=1, pathVis[3]=0 → safely skip (not on path)
    pathVis[2]=0
    return false
  pathVis[0]=0
  return false
```

Node `3` visited twice (from `1` and from `2`) but NOT flagged as cycle — because `pathVis[3]=0` when `2` tries to visit it. ✅

### Code (Approach 1)

```cpp
class Solution {
private:
    bool dfs(int node, vector<int>& vis, vector<int>& pathVis,
             vector<vector<int>>& adjLs) {

        vis[node] = 1;        // mark globally visited
        pathVis[node] = 1;    // mark on current path

        for(auto& neighbor : adjLs[node]) {
            if(!vis[neighbor]) {
                // Unvisited → recurse
                if(dfs(neighbor, vis, pathVis, adjLs) == true)
                    return true;
            }
            else if(pathVis[neighbor]) {
                // Visited AND on current path → cycle found
                return true;
            }
            // Visited but NOT on current path → safe to skip
        }

        pathVis[node] = 0;    // backtrack: remove from current path
        return false;
    }

public:
    bool isCyclic(int V, vector<vector<int>>& edges) {
        vector<vector<int>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back(it[1]);   // directed edge only
        }

        vector<int> vis(V, 0);
        vector<int> pathVis(V, 0);

        for(int i = 0; i < V; i++) {
            if(!vis[i]) {
                if(dfs(i, vis, pathVis, adjLs) == true)
                    return true;
            }
        }

        return false;
    }
};
```

---

## 5. Approach 2 — Single Array with 3 States

### The Optimization

Instead of two boolean arrays, use ONE array with three states:

| `vis[node]` | Meaning |
|---|---|
| `0` | Unvisited — never seen |
| `2` | In progress — currently on the active DFS path (was `pathVis=1` in approach 1) |
| `1` | Done — fully explored, removed from path (was `vis=1, pathVis=0` in approach 1) |

```
Approach 1 state  →  Approach 2 state
vis=0, pathVis=0  →  vis=0   (unvisited)
vis=1, pathVis=1  →  vis=2   (on current path)
vis=1, pathVis=0  →  vis=1   (done)
```

This merges both arrays into one — cleaner, uses half the space.

### The Key Lines

```cpp
vis[node] = 2;    // entering node → mark "in progress" (on path)
...
vis[node] = 1;    // finished node → mark "done" (off path)
```

When a node is first entered → `2` (in progress).  
When DFS finishes with it and backtracks → `1` (done, fully safe).

### The Three Cases

| Case | Condition | Action |
|---|---|---|
| Unvisited | `!vis[neighbor]` (== 0) | Recurse |
| In progress | `vis[neighbor] == 2` | **CYCLE → return true** |
| Done | `vis[neighbor] == 1` | Safely skip |

### Code (Approach 2)

```cpp
class Solution {
private:
    bool dfs(int node, vector<int>& vis,
             vector<vector<int>>& adjLs) {

        vis[node] = 2;    // mark in-progress (on current path)

        for(auto& neighbor : adjLs[node]) {
            if(!vis[neighbor]) {
                // Unvisited → recurse
                if(dfs(neighbor, vis, adjLs) == true)
                    return true;
            }
            else if(vis[neighbor] == 2) {
                // Currently in-progress → back edge → cycle
                return true;
            }
            // vis[neighbor] == 1 → done, safely skip
        }

        vis[node] = 1;    // mark done (backtrack: off current path)
        return false;
    }

public:
    bool isCyclic(int V, vector<vector<int>>& edges) {
        vector<vector<int>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back(it[1]);
        }

        vector<int> vis(V, 0);

        for(int i = 0; i < V; i++) {
            if(!vis[i]) {
                if(dfs(i, vis, adjLs) == true)
                    return true;
            }
        }

        return false;
    }
};
```

---

## 6. Story Points

---

**Story Point 1 — "Directed ≠ Undirected: you can't use `parent` trick"**

In undirected graphs, we tracked parent to avoid falsely claiming a cycle from retracing the same edge. In directed graphs, there's no "same edge" issue — edges are one-way. The problem is shared destinations (DAG nodes reachable from multiple paths). `pathVis` / state `2` solves this correctly.

---

**Story Point 2 — "pathVis = 0 on backtrack is the whole algorithm"**

The reset `pathVis[node] = 0` (or `vis[node] = 1`) is what makes everything correct. Without it, once a node is explored from one path, it would look "on path" to all future DFS calls — giving false cycle detections. The reset says "I'm done with this node, it's safe."

---

**Story Point 3 — "Cycle = back edge in directed DFS"**

In DFS on directed graphs, edges fall into categories:
- **Tree edge** — discovers unvisited node
- **Back edge** — visits a node currently on the DFS stack (ancestor in recursion)
- **Cross/Forward edge** — visits a node already fully done

A cycle exists if and only if a **back edge** exists. `pathVis[neighbor] == 1` or `vis[neighbor] == 2` = back edge detection.

---

**Story Point 4 — "Approach 2 is Approach 1 with merged arrays"**

Approach 1 uses `vis + pathVis` with values `{0,0}, {1,1}, {1,0}`.  
Approach 2 encodes these as `{0}, {2}, {1}`.  
Same logic, same TC/SC (in terms of complexity — approach 2 uses half the actual memory).

---

**Story Point 5 — "Directed edge only in adjacency list"**

```cpp
adjLs[u].push_back(v);   // only u→v, NOT v→u
```

Unlike undirected graph problems where we add both directions, directed graphs only add `u→v`. Missing this creates a fundamentally wrong graph (you'd be treating a directed graph as undirected).

---

## 7. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V vectors + E directed edges |
| All DFS calls combined | `O(V + E)` | Each node entered and exited once; each edge traversed once |

**Total: `O(V + E)`**

> Unlike undirected graphs (where each edge is `2E` in the adjacency list), directed graphs have each edge appearing exactly once → `O(V + E)` not `O(V + 2E)`.

---

### Space Complexity

**Approach 1:**

| Structure | Size |
|---|---|
| `vis[]` | `O(V)` |
| `pathVis[]` | `O(V)` |
| Recursion stack | `O(V)` worst case |
| **Total** | `O(3V)` = `O(V)` |

**Approach 2:**

| Structure | Size |
|---|---|
| `vis[]` (3 states) | `O(V)` |
| Recursion stack | `O(V)` worst case |
| **Total** | `O(2V)` = `O(V)` |

---

### Undirected vs Directed Cycle Detection

| | Undirected | Directed |
|---|---|---|
| **Key tracker** | `parent` node | `pathVis` / state `2` |
| **Cycle signal** | Visited neighbor ≠ parent | Visited neighbor is on current path |
| **False positive risk** | Parent revisit (same edge) | Shared DAG nodes (different paths) |
| **Arrays needed** | `vis[]` only | `vis[] + pathVis[]` OR `vis[]` with 3 states |
| **Edge count** | `2E` in adj list | `E` in adj list |
| **TC** | `O(V + 2E)` | `O(V + E)` |
