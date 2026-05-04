# Construct BST from Preorder Traversal

---

## 1. Brute Force — Insert Node by Node

### Approach

For each value in `preorder`, **traverse the BST from root** to find the correct insertion position, then insert it.

- First element → root
- Each subsequent element → walk down the tree using BST rules (go left if smaller, right if larger) until you find a NULL spot

```
preorder = [8, 5, 1, 7, 10, 12]

Insert 8  → root
Insert 5  → left of 8  (5 < 8)
Insert 1  → left of 5  (1 < 8, 1 < 5)
Insert 7  → right of 5 (7 < 8, 7 > 5)
Insert 10 → right of 8 (10 > 8)
Insert 12 → right of 10 (12 > 8, 12 > 10)
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N²)` | For each of N nodes, finding the insert position costs `O(N)` in worst case (skewed tree) |
| **Space** | `O(1)` | No extra data structures (excluding output tree) |

---

---

## 2. Better — Sort Preorder → Inorder + Build Tree

### Approach

BST's **inorder traversal is always sorted**. So if we sort the given preorder array, we get the inorder array. Then use the standard **preorder + inorder → construct tree** algorithm.

**Step 1:** Sort `preorder` → this gives `inorder`  
**Step 2:** Use both arrays to reconstruct the tree

```cpp
class Solution {
public:
    TreeNode* bstFromPreorder(vector<int>& preorder) {
        // Sort preorder to get inorder (BST property: inorder = sorted)
        vector<int> inorder = preorder;
        sort(inorder.begin(), inorder.end());   // O(N log N)

        // Build tree using preorder + inorder
        unordered_map<int, int> inMpp;
        for(int i = 0; i < inorder.size(); i++)
            inMpp[inorder[i]] = i;

        return buildTree(preorder, 0, preorder.size() - 1,
                         inorder,  0, inorder.size()  - 1, inMpp);
    }

    TreeNode* buildTree(vector<int>& preorder, int preStart, int preEnd,
                        vector<int>& inorder,  int inStart,  int inEnd,
                        unordered_map<int, int>& inMap) {
        if(preStart > preEnd || inStart > inEnd)
            return NULL;

        TreeNode* root = new TreeNode(preorder[preStart]);
        int inRoot   = inMap[root->val];
        int numsLeft = inRoot - inStart;

        // Left subtree uses left portion of both arrays
        root->left = buildTree(preorder, preStart + 1,            preStart + numsLeft,
                               inorder,  inStart,                 inRoot - 1, inMap);

        // Right subtree uses right portion of both arrays
        root->right = buildTree(preorder, preStart + numsLeft + 1, preEnd,
                                inorder,  inRoot + 1,              inEnd, inMap);
        return root;
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Sorting preorder costs `O(N log N)`; building tree costs `O(N)` |
| **Space** | `O(N)` | HashMap + inorder vector + recursive call stack |

---

---

## 3. Optimal — Upper Bound Trick

### Approach

Use BST's preorder structure directly — **no sorting, no hashmap**.

In preorder, the root comes first, then the left subtree (all values < root), then the right subtree (all values > root). We can use this with an **upper bound** to decide when to stop building the current subtree.

**Key Idea:**

Pass an `upperBound` to each recursive call. A node from preorder belongs to the current subtree **only if its value is less than `upperBound`**. The moment `preorder[i] > upperBound`, we've left the current subtree → return NULL.

- Building **left subtree** → `upperBound = root->val` (left must be < root)
- Building **right subtree** → `upperBound = parent's upperBound` (right is still bounded by ancestor)
- `i` is passed by **reference** so all recursive calls share and advance the same index

```cpp
class Solution {
public:
    TreeNode* bstFromPreorder(vector<int>& preorder) {
        int i = 0;
        return buildTree(preorder, i, INT_MAX);
    }

    TreeNode* buildTree(vector<int>& preorder, int& i, int upperBound) {
        // Base case: exhausted array OR current value exceeds allowed range
        if(i == preorder.size() || preorder[i] > upperBound)
            return NULL;

        // Current value is within range — create node and advance index
        TreeNode* root = new TreeNode(preorder[i++]);

        // Left subtree: values must be < root->val
        root->left  = buildTree(preorder, i, root->val);

        // Right subtree: values must be < parent's upperBound (passed down)
        root->right = buildTree(preorder, i, upperBound);

        return root;
    }
};
```

### Dry Run

```
preorder = [8, 5, 1, 7, 10, 12]
```

```
buildTree(i=0, upper=INT_MAX)
  root = 8, i=1
  ├── left: buildTree(i=1, upper=8)
  │     root = 5, i=2
  │     ├── left: buildTree(i=2, upper=5)
  │     │     root = 1, i=3
  │     │     ├── left:  buildTree(i=3, upper=1) → 7>1 → NULL
  │     │     └── right: buildTree(i=3, upper=5) → 7>5 → NULL
  │     │     return node(1)
  │     └── right: buildTree(i=3, upper=8)
  │           root = 7, i=4
  │           ├── left:  buildTree(i=4, upper=7) → 10>7 → NULL
  │           └── right: buildTree(i=4, upper=8) → 10>8 → NULL
  │           return node(7)
  │     return node(5)
  └── right: buildTree(i=4, upper=INT_MAX)
        root = 10, i=5
        ├── left:  buildTree(i=5, upper=10) → 12>10 → NULL
        └── right: buildTree(i=5, upper=INT_MAX)
              root = 12, i=6
              ├── left:  i==size → NULL
              └── right: i==size → NULL
              return node(12)
        return node(10)

Final Tree:
        8
       / \
      5   10
     / \    \
    1   7   12   ✅
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node processed exactly once — `i` only moves forward, never back |
| **Space** | `O(H)` auxiliary | Recursive call stack depth = height of tree; `O(log N)` balanced, `O(N)` skewed |

> **Why `O(3N)` simplifies to `O(N)`:** In the worst case, a node's frame is on the stack when we recurse left (1st visit), returns from left (2nd), recurses right (3rd). But `i` advances exactly once per node — so total work is still linear in N.

---

---

## 4. All Approaches at a Glance

| Approach | TC | SC | Key Idea |
|---|---|---|---|
| Brute — Insert node by node | `O(N²)` | `O(1)` | Insert each value into BST one at a time |
| Better — Sort + Build | `O(N log N)` | `O(N)` | Sort preorder → inorder, then use classic build algorithm |
| **Optimal — Upper Bound** | `O(N)` | `O(H)` | Use BST order + upper bound to place nodes in one pass |
