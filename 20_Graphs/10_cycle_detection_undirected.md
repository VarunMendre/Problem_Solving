# Cycle Detection in an Undirected Graph

---

## 1. Intuition / Approach

### What is a Cycle?

A **cycle** exists when you can start at a node, travel through edges, and return to the same node without reusing any edge.

```
Cyclic:              Acyclic:
0 — 1               0 — 1
|   |               |
2 — 3               2 — 3
(0→1→3→2→0)
```

---

### The Core Challenge — The Parent Problem

In an **undirected graph**, every edge appears in both directions. So when you're at node `B` and you came from node `A`, the adjacency list of `B` will always contain `A`.

If we just said "if any neighbor is already visited → cycle", we'd get false positives every time — because the parent `A` is always "already visited".

**We need to distinguish between:**
- Going back to the **parent** → just retracing the same edge → NOT a cycle
- Going to **any other visited node** → found a completely different path back → CYCLE

This is why we track the `parent` node.

---

### The `else if (adjacentNode != parent)` — In Full Depth

This is the heart of cycle detection in undirected graphs. Let's build the reasoning from scratch.

---

Suppose the graph looks like this:

```
A — B — C
|       |
+-------+
```

We start BFS/DFS from `A`. We visit `A`, then `B`, then from `B` we look at its neighbors.

**Standing at node `B`, looking at neighbors:**

```
B's neighbors: [A, C]
B's parent: A   (we came to B from A)
```

**Neighbor `A`:**

`A` is already visited. But `A` is also `B`'s parent. Going back to `A` is simply retracing the **same edge** `A→B` in reverse. There's no new path here — just the edge we just walked backwards. This is NOT a cycle. We skip it.

**Neighbor `C`:**

`C` is already visited. But `C` is **not** `B`'s parent.

Now think carefully: *How did `C` get visited?*

`C` wasn't visited by `B` — `B` is only now trying to visit it. So `C` must have been visited through a **completely different path**. In this example, the path was `A → C` (directly).

So now we know:
1. There's a path from the start that reached `C` (the original visit path)
2. There's another path that goes through `B` to reach `C`

Since this is an undirected graph, we can combine these two paths:

```
Path 1 (forward): A → C
Path 2 (through B): A → B → C

Combine: Start at A → go to C (path 1) → come back to A via B (path 2 reversed)
Full cycle: A → C → B → A ✅
```

The moment we find a visited neighbor that is NOT our parent, we've found **two independent paths to the same node** — and two paths between any pair of nodes in an undirected graph always form a cycle.

**This is exactly what the condition checks:**

```cpp
else if (adjacentNode != parent) {
    return true;   // visited + not parent = two paths = cycle
}
```

---

### Summary of the 3 Cases

When looking at `adjacentNode` from `node`:

| Case | Condition | Meaning | Action |
|---|---|---|---|
| Unvisited | `!vis[adjacentNode]` | First time reaching this node | Visit it, continue BFS/DFS |
| Visited + parent | `vis[adjacentNode] && adjacentNode == parent` | Just retracing the same edge backwards | **Skip — not a cycle** |
| Visited + not parent | `vis[adjacentNode] && adjacentNode != parent` | Found another path to an already-reached node | **CYCLE detected → return true** |

---

### Why the Outer Loop?

```cpp
for(int i = 0; i < V; i++) {
    if(!vis[i]) {
        if(detectCycle(i, adjLs, vis))
            return true;
    }
}
```

For **disconnected graphs**, one BFS/DFS call from node `0` only explores the component containing `0`. If there's a cycle in a different component, we'd miss it. The outer loop ensures every component is checked.

---

## 2. BFS Approach

### How `parent` is tracked in BFS

In BFS, we can't use a function parameter for parent (no recursion). So we store `{node, parent}` pairs in the queue itself.

```
queue stores: {node, parent}
```

When we dequeue a pair, we know which node we're at AND which node we came from — giving us the parent check.

### Dry Run (BFS)

```
Graph:
0 — 1
|   |
2 — 3

Adjacency List:
0 → [1, 2]
1 → [0, 3]
2 → [0, 3]
3 → [1, 2]

vis = [0,0,0,0]
```

**detectCycle(src=0):**
```
queue = [{0, -1}],  vis[0]=1

Pop (0, parent=-1):
  neighbor 1: not visited → vis[1]=1, push {1, 0}
  neighbor 2: not visited → vis[2]=1, push {2, 0}

queue = [{1,0}, {2,0}]

Pop (1, parent=0):
  neighbor 0: visited, 0 == parent(0) → skip (just our parent)
  neighbor 3: not visited → vis[3]=1, push {3, 1}

queue = [{2,0}, {3,1}]

Pop (2, parent=0):
  neighbor 0: visited, 0 == parent(0) → skip
  neighbor 3: visited, 3 ≠ parent(0) → CYCLE DETECTED ✅
  return true
```

### BFS Code

```cpp
class Solution {
private:
    bool detectCycle(int src, vector<vector<int>>& adjLs, vector<int>& vis) {
        // Queue stores {node, parent}
        queue<pair<int,int>> q;

        q.push({src, -1});
        vis[src] = 1;

        while(!q.empty()) {
            int node   = q.front().first;
            int parent = q.front().second;
            q.pop();

            for(auto& adjacentNode : adjLs[node]) {
                if(!vis[adjacentNode]) {
                    // Unvisited → explore it
                    vis[adjacentNode] = 1;
                    q.push({adjacentNode, node});
                }
                else if(adjacentNode != parent) {
                    // Visited + not our parent → two paths exist → cycle
                    return true;
                }
                // Visited + is our parent → just same edge backwards → skip
            }
        }

        return false;
    }

public:
    bool isCycle(int V, vector<vector<int>>& edges) {
        // Build adjacency list
        vector<vector<int>> adjLs(V);
        for(auto& edge : edges) {
            adjLs[edge[0]].push_back(edge[1]);
            adjLs[edge[1]].push_back(edge[0]);
        }

        vector<int> vis(V, 0);

        // Check every component
        for(int i = 0; i < V; i++) {
            if(!vis[i]) {
                if(detectCycle(i, adjLs, vis))
                    return true;
            }
        }

        return false;
    }
};
```

---

## 3. DFS Approach

### How `parent` is tracked in DFS

In DFS, `parent` is passed as a **function parameter** — naturally carried through the recursion call stack.

```cpp
detectCycle(node, parent, adjLs, vis)
```

The logic is identical to BFS — the only structural difference is recursion vs queue.

### Dry Run (DFS)

```
Same graph: 0—1—3—2—0

detectCycle(0, parent=-1):
  vis[0]=1
  neighbor 1: not visited →
    detectCycle(1, parent=0):
      vis[1]=1
      neighbor 0: visited, 0==parent → skip
      neighbor 3: not visited →
        detectCycle(3, parent=1):
          vis[3]=1
          neighbor 1: visited, 1==parent → skip
          neighbor 2: not visited →
            detectCycle(2, parent=3):
              vis[2]=1
              neighbor 0: visited, 0 ≠ parent(3) → CYCLE ✅ return true
```

### DFS Code

```cpp
class Solution {
private:
    bool detectCycle(int node, int parent,
                     vector<vector<int>>& adjLs, vector<int>& vis) {
        vis[node] = 1;

        for(auto& adjacentNode : adjLs[node]) {
            if(!vis[adjacentNode]) {
                // Unvisited → recurse deeper
                if(detectCycle(adjacentNode, node, adjLs, vis))
                    return true;
            }
            else if(adjacentNode != parent) {
                // Visited + not our parent → cycle found
                return true;
            }
            // Visited + is our parent → same edge backwards → skip
        }

        return false;
    }

public:
    bool isCycle(int V, vector<vector<int>>& edges) {
        vector<vector<int>> adjLs(V);
        for(auto& edge : edges) {
            adjLs[edge[0]].push_back(edge[1]);
            adjLs[edge[1]].push_back(edge[0]);
        }

        vector<int> vis(V, 0);

        for(int i = 0; i < V; i++) {
            if(!vis[i]) {
                if(detectCycle(i, -1, adjLs, vis))
                    return true;
            }
        }

        return false;
    }
};
```

---

## 4. Complexity Analysis

### Time Complexity — `O(V + 2E)`

| Work | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + 2E)` | V vectors + each undirected edge added twice |
| Outer loop | `O(V)` | Checks every node |
| BFS/DFS combined | `O(V + 2E)` | Each node visited once; each edge endpoint iterated once from each side |

**Total: `O(V + 2E)`**

---

### Space Complexity

**BFS:**

| Structure | Size |
|---|---|
| `vis[]` | `O(V)` |
| Queue (worst case) | `O(V)` |
| **Total** | `O(2V)` = `O(V)` |

**DFS:**

| Structure | Size |
|---|---|
| `vis[]` | `O(V)` |
| Recursion call stack (worst case) | `O(V)` — linear chain |
| **Total** | `O(2V)` = `O(V)` |

---

## 5. BFS vs DFS for Cycle Detection

| | BFS | DFS |
|---|---|---|
| **Parent tracking** | Stored in queue as `{node, parent}` | Passed as function parameter |
| **Code style** | Iterative | Recursive |
| **Stack overflow risk** | None | Yes, for very deep graphs |
| **Time Complexity** | `O(V + 2E)` | `O(V + 2E)` |
| **Space** | `O(V)` queue | `O(V)` call stack |
| **Result** | Same | Same |

> The **logic is identical** — the only difference is HOW the parent is tracked and HOW neighbors are processed (queue vs call stack).
