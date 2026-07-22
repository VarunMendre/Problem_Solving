# Find Eventual Safe Nodes

---

## 1. Problem Statement

You are given a directed graph with `V` nodes (labeled `0` to `V-1`) represented as an adjacency list `graph` where `graph[i]` contains all nodes that node `i` has a directed edge to.

A node is called a **safe node** if every path starting from that node eventually leads to a **terminal node** (a node with no outgoing edges).

Return **all safe nodes in sorted order**.

```
graph = [[1,2],[2,3],[5],[0],[5],[],[]]

Terminal nodes: 5, 6 (no outgoing edges)
Safe nodes: 2→5 (safe), 4→5 (safe), 5 (terminal=safe), 6 (terminal=safe)
Node 0→1→2→5 → safe? No, 0→1→3→0 creates a cycle → 0,1,3 are UNSAFE

Answer: [2, 4, 5, 6]
```

---

## 2. Intuition / Approach

### What Makes a Node Unsafe?

A node is **unsafe** if there exists ANY path from it that:
- Enters a cycle (loops forever), OR
- Leads to an unsafe node

Essentially: **unsafe nodes are nodes that are part of a cycle OR can reach a cycle**.

### What Makes a Node Safe?

A node is safe if **ALL paths from it terminate** — no path enters a cycle. This means:
- Terminal nodes (indegree = 0 in original, outdegree = 0 in original) are safe
- Nodes whose ALL neighbors are safe → also safe

---

### The Key Insight — Reverse the Graph

Finding "nodes that can reach a cycle" is hard directly. But the reverse is easier:

> **Safe nodes are exactly those that can be reached from terminal nodes in the REVERSED graph.**

In the reversed graph:
- Terminal nodes (originally had no outgoing edges) now have no INCOMING edges → **indegree = 0 in reversed graph**
- Kahn's algorithm on the reversed graph → starts from these terminal nodes → expands to nodes that only lead to terminals → these are exactly the safe nodes

This reversal transforms a hard "which nodes escape cycles?" problem into a simple topological sort.

---

### Why Reversal Works

```
Original:  A → B → C (terminal)
Reversed:  A ← B ← C

In reversed graph:
C has indegree=0 → process C
B's indegree drops to 0 → process B
A's indegree drops to 0 → process A
All processed = all safe ✅

Original:  A → B → A (cycle), both connected to C (terminal)
Reversed:  A ← B ← A ← C

C has indegree=0 → process C
A gets one indegree decremented but still has indegree from B
B has indegree from A — never reaches 0
Both A and B stuck → never processed → unsafe ✅
```

The cycle means nodes in it can never reach indegree = 0 in the reversed graph — exactly as intended.

---

### Building the Reversed Graph

Original graph: `graph[i]` contains neighbors of `i` → edges are `i → neighbor`

Reversed: flip all edges → for each `i → neighbor`, add `neighbor → i`

```cpp
for(int i = 0; i < V; i++) {
    for(auto& it : graph[i]) {
        adjRev[it].push_back(i);   // reverse: it → i becomes it ← i
        indegree[i]++;             // i now has one incoming edge in original
    }
}
```

Wait — `indegree[i]++` tracks the **original outdegree** of node `i` (how many edges `i` sends out). In Kahn's on reversed graph, a node is "ready" when all its original outgoing edges have been "satisfied" — i.e., when all nodes it originally pointed to are confirmed safe.

---

## 3. Dry Run

```
graph = [[1,2],[2,3],[5],[0],[5],[],[]]
V = 7

Original edges:
0→1, 0→2, 1→2, 1→3, 2→5, 3→0, 4→5
```

**Build reversed graph + indegree:**

```
For node 0: neighbors 1,2
  adjRev[1].push_back(0),  indegree[0]++  (=1)
  adjRev[2].push_back(0),  indegree[0]++  (=2)

For node 1: neighbors 2,3
  adjRev[2].push_back(1),  indegree[1]++  (=1)
  adjRev[3].push_back(1),  indegree[1]++  (=2)

For node 2: neighbor 5
  adjRev[5].push_back(2),  indegree[2]++  (=1)

For node 3: neighbor 0
  adjRev[0].push_back(3),  indegree[3]++  (=1)

For node 4: neighbor 5
  adjRev[5].push_back(4),  indegree[4]++  (=1)

For node 5: no neighbors → nothing
For node 6: no neighbors → nothing

adjRev:
0 → [3]
1 → [0]
2 → [0, 1]
3 → [1]
4 → []
5 → [2, 4]
6 → []

indegree = [2, 2, 1, 1, 1, 0, 0]
           [0, 1, 2, 3, 4, 5, 6]
```

**Seed queue (indegree == 0):**
```
Node 5: indegree=0 ✅
Node 6: indegree=0 ✅
queue = [5, 6], safeNodes = []
```

**Pop 5:**
```
safeNodes = [5]
Neighbors in adjRev: [2, 4]
  indegree[2]-- → 0 ✅ → push 2
  indegree[4]-- → 0 ✅ → push 4
queue = [6, 2, 4]
```

**Pop 6:**
```
safeNodes = [5, 6]
Neighbors in adjRev: [] → nothing
queue = [2, 4]
```

**Pop 2:**
```
safeNodes = [5, 6, 2]
Neighbors in adjRev: [0, 1]
  indegree[0]-- → 1 (not 0 yet)
  indegree[1]-- → 1 (not 0 yet)
queue = [4]
```

**Pop 4:**
```
safeNodes = [5, 6, 2, 4]
Neighbors in adjRev: [] → nothing
queue = []  → BFS ends

Nodes 0, 1, 3 never processed → stuck in cycle → unsafe
```

**Sort safeNodes:**
```
[5, 6, 2, 4] → sorted → [2, 4, 5, 6] ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Unsafe = on a cycle OR can reach a cycle. Safe = everything else."**

Don't think of safe nodes as special — think of unsafe nodes. Unsafe nodes are cycle participants + anyone who can reach them. Safe nodes are simply the complement. This framing makes the reversal trick obvious: instead of tracing "who can reach a cycle?", we trace "who can be reached FROM terminals (in reverse)?".

---

**Story Point 2 — "indegree[i] here = original OUTDEGREE of node i"**

This is subtle. In the reversed graph, `indegree[i]` counts how many edges point INTO `i` in the reversed graph — which equals how many edges point OUT OF `i` in the original graph (the original outdegree).

```
Original: i → 1, i → 2, i → 5    (outdegree[i] = 3)
Reversed: i ← 1, i ← 2, i ← 5   (indegree_reversed[i] = 3)
```

A node with outdegree = 0 in original (terminal) has indegree = 0 in reversed → enters queue first. ✅

---

**Story Point 3 — "The sort at the end is O(N log N) — not free"**

The problem requires results in **sorted order**. Kahn's processes nodes in BFS order (not sorted). The final sort adds `O(V log V)`. This is worth noting because it's the dominant factor over `O(V + E)` when V is small.

---

**Story Point 4 — "This is Kahn's on a reversed graph — same code, different graph"**

The BFS loop is identical to standard Kahn's topological sort. The only structural change is:
- Build `adjRev` instead of `adjLs`
- `indegree[i]` counts original outdegree

Everything else — seeding queue, decrementing, pushing when indegree hits 0 — is unchanged.

---

**Story Point 5 — "Nodes that never enter the queue are cycle participants"**

After BFS, any node with `indegree > 0` is still "waiting" for some prerequisite that never comes — because it's part of a cycle where everything waits on everything else. This is the same cycle-detection guarantee from Kahn's: `safeNodes.size() < V` means some nodes are in cycles.

---

## 5. Code

```cpp
class Solution {
public:
    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int V = graph.size();

        // Step 1: Build reversed adjacency list + track original outdegree as indegree
        vector<vector<int>> adjRev(V);
        vector<int> indegree(V, 0);

        for(int i = 0; i < V; i++) {
            for(auto& it : graph[i]) {
                adjRev[it].push_back(i);   // reverse edge: it ← i
                indegree[i]++;             // i's original outdegree
            }
        }

        // Step 2: Seed queue with terminal nodes (original outdegree = 0)
        queue<int> q;
        for(int i = 0; i < V; i++) {
            if(indegree[i] == 0)
                q.push(i);
        }

        // Step 3: Kahn's BFS on reversed graph
        vector<int> safeNodes;
        while(!q.empty()) {
            int node = q.front();
            q.pop();

            safeNodes.push_back(node);   // this node is safe

            for(auto& neighbour : adjRev[node]) {
                indegree[neighbour]--;             // one safe neighbor confirmed
                if(indegree[neighbour] == 0)
                    q.push(neighbour);             // all neighbors safe → node is safe
            }
        }

        // Step 4: Sort and return (problem requires sorted output)
        sort(safeNodes.begin(), safeNodes.end());
        return safeNodes;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(V + E + V log V)`

| Step | Cost | Reason |
|---|---|---|
| Build reversed graph + indegree | `O(V + E)` | Outer loop over V nodes, inner loop total = E edges |
| Seed queue | `O(V)` | Scan all V nodes |
| Kahn's BFS | `O(V + E)` | Each node and each reversed edge processed once |
| Sort safeNodes | `O(V log V)` | At most V nodes in safeNodes |

**Total: `O(V + E + V log V)`**

> For dense graphs where `E >> V`, dominated by `O(E)`. For sparse graphs, dominated by `O(V log V)` sort.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjRev` reversed graph | `O(V + E)` | V lists + E reversed edges |
| `indegree[]` | `O(V)` | One per node |
| BFS queue | `O(V)` | At most V nodes |
| `safeNodes` | `O(V)` | At most all V nodes |

**Total: `O(V + E)`**
