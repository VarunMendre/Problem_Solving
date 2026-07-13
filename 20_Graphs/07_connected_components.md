# Connected Components in a Graph

---

## 1. Intuition / Approach

### What is a Connected Component?

A **connected component** is a maximal group of nodes where every node is reachable from every other node in the group. "Maximal" means you can't add any more nodes to the group — it's already as large as it can be.

```
Graph:
0 — 1 — 2       5 — 6
    |
    3       4

Components:
  Component 1: {0, 1, 2, 3}
  Component 2: {4}
  Component 3: {5, 6}
```

---

### Where Does the Intuition Come From?

Think of it like spreading ink on a paper graph. Drop ink on node `0` — it flows through every edge and reaches every node connected to `0`. When the ink stops flowing, everything stained is one component.

Drop ink on the next unstained node → another component gets colored.

Repeat until every node is stained. The number of ink drops = number of components.

**DFS is the ink.** Starting from any unvisited node, DFS naturally "flows" through all reachable nodes via recursive calls — just like ink spreading through connected edges.

---

### The Extra `component` Vector

The difference from just **counting** components (previous problem) is that here we want to **record which nodes belong to which component**.

So we pass a `component` vector into each DFS call. Every time we visit a node, we record it in `component`. When DFS finishes (the ink stops spreading), that `component` vector holds all nodes of one complete connected component → push it into `result`.

---

### Algorithm (High Level)

```
1. Build adjacency list from edge list
2. Initialize vis[] = all 0s, result = empty
3. For each node i from 0 to V-1:
      If i is not visited:
          Create empty component[]
          Run DFS(i) → fills component[] with all reachable nodes
          Push component[] into result
4. Return result
```

---

## 2. Dry Run

```
V = 7
Edges: [(0,1), (1,2), (1,3), (5,6)]

Adjacency List:
0 → [1]
1 → [0, 2, 3]
2 → [1]
3 → [1]
4 → []         ← isolated node
5 → [6]
6 → [5]

vis = [0, 0, 0, 0, 0, 0, 0]
result = []
```

---

**i = 0:** `vis[0] == 0` → new component found

```
dfs(0):
  vis[0]=1, component=[0]
  neighbor 1 → unvisited → dfs(1):
    vis[1]=1, component=[0,1]
    neighbor 0 → visited → skip
    neighbor 2 → unvisited → dfs(2):
      vis[2]=1, component=[0,1,2]
      neighbor 1 → visited → skip
      ← return
    neighbor 3 → unvisited → dfs(3):
      vis[3]=1, component=[0,1,2,3]
      neighbor 1 → visited → skip
      ← return
    ← return dfs(1)
  ← return dfs(0)

component = [0, 1, 2, 3] → push to result
result = [[0,1,2,3]]
vis    = [1, 1, 1, 1, 0, 0, 0]
```

---

**i = 1:** `vis[1] == 1` → skip  
**i = 2:** `vis[2] == 1` → skip  
**i = 3:** `vis[3] == 1` → skip

---

**i = 4:** `vis[4] == 0` → new component found

```
dfs(4):
  vis[4]=1, component=[4]
  no neighbors → return immediately

component = [4] → push to result
result = [[0,1,2,3], [4]]
vis    = [1, 1, 1, 1, 1, 0, 0]
```

---

**i = 5:** `vis[5] == 0` → new component found

```
dfs(5):
  vis[5]=1, component=[5]
  neighbor 6 → unvisited → dfs(6):
    vis[6]=1, component=[5,6]
    neighbor 5 → visited → skip
    ← return
  ← return dfs(5)

component = [5, 6] → push to result
result = [[0,1,2,3], [4], [5,6]]
vis    = [1, 1, 1, 1, 1, 1, 1]
```

**i = 6:** `vis[6] == 1` → skip

**Final result: `[[0,1,2,3], [4], [5,6]]` ✅**

---

## 3. Code

```cpp
#include <bits/stdc++.h>

class Solution {
private:
    void dfs(int node, vector<int>& vis,
             vector<vector<int>>& adjLs,
             vector<int>& component) {

        vis[node] = 1;                      // mark current node as visited
        component.push_back(node);          // record it in the current component

        for(auto& neighbor : adjLs[node]) {
            if(!vis[neighbor])              // only recurse on unvisited neighbors
                dfs(neighbor, vis, adjLs, component);
        }
    }

public:
    vector<vector<int>> getComponents(int V, vector<vector<int>>& edges) {

        // Step 1: Build adjacency list from edge list
        vector<vector<int>> adjLs(V);
        for(auto& e : edges) {
            adjLs[e[0]].push_back(e[1]);   // undirected: add both directions
            adjLs[e[1]].push_back(e[0]);
        }

        // Step 2: Initialize visited array and result
        vector<int> vis(V, 0);
        vector<vector<int>> result;

        // Step 3: For every unvisited node, discover its component
        for(int i = 0; i < V; i++) {
            if(vis[i] == 0) {               // found a new component
                vector<int> component;
                dfs(i, vis, adjLs, component);  // DFS fills the component
                result.push_back(component);    // store it
            }
        }

        return result;
    }
};
```

---

## 4. Complexity Analysis

### Time Complexity — `O(V + 2E)`

Every piece of work can be accounted for:

| Work | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + 2E)` | V vectors created + each edge inserted twice (undirected) |
| Outer `for` loop | `O(V)` | Runs V times — one check per node |
| All DFS calls combined | `O(V + 2E)` | See breakdown below |

**Why all DFS calls together = `O(V + 2E)` and NOT `O(components × (V+E))`:**

The `vis[]` array is **global across all DFS calls**. Once a node is marked visited, it is **never entered again** by any future DFS call. So:

```
Total "mark visited" operations  = V     (each node exactly once)
Total neighbor iterations        = 2E    (each undirected edge seen from both endpoints)

All DFS combined = O(V + 2E)
```

The number of components doesn't matter — it's always `O(V + 2E)` total.

**Overall TC = `O(V + 2E)`**

---

### Space Complexity

**Auxiliary space to solve: `O(2V)`**

| Structure | Size | Reason |
|---|---|---|
| `vis[]` array | `O(V)` | One entry per node |
| Recursion call stack | `O(V)` worst case | Worst case: linear chain — DFS goes V levels deep |

**Space including output: `O(3V + 2E)`**

| Structure | Size | Reason |
|---|---|---|
| `adjLs` adjacency list | `O(V + 2E)` | Stores all edge info |
| `vis[]` array | `O(V)` | |
| Recursion call stack | `O(V)` | |
| `result` (output) | `O(V)` | All V nodes stored across all components |

---

### Worst Case for Recursion Stack

**Linear chain — deepest stack:**
```
0 — 1 — 2 — 3 — ... — V-1

DFS(0) → DFS(1) → DFS(2) → ... → DFS(V-1)
Stack depth = V → O(V)
```

**Star graph — shallowest stack:**
```
      0
  /|||||\ 
 1 2 3 4 5

DFS(0) → DFS(1) returns immediately
       → DFS(2) returns immediately
       → ...
Max stack depth = 2 → O(1)
```

---

### This vs Just Counting Components

| | Count Only | Return Components |
|---|---|---|
| **Extra structure** | None | `component[]` vector per DFS + `result[]` |
| **TC** | `O(V + 2E)` | `O(V + 2E)` — same |
| **SC** | `O(2V)` | `O(3V + 2E)` — extra V for result storage |
| **Output** | Single integer | Vector of vectors |
