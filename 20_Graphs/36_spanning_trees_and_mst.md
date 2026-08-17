# Spanning Trees and Minimum Spanning Trees
### Formal Notes for Revision

---

## 1. Prerequisite Definitions

| Term | Definition |
|---|---|
| **Graph** `G = (V, E)` | A set of vertices `V` connected by a set of edges `E`. |
| **Connected Graph** | A graph in which there exists a path between every pair of vertices. |
| **Tree** | A connected graph with no cycles. A tree with `n` vertices always has exactly `n − 1` edges. |
| **Subgraph** | A graph formed from a subset of the vertices and edges of a larger graph. |
| **Weighted Graph** | A graph in which each edge has an associated numeric cost (weight). |

---

## 2. Spanning Tree

### 2.1 Definition

A **Spanning Tree** of a connected, undirected graph `G = (V, E)` is a subgraph that:

1. Includes **all** the vertices of `G`.
2. Is **connected** (every vertex is reachable from every other vertex).
3. Contains **no cycles** (it is a tree).
4. Contains exactly **`|V| − 1` edges**, where `|V|` is the number of vertices.

In short: a spanning tree is the minimal set of edges required to keep a graph fully connected, with every redundant (cycle-forming) edge removed.

### 2.2 Key Properties

- A spanning tree exists **if and only if** the original graph is connected.
- Removing any single edge from a spanning tree disconnects it into two components.
- Adding any single edge back (from the edges not chosen) creates exactly one cycle.
- A graph can have **many different spanning trees**. For a complete graph with `n` vertices, the total number of distinct spanning trees is given by **Cayley's Formula**:

```
  Number of spanning trees = n^(n-2)
```

- For a general graph, the number of spanning trees can be computed using the **Matrix-Tree Theorem (Kirchhoff's Theorem)**, via the determinant of a reduced Laplacian matrix of the graph.

### 2.3 Visualization

**Original connected graph** (5 vertices, 6 edges — one more edge than a tree needs):

```
        A ------ B
        |  \      |
        |    \    |
        |      \  |
        D ------- C
         \       /
          \     /
             E
```

**One possible Spanning Tree** (5 vertices, 4 edges — cycle broken by removing edge A–C):

```
        A ------ B
        |         |
        |         |
        |         |
        D ------- C
         \
          \
             E
```

Every vertex (A, B, C, D, E) is still reachable, but no cycle remains — exactly `n − 1 = 4` edges.

---

## 3. Minimum Spanning Tree (MST)

### 3.1 Definition

Given a **connected, undirected, weighted graph**, a **Minimum Spanning Tree** is a spanning tree whose **sum of edge weights is the smallest possible** among all spanning trees of that graph.

- If all edge weights are distinct, the MST is **unique**.
- If some weights are equal, multiple MSTs may exist, but they will all share the same minimum total weight.

### 3.2 Why It Matters (Applications)

- Designing least-cost network layouts (telecom cables, road networks, electrical grids).
- Approximation algorithms for harder problems (e.g., the Traveling Salesman Problem).
- Cluster analysis in data mining (single-linkage clustering).
- Circuit design and wiring layout minimization.
- Image segmentation in computer vision.

### 3.3 Core Theoretical Properties

| Property | Explanation |
|---|---|
| **Cut Property** | For any cut (partition of vertices into two disjoint sets), the minimum-weight edge crossing the cut must belong to *some* MST. |
| **Cycle Property** | For any cycle in the graph, the maximum-weight edge in that cycle does **not** belong to any MST (assuming it's not needed elsewhere). |
| **Uniqueness** | If all edge weights are distinct, the MST is unique. |

### 3.4 Worked Example

Consider the following weighted graph:

```
          A
        / | \
      2/  |4 \6
      /   |   \
     B----C----D
      \3  |5  /1
       \  |  /
        \ | /
          E
```

Edge list with weights:

```
A–B : 2
A–C : 4
A–D : 6
B–C : 3
B–E : 3
C–D : 5
C–E : 5
D–E : 1
```

**Minimum Spanning Tree** (total weight = 2 + 3 + 1 + 4 = 10):

```
          A
        / 
      2/   
      /    
     B----C
           
       \3    
        \  
          E
          |
          |1
          D
```

Selected edges: `A–B (2)`, `B–C (3)`, `D–E (1)`, and `A–C (4)` or an equivalent minimal combination connecting all five vertices at the lowest total cost. The exact edges chosen may vary slightly depending on the algorithm and tie-breaking rules, but the **total minimum weight remains the same**.

---

## 4. Algorithms Used to Solve the MST Problem

The following are the classical algorithms used to construct a Minimum Spanning Tree, listed in the order they are typically introduced and studied:

1. **Kruskal's Algorithm**
   Sorts all edges by weight in ascending order and greedily adds each edge to the growing forest, skipping any edge that would form a cycle, until a spanning tree is formed. Relies on a **Union-Find (Disjoint Set Union)** structure to detect cycles efficiently.

2. **Prim's Algorithm**
   Grows a single tree from an arbitrary starting vertex, at each step adding the cheapest edge that connects a vertex already in the tree to a vertex outside it. Typically implemented with a **priority queue / min-heap**.

3. **Borůvka's Algorithm**
   The oldest of the three (1926). Proceeds in rounds: each component simultaneously selects its cheapest outgoing edge, and all selected edges are added at once, merging components until only one remains.

4. **Reverse-Delete Algorithm**
   The conceptual opposite of Kruskal's approach: starts with the full edge set sorted in **descending** order of weight, and removes edges one at a time as long as doing so does not disconnect the graph.

> **Note:** Kruskal's, Prim's, and Borůvka's algorithms are the three most commonly taught and used in practice; the Reverse-Delete algorithm is primarily of theoretical/pedagogical interest due to its higher computational cost.

---

## 5. Quick Comparison Summary

| Algorithm | Strategy | Typical Data Structure |
|---|---|---|
| Kruskal's | Edge-based, global greedy selection | Union-Find (DSU) |
| Prim's | Vertex-based, local greedy growth | Min-Heap / Priority Queue |
| Borůvka's | Component-based, parallel rounds | Component labeling |
| Reverse-Delete | Edge removal, connectivity checks | Graph connectivity check (e.g., BFS/DFS) |

---

## 6. Summary Takeaways

- A **Spanning Tree** connects all vertices of a graph using the minimum number of edges (`n − 1`) and no cycles.
- A **Minimum Spanning Tree** is the spanning tree with the least possible total edge weight in a weighted graph.
- MSTs are built using **greedy algorithms** — Kruskal's, Prim's, and Borůvka's — all of which are provably optimal due to the **cut property** of MSTs.
- The choice of algorithm in practice depends on graph density: **Kruskal's** tends to perform better on **sparse** graphs, while **Prim's** (with a heap) tends to perform better on **dense** graphs.
