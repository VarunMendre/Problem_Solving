# Introduction to Graphs

---

## 1. What is a Graph?

A **graph** is a non-linear data structure that consists of:
- A set of **vertices** (also called nodes) — the entities
- A set of **edges** — connections between those entities

```
Formal definition:
G = (V, E)
where V = set of vertices, E = set of edges
```

Graphs model **relationships** between objects. Unlike trees (which are a special type of graph), graphs can have **cycles**, **multiple paths** between nodes, and **no single root**.

### Real-World Examples
- **Google Maps** — cities as nodes, roads as edges
- **Social Networks** — users as nodes, friendships as edges
- **Internet** — websites as nodes, hyperlinks as edges
- **Circuit boards** — components as nodes, wires as edges

---

## 2. How Graphs Are Stored in Memory

### 2.1 Adjacency Matrix

A 2D array of size `V × V` where `matrix[i][j] = 1` if there's an edge from `i` to `j`, else `0`.

```
Graph:  0 — 1 — 2
            |
            3

Adjacency Matrix (4x4):
    0  1  2  3
0 [ 0  1  0  0 ]
1 [ 1  0  1  1 ]
2 [ 0  1  0  0 ]
3 [ 0  1  0  0 ]
```

| | Pros | Cons |
|---|---|---|
| | `O(1)` edge lookup | `O(V²)` space — wasteful for sparse graphs |
| | Simple to implement | Adding a vertex requires resizing the matrix |

---

### 2.2 Adjacency List

An array of lists where `list[i]` contains all neighbors of vertex `i`.

```
Graph:  0 — 1 — 2
            |
            3

Adjacency List:
0 → [1]
1 → [0, 2, 3]
2 → [1]
3 → [1]
```

| | Pros | Cons |
|---|---|---|
| | `O(V + E)` space — efficient for sparse graphs | Edge lookup is `O(degree)` not `O(1)` |
| | Easy to iterate over neighbors | Slightly more complex to implement |

> **Most graph problems use adjacency lists** — real-world graphs tend to be sparse (few edges relative to possible edges).

---

### 2.3 Edge List

A list of all edges as pairs `(u, v)`.

```
[(0,1), (1,2), (1,3)]
```

Used in algorithms like **Kruskal's MST** where we need to sort edges by weight.

---

## 3. Types of Graphs

### 3.1 Undirected Graph

Edges have **no direction** — if there's an edge between A and B, you can travel both ways.

```
A — B — C
    |
    D
```

- `edge(A,B)` means both `A→B` and `B→A` exist
- Represented in adjacency list by adding both directions

**Example:** Friendships on Facebook (if A is friends with B, B is friends with A)

---

### 3.2 Directed Graph (Digraph)

Edges have a **direction** — an edge from A to B does NOT mean B to A.

```
A → B → C
    ↓
    D
```

- `edge(A,B)` means only `A→B` exists
- Represented with one-directional edges

**Example:** Twitter follows (A follows B doesn't mean B follows A), web page hyperlinks

---

### 3.3 Cyclic Graph

A graph that contains at least one **cycle** — a path that starts and ends at the same vertex.

```
Cyclic:           Acyclic (DAG):
A → B             A → B
↑   ↓             ↓   ↓
D ← C             C → D
```

- **Cyclic directed graph** → contains a directed cycle
- **Acyclic directed graph** → called a **DAG (Directed Acyclic Graph)**

**Example of DAG:** Task scheduling, course prerequisites (CS101 → CS201 → CS301)

---

### 3.4 Weighted Graph

Each edge carries a **weight** (cost, distance, time, etc.).

```
A —(4)— B —(2)— C
         |
        (7)
         |
         D
```

**Example:** Road distances between cities, flight costs between airports

---

### 3.5 Connected vs Disconnected Graph

- **Connected** (undirected): there's a path between every pair of vertices
- **Disconnected**: some vertices have no path to others

```
Connected:           Disconnected:
A — B — C            A — B    C — D
    |                              |
    D                              E
```

---

### 3.6 Complete Graph

Every vertex is directly connected to every other vertex. Has `V*(V-1)/2` edges (undirected).

```
Complete graph K4:
A — B
|\ /|
| X |
|/ \|
C — D
```

---

## 4. Paths in a Graph

A **path** is a sequence of vertices where each consecutive pair is connected by an edge.

```
Graph: A — B — C — D — E

Path from A to E: A → B → C → D → E
```

### Properties of Paths

| Property | Description | Example |
|---|---|---|
| **Length** | Number of edges in the path | A→B→C has length 2 |
| **Simple Path** | No vertex is repeated | A→B→C→D (no repeats) |
| **Walk** | Edges and vertices can repeat | A→B→A→B→C (B repeated) |
| **Trail** | Edges don't repeat, but vertices can | A→B→C→A→D (A repeated, no edge repeated) |
| **Shortest Path** | Path with minimum edges (unweighted) or minimum weight (weighted) | Dijkstra's algorithm finds this |
| **Cycle** | Path where start = end | A→B→C→A |

### Shortest Path vs. All Paths

```
Graph:
A — B
|   |
C — D

From A to D:
Paths: A→B→D (length 2), A→C→D (length 2), A→B→C→D (length 3), etc.
Shortest path length = 2
```

---

## 5. Degrees of a Graph

### 5.1 Degree of an Undirected Graph

The **degree** of a vertex = number of edges connected to it.

```
Graph:
    A
   /|\
  B C D
  |
  E

degree(A) = 3  (connected to B, C, D)
degree(B) = 2  (connected to A, E)
degree(C) = 1  (connected to A only)
degree(D) = 1  (connected to A only)
degree(E) = 1  (connected to B only)
```

**Handshaking Lemma:** Sum of all degrees = 2 × (number of edges)

```
Sum = 3+2+1+1+1 = 8 = 2 × 4 edges ✅
```

This means: **a graph always has an even sum of degrees.**

---

### 5.2 Degree of a Directed Graph — Indegree & Outdegree

In a directed graph, each vertex has two degrees:

- **Indegree** = number of edges coming **into** the vertex
- **Outdegree** = number of edges going **out** of the vertex

```
Directed graph:
A → B
A → C
B → C
C → D

Vertex | Indegree | Outdegree
  A    |    0     |    2
  B    |    1     |    1
  C    |    2     |    1
  D    |    1     |    0
```

**Property:** Sum of all indegrees = Sum of all outdegrees = number of edges

```
Sum indegrees  = 0+1+2+1 = 4
Sum outdegrees = 2+1+1+0 = 4
Number of edges = 4 ✅
```

**Source:** Indegree = 0 (no incoming edges) → vertex `A`  
**Sink:** Outdegree = 0 (no outgoing edges) → vertex `D`

---

## 6. Edge Weights

An **edge weight** is a numerical value assigned to an edge representing cost, distance, time, capacity, etc.

```
Weighted graph (city distances in km):
Mumbai —(1400)— Delhi
  |                 |
(950)             (500)
  |                 |
Pune    —(500)— Hyderabad
```

### When to Use Weighted Graphs?

| Problem | Weight meaning |
|---|---|
| Shortest route (GPS) | Distance or time |
| Cheapest flight | Ticket cost |
| Network bandwidth | Capacity |
| Game pathfinding | Movement cost |

### Unweighted = Weight of 1 Everywhere

An unweighted graph can be treated as a weighted graph where all edges have weight = 1. BFS finds shortest paths in unweighted graphs; Dijkstra's handles weighted ones.

---

## 7. Graph Conventions

### 7.1 Numbering Conventions

- Vertices are typically numbered **0 to N-1** (0-indexed) in competitive programming
- Some problems use **1 to N** (1-indexed) — always check the problem statement

### 7.2 Representing Edges

```cpp
// Undirected edge between u and v
adj[u].push_back(v);
adj[v].push_back(u);   // both directions

// Directed edge from u to v
adj[u].push_back(v);   // only one direction
```

### 7.3 Self-Loops

An edge from a vertex to **itself**.

```
A → A (self-loop)
```

Usually avoided in standard graph problems but appears in some real-world models.

### 7.4 Multi-Edges (Parallel Edges)

Multiple edges between the **same pair** of vertices.

```
A ==== B   (two edges between A and B)
```

A graph with no self-loops and no multi-edges is called a **Simple Graph**.

---

## 8. Applications of Graphs

| Domain | Application | Graph representation |
|---|---|---|
| **Navigation** | Shortest route (GPS, Google Maps) | Weighted directed graph |
| **Social Networks** | Friend suggestions, degree of separation | Undirected graph |
| **Web** | PageRank, web crawling | Directed graph |
| **Networking** | Routing packets, network topology | Weighted graph |
| **Scheduling** | Task dependencies, deadlines | DAG |
| **Biology** | Protein interaction networks | Undirected graph |
| **Compilers** | Dependency resolution, call graphs | DAG |
| **Games** | Pathfinding (A*, BFS) | Weighted grid graph |

---

## 9. Graph Density

The **density** of a graph measures how many edges exist relative to the maximum possible.

```
For undirected graph:
Density = 2E / V(V-1)

For directed graph:
Density = E / V(V-1)
```

- **Sparse graph:** `E << V²` — adjacency list preferred
- **Dense graph:** `E ≈ V²` — adjacency matrix may be preferred

---

## 10. Key Terms Summary

| Term | Definition |
|---|---|
| **Vertex / Node** | A point in the graph |
| **Edge** | A connection between two vertices |
| **Adjacent** | Two vertices connected by an edge |
| **Neighbor** | A vertex directly connected to another |
| **Degree** | Number of edges at a vertex |
| **Path** | Sequence of vertices connected by edges |
| **Cycle** | A path that starts and ends at the same vertex |
| **Connected** | Every vertex is reachable from every other |
| **DAG** | Directed Acyclic Graph — no cycles |
| **Sparse** | Few edges relative to vertices |
| **Dense** | Many edges relative to vertices |
| **Source** | Vertex with indegree 0 |
| **Sink** | Vertex with outdegree 0 |
