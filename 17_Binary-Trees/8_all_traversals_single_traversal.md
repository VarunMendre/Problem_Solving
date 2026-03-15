# Binary Tree Traversals (Preorder, Inorder, Postorder) in One Traversal

## 1. Approach / Intuition

Normally, binary tree traversals are implemented separately:

• Preorder  → Root → Left → Right
• Inorder   → Left → Root → Right
• Postorder → Left → Right → Root

This usually requires **three different traversals of the tree**.

However, we can compute **all three traversals in a single traversal** using a stack and a "state" technique.

The key idea is to simulate the recursion stack manually.

When recursion runs, each node goes through **three stages**:

1. Before visiting the left child  → Preorder stage
2. After finishing left subtree    → Inorder stage
3. After finishing right subtree   → Postorder stage

We simulate these stages using a **state variable**.

State meanings:

1 → Preorder stage
2 → Inorder stage
3 → Postorder stage

To track this, we use a stack storing:

(node, state)

Each node is pushed back into the stack with the next state after processing.

Algorithm Steps:

1. If root is NULL return empty result.
2. Create a stack of pair<Node*, int>.
3. Push (root, 1).
4. While stack is not empty:

   Case 1 (state = 1)
   - Process preorder
   - Increase state to 2
   - Push node back
   - Push left child

   Case 2 (state = 2)
   - Process inorder
   - Increase state to 3
   - Push node back
   - Push right child

   Case 3 (state = 3)
   - Process postorder

This ensures all three traversals are collected in **one traversal**.

---

## 2. Code

```cpp
vector<vector<int>> preInPostTraversal(Node* root) {

    vector<int> pre, in, post;

    if (root == NULL) {
        return {};
    }

    stack<pair<Node*, int>> st;

    st.push({root, 1});

    while (!st.empty()) {
        auto it = st.top();
        st.pop();

        if (it.second == 1) {

            pre.push_back(it.first->data);

            it.second = 2;
            st.push(it);

            if (it.first->left != NULL) {
                st.push({it.first->left, 1});
            }
        }

        else if (it.second == 2) {

            in.push_back(it.first->data);

            it.second = 3;
            st.push(it);

            if (it.first->right != NULL) {
                st.push({it.first->right, 1});
            }
        }

        else {

            post.push_back(it.first->data);
        }
    }

    vector<vector<int>> result;
    result.push_back(pre);
    result.push_back(in);
    result.push_back(post);

    return result;
}
```

---

## 3. Code Explanation

Step 1

Three vectors are created:

pre  → stores preorder traversal
in   → stores inorder traversal
post → stores postorder traversal

Step 2

If the tree is empty, return an empty result.

Step 3

Create a stack storing:

(node, state)

Example:

(node pointer, traversal stage)

Step 4

Push the root node with state = 1.

(root,1)

Step 5

While stack is not empty:

Pop the top element.

Step 6

If state == 1

This represents **preorder stage**.

So:

• Add node value to preorder
• Change state to 2
• Push node back to stack
• Push left child

Step 7

If state == 2

This represents **inorder stage**.

So:

• Add node value to inorder
• Change state to 3
• Push node back
• Push right child

Step 8

If state == 3

This represents **postorder stage**.

So:

• Add node value to postorder

Step 9

Continue until stack becomes empty.

Step 10

Return all three traversals as a vector of vectors.

---

## 4. Time & Space Complexity

### Time Complexity: O(N)

Where N is number of nodes.

Each node is pushed and popped at most **3 times**.

Operations are still proportional to N.

Therefore:

Time Complexity = O(N)

---

### Space Complexity: O(N)

Stack may contain nodes along a tree path.

Worst case occurs in skewed trees.

Example:

1
 \
  2
   \
    3

Stack size may grow up to N.

Additionally, traversal arrays store N elements.

Therefore:

Space Complexity = O(N)

---

## 5. Key Interview Insight

This problem is extremely popular in tree interviews.

Interviewers ask it to test:

• Understanding of recursion simulation
• Stack based traversal
• Ability to combine multiple traversals

Important idea:

"Each node has three moments in recursion: before left, between left and right, and after right."

Those correspond exactly to:

Preorder
Inorder
Postorder

By tracking node states, we simulate the recursion stack and compute all traversals in **one pass of the tree**.

