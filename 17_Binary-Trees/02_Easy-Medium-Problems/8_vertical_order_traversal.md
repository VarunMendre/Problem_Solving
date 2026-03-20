# Binary Tree Vertical Order Traversal

---

## 1. Intuition / Approach

The goal is to return nodes of a binary tree grouped by their **vertical column**, where within each column nodes are sorted by **level (top to bottom)**, and nodes at the same level and column are sorted by **value**.

### Coordinate System

Assign every node a `(x, y)` coordinate:
- Root starts at `(0, 0)`
- Going **left** → `x - 1`, `y + 1`
- Going **right** → `x + 1`, `y + 1`

So `x` represents the **vertical column** and `y` represents the **depth/level**.

```
        1  (x=0, y=0)
       / \
      2   3     (x=-1,y=1)  (x=1,y=1)
     / \   \
    4   5   6   (x=-2,y=2) (x=0,y=2) (x=2,y=2)
```

### Data Structure Choice

```
map<int, map<int, multiset<int>>>
  ^         ^         ^
  x-axis    y-axis    node values (sorted, duplicates allowed)
```

- **Outer `map<int, ...>`** — keys are x-coordinates, auto-sorted left to right (negative → positive).
- **Inner `map<int, ...>`** — keys are y-coordinates (levels), auto-sorted top to bottom.
- **`multiset<int>`** — stores node values sorted, handles duplicates at the same `(x, y)`.

### BFS for Level-by-Level Traversal

We use a **BFS queue** where each entry carries the node along with its `(x, y)` coordinates. BFS naturally processes nodes level by level, but since we store by coordinates in the map, the ordering is fully handled by the map structure — not by BFS order.

### Building the Result

After BFS, we iterate over the outer map (columns left to right), and for each column iterate over the inner map (levels top to bottom), collecting all values from the multiset into a single column vector.

---

## 2. Code

```cpp
class Solution {
public:
    vector<vector<int>> verticalTraversal(TreeNode* root) {

        // map: x-axis -> (y-axis -> sorted node values)
        map<int, map<int, multiset<int>>> nodes;

        // BFS queue: {node, {x-coordinate, y-coordinate}}
        queue<pair<TreeNode*, pair<int, int>>> todo;

        todo.push({root, {0, 0}});

        while (!todo.empty()) {
            auto p = todo.front();
            todo.pop();

            TreeNode* node = p.first;
            int xaxis = p.second.first;   // vertical column
            int yaxis = p.second.second;  // depth / level

            // Insert node value at its (x, y) position
            nodes[xaxis][yaxis].insert(node->val);

            // Left child: column - 1, level + 1
            if (node->left) {
                todo.push({node->left, {xaxis - 1, yaxis + 1}});
            }
            // Right child: column + 1, level + 1
            if (node->right) {
                todo.push({node->right, {xaxis + 1, yaxis + 1}});
            }
        }

        // Build result: iterate columns (left to right), then levels (top to bottom)
        vector<vector<int>> ans;
        for (auto p : nodes) {
            vector<int> col;
            for (auto q : p.second) {
                // Append all values at this (column, level) — already sorted by multiset
                col.insert(col.end(), q.second.begin(), q.second.end());
            }
            ans.push_back(col);
        }

        return ans;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Each node is inserted into a `map` and `multiset` — both `O(log N)` per operation, over `N` nodes |
| **Space** | `O(N)` | The map stores all `N` nodes; BFS queue holds at most one level at a time |

> `N` = total number of nodes in the tree

**Why not `O(N)`?**  
The nested `map` insertions cost `O(log N)` each due to tree-based ordering, making the total `O(N log N)` rather than linear.
