# Topological Sort (DFS)

---

## 1. Problem Statement

Given a **Directed Acyclic Graph (DAG)** with `V` vertices and a list of directed edges, return a **topological ordering** of all vertices.

A topological ordering is a linear arrangement of vertices such that for every directed edge `u → v`, vertex `u` appears **before** `v` in the ordering.

```
Input:
Vertices: 6
Edges: 5→0, 5→2, 4→0, 4→1, 2→3, 3→1

Output (one valid ordering):
[5, 4, 2, 3, 1, 0]
```

> **Note:** Topological sort is only defined for **DAGs** (Directed Acyclic Graphs). A graph with a cycle has no valid topological ordering — if `A → B → A`, A can't come before B and B can't come before A simultaneously.

---

## 2. What is a Topological Sort? A Detailed Definition

### Formal Definition

A **topological sort** (or topological ordering) of a DAG is a linear ordering of all its vertices such that:

> For every directed edge `(u, v)` in the graph, vertex `u` appears **before** vertex `v` in the ordering.

This means: **dependencies come first**.

### Breaking It Down

Think of vertices as tasks and directed edges as dependencies:
- Edge `u → v` means: "task `u` must be completed before task `v` can start"
- A topological ordering gives a valid execution sequence — no task starts before all its prerequisites are done

### Key Properties

| Property | Explanation |
|---|---|
| **Only for DAGs** | Cyclic graphs have no valid topological order |
| **Not necessarily unique** | Multiple valid orderings can exist |
| **All vertices included** | Every vertex appears exactly once |
| **Respects all edges** | For every `u→v`, `u` appears before `v` |

### When Does a Valid Ordering NOT Exist?

If the graph has a cycle: `A → B → C → A`

```
A must come before B (A→B)
B must come before C (B→C)
C must come before A (C→A)

But then A must come before itself → impossible → no topological order
```

---

## 3. Examples of Topological Sort

### Example 1 — Course Prerequisites

```
Graph (courses with prerequisites):
CS101 → CS201 → CS301
CS101 → CS202 → CS301
           ↓
         CS401

Valid topological orders:
[CS101, CS201, CS202, CS301, CS401]
[CS101, CS202, CS201, CS301, CS401]
(both are valid — CS201 and CS202 have no ordering constraint between them)
```

### Example 2 — Build System

```
Graph (file compilation dependencies):
main.cpp → utils.h → config.h
main.cpp → io.h

Topological order: config.h, utils.h, io.h, main.cpp
(compile dependencies before the files that need them)
```

### Example 3 — Simple Graph

```
Graph:
5 → 0
5 → 2
4 → 0
4 → 1
2 → 3
3 → 1

Visual:
5 ——→ 0
|         ↑
↓    4 ——→|
2    |
|    ↓
↓    1
3 ————→ 1

Valid orderings include:
[4, 5, 0, 2, 3, 1]
[5, 4, 0, 2, 3, 1]
[5, 4, 2, 3, 0, 1]
(multiple valid answers — any that respects all edge directions)
```

---

## 4. Our Approach to Solving This Problem — DFS + Stack

### The Algorithm in One Line

**Run DFS. Push a node onto the stack only AFTER all its descendants have been fully explored.**

This guarantees that when we pop the stack, nodes come out in topological order.

### The Three Steps

```
Step 1: Build directed adjacency list from edges
Step 2: For each unvisited node, run DFS
        - Explore all neighbors first (recursively)
        - Then push current node onto stack (post-order)
Step 3: Pop stack into result array → topological order
```

---

## 5. Intuition — How We Developed This Insight and Its Rationale

### Starting From the Definition

Topological sort requires: for every `u → v`, `u` must appear BEFORE `v`.

Flipping it: `v` must appear AFTER `u`.

In other words: **a node can only be placed in the result AFTER all nodes it points to have been placed.**

This is exactly **post-order DFS** — process children before parent.

---

### Building the Intuition Step by Step

**Step 1 — What should the last node in topological order look like?**

The last node has no outgoing edges (or all nodes it points to are already placed). It's a "leaf" in the dependency chain — nothing depends on it being first, so it can safely go last.

**Step 2 — What should the first node look like?**

The first node has nothing pointing to IT — no prerequisites. It has zero indegree (no incoming edges from unplaced nodes).

**Step 3 — The recursive insight**

If we process children before parents (post-order DFS), then by the time we "place" node `u`, all nodes that `u` depends on (i.e., nodes reachable from `u`) are already placed.

So we push `u` to a **stack** (not directly to result), because we want to REVERSE the DFS post-order. Pushing to a stack then popping gives us the correct left-to-right order.

---

### Why Stack and Not Just Append to Result?

DFS explores deeply — it might push a deep child (like node `3`) long before a shallow parent (like node `5`). If we appended directly, children would come before parents in the result array — WRONG.

Stack reverses this: the first nodes pushed are the deepest ones (no outgoing edges). When popped, they come out LAST — which is exactly where they should be in topological order.

```
DFS post-order pushes: 1, 0, 3, 2, 5, 4   (deepest first)
Stack pop gives:       4, 5, 2, 3, 0, 1   (sources first = correct topological order)
```

---

### The Key Guarantee

> A node is pushed to the stack **only after** all its reachable nodes (in the current DFS tree) have been pushed.

So when we pop the stack, every node comes out before any node that it has a directed edge TO. That's precisely the topological ordering guarantee.

---

## 6. Dry Run

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

vis = [0,0,0,0,0,0], stack = []
```

---

**Outer loop i=0:**
```
vis[0]=0 → dfs(0):
  vis[0]=1
  no neighbors
  push 0 → stack=[0]
```

**i=1:**
```
vis[1]=0 → dfs(1):
  vis[1]=1
  no neighbors
  push 1 → stack=[0,1]
```

**i=2:**
```
vis[2]=0 → dfs(2):
  vis[2]=1
  neighbor 3: vis[3]=0 → dfs(3):
    vis[3]=1
    neighbor 1: vis[1]=1 → skip
    push 3 → stack=[0,1,3]
    ← return dfs(3)
  push 2 → stack=[0,1,3,2]
  ← return dfs(2)
```

**i=3:** vis[3]=1 → skip

**i=4:**
```
vis[4]=0 → dfs(4):
  vis[4]=1
  neighbor 0: vis[0]=1 → skip
  neighbor 1: vis[1]=1 → skip
  push 4 → stack=[0,1,3,2,4]
  ← return dfs(4)
```

**i=5:**
```
vis[5]=0 → dfs(5):
  vis[5]=1
  neighbor 0: vis[0]=1 → skip
  neighbor 2: vis[2]=1 → skip
  push 5 → stack=[0,1,3,2,4,5]
  ← return dfs(5)
```

**Pop stack into result:**
```
pop 5 → [5]
pop 4 → [5,4]
pop 2 → [5,4,2]
pop 3 → [5,4,2,3]
pop 1 → [5,4,2,3,1]
pop 0 → [5,4,2,3,1,0]
```

**Result: [5, 4, 2, 3, 1, 0]**

**Verify:**
```
5→0: 5 at index 0, 0 at index 5 → 0 < 5 ✅
5→2: 5 at index 0, 2 at index 2 → 0 < 2 ✅
4→0: 4 at index 1, 0 at index 5 → 1 < 5 ✅
4→1: 4 at index 1, 1 at index 4 → 1 < 4 ✅
2→3: 2 at index 2, 3 at index 3 → 2 < 3 ✅
3→1: 3 at index 3, 1 at index 4 → 3 < 4 ✅

All edges respected → valid topological order ✅
```

---

## 7. Code Implementation

```cpp
class Solution {
private:
    void topologicalDfs(int node, stack<int>& st, int vis[],
                        vector<vector<int>>& adjLs) {

        vis[node] = 1;   // mark as visited

        // First: explore all neighbors (go deeper)
        for(auto& neighbour : adjLs[node]) {
            if(!vis[neighbour])
                topologicalDfs(neighbour, st, vis, adjLs);
        }

        // Then: push current node AFTER all its descendants are pushed
        // This ensures all nodes that come "after" this node in the graph
        // are already in the stack below this node
        st.push(node);
    }

public:
    vector<int> topoSort(int V, vector<vector<int>>& edges) {
        // Step 1: Build directed adjacency list
        vector<vector<int>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].push_back(it[1]);   // directed: only u→v
        }

        int vis[V] = {0};
        stack<int> st;

        // Step 2: DFS from every unvisited node
        for(int i = 0; i < V; i++) {
            if(!vis[i])
                topologicalDfs(i, st, vis, adjLs);
        }

        // Step 3: Pop stack → topological order (reversed post-order)
        vector<int> result;
        while(!st.empty()) {
            result.push_back(st.top());
            st.pop();
        }

        return result;
    }
};
```

---

## 8. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists created + E directed edges inserted |
| All DFS calls combined | `O(V + E)` | Each node visited once; each directed edge traversed once |
| Stack pop into result | `O(V)` | V nodes popped |

**Total: `O(V + E)`**

> Each node is pushed to the stack exactly once (visited array prevents re-entry). Each directed edge is traversed exactly once (unlike undirected graphs which have `2E`).

---

### Space Complexity — `O(V)`

| Structure | Size | Reason |
|---|---|---|
| `vis[]` array | `O(V)` | One per node |
| `stack<int>` | `O(V)` | Holds all V nodes |
| Recursion call stack | `O(V)` worst case | Linear chain forces depth V |
| `result` vector | `O(V)` | Stores all V nodes |

**Total: `O(4V)` = `O(V)`**

---

### DFS Topological Sort vs BFS (Kahn's Algorithm)

| | DFS (this approach) | BFS / Kahn's Algorithm |
|---|---|---|
| **Mechanism** | Post-order DFS + stack | Process nodes with indegree 0 first |
| **Data structures** | `vis[]` + `stack` | `indegree[]` + `queue` |
| **Cycle detection** | No (assumes DAG) | Yes — if result size < V, cycle exists |
| **TC** | `O(V + E)` | `O(V + E)` |
| **SC** | `O(V)` | `O(V)` |
| **Natural for** | When you want post-order finalization | When you want to detect cycles too |
