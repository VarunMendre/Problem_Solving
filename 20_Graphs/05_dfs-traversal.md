# 05. DFS Traversal

## Problem Context
The goal of this function is to return the Depth First Search traversal of a graph starting from node `0`. In DFS, the traversal goes as deep as possible along one path before backtracking and exploring the next available path.

This code assumes:
- The graph is represented using an adjacency list.
- Nodes are numbered from `0` to `V - 1`.
- Traversal starts from node `0`.
- Only the connected component containing node `0` is traversed.

---

## 1. Approach / Intuition

### What is the core idea behind DFS?
The most important intuition behind DFS is this: if the task is to explore a graph by following one path completely before trying another, then the traversal needs a mechanism to remember where it came from and return when it cannot go further. That exact behavior is called **backtracking**.

So DFS is not just “visit neighbors.” It is a strategy of exploration where each node says: “go to one unvisited neighbor, then from there go to one more unvisited neighbor, and keep doing this until no further move is possible.” Only then does the traversal come back to the previous node and continue from the next remaining option.

### Why does DFS naturally lead to recursion?
Think about how a human would explore a maze:
1. Start from the entrance.
2. Pick one path and keep walking.
3. If a dead end is reached, come back to the previous junction.
4. Then try another unexplored path.

That is exactly what recursion does automatically. Every recursive call represents “going deeper” into the graph. When that path finishes, the function call ends and control returns to the previous call, which is equivalent to backtracking.

So the intuition for recursion is not random. It comes from the nature of the problem itself: **deep exploration with automatic return to the previous state**.

### Why not use a queue here?
A queue preserves first-in-first-out order, which is ideal for breadth-wise expansion, as in BFS. But DFS does not want breadth. It wants to continue with the most recently discovered unfinished path.

That means DFS naturally behaves like a stack:
- the latest node entered should be the first one to continue from,
- and when a path finishes, traversal should return to the previous active node.

Recursion uses the call stack internally, so this recursive DFS is actually stack-based traversal without explicitly writing a `stack<int>`.

### Why is a visited array still necessary?
Just like BFS, DFS also works on general graphs, not only trees. A graph may contain:
- cycles,
- multiple paths to the same node,
- self-loops,
- repeated incoming connections.

Without a visited array, DFS could keep revisiting the same nodes forever. For example, if `0 -> 1`, `1 -> 2`, and `2 -> 0`, then recursion would never stop because the traversal would cycle endlessly.

So the real rule is not “go deep.” The real rule is: **go deep only through unvisited nodes**. That is why before making a recursive call, the code checks whether the neighbor has already been visited.

### Why do we mark a node visited before exploring neighbors?
This is a very important intuition point. The code does:

```cpp
vis[node] = 1;
```

before exploring neighbors.

Why? Because the moment a node is entered, it should be considered discovered. If that node were marked visited later, then another recursive branch could come back to it before marking happens, creating duplicate recursion and possible infinite loops.

So the safe and correct thought process is:
- arrive at node,
- immediately mark it visited,
- record it in the answer,
- then explore outward.

### Why is the node pushed into the answer list immediately?
In this DFS traversal, the traversal order is **preorder style** for graphs. That means the node is recorded when it is first entered, not after all of its neighbors are processed.

This is why:

```cpp
ls.push_back(node);
```

comes before the neighbor loop.

That reflects the meaning of DFS traversal here: as soon as the traversal reaches a node for the first time, it becomes part of the DFS order.

### How does the intuition build from small steps?
The thought process to derive the algorithm can be built like this:
1. Start from a source node.
2. Visit it.
3. From that node, choose an unvisited neighbor.
4. Repeat the same process on that neighbor.
5. If at some node no unvisited neighbor exists, return to the previous node.
6. Continue until all reachable nodes from the source are handled.

Notice something beautiful here: step 4 says “repeat the same process.” Whenever a problem says “do the same task again on a smaller part,” recursion becomes a very natural fit.

### Full intuition of this DFS code
This implementation separates the logic into two parts:
- a helper function `dfs(node, adj, vis, ls)` that performs the actual recursive traversal,
- and a wrapper function `dfs(adj)` that prepares the visited array, the result list, and the starting node.

The helper function follows the exact DFS philosophy:
1. Mark current node as visited.
2. Add it to traversal result.
3. Check every adjacent node.
4. For each unvisited adjacent node, recursively do the same DFS process.

That means each recursive call takes responsibility for exploring one full branch of the graph before returning.

---

## 2. Dry Run

Consider this graph:

- `0 -> 1, 2`
- `1 -> 0, 3, 4`
- `2 -> 0, 5`
- `3 -> 1`
- `4 -> 1, 5`
- `5 -> 2, 4`

Adjacency list:

```cpp
adj[0] = {1, 2}
adj[1] = {0, 3, 4}
adj[2] = {0, 5}
adj[3] = {1}
adj[4] = {1, 5}
adj[5] = {2, 4}
```

Start node is `0`.

Initial state:

```cpp
visited = [0, 0, 0, 0, 0, 0]
result = []
```

Now the wrapper calls:

```cpp
dfs(0, adj, visited, result)
```

### Call 1: `dfs(0)`
At node `0`:
- mark `0` visited
- add `0` to result

Now:

```cpp
visited = [1, 0, 0, 0, 0, 0]
result = [0]
```

Neighbors of `0` are `{1, 2}`.
The first unvisited neighbor is `1`, so recursive call goes to `dfs(1)`.

### Call 2: `dfs(1)`
At node `1`:
- mark `1` visited
- add `1` to result

Now:

```cpp
visited = [1, 1, 0, 0, 0, 0]
result = [0, 1]
```

Neighbors of `1` are `{0, 3, 4}`.
- `0` is already visited, skip.
- `3` is unvisited, so call `dfs(3)`.

### Call 3: `dfs(3)`
At node `3`:
- mark `3` visited
- add `3` to result

Now:

```cpp
visited = [1, 1, 0, 1, 0, 0]
result = [0, 1, 3]
```

Neighbors of `3` are `{1}`.
- `1` is already visited, so nothing more to do.

This means node `3` has no unvisited neighbor left. So recursion ends here and control goes back to `dfs(1)`.

This return is the exact meaning of **backtracking**.

### Back to Call 2: `dfs(1)`
Traversal resumes from where it paused in node `1`’s neighbor loop.
The next neighbor after `3` is `4`.
- `4` is unvisited, so call `dfs(4)`.

### Call 4: `dfs(4)`
At node `4`:
- mark `4` visited
- add `4` to result

Now:

```cpp
visited = [1, 1, 0, 1, 1, 0]
result = [0, 1, 3, 4]
```

Neighbors of `4` are `{1, 5}`.
- `1` is already visited, skip.
- `5` is unvisited, so call `dfs(5)`.

### Call 5: `dfs(5)`
At node `5`:
- mark `5` visited
- add `5` to result

Now:

```cpp
visited = [1, 1, 0, 1, 1, 1]
result = [0, 1, 3, 4, 5]
```

Neighbors of `5` are `{2, 4}`.
- `2` is unvisited, so call `dfs(2)`.

### Call 6: `dfs(2)`
At node `2`:
- mark `2` visited
- add `2` to result

Now:

```cpp
visited = [1, 1, 1, 1, 1, 1]
result = [0, 1, 3, 4, 5, 2]
```

Neighbors of `2` are `{0, 5}`.
- `0` is already visited.
- `5` is already visited.

No unvisited neighbor exists, so return to `dfs(5)`.

### Backtracking chain
Now notice what happens:
- `dfs(2)` finishes and returns to `dfs(5)`.
- `dfs(5)` has no more unvisited neighbors, so it returns to `dfs(4)`.
- `dfs(4)` finishes and returns to `dfs(1)`.
- `dfs(1)` finishes and returns to `dfs(0)`.
- Back at `dfs(0)`, the next neighbor is `2`, but it is already visited, so skip.
- `dfs(0)` ends.

Final result:

```cpp
[0, 1, 3, 4, 5, 2]
```

### What does this dry run prove?
This dry run proves the exact nature of DFS:
- It does not stay level-wise.
- It keeps following one branch fully before trying another.
- Backtracking is not a separate extra step; it is built into recursive return.
- The order depends on adjacency list order. If neighbors were stored in a different order, DFS traversal order could change.

That last point is very important: DFS order is not always unique. It depends on the order in which neighbors are explored.

---

## 3. Code

```cpp
class Solution {
private:
    void dfs(int node, vector<vector<int>> &adj, vector<int> &vis, vector<int> &ls) {
        vis[node] = 1;
        ls.push_back(node);

        for (auto it : adj[node]) {
            if (!vis[it])
                dfs(it, adj, vis, ls);
        }
    }

public:
    vector<int> dfs(vector<vector<int>> &adj) {
        int V = adj.size();

        vector<int> visited(V, 0);
        int start = 0;
        vector<int> result;

        dfs(start, adj, visited, result);

        return result;
    }
};
```

### Line-by-line explanation

#### `void dfs(int node, vector<vector<int>> &adj, vector<int>& vis, vector<int>& ls)`
This is the recursive helper function. It is responsible for performing DFS from the current node. It receives:
- the current node,
- the adjacency list,
- the visited array,
- and the traversal result list.

These are passed by reference so that changes remain shared across recursive calls.

#### `vis[node] = 1;`
As soon as the traversal enters a node, it marks that node as visited. This prevents revisiting and avoids cycles causing infinite recursion.

#### `ls.push_back(node);`
The node is immediately added to the traversal order. This means the traversal records the node at entry time.

#### `for (auto it : adj[node])`
The code now explores all neighbors of the current node. Every adjacent node gets a chance to extend the DFS path.

#### `if (!vis[it])`
Only unvisited neighbors are explored recursively. Visited neighbors are ignored because they have already been handled earlier in the traversal.

#### `dfs(it, adj, vis, ls);`
This is the heart of DFS. Instead of processing the neighbor later, the function immediately dives into that neighbor and explores its entire branch first.

That is exactly why the traversal becomes depth-first.

#### `int V = adj.size();`
Inside the public function, the number of vertices is taken from the adjacency list size.

#### `vector<int> visited(V, 0);`
A visited array is created for all vertices. Initially every node is unvisited.

#### `int start = 0;`
Traversal is defined to begin from node `0`.

#### `vector<int> result;`
This stores the final DFS traversal order.

#### `dfs(start, adj, visited, result);`
The helper function is called on the starting node. From here, recursion handles the entire connected component reachable from `0`.

#### `return result;`
After recursion completes, the final traversal order is returned.

### Why this implementation is correct
This DFS works correctly because it maintains these guarantees:
1. A node is visited only once.
2. Every recursive call fully explores one node’s unvisited descendants.
3. After a branch is finished, control automatically returns to the parent call.
4. Every node reachable from `0` is eventually added to the result.

These guarantees together create valid DFS traversal.

---

## 4. Complexity Analysis

### Time Complexity
The time complexity is:

```cpp
TC = O(V) + O(2E)
```
for an undirected graph stored as an adjacency list.

In simplified asymptotic form, this becomes:

```cpp
O(V + E)
```

But the expanded form is much better for understanding the logic.

### Why `O(V)` appears
Each vertex is visited only once because after `vis[node] = 1`, that node never triggers a fresh DFS call again.

So operations like:
- entering the recursive function for a node,
- marking visited,
- adding node to result,
- returning after all neighbors are processed,

happen once per vertex.

That contributes:

```cpp
O(V)
```

### Why `O(2E)` appears in an undirected graph
The neighbor loop is:

```cpp
for (auto it : adj[node])
```

Across the entire DFS, this loop scans all adjacency lists. In an undirected graph, every edge `(u, v)` appears twice:
- once in `adj[u]`,
- once in `adj[v]`.

So total adjacency entries become `2E`, and scanning all of them costs:

```cpp
O(2E)
```

Hence:

```cpp
TC = O(V) + O(2E)
```

### If the graph is directed
In a directed graph, each edge is stored only once in the adjacency list, so the time complexity becomes:

```cpp
O(V) + O(E)
```

### Why it is not `O(V * E)`
The code has recursion and a loop, so beginners sometimes think complexity should multiply. But that is not how this traversal behaves.

The loop at each node processes only that node’s adjacency list. When summed over all nodes, the total work across all such loops is just the total number of adjacency entries, which is `2E` for an undirected graph. So the work is additive, not multiplicative.

That is why DFS also runs in linear time relative to the graph size.

---

### Space Complexity
Now consider all memory usage carefully.

### 1. Visited array
```cpp
vector<int> visited(V, 0);
```
This stores one entry for each vertex:

```cpp
O(V)
```

### 2. Result array
```cpp
vector<int> result;
```
The result may store all `V` vertices in the traversal order:

```cpp
O(V)
```

### 3. Recursion stack space
This is the most important extra space in recursive DFS.

When DFS keeps going deeper, each recursive call stays on the function call stack until its subtree or branch is fully processed. If the graph has a long chain-like structure, the recursion depth can become as large as `V` in the worst case.

For example, in a path graph:
- `0 -> 1 -> 2 -> 3 -> ...`

DFS would keep calling deeper and deeper before returning, so stack space becomes:

```cpp
O(V)
```

### Total detailed understanding
If only the auxiliary space used to solve the problem is counted:
- `O(V)` for visited array
- `O(V)` for recursion stack in the worst case

So that becomes:

```cpp
SC = O(2V)
```

which simplifies to:

```cpp
O(V)
```

### If the output array is also counted
Then add one more `O(V)` for the result vector:

```cpp
O(V)  -> visited array
O(V)  -> recursion stack
O(V)  -> result array
```

So total becomes:

```cpp
SC = O(3V)
```

and asymptotically it is still:

```cpp
O(V)
```

### Important interview-style explanation
A neat and detailed explanation is:

- **Auxiliary space / extra space used to solve the problem:** `O(2V)` = visited array + recursion stack.
- **If including the returned traversal array also:** `O(3V)`.

This is the DFS counterpart of the detailed complexity style used in BFS.

---

## Final Understanding
DFS is fundamentally a deep-exploration strategy. The intuition comes from wanting to completely follow one path before considering alternative paths. Recursion fits naturally because every time the traversal moves to an unvisited neighbor, the exact same subproblem appears again: “perform DFS from this new node.”

That is why this algorithm becomes the natural solution when the problem involves:
- traversal by going deep first,
- exploring connected components,
- cycle detection patterns,
- topological-style ideas,
- and many graph problems where backtracking structure matters.
