# Dijkstra's Algorithm — Set-Based Implementation

---

## 1. Problem Statement

Same as the priority queue version — find shortest distances from `src` to all nodes in a weighted undirected graph. This is an **alternate implementation** of Dijkstra using `std::set` instead of `priority_queue`.

---

## 2. Intuition — Why Use `set` Instead of `priority_queue`?

### Recall: The Priority Queue Problem

In the priority queue version, when we find a shorter distance to a node, we push a **new entry** with the updated distance. The old (stale) entry stays in the heap. We then check `distance > dist[node]` when popping to discard stale entries.

**Drawback:** The heap can accumulate many stale entries → up to `O(E)` entries in the worst case → larger heap → slightly worse constant factors.

---

### The `set` Solution — True Decrease-Key

`std::set` in C++ is a **sorted BST** (Red-Black Tree) that keeps all elements sorted at all times. Crucially, it supports `O(log n)` **deletion of any element by value**.

This lets us implement a **true decrease-key operation**:
1. Find old `{dist[neighbor], neighbor}` entry in the set → erase it `O(log n)`
2. Insert new `{newDist, neighbor}` entry → `O(log n)`

No stale entries. The set always contains exactly **one entry per node** — the current best known distance.

---

### `set` vs `priority_queue` — Side by Side

| | `priority_queue` | `set` |
|---|---|---|
| **Data structure** | Binary heap | Red-Black Tree (BST) |
| **Minimum element** | `pq.top()` | `*st.begin()` |
| **Insert** | `pq.push()` | `st.emplace()` |
| **Delete min** | `pq.pop()` | `st.erase(st.begin())` |
| **Delete specific element** | ❌ Not supported efficiently | ✅ `st.erase({dist, node})` — O(log n) |
| **Stale entries?** | Yes — need `continue` check | No — erase old entry on update |
| **Max entries in structure** | Up to O(E) | Exactly V at any time |
| **TC per operation** | `O(log E)` | `O(log V)` |
| **Overall TC** | `O((V+E) log E)` | `O((V+E) log V)` |

The `set` approach is **theoretically cleaner** — no stale entries, no `continue` check needed conceptually. However, the `continue` check remains in the code as a safety guard (see Story Points).

---

### Why `set<pair<int,int>>` Instead of `set<int>`?

We store `{distance, node}` pairs. The set orders by distance first (smallest distance = `begin()`). This is the same as the priority queue where we put distance first for min-ordering.

If we only stored node indices, we'd have no way to order by distance.

---

## 3. Dry Run

```
V=4, src=0
Edges: 0-1(4), 0-2(1), 2-1(2), 1-3(1)

adjLs:
0 → [(1,4), (2,1)]
1 → [(0,4), (2,2), (3,1)]
2 → [(0,1), (1,2)]
3 → [(1,1)]

dist = [0, INF, INF, INF]
st   = {(0,0)}
```

---

**Iteration 1 — Extract min {0,0}:**
```
erase (0,0) → st = {}
distance=0, node=0, 0 > dist[0]=0? NO → process

neighbor 1, wt=4: newDist=4 < INF
  dist[1]=INF (skip erase) → dist[1]=4, insert (4,1)
neighbor 2, wt=1: newDist=1 < INF
  dist[2]=INF (skip erase) → dist[2]=1, insert (1,2)

dist = [0, 4, 1, INF]
st   = {(1,2), (4,1)}
```

**Iteration 2 — Extract min {1,2}:**
```
erase (1,2) → st = {(4,1)}
distance=1, node=2, 1 > dist[2]=1? NO → process

neighbor 0, wt=1: newDist=2 < dist[0]=0? NO
neighbor 1, wt=2: newDist=3 < dist[1]=4? YES
  dist[1]=4 ≠ INF → erase (4,1) from st → st = {}
  dist[1]=3, insert (3,1) → st = {(3,1)}

dist = [0, 3, 1, INF]
st   = {(3,1)}
```

**Iteration 3 — Extract min {3,1}:**
```
erase (3,1) → st = {}
distance=3, node=1, 3 > dist[1]=3? NO → process

neighbor 0, wt=4: newDist=7 < dist[0]=0? NO
neighbor 2, wt=2: newDist=5 < dist[2]=1? NO
neighbor 3, wt=1: newDist=4 < dist[3]=INF? YES
  dist[3]=INF (skip erase) → dist[3]=4, insert (4,3)

dist = [0, 3, 1, 4]
st   = {(4,3)}
```

**Iteration 4 — Extract min {4,3}:**
```
erase (4,3) → st = {}
distance=4, node=3, 4 > dist[3]=4? NO → process
neighbor 1, wt=1: newDist=5 < dist[1]=3? NO

st = {} → done

Final dist = [0, 3, 1, 4] ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Erase old entry BEFORE inserting new one"**

```cpp
if(dist[adjNode] != INT_MAX)
    st.erase({dist[adjNode], adjNode});   // remove stale entry

dist[adjNode] = newDistance;
st.emplace(newDistance, adjNode);         // insert fresh entry
```

Order matters. We must erase the old `{dist[adjNode], adjNode}` entry first (using the OLD distance value to find it in the set). If we updated `dist[adjNode]` first, we'd lose the old distance value and couldn't find the entry to erase.

Also: we only erase if `dist[adjNode] != INT_MAX`. If the node hasn't been discovered yet, there's no entry in the set to erase.

---

**Story Point 2 — "Why does the `continue` check still exist?"**

```cpp
if(distance > dist[node])
    continue;
```

In theory, since we erase stale entries, this check should never trigger. In practice, it's kept as a **defensive guard** for correctness in edge cases.

But here's a subtle scenario where it could matter: if the same `{distance, node}` pair is somehow processed twice (shouldn't happen with a set, but the guard adds safety). In the `priority_queue` version, this check is essential. In the `set` version, it's optional but harmless.

---

**Story Point 3 — "`set::begin()` gives the minimum — not `set::end()`"**

```cpp
auto it = st.begin();
auto [distance, node] = *it;
st.erase(it);
```

`std::set` is ordered in ascending order by default. So `begin()` points to the smallest element — the `{distance, node}` pair with the smallest distance. This is our min-heap equivalent.

If you accidentally used `st.end()` (or `rbegin()`), you'd get the maximum → process farthest node first → wrong algorithm entirely.

---

**Story Point 4 — "Set stores exactly V entries at most — no bloat"**

In the priority_queue version, every relaxation pushes a new entry — potentially `O(E)` entries total. The set version erases the old entry on every relaxation, so the set always has **at most one entry per node**. Max size = V at any time. This is why the TC uses `log V` instead of `log E`.

---

**Story Point 5 — "`using PII = pair<int,int>` — type aliasing for readability"**

```cpp
using PII = pair<int, int>;
```

`vector<vector<pair<int,int>>>` is verbose. Aliasing to `PII` makes declarations cleaner. This is a C++ style choice — no functional difference. The structured binding `auto [distance, node] = *it` also improves readability over `.first` and `.second`.

---

## 5. Code

```cpp
class Solution {
public:
    vector<int> dijkstra(int V, vector<vector<int>>& edges, int src) {
        using PII = pair<int,int>;

        // Step 1: Build undirected weighted adjacency list
        vector<vector<PII>> adjLs(V);
        for(auto& it : edges) {
            adjLs[it[0]].emplace_back(it[1], it[2]);
            adjLs[it[1]].emplace_back(it[0], it[2]);
        }

        // Set stores {distance, node} — ordered by distance (begin = min)
        set<PII> st;
        st.emplace(0, src);

        vector<int> dist(V, INT_MAX);
        dist[src] = 0;

        while(!st.empty()) {
            // Extract node with smallest current distance
            auto it = st.begin();
            auto [distance, node] = *it;
            st.erase(it);

            // Safety guard (theoretically not needed with set)
            if(distance > dist[node]) continue;

            for(const auto& [adjNode, edgeWt] : adjLs[node]) {
                int newDistance = distance + edgeWt;

                if(newDistance < dist[adjNode]) {
                    // True decrease-key: erase old entry, insert updated one
                    if(dist[adjNode] != INT_MAX)
                        st.erase({dist[adjNode], adjNode});   // erase BEFORE updating dist

                    dist[adjNode] = newDistance;
                    st.emplace(newDistance, adjNode);
                }
            }
        }

        return dist;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O((V + E) log V)`

| Step | Cost | Reason |
|---|---|---|
| Build adjacency list | `O(V + E)` | 2E entries for undirected |
| Extract min (`st.begin()` + erase) | `O(V log V)` | At most V extractions, each `O(log V)` |
| Relaxation (erase + insert per edge) | `O(E log V)` | At most E relaxations, each erase+insert = `O(log V)` |

**Total: `O((V + E) log V)`**

> Unlike the priority_queue version which is `O((V+E) log E)`, this is truly `O((V+E) log V)` because the set size is bounded by V (no stale entries).

---

### Space Complexity — `O(V + E)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(V + 2E)` | Undirected edges stored twice |
| `dist[]` | `O(V)` | One per node |
| `set` | `O(V)` | At most V entries (one per node, no stale entries) |

**Total: `O(V + E)`**

---

## 7. `set` vs `priority_queue` — Full Comparison

| Aspect | `priority_queue` | `set` |
|---|---|---|
| **Min extraction** | `pq.top()` + `pq.pop()` | `*st.begin()` + `st.erase(begin)` |
| **Decrease key** | Push new + skip stale | Erase old + insert new |
| **Stale entries** | Yes — O(E) in worst case | No — always ≤ V entries |
| **`continue` check** | Essential | Technically optional (kept as guard) |
| **TC** | `O((V+E) log E)` | `O((V+E) log V)` |
| **SC** | `O(E)` for heap | `O(V)` for set |
| **Code complexity** | Simpler | Slightly more complex (erase step) |
| **Practical speed** | Often faster (cache-friendly) | Theoretically cleaner |
| **Industry preference** | Most competitive programming | Preferred when memory matters |

> **Which to use?** In competitive programming, `priority_queue` is standard due to simplicity. The `set` version is theoretically superior (smaller TC/SC bounds) and is closer to how Dijkstra was originally described with a proper "decrease-key" priority queue.
