# Amount of Time to Burn a Binary Tree

---

## 1. Intuition / Approach

A fire starts at node `start` and spreads to all **adjacent nodes** (left child, right child, parent) every second. We need to find the **total time** for the entire tree to burn.

This is essentially: **"find the maximum distance from `start` to any node in the tree"** — because the last node to burn is the farthest one.

---

### Why this is just "Nodes at Distance K" in disguise

The fire spreading per second is identical to a **BFS expanding one level per second**. The total time = the number of levels BFS takes to exhaust all nodes = **max distance from `start` to any node**.

---

### Two-Step Strategy

**Step 1 — `bfsToMapParent()`: Build parent map + find target node**

BFS from root to:
- Record every node's parent in `mpp`
- Identify and return the node whose `val == start`

This gives us **upward movement** ability (tree → undirected graph).

**Step 2 — `findMaxDistance()`: Multi-directional BFS from target**

Start BFS from `start` node, expanding in all 3 directions (left, right, parent) each second. Use `isBurned` flag — if **any** node was newly added to the queue in this second, increment `time`.

Stop when no new nodes can be reached → return `time`.

---

### The `isBurned` Flag

```cpp
int isBurned = 0;
// ... if any neighbor is newly visited:
isBurned = 1;

if(isBurned) time++;
```

`isBurned` tracks whether **at least one new node** burned this second. This avoids counting the final empty round as an extra second.

---

### Dry Run

```
        1
       / \
      2   3
     / \   \
    4   5   6

start = 2
```

```
parent map: {2:1, 3:1, 4:2, 5:2, 6:3}
target = node(2)

BFS from node(2):
Second 0 (initial): queue = [2]
  Neighbors of 2: left=4✅, right=5✅, parent=1✅ → isBurned=1 → time=1
Second 1: queue = [4, 5, 1]
  4: leaf, no unvisited neighbors
  5: leaf, no unvisited neighbors
  1: right=3✅ → isBurned=1 → time=2
Second 2: queue = [3]
  3: right=6✅ → isBurned=1 → time=3
Second 3: queue = [6]
  6: leaf, no unvisited neighbors → isBurned=0 → time stays 3

return 3 ✅
```

---

## 2. Code

```cpp
class Solution {
private:
    int findMaxDistance(TreeNode* target,
                        unordered_map<TreeNode*, TreeNode*>& mpp) {

        queue<TreeNode*> que;
        que.push(target);

        unordered_map<TreeNode*, bool> visited;
        visited[target] = true;

        int time = 0;

        while(!que.empty()) {
            int size = que.size();
            int isBurned = 0;   // tracks if any new node burned this second

            for(int i = 0; i < size; i++) {
                auto node = que.front();
                que.pop();

                // Spread to left child
                if(node->left && !visited[node->left]) {
                    isBurned = 1;
                    visited[node->left] = true;   // mark left child visited
                    que.push(node->left);
                }
                // Spread to right child
                if(node->right && !visited[node->right]) {
                    isBurned = 1;
                    visited[node->right] = true;  // mark right child visited
                    que.push(node->right);
                }
                // Spread to parent (upward movement via parent map)
                if(mpp[node] && !visited[mpp[node]]) {
                    isBurned = 1;
                    visited[mpp[node]] = true;
                    que.push(mpp[node]);
                }
            }

            // Only count this second if at least one new node burned
            if(isBurned) time++;
        }

        return time;
    }

    // BFS to build parent map and locate the start node
    TreeNode* bfsToMapParent(TreeNode* root,
                              unordered_map<TreeNode*, TreeNode*>& mpp,
                              int start) {
        queue<TreeNode*> que;
        que.push(root);

        TreeNode* res = nullptr;

        while(!que.empty()) {
            TreeNode* node = que.front();
            que.pop();

            // Identify the target node
            if(node->val == start)
                res = node;

            if(node->left) {
                mpp[node->left] = node;   // left child's parent = current node
                que.push(node->left);
            }
            if(node->right) {
                mpp[node->right] = node;  // right child's parent = current node
                que.push(node->right);
            }
        }

        return res;
    }

public:
    int amountOfTime(TreeNode* root, int start) {
        unordered_map<TreeNode*, TreeNode*> mpp;

        // Step 1: build parent map + find start node
        TreeNode* target = bfsToMapParent(root, mpp, start);

        // Step 2: BFS from start node — count seconds until fully burned
        return findMaxDistance(target, mpp);
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity — `O(N)`

| Phase | Work done | Cost |
|---|---|---|
| `bfsToMapParent()` | Visits every node once, records parent | `O(N)` |
| `findMaxDistance()` | Visits every node at most once (visited map prevents revisits) | `O(N)` |

**Total: `O(N) + O(N)` = `O(N)`**

---

### Space Complexity — `O(N)`

| Structure | What it stores | Size |
|---|---|---|
| `mpp` parent map | One entry per node (except root) | `O(N)` |
| `visited` map | One entry per visited node | `O(N)` |
| BFS `queue` | At most one full level at a time | `O(N)` worst case |

**Total: `O(N)`**

---

## 4. Similarity to "Nodes at Distance K"

| | Nodes at Distance K | Amount of Time to Burn |
|---|---|---|
| Start point | Given `target` node | Find node with `val == start` |
| Stop condition | Stop at level `k` | Run until queue is empty |
| What we collect | All nodes at level `k` | Count of levels (seconds) |
| Core mechanism | Multi-directional BFS | Multi-directional BFS |

Both problems use the **exact same parent-map + multi-directional BFS** pattern. The only difference is what we measure at the end.
