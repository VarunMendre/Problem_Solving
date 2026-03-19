# Binary Tree Boundary Traversal

---

## 1. Intuition / Approach

The **boundary traversal** of a binary tree prints all boundary nodes in **anti-clockwise order** starting from the root:

```
Root → Left Boundary (top to bottom) → Leaf Nodes (left to right) → Right Boundary (bottom to top)
```

The key insight is to **split the problem into 3 independent parts** and handle them separately:

---

### Part 1 — Left Boundary (Top → Bottom)
- Start from `root->left` and keep going **left** as much as possible.
- If no left child, go **right**.
- **Skip leaf nodes** (they'll be covered by the leaf pass).

### Part 2 — Leaf Nodes (Left → Right)
- Do a simple **recursive DFS** on the entire tree.
- Collect every node that has **no left or right child**.
- This naturally visits them left to right.

### Part 3 — Right Boundary (Bottom → Top)
- Start from `root->right` and keep going **right** as much as possible.
- If no right child, go **left**.
- **Skip leaf nodes**, and collect values in a **temp vector**.
- Reverse and append — because we need bottom-to-top order.

### Why separate the root?
The root is added **once at the start** (if it's not a leaf), so it doesn't get duplicated by the left/right boundary passes.

---

## 2. Code

```cpp
// Node structure for the binary tree
struct Node {
    int data;
    Node* left;
    Node* right;
    Node(int val) : data(val), left(nullptr), right(nullptr) {}
};

class Solution {
public:
    // Helper: check if a node is a leaf
    bool isLeaf(Node* root) {
        return !root->left && !root->right;
    }

    // Add left boundary nodes (top → bottom), excluding leaves
    void addLeftBoundary(Node* root, vector<int>& res) {
        Node* curr = root->left;
        while (curr) {
            if (!isLeaf(curr)) {
                res.push_back(curr->data);
            }
            // Prefer left, fall back to right
            if (curr->left) {
                curr = curr->left;
            } else {
                curr = curr->right;
            }
        }
    }

    // Add right boundary nodes (bottom → top), excluding leaves
    void addRightBoundary(Node* root, vector<int>& res) {
        Node* curr = root->right;
        vector<int> temp;
        while (curr) {
            if (!isLeaf(curr)) {
                temp.push_back(curr->data);
            }
            // Prefer right, fall back to left
            if (curr->right) {
                curr = curr->right;
            } else {
                curr = curr->left;
            }
        }
        // Reverse to get bottom → top order
        for (int i = temp.size() - 1; i >= 0; --i) {
            res.push_back(temp[i]);
        }
    }

    // Add all leaf nodes (left → right) via DFS
    void addLeaves(Node* root, vector<int>& res) {
        if (isLeaf(root)) {
            res.push_back(root->data);
            return;
        }
        if (root->left)  addLeaves(root->left, res);
        if (root->right) addLeaves(root->right, res);
    }

    // Main function: boundary traversal in anti-clockwise order
    vector<int> printBoundary(Node* root) {
        vector<int> res;
        if (!root) return res;

        // Step 1: Add root (only if it's not a leaf)
        if (!isLeaf(root)) {
            res.push_back(root->data);
        }

        // Step 2: Left boundary → Leaves → Right boundary
        addLeftBoundary(root, res);
        addLeaves(root, res);
        addRightBoundary(root, res);

        return res;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each node is visited at most once across all three passes |
| **Space** | `O(N)` | Result vector stores all boundary nodes; recursion stack depth is `O(H)` where H is tree height (`O(N)` worst case for skewed tree) |

> `N` = total number of nodes in the tree  
> `H` = height of the tree (`O(log N)` for balanced, `O(N)` for skewed)
