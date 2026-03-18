# Binary Tree Zigzag Level Order Traversal

---

## 1. Intuition / Approach

The goal is to traverse a binary tree **level by level**, but alternate the direction of each level:
- **Level 0 (root):** left → right
- **Level 1:** right → left
- **Level 2:** left → right
- ... and so on.

**Core Idea — BFS with a direction flag:**

We use a standard **BFS (Breadth-First Search)** with a queue. For each level, instead of simply appending values to a list, we use a **pre-sized row vector** and place each node's value at a calculated index based on the current direction.

- If direction is **left → right**: place value at index `i`
- If direction is **right → left**: place value at index `size - 1 - i`

This way, nodes are always enqueued in the natural left-to-right order, but values are *written* into the row in the correct zigzag position. After processing each level, we **flip** the `leftToRight` boolean.

**Why this works cleanly:**  
We avoid reversing arrays or using a deque for insertion from both ends. The index trick does the zigzagging in-place within a fixed-size vector.

---

## 2. Code

```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> result;
        if (root == nullptr) {
            return result;
        }

        queue<TreeNode*> nodesQueue;
        nodesQueue.push(root);

        bool leftToRight = true;

        while (!nodesQueue.empty()) {
            int size = nodesQueue.size();
            vector<int> row(size);           // pre-sized row for current level

            for (int i = 0; i < size; i++) {
                auto node = nodesQueue.front();
                nodesQueue.pop();

                // place value at correct index based on direction
                int index = (leftToRight) ? i : (size - 1 - i);
                row[index] = node->val;

                if (node->left)
                    nodesQueue.push(node->left);
                if (node->right)
                    nodesQueue.push(node->right);
            }

            leftToRight = !leftToRight;      // flip direction for next level
            result.push_back(row);
        }

        return result;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space** | `O(N)` | Queue holds at most `N/2` nodes at a time (last level), and the result stores all `N` values |

 `N` = total number of nodes in the tree
