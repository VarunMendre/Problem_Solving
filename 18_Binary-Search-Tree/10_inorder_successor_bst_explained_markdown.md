# Inorder Successor in BST

## Problem Statement
Given the root of a Binary Search Tree (BST) and a node `k`, find the **Inorder Successor** of `k`.

The inorder successor of a node is the **smallest node greater than the given node** in BST.

---

# 1. Brute Force Approach

## Intuition / Approach

In a BST, performing an **Inorder Traversal** gives nodes in **sorted order**.

So the idea is:

1. Store the inorder traversal in an array.
2. Traverse the array.
3. Find the first element greater than `k->data`.
4. Return that element.

### Why This Works?
Because inorder traversal of BST always produces nodes in sorted order.

Example:

```text
        5
      /   \
     3     7
    / \   / \
   2   4 6   8
```

Inorder Traversal:

```text
2 3 4 5 6 7 8
```

If `k = 5`, then the first element greater than `5` is `6`, so successor is `6`.

---

## Code

```cpp
class Solution {

  private: 

    void inorderTraversal(Node* root, vector<int>& inorder) {

        if(root == NULL)
            return;

        // Left -> Root -> Right
        inorderTraversal(root->left, inorder);

        inorder.push_back(root->data);

        inorderTraversal(root->right, inorder);
    }

  public:

    int inOrderSuccessor(Node *root, Node *k) {

        vector<int> inorder;

        inorderTraversal(root, inorder);

        for(int i = 0; i < inorder.size(); i++) {

            if(inorder[i] > k->data) {
                return inorder[i];
            }
        }

        return -1;
    }
};
```

---

## Time & Space Complexity

### Time Complexity

- Inorder Traversal → `O(N)`
- Linear Search → `O(N)`

Overall:

```text
O(N)
```

---

### Space Complexity

- Inorder array stores all nodes.
- Recursive stack space also used.

Overall:

```text
O(N)
```

---

## Issues in This Approach

### Extra Space Usage
We are storing the entire inorder traversal unnecessarily.

### Not Using BST Property Efficiently
BST provides ordered structure, but this approach treats it almost like a normal binary tree.

### Traversing Entire Tree
Even after finding the successor conceptually, traversal continues because we first build complete inorder traversal.

---

# 2. Better Approach (Morris Inorder Traversal)

## Intuition / Approach

Instead of storing the inorder traversal in an array, we can generate inorder traversal on the fly using **Morris Traversal**.

Morris Traversal:

- Performs inorder traversal.
- Uses no recursion.
- Uses no stack.
- Temporarily modifies tree links.

### Main Idea

While performing inorder traversal:

- The first node greater than `k->data` will be the inorder successor.
- Return immediately once found.

This avoids storing inorder traversal.

---

## Code

```cpp
class Solution {

    public:

    int inOrderSuccessor(Node *root, Node *k) {

        Node* cur = root;

        while (cur != NULL) {

            if (cur->left == NULL) {

                if (cur->data > k->data) {
                    return cur->data;
                }

                cur = cur->right;

            } else {

                Node* prev = cur->left;

                while (prev->right && prev->right != cur) {
                    prev = prev->right;
                }

                if (prev->right == NULL) {

                    prev->right = cur;
                    cur = cur->left;

                } else {

                    prev->right = NULL;

                    if (cur->data > k->data) {
                        return cur->data;
                    }

                    cur = cur->right;
                }
            }
        }

        return -1;
    }
};
```

---

## Time & Space Complexity

### Time Complexity

Each edge is visited at most twice.

```text
O(N)
```

---

### Space Complexity

No recursion stack.
No extra array.

```text
O(1)
```

---

## Issues in This Approach

### Still Traversing Many Nodes
Even though space is optimized, we may still traverse large portions of the tree.

### Temporary Tree Modification
Morris traversal temporarily changes tree links.
Although restored later, it makes implementation harder and less intuitive.

### Not Fully Utilizing BST Property
We are still doing inorder traversal rather than intelligently moving using BST ordering.

---

# 3. Optimal Approach

## Intuition / Approach

This approach fully utilizes BST properties.

### Key Observation

For inorder successor:

- We need the smallest value greater than `k->data`.

At every node:

### Case 1:
If:

```text
cur->data <= k->data
```

Then:

- Current node cannot be successor.
- Successor must exist in right subtree.

Move right.

---

### Case 2:
If:

```text
cur->data > k->data
```

Then:

- Current node can be a potential successor.
- But there might exist a smaller valid successor in left subtree.

Store current node as answer and move left.

---

## Dry Run

Example:

```text
        20
       /  \
      10   30
          /  \
         25   40
```

Find successor of `20`.

### Step 1
`cur = 20`

```text
20 <= 20
```

Move right.

---

### Step 2
`cur = 30`

```text
30 > 20
```

Potential successor = 30.
Move left.

---

### Step 3
`cur = 25`

```text
25 > 20
```

Better successor found.
Move left.

Left becomes NULL.

Answer = 25.

---

## Code

```cpp
class Solution {

  public:

    int inOrderSuccessor(Node *root, Node *k) {

        Node* cur = root;

        int successor = -1;

        while(cur) {

            if(cur->data <= k->data) {

                cur = cur->right;

            } else {

                successor = cur->data;

                cur = cur->left;
            }
        }

        return successor;
    }
};
```

---

## Time & Space Complexity

### Time Complexity

In BST, we only move along tree height.

```text
O(H)
```

Where:

- `H = Height of BST`

For:

- Balanced BST → `O(log N)`
- Skewed BST → `O(N)`

---

### Space Complexity

No recursion.
No extra data structure.

```text
O(1)
```

---

# Why Optimal Approach is Best

| Approach | Time | Space | Uses BST Property Efficiently |
|---|---|---|---|
| Brute Force | O(N) | O(N) | ❌ |
| Morris Traversal | O(N) | O(1) | ❌ |
| Optimal BST Traversal | O(H) | O(1) | ✅ |

---

# Final Takeaway

The optimal solution works because BST already maintains sorted ordering inherently.

Instead of generating inorder traversal explicitly, we intelligently move:

- Right when current node is too small.
- Left when current node can be a possible answer.

This reduces