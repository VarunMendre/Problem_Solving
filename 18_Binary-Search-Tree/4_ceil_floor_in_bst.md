# 🌳 Ceil & Floor in Binary Search Tree (BST)

---

## 🔼 Ceil in BST

### 1. 🧠 Approach / Intuition

The **ceil** of a value `x` in a BST is the **smallest value ≥ x**.

### 💡 Key Idea:
- If node value == x → that is the ceil
- If node value < x → ceil must be in **right subtree**
- If node value > x → this node *can be a ceil*, but check left for a smaller valid value

---

### 2. 💻 Code (Iterative)

```cpp
int findCeil(BinaryTreeNode<int> *node, int x) {
    int ceil = -1;

    while (node) {
        if (node->data == x) {
            ceil = node->data;
            return ceil;
        }

        if (x > node->data) {
            node = node->right;
        } else {
            ceil = node->data;
            node = node->left;
        }
    }

    return ceil;
}
```

### 🔎 Explanation:
- Move right if current value is smaller than x
- Otherwise store current node as potential ceil and move left
- Keep updating to get the **smallest valid ≥ x**

---

### 3. ⏱ Time & Space Complexity

| Case | Time | Space |
|------|------|-------|
| Best | O(1) | O(1) |
| Average | O(log N) | O(1) |
| Worst | O(N) | O(1) |

---

## 🔽 Floor in BST

### 1. 🧠 Approach / Intuition

The **floor** of a value `x` in a BST is the **largest value ≤ x**.

### 💡 Key Idea:
- If node value == x → that is the floor
- If node value > x → floor must be in **left subtree**
- If node value < x → this node *can be a floor*, but check right for a larger valid value

---

### 2. 💻 Code (Iterative)

```cpp
int floorInBST(TreeNode<int> *root, int x) {
    if (root == NULL)
        return -1;

    int floor = -1;

    while (root) {
        if (root->val == x) {
            floor = root->val;
            return floor;
        }

        if (x < root->val) {
            root = root->left;
        } else {
            floor = root->val;
            root = root->right;
        }
    }

    return floor;
}
```

### 🔎 Explanation:
- Move left if current value is greater than x
- Otherwise store current node as potential floor and move right
- Keep updating to get the **largest valid ≤ x**

---

### 3. ⏱ Time & Space Complexity

| Case | Time | Space |
|------|------|-------|
| Best | O(1) | O(1) |
| Average | O(log N) | O(1) |
| Worst | O(N) | O(1) |

---

## 📌 Summary

- **Ceil → Smallest value ≥ x (move left when possible)**
- **Floor → Largest value ≤ x (move right when possible)**
- Both use BST property to skip unnecessary nodes
- Efficient in balanced trees

---

Happy Coding 🚀

