# 04. BFS Traversal

## Problem Context
The goal of this function is to return the Breadth First Search traversal of a graph starting from node `0`. In BFS, the graph is explored level by level, which means all nodes at distance `1` from the start node are visited first, then all nodes at distance `2`, then distance `3`, and so on.

This specific code assumes:
- The graph is represented using an adjacency list.
- Nodes are numbered from `0` to `n - 1`.
- Traversal begins from node `0`.
- Only the connected component containing node `0` is traversed.

---

## 1. Approach / Intuition

### What is the real idea behind BFS?
The deepest intuition behind BFS is this: if the problem says **traverse a graph from a starting node in the order of closeness**, then a simple linear scan is not enough because a graph can branch in many directions. From one node, there may be multiple neighbors, and from each neighbor, more neighbors. So the first question is: how can the traversal stay organized and avoid randomly going deep into one branch?

The answer is to process nodes in the exact order in which they are discovered. The first discovered node should be processed first, then the next discovered node, and so on. This is exactly the behavior of a **queue**, which follows FIFO (First In First Out). That is why BFS naturally leads to a queue.

### Why does “level by level” matter?
Suppose node `0` is the source. Its direct neighbors are the closest nodes in the graph from the source. So before visiting nodes that are farther away, it makes sense to finish all immediate neighbors first. This creates the level-wise pattern:
- Level 0: source node itself.
- Level 1: all neighbors of the source.
- Level 2: neighbors of level 1 nodes that were not already visited.
- Level 3: neighbors of level 2 nodes, and so on.

This is the core intuition: **BFS expands like a wave**. It first touches everything near the source, then moves outward one layer at a time.

### How does one arrive at the queue intuition?
Think through the process manually:
1. Start at node `0`.
2. Visit its neighbors.
3. But once neighbors are found, which one should be handled first?
4. The most natural answer is: the one that was found first.
5. After handling that node, again its neighbors are appended to the pending work.

This “pending work” must preserve arrival order. A stack would break this idea because it would immediately go deep into the latest discovered branch, which becomes DFS behavior. A queue preserves breadth, so it becomes the perfect data structure.

### Why is a visited array necessary?
Graphs are not like trees. In a tree, there is exactly one path to each child from the root. In a graph, the same node can be reached from multiple directions, and cycles may exist.

For example, if `0 -> 1`, `1 -> 2`, and `2 -> 0`, then without a visited array, traversal can keep circling forever. So the intuition is not just “visit nodes”; it is “visit each node exactly once.”

That leads to the `visited` array:
- `visited[i] = 0` means node `i` has not been discovered yet.
- `visited[i] = 1` means node `i` is already discovered, so it should not be inserted into the queue again.

A very important intuition point is this: the node is marked visited **when it is pushed into the queue**, not when it is popped. This prevents duplicate queue insertions. If marking were delayed until popping, the same node could be pushed many times by different neighbors.

### Why start by marking node 0 visited and pushing it?
The algorithm starts from source node `0`, so two things must happen immediately:
- Mark `0` as discovered.
- Put `0` into the queue so it can be processed.

This means the queue always stores nodes that are discovered but not yet fully processed.

### Why do we push the popped node into the answer array?
When a node comes out of the queue, that means it is the next node in the BFS order. Therefore, at that moment, it should be added to the `bfs` result vector.

So the overall mental model becomes:
- Discover a node.
- Put it in the queue.
- Later, when its turn comes, remove it from the queue.
- Record it in the answer.
- Discover all of its unvisited neighbors.

### Full intuition of the algorithm
The algorithm keeps two areas of control:
- `queue<int> q` stores nodes waiting to be processed in breadth-first order.
- `vector<int> visited` ensures no node is processed more than once.

Then the traversal repeatedly does this:
1. Take the front node from the queue.
2. Add it to the BFS traversal.
3. Explore every adjacent node.
4. For every unvisited adjacent node, mark it visited and push it into the queue.

This continues until no discovered-but-unprocessed nodes remain. At that point, the queue becomes empty, and the BFS traversal is complete.

---

## 2. Dry Run

To understand the code deeply, consider this graph:

- `0 -> 1, 2`
- `1 -> 0, 3, 4`
- `2 -> 0, 5`
- `3 -> 1`
- `4 -> 1, 5`
- `5 -> 2, 4`

Adjacency list representation:

```cpp
adj[0] = {1, 2}
adj[1] = {0, 3, 4}
adj[2] = {0, 5}
adj[3] = {1}
adj[4] = {1, 5}
adj[5] = {2, 4}
```

Number of nodes: `n = 6`

### Initial state
```cpp
visited = [0, 0, 0, 0, 0, 0]
```

Then:
- `visited[0] = 1`
- `q.push(0)`

Now:
```cpp
visited = [1, 0, 0, 0, 0, 0]
q = [0]
bfs = []
```

### Iteration 1
Queue front is `0`.
- Pop `0`
- Add `0` to BFS

Now:
```cpp
q = []
bfs = [0]
```

Explore neighbors of `0`: `{1, 2}`

- Neighbor `1` is unvisited, so mark visited and push into queue.
- Neighbor `2` is unvisited, so mark visited and push into queue.

Now:
```cpp
visited = [1, 1, 1, 0, 0, 0]
q = [1, 2]
bfs = [0]
```

### Iteration 2
Queue front is `1`.
- Pop `1`
- Add `1` to BFS

Now:
```cpp
q = [2]
bfs = [0, 1]
```

Explore neighbors of `1`: `{0, 3, 4}`

- Neighbor `0` is already visited, ignore it.
- Neighbor `3` is unvisited, mark visited and push.
- Neighbor `4` is unvisited, mark visited and push.

Now:
```cpp
visited = [1, 1, 1, 1, 1, 0]
q = [2, 3, 4]
bfs = [0, 1]
```

### Iteration 3
Queue front is `2`.
- Pop `2`
- Add `2` to BFS

Now:
```cpp
q = [3, 4]
bfs = [0, 1, 2]
```

Explore neighbors of `2`: `{0, 5}`

- Neighbor `0` is already visited, ignore it.
- Neighbor `5` is unvisited, mark visited and push.

Now:
```cpp
visited = [1, 1, 1, 1, 1, 1]
q = [3, 4, 5]
bfs = [0, 1, 2]
```

### Iteration 4
Queue front is `3`.
- Pop `3`
- Add `3` to BFS

Now:
```cpp
q = [4, 5]
bfs = [0, 1, 2, 3]
```

Explore neighbors of `3`: `{1}`
- Neighbor `1` is already visited, so do nothing.

State remains:
```cpp
visited = [1, 1, 1, 1, 1, 1]
q = [4, 5]
bfs = [0, 1, 2, 3]
```

### Iteration 5
Queue front is `4`.
- Pop `4`
- Add `4` to BFS

Now:
```cpp
q = [5]
bfs = [0, 1, 2, 3, 4]
```

Explore neighbors of `4`: `{1, 5}`
- Neighbor `1` is already visited.
- Neighbor `5` is already visited, because it was discovered earlier from node `2`.

State remains:
```cpp
visited = [1, 1, 1, 1, 1, 1]
q = [5]
bfs = [0, 1, 2, 3, 4]
```

### Iteration 6
Queue front is `5`.
- Pop `5`
- Add `5` to BFS

Now:
```cpp
q = []
bfs = [0, 1, 2, 3, 4, 5]
```

Explore neighbors of `5`: `{2, 4}`
- Both are already visited.

Queue is empty, so traversal stops.

### Final BFS traversal
```cpp
[0, 1, 2, 3, 4, 5]
```

### What important behavior does this dry run prove?
This dry run proves multiple key intuition points:
- Nodes are processed in the order they are discovered.
- The queue guarantees breadth-wise movement.
- The visited array prevents cycles and duplicate processing.
- A node can be discovered from one parent earlier, and if another parent reaches it later, it is ignored.
- The BFS answer reflects layers from the source.

Distance-wise from node `0`:
- Distance `0`: `0`
- Distance `1`: `1, 2`
- Distance `2`: `3, 4, 5`

That is exactly the BFS nature.

---

## 3. Code

```cpp
#include<bits/stdc++.h>
using namespace std;

vector<int> bfsTraversal(int n, vector<vector<int>> &adj) {
    vector<int> visited(n, 0);
    visited[0] = 1;

    queue<int> q;
    q.push(0);

    vector<int> bfs;

    while (!q.empty()) {
        int node = q.front();
        q.pop();

        bfs.push_back(node);

        for (auto it : adj[node]) {
            if (!visited[it]) {
                visited[it] = 1;
                q.push(it);
            }
        }
    }

    return bfs;
}
```

### Line-by-line explanation

#### `vector<int> visited(n, 0);`
A visited array of size `n` is created. Initially, every node is marked unvisited. This means the algorithm has not discovered any node yet.

#### `visited[0] = 1;`
The source node is node `0`, so it is immediately marked visited. This is done before pushing it into the queue so that if any edge points back to `0`, it will not be inserted again.

#### `queue<int> q;`
A queue is created to maintain BFS order. This queue stores the nodes that have been discovered but are still waiting to be processed.

#### `q.push(0);`
The source node is inserted into the queue. BFS cannot start unless the initial node is placed into the pending structure.

#### `vector<int> bfs;`
This vector stores the traversal order. Whenever a node is popped from the queue, it is appended here.

#### `while (!q.empty())`
As long as there exists at least one discovered but unprocessed node, BFS should continue. The moment the queue becomes empty, no more reachable nodes remain.

#### `int node = q.front();`
Take the node at the front of the queue. This is the earliest discovered node among all pending nodes, which preserves breadth-first order.

#### `q.pop();`
Remove that node from the queue because it is now being processed.

#### `bfs.push_back(node);`
Record this node in the result traversal. This is the exact moment when the node becomes part of the BFS order.

#### `for (auto it : adj[node])`
Traverse all neighbors of the current node. Since the graph is stored as an adjacency list, `adj[node]` gives all nodes directly connected to `node`.

#### `if (!visited[it])`
Only unvisited neighbors should be discovered. If the neighbor is already visited, that means it has already been queued earlier or processed already.

#### `visited[it] = 1;`
Mark the neighbor visited immediately at discovery time. This is a subtle but very important BFS design choice, because it prevents the same node from entering the queue multiple times.

#### `q.push(it);`
Push the newly discovered neighbor into the queue so that its own neighbors can be explored later in BFS order.

#### `return bfs;`
Once the queue becomes empty, the traversal is complete, and the BFS order is returned.

### Why this exact pattern is correct
This code is correct because it maintains three guarantees throughout execution:
1. Every node enters the queue at most once.
2. Every reachable node from `0` is eventually processed.
3. The queue preserves the discovery order, which creates breadth-first traversal.

Those three guarantees are the reason this implementation works reliably on general graphs.

---

## 4. Complexity Analysis

### Time Complexity
The time complexity is:

```cpp
TC = O(N) + O(2E)
```
for an undirected graph represented with an adjacency list.

This expression needs proper intuition, not just memorization.

### Why `O(N)` appears
Each node is popped from the queue at most once, because once a node is marked visited, it is never pushed again. Also, each node is added to the BFS array exactly once.

So all node-based work such as:
- popping from queue,
- pushing into BFS result,
- checking loop conditions for that node,

collectively contributes `O(N)`.

### Why `O(2E)` appears in an undirected graph
The inner loop is:

```cpp
for (auto it : adj[node])
```

This loop runs over adjacency lists. The total cost of all such loops together depends on the total size of all adjacency lists.

In an undirected graph:
- every edge `(u, v)` is stored in `adj[u]`,
- and the same edge is also stored in `adj[v]`.

So one undirected edge contributes two adjacency entries. Therefore, if the graph has `E` edges, the total adjacency entries become `2E`.

That is why traversing all adjacency lists across the entire BFS costs:

```cpp
O(2E)
```

Hence total time complexity becomes:

```cpp
O(N) + O(2E)
```

In asymptotic notation, constants are ignored, so this is equivalent to:

```cpp
O(N + E)
```

But while explaining in interviews or notes, writing `O(N) + O(2E)` is often useful because it shows the intuition very clearly: node work plus edge-list traversal work.

### If the graph is directed
For a directed graph, each edge is stored only once in the adjacency list, so the traversal cost becomes:

```cpp
O(N) + O(E)
```

### Why the time complexity is not `O(N * E)`
Sometimes beginners think there is a nested loop, so complexity should multiply. That is not correct here.

Although there is a `for` loop inside a `while` loop, the inner loop does **not** run `E` times for each node independently. Across the full execution, every adjacency entry is scanned only once. So the total scanning work is cumulative, not multiplicative.

That is the key reason BFS remains linear in the size of the graph representation.

---

### Space Complexity
Space complexity is often written in detailed form as:

```cpp
SC = O(visited array) + O(queue) + O(answer)
```

Now analyze each part carefully.

### 1. Visited array
```cpp
vector<int> visited(n, 0);
```
This stores one value for each node, so it takes:

```cpp
O(N)
```

### 2. Queue
```cpp
queue<int> q;
```
In the worst case, the queue may hold many nodes at once. For example, in a wide graph, an entire level may be waiting inside the queue simultaneously. In the worst case, that can be up to `N` nodes.

So queue space is:

```cpp
O(N)
```

### 3. BFS answer vector
```cpp
vector<int> bfs;
```
This stores the traversal result. If all reachable nodes are traversed and the graph is connected from node `0`, then this vector can store up to `N` nodes.

So answer space is:

```cpp
O(N)
```

### Total auxiliary understanding
If explaining in expanded form:

```cpp
SC = O(N) + O(N)
```
when only counting the working memory used during traversal:
- `O(N)` for visited array
- `O(N)` for queue

This is often described informally as:

```cpp
SC = O(2N)
```

Since constants are ignored, asymptotically this becomes:

```cpp
O(N)
```

### If the returned traversal array is also counted
Sometimes, while writing detailed notes, the result vector is mentioned separately:

```cpp
O(N)  -> visited array
O(N)  -> queue
O(N)  -> bfs result
```

So total becomes:

```cpp
O(3N)
```

Again, asymptotically:

```cpp
O(N)
```

### Important interview-style explanation
A very clear and accepted way to explain it is:

- **Auxiliary space / extra space used to solve the problem:** `O(2N)` which simplifies to `O(N)`.
- **If including the output array also:** `O(3N)` which simplifies to `O(N)`.

This matches the style of explanation you asked for.

---

## Final Understanding
BFS is not just “use queue and visited.” The real intuition is that graph traversal must preserve discovery order when the requirement is to move outward level by level from the source. The queue provides that order, and the visited array makes the traversal safe and non-redundant in the presence of cycles and multiple paths.

That is why this algorithm is the natural solution whenever the problem asks for:
- level order traversal in a graph,
- shortest path in an unweighted graph,
- minimum number of edges from a source,
- or plain BFS traversal from a starting node.
