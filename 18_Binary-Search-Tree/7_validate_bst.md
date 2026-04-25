# Validate Binary Search Tree

---

## 1. Intuition / Approach

### BST Property

For a tree to be a valid BST, every node must satisfy:
- All nodes in its **left subtree** have values **strictly less than** the node
- All nodes in its **right subtree** have values **strictly greater than** the node

### The Common Mistake

A naive approach only checks **immediate children**:
```
node->left->val < node->val   ✅ checks only direct child
```

This fails on cases like:
```
    5
   / \
  1   4
     / \
    3   6
```
Node 4's left child is 3, and `3 < 4` ✅ locally — but 3 is in the **right subtree of 5**, so it must be `> 5`. The tree is **invalid**, but a naive check misses this.

---

### The Fix — Valid Range `[minVal, maxVal]`

Pass a **valid range** down the recursion. Every node must satisfy:

```
minVal < node->val < maxVal
```

- When going **left** → the current node's value becomes the new **maxVal** (left subtree must be smaller)
- When going **right** → the current node's value becomes the new **minVal** (right subtree must be larger)

Start with `[LONG_MIN, LONG_MAX]` at root so no constraint is violated initially.

---

### Why `LONG_MIN` / `LONG_MAX` instead of `INT_MIN` / `INT_MAX`?

Node values can be `INT_MIN` or `INT_MAX` themselves. If we used `INT_MIN` as the initial lower bound and a node had value `INT_MIN`, the check `root->val <= minVal` would incorrectly return `false`. Using `long` bounds avoids this edge case.

---

### Dry Run

```
    5
   / \
  1   7
     / \
    6   8
```

```
isValidBST(5, LONG_MIN, LONG_MAX)
  5 in (LONG_MIN, LONG_MAX) ✅
  ├── isValidBST(1, LONG_MIN, 5)
  │     1 in (LONG_MIN, 5) ✅
  │     ├── isValidBST(NULL) → true
  │     └── isValidBST(NULL) → true
  └── isValidBST(7, 5, LONG_MAX)
        7 in (5, LONG_MAX) ✅
        ├── isValidBST(6, 5, 7)
        │     6 in (5, 7) ✅ → true
        └── isValidBST(8, 7, LONG_MAX)
              8 in (7, LONG_MAX) ✅ → true

Result: true ✅
```

Now with the **invalid tree**:
```
    5
   / \
  1   4
     / \
    3   6
```

```
isValidBST(4, 5, LONG_MAX)
  4 >= 5? NO — 4 < 5 → violates minVal constraint → return false ❌
```

---

## 2. Code

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        // Start with full long range to handle INT_MIN/INT_MAX edge cases
        return isValidBST(root, LONG_MIN, LONG_MAX);
    }

    bool isValidBST(TreeNode* root, long minVal, long maxVal) {
        // Empty tree / leaf's child is always valid
        if(root == nullptr) return true;

        // Node must be strictly within (minVal, maxVal)
        if(root->val >= maxVal || root->val <= minVal)
            return false;

        // Left subtree: all values must be < root->val → new maxVal = root->val
        // Right subtree: all values must be > root->val → new minVal = root->val
        return isValidBST(root->left,  minVal,    root->val) &&
               isValidBST(root->right, root->val, maxVal);
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited exactly once |
| **Space** | `O(H)` auxiliary | Recursive call stack depth = height of tree; `O(log N)` balanced, `O(N)` skewed |

> `N` = total number of nodes | `H` = height of tree
