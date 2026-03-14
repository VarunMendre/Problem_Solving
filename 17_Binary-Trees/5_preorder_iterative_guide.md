# Binary Tree Preorder Traversal (Iterative using Stack)

## 1. Approach / Intuition

Preorder traversal is a Depth First Search (DFS) technique used to traverse a binary tree.

The order followed in preorder traversal is:

Root → Left → Right

This means:
1. Visit the current node.
2. Traverse the left subtree.
3. Traverse the right subtree.

Recursive solutions are the most common way to implement preorder traversal. However, interviewers often ask for the iterative version because it demonstrates a deeper understanding of how recursion works internally.

To implement the iterative approach, we use a **stack** data structure.

Why Stack?

A stack follows the LIFO principle (Last In First Out). This mimics how recursive function calls are stored in the call stack.

Key idea of the algorithm:

1. Start from the root node.
2. Push the root node into the stack.
3. While the stack is not empty:
   - Remove the top node.
   - Visit that node (store its value).
   - Push its right child into the stack.
   - Push its left child into the stack.

Why push **right first and then left**?

Because the stack processes elements in reverse order (LIFO). If we push the right child first and then the left child, the left child will be processed first, maintaining the preorder rule:

Root → Left → Right

Example Tree:

        1
       / \
      2   3
     / \
    4   5

Preorder traversal result:

[1, 2, 4, 5, 3]

Traversal flow:

Visit root → go left → continue left → then right → then right subtree.

---

## 2. Code

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> answer;
        
        if(root == NULL) return answer;

        stack<TreeNode*> st;
        st.push(root);

        while(!st.empty()) {
            TreeNode* node = st.top();
            st.pop();
            
            answer.push_back(node->val);

            if(node->right) st.push(node->right);
            if(node->left) st.push(node->left);
        }

        return answer;
    }
};
```

Code Explanation:

1. A vector named `answer` is created to store the preorder traversal result.

2. If the root node is NULL, the tree is empty, so return the empty result.

3. A stack is created to help perform iterative DFS traversal.

4. The root node is pushed into the stack because traversal always begins from the root.

5. A loop runs until the stack becomes empty.

6. `st.top()` retrieves the node at the top of the stack.

7. `st.pop()` removes that node after retrieving it.

8. The node value is added to the result because preorder visits the node immediately.

9. If the node has a right child, push it into the stack.

10. If the node has a left child, push it into the stack.

11. The left child will be processed before the right child because the stack pops the last pushed element first.

12. Continue this process until the stack becomes empty.

13. Return the final traversal stored in `answer`.

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

Worst case occurs when the tree is highly unbalanced (like a linked list).

Example:

1
 \
  2
   \
    3
     \
      4

In such cases, the stack may store all nodes along the path.

Maximum stack size = N

Additionally, the result vector also stores all N nodes.

Therefore:

Space Complexity = **O(N)**

---

## Key Interview Insight

This iterative approach is essentially simulating recursion using a stack.

Recursive preorder traversal internally performs the same steps using the program's call stack.

Interviewers often prefer this solution because it shows:

• Understanding of DFS
• Knowledge of stack-based traversal
• Ability to convert recursion into iteration

