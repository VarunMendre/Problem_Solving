#

#  Tree Traversal Concepts (DFS & BFS)

Tree traversal means **visiting every node of the tree exactly once in a specific order**.

Traversal is extremely important because most algorithms on trees rely on visiting nodes in a structured way.

There are two fundamental traversal strategies:

1. **Depth First Search (DFS)**
2. **Breadth First Search (BFS)**

These strategies define *how we explore the tree structure*.

---

# 1. Depth First Search (DFS)

Depth First Search explores the tree by **going as deep as possible into one branch before backtracking**.

Think of it like exploring a cave:

- You go deep into one tunnel
- When it ends, you come back
- Then explore another tunnel

DFS follows the natural recursive structure of trees.

Example tree:

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

DFS traversal explores nodes by diving into the left subtree first and then the right subtree.

DFS has **three important traversal orders**:

1. Preorder
2. Inorder
3. Postorder

These differ only in **when the root node is processed relative to its children**.

---

# 1.1 Preorder Traversal

Preorder traversal follows the order:

```
Root → Left Subtree → Right Subtree
```

Conceptually:

1. Visit the current node first
2. Traverse the left subtree
3. Traverse the right subtree

Example:

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

Traversal order:

```
A → B → D → E → C → F
```

Why this order matters:

Preorder is useful when we want to **copy or serialize a tree structure** because the root is processed before its children.

Applications:

- Creating a copy of a tree
- Prefix expression evaluation
- Tree serialization

---

# 1.2 Inorder Traversal

Inorder traversal follows the order:

```
Left Subtree → Root → Right Subtree
```

Conceptually:

1. Traverse the left subtree first
2. Visit the root node
3. Traverse the right subtree

Example tree:

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

Traversal order:

```
D → B → E → A → C → F
```

Why this traversal is important:

In a **Binary Search Tree**, inorder traversal produces the nodes in **sorted order**.

Applications:

- Getting sorted output from a BST
- Expression trees
- Syntax tree evaluation

---

# 1.3 Postorder Traversal

Postorder traversal follows the order:

```
Left Subtree → Right Subtree → Root
```

Conceptually:

1. Traverse the left subtree
2. Traverse the right subtree
3. Visit the root node last

Example:

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

Traversal order:

```
D → E → B → F → C → A
```

Why postorder matters:

This traversal processes children before their parent, which is useful when **deleting or evaluating trees**.

Applications:

- Deleting a tree
- Postfix expression evaluation
- Memory cleanup of hierarchical structures

---

# 2. Breadth First Search (BFS)

Breadth First Search explores the tree **level by level**.

Instead of going deep first, BFS explores all nodes at the same level before moving to the next level.

Example tree:

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

BFS traversal order:

```
A → B → C → D → E → F
```

This traversal is also called:

```
Level Order Traversal
```

Why BFS is useful:

Because it processes nodes level‑wise, BFS is extremely useful for problems involving **distance, shortest path, or level grouping**.

Applications:

- Level order traversal
- Shortest path in trees
- Finding minimum depth
- Printing tree level by level

---

# 3. Conceptual Difference: DFS vs BFS

DFS explores one branch deeply before backtracking.

BFS explores nodes level by level across the tree.

Visual intuition:

DFS movement:

```
A → B → D → back → E → back → C → F
```

BFS movement:

```
A → B → C → D → E → F
```

Key conceptual difference:

- DFS prioritizes **depth**
- BFS prioritizes **levels**

Both are fundamental techniques used across:

- Tree algorithms
- Graph algorithms
- Path finding
- AI search systems

---

# 4. Why Tree Traversals Are Critical in DSA

Understanding traversal patterns is essential because almost every binary tree problem relies on one of these strategies.

Examples of problems solved using traversals:

- Tree height calculation
- Diameter of tree
- Lowest common ancestor
- Maximum path sum
- Tree serialization
- Binary search tree validation

Traversal is essentially the **foundation for all tree algorithms**.

Once these concepts are clear, the next step is learning how to **apply these traversals to solve real interview problems**.

---

End of conceptual traversal guide.

