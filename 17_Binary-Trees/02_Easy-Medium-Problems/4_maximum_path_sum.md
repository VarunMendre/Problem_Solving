# Maximum Path Sum in Binary Tree

## Problem
The **maximum path sum** is the maximum sum of values of any path in the binary tree.

- A path can start and end at **any node**.
- The path must follow **parent-child connections**.
- The path does **not need to pass through the root**.

---

## Key Idea

At every node, we consider:

- Maximum contribution from left subtree
- Maximum contribution from right subtree

We ignore negative paths because they reduce the sum.

---

## Algorithm

For every node:

1. Recursively get left subtree sum
2. Recursively get right subtree sum
3. Ignore negative sums:

```
leftSum = max(0, leftSum)
rightSum = max(0, rightSum)
```

4. Update global maximum:

```
maxi = max(maxi, leftSum + rightSum + root->val)
```

5. Return the best path including current node:

```
return root->val + max(leftSum, rightSum)
```

---

## Code

```cpp
class Solution {
private:
    int dfs(TreeNode* root, int& maxi) {
        if(root == nullptr) return 0;

        int leftSum = max(0, dfs(root->left, maxi));
        int rightSum = max(0, dfs(root->right, maxi));

        maxi = max(maxi, leftSum + rightSum + root->val);

        return root->val + max(leftSum, rightSum);
    }

public:
    int maxPathSum(TreeNode* root) {
        int maxi = INT_MIN;
        dfs(root, maxi);
        return maxi;
    }
};
```

---

## Time Complexity

```
O(N)
```

- Every node is visited exactly once.

---

## Space Complexity

```
O(H)
```

- Due to recursion stack.
- In worst case (skewed tree): **O(N)**

---

## Intuition (Very Important)

At each node, there are **two roles**:

### 1. As a "connector" (Full path through node)

```
leftSum + root->val + rightSum
```

Used to update the global answer.

---

### 2. As a "path extender" (Return value)

```
root->val + max(leftSum, rightSum)
```

Used to pass value to parent.

---

## Why max(0, ...)?

If a subtree gives negative sum:

- It will reduce overall path sum
- So we **ignore it completely**

---

## Important Interview Insight

This problem follows the pattern:

```
Return something to parent
Update global answer separately
```

Used in:

- Maximum Path Sum
- Diameter of Binary Tree
- Balanced Binary Tree

---

## Common Mistakes

1. Not handling negative values properly
2. Returning full path instead of single path
3. Forgetting to update global maximum
4. Initializing maxi with 0 instead of INT_MIN

---

## Summary

- Use DFS
- Ignore negative paths
- Track global maximum
- Return best single path to parent

