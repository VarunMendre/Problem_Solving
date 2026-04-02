# Count Nodes in a Complete Binary Tree

---

## 1. Brute Force

### Approach

Simple **preorder DFS** — visit every node and increment a counter.

```cpp
class Solution {
public:
    void preOrder(TreeNode* node, int& count) {
        if(node == NULL) return;

        count++;                        // count this node
        preOrder(node->left, count);    // recurse left
        preOrder(node->right, count);   // recurse right
    }

    int countNodes(TreeNode* root) {
        int count = 0;
        preOrder(root, count);
        return count;
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited exactly once |
| **Space** | `O(H)` auxiliary | Recursive call stack = height of tree; `O(N)` worst case for skewed tree |

> This approach doesn't use the **complete binary tree** property at all — it treats any tree the same way. That's why there's room for a better solution.

---

---

## 2. Optimal

### Approach — Exploit the Complete Binary Tree Property

A **complete binary tree** has all levels fully filled except possibly the last, which is filled from left to right.

### Key Observation

For any subtree rooted at a node, check:
- `leftHeight` = height going **all the way left**
- `rightHeight` = height going **all the way right**

**If `leftHeight == rightHeight`:**
The subtree is a **perfect binary tree** (all levels fully filled).
Node count = `2^height - 1` → computed in `O(1)` using bit shift: `(1 << height) - 1`

**If `leftHeight != rightHeight`:**
The subtree is **not perfect** — recurse on left and right subtrees separately and add 1 for the current node.

```
return 1 + countNodes(root->left) + countNodes(root->right);
```

---

### Why does `leftHeight == rightHeight` mean it's a perfect binary tree?

In a **complete binary tree**:
- The leftmost path is always the longest (last level fills left to right)
- If left and right heights are equal, the last level is completely filled → **perfect binary tree**
- If they differ, the last level is partially filled → need to recurse deeper

---

### Dry Run

```
        1
       / \
      2   3
     / \ /
    4  5 6
```

```
countNodes(root=1):
  leftH=3 (1→2→4), rightH=2 (1→3) → not equal → recurse

  countNodes(left=2):
    leftH=2 (2→4), rightH=2 (2→5) → equal! → return (1<<2)-1 = 3

  countNodes(right=3):
    leftH=2 (3→6), rightH=1 (3) → not equal → recurse
    countNodes(left=6): leftH=1, rightH=1 → return (1<<1)-1 = 1
    countNodes(right=NULL): return 0
    → return 1 + 1 + 0 = 2

→ return 1 + 3 + 2 = 6 ✅
```

---

## 2. Code

```cpp
class Solution {
public:
    int countNodes(TreeNode* root) {
        if(root == NULL) return 0;

        int leftHeight  = countLeftHeight(root);
        int rightHeight = countRightHeight(root);

        // Perfect binary tree — use formula directly
        if(leftHeight == rightHeight)
            return (1 << leftHeight) - 1;   // 2^height - 1

        // Not perfect — recurse on both subtrees
        return 1 + countNodes(root->left) + countNodes(root->right);
    }

    // Count height going always left
    int countLeftHeight(TreeNode* root) {
        int height = 0;
        while(root) {
            height++;
            root = root->left;
        }
        return height;
    }

    // Count height going always right
    int countRightHeight(TreeNode* root) {
        int height = 0;
        while(root) {
            height++;
            root = root->right;
        }
        return height;
    }
};
```

---

## 3. Time & Space Complexity (In-Depth)

### Time Complexity — `O(log²N)`

At each recursive call, we do two height calculations:
- `countLeftHeight` → `O(log N)`
- `countRightHeight` → `O(log N)`

**How many recursive calls are made?**

At each node, if heights are unequal, we recurse on both children. But crucially — at least **one subtree will always be a perfect binary tree** in a complete binary tree. That subtree returns immediately in `O(1)` via the formula.

So only **one branch truly recurses deeper** at each level → `O(log N)` recursive calls total.

Each call does `O(log N)` work (height calculations):

**Total: `O(log N × log N)` = `O(log²N)`**

---

### Space Complexity — `O(log N)`

| Structure | Size | Reason |
|---|---|---|
| Recursive call stack | `O(log N)` | Complete binary tree height is always `O(log N)` — guaranteed |

> Unlike brute force where SC could reach `O(N)` for a skewed tree, here the tree is **guaranteed complete** — height is always `O(log N)`, never worse.

---

## 4. Brute vs Optimal

| | Brute Force | Optimal |
|---|---|---|
| **Uses complete tree property?** | ❌ No | ✅ Yes |
| **Time** | `O(N)` | `O(log²N)` |
| **Space** | `O(H)` up to `O(N)` | `O(log N)` guaranteed |
| **Key idea** | Count every node | Skip perfect subtrees using `2^h - 1` formula |
