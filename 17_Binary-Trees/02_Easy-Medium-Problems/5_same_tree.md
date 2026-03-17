# Same Tree (Binary Tree)

## Problem
Given two binary trees `p` and `q`, check whether they are **identical**.

Two trees are identical if:
- They have the **same structure**
- Corresponding nodes have the **same values**

---

## Core Idea
Use **DFS recursion** to compare both trees **node by node**.

At each step:
1. If both nodes are `NULL` → they are identical
2. If one is `NULL` and the other is not → not identical
3. If values differ → not identical
4. Otherwise, recursively check **left** and **right** subtrees

---

## Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (p == NULL || q == NULL) {
            return (p == q);
        }

        return (p->val == q->val) &&
               isSameTree(p->left, q->left) &&
               isSameTree(p->right, q->right);
    }
};
```

---

## Dry Run (Quick)

```
Tree 1:        Tree 2:
   1              1
  / \            / \
 2   3          2   3
```

- Compare 1 == 1 ✅
- Compare left: 2 == 2 ✅
- Compare right: 3 == 3 ✅

Result → **true**

---

## Time Complexity

```
O(N)
```

- Visit each node once (in worst case, both trees have N nodes)

---

## Space Complexity

```
O(H)
```

- Due to recursion stack
- H = height of tree
- Worst case (skewed): O(N)

---

## Intuition

This is a **structure + value comparison problem**.

Think of it like:

```
Are these two nodes equal?
→ Yes → Check children
→ No  → Return false
```

---

## Important Pattern

This problem follows the pattern:

```
if (base case mismatch) → return false
else → recurse on left & right
```

---

## Common Mistakes

1. Forgetting to handle NULL cases properly
2. Only comparing values, not structure
3. Missing short-circuit evaluation (using && correctly matters)

---

## Variations (Important for Interviews)

- Symmetric Tree (mirror comparison)
- Subtree of Another Tree
- Check if two trees are mirror images

---

## Summary

- Use DFS recursion
- Compare nodes