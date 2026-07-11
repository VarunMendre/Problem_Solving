# Number of Provinces (Connected Components via DFS)

---

## 1. Problem Understanding

We're given an `N×N` matrix `isConnected` where:
- `isConnected[i][j] = 1` → city `i` and city `j` are **directly connected**
- `isConnected[i][j] = 0` → not directly connected

A **province** is a group of cities that are directly or indirectly connected. We need to count the **total number of provinces** — which is exactly the number of **connected components** in the graph.

```
isConnected = [[1,1,0],
               [1,1,0],
               [0,0,1]]

Cities 0 and 1 are connected → one province
City 2 is alone → another province
Answer = 2
```

---

## 2. The Two-Phase Approach

### Phase 1 — Convert Matrix to Adjacency List

The input is an **adjacency matrix**. Most graph algorithms work better with an adjacency list, so we convert first.

```cpp
vector<vector<int>> adjLs(V);

for(int i = 0; i < V; i++) {
    for(int j = 0; j < V; j++) {
        if(isConnected[i][j] == 1 && i != j) {
            adjLs[i].push_back(j);
        }
    }
}
```

Why `i != j`? Because `isConnected[i][i] = 1` always (a city is connected to itself), but we don't want self-loops in our adjacency list — they serve no purpose for traversal.

```
isConnected:           Adjacency List:
[[1,1,0],              0 → [1]
 [1,1,0],              1 → [0]
 [0,0,1]]              2 → []
```

---

### Phase 2 — Count Connected Components with DFS

```cpp
vector<int> vis(V, 0);
int cnt = 0;

for(int i = 0; i < V; i++) {
    if(!vis[i]) {
        cnt++;                    // new unvisited node = new province
        dfs(i, vis, adjLs);      // mark all reachable nodes from i
    }
}
```

Every time we find an **unvisited node**, it means we've discovered a brand new connected component (province). We increment `cnt` and run DFS to mark the entire component as visited.

---

## 3. How DFS Works Here

```cpp
void dfs(int node, vector<int>& vis, vector<vector<int>>& arr) {
    vis[node] = 1;                    // mark current node visited

    for(auto it : arr[node]) {        // look at all neighbors
        if(!vis[it])
            dfs(it, vis, arr);        // recurse if not visited
    }
}
```

DFS dives **as deep as possible** before backtracking — following one path all the way to its end, then coming back and trying other branches.

```
Graph: 0—1—2    3—4    5

DFS from 0:
  visit 0
    visit 1 (neighbor of 0)
      visit 2 (neighbor of 1)
        no unvisited neighbors → backtrack
      backtrack to 1 → no more unvisited → backtrack
    backtrack to 0 → no more unvisited → done
  cnt = 1

Outer loop hits 1, 2 → already visited → skip
Outer loop hits 3 → not visited:
  cnt = 2, DFS from 3 → visits 3 and 4
Outer loop hits 4 → already visited → skip
Outer loop hits 5 → not visited:
  cnt = 3, DFS from 5 → visits 5 (isolated)

Answer = 3
```

---

## 4. Dry Run — Full Example

```
V = 6
isConnected:
    0 1 2 3 4 5
0 [ 1 1 0 0 0 0 ]
1 [ 1 1 1 0 0 0 ]
2 [ 0 1 1 0 0 0 ]
3 [ 0 0 0 1 1 0 ]
4 [ 0 0 0 1 1 0 ]
5 [ 0 0 0 0 0 1 ]

Adjacency List (after conversion, excluding self-loops):
0 → [1]
1 → [0, 2]
2 → [1]
3 → [4]
4 → [3]
5 → []

vis = [0,0,0,0,0,0], cnt = 0
```

**Outer loop i=0:** `vis[0]=0` → `cnt=1`, DFS(0)
```
DFS(0): vis[0]=1
  neighbor 1: not visited → DFS(1)
    DFS(1): vis[1]=1
      neighbor 0: visited → skip
      neighbor 2: not visited → DFS(2)
        DFS(2): vis[2]=1
          neighbor 1: visited → skip
          backtrack
      backtrack
  backtrack
```
vis = [1,1,1,0,0,0], cnt=1

**Outer loop i=1,2:** already visited → skip

**Outer loop i=3:** `vis[3]=0` → `cnt=2`, DFS(3)
```
DFS(3): vis[3]=1
  neighbor 4: not visited → DFS(4)
    DFS(4): vis[4]=1
      neighbor 3: visited → skip
      backtrack
  backtrack
```
vis = [1,1,1,1,1,0], cnt=2

**Outer loop i=4:** already visited → skip

**Outer loop i=5:** `vis[5]=0` → `cnt=3`, DFS(5)
```
DFS(5): vis[5]=1
  no neighbors → return immediately
```
vis = [1,1,1,1,1,1], cnt=3

**Return 3** — three provinces: {0,1,2}, {3,4}, {5}

---

## 5. Time Complexity — In Depth: `O(N + V + 2E)`

This is the most misunderstood part. Let's build it from scratch.

### The Wrong Assumption

One might think: "We call DFS for each connected component, and each DFS does O(V+E) work, so total = O(components × (V+E))?"

**This is wrong.** Here's why.

### The Correct View — Work is Shared

The key insight: **across all DFS calls, each node is visited exactly once, and each edge is traversed exactly twice (once from each endpoint in an undirected graph).**

It doesn't matter whether the work happens in 1 DFS call or 100 DFS calls — the total work is the same.

---

### Breaking Down `O(N + V + 2E)`

**Part 1 — Outer `for` loop: `O(V)` = `O(N)`**

```cpp
for(int i = 0; i < V; i++) {    // runs N times
    if(!vis[i]) { ... }
}
```

This loop runs **V times** regardless — checking every node to see if it's been visited. Contributes `O(V)` = `O(N)`.

---

**Part 2 — All DFS calls combined: `O(V + 2E)`**

Each DFS call does two things:
1. Marks the node visited: `vis[node] = 1` → `O(1)` per node
2. Iterates over neighbors: `for(auto it : arr[node])` → `O(degree)` per node

Across **all** DFS calls combined:
- Total "mark visited" operations = **V** (each node visited exactly once)
- Total neighbor iterations = sum of all degrees = **2E** (each undirected edge counted from both endpoints — Handshaking Lemma)

So all DFS work combined = `O(V + 2E)`.

---

**Combined total: `O(N)` + `O(V + 2E)` = `O(N + V + 2E)`**

Since `N = V`:

```
O(V + V + 2E) = O(2V + 2E) = O(V + E)
```

> In the instructor's notation `O(N + V + 2E)`: `N` from outer loop, `V + 2E` from DFS internals. Since `N = V`, often simplified to `O(V + 2E)` or `O(V + E)`.

---

### Visualizing with Example

```
Graph: 0—1—2—3    4—5—6    7

3 DFS calls:
  DFS from 0: visits {0,1,2,3} → 4 nodes, 3 edges × 2 = 6 edge traversals
  DFS from 4: visits {4,5,6}   → 3 nodes, 2 edges × 2 = 4 edge traversals
  DFS from 7: visits {7}       → 1 node,  0 edges

Total: 8 nodes (= V), 10 edge traversals (= 2E for 5 edges)
Total work = V + 2E — regardless of how many DFS calls!
```

---

### Edge Case — Isolated Nodes (No Edges)

```
Graph: 0    1    2    3    4   (no edges at all, E=0)
```

Every DFS call hits a node with an empty adjacency list → `O(1)` each.
Total DFS work = `O(V)`. Outer loop = `O(N)`.
**Total = `O(N)`.** Complexity simplifies when E=0.

---

## 6. Space Complexity — In Depth: `O(2N)`

### Auxiliary Space (to SOLVE): `O(2N)`

| Structure | Size | Reason |
|---|---|---|
| `vis[]` visited array | `O(N)` | One entry per vertex |
| Recursion call stack | `O(N)` worst case | Stack depth = length of deepest path DFS follows |

**Total = `O(N) + O(N)` = `O(2N)`**

---

### Why is the Recursion Stack `O(N)` in Worst Case?

The call stack grows as deep as the **longest path DFS follows without backtracking**.

**Worst case — linear chain (skewed graph):**

```
0 — 1 — 2 — 3 — 4 — ... — N-1

DFS(0) calls DFS(1) calls DFS(2) ... calls DFS(N-1)

Call stack at deepest point:
| DFS(0)   |
| DFS(1)   |
| DFS(2)   |
|  ...     |
| DFS(N-1) |  ← N frames deep = O(N)
```

**Best case — star graph:**

```
Node 0 connects to all others: 1, 2, 3, ..., N-1
DFS(0) → DFS(1) → back → DFS(2) → back → ...
Maximum depth = 2 at any point → O(1) stack
```

We always state **worst case = O(N)**.

---

### Output Space

The result is a single integer `cnt` → `O(1)` output.
No extra output array needed.

**Total space = `O(2N)`**

---

## 7. DFS vs BFS for This Problem

| | DFS | BFS |
|---|---|---|
| **How it explores** | Deep first — follows one path to end | Wide first — explores all neighbors first |
| **Data structure** | Recursion call stack (implicit) | Explicit queue |
| **Stack space** | `O(N)` call stack | `O(N)` queue |
| **TC** | `O(V + 2E)` | `O(V + 2E)` |
| **SC** | `O(2N)` | `O(2N)` |
| **For connected components** | Both work identically |  |

For counting components, either works — what matters is that every node gets marked visited, not the order of traversal within a component.

---

## 8. Complete Code

```cpp
class Solution {
private:
    void dfs(int node, vector<int>& vis, vector<vector<int>>& arr) {
        vis[node] = 1;                      // mark current node visited
        for(auto it : arr[node]) {          // iterate over all neighbors
            if(!vis[it])                    // only visit unvisited neighbors
                dfs(it, vis, arr);          // recurse deeper into the component
        }
    }

public:
    int findCircleNum(vector<vector<int>>& isConnected) {
        int V = isConnected.size();

        // Step 1: Convert adjacency matrix → adjacency list
        vector<vector<int>> adjLs(V);
        for(int i = 0; i < V; i++) {
            for(int j = 0; j < V; j++) {
                // skip self-loops (diagonal where i==j is always 1)
                if(isConnected[i][j] == 1 && i != j) {
                    adjLs[i].push_back(j);
                }
            }
        }

        // Step 2: Count connected components
        vector<int> vis(V, 0);
        int cnt = 0;

        for(int i = 0; i < V; i++) {
            if(!vis[i]) {             // unvisited node → new province found
                cnt++;
                dfs(i, vis, adjLs);   // mark the entire component as visited
            }
        }

        return cnt;   // total number of provinces (connected components)
    }
};
```

---

## 9. Key Takeaways

| Concept | Insight |
|---|---|
| **Outer loop** | Handles disconnected graphs — starts a new DFS for each unvisited node |
| **cnt before DFS** | We discover the new component the moment we find an unvisited node |
| **DFS work is shared** | All calls combined do O(V + 2E) — not per call |
| **TC = O(N + V + 2E)** | N from outer loop + V+2E from all DFS combined |
| **SC = O(2N)** | visited array + recursion call stack |
| **Worst case stack** | Linear chain graph → N levels deep |
