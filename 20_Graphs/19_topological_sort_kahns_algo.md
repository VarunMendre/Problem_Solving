# Topological Sort — Kahn's Algorithm (BFS)

---

## 1. Approach — Kahn's Algorithm

Kahn's Algorithm solves topological sort using **BFS** instead of DFS. The core idea is built entirely around one concept:

> **A node with no prerequisites (indegree = 0) can always be placed first.**

We repeatedly pick nodes with zero indegree, add them to the result, and "remove" their contribution to their neighbors — which may create new zero-indegree nodes. Repeat until all nodes are placed.

### The Three Steps

```
Step 1: Calculate indegree of every node
        (indegree = number of edges coming INTO the node)

Step 2: Push all nodes with indegree == 0 into a queue
        (these have no prerequisites — safe to process first)

Step 3: BFS loop:
          - Pop a node → add to result
          - For each neighbor: decrement its indegree by 1
          - If neighbor's indegree becomes 0 → push to queue
            (all its prerequisites have been placed)
```

---

## 2. Intuition — How and Why This Works

### What Does Indegree Tell Us?

**Indegree of a node = number of unprocessed prerequisites it still has.**

When we initialize, `indegree[v]` = total edges pointing into `v` = total prerequisites of `v`.

When we process a node `u` and "remove" it from the graph, we decrement `indegree[v]` for every neighbor `v` of `u`. This means: "one of `v`'s prerequisites (`u`) has been satisfied."

When `indegree[v]` hits `0` — ALL prerequisites of `v` are satisfied → `v` is ready to be processed → push to queue.

---

### The Removal Analogy

Imagine the graph as a pile of tasks with dependency strings attached. You can only pick up a task if all strings connecting it to unfinished tasks are gone.

- Start: pull out all tasks with NO strings attached (indegree = 0)
- Each task you complete: cut the strings it was holding onto other tasks
- Some tasks become free (indegree → 0) → pull those out next
- Repeat until all tasks are done

This is exactly Kahn's algorithm.

---

### Why BFS and Not DFS?

DFS for topological sort works backwards — goes as deep as possible, then places on a stack, relies on reversal. It's less intuitive.

Kahn's is **forward thinking** — at every moment, we're placing a node that is **currently safe to place** (indegree = 0 at this moment). No reversal needed, no stack needed. The queue naturally processes in the correct order.

---

### The Bonus — Cycle Detection

If the graph has a cycle, no node in the cycle ever reaches indegree = 0 (they all depend on each other). So the BFS queue empties before all nodes are placed.

```
If result.size() < V → cycle exists in the graph
If result.size() == V → no cycle, valid topological order returned
```

This is a **free** cycle detector — DFS topological sort doesn't give this without additional logic.

---

### Indegree vs Visited Array

Note that Kahn's uses NO `vis[]` array. The `indegree` array serves the same purpose — once a node's indegree hits 0 and it's pushed to the queue, it's never pushed again (its indegree can only decrease, never increase). So duplicate processing is naturally impossible.

---

## 3. Dry Run

```
V = 6
Edges: 5→0, 5→2, 4→0, 4→1, 2→3, 3→1

Adjacency List:
0 → []
1 → []
2 → [3]
3 → [1]
4 → [0, 1]
5 → [0, 2]
```

**Step 1 — Calculate indegrees:**

```
Edge 5→0: indegree[0]++
Edge 5→2: indegree[2]++
Edge 4→0: indegree[0]++
Edge 4→1: indegree[1]++
Edge 2→3: indegree[3]++
Edge 3→1: indegree[1]++

indegree = [2, 2, 1, 1, 0, 0]
           [0, 1, 2, 3, 4, 5]
```

**Step 2 — Seed queue with indegree == 0:**

```
Node 4: indegree=0 ✅ → push
Node 5: indegree=0 ✅ → push

queue = [4, 5]
result = []
```

---

**BFS Iteration 1 — Pop 4:**

```
result = [4]
Neighbors of 4: [0, 1]
  indegree[0]-- → 2-1 = 1 (not 0 yet)
  indegree[1]-- → 2-1 = 1 (not 0 yet)

indegree = [1, 1, 1, 1, 0, 0]
queue = [5]
```

**BFS Iteration 2 — Pop 5:**

```
result = [4, 5]
Neighbors of 5: [0, 2]
  indegree[0]-- → 1-1 = 0 ✅ → push 0
  indegree[2]-- → 1-1 = 0 ✅ → push 2

indegree = [0, 1, 0, 1, 0, 0]
queue = [0, 2]
```

**BFS Iteration 3 — Pop 0:**

```
result = [4, 5, 0]
Neighbors of 0: [] → nothing

queue = [2]
```

**BFS Iteration 4 — Pop 2:**

```
result = [4, 5, 0, 2]
Neighbors of 2: [3]
  indegree[3]-- → 1-1 = 0 ✅ → push 3

indegree = [0, 1, 0, 0, 0, 0]
queue = [3]
```

**BFS Iteration 5 — Pop 3:**

```
result = [4, 5, 0, 2, 3]
Neighbors of 3: [1]
  indegree[1]-- → 1-1 = 0 ✅ → push 1

queue = [1]
```

**BFS Iteration 6 — Pop 1:**

```
result = [4, 5, 0, 2, 3, 1]
Neighbors of 1: [] → nothing

queue = []  → BFS ends
```

**Final result: [4, 5, 0, 2, 3, 1]**

**Verify all edges:**
```
5→0: 5 at idx 1, 0 at idx 2 → 1 < 2 ✅
5→2: 5 at idx 1, 2 at idx 3 → 1 < 3 ✅
4→0: 4 at idx 0, 0 at idx 2 → 0 < 2 ✅
4→1: 4 at idx 0, 1 at idx 5 → 0 < 5 ✅
2→3: 2 at idx 3, 3 at idx 4 → 3 < 4 ✅
3→1: 3 at idx 4, 1 at idx 5 → 4 < 5 ✅

All edges respected ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Indegree = unfinished prerequisites"**

This is the core mental model. Don't think of indegree as a static property — think of it as a **live counter** that decrements as prerequisites are completed. When it hits zero, all prerequisites are done → the node is ready.

---

**Story Point 2 — "No vis[] array needed — indegree does the job"**

Once a node is pushed to the queue (indegree → 0), its indegree can never go back up (we only decrement). So it can never be pushed again. The indegree array naturally prevents duplicate processing — no separate visited array required.

---

**Story Point 3 — "Multiple valid answers are possible"**

The order in which nodes with indegree = 0 enter the queue determines the output. In the dry run, nodes 4 and 5 both start with indegree = 0. Scanning left to right gives `[4, 5, ...]`. Scanning right to left would give `[5, 4, ...]`. Both are valid topological orderings.

---

**Story Point 4 — "Cycle detection is free"**

```cpp
if(result.size() != V)
    // cycle detected — not a valid DAG
```

Nodes in a cycle never reach indegree = 0 (they all point to each other). They never enter the queue and never join the result. So `result.size() < V` is a direct indicator of a cycle. This is something DFS topological sort can't do without extra logic (`pathVis`).

---

**Story Point 5 — "Kahn's is inherently forward, DFS is inherently backward"**

| | DFS Topological Sort | Kahn's Algorithm |
|---|---|---|
| Direction | Explores depth-first, reverses via stack | Processes ready nodes first, moves forward |
| Needs reversal? | Yes (stack reverses post-order) | No (queue gives correct order directly) |
| Natural for | Post-processing after full exploration | Greedy "pick what's ready now" |
| Cycle detection | Needs extra `pathVis` array | Free — check `result.size() == V` |

---

## 5. Code

```cpp
class Solution {
public:
    vector<int> topoSort(int V, vector<vector<int>>& edges) {

        // Step 1: Build directed adjacency list + compute indegrees
        vector<vector<int>> adjLs(V);
        vector<int> indegree(V, 0);

        for(auto& it : edges) {
            int u = it[0], v = it[1];
            adjLs[u].push_back(v);   // directed: u→v only
            indegree[v]++;            // v has one more prerequisite
        }

        // Step 2: Seed queue with all nodes that have no prerequisites
        queue<int> q;
        for(int i = 0; i < V; i++) {
            if(indegree[i] == 0)
                q.push(i);
        }

        vector<int> result;

        // Step 3: BFS — process ready nodes, unlock their neighbors
        while(!q.empty()) {
            int node = q.front();
            q.pop();

            result.push_back(node);   // this node is placed in topo order

            for(auto& neighbour : adjLs[node]) {
                indegree[neighbour]--;           // one prerequisite satisfied
                if(indegree[neighbour] == 0)     // all prerequisites done
                    q.push(neighbour);           // ready to be placed
            }
        }

        // Optional cycle check: if result.size() != V → cycle exists
        return result;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list + indegree | `O(V + E)` | V lists + E edges processed |
| Initial queue seeding | `O(V)` | Scan all V nodes for indegree = 0 |
| BFS loop | `O(V + E)` | Each node dequeued once `O(V)` + each edge decrements one neighbor `O(E)` |

**Total: `O(V + E)`**

> Each node enters and exits the queue exactly once → `O(V)` queue operations. Each directed edge causes exactly one `indegree[v]--` → `O(E)` total decrements. Combined: `O(V + E)`.

---

### Space Complexity — `O(V)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` adjacency list | `O(V + E)` | V lists + E entries |
| `indegree[]` array | `O(V)` | One per node |
| `queue` | `O(V)` | At most V nodes simultaneously |
| `result` vector | `O(V)` | Stores all V nodes |

**Total auxiliary: `O(V)` (excluding adjacency list storage)**

---

### DFS Topo Sort vs Kahn's (Complete Comparison)

| | DFS Topological Sort | Kahn's Algorithm |
|---|---|---|
| **Traversal** | DFS + Stack | BFS + Queue |
| **Key structure** | `vis[]` + `stack` | `indegree[]` + `queue` |
| **Direction** | Backward (reversal via stack) | Forward (greedy: place ready nodes) |
| **Cycle detection** | Needs extra `pathVis` array | Free — `result.size() != V` |
| **Time Complexity** | `O(V + E)` | `O(V + E)` |
| **Space Complexity** | `O(V)` | `O(V)` |
| **Intuition** | Post-order → reverse = topo order | Indegree = prerequisites counter |
| **Multiple valid outputs?** | Yes | Yes |
