# 🌳 Insert into Binary Search Tree (BST)

---

## 1. 🧠 Approach / Intuition

Insertion in a **Binary Search Tree (BST)** follows the same fundamental rule:

```
Left < Root < Right
```

### 💡 Key Idea:
We traverse the tree and find the **correct position** where the new value should be inserted such that BST property is maintained.

### 🚀 Step-by-step:

1. If tree is empty → create a new node and return it
2. Start from the root
3. Compare `val` with current node:
   - If `val > curr->val` → go to **right subtree**
   - Else → go to **left subtree**
4. Repeat until we find a NULL position
5. Insert the new node there

---

## 2. 💻 Code (Iterative)

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if (root == NULL)
            return new TreeNode(val);

        TreeNode* curr = root;

        while (true) {
            if (curr->val < val) {
                if (curr->right)
                    curr = curr->right;
                else {
                    curr->right = new TreeNode(val);
                    break;
                }
            } else {
                if (curr->left)
                    curr = curr->left;
                else {
                    curr->left = new TreeNode(val);
                    break;
                }
            }
        }

        return root;
    }
};
```

### 🔎 Explanation:
- If tree is empty → directly create node
- Traverse down the tree using a loop
- Compare and move left/right accordingly
- Insert when a NULL child is found

---

## 3. ⏱ Time & Space Complexity

### ⏳ Time Complexity
- **Best Case:** O(1) → inserting at root (empty tree)
- **Average Case:** O(log N) → balanced BST
- **Worst Case:** O(N) → skewed tree (like a linked list)

### 📦 Space Complexity
- **O(1)** → iterative approach (no recursion stack)

---

## 📌 Summary

- Traverse the BST like searching
- Insert at the correct NULL position
- Maintains BST property
- Performance depends on tree height

---

Happy Coding 🚀

