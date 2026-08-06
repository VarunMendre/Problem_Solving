# Cheapest Flights Within K Stops

---

## 1. Problem Statement

There are `n` cities connected by flights. You are given:
- `flights[i] = [from, to, price]` — a directed flight with a price
- `src` — source city
- `dst` — destination city
- `k` — maximum number of **stops** allowed (NOT including src and dst)

Find the **cheapest price** from `src` to `dst` with **at most `k` stops**. Return `-1` if no such route exists.

```
n=4, src=0, dst=3, k=1

Flights:
0→1 ($100), 0→2 ($500), 1→3 ($100), 2→3 ($100)

Path with 0 stops: 0→3 (no direct flight)
Path with 1 stop:  0→1→3 = $200 ✅
                   0→2→3 = $600
Path with 2 stops: not allowed (k=1)

Answer: 200
```

---

## 2. Why Standard Dijkstra FAILS Here

### The Constraint That Breaks Dijkstra

Standard Dijkstra has no concept of "stops". It only minimizes total cost, freely using any number of edges. But here, we have a hard constraint: **at most `k` stops** (i.e., at most `k+1` edges on the path).

This constraint creates a scenario where Dijkstra's finalization guarantee **breaks**.

---

### Formal Example — Dijkstra Gives Wrong Answer

```
n=4, src=0, dst=3, k=1

Flights:
0→1: $100
0→2: $500
1→3: $100
1→2: $200   ← critical edge
2→3: $10    ← very cheap

Graph:
       $100      $100
0 ——→ 1 ——→ 3
|         ↘ $200 ↗ $10
$500       2
|          ↑
└—————————→┘
```

**Run standard Dijkstra from 0:**

```
Initial: dist = [0, INF, INF, INF], pq = [{0,0}]

Pop (0, node=0):
  → neighbor 1, cost=100: dist[1]=100, push (100,1)
  → neighbor 2, cost=500: dist[2]=500, push (500,2)

Pop (100, node=1):   ← 1 stop used (0→1)
  → neighbor 3, cost=100: dist[3]=200, push (200,3)
  → neighbor 2, cost=200: 100+200=300 < 500 → dist[2]=300, push (300,2)

Pop (200, node=3):   ← Dijkstra FINALIZES dist[3]=200
  Dijkstra returns 200
```

Dijkstra says **$200** via path `0→1→3` (1 stop). ✅ This happens to be correct here.

**But now change k=0 (zero stops allowed — only direct flights):**

There's no direct `0→3` flight → answer should be **-1**.

```
Dijkstra doesn't know about k.
It still finds 0→1→3 for $200 and returns 200.
WRONG — this path uses 1 stop, but k=0. ❌
```

---

### Deeper Failure — When Cheap Path Needs Too Many Stops

```
n=4, src=0, dst=3, k=1

Flights:
0→1: $1
0→2: $1000
1→2: $1
2→3: $1

Correct answer with k=1: 0→2→3 = $1001 (1 stop)
Path 0→1→2→3 = $3 (2 stops, EXCEEDS k=1, INVALID)

Run Dijkstra (no stop tracking):
  dist[0]=0
  Pop 0: push (1, node=1), push (1000, node=2)
  Pop (1, node=1): push (2, node=2) → dist[2]=2
  Pop (2, node=2): push (3, node=3) → dist[3]=3
  
  Dijkstra returns dist[3] = 3  via 0→1→2→3
  
  But this path uses 2 stops (k=1 is the limit)!
  The path 0→1→2→3 is INVALID.
  Correct answer is 1001, but Dijkstra says 3. ❌
```

**Root cause:** Dijkstra's finalization says "when a node is popped, its cost is final." But this cost was achieved via a path with too many stops. With the stop constraint, a more expensive path (fewer stops) might be the only valid one.

Dijkstra has no mechanism to track how many stops were used — it can't distinguish between a 2-stop path and a 1-stop path to the same node.

---

### Why Can't We Just Add Stop Count to Dijkstra?

```
Store {cost, stops, node} in heap.
Finalize when {cost, stops} for a node is optimal.
```

Problem: now there are two dimensions (cost AND stops). A node can have many valid states — the "best" one depends on what you care about. This turns into a complex multi-dimensional optimization, losing the simple greedy guarantee.

Dijkstra can only finalize based on ONE metric. When we add a constraint on a second metric (stops), we need a different approach.

---

## 3. Why BFS (Level by Level) Is the Right Approach

### The Key Observation

In this problem, **stops = BFS levels**. Each BFS level corresponds to one more stop:
- Level 0: nodes reachable in 0 stops = just `src`
- Level 1: nodes reachable in 1 stop = direct neighbors of `src`
- Level 2: nodes reachable in 2 stops
- ...

Processing **level by level (BFS)** naturally enforces the stop constraint:
- Process all `k+1` levels → automatically ensures we never use more than `k` stops

This is why we use a **regular queue** (FIFO/BFS), NOT a priority queue (Dijkstra):
- Queue processes stops in order: level 0 → level 1 → level 2 → ...
- Priority queue processes by cost, jumping between levels out of order

---

### The `stops <= k` Check

```cpp
if(stops > k)
    continue;   // exceeded stop limit → skip this path
```

This prevents extending any path that has already used `k` stops. If `stops == k`, we can still arrive at neighbors (adding them costs nothing because they'd have `k+1` stops — past the limit), but we can't extend further from them.

Wait — let's re-read: `stops` here is the number of stops used so far. When we push `{stops+1, adjNode, newCost}`, `stops+1` is the stop count for reaching `adjNode`. If `stops+1 > k`, the `if(stops > k) continue` on the NEXT iteration will skip it.

Actually: `stops` when popped = number of stops to reach `node`. We can extend to neighbors if `stops <= k` (we still have stop budget left).

---

### Why Regular Queue and Not Priority Queue?

**Priority queue (Dijkstra)** orders by cost. A 0-stop path costing $1000 would be processed AFTER a 5-stop path costing $5. This jumbles the levels, making stop tracking inaccurate.

**Regular queue (BFS)** processes in insertion order — which is level order. All 1-stop entries are processed before any 2-stop entries are added (since 2-stop entries are only pushed when processing 1-stop entries). This naturally separates levels.

Wait — actually in this implementation, we're using a regular queue (not strictly BFS). It processes in FIFO order. Since we push `stops+1` entries when processing `stops` entries, and a regular queue is FIFO, the ordering isn't perfectly level-by-level... but the `stops > k` guard handles out-of-order correctly regardless.

This is essentially a **BFS-style relaxation** with cost tracking, rather than strict BFS or Dijkstra.

---

## 4. Dry Run

```
n=4, src=0, dst=3, k=1

Flights: 0→1($100), 0→2($500), 1→3($100), 2→3($100)

adjLs:
0 → [(1,100), (2,500)]
1 → [(3,100)]
2 → [(3,100)]

dist = [0, INF, INF, INF]
queue = [{stops=0, node=0, cost=0}]
```

---

**Pop {0, node=0, cost=0}:**
```
stops=0 ≤ k=1 → process

neighbor 1, edgeCost=100: newCost=100 < dist[1]=INF
  dist[1]=100, push {stops=1, node=1, cost=100}

neighbor 2, edgeCost=500: newCost=500 < dist[2]=INF
  dist[2]=500, push {stops=1, node=2, cost=500}

dist  = [0, 100, 500, INF]
queue = [{1,1,100}, {1,2,500}]
```

**Pop {1, node=1, cost=100}:**
```
stops=1 ≤ k=1 → process

neighbor 3, edgeCost=100: newCost=200 < dist[3]=INF
  AND stops(1) ≤ k(1) → push {stops=2, node=3, cost=200}
  dist[3]=200

dist  = [0, 100, 500, 200]
queue = [{1,2,500}, {2,3,200}]
```

**Pop {1, node=2, cost=500}:**
```
stops=1 ≤ k=1 → process

neighbor 3, edgeCost=100: newCost=600 < dist[3]=200? NO → skip

queue = [{2,3,200}]
```

**Pop {2, node=3, cost=200}:**
```
stops=2 > k=1 → continue (skip) 

queue = []
```

**Result: dist[3] = 200 → return 200 ✅**

---

**Now the tricky case — k=0:**
```
dist = [0, INF, INF, INF]
queue = [{0, node=0, cost=0}]

Pop {0, node=0, cost=0}:
  stops=0 ≤ k=0 → process
  neighbor 1: dist[1]=100, push {1, 1, 100}
  neighbor 2: dist[2]=500, push {1, 2, 500}

Pop {1, node=1, cost=100}:
  stops=1 > k=0 → SKIP

Pop {1, node=2, cost=500}:
  stops=1 > k=0 → SKIP

queue = []
dist[3] = INF → return -1 ✅
```

The stop constraint correctly prevents using any path that requires a stop!

---

## 5. Story Points

---

**Story Point 1 — "Dijkstra finalizes cost, but cost depends on path used, and path is constrained"**

This is the fundamental incompatibility. Dijkstra finalizes `dist[node]` when popped, assuming no future path can improve it. But when k is small, the cheap path to a node might need too many stops. A more expensive (but fewer stops) path might be the only valid one. Dijkstra can't account for this because it doesn't track stops.

---

**Story Point 2 — "BFS level = stop count — this is the natural structure of the problem"**

Each edge = 1 stop. Processing level-by-level = processing by stop count. BFS naturally respects this. The moment you use a priority queue (ordering by cost), you lose the level-ordering and mix up entries from different stop counts.

---

**Story Point 3 — "The queue stores `{stops, node, cost}` — cost is NOT the ordering key"**

Unlike Dijkstra where `{cost, node}` is stored and min-heap orders by cost, here the queue is ordered by insertion (FIFO). Cost is just data we carry along — it's used for relaxation (`newCost < dist[adjNode]`), not for ordering.

---

**Story Point 4 — "A node CAN be visited multiple times with different stop counts"**

In Dijkstra, once a node is finalized, it's never updated. Here, the same node might be updated multiple times:

```
dist[node] = 100 (via 1-stop path)
dist[node] = 50  (via 2-stop path, if k≥2)
```

Both updates are valid and necessary — the stop-constrained problem has multiple "dimensions" of optimality. A node with `dist=50` via 2 stops is different from the same node with `dist=100` via 1 stop depending on what `k` is.

---

**Story Point 5 — "This approach has a subtle issue with over-processing"**

The `newCost < dist[adjNode]` check helps prune, but it's possible to push the same node to the queue multiple times if it's relaxed via different stop-count paths. This is the trade-off: we don't have Dijkstra's finalization guarantee, so we may do more work. But the `stops > k` guard caps the maximum depth to `k` levels, bounding total work.

---

## 6. Code

```cpp
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights,
                          int src, int dst, int k) {
        // Build directed adjacency list
        vector<vector<pair<int,int>>> adjLs(n);
        for(auto& it : flights) {
            adjLs[it[0]].push_back({it[1], it[2]});
        }

        // Regular queue: {stops, {node, cost}}
        // stops = number of intermediate stops used to reach node
        queue<pair<int, pair<int,int>>> q;
        q.push({0, {src, 0}});

        vector<int> dist(n, INT_MAX);
        dist[src] = 0;

        while(!q.empty()) {
            int stops = q.front().first;
            int node  = q.front().second.first;
            int cost  = q.front().second.second;
            q.pop();

            // Exceeded stop limit — don't extend further
            if(stops > k) continue;

            for(auto& it : adjLs[node]) {
                int adjNode  = it.first;
                int edgeCost = it.second;
                int newCost  = cost + edgeCost;

                // Relax only if cheaper AND within stop budget
                if(newCost < dist[adjNode] && stops <= k) {
                    dist[adjNode] = newCost;
                    q.push({stops + 1, {adjNode, newCost}});
                }
            }
        }

        return dist[dst] == INT_MAX ? -1 : dist[dst];
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(E × K)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(E)` | E flights |
| BFS processing | `O(E × K)` | Each edge can be relaxed up to K times (once per stop level) |

**Total: `O(E × K)`**

> Each edge can generate one queue entry per stop level (0 to k). So total queue entries ≤ `E × K`. Processing each entry = `O(1)` + neighbor lookups. Total edges in all neighbor lookups ≤ `E × K`.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + E)` | Adjacency list |
| `dist[]` | `O(V)` | One per city |
| Queue | `O(V × K)` worst case | At most V nodes × K stop levels |

**Total: `O(V × K + E)`**

---

### Dijkstra vs BFS for This Problem

| | Dijkstra | BFS (this approach) |
|---|---|---|
| **Ordering** | By cost (min-heap) | By insertion (FIFO) |
| **Stop tracking** | ❌ None | ✅ Explicit `stops` counter |
| **Finalization** | When popped (cost-based) | Never — cost updated if cheaper found |
| **Correctness with k** | ❌ Wrong for small k | ✅ Correct |
| **TC** | `O((V+E) log V)` | `O(E × K)` |
| **When to use** | No stop constraint | With stop constraint |
