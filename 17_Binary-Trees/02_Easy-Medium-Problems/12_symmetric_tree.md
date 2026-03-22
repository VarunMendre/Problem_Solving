# Symmetric Tree

---

## 1. Intuition / Approach

A tree is **symmetric** if it is a **mirror of itself** around the center — meaning the left subtree is a mirror reflection of the right subtree.

### What makes two subtrees a mirror?

Two nodes `left` and `right` are mirrors of each other if:
1. Both are `null` → ✅ symmetric
2. One is `null`, the other isn't → ❌ not symmetric
3. Both exist but `left->val != right->val` → ❌ not symmetric
4. Both exist and values match → recursively check:
   - `left->left` mirrors `right->right` (outer pair)
   - `left->right` mirrors `right->left` (inner pair)

### Visual

```
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

We compare:
```
isSymmetricHelper(2, 2)
  ├── isSymmetricHelper(3, 3)  ← outer pair ✅
  └── isSymmetricHelper(4, 4)  ← inner pair ✅
```

---

### The Mirror Recursion

```
left->left  ↔  right->right   (outer children)
left->right ↔  right->left    (inner children)
```

This is the core of the solution — instead of comparing a node with itself, we compare **symmetrically opposite nodes** across the center axis.

---

## 2. Code

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        // An empty tree is symmetric by definition
        return root == nullptr || isSymmetricHelper(root->left, root->right);
    }

    bool isSymmetricHelper(TreeNode* left, TreeNode* right) {
        // Both null → mirror ✅
        // One null, one not → not mirror ❌
        if (left == nullptr || right == nullptr) {
            return left == right;
        }

        // Values must match for a valid mirror
        if (left->val != right->val)
            return false;

        // Outer pair: left->left must mirror right->right
        // Inner pair: left->right must mirror right->left
        return isSymmetricHelper(left->left, right->right)
            && isSymmetricHelper(left->right, right->left);
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space** | `O(N)` | Recursive call stack — `O(H)` where H = height; worst case `O(N)` for a skewed tree |

> `N` = total number of nodes | `H` = height of the tree  
> `H` is `O(log N)` for a balanced tree and `O(N)` for a skewed tree
