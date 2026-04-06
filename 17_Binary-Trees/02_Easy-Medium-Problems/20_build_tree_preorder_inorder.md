# Construct Binary Tree from Preorder and Inorder Traversal

---

## 1. Intuition / Approach

### Key Properties of Traversals

| Traversal | Property |
|---|---|
| **Preorder** | First element is always the **root** |
| **Inorder** | Root splits the array — everything **left** of root = left subtree, everything **right** = right subtree |

We exploit both properties together recursively:
1. Pick `preorder[preStart]` as the **current root**
2. Find that root in `inorder` → everything to its left belongs to the **left subtree**, everything to its right belongs to the **right subtree**
3. Calculate `numsLeft` = number of nodes in the left subtree
4. Use `numsLeft` to correctly **slice both arrays** for the next recursive call
5. Recurse on left and right portions

---

### Why the `unordered_map`?

Finding the root in `inorder` naively takes `O(N)` per call → `O(N²)` total.

By pre-storing `inorder[i] → i` in a hashmap, every lookup becomes `O(1)` → total time stays `O(N)`.

---

### The Index Slicing — Most Important Part

Given current root at `preorder[preStart]` and its inorder index `inRoot`:

```
numsLeft = inRoot - inStart   ← count of left subtree nodes
```

| | Preorder range | Inorder range |
|---|---|---|
| **Left subtree** | `[preStart+1 , preStart+numsLeft]` | `[inStart , inRoot-1]` |
| **Right subtree** | `[preStart+numsLeft+1 , preEnd]` | `[inRoot+1 , inEnd]` |

```
preorder: [3, 9, 20, 15, 7]
inorder:  [9, 3, 15, 20, 7]

root = 3 (preorder[0])
inRoot = 1 (position of 3 in inorder)
numsLeft = 1 - 0 = 1

Left subtree:
  preorder: [preStart+1, preStart+1] = [9]
  inorder:  [inStart, inRoot-1]      = [9]

Right subtree:
  preorder: [preStart+2, preEnd]     = [20, 15, 7]
  inorder:  [inRoot+1, inEnd]        = [15, 20, 7]
```

---

### Dry Run

```
preorder = [3, 9, 20, 15, 7]
inorder  = [9, 3, 15, 20, 7]
```

```
buildTree([3,9,20,15,7], [9,3,15,20,7]):
  root = 3, inRoot=1, numsLeft=1
  ├── left:  buildTree([9], [9])
  │     root = 9 → leaf → return node(9)
  └── right: buildTree([20,15,7], [15,20,7])
        root = 20, inRoot=1, numsLeft=1
        ├── left:  buildTree([15], [15])
        │     root = 15 → leaf → return node(15)
        └── right: buildTree([7], [7])
              root = 7 → leaf → return node(7)

Final Tree:
        3
       / \
      9  20
        /  \
       15   7   ✅
```

---

## 2. Code

```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        // Pre-store inorder indices for O(1) lookup
        unordered_map<int, int> inMpp;
        for(int i = 0; i < inorder.size(); i++) {
            inMpp[inorder[i]] = i;
        }

        return buildTree(preorder, 0, preorder.size() - 1,
                         inorder,  0, inorder.size()  - 1, inMpp);
    }

    TreeNode* buildTree(vector<int>& preorder, int preStart, int preEnd,
                        vector<int>& inorder,  int inStart,  int inEnd,
                        unordered_map<int, int>& inMap) {

        // Base case: no elements left to process
        if(preStart > preEnd || inStart > inEnd)
            return NULL;

        // First element of current preorder window = root
        TreeNode* root = new TreeNode(preorder[preStart]);

        // Find root's position in inorder
        int inRoot   = inMap[root->val];

        // Number of nodes in left subtree
        int numsLeft = inRoot - inStart;

        // Recurse on left subtree
        // preorder: [preStart+1 ... preStart+numsLeft]
        // inorder:  [inStart    ... inRoot-1         ]
        root->left = buildTree(preorder, preStart + 1, preStart + numsLeft,
                               inorder,  inStart,      inRoot - 1, inMap);

        // Recurse on right subtree
        // preorder: [preStart+numsLeft+1 ... preEnd]
        // inorder:  [inRoot+1            ... inEnd ]
        root->right = buildTree(preorder, preStart + numsLeft + 1, preEnd,
                                inorder,  inRoot + 1,              inEnd, inMap);

        return root;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity — `O(N)`

| Work | Cost |
|---|---|
| Building `inMpp` (one pass over inorder) | `O(N)` |
| Recursive calls — each node is processed exactly once | `O(N)` |
| Each `inMap` lookup inside recursion | `O(1)` average (unordered_map) |

**Total: `O(N) + O(N)` = `O(N)`**

> Without the hashmap, finding `inRoot` would cost `O(N)` per call → `O(N²)` total. The map is what keeps it linear.

---

### Space Complexity — `O(N)`

| Structure | Size | Reason |
|---|---|---|
| `inMpp` hashmap | `O(N)` | Stores one entry per inorder element |
| Recursive call stack | `O(N)` | One frame per node in the worst case (skewed tree); `O(log N)` for balanced |
| Output tree | `O(N)` | N nodes created |

**Total: `O(N)`**

> The auxiliary stack space is `O(H)` where H = height — `O(log N)` for balanced, `O(N)` for skewed.
