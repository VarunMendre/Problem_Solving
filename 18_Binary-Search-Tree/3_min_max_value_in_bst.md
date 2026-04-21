# 🌳 Minimum & Maximum Value in Binary Search Tree (BST)

---

## 🔽 Minimum Value in BST

### 1. 🧠 Approach / Intuition

In a **Binary Search Tree**, the smallest value always lies in the **leftmost node**.

Why?
Because BST follows:

```
Left < Root < Right
```

So if we keep moving to the **left child**, we eventually reach the smallest element.

---

### 2. 💻 Code (Recursive)

```cpp
int minVal(Node* root) {
    if (!root)
        return -1;

    if (root->left)
        return minVal(root->left);

    return root->data;
}
```

### 🔎 Explanation:
- Base case: if tree is empty → return -1
- If left child exists → go left recursively
- If no left child → current node is the minimum

---

### 2. 💻 Code (Iterative)

```cpp
int minVal(Node* root) {
    if (!root)
        return -1;

    while (root->left) {
        root = root->left;
    }

    return root->data;
}
```

### 🔎 Explanation:
- Traverse left until `root->left == NULL`
- That node is the minimum

---

### 3. ⏱ Time & Space Complexity

| Case | Time | Space |
|------|------|-------|
| Best (root is min) | O(1) | O(1) |
| Average (balanced) | O(log N) | O(1) iterative / O(log N) recursive |
| Worst (skewed) | O(N) | O(1) iterative / O(N) recursive |

---

## 🔼 Maximum Value in BST

### 1. 🧠 Approach / Intuition

The largest value in a BST lies in the **rightmost node**.

Because:

```
Left < Root < Right
```

So we keep moving to the **right child**.

---

### 2. 💻 Code (Recursive)

```cpp
int maxVal(Node* root) {
    if (!root)
        return -1;

    if (root->right)
        return maxVal(root->right);

    return root->data;
}
```

---

### 2. 💻 Code (Iterative)

```cpp
int maxVal(Node* root) {
    if (!root)
        return -1;

    while (root->right) {
        root = root->right;
    }

    return root->data;
}
```

---

### 🔎 Explanation:
- Move right until `root->right == NULL`
- That node is the maximum

---

### 3. ⏱ Time & Space Complexity

| Case | Time | Space |
|------|------|-------|
| Best (root is max) | O(1) | O(1) |
| Average (balanced) | O(log N) | O(1) iterative / O(log N) recursive |
| Worst (skewed) | O(N) | O(1) iterative / O(N) recursive |

---

## 📌 Summary

- **Minimum → Go LEFT till NULL**
- **Maximum → Go RIGHT till NULL**
- Works efficiently in balanced BST
- Iterative approach is more space-efficient

---

Happy Coding 🚀

