# Binary Tree Postorder Traversal (Iterative)

Postorder traversal is another Depth First Search (DFS) technique used to traverse binary trees.

Traversal order:

Left → Right → Root

This order is slightly harder to implement iteratively compared to preorder and inorder. Because the root must be processed **after** both left and right subtrees.

Below are two commonly used iterative approaches:

1. Using **Two Stacks**
2. Using **One Stack + Reverse Trick**

---

# 1. Approach / Intuition (Two Stack Method)

The main idea is to simulate the reverse of preorder traversal.

Normal preorder order:

Root → Left → Right

If we modify it to:

Root → Right → Left

and then reverse the result, we get:

Left → Right → Root

which is exactly **postorder traversal**.

Instead of reversing the result vector, we can use another stack to reverse the order automatically.

Algorithm steps:

1. Create two stacks: `st1` and `st2`.
2. Push the root node into `st1`.
3. While `st1` is not empty:
   - Pop a node from `st1`.
   - Push it into `st2`.
   - Push its left child into `st1`.
   - Push its right child into `st1`.
4. After traversal finishes, pop elements from `st2`.
5. The order obtained from `st2` will be postorder.

Why does this work?

Because `st2` reverses the order created by `st1`, giving us the correct postorder sequence.

---

# Code (Two Stack Method)

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> answer;

        if(root == NULL) return answer;

        stack<TreeNode*> st1, st2;

        st1.push(root);

        while(!st1.empty()) {
            root = st1.top();
            st1.pop();

            st2.push(root);

            if(root->left) {
                st1.push(root->left);
            }
            if(root->right) {
                st1.push(root->right);
            }
        }

        while(!st2.empty()) {
            answer.push_back(st2.top()->val);
            st2.pop();
        }

        return answer;
    }
};
```

Code Explanation:

1. `answer` vector stores the final postorder traversal.

2. If the tree is empty, return the empty vector.

3. Two stacks are used:
   - `st1` for traversal
   - `st2` to reverse the order

4. Root node is pushed into `st1`.

5. While `st1` is not empty:

   - Pop the top node.
   - Push it into `st2`.

6. Push the left child of the node into `st1`.

7. Push the right child of the node into `st1`.

8. After traversal, `st2` contains nodes in reverse order.

9. Pop elements from `st2` and store their values in the result.

10. The final result becomes **Left → Right → Root**.

---

# 2. Approach / Intuition (One Stack + Reverse Method)

This approach is an optimization of the previous idea.

Instead of using two stacks, we:

1. Use one stack
2. Store nodes in **Root → Right → Left order**
3. Reverse the result at the end

Why this works:

If we reverse

Root → Right → Left

we get

Left → Right → Root

which is postorder traversal.

Algorithm:

1. Push the root node into the stack.
2. While the stack is not empty:
   - Pop the node.
   - Add its value to the result.
   - Push its left child.
   - Push its right child.
3. After traversal, reverse the result vector.

---

# Code (One Stack Method)

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> answer;

        if(root == NULL) return answer;

        stack<TreeNode*> st1;

        st1.push(root);

        while(!st1.empty()) {
            root = st1.top();
            st1.pop();

            answer.push_back(root->val);

            if(root->left) {
                st1.push(root->left);
            }
            if(root->right) {
                st1.push(root->right);
            }
        }

        reverse(answer.begin(), answer.end());
        return answer;
    }
};
```

Code Explanation:

1. `answer` stores traversal result.

2. If the tree is empty, return the empty vector.

3. Create a stack and push the root.

4. While the stack is not empty:

   - Pop the top node.
   - Add its value to the result vector.

5. Push the left child into the stack.

6. Push the right child into the stack.

7. This creates traversal order:

Root → Right → Left

8. Reverse the result vector to obtain:

Left → Right → Root

which is postorder traversal.

---

# Time & Space Complexity

## Time Complexity: O(N)

Where **N** is the number of nodes in the binary tree.

Each node is:

• pushed into the stack once
• popped from the stack once

Thus every node is processed exactly one time.

Total operations are proportional to the number of nodes.

Time Complexity = **O(N)**

---

## Space Complexity: O(N)

Worst case occurs when the tree is skewed.

Example:

1
 \
  2
   \
    3
     \
      4

In this case the stack may store all nodes along the path.

Maximum stack size = **N**.

Additionally, the result vector stores **N nodes**.

Total Space Complexity = **O(N)**.

---

# Key Interview Insight

Postorder traversal is considered the **hardest traversal to implement iteratively**.

Interviewers usually expect you to know:

• Two stack solution
• One stack + reverse trick

Advanced interviews may also ask for:

• **Single stack true postorder traversal** (harder approach).

Understanding these techniques helps in solving many tree problems such as:

• Deleting a binary tree
• Evaluating expression trees
• Tree dynamic programming problems

