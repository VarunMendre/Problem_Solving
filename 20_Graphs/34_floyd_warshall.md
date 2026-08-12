# Floyd-Warshall Algorithm — All Pairs Shortest Path

---

## 1. Problem Statement

Given a weighted directed graph represented as an `n × n` adjacency matrix `dist` where:
- `dist[i][j]` = weight of direct edge from `i` to `j`
- `dist[i][j] = INF (1e8)` if no direct edge exists
- `dist[i][i] = 0` (distance from node to itself)

Update `dist` **in-place** so that `dist[i][j]` contains the **shortest distance from node `i` to node `j`** for ALL pairs `(i, j)`.

```
Initial dist (4 nodes):
     0    1    2    3
0 [  0    3   INF   7  ]
1 [ INF   0    2   INF ]
2 [ INF  INF   0    1  ]
3 [  6   INF  INF   0  ]

After Floyd-Warshall:
     0    1    2    3
0 [  0    3    5    6  ]
1 [ INF   0    2    3  ]
2 [  7   INF   0    1  ]
3 [  6    9   11    0  ]
```

---

## 2. Why Floyd-Warshall?

### When to Choose Floyd-Warshall

| Situation | Algorithm |
|---|---|
| Single source, positive weights | Dijkstra: `O((V+E) log V)` |
| Single source, negative weights | Bellman-Ford: `O(V × E)` |
| **All pairs shortest paths** | **Floyd-Warshall: `O(V³)`** |
| Dense graph, all pairs | Floyd-Warshall (simple, constant factors better than V × Dijkstra) |

Floyd-Warshall solves the **All Pairs Shortest Path (APSP)** problem — every node as source simultaneously. Running Dijkstra from each node would cost `O(V × (V+E) log V)`. For dense graphs (`E ≈ V²`), Floyd-Warshall's `O(V³)` is competitive.

---

## 3. Intuition / Approach

### The "Via Node" Insight

The key question is: what's the shortest path from `i` to `j`?

The shortest path either:
1. Goes **directly** from `i` to `j` (no intermediate nodes)
2. Goes from `i` → ... → **some intermediate node `via`** → ... → `j`

Floyd-Warshall's brilliance: **enumerate all possible intermediate nodes one by one**.

For each candidate intermediate node `via` (from `0` to `n-1`):
```
For every pair (i, j):
    shortest path through `via` = dist[i][via] + dist[via][j]
    dist[i][j] = min(dist[i][j], dist[i][via] + dist[via][j])
```

**The outer loop over `via`** is the critical ordering. After processing `via = k`, `dist[i][j]` contains the shortest path from `i` to `j` using only nodes `{0, 1, ..., k}` as intermediate nodes.

---

### Why the `via` Loop is Outermost

This is the most important structural insight. Let's think about it:

When we process `via = k`, we compute:
```
dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

For this to be correct, `dist[i][k]` and `dist[k][j]` must already contain the shortest paths using intermediate nodes `{0, 1, ..., k-1}`.

**Are they?** Yes — because `via` goes from `0` to `n-1` in order. By the time we process `via = k`:
- `dist[i][k]` was computed during `via = 0, 1, ..., k-1` → correctly reflects shortest path using `{0..k-1}` as intermediates
- `dist[k][j]` similarly

If `i` or `j` loops were outermost instead, `dist[i][k]` and `dist[k][j]` might not yet reflect all possible shorter sub-paths — giving incorrect results.

---

### Analogy — Building a Road Network

Imagine you have cities `0, 1, 2, 3`. You want shortest routes between all pairs.

**Round 1 (via=0):** "Can any pair of cities get a shorter route by going through city 0?"
**Round 2 (via=1):** "Can any pair of cities get a shorter route by going through city 1?" (using routes from round 1 as building blocks)
**Round 3 (via=2):** "...city 2?" (using routes from rounds 1 and 2)
**Round 4 (via=3):** "...city 3?"

After all rounds: every possible intermediate city has been considered for every pair → all shortest paths found.

---

### Dynamic Programming Perspective

Floyd-Warshall is a **DP algorithm**. Define:

```
dp[via][i][j] = shortest path from i to j using only {0, 1, ..., via} as intermediates
```

Base case: `dp[-1][i][j] = dist[i][j]` (direct edge or INF)

Recurrence:
```
dp[via][i][j] = min(
    dp[via-1][i][j],              ← don't use node `via`
    dp[via-1][i][via] + dp[via-1][via][j]  ← go through node `via`
)
```

The 3D DP collapses to a 2D array because `dp[via][i][via] = dp[via-1][i][via]` — including `via` as an intermediate doesn't change paths that START or END at `via`. So we can safely update in-place.

---

### The INF Guard

```cpp
if(dist[i][via] != INF && dist[via][j] != INF)
```

If either sub-path is unreachable (INF), computing `dist[i][via] + dist[via][j]` would overflow. This guard prevents overflow and nonsensical updates. Only consider the via-route if BOTH halves are reachable.

---

## 4. Dry Run

```
Initial dist:
     0    1    2    3
0 [  0    3   INF   7  ]
1 [ INF   0    2   INF ]
2 [ INF  INF   0    1  ]
3 [  6   INF  INF   0  ]

INF = 1e8
```

---

**Via = 0 (consider node 0 as intermediate):**

For every pair `(i,j)`, check if going through `0` helps.

```
(1,3): dist[1][0]=INF → skip (can't reach 0 from 1 directly)
(2,3): dist[2][0]=INF → skip
(3,1): dist[3][0]=6, dist[0][1]=3 → 6+3=9 < INF → dist[3][1]=9
(3,2): dist[3][0]=6, dist[0][2]=INF → skip
(3,3): dist[3][0]=6, dist[0][3]=7 → 6+7=13 > dist[3][3]=0 → no change

After via=0:
     0    1    2    3
0 [  0    3   INF   7  ]
1 [ INF   0    2   INF ]
2 [ INF  INF   0    1  ]
3 [  6    9   INF   0  ]   ← dist[3][1] updated to 9
```

---

**Via = 1 (consider node 1 as intermediate):**

```
(0,2): dist[0][1]=3, dist[1][2]=2 → 3+2=5 < INF → dist[0][2]=5
(0,3): dist[0][1]=3, dist[1][3]=INF → skip
(3,2): dist[3][1]=9, dist[1][2]=2 → 9+2=11 < INF → dist[3][2]=11
(3,0): skip (dist[3][1]=9, dist[1][0]=INF)

After via=1:
     0    1    2    3
0 [  0    3    5    7  ]   ← dist[0][2] = 5
1 [ INF   0    2   INF ]
2 [ INF  INF   0    1  ]
3 [  6    9   11    0  ]   ← dist[3][2] = 11
```

---

**Via = 2 (consider node 2 as intermediate):**

```
(0,3): dist[0][2]=5, dist[2][3]=1 → 5+1=6 < dist[0][3]=7 → dist[0][3]=6
(1,3): dist[1][2]=2, dist[2][3]=1 → 2+1=3 < INF → dist[1][3]=3
(3,3): dist[3][2]=11, dist[2][3]=1 → 11+1=12 > 0 → no change

After via=2:
     0    1    2    3
0 [  0    3    5    6  ]   ← dist[0][3] = 6
1 [ INF   0    2    3  ]   ← dist[1][3] = 3
2 [ INF  INF   0    1  ]
3 [  6    9   11    0  ]
```

---

**Via = 3 (consider node 3 as intermediate):**

```
(0,0): dist[0][3]=6, dist[3][0]=6 → 12 > 0 → no change
(1,0): dist[1][3]=3, dist[3][0]=6 → 9 < INF → dist[1][0]=9? 
       Wait: dist[1][0]=INF → 3+6=9 < INF → dist[1][0]=9
(2,0): dist[2][3]=1, dist[3][0]=6 → 1+6=7 < INF → dist[2][0]=7
(2,1): dist[2][3]=1, dist[3][1]=9 → 1+9=10 < INF → dist[2][1]=10
(3,0): dist[3][3]=0, dist[3][0]=6 → 0+6=6, not better

After via=3:
     0    1    2    3
0 [  0    3    5    6  ]
1 [  9    0    2    3  ]   ← dist[1][0] = 9
2 [  7   10    0    1  ]   ← dist[2][0] = 7, dist[2][1] = 10
3 [  6    9   11    0  ]

Final result ✅
```

---

## 5. Story Points

---

**Story Point 1 — "`via` must be outermost — not `i` or `j`"**

If `i` were outermost:
```
i=0: update dist[0][j] for all j
i=1: update dist[1][j] for all j (but dist[0][j] used here might not be optimal yet)
```
When computing paths for `i=1`, the intermediate routes involving `i=0` might not be finalized yet. The outer `via` loop guarantees that when we consider `via=k`, ALL shorter paths through nodes `{0..k-1}` are already computed. No other ordering provides this guarantee.

---

**Story Point 2 — "Floyd-Warshall works in-place because of the self-reference property"**

The recurrence uses `dist[i][via]` and `dist[via][j]`. When `i == via` or `j == via`:

- `dist[via][via]` = 0 (distance from any node to itself) — never changes
- Updating `dist[i][via]`: path from `i` to `via` through `via` = `dist[i][via] + dist[via][via]` = `dist[i][via] + 0` = no improvement
- Similarly for `dist[via][j]`

So paths starting or ending at `via` are never "corrupted" by processing `via` in the inner loop. The in-place update is mathematically sound.

---

**Story Point 3 — "Floyd-Warshall handles negative weights but NOT negative cycles (like Bellman-Ford)"**

Negative edge weights → perfectly fine, Floyd-Warshall handles them.

Negative cycle → `dist[i][i]` will become negative (a negative cycle through `i` creates a shortcut back to itself with negative cost). After running Floyd-Warshall, **check if any `dist[i][i] < 0`** → negative cycle detected.

```cpp
// Negative cycle detection for Floyd-Warshall:
for(int i = 0; i < n; i++) {
    if(dist[i][i] < 0) {
        // negative cycle exists
    }
}
```

---

**Story Point 4 — "Floyd-Warshall gives transitive closure for free"**

If we replace edge weights with `1` (exists) and `INF` (doesn't exist), then after Floyd-Warshall, `dist[i][j] < INF` means node `j` is **reachable** from node `i`. This is the **transitive closure** of the graph — which nodes can reach which other nodes.

---

**Story Point 5 — "O(V³) sounds slow but is simple and cache-friendly"**

Three nested loops over `V` → `O(V³)`. For `V=1000`, that's 10⁹ operations — potentially too slow. But for small `V` (say `V ≤ 500`), the simplicity and cache-friendliness of the 2D array operations make it competitive against running Dijkstra `V` times.

---

## 6. Code

```cpp
class Solution {
public:
    void floydWarshall(vector<vector<int>>& dist) {
        int n = dist.size();
        const int INF = 1e8;

        // Consider each node as a potential intermediate node
        for(int via = 0; via < n; via++) {

            // For every source node i
            for(int i = 0; i < n; i++) {

                // For every destination node j
                for(int j = 0; j < n; j++) {

                    // Only update if both halves of the path are reachable
                    // (guard against INF + wt = overflow)
                    if(dist[i][via] != INF && dist[via][j] != INF) {

                        // If going i → via → j is shorter than known i → j
                        dist[i][j] = min(dist[i][j],
                                         dist[i][via] + dist[via][j]);
                    }
                }
            }
        }
    }
};
```

---

## 7. Complexity Analysis

### Time Complexity — `O(V³)`

| Loop | Iterations | Reason |
|---|---|---|
| via loop | V | Consider each node as intermediate |
| i loop | V | Each source node |
| j loop | V | Each destination node |

**Total: `O(V × V × V)` = `O(V³)`**

Every pair `(i,j)` is updated once per `via` node → V pairs × V via nodes = V³ updates.

---

### Space Complexity — `O(V²)`

| Structure | Size | Reason |
|---|---|---|
| `dist[][]` (input/output) | `O(V²)` | N×N adjacency matrix |

**Total: `O(V²)`** — in-place update, no extra space needed.

---

## 8. Algorithm Comparison — All Shortest Path Algorithms

| Algorithm | Type | TC | SC | Negative Weights | Negative Cycle Detection | All Pairs? |
|---|---|---|---|---|---|---|
| **BFS** | Single source | `O(V+E)` | `O(V)` | ❌ | ❌ | ❌ |
| **Dijkstra** | Single source | `O((V+E)log V)` | `O(V+E)` | ❌ | ❌ | ❌ |
| **Bellman-Ford** | Single source | `O(V×E)` | `O(V)` | ✅ | ✅ | ❌ |
| **DAG Topo Sort** | Single source DAG | `O(V+E)` | `O(V)` | ✅ | N/A | ❌ |
| **Floyd-Warshall** | All pairs | `O(V³)` | `O(V²)` | ✅ | ✅ (if `dist[i][i]<0`) | ✅ |

> **Rule of thumb:** If you need shortest paths from ALL sources → Floyd-Warshall. If from ONE source with positive weights → Dijkstra. With negative weights → Bellman-Ford.
