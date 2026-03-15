# Binary Tree Traversals – Implementation Guide (C++)

This document explains the **implementation of the four fundamental binary tree traversals** commonly asked in coding interviews and platforms like LeetCode.

Traversals covered:

1. Preorder Traversal (DFS)
2. Postorder Traversal (DFS)
3. Inorder Traversal (DFS)
4. Level Order Traversal (BFS)

All traversals visit every node once.

Time Complexity: **O(N)**  
Space Complexity: **O(N)**

Where **N = number of nodes in the tree**.

---

# Binary Tree Node Structure

Typical structure used in problems:

```cpp
/**
 * Definition for a binary tree node.
 */
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;

    TreeNode() : val(0), left(nullptr), right(nullptr) {}

    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}

    TreeNode(int x, TreeNode *left, TreeNode *right)
        : val(x), left(left), right(right) {}
};
```

Each node contains:

- value (`val`)
- pointer to left child
- pointer to right child

---

# 1. Preorder Traversal

Traversal Order:

```
Root → Left → Right
```

Concept:

1. Process current node
2. Traverse left subtree
3. Traverse right subtree

Example Tree

```
        1
       / \
      2   3
     / \
    4   5
```

Preorder Output:

```
1 2 4 5 3
```

Implementation:

```cpp
class Solution {
    vector<int> answer;

public:

    vector<int> preorderTraversal(TreeNode* root) {

        if(root == NULL)
            return answer;

        answer.push_back(root->val);

        preorderTraversal(root->left);
        preorderTraversal(root->right);

        return answer;
    }
};
```

---

# 2. Postorder Traversal

Traversal Order:

```
Left → Right → Root
```

Concept:

1. Traverse left subtree
2. Traverse right subtree
3. Process root node

Example Tree

```
        1
       / \
      2   3
     / \
    4   5
```

Postorder Output

```
4 5 2 3 1
```

Implementation:

```cpp
class Solution {

    vector<int> answer;

public:

    vector<int> postorderTraversal(TreeNode* root) {

        if(root == NULL)
            return answer;

        postorderTraversal(root->left);
        postorderTraversal(root->right);

        answer.push_back(root->val);

        return answer;
    }
};
```

---

# 3. Inorder Traversal

Traversal Order:

```
Left → Root → Right
```

Concept:

1. Traverse left subtree
2. Process root
3. Traverse right subtree

Example Tree

```
        1
       / \
      2   3
     / \
    4   5
```

Inorder Output

```
4 2 5 1 3
```

Important Property:

For a **Binary Search Tree (BST)**, inorder traversal produces **sorted output**.

Implementation:

```cpp
class Solution {

    vector<int> answer;

public:

    vector<int> inorderTraversal(TreeNode* root) {

        if(root == NULL)
            return answer;

        inorderTraversal(root->left);

        answer.push_back(root->val);

        inorderTraversal(root->right);

        return answer;
    }
};
```

---

# 4. Level Order Traversal (BFS)

Traversal Order:

```
Level by Level
```

Concept:

Instead of going deep like DFS, we process nodes **level by level using a queue**.

Example Tree

```
        1
       / \
      2   3
     / \
    4   5
```

Level Order Output

```
[ [1], [2,3], [4,5] ]
```

Implementation:

```cpp
class Solution {
public:

    vector<vector<int>> levelOrder(TreeNode* root) {

        vector<vector<int>> ds;

        if(root == NULL)
            return ds;

        queue<TreeNode*> q;
        q.push(root);

        while(!q.empty()) {

            int size = q.size();
            vector<int> level;

            for(int i = 0; i < size; i++) {

                TreeNode* node = q.front();
                q.pop();

                if(node->left != NULL)
                    q.push(node->left);

                if(node->right != NULL)
                    q.push(node->right);

                level.push_back(node->val);
            }

            ds.push_back(level);
        }

        return ds;
    }
};
```

---

# Complexity Analysis

For all traversals:

### Time Complexity

```
O(N)
```

Every node is visited exactly once.

### Space Complexity

```
O(N)
```

Reason:

- recursion stack in DFS
- queue storage in BFS

---

# Key Interview Insight

These four traversals form the **foundation of almost all binary tr