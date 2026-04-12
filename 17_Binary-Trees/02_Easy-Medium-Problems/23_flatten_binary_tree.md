# Flatten Binary Tree to Linked List

The goal is to flatten a binary tree **in-place** into a linked list using the `right` pointer, following **preorder traversal** order. All `left` pointers must be set to `NULL`.

---

## 1. Brute Force — Reverse Postorder DFS

### Approach

Process nodes in **reverse preorder** (right → left → root). Use a global `prev` pointer that always points to the previously processed node.

At each node:
- Set `root->right = prev` (link to the next node in preorder)
- Set `root->left = NULL`
- Update `prev = root`

Since we process right subtree first, then left, then root — by the time we process any node, its entire right chain is already correctly linked.

```cpp
class Solution {
public:
    TreeNode* prev = NULL;

    void flatten(TreeNode* root) {
        if(root == NULL) return;

        flatten(root->right);   // process right subtree first
        flatten(root->left);    // then left subtree

        root->right = prev;     // link this node to the already-flattened rest
        root->left  = NULL;
        prev = root;            // update prev for the next (parent) node
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited exactly once |
| **Space** | `O(N)` | Recursive call stack — `O(H)`, up to `O(N)` for skewed tree |

---

---

## 2. Better — Iterative with Stack

### Approach

Simulate preorder traversal using an explicit **stack**. At each step:
- Pop a node
- Push right child first, then left child (so left is processed first — LIFO)
- Set `node->right = stack.top()` (the next node to be processed in preorder)
- Set `node->left = NULL`

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        if(root == NULL) return;

        stack<TreeNode*> st;
        st.push(root);

        while(!st.empty()) {
            TreeNode* node = st.top();
            st.pop();

            if(node->right) st.push(node->right);  // push right first
            if(node->left)  st.push(node->left);   // push left second (processed first)

            // next node in preorder is now at top of stack
            if(!st.empty())
                node->right = st.top();

            node->left = NULL;
        }
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited exactly once |
| **Space** | `O(N)` | Stack holds up to `O(H)` nodes; worst case `O(N)` for skewed tree |

---

---

## 3. Better (2) — Morris Preorder Traversal + Vector

### Approach

Collect all nodes in **preorder using Morris traversal** (no recursion, no stack — `O(1)` space during collection) into a `vector`, then link them.

Morris traversal works by temporarily threading nodes:
- If `curr->left` is NULL → visit node, move right
- Else → find the **inorder predecessor** (rightmost node of left subtree):
  - If its right is NULL → thread it to `curr`, visit `curr`, go left
  - If its right is `curr` → unthread it, go right

After collection, link each `nodes[i]->right = nodes[i+1]`, `left = NULL`.

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        if(root == NULL) return;

        vector<TreeNode*> nodes;
        collectPreorder(root, nodes);

        // Link each node to the next
        for(int i = 0; i < (int)nodes.size() - 1; i++) {
            nodes[i]->left  = NULL;
            nodes[i]->right = nodes[i + 1];
        }

        // Clean up the last node
        nodes.back()->left  = NULL;
        nodes.back()->right = NULL;
    }

private:
    void collectPreorder(TreeNode* root, vector<TreeNode*>& nodes) {
        TreeNode* curr = root;

        while(curr != NULL) {
            if(curr->left == NULL) {
                // No left child — visit and move right
                nodes.push_back(curr);
                curr = curr->right;
            } else {
                // Find inorder predecessor (rightmost of left subtree)
                TreeNode* prev = curr->left;
                while(prev->right && prev->right != curr)
                    prev = prev->right;

                if(prev->right == NULL) {
                    // Thread: link predecessor back to curr, visit curr, go left
                    prev->right = curr;
                    nodes.push_back(curr);
                    curr = curr->left;
                } else {
                    // Unthread: restore and move right
                    prev->right = NULL;
                    curr = curr->right;
                }
            }
        }
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node visited at most twice in Morris traversal |
| **Space** | `O(N)` | Vector stores all N node pointers |

> Morris traversal itself is `O(1)` auxiliary space — but the `vector` brings total SC back to `O(N)`. The optimal solution below removes even that.

---

---

## 4. Optimal — Morris-style In-Place (No Extra Space)

### Approach

Same idea as Morris traversal but done **directly in-place** — no vector, no stack, no recursion.

At each node with a left child:
1. Find the **rightmost node of the left subtree** (inorder predecessor)
2. Thread it → `prev->right = curr->right` (attach current right subtree at the end of left subtree)
3. Move left subtree to the right → `curr->right = curr->left`
4. Clear left → `curr->left = NULL`
5. Move to `curr->right`

This effectively **inserts the left subtree between the current node and its right subtree** in one pass.

---

### Visual

```
Before:
    1
   / \
  2   5
 / \
3   4

Step at node 1:
  prev = rightmost of left subtree = node 4
  prev->right = 5      →  4's right now points to 5
  curr->right = 2      →  1's right now points to 2
  curr->left  = NULL

After step:
    1
     \
      2
     / \
    3   4
         \
          5

Next iteration processes node 2, then 3, and so on.

Final:
1 → 2 → 3 → 4 → 5  ✅
```

---

### Code

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        if(!root) return;

        TreeNode* curr = root;

        while(curr) {
            if(curr->left) {
                // Find rightmost node of left subtree
                TreeNode* prev = curr->left;
                while(prev->right)
                    prev = prev->right;

                // Thread: attach current right subtree after left subtree
                prev->right = curr->right;

                // Move left subtree to the right
                curr->right = curr->left;

                // Clear left pointer
                curr->left = NULL;
            }

            // Move to next node in the flattened list
            curr = curr->right;
        }
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node visited at most twice (once as `curr`, once as `prev`) |
| **Space** | `O(1)` | No recursion, no stack, no extra data structures — pure in-place |

---

---

## 5. All Approaches at a Glance

| Approach | TC | SC | Key Idea |
|---|---|---|---|
| Brute — Reverse Postorder DFS | `O(N)` | `O(N)` | Process right→left→root using `prev` pointer |
| Better — Iterative Stack | `O(N)` | `O(N)` | Simulate preorder with explicit stack |
| Better (2) — Morris + Vector | `O(N)` | `O(N)` | Collect preorder via Morris, then link |
| **Optimal — Morris In-Place** | `O(N)` | **`O(1)`** | Thread left subtree directly into right in-place |
