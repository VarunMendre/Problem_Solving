# Kosaraju's Algorithm — Strongly Connected Components (SCC)

---

## 1. Problem Statement

Given a **directed graph** with `V` vertices and edges, find the number of **Strongly Connected Components (SCCs)**.

A **Strongly Connected Component** is a maximal group of vertices such that there is a path from every vertex to every other vertex **in both directions**.

```
Graph:
0→1→2→0  (cycle)
2→3
3→4→5→3  (another cycle)

SCCs:
SCC 1: {0,1,2}   (all can reach each other)
SCC 2: {3,4,5}   (all can reach each other)

Node 2 can reach node 3, but node 3 CANNOT reach node 2
→ they are in different SCCs

Answer: 2 SCCs
```

---

## 2. What is an SCC? — Deep Intuition

### The Reachability Property

In a directed graph, "A can reach B" does NOT imply "B can reach A". An SCC is a maximal set where:
- Every node can reach every other node
- The group is as large as possible (can't add any more nodes while maintaining the property)

### Think of it as: "Who's in the same loop?"

```
0 → 1 → 2
↑         ↓
←←←←←←←←←

These 3 form an SCC — they're all in the same "loop" (cycle).
```

If you start at ANY node in an SCC and follow directed edges, you can return to where you started. An SCC is exactly a maximal group where this loop property holds.

---

## 3. Why Kosaraju's Algorithm — The Core Insight

### The Problem with Naive Approach

For each node, run DFS to find all reachable nodes, then run DFS on the reversed graph to find all nodes that can reach it. Their intersection = the SCC of that node.

This is `O(V × (V+E))` — too slow for large graphs.

---

### Kosaraju's Brilliant Observation

**Key insight:** In a **transposed graph** (all edges reversed), the SCCs remain the same, but the connections BETWEEN SCCs are reversed.

```
Original:     SCC_A → SCC_B → SCC_C    (edges between SCCs)
Transposed:   SCC_A ← SCC_B ← SCC_C   (reversed inter-SCC edges)

SCCs themselves: unchanged (internal edges all reversed = still a cycle)
```

**Now the magic:**

If we run DFS on the original graph and track when we FINISH exploring each component (post-order), the SCC that has NO outgoing edges to other SCCs finishes LAST.

In the transposed graph, this "sink SCC" (no outgoing edges in original = no incoming edges in transposed) becomes an SCC that can't reach ANY other SCC.

So: **process nodes in reverse finish order on the transposed graph** → each DFS from an unvisited node explores exactly one SCC.

---

### Why "Reverse Finish Order"?

Let's think about what finish order tells us:

```
Original graph (between SCCs):
SCC_A → SCC_B → SCC_C

DFS from SCC_A: explores A, then B (through A→B edge), then C (through B→C edge)
Finish order: C finishes first, then B, then A

Reverse finish order: A first, then B, then C
```

After reversing edges (transpose):
```
Transposed: SCC_A ← SCC_B ← SCC_C
```

Processing in reverse finish order (A, B, C):
- DFS from A (in transposed): A can't reach B or C (edges reversed) → finds SCC_A
- DFS from B (in transposed): B can't reach C → finds SCC_B
- DFS from C (in transposed): finds SCC_C

Each DFS call on the transposed graph, starting from the right node, finds exactly one SCC!

---

## 4. The Three Steps of Kosaraju's

### Step 1 — DFS on Original Graph: Track Finish Order

```cpp
void dfs(int node, vector<int>& visited, stack<int>& finishOrder, ...) {
    visited[node] = 1;
    for(int neighbor : adjList[node]) {
        if(!visited[neighbor])
            dfs(neighbor, visited, finishOrder, adjList);
    }
    finishOrder.push(node);   // ← push AFTER all neighbors explored
}
```

A node is pushed to `finishOrder` only AFTER its entire reachable subgraph is explored. This gives **topological order** of SCCs — the SCC that nothing else depends on finishes first (pushed first → at bottom of stack). The independent "source" SCC finishes last → at top of stack.

**`finishOrder` stack:** pop order = reverse finish order = source SCCs first.

---

### Step 2 — Build Transpose Graph

```cpp
for(int node = 0; node < V; node++) {
    for(int neighbor : adjList[node]) {
        transposeGraph[neighbor].push_back(node);   // reverse every edge
    }
}
```

Every edge `u → v` becomes `v → u`. SCCs are preserved internally (cycle reversed = still a cycle). Inter-SCC edges are reversed — sink SCCs become source SCCs.

---

### Step 3 — DFS on Transposed Graph in Reverse Finish Order

```cpp
while(!finishOrder.empty()) {
    int node = finishOrder.top();
    finishOrder.pop();

    if(!visited[node]) {
        dfsTranspose(node, visited, transposeGraph);
        sccCount++;   // each new unvisited node starts a new SCC
    }
}
```

Process nodes from `finishOrder` (which gives source SCCs first in transposed graph). Each DFS on the transposed graph from an unvisited source SCC explores ONLY that SCC — because inter-SCC edges are now reversed (pointing toward the source, not away).

---

## 5. Dry Run

```
V=6
Edges: 0→1, 1→2, 2→0, 1→3, 3→4, 4→5, 5→3

Graph:
0→1→2→0   (triangle: SCC {0,1,2})
    ↓
    3→4→5→3   (triangle: SCC {3,4,5})

adjList:
0 → [1]
1 → [2, 3]
2 → [0]
3 → [4]
4 → [5]
5 → [3]
```

---

### Step 1 — DFS + Finish Order

**dfs(0):**
```
visit 0 → visit 1 → visit 2 → neighbor 0 (visited, skip) → push 2
           → visit 3 → visit 4 → visit 5 → neighbor 3 (visited, skip) → push 5
           → push 4
           → push 3
           → push 1
→ push 0

finishOrder stack (bottom→top): 2, 5, 4, 3, 1, 0
Pop order (top first): 0, 1, 3, 4, 5, 2
```

---

### Step 2 — Transpose Graph

```
Original edges reversed:
0←1, 1←2, 2←0, 3←1, 4←3, 5←4, 3←5

transposeGraph:
0 → [2]
1 → [0]
2 → [1]
3 → [1, 5]
4 → [3]
5 → [4]

SCCs still {0,1,2} and {3,4,5} internally:
0→2→1→0 (still a cycle) ✅
3→5→4→3 (still a cycle) ✅
Inter-SCC edge 1→3 reversed to 3←1... wait:
Original: 1→3, transpose makes 3→1... 
So in transpose: 3 has edge to 1? Yes → SCC{3,4,5} can reach SCC{0,1,2} in transpose
But SCC{0,1,2} cannot reach SCC{3,4,5} in transpose → DFS from {0,1,2} nodes stays within ✅
```

---

### Step 3 — DFS on Transpose in Pop Order

**Reset visited = [0,0,0,0,0,0]**

**Pop 0 (top of stack):**
```
visited[0]=0 → new SCC! sccCount=1

dfsTranspose(0):
  visit 0 → neighbor 2 → dfsTranspose(2):
    visit 2 → neighbor 1 → dfsTranspose(1):
      visit 1 → neighbor 0 (visited, skip)
  (SCC {0,1,2} explored)
```

**Pop 1:** visited[1]=1 → skip

**Pop 3:**
```
visited[3]=0 → new SCC! sccCount=2

dfsTranspose(3):
  visit 3 → neighbors: 1(visited,skip), 5
    dfsTranspose(5):
      visit 5 → neighbor 4
        dfsTranspose(4):
          visit 4 → neighbor 3 (visited, skip)
  (SCC {3,4,5} explored)
```

**Pop 4:** visited[4]=1 → skip
**Pop 5:** visited[5]=1 → skip
**Pop 2:** visited[2]=1 → skip

**sccCount = 2 ✅**

---

## 6. Story Points

---

**Story Point 1 — "Finish order = topological order of SCCs"**

The DFS finish order sorts SCCs such that SCCs that are "downstream" (sinks — nothing else depends on them) finish first. Reversing this gives "upstream" (source) SCCs first.

In the transposed graph, source SCCs become sinks (no outgoing inter-SCC edges). A DFS from a source SCC in the transposed graph CANNOT leave that SCC → explores exactly one SCC.

---

**Story Point 2 — "Why transposing preserves SCCs"**

Within an SCC, if `A→B` and `B→C→...→A` form a cycle, reversing all edges gives `B→A` and `A→...→C→B` — still a cycle, just in reverse direction. The SCC is preserved.

Between SCCs (say SCC_X → SCC_Y in original): after transpose, SCC_X ← SCC_Y. This prevents DFS on the transposed graph from "leaking" out of SCC_X into SCC_Y when starting from SCC_X — because the arrow now points toward SCC_X, not away.

---

**Story Point 3 — "Why push to `finishOrder` AFTER exploring all neighbors"**

A node pushed to the stack AFTER its entire subtree = post-order = the "last to finish" is the one with the most connections (closest to source in topological order of SCCs).

If we pushed before exploring (pre-order), finish order would be meaningless — we'd get discovery order, not finish order.

---

**Story Point 4 — "The outer loop handles disconnected graphs"**

```cpp
for(int node = 0; node < V; node++) {
    if(!visited[node])
        dfs(node, visited, finishOrder, adjList);
}
```

A single DFS call might not visit all nodes (if graph is disconnected or some nodes unreachable). The outer loop ensures every node gets into `finishOrder` exactly once, regardless of connectivity.

---

**Story Point 5 — "Why we need two separate `visited` arrays (resetting between Steps 1 and 3)"**

```cpp
fill(visited.begin(), visited.end(), 0);   // reset before Step 3
```

After Step 1, all nodes are marked visited. If we don't reset, Step 3's DFS would immediately skip every node → sccCount=0 (wrong).

The reset is essential: Step 1's visited ensures we don't revisit during original DFS (correct finish order). Step 3's visited ensures we don't revisit during transposed DFS (each SCC counted once).

---

## 7. Code

```cpp
class Solution {
private:
    // Step 1 DFS: track finish order
    void dfs(int node, vector<int>& visited, stack<int>& finishOrder,
             vector<vector<int>>& adjList) {
        visited[node] = 1;
        for(int neighbor : adjList[node]) {
            if(!visited[neighbor])
                dfs(neighbor, visited, finishOrder, adjList);
        }
        finishOrder.push(node);   // push AFTER exploring all reachable nodes
    }

    // Step 3 DFS: explore SCC on transposed graph
    void dfsTranspose(int node, vector<int>& visited,
                      vector<vector<int>>& transposeGraph) {
        visited[node] = 1;
        for(int neighbor : transposeGraph[node]) {
            if(!visited[neighbor])
                dfsTranspose(neighbor, visited, transposeGraph);
        }
        // No finishOrder needed — just marking the SCC
    }

public:
    int kosaraju(int V, vector<vector<int>>& edges) {
        // Build original directed graph
        vector<vector<int>> adjList(V);
        for(auto& edge : edges) {
            adjList[edge[0]].push_back(edge[1]);
        }

        // Step 1: DFS to get finish order
        vector<int> visited(V, 0);
        stack<int> finishOrder;
        for(int node = 0; node < V; node++) {
            if(!visited[node])
                dfs(node, visited, finishOrder, adjList);
        }

        // Step 2: Build transpose (reverse all edges)
        vector<vector<int>> transposeGraph(V);
        for(int node = 0; node < V; node++) {
            for(int neighbor : adjList[node]) {
                transposeGraph[neighbor].push_back(node);
            }
        }

        // Step 3: DFS on transposed graph in reverse finish order
        fill(visited.begin(), visited.end(), 0);   // RESET visited!
        int sccCount = 0;

        while(!finishOrder.empty()) {
            int node = finishOrder.top();
            finishOrder.pop();

            if(!visited[node]) {
                dfsTranspose(node, visited, transposeGraph);
                sccCount++;   // each new DFS = one new SCC
            }
        }

        return sccCount;
    }
};
```

---

## 8. Complexity Analysis

### Time Complexity — `O(V + E)`

| Step | Cost | Reason |
|---|---|---|
| Build adjList | `O(V + E)` | V lists + E edges |
| Step 1: DFS all nodes | `O(V + E)` | Each node and edge visited once |
| Step 2: Build transpose | `O(V + E)` | Reverse every edge |
| Step 3: DFS on transpose | `O(V + E)` | Each node and edge visited once |

**Total: `O(V + E)`**

---

### Space Complexity — `O(V + E)`

| Structure | Size |
|---|---|
| `adjList` | `O(V + E)` |
| `transposeGraph` | `O(V + E)` |
| `visited[]` | `O(V)` |
| `finishOrder` stack | `O(V)` |
| Recursion call stack | `O(V)` worst case |

**Total: `O(V + E)`**

---

## 9. Tarjan's vs Kosaraju's

Both find SCCs, but differently:

| | Kosaraju's | Tarjan's |
|---|---|---|
| **Passes** | 2 DFS passes | 1 DFS pass |
| **Extra structure** | Transpose graph + stack | `low[]`, `tin[]`, SCC stack |
| **TC** | `O(V + E)` | `O(V + E)` |
| **SC** | `O(V + E)` (two graphs) | `O(V)` |
| **Conceptual simplicity** | ✅ Easier to understand | More complex |
| **Memory** | More (stores transpose) | Less |
| **Real-world preference** | Education / interviews | Production code |

> Kosaraju's: "find finish order on original, then traverse transposed in that order."
> Tarjan's: "single DFS, use low-link values to detect cycle completion."
