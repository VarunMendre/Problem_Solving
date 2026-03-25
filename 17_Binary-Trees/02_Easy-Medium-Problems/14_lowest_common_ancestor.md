# Lowest Common Ancestor of a Binary Tree

---

## 1. Brute Force

### Approach

- Find the path from `root → p` and store it in a vector (e.g. `pathP`)
- Find the path from `root → q` and store it in a vector (e.g. `pathQ`)
- Traverse **both vectors simultaneously** and compare elements
- The last matching element before they diverge is the **LCA**

```
        3
       / \
      5   1
     / \
    6   2

p = 6, q = 2

pathP = [3, 5, 6]
pathQ = [3, 5, 2]

Compare:
  index 0 → 3 == 3 ✅ → last common = 3
  index 1 → 5 == 5 ✅ → last common = 5
  index 2 → 6 != 2 ❌ → stop

LCA = 5 ✅
```

---

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Finding each path takes `O(N)` in worst case; comparing paths takes `O(H)` — overall `O(N)` |
| **Space** | `O(H)` for each path | Path length = height of tree = `O(H)`; two paths = `O(H)` total, not `O(N)` |

> **Correction on your SC:** You said `O(N)` for storing paths — but a path from root to a node has length equal to the **height H**, not N. So it's `O(H)` per path → `O(H)` total space. Only in a skewed tree does `H = N`, making it `O(N)` in the worst case.

---

---

## 2. Optimal

### Approach

We use a single **postorder DFS** — check left and right subtrees first, then decide at the current node.

### Base Cases

```cpp
if(root == NULL || p == root || q == root)
    return root;
```

- `root == NULL` → nothing found here, return `NULL`
- `p == root` or `q == root` → found one of the targets, return it immediately (no need to go deeper — even if the other node is in its subtree, this node is still the LCA)

### Core Logic

After recursing left and right:

| `left` | `right` | Meaning | Return |
|--------|---------|---------|--------|
| `NULL` | node | both p & q found in right subtree | `right` |
| node | `NULL` | both p & q found in left subtree | `left` |
| node | node | p found on one side, q on the other | `root` (current node is LCA) |

### Dry Run

```
        3
       / \
      5   1
     / \
    6   2

p = 6, q = 1
```

```
LCA(3, 6, 1)
  ├── LCA(5, 6, 1)
  │     ├── LCA(6, 6, 1) → returns 6  (base case: p == root)
  │     └── LCA(2, 6, 1) → returns NULL
  │     left=6, right=NULL → return 6
  └── LCA(1, 6, 1) → returns 1  (base case: q == root)
  left=6, right=1 → both non-null → return root(3)

LCA = 3 ✅
```

---

### Code

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // Base case: reached null, or found p or q
        if(root == NULL || p == root || q == root) {
            return root;
        }

        // Search in left and right subtrees
        TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);

        // p and q both in right subtree
        if(left == NULL) {
            return right;
        }
        // p and q both in left subtree
        else if(right == NULL) {
            return left;
        }
        // p and q on opposite sides → current node is LCA
        else {
            return root;
        }
    }
};
```

---

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space** | `O(H)` auxiliary | Recursive call stack depth = height of tree; `O(N)` worst case for a skewed tree |

> `N` = total number of nodes | `H` = height of the tree
