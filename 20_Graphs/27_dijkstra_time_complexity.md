# Dijkstra's Algorithm — Deep Dive into Time Complexity

---

## 1. The Question We're Answering

Why is Dijkstra's time complexity written as `O(E log V)` and not something else? Where does each part come from? This document builds the answer piece by piece — from scratch.

---

## 2. What Operations Does Dijkstra Actually Do?

Strip the algorithm down to its core operations:

```
While the priority queue is not empty:
  1. Extract the minimum-distance node      ← heap operation
  2. For each outgoing edge from that node:
       Relax the edge (maybe update dist)   ← computation
       If dist updated → push to heap       ← heap operation
```

Every heap operation (push or pop) costs `O(log H)` where `H` = current heap size.

So the total cost = **(number of heap operations) × (cost per heap operation)**

We need to count both carefully.

---

## 3. Counting Heap Operations

### Extractions (Pops) — How Many?

Each time we pop from the heap, we're processing a `{distance, node}` entry. In the worst case (priority queue version with stale entries), each edge can push one entry to the heap. So heap can have up to `O(E)` entries.

But in terms of **actual node processing**, each node is meaningfully processed at most once (the first pop with a non-stale entry). Subsequent pops of the same node are caught by the stale check.

> Number of pops = total entries pushed = up to `O(E)` (worst case)

### Insertions (Pushes) — How Many?

An entry is pushed whenever a distance is updated (relaxation succeeds):

```cpp
if(newDist < dist[neighbour]) {
    dist[neighbour] = newDist;
    pq.push({newDist, neighbour});   ← one push per successful relaxation
}
```

In the worst case, every edge causes a successful relaxation → up to `E` pushes.

> Total heap insertions = up to `O(E)`

---

## 4. The Heap Size `H` — Why It Matters

Each heap operation costs `O(log H)`. So what is `H` at its worst?

In the **priority queue version** (no stale entry removal), every relaxation pushes a new entry. Old entries are NOT removed. So over time, the heap accumulates entries:

```
Start: H = 1   (just source)
After processing source's edges: H grows by (degree of source)
After processing next node's edges: H grows by (degree of that node)
...
In worst case: H = O(E) total entries pushed
```

So cost per heap operation = `O(log H)` = `O(log E)`.

**But since `E ≤ V²`:**

```
log E ≤ log(V²) = 2 log V
```

So `O(log E)` = `O(log V²)` = `O(2 log V)` = `O(log V)`

> **Key equivalence: `O(log E) = O(log V)`**

This is why the complexity is written with `log V` even though the heap technically has `O(E)` entries — they're asymptotically the same.

---

## 5. Building the Full Complexity

### Step-by-Step Derivation

**Total work = (number of heap operations) × (cost per operation)**

```
Extractions: O(V) meaningful pops + O(E) stale pops = O(V + E) ≈ O(E) for connected graphs
Insertions:  O(E) pushes
Cost each:   O(log H) = O(log E) = O(log V)

Total = O(E) × O(log V) = O(E log V)
```

---

### The Dense Graph Worst Case (From the Video)

Let's verify with a dense graph where `E ≈ V²`:

In a **complete graph** (every node connected to every other node):
- `V` nodes, `E = V(V-1)/2 ≈ V²` edges
- Each node has `V-1` neighbors
- Processing one node → up to `V-1` relaxations → up to `V-1` heap pushes

Total heap operations ≈ V nodes × (V-1) edge relaxations each = `V²` ≈ `E`

Cost per operation = `O(log H)` where `H ≈ V²` = `O(log V²)` = `O(2 log V)` = `O(log V)`

```
Total = V × (V-1) × O(log V²)
      = V² × O(2 log V)
      = O(V² log V)
      = O(E log V)    [since E ≈ V² for dense graph]
```

This confirms: **`O(E log V)` in the worst case for dense graphs**.

---

## 6. Why a Simple Queue Gives Worse Complexity

What happens if we replace the priority queue with a regular queue (FIFO)?

A simple queue processes nodes in the order they were enqueued — NOT in order of shortest distance. This means:

```
Problem: A node can be enqueued multiple times with different distances,
         AND processed multiple times (even with longer distances first).

Example:
  Node 3 enqueued with distance 7 (found via path A)
  Node 3 enqueued with distance 3 (found later via shorter path B)
  Queue order: [... (7,3) ... (3,3) ...]
  
  Process (7,3) first → relax neighbors with dist=7 → wrong updates
  Then process (3,3) → re-relax neighbors with dist=3 → correct updates
  → All neighbors of node 3 processed TWICE
```

Each node can be processed up to `V` times (updated by each of its predecessors). Each processing inspects all of the node's edges.

```
Worst case with simple queue:
  O(V) updates per node × V nodes × O(E/V) average edges per node
= O(V × E)
= O(EV)
```

Compare:
- Priority queue: `O(E log V)` 
- Simple queue: `O(EV)` (can be up to `O(V³)` for dense graphs!)

---

## 7. The Complete Cost Breakdown Table

| Component | What it represents | Count | Cost each | Total |
|---|---|---|---|---|
| **Meaningful pops** | First time a node is popped (non-stale) | `O(V)` | `O(log E)` = `O(log V)` | `O(V log V)` |
| **Stale pops** | Discarded by `continue` check | Up to `O(E)` | `O(log E)` = `O(log V)` | `O(E log V)` |
| **Pushes** | One per successful edge relaxation | Up to `O(E)` | `O(log E)` = `O(log V)` | `O(E log V)` |
| **Edge relaxation logic** | `dist[node] + wt < dist[neighbor]` check | `O(E)` | `O(1)` | `O(E)` |

**Grand Total: `O(E log V)`** (dominated by push/pop operations)

---

## 8. Set-Based Implementation — Why `O(E log V)` Becomes Tighter

With `std::set` (true decrease-key), stale entries don't accumulate. The set has **at most V entries** at any time.

```
Set size H ≤ V   (one entry per node, old entries erased)

Cost per operation = O(log V)   [same]
Number of operations = O(V) extractions + O(E) relaxations (erase + insert)

Total = O((V + E) log V) = O(E log V)   [same asymptotic]
```

Same asymptotic complexity, but smaller constant — the `log` factor is truly `log V` (not `log E` that happens to simplify to `log V`). In practice, fewer cache misses from smaller data structure.

---

## 9. Fibonacci Heap — The Theoretical Optimum

For completeness:

| Priority Queue Type | TC for Dijkstra | Notes |
|---|---|---|
| Binary heap (standard) | `O((V + E) log V)` | Most common in practice |
| `std::set` (RB-tree) | `O((V + E) log V)` | True decrease-key, `O(log V)` |
| Fibonacci heap | `O(E + V log V)` | `O(1)` amortized decrease-key |
| Array (no heap) | `O(V²)` | Better for dense graphs! |

**Fibonacci heap** achieves `O(1)` amortized for decrease-key → `O(E + V log V)` total. For sparse graphs (`E ≈ V`), this is `O(V log V)`. Theoretically optimal but complex to implement → rarely used in practice.

**Array (no heap):** For dense graphs where `E ≈ V²`, `O(V²)` beats `O(E log V) = O(V² log V)`. Linear scan to find minimum = `O(V)` per extraction × `V` extractions = `O(V²)`. Better constant than heap-based for dense graphs.

---

## 10. Intuition Summary — Why Each Part of `O(E log V)` Exists

```
O(E log V)
│   │
│   └── log V: cost of each heap operation
│              heap size ≤ O(E) ≤ O(V²)
│              log(V²) = 2 log V = O(log V)
│
└── E: total number of heap operations
       each edge causes at most one push
       each entry causes at most one pop
       → total ops proportional to E
```

**In plain English:**

> We look at every edge at least once (`E` factor). For each edge that causes a distance update, we do a heap operation that costs logarithmic time in the number of nodes (`log V` factor). That's where `E log V` comes from.

---

## 11. When to Use Which Dijkstra Variant

| Graph type | Best approach | Why |
|---|---|---|
| Sparse (`E ≈ V`) | Binary heap / `set` | `O(V log V)` — heap overhead worthwhile |
| Dense (`E ≈ V²`) | Array (no heap) | `O(V²)` beats `O(V² log V)` |
| Need true decrease-key | `std::set` | No stale entries, `O(log V)` per op |
| Theoretical research | Fibonacci heap | `O(E + V log V)` optimal |
| Competitive programming | `priority_queue` | Simplest correct code |
