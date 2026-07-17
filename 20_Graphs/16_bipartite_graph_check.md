# Check if a Graph is Bipartite

---

## 1. Problem Statement

A graph is **bipartite** if its nodes can be divided into **two groups** such that every edge connects a node from group A to a node from group B — no edge connects two nodes within the same group.

In other words: can we **2-color** the graph such that no two adjacent nodes share the same color?

```
Bipartite:              Not Bipartite:
0 — 1                   0 — 1
|   |                   |   |
3 — 2                   2 — 0  ← odd cycle (triangle)
                        (can't 2-color a triangle)

Color: 0→red, 1→blue    Coloring attempt fails
       2→red, 3→blue
```

Return `true` if the graph is bipartite, `false` otherwise.

---

## 2. Intuition / Approach

### The 2-Coloring Insight

Bipartite checking is equivalent to **2-coloring** — can we assign one of two colors (0 or 1) to every node such that no two adjacent nodes have the same color?

**Algorithm:**
- Start from any unvisited node, color it `0`
- Color all its neighbors `1`
- Color all their neighbors `0`
- And so on (alternating colors)
- If at any point we try to color a node but it's already colored with the same color as its neighbor → **conflict → not bipartite**

---

### Why DFS Works Naturally Here

DFS naturally traces one path as deep as possible. As it goes deeper, it alternates colors (`0 → 1 → 0 → 1...`). When DFS backtracks and tries another branch, colors are already assigned — we just check for conflicts.

The `!col` trick elegantly handles alternation:
```cpp
dfs(ng, !col, ...)   // flip color for every neighbor
```
`!0 = 1`, `!1 = 0` — perfectly alternates between the two groups.

---

### The Three Cases When Visiting a Neighbor

| Case | Condition | Meaning | Action |
|---|---|---|---|
| **Unvisited** | `color[ng] == -1` | First time seeing this node | Color it `!col`, recurse deeper |
| **Different color** | `color[ng] != col` | Already colored correctly | Skip — no conflict |
| **Same color** | `color[ng] == col` | Conflict! Same color on both ends of edge | Return `false` |

---

### Why the Outer Loop?

Handles **disconnected graphs**. One DFS call from node `0` only explores the component containing `0`. If there are multiple components, we check each independently.

```cpp
for(int i = 0; i < n; i++) {
    if(color[i] == -1) {           // unvisited → new component
        if(dfs(i, 0, ...) == false)
            return false;
    }
}
```

Each component is independently bipartite-checked. If ANY component fails → whole graph fails.

---

### When is a Graph NOT Bipartite?

A graph is NOT bipartite if and only if it contains an **odd-length cycle** (cycle with 1, 3, 5... edges).

```
Even cycle (4-cycle):       Odd cycle (3-cycle / triangle):
0 — 1                       0 — 1
|   |                           |
3 — 2                       2 — 0
Colors: 0,1,0,1 ✅           Try: 0→1→0→but 2 connects to 0(same color!) ❌
```

2-coloring naturally fails exactly on odd cycles — the alternation "comes back" to the start with the wrong color.

---

## 3. Dry Run

### Test Case 1 — Bipartite

```
graph = [[1,3], [0,2], [1,3], [0,2]]

Structure:
0 — 1
|   |
3 — 2

color = [-1,-1,-1,-1]
```

**Outer loop i=0, color[0]==-1:**

```
dfs(0, col=0):
  color[0] = 0
  neighbor 1: color=-1 → dfs(1, col=1):
    color[1] = 1
    neighbor 0: color=0 ≠ 1 → no conflict → skip
    neighbor 2: color=-1 → dfs(2, col=0):
      color[2] = 0
      neighbor 1: color=1 ≠ 0 → skip
      neighbor 3: color=-1 → dfs(3, col=1):
        color[3] = 1
        neighbor 0: color=0 ≠ 1 → skip
        neighbor 2: color=0 ≠ 1 → skip
        return true ← dfs(3)
      return true ← dfs(2)
    return true ← dfs(1)
  neighbor 3: color=1 ≠ 0 → skip
  return true ← dfs(0)

color = [0, 1, 0, 1]
```

All nodes visited in one DFS. Return `true` ✅

```
Group 0 (color=0): {0, 2}
Group 1 (color=1): {1, 3}
Every edge (0-1, 1-2, 2-3, 3-0) connects different groups ✅
```

---

### Test Case 2 — Not Bipartite (Odd Cycle)

```
graph = [[1,2,3], [0,2], [0,1,3], [0,2]]

Contains triangle: 0—1—2—0

color = [-1,-1,-1,-1]
```

**dfs(0, col=0):**
```
color[0] = 0
neighbor 1: color=-1 → dfs(1, col=1):
  color[1] = 1
  neighbor 0: color=0 ≠ 1 → skip
  neighbor 2: color=-1 → dfs(2, col=0):
    color[2] = 0
    neighbor 0: color=0 == 0 → CONFLICT → return false ❌
  dfs(2) returned false → return false ← dfs(1)
dfs(1) returned false → return false ← dfs(0)
```

Return `false` ✅ (triangle = odd cycle = not bipartite)

---

## 4. Story Points

---

**Story Point 1 — "Bipartite = 2-colorable"**

These are mathematically equivalent. If you can 2-color the graph, it's bipartite. If it's bipartite, you can 2-color it. So checking bipartiteness reduces to: does a consistent 2-coloring exist? DFS does this naturally by alternating colors.

---

**Story Point 2 — "`!col` is the elegance"**

```cpp
dfs(ng, !col, color, graph)
```

This single expression handles the entire alternation logic. No if-else, no `col == 0 ? 1 : 0`. Just flip. It works because we only have two colors (0 and 1), and NOT on a bit flips between them.

---

**Story Point 3 — "color[] serves as both visited array and conflict detector"**

`color[node] == -1` means unvisited. Once colored (`0` or `1`), it's "visited". The color itself IS the visited state. When we revisit a colored node, we check if its color conflicts with the current path's expected color. One array does two jobs.

---

**Story Point 4 — "Odd cycle = guaranteed failure"**

You don't need to explicitly detect cycles. The 2-coloring attempt naturally fails on odd cycles. An even cycle (4-cycle, 6-cycle) will always succeed because the alternation completes consistently. Only odd cycles create a contradiction. Bipartite check = odd cycle detection.

---

**Story Point 5 — "Each disconnected component is checked independently"**

A graph with multiple components is bipartite if and only if EVERY component is bipartite. The outer loop ensures no component is skipped. If any one fails → entire graph fails.

---

## 5. Code

```cpp
class Solution {
private:
    bool dfs(int node, int col, vector<int>& color,
             vector<vector<int>>& graph) {

        color[node] = col;   // color current node

        for(auto& ng : graph[node]) {
            if(color[ng] == -1) {
                // Unvisited neighbor → color it opposite, recurse
                if(dfs(ng, !col, color, graph) == false)
                    return false;
            }
            else if(color[ng] == col) {
                // Already colored same as current → conflict → not bipartite
                return false;
            }
            // color[ng] != col → already correctly colored → no action
        }

        return true;
    }

public:
    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> color(n, -1);   // -1 = unvisited

        // Check every component
        for(int i = 0; i < n; i++) {
            if(color[i] == -1) {
                if(dfs(i, 0, color, graph) == false)
                    return false;
            }
        }

        return true;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(V + 2E)`

| Step | Cost | Reason |
|---|---|---|
| Outer loop | `O(V)` | Checks every node |
| All DFS calls combined | `O(V + 2E)` | Each node visited once; each undirected edge traversed from both endpoints |

**Total: `O(V + 2E)` = `O(V + E)`**

---

### Space Complexity — `O(V)`

| Structure | Size | Reason |
|---|---|---|
| `color[]` array | `O(V)` | One entry per node |
| DFS recursion stack | `O(V)` worst case | Linear chain forces max depth V |

**Total: `O(2V)` = `O(V)`**

---

### DFS vs BFS for Bipartite Check

The BFS version uses a queue instead of recursion, storing `{node, color}` pairs — same logic, iterative instead of recursive.

| | DFS | BFS |
|---|---|---|
| Color propagation | Via recursion parameter `!col` | Via queue with `{node, col}` |
| Stack overflow risk | Yes, for deep graphs | None |
| TC | `O(V + E)` | `O(V + E)` |
| SC | `O(V)` call stack | `O(V)` queue |
| Result | Same | Same |
