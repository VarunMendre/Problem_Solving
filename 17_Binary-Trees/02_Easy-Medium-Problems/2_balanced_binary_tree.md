# Balanced Binary Tree

A Binary Tree is considered **balanced** if for every node in the tree, the difference between the height of the left subtree and the height of the right subtree is **not more than 1**.

Mathematically:

| height(left) - height(right) | ≤ 1

If this condition holds for **every node**, the tree is balanced.

Example of Balanced Tree:

        1
       / \
      2   3
     / \
    4   5

Example of Unbalanced Tree:

        1
       /
      2
     /
    3
   /
  4

The height difference becomes greater than 1.

---

# 1. Brute Force Approach

## Intuition

To determine whether a tree is balanced, we need the **height of the left subtree and the height of the right subtree for every node**.

Steps:

1. For each node, calculate the height of its left subtree.
2. Calculate the height of its right subtree.
3. Check if the difference between the two heights is ≤ 1.
4. Recursively check the same condition for the left subtree.
5. Recursively check the same condition for the right subtree.

If all nodes satisfy the condition, the tree is balanced.

However, this method repeatedly recalculates subtree heights many times.

---

## Code (Brute Force)

```cpp
class Solution {
private:
    int getHeight(TreeNode* root) {
        if(root == nullptr) {
            return 0;
        }

        int leftHeight = getHeight(root->left);
        int rightHeight = getHeight(root->right);

        return max(leftHeight, rightHeight) + 1;
    }

public:

    bool isBalanced(TreeNode* root) {

        if(root == nullptr) return true;

        int leftHeight = getHeight(root->left);
        int rightHeight = getHeight(root->right);

        if(abs(leftHeight - rightHeight) <= 1 &&
           isBalanced(root->left) &&
           isBalanced(root->right)) {
            return true;
        }

        return false;
    }
};
```

---

## Code Explanation

1. `getHeight()` calculates the height of a subtree.

2. Base condition:

```
if(root == nullptr) return 0;
```

3. Recursively compute left and right heights.

4. Return the height of the current node.

```
max(leftHeight, rightHeight) + 1
```

5. In `isBalanced()`:

- compute left subtree height
- compute right subtree height

6. Check the balance condition:

```
abs(leftHeight - rightHeight) <= 1
```

7. Also recursively check if the left subtree and right subtree are balanced.

Only if **all three conditions are true**, the tree is balanced.

---

## Time Complexity

O(N²)

Reason:

For every node we compute height again using recursion.

Example worst case:

Skewed tree with N nodes

Height calculation takes O(N)

And we perform it for each node.

So total time becomes:

O(N × N) = O(N²)

---

## Space Complexity

O(H)

Where H is the height of the tree due to recursion stack.

Worst Case:

Skewed tree

H = N

Best Case:

Balanced tree

H = log N

---

# 2. Optimal Approach

## Intuition

The main inefficiency in the brute force solution is **recomputing heights repeatedly**.

We can optimize by calculating **height and balance condition simultaneously**.

Idea:

During DFS, return:

• Height if subtree is balanced
• -1 if subtree is unbalanced

Once we detect `-1`, we propagate it upward and stop further unnecessary calculations.

This avoids repeated height computations.

---

## Code (Optimal)

```cpp
class Solution {
public:

    bool isBalanced(TreeNode* root) {
        return dfsHeight(root) != -1;
    }

    int dfsHeight(TreeNode* root) {

        if (root == NULL)
            return 0;

        int leftHeight = dfsHeight(root->left);

        if (leftHeight == -1)
            return -1;

        int rightHeight = dfsHeight(root->right);

        if (rightHeight == -1)
            return -1;

        if (abs(leftHeight - rightHeight) > 1)
            return -1;

        return max(leftHeight, rightHeight) + 1;
    }
};
```

---

## Code Explanation

Step 1

`dfsHeight()` calculates the height of the subtree.

Step 2

If root is NULL return height = 0.

Step 3

Recursively compute left subtree height.

Step 4

If left subtree is already unbalanced, immediately return `-1`.

Step 5

Compute right subtree height.

Step 6

If right subtree is unbalanced, return `-1`.

Step 7

Check balance condition:

```
abs(leftHeight - rightHeight) > 1
```

If true, return `-1`.

Step 8

If balanced, return the height:

```
max(leftHeight, rightHeight) + 1
```

Step 9

In the main function:

```
return dfsHeight(root) != -1;
```

If the result is `-1`, the tree is not balanced.

Otherwise, it is balanced.

---

# Time & Space Complexity

## Time Complexity

O(N)

Each node is visited **only once**.

So operations are proportional to the number of nodes.

---

## Space Complexity

O(H)

H = height of the tree.

Due to recursion stack.

Worst Case:

Skewed tree

H = N

Best Case:

Balanced tree

H = log N

---

# Key Interview Insight

This problem demonstrates an important tree optimization pattern:

"Combine computation and validation in a single DFS traversal."

Instead of computing height separately for each node, we propagate failure using a special value (`-1`).

This pattern appears in many advanced tree problems such as:

• Diameter of Binary Tree
• Maximum Path Sum
• Validate BST
• Tree DP problems

Understanding this optimization is extremely important for technical interviews.

