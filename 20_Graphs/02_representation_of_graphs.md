# Representation of Graphs

---

## 1. Adjacency Matrix

A 2D array of size `(N+1) × (N+1)` (using 1-based indexing) where:
- `adj[u][v] = 1` → edge exists between `u` and `v`
- `adj[u][v] = 0` → no edge

### Unweighted Graph

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;   // n = vertices, m = edges

    // 1-based indexing → size (n+1) x (n+1)
    int adj[n+1][n+1];

    // Initialize all to 0
    memset(adj, 0, sizeof(adj));

    for(int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;

        adj[u][v] = 1;   // edge from u to v
        adj[v][u] = 1;   // edge from v to u (undirected)
    }

    return 0;
}
```

### Weighted Graph

For weighted graphs, store the weight instead of `1`.  
`adj[u][v] = weight` → edge between `u` and `v` with that weight.

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;

    // Store weights instead of 0/1
    int adj[n+1][n+1];
    memset(adj, 0, sizeof(adj));

    for(int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;   // u, v = endpoints, w = weight

        adj[u][v] = w;   // weight of edge u→v
        adj[v][u] = w;   // weight of edge v→u (undirected)
    }

    return 0;
}
```

### Example

```
Graph:
1 —(4)— 2 —(7)— 3
         |
        (2)
         |
         4

Input:
n=4, m=3
1 2 4
2 3 7
2 4 2

Adjacency Matrix (weighted):
    1   2   3   4
1 [ 0   4   0   0 ]
2 [ 4   0   7   2 ]
3 [ 0   7   0   0 ]
4 [ 0   2   0   0 ]
```

### Pros & Cons

| Pros | Cons |
|---|---|
| `O(1)` edge lookup — `adj[u][v]` | `O(N²)` space — wasteful for sparse graphs |
| Simple to implement | Adding a vertex requires resizing the entire matrix |
| Easy to check if edge exists | Iterating over all neighbors is `O(N)` even if few exist |

> **Use when:** Dense graph (`E ≈ N²`), or when frequent edge existence checks are needed.

---

---

## 2. Adjacency List

An array of vectors where `adj[u]` stores all vertices directly connected to `u`.

### Unweighted Graph

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;   // n = vertices, m = edges

    // 1-based indexing → size n+1
    vector<vector<int>> adj(n+1);

    for(int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;

        adj[u].push_back(v);   // edge from u to v
        adj[v].push_back(u);   // edge from v to u (undirected)
    }

    return 0;
}
```

### Weighted Graph

For weighted graphs, store pairs `{neighbor, weight}` instead of just the neighbor.

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, m;
    cin >> n >> m;

    // Each entry is {neighbor, weight}
    vector<vector<pair<int,int>>> adj(n+1);

    for(int i = 0; i < m; i++) {
        int u, v, w;
        cin >> u >> v >> w;   // u, v = endpoints, w = weight

        adj[u].push_back({v, w});   // u connects to v with weight w
        adj[v].push_back({u, w});   // v connects to u with weight w (undirected)
    }

    return 0;
}
```

### Example

```
Graph:
1 —(4)— 2 —(7)— 3
         |
        (2)
         |
         4

Input:
n=4, m=3
1 2 4
2 3 7
2 4 2

Adjacency List (weighted):
1 → [(2,4)]
2 → [(1,4), (3,7), (4,2)]
3 → [(2,7)]
4 → [(2,2)]
```

### Iterating Over Neighbors (Weighted)

```cpp
// Print all neighbors and weights of vertex u
for(auto [neighbor, weight] : adj[u]) {
    cout << u << " → " << neighbor << " (weight: " << weight << ")\n";
}
```

### Pros & Cons

| Pros | Cons |
|---|---|
| `O(V + E)` space — efficient for sparse graphs | Edge lookup is `O(degree(u))` not `O(1)` |
| Iterating over neighbors is fast | Slightly more complex (pairs for weighted) |
| Scales well with large sparse graphs | |

> **Use when:** Sparse graph (`E << N²`), which covers most real-world problems.

---

---

## 3. Directed Graph Representation

For a **directed graph**, only add the edge in one direction:

```cpp
// Undirected:
adj[u].push_back(v);
adj[v].push_back(u);   // both directions

// Directed (u → v only):
adj[u].push_back(v);   // only one direction
```

**Directed + Weighted:**

```cpp
adj[u].push_back({v, w});   // edge from u to v with weight w
// do NOT add adj[v].push_back({u, w})
```

---

---

## 4. Comparison — Matrix vs List

| | Adjacency Matrix | Adjacency List |
|---|---|---|
| **Space** | `O(N²)` | `O(N + E)` |
| **Edge lookup** | `O(1)` | `O(degree)` |
| **Iterate neighbors** | `O(N)` | `O(degree)` |
| **Add vertex** | `O(N²)` (resize) | `O(1)` |
| **Add edge** | `O(1)` | `O(1)` |
| **Best for** | Dense graphs | Sparse graphs |
| **Used in** | Floyd-Warshall, simple problems | BFS, DFS, Dijkstra, most algorithms |

---

## 5. Common Graph Input Format (Competitive Programming)

```
5 6          ← n vertices, m edges
1 2          ← edge between 1 and 2
1 3
2 4
3 4
4 5
3 5
```

**Reading this:**
```cpp
int n, m;
cin >> n >> m;

vector<vector<int>> adj(n+1);

for(int i = 0; i < m; i++) {
    int u, v;
    cin >> u >> v;
    adj[u].push_back(v);
    adj[v].push_back(u);
}
```

**Weighted version input:**
```
4 3          ← n vertices, m edges
1 2 4        ← edge between 1-2 with weight 4
2 3 7
2 4 2
```

```cpp
vector<vector<pair<int,int>>> adj(n+1);

for(int i = 0; i < m; i++) {
    int u, v, w;
    cin >> u >> v >> w;
    adj[u].push_back({v, w});
    adj[v].push_back({u, w});
}
```
