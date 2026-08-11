# Number of Ways to Arrive at Destination

---

## 1. Problem Statement

You are in a city with `n` intersections (labeled `0` to `n-1`) connected by bidirectional roads with travel times. Find the **number of ways** to travel from intersection `0` to intersection `n-1` in the **minimum time**.

Return the answer modulo `10⁹ + 7`.

```
n=7
Roads: 0-6(7), 0-1(2), 1-2(3), 1-3(3), 6-3(3), 3-5(1), 6-5(1), 2-5(1), 0-4(5), 4-6(2)

Minimum time from 0 to 6 = 7

Paths achieving time 7:
- 0→6 directly (time 7)
- 0→4→6 (5+2=7)
- 0→1→2→5→6 (2+3+1+1=7)
- 0→1→3→5→6 (2+3+1+1=7)

Answer: 4
```

---

## 2. Intuition / Approach

### What This Problem Adds to Dijkstra

Standard Dijkstra finds the **shortest distance** to each node. This problem asks: **how many shortest paths exist** to the destination?

We need to count the number of ways to reach each node via the shortest path. This requires one extra array:

- `dist[i]` = shortest distance from `0` to node `i` (standard Dijkstra)
- `ways[i]` = number of shortest paths from `0` to node `i` (new addition)

---

### The Two Cases During Relaxation

For each neighbor `adjNode` when processing from `node`:

```cpp
long long newDistance = distance + adjWt;
```

**Case 1 — Found a strictly shorter path: `newDistance < dist[adjNode]`**

```cpp
dist[adjNode] = newDistance;
ways[adjNode] = ways[node];   // ← RESET ways
pq.push({newDistance, adjNode});
```

We found a better path. All previously counted paths to `adjNode` are now suboptimal — DISCARD them. The only ways to reach `adjNode` optimally are the ways we can reach `node` (then extend via this edge). So `ways[adjNode] = ways[node]`.

**Case 2 — Found an equally short path: `newDistance == dist[adjNode]`**

```cpp
ways[adjNode] = (ways[adjNode] + ways[node]) % mod;
```

We found ANOTHER path of the same optimal length. ADD the number of ways we can reach `node` to the existing count for `adjNode`. Both old paths AND new paths through `node` are valid optimal paths.

**Case 3 — Found a longer path: `newDistance > dist[adjNode]`**

```
Do nothing. This path is suboptimal — ignore completely.
```

---

### Why Dijkstra's Finalization Makes This Correct

When a node `u` is popped from the min-heap, `dist[u]` is finalized. This means `ways[u]` is also finalized — all shorter or equal-length paths to `u` have already been discovered (since Dijkstra processes nodes in non-decreasing distance order).

When we process `u`'s neighbors:
- Any neighbor that hasn't been finalized yet gets updated
- The stale entry check (`distance > dist[node]`) ensures we don't process a node with an outdated distance, which would give incorrect `ways` counts

---

### Initialization — Why `ways[0] = 1`?

There is exactly **1 way** to reach the source (node `0`) — by starting there. This seeds the counting: when we reach a neighbor `v` of `0`, `ways[v] = ways[0] = 1` (one path: via node `0`).

---

### Why `LLONG_MAX` Instead of `INT_MAX`?

Road weights can be large, and we sum them. `distance + adjWt` could overflow `int`. Using `long long` prevents overflow. `LLONG_MAX` = max value of `long long`.

Similarly, `ways` values can grow large (many paths) before modding — using `long long` and `% mod` keeps them in range.

---

### The `PLI = pair<long long, int>` Type

```cpp
using PLI = pair<long long, int>;  // {distance, node}
```

Distance must be `long long` (to avoid overflow). Node index is `int`. The pair is ordered by distance (first element) in the min-heap.

---

## 3. Dry Run

```
n=5
Roads: 0-1(1), 0-2(4), 1-2(2), 1-3(5), 2-3(1)

adjLs:
0 → [(1,1), (2,4)]
1 → [(0,1), (2,2), (3,5)]
2 → [(0,4), (1,2), (3,1)]
3 → [(1,5), (2,1)]

dist = [0, INF, INF, INF, INF] (n=5 but only 4 nodes used)
ways = [1, 0,   0,   0,   0]
pq   = [{0, 0}]
```

---

**Pop {0, node=0}:**
```
0 > dist[0]=0? NO → process

neighbor 1, wt=1: newDist=1 < INF
  Case 1: dist[1]=1, ways[1]=ways[0]=1, push {1,1}

neighbor 2, wt=4: newDist=4 < INF
  Case 1: dist[2]=4, ways[2]=ways[0]=1, push {4,2}

dist = [0, 1, 4, INF]
ways = [1, 1, 1,   0]
pq   = [{1,1}, {4,2}]
```

**Pop {1, node=1}:**
```
1 > dist[1]=1? NO → process

neighbor 0, wt=1: newDist=2 > dist[0]=0 → Case 3: skip
neighbor 2, wt=2: newDist=3 < dist[2]=4
  Case 1: dist[2]=3, ways[2]=ways[1]=1, push {3,2}
  (old ways[2]=1 DISCARDED — that path cost 4, this costs 3)
neighbor 3, wt=5: newDist=6 < INF
  Case 1: dist[3]=6, ways[3]=ways[1]=1, push {6,3}

dist = [0, 1, 3, 6]
ways = [1, 1, 1, 1]
pq   = [{3,2}, {4,2}, {6,3}]
```

**Pop {3, node=2}:**
```
3 > dist[2]=3? NO → process

neighbor 0, wt=4: 7 > 0 → skip
neighbor 1, wt=2: 5 > 1 → skip
neighbor 3, wt=1: newDist=4 < dist[3]=6
  Case 1: dist[3]=4, ways[3]=ways[2]=1, push {4,3}
  (old ways[3]=1 DISCARDED — that path cost 6, this costs 4)

dist = [0, 1, 3, 4]
ways = [1, 1, 1, 1]
pq   = [{4,2}, {4,3}, {6,3}]
```

**Pop {4, node=2} (STALE):**
```
4 > dist[2]=3? YES → STALE → continue
```

**Pop {4, node=3}:**
```
4 > dist[3]=4? NO → process

neighbor 1, wt=5: 9 > 1 → skip
neighbor 2, wt=1: 5 > 3 → skip

No updates.
```

**Pop {6, node=3} (STALE):**
```
6 > dist[3]=4? YES → STALE → continue

queue = []
```

**Answer: ways[3] = 1** (only one shortest path: `0→1→2→3` with cost 4)

---

**Let's add one more road `0→3(4)` to get TWO shortest paths:**

```
Roads: 0-1(1), 0-2(4), 1-2(2), 1-3(5), 2-3(1), 0-3(4)

Now: 0→1→2→3 = 1+2+1 = 4, and 0→3 = 4

After processing node 0:
  dist[3]=4, ways[3]=1  (via direct edge 0→3)

After processing node 1, node 2:
  dist[3] = 4 (same as before)
  but 0→1→2→3 = 4 == dist[3]=4 → Case 2!
  ways[3] = (1 + ways[2]) = (1 + 1) = 2

Answer: 2 ✅ (two paths, both cost 4)
```

---

## 4. Story Points

---

**Story Point 1 — "Reset ways on strictly shorter, ADD ways on equal distance"**

This is the heart of the algorithm. The two cases:
- `newDist < dist[adj]` → REPLACE: found better path, old paths are obsolete
- `newDist == dist[adj]` → ADD: found another equally good path, both count

Forgetting the reset (Case 1) would count suboptimal paths. Forgetting the add (Case 2) would miss valid equally-short paths. Both cases must be handled.

---

**Story Point 2 — "Stale entry check prevents wrong `ways` counts"**

```cpp
if(distance > dist[node]) continue;
```

If we process a stale entry (outdated distance), `ways[node]` at that point might be stale too. We'd propagate incorrect way counts to neighbors. The stale check prevents this — we only propagate `ways[node]` when processing the current best distance for `node`.

---

**Story Point 3 — "Modular arithmetic only in Case 2 addition"**

```cpp
ways[adjNode] = (ways[adjNode] + ways[node]) % mod;
```

The mod is applied when ADDING ways (Case 2). In Case 1 (assignment), we do `ways[adjNode] = ways[node]` without mod — `ways[node]` is already within mod range from previous operations. The modular arithmetic prevents `ways` values from overflowing `long long`.

---

**Story Point 4 — "When node n-1 is first popped — its `dist` AND `ways` are both finalized"**

Just like standard Dijkstra finalizes `dist[node]` on first pop, here BOTH `dist[node]` and `ways[node]` are finalized. All paths of shorter length have already been processed, and all equal-length paths have been counted via the Case 2 additions during earlier processing. So `ways[n-1]` when we return is the exact count of shortest paths.

---

**Story Point 5 — "This pattern generalizes: count optimal paths in any shortest path problem"**

The same `dist[] + ways[]` double-array pattern applies to:
- BFS shortest paths (unweighted) → same logic, queue instead of heap
- DAG shortest paths (topo sort) → same logic, processed in topo order
- Grid shortest paths (Dijkstra on grids) → same logic

Any time you need "number of ways to reach destination optimally", add the `ways` array and the two cases.

---

## 5. Code

```cpp
class Solution {
public:
    using PLI = pair<long long, int>;   // {distance, node}

    int countPaths(int n, vector<vector<int>>& roads) {
        // Build undirected weighted adjacency list
        vector<vector<pair<int,int>>> adjLs(n);
        for(auto& it : roads) {
            adjLs[it[0]].push_back({it[1], it[2]});
            adjLs[it[1]].push_back({it[0], it[2]});
        }

        // Min-heap: {distance, node}
        priority_queue<PLI, vector<PLI>, greater<PLI>> pq;

        vector<long long> dist(n, LLONG_MAX);   // shortest distance
        vector<long long> ways(n, 0);            // count of shortest paths

        const long long mod = 1e9 + 7;

        dist[0] = 0;
        ways[0] = 1;    // exactly 1 way to reach source (start here)
        pq.push({0, 0});

        while(!pq.empty()) {
            auto [distance, node] = pq.top();
            pq.pop();

            // Stale entry: dist already improved → skip
            if(distance > dist[node]) continue;

            for(auto& it : adjLs[node]) {
                int adjNode = it.first;
                int adjWt   = it.second;
                long long newDistance = distance + adjWt;

                if(newDistance < dist[adjNode]) {
                    // Case 1: Found strictly shorter path
                    // Reset ways — previous counts were for a longer path
                    dist[adjNode] = newDistance;
                    ways[adjNode] = ways[node];
                    pq.push({newDistance, adjNode});
                }
                else if(newDistance == dist[adjNode]) {
                    // Case 2: Found another equally shortest path
                    // Add the number of ways to reach current node
                    ways[adjNode] = (ways[adjNode] + ways[node]) % mod;
                }
                // Case 3: newDistance > dist[adjNode] → longer path, ignore
            }
        }

        return ways[n-1] % mod;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O((V + E) log V)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | V lists + 2E entries (undirected) |
| Dijkstra BFS | `O((V + E) log V)` | Same as standard Dijkstra |
| Extra `ways` operations | `O(1)` per relaxation | Just addition/assignment |

**Total: `O((V + E) log V)`**

> The `ways` array adds only `O(1)` work per edge — no asymptotic overhead over standard Dijkstra.

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + 2E)` | Undirected graph |
| `dist[]` | `O(V)` | `long long` per node |
| `ways[]` | `O(V)` | `long long` per node |
| Priority queue | `O(E)` worst case | One entry per edge push |

**Total: `O(V + E)`**

---

### Standard Dijkstra vs This Problem

| Aspect | Standard Dijkstra | Count Shortest Paths |
|---|---|---|
| **What we track** | `dist[]` | `dist[]` + `ways[]` |
| **On strictly shorter** | Update dist, push | Update dist, RESET ways, push |
| **On equal distance** | Nothing | ADD ways (modular) |
| **Final answer** | `dist[n-1]` | `ways[n-1]` |
| **Extra cost** | — | `O(1)` per edge (ways update) |
| **TC** | `O((V+E) log V)` | `O((V+E) log V)` — same |
