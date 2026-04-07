# Construct Binary Tree from Inorder and Postorder Traversal

---

## 1. Intuition / Approach

### Key Properties of Traversals

| Traversal | Property |
|---|---|
| **Postorder** | **Last element** is always the root |
| **Inorder** | Root splits the array — everything **left** = left subtree, everything **right** = right subtree |

This is the **mirror problem** of preorder + inorder construction. The only difference:
- Preorder → root is at the **front**
- Postorder → root is at the **back**

---

### Step-by-Step

1. Pick `postorder[postEnd]` as the **current root**
2. Find that root in `inorder` → gives `inRoot`
3. Calculate `numsLeft = inRoot - inStart` (nodes in left subtree)
4. Slice both arrays correctly for left and right subtrees
5. Recurse

---

### The Index Slicing

Given root at `postorder[postEnd]` and its inorder position `inRoot`:

```
numsLeft = inRoot - inStart
```

| | Inorder range | Postorder range |
|---|---|---|
| **Left subtree** | `[inStart , inRoot-1]` | `[postStart , postStart + numsLeft - 1]` |
| **Right subtree** | `[inRoot+1 , inEnd]` | `[postStart + numsLeft , postEnd - 1]` |

> Note: Right subtree's postorder ends at `postEnd - 1` because `postEnd` is the current root, already consumed.

---

### Preorder vs Postorder — Side by Side

| | Preorder + Inorder | Postorder + Inorder |
|---|---|---|
| Root location | `preorder[preStart]` (front) | `postorder[postEnd]` (back) |
| Left preorder/postorder start | `preStart + 1` | `postStart` |
| Left preorder/postorder end | `preStart + numsLeft` | `postStart + numsLeft - 1` |
| Right preorder/postorder start | `preStart + numsLeft + 1` | `postStart + numsLeft` |
| Right preorder/postorder end | `preEnd` | `postEnd - 1` |

---

### Dry Run

```
inorder   = [9, 3, 15, 20, 7]
postorder = [9, 15, 7, 20, 3]
```

```
buildTree(in=[9,3,15,20,7], post=[9,15,7,20,3]):
  root = 3 (postorder[4])
  inRoot = 1, numsLeft = 1

  Left:
    inorder:   [inStart=0, inRoot-1=0]  → [9]
    postorder: [postStart=0, 0+1-1=0]   → [9]
    root = 9 → leaf ✅

  Right:
    inorder:   [inRoot+1=2, inEnd=4]    → [15, 20, 7]
    postorder: [postStart+numsLeft=1, postEnd-1=3] → [15, 7, 20]
    root = 20 (postorder[3])
    inRoot = 3, numsLeft = 1

    Left:
      inorder:   [2, 2] → [15]
      postorder: [1, 1] → [15]
      root = 15 → leaf ✅

    Right:
      inorder:   [4, 4] → [7]
      postorder: [2, 2] → [7]
      root = 7 → leaf ✅

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
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        // Pre-store inorder indices for O(1) lookup
        unordered_map<int, int> inorderHash;
        for(int i = 0; i < inorder.size(); i++) {
            inorderHash[inorder[i]] = i;
        }

        return buildTree(inorder, 0, inorder.size() - 1,
                         postorder, 0, postorder.size() - 1, inorderHash);
    }

    TreeNode* buildTree(vector<int>& inorder, int inStart, int inEnd,
                        vector<int>& postorder, int postStart, int postEnd,
                        unordered_map<int, int>& inorderHash) {

        // Base case: no elements left to process
        if(inStart > inEnd || postStart > postEnd)
            return NULL;

        // Last element of current postorder window = root
        TreeNode* root = new TreeNode(postorder[postEnd]);

        // Find root's position in inorder
        int inRoot = inorderHash[root->val];

        // Inorder window boundaries for left subtree
        int inLLeftPoint  = inStart;
        int inLRightPoint = inRoot - 1;

        // Postorder window boundaries for left subtree
        int postLLeftPoint  = postStart;
        int postLRightPoint = postStart + (inRoot - inStart) - 1;  // postStart + numsLeft - 1

        // Recurse left subtree
        root->left = buildTree(inorder,   inLLeftPoint,         inLRightPoint,
                               postorder, postLLeftPoint,        postLRightPoint, inorderHash);

        // Recurse right subtree
        // postorder right ends at postEnd-1 (postEnd is current root, already used)
        root->right = buildTree(inorder,   inRoot + 1,           inEnd,
                                postorder, postLRightPoint + 1,  postEnd - 1, inorderHash);

        return root;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity — `O(N)`

| Work | Cost |
|---|---|
| Building `inorderHash` (one pass over inorder) | `O(N)` |
| Recursive calls — each node processed exactly once | `O(N)` |
| Each `inorderHash` lookup inside recursion | `O(1)` average |

**Total: `O(N) + O(N)` = `O(N)`**

---

### Space Complexity — `O(N)`

| Structure | Size | Reason |
|---|---|---|
| `inorderHash` hashmap | `O(N)` | One entry per inorder element |
| Recursive call stack | `O(H)` | `O(log N)` balanced, `O(N)` skewed |
| Output tree | `O(N)` | N nodes created |

**Total: `O(N)`**

> `H` = height of tree → `O(log N)` for balanced, `O(N)` worst case for skewed
