# Disjoint Set Union (DSU) — Union Find

---

## 1. What is a Disjoint Set?

A **Disjoint Set** (also called **Union-Find**) is a data structure that keeps track of elements partitioned into a number of **non-overlapping (disjoint) groups/components**.

"Disjoint" means the groups have **no element in common** — every element belongs to exactly one group.

```
Initial (7 elements, each in its own group):
{1} {2} {3} {4} {5} {6} {7}

After some unions:
{1, 2, 3}  {4, 5, 6, 7}

All 7 elements, 2 groups, no element appears in both. This is "Disjoint".
```

Each group has a **representative** (also called **root** or **parent**) — a single element that identifies the group. Any element in the group can find its representative by following parent pointers.

---

## 2. Why Is It Used? Where Is It Used?

### The Core Operation

DSU efficiently answers two questions:

1. **Find:** "Which component does element `x` belong to?" (Who is `x`'s representative?)
2. **Union:** "Merge the components containing `x` and `y`."

### Where It's Used

| Application | How DSU Helps |
|---|---|
| **Kruskal's MST** | Check if two nodes are already connected (adding edge would create cycle) |
| **Network connectivity** | Are two computers in the same network? |
| **Image processing** | Group pixels into connected regions |
| **Percolation problems** | Does water flow from top to bottom? |
| **Social networks** | Are two people in the same friend group? |
| **Dynamic graph connectivity** | Handle edge additions and connectivity queries efficiently |

---

## 3. What Problem Does DSU Solve? (vs Brute Force)

### The Problem: Connectivity Queries

Given a graph that grows over time (edges are added), repeatedly answer: **"Are nodes `u` and `v` in the same connected component?"**

---

### Brute Force Approach — BFS/DFS

For each connectivity query, run BFS/DFS from `u` and check if `v` is reachable.

```
Time per query: O(V + E)
For Q queries: O(Q × (V + E))

Example: 10,000 queries on a graph with 10,000 nodes and 50,000 edges:
= 10,000 × 60,000 = 600,000,000 operations → TOO SLOW
```

Every query re-explores the entire graph from scratch — no information is reused between queries.

---

### DSU Approach

After each edge addition (union operation), DSU updates its internal structure. Each connectivity query is then nearly `O(1)`.

```
Time per union:  O(α(V)) ≈ O(1) amortized (α = inverse Ackermann function)
Time per query:  O(α(V)) ≈ O(1) amortized
For Q queries:   O(Q × α(V)) ≈ O(Q)

Same 10,000 queries: ≈ 10,000 operations → EXTREMELY FAST
```

**DSU is a 600,000× speedup** for this example.

---

## 4. What Are Dynamic Graphs?

A **static graph** is given all at once — you process it and you're done.

A **dynamic graph** changes over time:
- Edges/nodes are **added** while you're processing
- Between additions, you answer queries ("are u and v connected?")

```
Example: Chat application
- Users join (nodes added)
- Friendships form (edges added)
- Query: "Are Alice and Bob in the same friend group?"
→ This happens millions of times as the graph grows
```

---

### Why Dynamic Graphs Need DSU

For a static graph, you can precompute all connected components once with BFS (`O(V+E)`). But with a dynamic graph:
- Running BFS after every edge addition: `O(new_edges × (V+E))` — too slow
- DSU updates the component structure incrementally with each edge addition: `O(α(V))` per addition

DSU is essentially a **lazy, incremental connected-component tracker** — perfect for dynamic graphs.

---

## 5. The Three Core Operations

### i. `findParent(node)` — Find the Representative

**What:** Find the root/representative of the component containing `node`.

**Why:** Two nodes are in the same component if and only if they have the **same representative**.

```
Check if u and v are connected:
  if findParent(u) == findParent(v) → same component ✅
  else                              → different components ❌
```

**How (naïve):** Each node has a `parent[]`. Repeatedly follow parent pointers until you reach a node that is its own parent (the root).

```
parent = [0:0, 1:1, 2:1, 3:2, 4:4, 5:4]
findParent(3):
  3 → parent[3]=2 → parent[2]=1 → parent[1]=1 (root!) → return 1
```

**Problem with naïve:** In the worst case (a long chain), this takes `O(n)` per call → `O(n)` per query → same as BFS.

**Solution:** Path compression (explained in Section 8).

---

### ii. `unionByRank(u, v)` — Merge by Tree Height

**What:** Merge the components of `u` and `v`. Attach the **shorter tree under the taller tree**.

**Why rank?** "Rank" ≈ approximate height of the tree. When merging, we always attach the shallower tree under the deeper one to keep the overall tree height small.

```
Rank controls tree height → small height → fast findParent
```

**The three cases:**
```
Case 1: rank[rootU] < rank[rootV]
  → attach rootU under rootV (rootU becomes child)
  → rootV's rank stays the same (tree height unchanged)

Case 2: rank[rootU] > rank[rootV]
  → attach rootV under rootU
  → rootU's rank stays the same

Case 3: rank[rootU] == rank[rootV]
  → attach either (we choose rootV under rootU)
  → rootU's rank increases by 1 (tree got taller by one level)
```

Rank only increases when two equal-height trees merge → merges are infrequent → rank stays small.

---

### iii. `unionBySize(u, v)` — Merge by Component Size

**What:** Merge by attaching the **smaller component under the larger component**.

**Why size instead of rank?** Size is the **exact count of nodes** in a component — a more precise metric than approximate height. The logic is the same: keep the "center of gravity" in the larger component.

```
size[root] = total nodes in that component

Merge: always attach smaller → larger
  → larger component's root becomes the root of the merged component
  → update size: size[larger] += size[smaller]
```

**Why smaller under larger?**

Attaching a large tree as a subtree of a small tree creates a very tall structure. For example:

```
Small tree (root A, size 2):    Large tree (root B, size 100):
    A                                   B
    |                               (100 nodes below)
    x

If we attach B under A:          If we attach A under B:
      A                                  B
     / \                              (100 nodes) ─── A ─── x
    x   B                         
      (100 nodes)

Height = 100+                    Height = 3 (much shorter!)
```

Smaller under larger keeps trees balanced → shorter height → faster find.

---

## 6. Dry Run with Visualization

```
Operations: unionBySize(1,2), unionBySize(2,3), unionBySize(4,5),
            unionBySize(6,7), unionBySize(5,6)

Initial state (n=7):
parent = [0,1,2,3,4,5,6,7]  (each node is its own parent, 1-indexed)
size   = [0,1,1,1,1,1,1,1]  (each component has 1 node)

Visualization:
1  2  3  4  5  6  7   (7 separate components)
```

---

**`unionBySize(1, 2)`:**
```
findUParent(1) = 1
findUParent(2) = 2
→ Different roots. size[1]=1, size[2]=1 → equal → attach 2 under 1
parent[2] = 1, size[1] += size[2] = 2

parent = [0, 1, 1, 3, 4, 5, 6, 7]
size   = [0, 2, 1, 1, 1, 1, 1, 1]

Visualization:
  1       3  4  5  6  7
  |
  2
```

---

**`unionBySize(2, 3)`:**
```
findUParent(2) → parent[2]=1, parent[1]=1 → root=1
findUParent(3) → root=3
→ size[1]=2 > size[3]=1 → attach 3 under 1
parent[3] = 1, size[1] += size[3] = 3

parent = [0, 1, 1, 1, 4, 5, 6, 7]
size   = [0, 3, 1, 1, 1, 1, 1, 1]

Visualization:
    1         4  5  6  7
   /|\
  2   3
```

---

**`unionBySize(4, 5)`:**
```
findUParent(4) = 4, findUParent(5) = 5
→ size[4]=size[5]=1 → attach 5 under 4
parent[5] = 4, size[4] = 2

Visualization:
    1       4     6  7
   / \      |
  2   3     5
```

---

**`unionBySize(6, 7)`:**
```
findUParent(6) = 6, findUParent(7) = 7
→ equal size → attach 7 under 6
parent[7] = 6, size[6] = 2

Visualization:
    1       4       6
   / \      |       |
  2   3     5       7
```

---

**`unionBySize(5, 6)`:**
```
findUParent(5): parent[5]=4, parent[4]=4 → root=4
findUParent(6): parent[6]=6 → root=6

size[4]=2, size[6]=2 → equal → attach 6 under 4
parent[6] = 4, size[4] += size[6] = 4

parent = [0, 1, 1, 1, 4, 4, 4, 6]
size   = [0, 3, 1, 1, 4, 1, 2, 1]

Visualization:
    1           4
   / \         /|\
  2   3       5  6  (6 is now under 4)
                 |
                 7

Two components: {1,2,3} and {4,5,6,7}
```

---

**`sameComponent(3, 7)`:**
```
findUParent(3): parent[3]=1, parent[1]=1 → root=1
findUParent(7): parent[7]=6, parent[6]=4, parent[4]=4 → root=4
1 ≠ 4 → "no" ✅
```

---

**`unionBySize(3, 7)`:**
```
roots: 1 and 4
size[1]=3, size[4]=4 → size[4] > size[1] → attach 1 under 4
parent[1] = 4, size[4] += size[1] = 7

Final tree:
            4
          / | \ \
         5  6  1  ← entire {1,2,3} subtree moved under 4
            |  |\ 
            7  2  3

All 7 nodes in one component!
```

**`sameComponent(3, 7)`:**
```
findUParent(3): 3→1→4 → root=4
findUParent(7): 7→6→4 → root=4
4 == 4 → "yes" ✅
```

**`findUParent(2)`:**
```
2→1→4 → root=4 ✅
```

---

## 7. Time Complexity Analysis

### Naïve `findParent` (without path compression): `O(n)`

In the worst case (degenerate chain: 1→2→3→...→n):
```
findParent(1) follows n pointers → O(n)
```

### With Union by Rank/Size: `O(log n)`

Union by rank/size keeps tree height bounded at `O(log n)`. So `findParent` traverses at most `O(log n)` pointers.

**Why `O(log n)` height?**

With union by size: when a node's depth increases (gets put deeper), its component size at least doubles (we attached it under a component at least as large). Starting from size 1, you can only double `log₂(n)` times before reaching size `n`. So depth ≤ `log₂(n)`.

### With Path Compression + Union by Rank/Size: `O(α(n))`

`α(n)` = inverse Ackermann function. Grows INCREDIBLY slowly:
- `α(1) = 0`
- `α(4) = 1`
- `α(2^65536) = 4`

For any practical input, `α(n) ≤ 4`. Essentially `O(1)` per operation.

### Summary Table

| Method | TC per operation |
|---|---|
| Naïve find (no optimization) | `O(n)` |
| Union by Rank/Size only | `O(log n)` |
| Path Compression only | `O(log n)` amortized |
| **Both (Rank/Size + Compression)** | **`O(α(n))` ≈ `O(1)`** |

---

## 8. Path Compression — In Depth

### The Problem Without Path Compression

After several unions, a long chain can form:

```
1 → 2 → 3 → 4 → 5 (root)

findParent(1): 1→2→3→4→5 → 4 pointer hops
findParent(1) again: 1→2→3→4→5 → same 4 hops again!
```

Every `find` call re-traverses the entire chain. No learning from previous calls.

---

### The Path Compression Idea

**"After finding the root, make every node on the path point DIRECTLY to the root."**

```
Before:
1 → 2 → 3 → 4 → 5 (root)

After findParent(1) with path compression:
All of 1, 2, 3, 4 now point directly to 5:

1 → 5
2 → 5
3 → 5
4 → 5
5 (root)

Next findParent(1): 1→5 → just 1 hop! ✅
Next findParent(2): 2→5 → just 1 hop! ✅
```

---

### How It's Implemented

```cpp
int findUParent(int node) {
    if(node == parent[node])
        return node;   // reached root → return

    // Recursive call finds root, THEN on the way back,
    // sets parent[node] = root (path compression)
    return parent[node] = findUParent(parent[node]);
}
```

The magic is in `parent[node] = findUParent(parent[node])`:
- Recursively finds the root
- On the RETURN path, each node's parent gets updated to point directly to root
- One call compresses the entire path

---

### Path Compression vs No Compression

```
Without compression:
  Q queries on chain of n nodes: O(Q × n)
  10,000 queries, 10,000 nodes: 100,000,000 ops

With path compression:
  First query on chain: O(n) — compresses the path
  All subsequent queries: O(1) — direct to root
  Total: O(n + Q) ≈ O(Q) for large Q
  10,000 queries: ~10,000 ops → 10,000× faster
```

---

### Why Smaller Under Larger? (Formal Proof Sketch)

When we attach the smaller component under the larger one, any node in the smaller component that moves deeper does so because it joined a component **at least twice as large** as its previous one.

```
Node starts in component of size 1.
After being attached under a larger component: size ≥ 2.
Attached again: size ≥ 4.
Attached again: size ≥ 8.
...
After k such merges: size ≥ 2^k.
Maximum size = n → k ≤ log₂(n).
So any node's depth ≤ log₂(n). ✅
```

If we attached larger under smaller (the wrong way):
- A node could be in the root position and never go deeper (it absorbs others)
- But the nodes in the larger component could all become very deep suddenly

```
Wrong: attach large (size 100) under small (size 1)
  All 100 nodes in the large component now have depth ≥ 2.
  findParent for any of them = 2 hops minimum.
  
Right: attach small (size 1) under large (size 100)
  Only 1 node (from small) becomes deeper.
  All 100 nodes in large component unchanged.
```

Smaller under larger **minimizes the total depth increase** across all nodes.

---

## 9. The Complete Code Explained

```cpp
class DisjointSet {
    vector<int> rank, parent, size;

public:
    DisjointSet(int n) {
        rank.resize(n + 1, 0);    // all ranks start at 0
        parent.resize(n + 1, 0);
        size.resize(n + 1, 1);    // all components have size 1

        for(int i = 0; i <= n; i++)
            parent[i] = i;        // each node is its own parent (own component)
    }

    // Find with PATH COMPRESSION
    int findUParent(int node) {
        if(node == parent[node])
            return node;          // found root

        // Recursively find root AND compress path on return
        return parent[node] = findUParent(parent[node]);
    }

    // Union by RANK (approximate height)
    void unionByRank(int u, int v) {
        int uParentU = findUParent(u);
        int uParentV = findUParent(v);

        if(uParentU == uParentV) return;   // already in same component

        if(rank[uParentU] < rank[uParentV]) {
            parent[uParentU] = uParentV;   // shorter under taller
        } else if(rank[uParentV] < rank[uParentU]) {
            parent[uParentV] = uParentU;   // shorter under taller
        } else {
            parent[uParentV] = uParentU;   // equal → choose one, increase rank
            rank[uParentU]++;
        }
    }

    // Union by SIZE (exact count)
    void unionBySize(int u, int v) {
        int uParentU = findUParent(u);
        int uParentV = findUParent(v);

        if(uParentU == uParentV) return;   // already in same component

        if(size[uParentU] < size[uParentV]) {
            parent[uParentU] = uParentV;   // smaller under larger
            size[uParentV] += size[uParentU];
        } else {
            parent[uParentV] = uParentU;   // smaller (or equal) under larger
            size[uParentU] += size[uParentV];
        }
    }

    // Check if u and v are in same component
    string sameComponent(int u, int v) {
        // BUG in original: uses hardcoded 3 and 7!
        // Should be: return findUParent(u) == findUParent(v) ? "yes" : "no";
        return findUParent(3) == findUParent(7) ? "yes" : "no";
    }
};
```

> **Bug Note:** The `sameComponent` function in the original code has a bug — it hardcodes `3` and `7` instead of using the parameters `u` and `v`. The correct implementation should be:
> ```cpp
> return findUParent(u) == findUParent(v) ? "yes" : "no";
> ```

---

## 10. Full Driver Code Trace

```cpp
DisjointSet ds(7);

ds.unionBySize(1, 2);   // {1,2} {3} {4} {5} {6} {7}
ds.unionBySize(2, 3);   // {1,2,3} {4} {5} {6} {7}
ds.unionBySize(4, 5);   // {1,2,3} {4,5} {6} {7}
ds.unionBySize(6, 7);   // {1,2,3} {4,5} {6,7}
ds.unionBySize(5, 6);   // {1,2,3} {4,5,6,7}

cout << ds.sameComponent(3, 7);  // "no" ✅ (different components)

ds.unionBySize(3, 7);   // {1,2,3,4,5,6,7}

cout << ds.sameComponent(3, 7);  // "yes" ✅ (same component now)

cout << ds.findUParent(2);       // 4 ✅ (root of merged component is 4)
```

---

## 11. Overall Complexity Summary

| Operation | Naïve | +Union by Size | +Path Compression | Both |
|---|---|---|---|---|
| `findParent` | `O(n)` | `O(log n)` | `O(log n)` amort | `O(α(n))` |
| `union` | `O(n)` | `O(log n)` | `O(log n)` amort | `O(α(n))` |
| `sameComponent` | `O(n)` | `O(log n)` | `O(log n)` amort | `O(α(n))` |
| Q operations | `O(Qn)` | `O(Q log n)` | `O(Q log n)` | `O(Qα(n))` |
| Space | `O(n)` | `O(n)` | `O(n)` | `O(n)` |

> `α(n)` = inverse Ackermann function ≈ 4 for all practical inputs → essentially `O(1)`.

---

## 12. Interview Questions & Answers

---

### Q1. What is the difference between Union by Rank and Union by Size?

**Union by Rank** uses an *approximate* upper bound on tree height. Rank doesn't always equal actual height (especially after path compression flattens trees). Rank only increases when two equal-rank trees merge.

**Union by Size** uses the *exact* node count of each component. It's more precise and generally preferred in practice because:
- Size gives a tighter bound on tree height
- Size is always accurate (rank can become stale after path compression)
- Easier to reason about (exact count vs approximate height)

```
Both give O(log n) tree height without path compression.
Both give O(α(n)) with path compression.
Union by size is generally preferred for its precision.
```

---

### Q2. Can you implement DSU without recursion (iterative findParent)?

Yes. Two passes — first find root, then compress path:

```cpp
int findUParent(int node) {
    // Pass 1: find root
    int root = node;
    while(root != parent[root])
        root = parent[root];

    // Pass 2: path compression — point all nodes directly to root
    while(node != root) {
        int next = parent[node];
        parent[node] = root;
        node = next;
    }

    return root;
}
```

This avoids recursion stack overflow for very deep trees (though path compression prevents deep trees in practice).

---

### Q3. What happens if we call union on two nodes already in the same component?

```cpp
if(uParentU == uParentV) return;
```

The guard at the top of both `unionByRank` and `unionBySize` handles this. If both nodes have the same root → they're already in the same component → do nothing and return. No changes to parent, rank, or size.

This check is also used in **Kruskal's algorithm** to detect if adding an edge would create a cycle.

---

### Q4. How does DSU detect a cycle in an undirected graph?

Before adding edge `(u, v)`:
```
if findUParent(u) == findUParent(v):
    → u and v are already connected
    → adding this edge CREATES A CYCLE
else:
    → safe to add, then union(u, v)
```

This is exactly how Kruskal's MST algorithm works — sort edges by weight, add each edge only if it doesn't create a cycle (i.e., endpoints have different parents).

```cpp
// Cycle detection using DSU
for(auto& edge : edges) {
    int u = edge[0], v = edge[1];
    if(ds.findUParent(u) == ds.findUParent(v)) {
        cout << "Cycle detected!" << endl;
        break;
    }
    ds.unionBySize(u, v);
}
```

---

### Q5. What is the inverse Ackermann function α(n)? Why does it matter?

The **Ackermann function** A(m,n) grows incredibly fast — faster than any primitive recursive function.

Its **inverse** α(n) = smallest `m` such that A(m,m) ≥ n. This grows incredibly slowly:

```
α(1)         = 0
α(4)         = 1
α(16)        = 2
α(65536)     = 3
α(2^65536)   = 4
```

For ANY input you'll ever encounter in computing (including the number of atoms in the universe ≈ 10^80), α(n) ≤ 4.

**Why it matters:** With both path compression AND union by rank/size, the amortized cost per DSU operation is `O(α(n))`. Since α(n) ≤ 4 in practice, this is effectively O(1). DSU is as fast as theoretically possible for this problem.

---

### Q6. Is DSU possible for directed graphs?

Standard DSU is designed for **undirected graphs** — it treats every connection as bidirectional. For directed graphs, a different structure is needed.

However, if the question is "can we reach v from u?" in a directed graph, DSU doesn't directly apply (reachability in directed graphs isn't symmetric). You'd need BFS/DFS or Floyd-Warshall for that.

---

### Q7. Can DSU support edge deletions (not just additions)?

Standard DSU only supports **additions** (union operations). Undoing a union is not straightforward because path compression irreversibly modifies the tree structure.

There are advanced techniques:
- **Link-Cut Trees:** Support both edge addition and deletion in `O(log n)` per operation
- **Offline Dynamic Connectivity:** If queries and deletions are known in advance, can process efficiently

For most interview purposes, DSU = additions only.

---

### Q8. How do you count the number of connected components using DSU?

**Method 1:** Track count during unions.

```cpp
int components = n;  // initially n components

void unionBySize(int u, int v) {
    int rootU = findUParent(u);
    int rootV = findUParent(v);
    if(rootU == rootV) return;  // same component, no merge

    // merge...
    components--;  // two components merged into one
}
```

**Method 2:** Count distinct roots after all operations.

```cpp
int countComponents(int n) {
    unordered_set<int> roots;
    for(int i = 0; i < n; i++)
        roots.insert(findUParent(i));
    return roots.size();
}
```

---

### Q9. Rank vs Actual Height — Can rank lie?

Yes! After path compression, the actual tree height can become much smaller than the rank. Rank is an **upper bound** on height, not the exact height.

```
Tree before path compression:
1 → 2 → 3 → 4 (root, rank=3)

After findParent(1): 1,2,3 all point directly to 4
Actual height = 1, but rank[4] is still 3.
```

This is why union by SIZE is more precise — size always reflects the true count of nodes, which never becomes inaccurate.

---

### Q10. What's the space complexity? Can we reduce it?

Standard DSU uses `O(n)` space for `parent[]`, `rank[]` (or `size[]`).

**Can we reduce it?** No meaningful way — you need at least `O(n)` to store each node's parent. The `rank`/`size` array is a second `O(n)` but it's essential for the `O(log n)` tree height guarantee.

Total: `O(n)` — very space efficient compared to adjacency lists `O(V+E)` for graph algorithms.

---

### Q11. Prim's vs Kruskal's for MST — when to use DSU?

DSU is used in **Kruskal's algorithm**, not Prim's:

| Algorithm | Uses DSU? | TC | Best for |
|---|---|---|---|
| **Kruskal's** | ✅ Yes (cycle detection) | `O(E log E)` | Sparse graphs |
| **Prim's** | ❌ No (uses min-heap + visited) | `O(E log V)` | Dense graphs |

Kruskal's: sort all edges → add cheapest that doesn't form cycle (DSU check) → stop after V-1 edges added.

---

### Q12. What if n elements are labeled 1 to n vs 0 to n-1? Does DSU code change?

The code allocates size `n+1` (indices 0 to n) to support 1-indexed elements safely:

```cpp
rank.resize(n + 1, 0);
parent.resize(n + 1, 0);
size.resize(n + 1, 1);
for(int i = 0; i <= n; i++)
    parent[i] = i;
```

Index `0` is allocated but unused (wasted slot) for convenience. For 0-indexed elements (0 to n-1), just allocate size `n` and initialize from `i=0` to `i<n`. The logic is identical.

---

### Q13. How is DSU different from a hash map for grouping?

A hash map (e.g., `unordered_map<int, int>` mapping node → group_id) could track components but:
- **Merging two groups** with a hash map requires updating ALL elements of one group → `O(group_size)` per merge
- **DSU merge** is `O(α(n))` — just update one parent pointer, regardless of group size

DSU uses the tree structure with path compression to make merges fast without touching every element.

---

### Q14. Can rank become negative?

No. Rank starts at 0 and only increases (by 1 when two equal-rank trees merge). It never decreases. Size starts at 1 and only increases (adds the other component's size). Neither can go below their starting value.

---

### Q15. Write a DSU from scratch in an interview — what's the minimal correct version?

```cpp
struct DSU {
    vector<int> parent, size;

    DSU(int n) : parent(n), size(n, 1) {
        iota(parent.begin(), parent.end(), 0);  // parent[i] = i
    }

    int find(int x) {
        if(parent[x] != x)
            parent[x] = find(parent[x]);   // path compression
        return parent[x];
    }

    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if(x == y) return false;           // already connected

        if(size[x] < size[y]) swap(x, y); // smaller under larger
        parent[y] = x;
        size[x] += size[y];
        return true;                       // successfully merged
    }

    bool connected(int x, int y) {
        return find(x) == find(y);
    }
};
```

This is the clean, minimal, interview-ready DSU with union by size + path compression. `unite` returns `true` if a merge happened (useful for Kruskal's — count edges added to MST).
