# Children Sum Property in a Binary Tree

---

## 1. Intuition / Approach

The **Children Sum Property** states that for every node, its value must equal the **sum of its left and right children's values**.

The goal is to modify the tree so that every node satisfies this property. We are allowed to **only increment** node values (not decrement).

---

### The Challenge

A simple bottom-up approach won't work — if a parent is larger than its children's sum, we'd need to increase the children, but that might break the property for nodes below them.

### The Two-Phase Strategy

We solve this with a **modified preorder DFS** (process node → recurse → update node):

---

**Phase 1 — Going Down (preorder): Fix children before recursing**

At each node, compare `childSum` with `root->data`:

- **If `childSum >= root->data`:**
  Promote the parent's value up → `root->data = childSum`
  *(parent takes the larger sum — property holds locally)*

- **If `childSum < root->data`:**
  Push the parent's value **down** to children → whichever child exists gets `root->data`
  *(we inflate the child so it's at least as large as the parent — this preserves the ability to satisfy the property deeper down)*

**Phase 2 — Coming Back Up (postorder): Recalculate parent from updated children**

After both subtrees are fully processed:
```cpp
if(root->left || root->right)
    root->data = root->left->data + root->right->data;
```
Now the parent is updated to the **true sum of its children** — guaranteeing the property holds on the way back up.

---

### Why push the parent's value down?

If child sum < parent, we can't decrease the parent (not allowed). So instead we **inflate the child** to match the parent. This ensures that when we come back up after recursion and recompute, the parent will always be ≥ its children's sum at minimum.

---

### Dry Run

```
        50
       /  \
      7    2
     / \    \
    3   5    1
```

**Going down:**
- Node 50: childSum = 7+2 = 9 < 50 → push 50 down to left child → left = 50
- Node 50: childSum = 50+2 = 52 ≥ 50 → root = 52... 

Actually let's trace more carefully:

```
Node(50): childSum=9 < 50 → left->data = 50
  Node(50): childSum = 3+5 = 8 < 50 → left->data = 50
    Node(50): leaf → return
    Node(5): leaf → return
  ← back at Node(50): total = 50+5 = 55 → node = 55
  Node(2): childSum = 0+1 = 1 < 2 → right->data = 2
    Node(2): leaf → return
  ← back at Node(2): total = 0+2 = 2 → node = 2
← back at Node(52): total = 55+2 = 57 → root = 57
```

Final tree satisfies children sum property at every node ✅

---

## 2. Code

```cpp
void changeTree(BinaryTreeNode<int>* root) {
    if(root == NULL) return;

    // Calculate sum of immediate children
    int childSum = 0;
    if(root->left)  childSum += root->left->data;
    if(root->right) childSum += root->right->data;

    // Phase 1 (going down): ensure children are >= parent
    if(childSum >= root->data) {
        // Children sum is sufficient — promote it to parent
        root->data = childSum;
    } else {
        // Parent is larger — push parent's value down to a child
        // so deeper recursion can propagate it further
        if(root->left)       root->left->data  = root->data;
        else if(root->right) root->right->data = root->data;
    }

    // Recurse on both subtrees
    changeTree(root->left);
    changeTree(root->right);

    // Phase 2 (coming back up): recompute parent as sum of updated children
    int total = 0;
    if(root->left)  total += root->left->data;
    if(root->right) total += root->right->data;

    // Only update if not a leaf
    if(root->left || root->right)
        root->data = total;
}
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space** | `O(H)` auxiliary | Recursive call stack depth = height of tree; `O(N)` worst case for a skewed tree |

> `N` = total number of nodes | `H` = height of the tree
