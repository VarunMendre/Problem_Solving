# Predecessor and Successor in a BST

---

## 1. Intuition / Approach

**Predecessor** = largest value **strictly less than** `key`  
**Successor** = smallest value **strictly greater than** `key`

We use the **BST property** to navigate directly — no need to visit all nodes.

---

### Finding the Successor

We want the **smallest value > key**. Navigate the BST:

- If `curr->data <= key` → the successor must be in the **right** subtree (current node is too small or equal) → go right
- If `curr->data > key` → current node **could be** the successor (it's greater than key), record it, but there might be a smaller valid candidate in the **left** subtree → go left

The last recorded value when going left is the answer — it's the smallest value seen that was still > key.

```
BST:       20
          /  \
         8   22
        / \
       4  12
         /  \
        10  14

key = 8

curr=20: 20 > 8  → successor=20, go left
curr=8:  8  <= 8 → go right
curr=12: 12 > 8  → successor=12, go left
curr=10: 10 > 8  → successor=10, go left
curr=NULL → stop

Successor = 10 ✅
```

---

### Finding the Predecessor

We want the **largest value < key**. Navigate the BST:

- If `curr->data >= key` → current node is too large or equal, predecessor must be in the **left** subtree → go left
- If `curr->data < key` → current node **could be** the predecessor, record it, but there might be a larger valid candidate in the **right** subtree → go right

```
key = 8

curr=20: 20 >= 8 → go left
curr=8:  8  >= 8 → go left
curr=4:  4  < 8  → predecessor=4, go right
curr=NULL → stop

Predecessor = 4 ✅
```

---

### Why two separate passes?

Both traversals start from root and use BST navigation independently. They could theoretically be combined in one pass, but keeping them separate makes the logic cleaner and easier to reason about. TC remains the same either way.

---

## 2. Code

```cpp
pair<int, int> predecessorSuccessor(TreeNode* root, int key) {
    TreeNode* curr = root;
    int successor   = -1;
    int predecessor = -1;

    // Find Successor: smallest value strictly greater than key
    while(curr) {
        if(curr->data <= key) {
            // Current node too small or equal → go right
            curr = curr->right;
        } else {
            // Current node is a candidate successor → record it, try smaller in left
            successor = curr->data;
            curr = curr->left;
        }
    }

    // Find Predecessor: largest value strictly less than key
    curr = root;
    while(curr) {
        if(curr->data >= key) {
            // Current node too large or equal → go left
            curr = curr->left;
        } else {
            // Current node is a candidate predecessor → record it, try larger in right
            predecessor = curr->data;
            curr = curr->right;
        }
    }

    return {predecessor, successor};
}
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(H)` | Each pass traverses at most one root-to-leaf path; `O(log N)` balanced, `O(N)` skewed |
| **Space** | `O(1)` | No extra data structures, no recursion |

> `H` = height of tree | `N` = total nodes  
> Two passes of `O(H)` each → total `O(2H)` = `O(H)`

---

## 4. Pattern Summary

Both searches follow the same pattern — **record when you might have the answer, keep going to find a better one**:

| | Go this way when... | Record when... | Look for better in... |
|---|---|---|---|
| **Successor** | `curr <= key` → right | `curr > key` | left (smaller values) |
| **Predecessor** | `curr >= key` → left | `curr < key` | right (larger values) |
