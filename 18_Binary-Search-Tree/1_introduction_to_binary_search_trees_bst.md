# Introduction to Binary Search Trees (BST)

## 🌳 What is a Binary Search Tree?
A **Binary Search Tree (BST)** is a special type of **Binary Tree** that is designed to make searching, insertion, and deletion operations efficient.

In a BST, each node follows a strict rule:

```
Left Child < Node < Right Child
```

This means:
- All values in the **left subtree** are smaller than the node
- All values in the **right subtree** are greater than the node

Also, **every subtree itself is a BST**.

---

## 🧠 Key Properties of BST
- Each node has at most **two children** (left and right)
- The tree maintains a **sorted structure**
- **No duplicate values** are allowed (in standard BST)
- Recursive nature: every subtree is also a BST

---

## 🌲 Example Structure

```
        10
       /  \
      5    15
     / \     \
    2   7     20
```

Here:
- Left subtree of 10 → {5, 2, 7} (all smaller)
- Right subtree of 10 → {15, 20} (all greater)

---

## ❗ Handling Duplicates (Optional Cases)
Normally, BST does **not allow duplicate values**, but in some implementations:
- Duplicates can be stored in the **right subtree**
- Or maintain a **count field** in the node

---

## 🚀 Why Do We Need BST?

### 🔍 Faster Searching
BST allows efficient searching by eliminating half of the tree at each step (similar to Binary Search in arrays).

### ⏱ Time Complexity (Average Case)
| Operation | Time Complexity |
|----------|---------------|
| Search   | O(log N)      |
| Insert   | O(log N)      |
| Delete   | O(log N)      |

### ⚠️ Worst Case
If the tree becomes **skewed** (like a linked list):
- Height becomes **O(N)**
- Operations degrade to **O(N)**

---

## 🌟 How Searching Works in BST

1. Start from the **root node**
2. Compare the value:
   - If equal → Element found ✅
   - If smaller → Move to left subtree
   - If greater → Move to right subtree
3. Repeat until found or node becomes NULL

---

## 🔗 BST vs Normal Binary Tree

| Feature | Binary Tree | Binary Search Tree |
|--------|------------|--------------------|
| Order  | No specific order | Sorted structure |
| Search | O(N) | O(log N) (average) |
| Use case | General tree problems | Fast searching, sorting |

---

## 💡 Real-Life Analogy
Think of BST like a **dictionary**:
- Words are arranged in order
- You don’t scan every word
- You jump left or right based on comparison

---

## 📌 Summary
- BST is an optimized version of a binary tree for searching
- Follows: **Left < Root < Right**
- Provides efficient operations in average case
- Performance depends on tree balance

---

## 🚀 Next Steps
Once you're comfortable with basics, you should learn:
- Insertion in BST
- Deletion in BST
- Traversals (Inorder, Preorder, Postorder)
- Balanced BSTs (AVL, Red-Black Trees)

---

Happy Coding! 💻

