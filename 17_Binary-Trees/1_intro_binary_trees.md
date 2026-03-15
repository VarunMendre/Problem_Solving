# Complete Guide: Binary Trees (In‑Depth)

## 1. Introduction to Trees

Before trees, most data structures we learn are **linear**:

- Array
- Linked List
- Stack
- Queue

In linear structures, elements are arranged sequentially.

Trees are **non‑linear data structures** that represent **hierarchical relationships**.

Example from real life:

Computer File System

```
Root
 ├── Documents
 │    ├── Resume.pdf
 │    └── Notes.txt
 └── Pictures
      ├── Trip
      └── Family
```

This hierarchy is exactly how a **tree data structure** works.

---

# 2. What is a Binary Tree?

A **Binary Tree** is a hierarchical data structure where:

- Each node contains data
- Each node can have **at most two children**

These children are called:

- Left Child
- Right Child

### Simple Binary Tree

```
        A
       / \
      B   C
     / \   \
    D   E   F
```

---

# 3. Components of a Binary Tree

## Node

A node stores:

- Data
- Pointer to left child
- Pointer to right child

Example structure (C++):

```
struct Node {
    int data;
    Node* left;
    Node* right;
};
```

---

## Root Node

The **topmost node** of the tree.

```
        A   <- Root
       / \
      B   C
```

---

## Parent and Child Nodes

A node connected downward is a **child**.

```
      A
     / \
    B   C
```

A = Parent

B,C = Children

---

## Leaf Nodes

Nodes that have **no children**.

```
        A
       / \
      B   C
     /
    D
```

Leaf Nodes: **C, D**

---

## Ancestors

Nodes that lie on the path from a node to the root.

Example:

```
        A
       / \
      B   C
     / \
    D   E
```

Ancestors of **E**:

```
B -> A
```

---

# 4. Types of Binary Trees

---

# 4.1 Full Binary Tree

Also called **Strict Binary Tree**.

Property:

Every node has either:

- 0 children
- 2 children

No node has only **one child**.

### Example

```
        A
       / \
      B   C
     / \ / \
    D  E F  G
```

Characteristics:

- Internal nodes always have 2 children
- Leaf nodes have 0 children

Benefits:

- More predictable structure
- Better balance

---

# 4.2 Complete Binary Tree

Definition:

All levels are completely filled **except possibly the last level**.

The last level is filled **from left to right**.

### Example

```
        1
       / \
      2   3
     / \  /
    4  5 6
```

Key Properties:

- Leaves appear in the last or second last level
- Nodes are filled left to right

Used in:

- **Heap Data Structure**

Because array representation becomes very efficient.

---

# 4.3 Perfect Binary Tree

A tree where:

- All internal nodes have 2 children
- All leaf nodes are at the same level
- All levels are completely filled

### Example

```
        1
       / \
      2   3
     / \ / \
    4  5 6  7
```

Properties:

Number of nodes =

```
2^h - 1
```

Where **h = height**.

Characteristics:

- Completely balanced
- Maximum number of nodes

However, rarely happens in real applications.

---

# 4.4 Balanced Binary Tree

A tree where:

The difference between the height of left and right subtree is:

```
<= 1
```

Example:

```
        10
       /  \
      5    15
     / \     \
    3   7     18
```

Benefits:

- Height remains **log(N)**
- Searching becomes efficient

Time complexity:

Search = **O(log n)**

Examples of balanced trees:

- AVL Tree
- Red Black Tree

---

# 4.5 Degenerate Binary Tree

A **degenerate tree** behaves like a **linked list**.

Every parent has only **one child**.

### Example

```
1
 \
  2
   \
    3
     \
      4
```

Problems:

- Height becomes **N**
- Search complexity becomes

```
O(n)
```

This is the **worst case of a binary tree**.

---

# 5. Important Terminology

## Height of Tree

Number of edges in longest path from root to leaf.

Example:

```
        A
       / \
      B   C
     /
    D
```

Height = 2

---

## Depth of Node

Distance from root to that node.

Example:

Depth(D) = 2

---

## Level of Node

```
Level = Depth + 1
```

Root is level **1**.

---

# 6. Binary Tree Representation

## 1. Pointer Representation

```
class Node {
  public:
  int data;
  Node* left;
  Node* right;
};
```

---

## 2. Array Representation (Used in Heap)

Example tree

```
        10
       /  \
      20   30
     /  \
    40  50
```

Array representation:

```
[10, 20, 30, 40, 50]
```

Relations:

```
Left child  = 2*i + 1
Right child = 2*i + 2
Parent      = (i-1)/2
```

---

# 7. Why Binary Trees are Important

Binary Trees are the foundation of many advanced data structures:

- Binary Search Tree
- Heap
- AVL Tree
- Red Black Tree
- Segment Tree
- Fenwick Tree

They are used in:

- Databases
- File systems
- Expression parsing
- Compilers
- Artificial Intelligence

---

# 8. Summary

Binary Trees represent **hierarchical data structures** where each node can have at most **two children**.

Key concepts include:

- Root node
- Leaf nodes
- Ancestors
- Parent / child relationships

Important tree types:

- Full Binary Tree
- Complete Binary Tree
- Perfect Binary Tree
- Balanced Binary Tree
- Degenerate Tree

Understanding these concepts forms the **foundation for solving tree problems in DSA interviews**.

The next important step after this topic is:

1. Tree Traversals
2. Binary Search Tree
3. Heap / Priority Queue

