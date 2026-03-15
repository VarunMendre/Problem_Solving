# Binary Tree Level Order Traversal

## 1. Approach / Intuition

Level Order Traversal means visiting the nodes of a binary tree level by level from left to right.

The best way to achieve this is by using Breadth First Search (BFS). BFS naturally processes nodes level by level using a Queue data structure.

Why Queue?
A queue follows FIFO (First In First Out). The node that is discovered first is processed first. This behavior perfectly matches how we want to traverse tree levels.

Steps of the idea:

1. If the root is NULL, return an empty result.
2. Create a queue and push the root node.
3. While the queue is not empty:
   - Determine the number of nodes present in the current level using queue.size().
   - Create a temporary vector to store values of that level.
   - Traverse all nodes of that level.
4. For each node:
   - Remove the node from the queue.
   - Add its value to the level vector.
   - Push its left child to the queue if it exists.
   - Push its right child to the queue if it exists.
5. After processing all nodes of that level, store the level vector in the final result.
6. Continue until the queue becomes empty.

This guarantees that nodes are processed level by level.

Example Tree:

        3
       / \
      9  20
         / \
        15  7

Traversal Output:

[
 [3],
 [9,20],
 [15,7]
]

Each inner vector represents one level of the tree.

---

## 2. Code

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ds;

        if(root == NULL) return ds;

        queue<TreeNode*> q;
        q.push(root);

        while(!q.empty()) {
            int size = q.size();
            vector<int> level;

            for(int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                if(node->left != NULL) q.push(node->left);
                if(node->right != NULL) q.push(node->right);

                level.push_back(node->val);
            }

            ds.push_back(level);
        }

        return ds;
    }
};
```

Explanation of the code:

1. ds stores the final answer where each vector represents one level.

2. If the tree is empty (root == NULL), return an empty result.

3. A queue is created to help perform BFS traversal.

4. Root node is pushed into the queue because traversal always starts from the root.

5. The while loop runs until the queue becomes empty.

6. size = q.size() tells us how many nodes belong to the current level.

7. A new vector level is created to store nodes of that level.

8. The for loop processes exactly "size" nodes so we only process the current level.

9. q.front() gets the first node in the queue.

10. q.pop() removes that node after processing.

11. If the node has a left child, push it into the queue.

12. If the node has a right child, push it into the queue.

13. Add the node's value to the level vector.

14. After finishing the loop, push the level vector into ds.

15. Continue the process until the queue becomes empty.

Finally return ds which contains all levels of the tree.

---

## 3. Time & Space Complexity

Time Complexity: O(N)

Where N is the number of nodes in the binary tree.

Reason:
Each node of the tree is visited exactly once. For every node we perform constant operations like push, pop, and insertion.

Total operations = proportional to number of nodes.

Therefore:

Time Complexity = O(N)


Space Complexity: O(N)

There are two reasons for this space usage.

1. Queue Storage

In the worst case, the queue may store an entire level of the tree. In a complete binary tree, the last level may contain about N/2 nodes.

Therefore queue space = O(N).

2. Result Storage

The output vector ds stores all nodes of the tree as part of the result.

Therefore result storage = O(N).

Total Space Complexity = O