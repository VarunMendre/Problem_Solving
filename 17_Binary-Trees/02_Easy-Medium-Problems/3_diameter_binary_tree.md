# Diameter of Binary Tree

## Problem
The **diameter of a binary tree** is the length of the longest path between any two nodes in the tree. 

- The path **may or may not pass through the root**.
- The diameter is measured in **number of edges**.

Example:

```
        1
       / \
      2   3
     / \
    4   5
```

Longest path: `4 → 2 → 5` or `4 → 2 → 1 → 3`

Diameter = **3 edges** (depending on definition used in platform).

---

# 1. Brute Force Approach

### Idea
For every node:

1. Calculate the **height of left subtree**
2. Calculate the **height of right subtree**
3. Diameter through that node =

```
leftHeight + rightHeight
```

4. Keep updating the global maximum diameter.

The problem is that **height is recalculated multiple times**, which increases the time complexity.

---

### Code

```cpp
class Solution {
public:
    int diameter = 0;

    int calculateHeight(TreeNode* root) {
        if (root == nullptr)
            return 0;

        int leftHeight = calculateHeight(root->left);
        int rightHeight = calculateHeight(root->right);

        diameter = max(diameter, leftHeight + rightHeight);

        return 1 + max(leftHeight, rightHeight);
    }

    int diameterOfBinaryTree(TreeNode* root) {
        calculateHeight(root);
        return diameter;
    }
};
```

---

### Time Complexity

```
O(N²)
```

Because:
- For every node we calculate height
- Height calculation itself takes **O(N)** in worst case

---

### Space Complexity

```
O(H)
```

Where:

- **H = height of tree**
- Space is used by the recursion stack.

---

# 2. Optimal Approach

### Key Idea
Instead of calculating height **separately**, we compute:

- **Height of subtree**
- **Diameter at that node**

**in the same DFS traversal**.

This avoids recomputing heights multiple times.

---

### Algorithm

For every node:

1. Recursively calculate **left subtree height**
2. Recursively calculate **right subtree height**
3. Update diameter

```
maxi = max(maxi, leftHeight + rightHeight)
```

4. Return height

```
1 + max(leftHeight, rightHeight)
```

---

### Code

```cpp
class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int maxi = 0;
        findHeight(root, maxi);
        return maxi;
    }

    int findHeight(TreeNode* root, int &maxi) {
        if(root == nullptr) return 0;

        int leftHeight = findHeight(root->left, maxi);
        int rightHeight = findHeight(root->right, maxi);

        maxi = max(maxi, leftHeight + rightHeight);

        return 1 + max(leftHeight , rightHeight);
    }
};
```

---

### Time Complexity

```
O(N)
```

Reason:
- Every node is visited **only once**.

---

### Space Complexity

```
O(H)
```

Where:

- **H = height of the tree**
- Due to recursion stack.

Note: Sometimes people write **O(1)** extra space because no extra data structures are used, but the **actual auxiliary recursion stack is O(H)**.

---

# Important Interview Insight

Many binary tree problems follow this **pattern**:

```
Return height
Update global answer during recursion
```

This pattern is used in problems like:

- Diameter of Binary Tree
- Balanced Binary Tree
- Maximum Path Sum

If you remember this pattern, many tree problems become