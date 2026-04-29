# Lowest Common Ancestor of a BST

---

## 1. Brute Force — Generic LCA (Ignores BST Property)

### Approach

This is the **exact same solution as LCA of a Binary Tree** — it works on any binary tree, not just a BST. It does not use the BST property at all.

Postorder DFS — search left and right subtrees, then decide at the current node:

| `left` | `right` | Return |
|--------|---------|--------|
| `NULL` | node | `right` |
| node | `NULL` | `left` |
| node | node | `root` (current node is LCA) |

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // Base case: reached null, or found p or q
        if(root == NULL || p == root || q == root)
            return root;

        // Search both subtrees
        TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);

        if(left == NULL)       return right;   // both in right subtree
        else if(right == NULL) return left;    // both in left subtree
        else                   return root;    // split across both sides → LCA
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited in worst case |
| **Space** | `O(H)` | Recursive call stack; `O(N)` worst case for skewed tree |

> Works correctly but visits the entire tree without using the sorted structure of a BST.

---

---

## 2. Optimal — Exploit BST Property

### Approach

In a BST, every node satisfies:
- Left subtree values < current node
- Right subtree values > current node

This lets us **navigate directly** to the LCA without exploring the whole tree.

### Key Insight

The LCA is the **first node where `p` and `q` stop being on the same side**:

| Condition | Meaning | Action |
|---|---|---|
| `curr < p` and `curr < q` | Both nodes are in the **right** subtree | Go right |
| `curr > p` and `curr > q` | Both nodes are in the **left** subtree | Go left |
| Neither | `p` and `q` are on **opposite sides** (or one equals `curr`) | Current node is **LCA** |

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root == NULL) return NULL;

        int curr = root->val;

        // Both nodes greater → LCA must be in right subtree
        if(curr < p->val && curr < q->val)
            return lowestCommonAncestor(root->right, p, q);

        // Both nodes smaller → LCA must be in left subtree
        if(curr > p->val && curr > q->val)
            return lowestCommonAncestor(root->left, p, q);

        // Split point — current node is the LCA
        return root;
    }
};
```

### Dry Run

```
        6
       / \
      2   8
     / \ / \
    0  4 7  9
      / \
     3   5

p = 2, q = 4
```

```
curr = 6: 6 > 2 and 6 > 4 → go left
curr = 2: neither (2 == p) → return node(2)

LCA = 2 ✅

p = 2, q = 8
curr = 6: 2 < 6 < 8 → split! → return node(6)

LCA = 6 ✅
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(H)` | Only travels one path root → LCA; `O(log N)` balanced, `O(N)` skewed |
| **Space** | `O(H)` | Recursive call stack depth = path length to LCA |

> **This is a genuine improvement over brute force** — instead of `O(N)` always, we now only traverse the height of the tree. For a balanced BST, that's `O(log N)`.

---

---

## 3. Brute vs Optimal

| | Brute Force | Optimal |
|---|---|---|
| **Uses BST property?** | ❌ No | ✅ Yes |
| **Time** | `O(N)` always | `O(H)` — `O(log N)` balanced |
| **Space** | `O(H)` | `O(H)` |
| **Nodes visited** | All N nodes | Only nodes on root → LCA path |
