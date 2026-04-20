# 🔍 Search in Binary Search Tree (BST)

---

## 1. 🧠 Approach / Intuition

The key idea behind searching in a **Binary Search Tree (BST)** comes from its defining property:

```
Left Subtree < Node < Right Subtree
```

Because of this property, we don’t need to check every node (like in a normal binary tree). Instead, we can **eliminate half of the tree at each step**, similar to Binary Search.

### 💡 Step-by-step intuition:

1. Start from the **root node**
2. Compare the target value (`val`) with the current node:
   - If `val == root->val` → ✅ Found the node
   - If `val < root->val` → Move to **left subtree**
   - If `val > root->val` → Move to **right subtree**
3. Repeat until:
   - Node is found
   - OR we reach `NULL` (value not present)

### 🚀 Why this is efficient?
At every step, we **discard half of the tree**, reducing the search space quickly.

---

## 2. 💻 Code (C++)

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
    TreeNode* searchBST(TreeNode* root, int val) {
        while(root != NULL && root->val != val) {
            // If value is smaller, go left; otherwise go right
            root = val < root->val ? root->left : root->right;
        }

        return root;
    }
};
```

### 🔎 Code Explanation:
- We use a **while loop** to traverse the tree iteratively
- At each step, we decide direction using a **ternary operator**
- Loop stops when:
  - We find the node (`root->val == val`)
  - OR `root` becomes `NULL`

---

## 3. ⏱ Time & Space Complexity

### ⏳ Time Complexity
- **Best Case:** `O(1)` → Element found at root
- **Average Case:** `O(log N)` → Balanced BST
- **Worst Case:** `O(N)` → Skewed tree (like a linked list)

### 📦 Space Complexity
- **O(1)** → Iterative solution (no recursion stack used)

---

## 📌 Summary
- Uses BST property to avoid unnecessary traversal
- Works like Binary Search on trees
- Efficient in balanced trees
- Iterative approach keeps space usage minimal

---

Happy Coding! 🚀

