# Maximum Depth of Binary Tree

## 1. Approach / Intuition

The **maximum depth (or height) of a binary tree** is the number of nodes along the longest path from the root node down to the farthest leaf node.

Example:

```
    1
   / \
  2   3
 / \
4   5
```

Depth = 3

Path example:

1 → 2 → 4

or

1 → 2 → 5

Both paths contain **3 nodes**, so the maximum depth is **3**.

---

### Key Idea

A binary tree naturally follows a **recursive structure**.

For every node we ask:

"What is the maximum height of my left subtree and my right subtree?"

The height of the current node becomes:

height = 1 + max(leftHeight, rightHeight)

Where:

• `1` represents the current node
• `leftHeight` is the height of the left subtree
• `rightHeight` is the height of the right subtree

So the recursion works **bottom-up**.

Leaf nodes return height = 1.

---

### Base Condition

If the node is `NULL`, it means we reached beyond a leaf node.

So:

height = 0

---

### Recursive Process

For each node:

1. Recursively calculate the height of the left subtree.
2. Recursively calculate the height of the right subtree.
3. Return `1 + max(leftHeight, rightHeight)`.

This guarantees we always take the **longest path from root to leaf**.

---

## 2. Code

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {

        if(root == NULL) return 0;

        int leftHeight = maxDepth(root->left);
        int rightHeight = maxDepth(root->right);

        return 1 + max(leftHeight, rightHeight);
    }
};
```

---

## 3. Code Explanation

Step 1

Check if the node is NULL.

```
if(root == NULL) return 0;
```

If the node does not exist, its height is **0**.

---

Step 2

Recursively calculate the height of the left subtree.

```
int leftHeight = maxDepth(root->left);
```

---

Step 3

Recursively calculate the height of the right subtree.

```
int rightHeight = maxDepth(root->right);
```

---

Step 4

Return the height of the current node.

```
return 1 + max(leftHeight, rightHeight);
```

The `1` represents the current node itself.

We take the maximum because we want the **longest path**.

---

## Example Dry Run

Tree:

```
    1
   / \
  2   3
 /
4
```

Calculation:

Node 4 → height = 1

Node 2 →

1 + max(1,0) = 2

Node 3 → height = 1

Node 1 →

1 + max(2,1) = 3

Final Answer = **3**

---

## 4. Time & Space Complexity

### Time Complexity: O(N)

Where **N** is the number of nodes in the tree.

Reason:

Each node is visited **exactly once** during recursion.

Therefore:

Time Complexity = **O(N)**

---

### Space Complexity: O(H)

Where **H** is the height of the tree.

This space is used by the **recursion call stack**.

Worst Case (Skewed Tree):

1

2

3

4

Height = N

So space complexity becomes:

O(N)

Best Case (Balanced Tree):

Height = log N

So space complexity becomes:

O(log N)

---

## 5. Key Interview Insight

This problem teaches an important tree concept:

"Solve the problem for children first, then use their results to compute the parent."

This pattern appears in many tree problems:

• Diameter of Binary Tree
• Balanced Binary Tree
• Maximum Path Sum
• Lowest Common Ancestor

So understanding this recursive pattern is **extremely important for tree problems**.
