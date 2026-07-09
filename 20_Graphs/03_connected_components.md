# Connected Components in Graph

A **connected component** is a group of vertices in an undirected graph where every vertex can reach every other vertex in that same group.[cite:32][cite:23]
If a graph has separate groups with no path between them, each group is a different connected component.[cite:32][cite:31]

## Quick idea

- One fully connected graph -> one component.[cite:32]
- Separate groups -> multiple components.[cite:23][cite:31]
- An isolated vertex also forms a component by itself.[cite:32]

## Example 1

Graph:

```text
0 -- 1 -- 2 -- 3
```

Another diagram of the same idea:

```text
0 ----- 1 ----- 2 ----- 3
```

Connected components:

```text
{0, 1, 2, 3}
```

This graph has only one connected component because all vertices are reachable from one another.[cite:32][cite:23]

## Example 2

Graph:

```text
0 -- 1 -- 2      3 -- 4
```

Another diagram:

```text
(0)---(1)---(2)    (3)---(4)
```

Connected components:

```text
{0, 1, 2}
{3, 4}
```

This graph has two connected components because there is no path between the two groups.[cite:32][cite:31]

## Example 3

Graph:

```text
0 -- 1      2      3 -- 4
```

Another diagram:

```text
A -- B      C      D -- E
```

Connected components:

```text
{0, 1}
{2}
{3, 4}
```

Vertex `2` is isolated, so it forms a component by itself.[cite:32]

## Example 4

Graph:

```text
    1
   / \
  0---2
```

Another diagram:

```text
0 -- 1
 \  /
  2
```

Connected components:

```text
{0, 1, 2}
```

Even though the shape is different, all three vertices are still in one connected component.[cite:31][cite:32]

## Example 5

Graph:

```text
0 -- 1 -- 2      3 -- 4      5
```

Another diagram:

```text
[A]-[B]-[C]    [D]-[E]    [F]
```

Connected components:

```text
{0, 1, 2}
{3, 4}
{5}
```

This graph has three components: one chain of three vertices, one pair, and one isolated vertex.[cite:32][cite:31]

## Example 6

Graph:

```text
0 -- 1      2 -- 3 -- 4      6
     |             |
     5             7
```

Another diagram:

```text
(0)--(1)    (2)--(3)--(4)    (6)
       |             |
      (5)           (7)
```

Connected components:

```text
{0, 1, 5}
{2, 3, 4, 7}
{6}
```

This graph has three different groups, and none of them are connected to each other.[cite:32][cite:23]

## Pseudocode

```text
FIND-CONNECTED-COMPONENTS(graph)
    mark all vertices as unvisited

    for each vertex v in graph
        if v is unvisited
            start a new component
            collect all vertices reachable from v
            mark them visited
            store that group
```

## Counting idea

```text
COUNT-COMPONENTS(graph)
    count = 0
    mark all vertices as unvisited

    for each vertex v in graph
        if v is unvisited
            collect all reachable vertices from v
            mark them visited
            count = count + 1
```

## Note

This topic is usually found using DFS or BFS, but at this stage the main idea is only to understand **grouping reachable vertices together**.[cite:23][cite:31]
