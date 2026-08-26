# Articulation Points in a Graph

---

## 1. Problem Statement

Given an undirected graph with `V` vertices and edges, find all **articulation points** (also called **cut vertices**).

A vertex `u` is an articulation point if removing `u` (and all edges incident to it) **disconnects** the graph — increases the number of connected components.

```
Graph:
0—1—3
|/
2

Remove vertex 1: 0-2 remain connected, but 3 is isolated → 1 is ARTICULATION POINT
Remove vertex 0: 1-2-3 remain connected → 0 is NOT an articulation point
Remove vertex 3: 0-1-2 remain connected → 3 is NOT an articulation point

Answer: [1]
```

---

## 2. Background — `tin[]` and `low[]` (Same as Bridges)

**`tin[node]`** = discovery time of `node` (timestamp when DFS first visits it). Fixed after assignment.

**`low[node]`** = the **lowest `tin`** reachable from `node`'s subtree using at most one back edge. Starts at `tin[node]`, can only decrease.

The two are used identically to the Bridge-finding algorithm. The difference is in the **condition** used and the **handling of the root node**.

---

## 3. The Three Critical Cases — In Depth

---

### Case 1 — Non-Root Articulation Point

```cpp
if(low[adjNode] >= tin[node] && parent != -1) {
    st.insert(node);
}
```

#### Thought Process

When DFS is at `node` and has just returned from exploring child `adjNode`'s entire subtree:

**`low[adjNode]`** tells us: "The earliest ancestor that `adjNode`'s subtree can reach via any back edge."

**`tin[node]`** tells us: "When `node` was first discovered."

The condition `low[adjNode] >= tin[node]` means:

> `adjNode`'s subtree cannot reach any ancestor **higher than** (discovered before) `node`.

In other words: the ONLY way to get from `adjNode`'s subtree to any ancestor of `node` is by going **through `node` itself**.

If you remove `node`, `adjNode`'s subtree becomes disconnected from the rest of the graph → `node` is an articulation point.

---

#### Why `>=` and Not `>` (Unlike Bridges)?

For **bridges** the condition was `low[adjNode] > tin[node]` (strict greater).

For **articulation points** it's `low[adjNode] >= tin[node]` (includes equality).

**The difference:** When `low[adjNode] == tin[node]`, `adjNode`'s subtree can reach exactly `node` via a back edge, but NOT any ancestor of `node`. So:

- **Bridge case:** Even if `adjNode` can reach back to `node`, the direct edge `(node,adjNode)` is still a bridge if we need to TRAVERSE it (going from below to above)
- **Articulation point case:** If `adjNode` can only reach back to `node` (not higher), removing `node` still disconnects `adjNode`'s subtree — because the back edge leads to `node` which is being removed!

```
Example:
    0
   / \
  1   2
  |
  3 — 4
      |
      1 (back edge from 4 to 1!)

At node 1: low[3]=tin[3], low[4]=tin[1]
  low[subtree through 3,4] = tin[1]

Condition: low[child] >= tin[1]?
  low[3] = tin[3] > tin[1] → 1 is articulation (3's subtree can only reach 3)
  But from 4 there's a back edge to 1 (tin[1])
  low[4] = tin[1] = tin[node=1] → >= condition satisfied → 1 is still articulation!

If we remove 1: 3-4 subtree disconnects from 0-2 ✅
```

**Why this is correct:** The back edge from `4` goes TO `node=1`, which is being removed. So that back edge provides no alternate connectivity — `1` being an articulation point is still valid.

---

#### Example

```
Graph: 0—1—2—0 (triangle) with extra node 3 hanging off 1

   0
  / \
 1 — 2
 |
 3

DFS order: visit 0 (tin=1), visit 1 (tin=2), visit 2 (tin=3)
Back edge 2→0: low[2] = min(low[2], tin[0]) = min(3,1) = 1
Return to 1: low[1] = min(low[1], low[2]) = min(2,1) = 1

Visit 3 from 1 (tin=4, low=4, no back edges)
Return to 1: low[1] = min(low[1], low[3]) = min(1,4) = 1

Check at node 1:
  For child 2: low[2]=1 >= tin[1]=2? NO → not articulation from this child
  For child 3: low[3]=4 >= tin[1]=2? YES, parent≠-1 → 1 is ARTICULATION POINT ✅

If we remove 1: node 3 disconnects from 0-2 ✅
```

---

### Case 2 — Back Edge Handling: `low[node] = min(low[node], tin[adjNode])`

```cpp
} else {
    low[node] = min(low[node], tin[adjNode]);
}
```

This is the **back edge** case — `adjNode` is already visited (it's an ancestor in the DFS tree, not the parent).

#### Thought Process

A **back edge** connects `node` to an ancestor `adjNode` that was discovered BEFORE `node`. This creates an alternative path — instead of going up through the DFS tree one node at a time, `node` can "jump" all the way back to `adjNode`.

When we see this back edge, we update:
```
low[node] = min(low[node], tin[adjNode])
```

This tells `node`: "I can reach as far back in time as `adjNode`'s discovery time — I have a path that goes around the DFS tree back to `adjNode`."

---

#### Why `tin[adjNode]` Not `low[adjNode]`?

This is the key difference from how we update `low` for tree edges (where we use `low[adjNode]`).

For a back edge, `adjNode` is an **ancestor** — already fully processed in the sense that it's on the current DFS stack. We know exactly where it is in time: `tin[adjNode]`.

Using `low[adjNode]` would be wrong because:
- `low[adjNode]` could be even lower than `tin[adjNode]` (if `adjNode` itself has back edges to ancestors)
- But `node`'s back edge only reaches `adjNode` directly — it doesn't automatically grant access to `adjNode`'s own back edges' targets

```
Counterexample of using low[adjNode] for back edges:

DFS tree: 0→1→2→3, back edge 3→0

At node 3: back edge to 0
  tin[0]=1, low[0]=1 (no back edges from 0 itself)
  → low[3] = min(low[3], tin[0]) = 1 ✅ (correct — can reach as far back as 0)

Now: if 0 also had a back edge to some node -1 (hypothetically):
  low[0] might be 0
  Using low[0]: low[3] would become 0 ← INCORRECT
  The path 3→0→-1 is not a single back edge, it's two edges
  Using tin[0] correctly captures: node 3 can reach ONLY to node 0 via this back edge
```

The `tin` value precisely represents "this ancestor's position in DFS discovery order" — using it for back edges is the correct interpretation.

---

#### Why `low` Propagates Upward via Tree Edges

After using `low[adjNode]` for tree edges:
```cpp
low[node] = min(low[node], low[adjNode]);
```

This is different from back edges. Here, `adjNode` is a child in the DFS tree that we just finished exploring. `low[adjNode]` already incorporates ALL back edges from `adjNode`'s entire subtree. So if `adjNode`'s subtree can reach back to an ancestor (say, discovery time 1), then `node` can ALSO reach back to that ancestor through the path `node → adjNode → (subtree back edge) → ancestor`.

**Summary:**
- **Tree edge to child:** `low[node] = min(low[node], low[adjNode])` — inherit child's full reach
- **Back edge to ancestor:** `low[node] = min(low[node], tin[adjNode])` — reach only that ancestor directly

---

#### Example

```
Graph: 0—1—2—3, back edge 3→1

DFS: 0(tin=1) → 1(tin=2) → 2(tin=3) → 3(tin=4)
  At 3: back edge to 1
    low[3] = min(low[3]=4, tin[1]=2) = 2

  Return to 2: low[2] = min(low[2]=3, low[3]=2) = 2
  Return to 1: low[1] = min(low[1]=2, low[2]=2) = 2
  Return to 0: low[0] = min(low[0]=1, low[1]=2) = 1

Final low = [1, 2, 2, 2]
```

Node 3's back edge to node 1 (tin=2) propagates upward:
- low[3]=2, low[2]=2 (inherited from 3), low[1]=2 (already 2)

Edge (0,1): low[1]=2 >= tin[0]=1? YES → 0 is articulation (but wait, check parent=-1 condition)
Edge (1,2): low[2]=2 >= tin[1]=2? YES → 1 is articulation ✅ (removing 1 disconnects 0 from 2-3)

---

### Case 3 — Root Node Special Case

```cpp
if(childs > 1 && parent == -1) {
    st.insert(node);
}
```

#### Thought Process

The root of the DFS tree (the first node DFS starts from) is a **special case** that the standard `low[adjNode] >= tin[node]` condition does NOT handle correctly.

**Why the standard condition fails for root:**

The standard condition checks: "can `adjNode`'s subtree reach an ancestor of `node`?"

For the ROOT, there IS no ancestor. Every `low[child]` will be `>= tin[root]` trivially — because `tin[root]` is the minimum discovery time (1), and `low[child]` can't go below `tin[root]`.

If we applied the standard condition to root, we'd say root is an articulation point for EVERY child — even when it's not!

```
Graph: 0—1—2 (a path, 0 is root)

tin[0]=1, low[1]=2, low[2]=3

Standard condition at 0 for child 1: low[1]=2 >= tin[0]=1? YES
→ Would wrongly add 0 as articulation point!

But removing 0 just gives us 1—2 (still connected) → 0 is NOT an articulation point here!
```

**The actual rule for root:**

Root is an articulation point if and only if it has **more than one child in the DFS tree**.

Why? If root has:
- **1 child:** The entire graph is connected through that single child subtree. Remove root → child subtree still connected. Root is NOT articulation.
- **2+ children:** Each child's subtree is only connected TO EACH OTHER via the root. Remove root → child subtrees become disconnected. Root IS articulation.

```
Graph: Star with center 0:
  0—1, 0—2, 0—3

DFS from 0: visits 1, 2, 3 as separate children (no edges between 1,2,3)
  childs = 3 (each is an independent DFS subtree from root)
  childs > 1 → 0 is ARTICULATION POINT ✅

Removing 0: 1, 2, 3 all disconnected ✅

Graph: Path 0—1—2—3—4
DFS from 0: visits 1 (only child of 0)
  childs = 1
  childs > 1? NO → 0 is NOT articulation point ✅
```

---

#### Why Count Only Unvisited Neighbors as `childs`?

```cpp
if(!vist[adjNode]) {
    dfs(adjNode, ...);
    childs++;   // ← only count tree edges (new discoveries)
}
```

`childs` counts **DFS tree children** — nodes discovered BY root for the first time. Already-visited neighbors (back edges) are NOT counted as children because they don't represent new branches in the DFS tree.

```
Graph: 0—1, 0—2, 1—2 (triangle, 0 is root)

DFS from 0:
  Visit 1 (child 1), childs=1
    From 1: neighbor 2 (new) → visit 2
      From 2: neighbor 0 (visited, not parent) → back edge
    Return to 1
  Return to 0
  Neighbor 2 from 0: already visited! → not a new child
  childs = 1

childs > 1? NO → 0 is NOT articulation ✅ (removing 0 leaves 1-2 connected)
```

If we counted back edges as children: childs=2 → wrongly say 0 is articulation.

---

## 4. Full Dry Run

```
V=5, Edges: 1-0, 0-2, 2-1, 0-3, 3-4

Graph:
  1—0—3—4
  |/
  2

adjLs:
0 → [1,2,3]
1 → [0,2]
2 → [0,1]
3 → [0,4]
4 → [3]

DFS from 0 (parent=-1), timer starts at 1
```

**dfs(0, -1):**
```
visited[0]=1, tin[0]=low[0]=1, timer=2, childs=0

Neighbor 1 (unvisited):
  dfs(1, 0):
    visited[1]=1, tin[1]=low[1]=2, timer=3

    Neighbor 0: parent! SKIP

    Neighbor 2 (unvisited):
      dfs(2, 1):
        visited[2]=1, tin[2]=low[2]=3, timer=4

        Neighbor 0 (visited, not parent=1):
          low[2] = min(3, tin[0]) = min(3,1) = 1    ← Case 2 (back edge)

        Neighbor 1 (parent!) SKIP

        Return dfs(2)

      Back in dfs(1):
      low[1] = min(low[1]=2, low[2]=1) = 1          ← tree edge propagation

      Check Case 1: low[2]=1 >= tin[1]=2? NO → 1 not articulation from child 2

    Return dfs(1)

  Back in dfs(0):
  low[0] = min(1, low[1]=1) = 1
  childs++ = 1
  Check Case 1: low[1]=1 >= tin[0]=1? YES, but parent==-1 → SKIP (Case 3 handles root)

Neighbor 2 (VISITED, not parent):
  low[0] = min(low[0]=1, tin[2]=3) = 1              ← Case 2 (back edge to visited node)
  (no change)

Neighbor 3 (unvisited):
  dfs(3, 0):
    visited[3]=1, tin[3]=low[3]=5, timer=6

    Neighbor 0 (parent!) SKIP

    Neighbor 4 (unvisited):
      dfs(4, 3):
        visited[4]=1, tin[4]=low[4]=6, timer=7

        Neighbor 3 (parent!) SKIP

        Return dfs(4)

      Back in dfs(3):
      low[3] = min(low[3]=5, low[4]=6) = 5

      Check Case 1: low[4]=6 >= tin[3]=5? YES, parent≠-1 → ADD 3 to st ✅

    Return dfs(3)

  Back in dfs(0):
  low[0] = min(1, low[3]=5) = 1
  childs++ = 2
  Check Case 1: low[3]=5 >= tin[0]=1? YES, but parent==-1 → SKIP

Case 3: childs=2 > 1, parent==-1 → ADD 0 to st ✅
```

**Final:** Articulation points = {0, 3}

**Verification:**
- Remove 0: {1,2} disconnected from {3,4} ✅
- Remove 3: {0,1,2} disconnected from {4} ✅

---

## 5. Code

```cpp
class Solution {
    int timer = 1;

private:
    void dfs(int node, int parent, vector<int>& vist, vector<int>& tin,
             vector<int>& low, set<int>& st, vector<int> adjLs[]) {
        vist[node] = 1;
        low[node] = tin[node] = timer++;

        int childs = 0;

        for(auto adjNode : adjLs[node]) {
            if(adjNode == parent) continue;

            if(!vist[adjNode]) {
                // TREE EDGE: recurse into unvisited child
                dfs(adjNode, node, vist, tin, low, st, adjLs);

                // Propagate child's reach upward
                low[node] = min(low[node], low[adjNode]);

                // CASE 1: Non-root articulation point check
                if(low[adjNode] >= tin[node] && parent != -1)
                    st.insert(node);

                childs++;   // count only tree children

            } else {
                // CASE 2: BACK EDGE — reached an ancestor
                low[node] = min(low[node], tin[adjNode]);
            }
        }

        // CASE 3: Root special case
        if(childs > 1 && parent == -1)
            st.insert(node);
    }

public:
    vector<int> articulationPoints(int V, vector<vector<int>>& edges) {
        vector<int> adjLs[V];
        for(auto& it : edges) {
            adjLs[it[0]].push_back(it[1]);
            adjLs[it[1]].push_back(it[0]);
        }

        vector<int> vist(V, 0), tin(V, 0), low(V, 0);
        set<int> st;

        for(int i = 0; i < V; i++) {
            if(!vist[i])
                dfs(i, -1, vist, tin, low, st, adjLs);
        }

        vector<int> answer(st.begin(), st.end());
        return answer.empty() ? vector<int>{-1} : answer;
    }
};
```

---

## 6. Bridges vs Articulation Points — Side by Side

| | Bridges | Articulation Points |
|---|---|---|
| **What we find** | Critical EDGES | Critical VERTICES |
| **Condition** | `low[child] > tin[node]` (strict >) | `low[child] >= tin[node]` (>= includes equal) |
| **Root special case** | None | `childs > 1` check |
| **Back edge update** | `low[node] = min(low[node], low[adjNode])` OR `tin[adjNode]` | `low[node] = min(low[node], tin[adjNode])` (strictly `tin`) |
| **Use `set`?** | No (list is fine) | Yes (same node can be added multiple times from different children) |
| **Why set?** | N/A | `node` is checked once per child — might satisfy condition for multiple children → `set` deduplicates |

---

## 7. Complexity Analysis

### Time Complexity — `O(V + E)`

DFS visits each node once and each edge twice (undirected) → `O(V + E)`.
Set insertions: `O(log V)` per insertion, at most `V` insertions → `O(V log V)`.

**Total: `O(V + E + V log V)` = `O((V + E) log V)` dominated by sort in set for dense graphs, but practically `O(V + E)`.**

### Space Complexity — `O(V + E)`

| Structure | Size |
|---|---|
| Adjacency list | `O(V + 2E)` |
| `vist[]`, `tin[]`, `low[]` | `O(V)` each |
| DFS call stack | `O(V)` worst case |
| Set `st` | `O(V)` |
