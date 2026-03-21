# Top View of a Binary Tree

---

## 1. Intuition / Approach

The **top view** of a binary tree is the set of nodes visible when the tree is viewed from **directly above**. For each vertical column, only the **first node encountered** (topmost) is visible.

### Coordinate System

Assign every node a **vertical column** number:
- Root starts at `vertical = 0`
- Going **left** → `vertical - 1`
- Going **right** → `vertical + 1`

```
        1       vertical = 0
       / \
      2   3     vertical = -1, 1
     / \   \
    4   5   6   vertical = -2, 0, 2
```

Top view = one node per vertical column = `[4, 2, 1, 3, 6]`

---

### Why BFS and not DFS?

BFS processes nodes **level by level (top to bottom)**. This guarantees that the **first time** we visit a vertical column, it's always the topmost node for that column.

With DFS, we might reach a deeper node in the same column before the shallower one, requiring extra level tracking.

---

### Key Logic

```cpp
if(mpp.find(vertical) == mpp.end()) {
    mpp[vertical] = node->data;
}
```

We only insert into the map if the vertical column has **not been seen before**. Since BFS goes top to bottom, the first node at any vertical is always the topmost — so we never overwrite it.

---

### Data Structures Used

| Structure | Purpose |
|---|---|
| `map<int, int>` | Maps each vertical column → topmost node value (auto-sorted by column) |
| `queue<pair<Node*, int>>` | BFS queue carrying each node along with its vertical coordinate |

---

## 2. Code

```cpp
class Solution {
  public:
    vector<int> topView(Node *root) {
        vector<int> ans;
        if(root == nullptr) return ans;

        // vertical column → topmost node's value
        map<int, int> mpp;

        // BFS queue: {node, vertical}
        queue<pair<Node*, int>> todo;
        todo.push({root, 0});

        while(!todo.empty()) {
            auto it = todo.front();
            todo.pop();

            Node* node = it.first;
            int vertical = it.second;

            // Only insert if this vertical hasn't been seen yet
            // BFS guarantees this is always the topmost node
            if(mpp.find(vertical) == mpp.end()) {
                mpp[vertical] = node->data;
            }

            // Left child: column - 1
            if(node->left) {
                todo.push({node->left, vertical - 1});
            }

            // Right child: column + 1
            if(node->right) {
                todo.push({node->right, vertical + 1});
            }
        }

        // map is sorted by key (vertical), so result is left to right
        for(auto it : mpp) {
            ans.push_back(it.second);
        }

        return ans;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Each node is processed once in BFS — `O(N)`, but each `map` insertion/lookup costs `O(log N)` |
| **Space** | `O(N)` | Map stores at most `N` entries; BFS queue holds at most one level at a time |

> `N` = total number of nodes in the tree

---

## 4. Top View vs Vertical Order Traversal

| | Top View | Vertical Order |
|---|---|---|
| Nodes per column | **Only the first (topmost)** | **All nodes** |
| Map value type | `int` (single value) | `map<int, multiset<int>>` (all values by level) |
| Need level tracking? | ❌ No | ✅ Yes |
