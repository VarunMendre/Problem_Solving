# Minimum Steps to Reach End (Multiplication mod 1000)

---

## 1. Problem Statement

You are given an array `arr` of integers, a `start` value, and an `end` value. In each step, you can multiply the **current number** by **any element in `arr`**, then take the result **modulo 1000**.

Find the **minimum number of steps** to reach `end` from `start`. Return `-1` if impossible.

```
arr = [2, 5], start = 3, end = 500

Step 1: 3 × 2 = 6
Step 2: 6 × 5 = 30
Step 3: 30 × 5 = 150
Step 4: 150 × 5 = 750 (mod 1000 = 750)
Step 5: 750 × 2 = 1500 (mod 1000 = 500) ← reached end!

Minimum steps = 5
```

---

## 2. Intuition / Approach

### Modelling as a Graph Problem

Think of each number from `0` to `999` (since we always take `mod 1000`) as a **node** in a graph. An **edge** exists from node `u` to node `v` if:

```
v = (u × arr[i]) % 1000   for some arr[i]
```

Each step (multiply by one element of `arr`) costs 1. We need the **minimum number of steps** from `start` to `end` — this is **shortest path in an unweighted graph → BFS**.

---

### Why `mod = 1000`?

The problem constrains all values to be taken modulo 1000. This means:
- The state space is bounded: only values `0` to `999` can ever appear
- Total possible "nodes" in our graph = 1000
- After any multiplication, the result is `% 1000` → stays in `[0, 999]`

This is crucial — without the mod, numbers could grow infinitely. The mod bounds the state space to 1000 nodes, making BFS feasible.

---

### Why BFS and Not Dijkstra?

Every step costs 1 (multiply by one `arr[i]` = one step). All edges have equal weight. For unweighted graphs, BFS finds the shortest path in `O(V + E)` which is faster than Dijkstra's `O((V+E) log V)`.

BFS level = number of steps taken. The first time we reach `end`, it's via the minimum number of steps — BFS guarantee.

---

### The `dist[]` Array

`dist[num]` = minimum steps to reach `num` from `start`.

- `dist[start] = 0`
- `dist[all others] = INT_MAX` (unreached)

When we find `steps + 1 < dist[num]` → we found a shorter path to `num` → update and enqueue.

---

### Early Exit

```cpp
if(num == end)
    return steps + 1;
```

The moment BFS reaches `end`, we know this is the minimum steps (BFS processes level by level — first time = shortest). Return immediately.

---

### Trivial Case

```cpp
if(start == end) return 0;
```

If we're already at the destination, zero steps needed.

---

## 3. Dry Run

```
arr = [2, 3], start = 5, end = 20
mod = 1000

dist = all INT_MAX, dist[5] = 0
queue = [{steps=0, node=5}]
```

---

**Pop {0, node=5}:**
```
Multiply by 2: (5 × 2) % 1000 = 10
  steps+1=1 < dist[10]=INF → dist[10]=1, push {1,10}

Multiply by 3: (5 × 3) % 1000 = 15
  steps+1=1 < dist[15]=INF → dist[15]=1, push {1,15}

queue = [{1,10}, {1,15}]
```

**Pop {1, node=10}:**
```
Multiply by 2: (10×2)%1000 = 20
  2 < INF → dist[20]=2, 20==end → return 2 ✅
```

**Minimum steps: 2** (path: `5 → 10 → 20`)

---

**Now test with a `mod` wrapping case:**

```
arr=[3], start=500, end=500  → return 0 (start==end)

arr=[3], start=400, end=200, mod=1000

Step 1: 400×3=1200 % 1000 = 200 ← end! → return 1 ✅
```

**The mod wrapping makes otherwise unreachable numbers reachable!**

```
arr=[7], start=3, end=900
Step 1: 3×7=21
Step 2: 21×7=147
Step 3: 147×7=1029 → 1029%1000=29
Step 4: 29×7=203
Step 5: 203×7=1421 → 421
Step 6: 421×7=2947 → 947
Step 7: 947×7=6629 → 629
...BFS explores all reachable states from 3 via ×7
```

---

## 4. Story Points

---

**Story Point 1 — "mod 1000 bounds the state space to exactly 1000 nodes"**

Without mod, multiplication can produce arbitrarily large numbers → infinite state space → BFS is impossible. The mod 1000 constraint collapses all possible states to exactly `[0, 999]`. This transforms an infinite problem into a graph with 1000 nodes and at most `1000 × |arr|` directed edges — fully tractable for BFS.

---

**Story Point 2 — "Each element of `arr` creates one outgoing edge from every node"**

From any current number `node`, multiplying by `arr[i]` gives exactly one result `(node × arr[i]) % 1000`. This is a directed edge from `node` to that result. So each node has exactly `|arr|` outgoing edges. Total edges in the graph = `1000 × |arr|`.

---

**Story Point 3 — "BFS is correct because all edges have equal weight (1 step)"**

Every multiplication = 1 step, regardless of which `arr[i]` is used. All edges cost 1. BFS finds minimum edges = minimum steps. If different multiplications had different costs, we'd need Dijkstra.

---

**Story Point 4 — "The `dist` array prevents revisiting — same role as `vis[]`"**

```cpp
if(steps + 1 < dist[num]) {
    dist[num] = steps + 1;
    ...
}
```

If `num` was already reached with fewer steps, `steps + 1 < dist[num]` fails → skip. This prevents re-processing states that were already optimally reached. For a standard unweighted BFS, this is equivalent to checking `!vis[num]` because the first time we reach `num`, it's already with the minimum steps — `steps + 1 < dist[num]` will only be true once.

---

**Story Point 5 — "Cycles are possible — `dist` prevents infinite loops"**

```
arr=[10], start=1, mod=1000
1 → 10 → 100 → 1000%1000=0 → 0×10=0 → 0 → 0 (cycle!)
```

Without the `dist` check, BFS would loop forever on `0 → 0 → 0 → ...`. The `dist` check ensures `0` is only processed once — after that, `steps + 1 < dist[0]` always fails.

---

## 5. Code

```cpp
class Solution {
public:
    int minSteps(vector<int>& arr, int start, int end) {
        // Trivial case: already at destination
        if(start == end) return 0;

        int mod = 1000;   // state space bounded to [0, 999]

        queue<pair<int,int>> q;   // {steps, current_number}
        vector<int> dist(mod, INT_MAX);

        q.push({0, start});
        dist[start] = 0;

        while(!q.empty()) {
            int steps = q.front().first;
            int node  = q.front().second;
            q.pop();

            // Try multiplying by each element of arr
            for(auto& it : arr) {
                long long num = ((long long)it * node) % mod;

                // Only proceed if this is a shorter path to `num`
                if(steps + 1 < dist[num]) {
                    dist[num] = steps + 1;

                    // Early exit: reached destination
                    if(num == end)
                        return steps + 1;

                    q.push({steps + 1, (int)num});
                }
            }
        }

        return -1;   // end unreachable from start
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(mod × |arr|)`

| Step | Cost | Reason |
|---|---|---|
| BFS — nodes processed | `O(mod)` = `O(1000)` | At most 1000 distinct states |
| Per node: try all `arr` elements | `O(|arr|)` | One multiplication per element |
| Each transition | `O(1)` | Multiplication + mod + dist check |

**Total: `O(mod × |arr|)` = `O(1000 × |arr|)`**

Since `mod = 1000` is a constant → **`O(|arr|)`** in terms of input size, or **`O(1)`** if we treat both as constants.

---

### Space Complexity — `O(mod)`

| Structure | Size | Reason |
|---|---|---|
| `dist[]` | `O(mod)` = `O(1000)` | One entry per possible state |
| BFS queue | `O(mod)` worst case | At most 1000 states enqueued |

**Total: `O(mod)` = `O(1000)` = `O(1)`** (constant)

---

### Graph Perspective Summary

| Graph Concept | This Problem |
|---|---|
| **Nodes** | Numbers `0` to `999` |
| **Edges** | `(u, (u × arr[i]) % 1000)` for each `arr[i]` |
| **Edge weight** | 1 (one step per multiplication) |
| **Source** | `start` |
| **Destination** | `end` |
| **Algorithm** | BFS (unweighted shortest path) |
| **Total nodes** | 1000 |
| **Total edges** | `1000 × |arr|` |
