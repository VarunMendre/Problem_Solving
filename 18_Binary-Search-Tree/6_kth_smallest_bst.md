# Kth Smallest Element in a BST

### Key Property Used
**Inorder traversal of a BST gives nodes in sorted (ascending) order.**  
So the kth smallest = the kth node visited in inorder traversal.

---

## 1. Brute Force — DFS + Sort

### Approach
1. Do any DFS traversal and collect all node values into a vector
2. Sort the vector
3. Return `vector[k-1]`

```cpp
class Solution {
public:
    void dfs(TreeNode* root, vector<int>& vals) {
        if(root == NULL) return;
        vals.push_back(root->val);  // collect all values
        dfs(root->left,  vals);
        dfs(root->right, vals);
    }

    int kthSmallest(TreeNode* root, int k) {
        vector<int> vals;
        dfs(root, vals);            // O(N) — visit every node
        sort(vals.begin(), vals.end()); // O(N log N) — sort all values
        return vals[k - 1];         // kth smallest (0-indexed)
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N) + O(N log N)` = `O(N log N)` | DFS visits all N nodes + sorting costs `O(N log N)` |
| **Space** | `O(N)` | Vector stores all N values + `O(H)` call stack |

> Doesn't use the BST property at all — treats it like any random tree.

---

---

## 2. Better — Iterative Inorder with Stack

### Approach

Use an explicit **stack** to simulate inorder traversal (left → root → right) iteratively. Since inorder of BST is sorted, the **kth node popped** is the kth smallest.

Stop early if needed — no need to traverse the whole tree once we find the kth element (though this implementation collects all into a vector first).

```cpp
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        if(root == nullptr) return NULL;

        vector<int> inOrder;
        TreeNode* node = root;
        stack<TreeNode*> st;

        while(true) {
            if(node != nullptr) {
                // Keep going left, pushing nodes onto stack
                st.push(node);
                node = node->left;
            } else {
                if(st.empty()) break;

                // Pop = visit node (inorder position)
                node = st.top();
                st.pop();
                inOrder.push_back(node->val);

                // Move to right subtree
                node = node->right;
            }
        }

        return inOrder[k - 1];  // kth element in sorted order
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node pushed and popped from stack exactly once |
| **Space** | `O(N)` | Stack holds up to `O(H)` nodes; vector stores up to N values |

> Exploits BST property (inorder = sorted) — no sorting needed. But still uses `O(N)` space for stack and vector.

---

---

## 3. Optimal — Morris Inorder Traversal

### Approach

**Morris traversal** does inorder traversal with **`O(1)` extra space** — no stack, no recursion — by temporarily threading nodes.

Maintain a `count` and `result`. Every time a node is visited in inorder order (the "root" visit moment), increment `count`. When `count == k`, record the result.

### How Morris Inorder Works

At each `cur` node:

- **No left child** → visit `cur` (inorder moment), move right
- **Has left child** → find inorder predecessor (rightmost node of left subtree):
  - If predecessor's right is `NULL` → thread it (`prev->right = cur`), go left
  - If predecessor's right is `cur` → unthread (`prev->right = NULL`), **visit `cur`** (inorder moment), go right

The "visit" moment = when we unthread (second time we see the node) or when there's no left child.

```cpp
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        TreeNode* cur = root;
        int count  = 0;
        int result = 0;

        while(cur != NULL) {
            if(cur->left == NULL) {
                // No left child — this IS the inorder visit moment
                if(++count == k) result = cur->val;
                cur = cur->right;
            } else {
                // Find inorder predecessor (rightmost of left subtree)
                TreeNode* prev = cur->left;
                while(prev->right && prev->right != cur)
                    prev = prev->right;

                if(prev->right == NULL) {
                    // First visit: thread predecessor back to cur, go left
                    prev->right = cur;
                    cur = cur->left;
                } else {
                    // Second visit: unthread, this IS the inorder visit moment
                    prev->right = NULL;
                    if(++count == k) result = cur->val;
                    cur = cur->right;
                }
            }
        }

        return result;
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node visited at most twice (once to thread, once to unthread) |
| **Space** | `O(1)` | No stack, no recursion, no extra data structures |

---

---

## 4. All Approaches at a Glance

| Approach | TC | SC | Key Idea |
|---|---|---|---|
| Brute — DFS + Sort | `O(N log N)` | `O(N)` | Collect all values, sort, index |
| Better — Iterative Inorder | `O(N)` | `O(N)` | Inorder with explicit stack — kth pop is answer |
| **Optimal — Morris Traversal** | `O(N)` | **`O(1)`** | Thread nodes to do inorder with zero extra space |
