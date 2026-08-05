# Path With Minimum Effort

---

## 1. Problem Statement

You are given an `n × m` matrix `heights` where `heights[i][j]` represents the height of cell `(i,j)`. A path from `(0,0)` to `(n-1, m-1)` consists of 4-directionally adjacent cells.

The **effort** of a path is defined as the **maximum absolute difference** in heights between any two consecutive cells on the path.

Return the **minimum effort** among all possible paths from top-left to bottom-right.

```
Heights:
[ 1   2   2 ]
[ 3   8   2 ]
[ 5   3   5 ]

Path 1: (0,0)→(0,1)→(0,2)→(1,2)→(2,2)
Differences: |1-2|=1, |2-2|=0, |2-2|=0, |2-5|=3 → effort = max = 3

Path 2: (0,0)→(1,0)→(2,0)→(2,1)→(2,2)
Differences: |1-3|=2, |3-5|=2, |5-3|=2, |3-5|=2 → effort = max = 2

Minimum effort = 2 ✅
```

---

## 2. Intuition / Approach

### The Key Insight — Redefine "Distance"

Standard Dijkstra minimizes the **sum** of edge weights along a path. This problem asks us to minimize the **maximum** edge weight along a path.

The edge weight between two adjacent cells `(r,c)` and `(r',c')` is:
```
|heights[r][c] - heights[r'][c']|
```

We need the path where this maximum difference is as small as possible.

**Can we still use Dijkstra?** Yes — but we need to change what `dist[r][c]` represents:

> `dist[r][c]` = **minimum possible effort to reach `(r,c)` from `(0,0)`**  
> = minimum over all paths of (maximum height difference along that path)

---

### The Modified "Relaxation" Step

In standard Dijkstra:
```
newDist = currentDist + edgeWeight   (sum)
```

Here:
```cpp
newEffort = max(abs(heights[row][col] - heights[nrow][ncol]), diff)
```

- `abs(heights[row][col] - heights[nrow][ncol])` = height difference for this single step
- `diff` = maximum height difference encountered so far on the path to `(row,col)`
- `newEffort` = **new maximum** if we extend the path to `(nrow, ncol)`

We update `dist[nrow][ncol]` only if `newEffort < dist[nrow][ncol]` — i.e., we found a path to `(nrow,ncol)` with a smaller maximum difference.

---

### Why Dijkstra Still Works Here

Dijkstra's correctness relies on one property: **the "distance" function must be monotonically non-decreasing as we extend a path**.

- In standard Dijkstra: adding a non-negative weight to path sum → sum never decreases ✅
- Here: taking `max(currentMaxDiff, newDiff)` → the maximum never decreases as we extend ✅

Since effort only stays the same or increases as we extend a path, the greedy "process minimum effort first" strategy is still valid. When a cell is popped from the min-heap, its effort is finalized — no future extension can give it a smaller maximum difference.

---

### `dist[0][0] = 0` — Why 0 This Time?

Unlike the binary matrix problem where path length counted cells, here the **effort** of reaching `(0,0)` from itself is `0` — no height differences encountered. We haven't moved yet, so the maximum difference on the empty path is `0`.

---

### Early Exit at Destination Pop

```cpp
if(row == n-1 && col == m-1)
    return diff;
```

When the destination is popped from the min-heap, `diff` contains the minimum effort to reach it — guaranteed by Dijkstra's finalization. Return immediately.

No need to check `dist[n-1][m-1]` at the end — if reachable, we'll always pop it during the loop.

---

## 3. Dry Run

```
Heights:
[ 1  2  2 ]
[ 3  8  2 ]
[ 5  3  5 ]

n=3, m=3
dist = all INT_MAX, dist[0][0]=0
pq   = [{0, {0,0}}]
```

---

**Pop {0, (0,0)}:**
```
(0,0) ≠ destination → process

Neighbors:
  up    (-1,0): OOB
  right (0,1):  |heights[0][0]-heights[0][1]| = |1-2| = 1
                newEffort = max(1, 0) = 1 < INF → dist[0][1]=1, push {1,(0,1)}
  down  (1,0):  |1-3| = 2
                newEffort = max(2, 0) = 2 < INF → dist[1][0]=2, push {2,(1,0)}
  left  (0,-1): OOB

dist:
[ 0   1  INF ]
[ 2  INF INF ]
[INF INF INF ]
pq = [{1,(0,1)}, {2,(1,0)}]
```

**Pop {1, (0,1)}:**
```
Neighbors:
  up    (-1,1): OOB
  right (0,2):  |2-2| = 0, newEffort = max(0, 1) = 1 < INF → dist[0][2]=1, push {1,(0,2)}
  down  (1,1):  |2-8| = 6, newEffort = max(6, 1) = 6 < INF → dist[1][1]=6, push {6,(1,1)}
  left  (0,0):  newEffort = max(1, 1) = 1 < dist[0][0]=0? NO

dist:
[ 0  1   1  ]
[ 2  6  INF ]
[INF INF INF]
pq = [{1,(0,2)}, {2,(1,0)}, {6,(1,1)}]
```

**Pop {1, (0,2)}:**
```
Neighbors:
  right (0,3): OOB
  down  (1,2): |2-2| = 0, newEffort = max(0,1) = 1 < INF → dist[1][2]=1, push {1,(1,2)}
  left  (0,1): max(0,1)=1 < dist[0][1]=1? NO
  up    (-1,2): OOB

dist:
[ 0  1  1 ]
[ 2  6  1 ]
[INF INF INF]
pq = [{1,(1,2)}, {2,(1,0)}, {6,(1,1)}]
```

**Pop {1, (1,2)}:**
```
Neighbors:
  up    (0,2): max(0,1)=1 < 1? NO
  right (1,3): OOB
  down  (2,2): |2-5|=3, newEffort=max(3,1)=3 < INF → dist[2][2]=3, push {3,(2,2)}
  left  (1,1): |2-8|=6, newEffort=max(6,1)=6 < dist[1][1]=6? NO

pq = [{2,(1,0)}, {3,(2,2)}, {6,(1,1)}]
```

**Pop {2, (1,0)}:**
```
  up    (0,0): max(2,2)=2 < 0? NO
  right (1,1): |3-8|=5, newEffort=max(5,2)=5 < dist[1][1]=6? YES → dist[1][1]=5, push {5,(1,1)}
  down  (2,0): |3-5|=2, newEffort=max(2,2)=2 < INF → dist[2][0]=2, push {2,(2,0)}
  left  (1,-1): OOB

dist:
[ 0  1  1 ]
[ 2  5  1 ]
[ 2  INF 3]
pq = [{2,(2,0)}, {3,(2,2)}, {5,(1,1)}, {6,(1,1)}]
```

**Pop {2, (2,0)}:**
```
  up    (1,0): max(2,2)=2 < 2? NO
  right (2,1): |5-3|=2, newEffort=max(2,2)=2 < INF → dist[2][1]=2, push {2,(2,1)}
  down  (3,0): OOB

pq = [{2,(2,1)}, {3,(2,2)}, {5,(1,1)}, {6,(1,1)}]
```

**Pop {2, (2,1)}:**
```
  right (2,2): |3-5|=2, newEffort=max(2,2)=2 < dist[2][2]=3? YES → dist[2][2]=2, push {2,(2,2)}
  up    (1,1): max(2,2)=2 < dist[1][1]=5? YES → dist[1][1]=2, push {2,(1,1)}
  left  (2,0): max(2,2)=2 < 2? NO

pq = [{2,(2,2)}, {2,(1,1)}, {3,(2,2)}, {5,(1,1)}, {6,(1,1)}]
```

**Pop {2, (2,2)}:**
```
row=2==n-1=2, col=2==m-1=2 → DESTINATION → return 2 ✅
```

---

## 4. Story Points

---

**Story Point 1 — "This is Dijkstra but with max instead of sum"**

Standard Dijkstra minimizes **sum of weights**. This problem minimizes **maximum of weights** along a path. The only change in code is the relaxation step:

```cpp
// Standard Dijkstra:
newDist = dist[node] + edgeWeight

// This problem:
newEffort = max(edgeWeight, diff)   ← max instead of +
```

The min-heap, stale entry check, and early exit remain identical. This is a powerful insight — Dijkstra generalizes beyond sum to any monotonically non-decreasing path cost function.

---

**Story Point 2 — "Why max() preserves Dijkstra's correctness"**

Dijkstra needs: extending a path never decreases its cost.

- Sum: `a + b ≥ a` for `b ≥ 0` ✅
- Max: `max(a, b) ≥ a` always ✅

Both satisfy the monotonicity requirement. So Dijkstra's greedy guarantee (minimum cost entry popped = finalized) holds for both.

This would FAIL for subtraction or division — those can decrease the path cost, breaking the greedy argument.

---

**Story Point 3 — "`diff` in the heap = maximum effort seen so far on path to this cell"**

The heap stores `{effort, {row, col}}` where `effort` = maximum height difference encountered on the best-known path to `(row, col)`. When we pop a cell, `diff` is this stored effort — we use it to compute the new effort when extending to neighbors:

```cpp
newEffort = max(abs(heights[row][col] - heights[nrow][ncol]), diff)
```

The `diff` carries the "path history" (the running maximum) compactly as a single number.

---

**Story Point 4 — "No stale entry `continue` check — but it's implicitly handled"**

Wait — this code has no explicit `if(distance > dist[row][col]) continue;` stale check! How?

Because the check `newEffort < dist[nrow][ncol]` at push time prevents pushing many stale entries. And at the early exit `if(row == n-1 && col == m-1) return diff;`, we return immediately — so even if stale entries exist, we stop as soon as we pop the destination with its optimal effort.

For cells other than the destination, stale entries will have `diff > dist[row][col]` — they'll attempt relaxations that all fail the `newEffort < dist[nrow][ncol]` check, so no incorrect updates occur. Functionally safe.

---

**Story Point 5 — "Binary search + BFS is an alternative O(N² log(max_height)) approach"**

An alternative: binary search on the answer (effort value `k` from 0 to max possible difference), and for each `k`, run BFS to check if a path exists where all consecutive differences are ≤ `k`.

```
Binary search: O(log(max_height))
BFS per check: O(N × M)
Total: O(N × M × log(max_height))
```

Dijkstra: `O(N × M × log(N × M))`

For large heights, binary search + BFS can be faster. But Dijkstra is a single clean pass.

---

## 5. Code

```cpp
class Solution {
public:
    using Pair = pair<int, pair<int, int>>;  // {effort, {row, col}}

    int minimumEffortPath(vector<vector<int>>& heights) {
        int n = heights.size();
        int m = heights[0].size();

        // dist[r][c] = min effort to reach (r,c) from (0,0)
        vector<vector<int>> dist(n, vector<int>(m, INT_MAX));
        dist[0][0] = 0;   // 0 effort to reach source (no moves yet)

        // Min-heap ordered by effort (smallest effort processed first)
        priority_queue<Pair, vector<Pair>, greater<Pair>> pq;
        pq.push({0, {0, 0}});

        const int drow[] = {-1, 0, +1, 0};
        const int dcol[] = { 0,+1,  0,-1};

        while(!pq.empty()) {
            auto [diff, cell] = pq.top();
            auto [row, col] = cell;
            pq.pop();

            // Destination reached — diff is the minimum effort (Dijkstra guarantee)
            if(row == n-1 && col == m-1)
                return diff;

            for(int i = 0; i < 4; i++) {
                int nrow = row + drow[i];
                int ncol = col + dcol[i];

                if(nrow < 0 || ncol < 0 || nrow >= n || ncol >= m)
                    continue;

                // New effort = max(current path max, this step's difference)
                int newEffort = max(abs(heights[row][col] - heights[nrow][ncol]), diff);

                // Relax: update if we found a path with smaller max effort
                if(newEffort < dist[nrow][ncol]) {
                    dist[nrow][ncol] = newEffort;
                    pq.push({newEffort, {nrow, ncol}});
                }
            }
        }

        return 0;   // grid is always connected, so this is never reached
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M × log(N × M))`

| Step | Cost | Reason |
|---|---|---|
| Initialize `dist` | `O(N × M)` | N×M grid |
| Each cell enqueued/dequeued | `O(N × M × log(N × M))` | At most N×M cells, each with `O(log(NM))` heap op |
| 4 direction relaxations per cell | `O(4)` = `O(1)` | Constant |

**Total: `O(N × M × log(N × M))`**

---

### Space Complexity — `O(N × M)`

| Structure | Size |
|---|---|
| `dist[][]` | `O(N × M)` |
| Priority queue | `O(N × M)` worst case |

**Total: `O(N × M)`**

---

### Standard Dijkstra vs This Problem

| Aspect | Standard Dijkstra | Minimum Effort |
|---|---|---|
| **What `dist[node]` stores** | Min sum of weights to reach node | Min max-difference to reach cell |
| **Relaxation formula** | `dist[node] + edgeWeight` | `max(edgeWeight, currentDiff)` |
| **Why Dijkstra works** | Sum is monotone (+ non-neg) | Max is monotone (always ≥ current) |
| **`dist[src]`** | `0` (0 edges traversed) | `0` (0 differences seen) |
| **Direction count** | Graph edges | 4-directional |
