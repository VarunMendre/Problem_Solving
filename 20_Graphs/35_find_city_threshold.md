# Find the City With the Smallest Number of Neighbors at a Threshold Distance

---

## 1. Problem Statement

There are `n` cities numbered `0` to `n-1` with bidirectional weighted edges between some pairs. Given a `distanceThreshold`, find the city that has the **smallest number of cities reachable within `distanceThreshold` distance**.

If multiple cities have the same minimum count, return the one with the **largest index**.

```
n=4, distanceThreshold=4
Edges: 0-1(3), 1-2(1), 1-3(4), 2-3(1)

After Floyd-Warshall (all-pairs shortest paths):
      0   1   2   3
  0 [ 0   3   4   4 ]    cities reachable ≤ 4: {1,2,3} → cnt=3 (exclude self? no, dist[0][0]=0≤4)
  1 [ 3   0   1   2 ]    cities ≤ 4: {0,2,3} → cnt=3
  2 [ 4   1   0   1 ]    cities ≤ 4: {0,1,3} → cnt=3
  3 [ 4   2   1   0 ]    cities ≤ 4: {1,2} → cnt=2  ← fewest!

Actually count includes self (dist[i][i]=0≤threshold):
City 0: 0,1,2,3 → 4 cities
City 1: 0,1,2,3 → 4 cities
City 2: 0,1,2,3 → 4 cities
City 3: 1,2,3 → 3 cities (dist[3][0]=4, only if ≤4 yes but let's say threshold=4, all are 4 or less)

Hmm let me use threshold=3 for a clearer example:
City 0: dist ≤3: {0(0), 1(3)} → cnt=2
City 1: dist ≤3: {0(3), 1(0), 2(1), 3(2)} → cnt=4
City 2: dist ≤3: {1(1), 2(0), 3(1)} → cnt=3
City 3: dist ≤3: {1(2), 2(1), 3(0)} → cnt=3

City 0 has fewest (2). Answer: 0
```

---

## 2. Intuition / Approach

### Why Floyd-Warshall?

The problem requires the **shortest distance from every city to every other city** — this is the All Pairs Shortest Path (APSP) problem. Floyd-Warshall solves exactly this in `O(n³)`.

After Floyd-Warshall, `dist[i][j]` = shortest distance between cities `i` and `j`. Then for each city, count how many cities are within `distanceThreshold`.

---

### The Two Phases

```
Phase 1 — Floyd-Warshall: Compute all-pairs shortest paths
Phase 2 — Count & Compare: For each city, count reachable cities within threshold
                           Track the city with minimum count (prefer largest index)
```

---

### The `≤ cntCity` Instead of `< cntCity` for Largest Index

```cpp
if(cnt <= cntCity) {
    cntCity = cnt;
    cityNo = city;
}
```

Note the `<=` (not `<`). This means: if multiple cities have the same minimum count, we UPDATE to the later one. Since `city` iterates from `0` to `n-1` in order, the LAST city with the minimum count wins → returns the **largest index** among ties.

If it were `<`, we'd keep the first (smallest index) city. The `<=` ensures the largest index requirement is met.

---

### Why Count INCLUDES Self (`dist[i][i] = 0`)?

`dist[i][i] = 0 ≤ distanceThreshold` (threshold is always positive), so city `i` always counts itself. This means the count represents "total cities within threshold including self". Since every city counts itself, it doesn't change the relative comparison — the city with the fewest reachable neighbors still wins.

---

## 3. Dry Run

```
n=4, distanceThreshold=4
Edges: 0-1(3), 1-2(1), 1-3(4), 2-3(1)

Initialize dist:
     0     1     2     3
0 [  0     3    INF   INF ]
1 [  3     0     1     4  ]
2 [ INF    1     0     1  ]
3 [ INF    4     1     0  ]

(dist[i][i]=0 for all i)
```

---

**Floyd-Warshall:**

**Via=0:**
```
No pair benefits from going through 0:
  (1,2): dist[1][0]=3, dist[0][2]=INF → skip
  (1,3): 3+INF → skip
  (2,3): INF+INF → skip
No changes.
```

**Via=1:**
```
(0,2): dist[0][1]=3, dist[1][2]=1 → 3+1=4 < INF → dist[0][2]=4
(0,3): dist[0][1]=3, dist[1][3]=4 → 3+4=7 < INF → dist[0][3]=7? 
       Wait: dist[0][3]=INF → 7 < INF → dist[0][3]=7
(2,0): dist[2][1]=1, dist[1][0]=3 → 4 < INF → dist[2][0]=4
(3,0): dist[3][1]=4, dist[1][0]=3 → 7 < INF → dist[3][0]=7
(3,2): dist[3][1]=4, dist[1][2]=1 → 5 > dist[3][2]=1 → no change

After via=1:
     0     1     2     3
0 [  0     3     4     7  ]
1 [  3     0     1     4  ]
2 [  4     1     0     1  ]
3 [  7     4     1     0  ]
```

**Via=2:**
```
(0,3): dist[0][2]=4, dist[2][3]=1 → 5 < 7 → dist[0][3]=5
(1,3): dist[1][2]=1, dist[2][3]=1 → 2 < 4 → dist[1][3]=2
(3,0): dist[3][2]=1, dist[2][0]=4 → 5 < 7 → dist[3][0]=5
(3,1): dist[3][2]=1, dist[2][1]=1 → 2 < 4 → dist[3][1]=2

After via=2:
     0     1     2     3
0 [  0     3     4     5  ]
1 [  3     0     1     2  ]
2 [  4     1     0     1  ]
3 [  5     2     1     0  ]
```

**Via=3:**
```
(0,1): dist[0][3]=5, dist[3][1]=2 → 7 > 3 → no change
(0,2): dist[0][3]=5, dist[3][2]=1 → 6 > 4 → no change
(1,0): dist[1][3]=2, dist[3][0]=5 → 7 > 3 → no change
(2,0): dist[2][3]=1, dist[3][0]=5 → 6 > 4 → no change
No improvements.

Final dist:
     0     1     2     3
0 [  0     3     4     5  ]
1 [  3     0     1     2  ]
2 [  4     1     0     1  ]
3 [  5     2     1     0  ]
```

---

**Phase 2 — Count cities within threshold=4:**

```
City 0: dist[0] = [0, 3, 4, 5]
  ≤ 4: 0(0✅), 1(3✅), 2(4✅), 3(5❌) → cnt=3
  cntCity=4(n) → 3≤4 → cntCity=3, cityNo=0

City 1: dist[1] = [3, 0, 1, 2]
  ≤ 4: 0(3✅), 1(0✅), 2(1✅), 3(2✅) → cnt=4
  4≤3? NO → no update

City 2: dist[2] = [4, 1, 0, 1]
  ≤ 4: 0(4✅), 1(1✅), 2(0✅), 3(1✅) → cnt=4
  4≤3? NO → no update

City 3: dist[3] = [5, 2, 1, 0]
  ≤ 4: 0(5❌), 1(2✅), 2(1✅), 3(0✅) → cnt=3
  3≤3? YES (≤ not <) → cntCity=3, cityNo=3

Answer: cityNo=3 ✅
(City 3 has same count as city 0, but largest index wins)
```

---

## 4. Story Points

---

**Story Point 1 — "`INT_MAX` guard vs `1e8` guard"**

This code uses `INT_MAX` (unlike the previous Floyd-Warshall which used `1e8`). The guard is:
```cpp
if(dist[i][via] == INT_MAX || dist[via][j] == INT_MAX)
    continue;
```

This is SAFER than `1e8` because `INT_MAX + any_value` overflows. By using `continue` (skip) instead of the addition, we completely avoid the overflow. The previous version's `!= 1e8` approach works but `1e8` is an arbitrary sentinel — `INT_MAX` + guard is more correct.

---

**Story Point 2 — "`<=` for ties → largest index wins"**

```cpp
if(cnt <= cntCity) {   // ← note <=, not <
```

Scanning cities `0 → n-1`, we UPDATE whenever we find a city with count ≤ current minimum. Ties get overwritten by later (larger index) cities. Final `cityNo` = largest index city among those with minimum neighbor count.

---

**Story Point 3 — "Phase 1 is Floyd-Warshall, Phase 2 is just a scan — O(n²) total for phase 2"**

The counting phase is `O(n²)`:
- Outer loop: `n` cities
- Inner loop: `n` cities each
- Total: `n²` comparisons

This is dominated by Floyd-Warshall's `O(n³)`. Phase 2 adds no asymptotic overhead.

---

**Story Point 4 — "This problem could also be solved with Dijkstra from each node"**

Run Dijkstra from each city → `n` times Dijkstra. For sparse graphs:
- `n × O((V+E) log V)` = `O(n(n+E) log n)`

For dense graphs (`E ≈ n²`):
- `O(n³ log n)` — worse than Floyd-Warshall's `O(n³)`

Floyd-Warshall is simpler AND faster for this APSP problem.

---

**Story Point 5 — "Initializing `dist[i][i] = 0` is crucial"**

Without setting diagonal to 0, `dist[i][i]` would start as `INT_MAX`. During Floyd-Warshall:
- `dist[i][i]` could be updated to some path cost (going around a cycle)
- More importantly, the path `i → via → ... → via → i` might corrupt results

Setting `dist[i][i] = 0` correctly represents "no cost to stay at the same city" and anchors Floyd-Warshall's correctness.

---

## 5. Code

```cpp
class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {

        // Initialize dist matrix
        vector<vector<int>> dist(n, vector<int>(n, INT_MAX));

        // Set edge weights
        for(auto& it : edges) {
            dist[it[0]][it[1]] = it[2];
            dist[it[1]][it[0]] = it[2];   // undirected
        }

        // Set self-distance to 0
        for(int i = 0; i < n; i++)
            dist[i][i] = 0;

        // Phase 1: Floyd-Warshall — all pairs shortest paths
        for(int via = 0; via < n; via++) {
            for(int i = 0; i < n; i++) {
                for(int j = 0; j < n; j++) {
                    // Guard: skip if either half of path is unreachable
                    if(dist[i][via] == INT_MAX || dist[via][j] == INT_MAX)
                        continue;

                    dist[i][j] = min(dist[i][j], dist[i][via] + dist[via][j]);
                }
            }
        }

        // Phase 2: Find city with fewest reachable cities within threshold
        int cntCity = n;    // start with max possible count
        int cityNo  = -1;

        for(int city = 0; city < n; city++) {
            int cnt = 0;
            for(int adjCity = 0; adjCity < n; adjCity++) {
                if(dist[city][adjCity] <= distanceThreshold)
                    cnt++;
            }

            // <= ensures largest index wins in case of tie
            if(cnt <= cntCity) {
                cntCity = cnt;
                cityNo  = city;
            }
        }

        return cityNo;
    }
};
```

---

## 6. Dijkstra-Based Approach

Instead of computing all-pairs shortest paths with Floyd-Warshall, run Dijkstra separately from each city.

```cpp
class Solution {
private:
    vector<int> dijkstra(int src, int n,
                         vector<vector<pair<int,int>>>& adjLs) {
        vector<int> dist(n, INT_MAX);
        dist[src] = 0;

        priority_queue<pair<int,int>,
                       vector<pair<int,int>>,
                       greater<>> pq;
        pq.push({0, src});

        while(!pq.empty()) {
            auto [d, node] = pq.top(); pq.pop();
            if(d > dist[node]) continue;

            for(auto& [nbr, wt] : adjLs[node]) {
                if(dist[node] + wt < dist[nbr]) {
                    dist[nbr] = dist[node] + wt;
                    pq.push({dist[nbr], nbr});
                }
            }
        }
        return dist;
    }

public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
        vector<vector<pair<int,int>>> adjLs(n);

        for(auto& it : edges) {
            adjLs[it[0]].push_back({it[1], it[2]});
            adjLs[it[1]].push_back({it[0], it[2]});
        }

        int cntCity = n, cityNo = -1;

        // Run Dijkstra from each city
        for(int city = 0; city < n; city++) {
            vector<int> dist = dijkstra(city, n, adjLs);

            int cnt = 0;
            for(int adjCity = 0; adjCity < n; adjCity++) {
                if(dist[adjCity] <= distanceThreshold)
                    cnt++;
            }

            if(cnt <= cntCity) {
                cntCity = cnt;
                cityNo  = city;
            }
        }

        return cityNo;
    }
};
```

---

## 7. Complexity Analysis

### Floyd-Warshall Approach

| Step | Cost | Reason |
|---|---|---|
| Initialize matrix | `O(n²)` | n×n grid |
| Floyd-Warshall | `O(n³)` | Triple nested loop |
| Count phase | `O(n²)` | n cities × n neighbors |

**Total TC: `O(n³)`**  
**Total SC: `O(n²)`** — n×n distance matrix

---

### Dijkstra Approach

| Step | Cost | Reason |
|---|---|---|
| Single Dijkstra | `O((n+E) log n)` | Standard Dijkstra |
| Run n times | `O(n × (n+E) log n)` | One per source city |
| Count phase | `O(n²)` | n × n |

**Total TC: `O(n(n+E) log n)`**  
**Total SC: `O(n + E)`** — adjacency list + dist array per run

---

### Which to Use?

| | Floyd-Warshall | Dijkstra (n times) |
|---|---|---|
| **Dense graph** (`E≈n²`) | `O(n³)` ✅ | `O(n³ log n)` ❌ |
| **Sparse graph** (`E≈n`) | `O(n³)` ❌ | `O(n² log n)` ✅ |
| **Code simplicity** | Very simple | More code (helper function) |
| **Negative weights** | ✅ | ❌ |
| **Memory** | `O(n²)` | `O(n + E)` per run |

> For this problem: `n ≤ 100` (LeetCode constraint) → both approaches easily within time. Floyd-Warshall is simpler. For larger n with sparse edges, Dijkstra wins.
