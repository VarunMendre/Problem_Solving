# BST Iterator

---

## 1. Intuition / Approach

Design an iterator that returns BST nodes in **ascending order** (inorder) one at a time, with `O(H)` space instead of `O(N)`.

### Naive Approach (Why We Don't Do It)

Do a full inorder traversal upfront, store all values in a vector, maintain an index. Works but uses `O(N)` space — defeats the purpose of an iterator.

---

### Optimal — Lazy Inorder with a Stack

The key insight is to **simulate inorder traversal on demand** — only compute the next element when `next()` is called.

**Core idea:** At any point, the stack holds the "pending left spine" — the leftmost path from the current position. The top of the stack is always the **next smallest element** to be returned.

---

### `pushAll(node)` — The Helper

```cpp
void pushAll(TreeNode* node) {
    while(node) {
        st.push(node);
        node = node->left;
    }
}
```

Pushes an entire **left spine** onto the stack (node → node->left → node->left->left → ...). This sets up the stack so the topmost element is always the smallest unvisited node.

---

### `next()` — Get Smallest Unvisited

```cpp
int next() {
    TreeNode* node = st.top();
    st.pop();
    pushAll(node->right);   // push right subtree's left spine
    return node->val;
}
```

1. Pop the top (smallest unvisited node)
2. Push the **left spine of its right subtree** — these are the next candidates in inorder order
3. Return the value

---

### `hasNext()` — Any More Elements?

```cpp
bool hasNext() { return !st.empty(); }
```

Stack empty → all nodes exhausted.

---

### Dry Run

```
        7
       / \
      3  15
        /  \
       9   20

Constructor: pushAll(7) → stack = [7, 3] (3 on top)

next():
  pop 3 → pushAll(3->right = NULL) → nothing pushed
  stack = [7]
  return 3

next():
  pop 7 → pushAll(7->right = 15) → push 15, then 9
  stack = [15, 9] (9 on top)
  return 7

next():
  pop 9 → pushAll(9->right = NULL) → nothing pushed
  stack = [15]
  return 9

next():
  pop 15 → pushAll(15->right = 20) → push 20
  stack = [20]
  return 15

next():
  pop 20 → pushAll(NULL) → nothing pushed
  stack = []
  return 20

hasNext() → false ✅

Sequence: 3, 7, 9, 15, 20 ✅ (sorted ascending)
```

---

## 2. Code

```cpp
class BSTIterator {
    stack<TreeNode*> st;

private:
    // Push the entire left spine starting from node
    void pushAll(TreeNode* node) {
        while(node) {
            st.push(node);
            node = node->left;
        }
    }

public:
    // Initialize: push left spine from root
    BSTIterator(TreeNode* root) {
        pushAll(root);
    }

    // Returns next smallest element
    int next() {
        TreeNode* node = st.top();
        st.pop();

        // Push left spine of right subtree — next candidates in inorder
        pushAll(node->right);

        return node->val;
    }

    // Returns true if more elements exist
    bool hasNext() {
        return !st.empty();
    }
};
```

---

## 3. Time & Space Complexity

| Operation | Time | Reason |
|---|---|---|
| `BSTIterator()` | `O(H)` | Pushes left spine from root — at most H nodes |
| `next()` | `O(H)` amortized | Each node pushed and popped exactly once across all calls; `pushAll` may push up to H nodes |
| `hasNext()` | `O(1)` | Just checks if stack is empty |

| | Space | Reason |
|---|---|---|
| **Stack** | `O(H)` | Stack holds at most one left spine at a time — H nodes |

> `H` = height of tree = `O(log N)` balanced, `O(N)` skewed  
> **`next()` is `O(H)` amortized, not `O(H)` per call** — across N calls to `next()`, total pushAll work = N (each node pushed once) → `O(N)/N` = `O(1)` per call on average. But worst case single call is `O(H)`.
