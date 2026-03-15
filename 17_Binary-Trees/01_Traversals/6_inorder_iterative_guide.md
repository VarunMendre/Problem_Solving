# Binary Tree Inorder Traversal (Iterative using Stack)

## 1. Approach / Intuition

Inorder traversal is a Depth First Search (DFS) technique used to traverse a binary tree.

The order followed in inorder traversal is:

Left → Root → Right

This means:
1. First traverse the entire left subtree.
2. Visit the current node.
3. Then traverse the right subtree.

Recursive solutions are very straightforward for inorder traversal. However, to implement it iteratively we must simulate the recursion using a stack.

Why do we need a stack?

During recursion, function calls are stored in the program's call stack. When we convert recursion to iteration, we manually manage this stack using a stack data structure.

The key challenge in inorder traversal is that we must reach the **leftmost node first** before visiting any node.

Key Idea of the Algorithm:

1. Start with the root node.
2. Keep pushing nodes into the stack while moving toward the left child.
3. When we reach a NULL node, it means the leftmost node has been reached.
4. Now we start processing nodes from the stack.
5. Pop the top node from the stack and add its value to the result.
6. Move to the right child of that node.
7. Repeat the same process until both the stack is empty and the node becomes NULL.

Example Tree:

        1
       / \
      2   3
     / \
    4   5

Inorder traversal result:

[4, 2, 5, 1, 3]

Traversal flow:

Go left as far as possible → process node → go right → repeat.

---

## 2. Code

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> inOrder;
        if(root == NULL) return inOrder;

        TreeNode* node = root;
        stack<TreeNode*> st;

        while(true) {
            if(node != NULL) {
                st.push(node);
                node = node->left;
            }
            else {
                if(st.empty()) break;
                node = st.top();
                st.pop();
                inOrder.push_back(node->val);
                node = node->right;
            }
        }

        return inOrder;
    }
};
```

Code Explanation:

1. A vector named `inOrder` is created to store the traversal result.

2. If the root node is NULL, the tree is empty, so return the empty vector.

3. A pointer `node` is initialized to the root. This pointer will help traverse the tree.

4. A stack is created to simulate the recursion stack used in the recursive approach.

5. The loop runs indefinitely using `while(true)`.

6. If the current node is not NULL:
   - Push the node into the stack.
   - Move to the left child.

   This step continues until the algorithm reaches the leftmost node.

7. If the current node becomes NULL:
   - Check if the stack is empty.
   - If the stack is empty, traversal is complete.

8. Otherwise:
   - Retrieve the node from the top of the stack.
   - Remove it from the stack.

9. Add that node's value to the result vector because inorder visits the node after finishing the left subtree.

10. Move to the right child of that node.

11. The process repeats: again exploring the left subtree of the right child.

12. The algorithm stops when both conditions occur:
   - `node == NULL`
   - `stack is empty`

13. Finally return the `inOrder` vector containing the traversal.

---

## 3. Time & Space Complexity

### Time Complexity: O(N)

Where **N** is the number of nodes in the binary tree.

Reason:

Each node is:
- pushed into the stack once
- popped from the stack once

Thus every node is processed exactly one time.

Total operations are proportional to the number of nodes.

Therefore:

Time Complexity = **O(N)**

---

### Space Complexity: O(N)

The stack stores nodes during traversal.

Worst case occurs when the tree is highly skewed.

Example:

1
/
2
/
3
/
4

In such cases, the stack may store all nodes along the path.

Maximum stack size = N

Additionally, the result vector also stores all N nodes.

Therefore:

Space Complexity = **O(N)**

---

## Key Interview Insight

This iterative algorithm simulates recursion using an explicit stack.

The main trick interviewers expect you to understand is:

"Always go to the leftmost node first before processing the node."

This ensures the traversal follows the inorder rule:

Left → Root → Right

Understanding this pattern helps solve many advanced tree problems such as:

• Binary Search Tree validation
• Kth smallest element in BST
• Tree flattening problems

