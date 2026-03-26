# Maximum Width of Binary Tree

---

## 1. Intuition / Approach

The **width** of a level is defined as the distance between the **leftmost and rightmost non-null nodes**, including any null nodes in between.

### Indexing Trick

Imagine the tree as a **complete binary tree** and assign indices to every node like a heap array:

```
        1           index = 0
       / \
      2   3         index = 1, 2
     / \ / \
    4  5 6  7       index = 3, 4, 5, 6
```

For a node at index `i`:
- Left child → `2*i + 1`
- Right child → `2*i + 2`

Width at any level = `last index - first index + 1`

---

### The Overflow Problem & The Fix

As we go deeper, indices grow **exponentially** (`2^depth`). Even `long long` can overflow for deep trees.

**Fix:** At the start of each level, **normalize all indices** by subtracting the minimum index of that level:

```cpp
long long currInd = todos.front().second - minInd;
```

This resets the leftmost node's index to `0` at every level, keeping numbers small while **preserving relative gaps** between nodes — so width calculation stays correct.

---

### BFS Level by Level

We use **BFS** so we can process one complete level at a time:
- Track `first` index (when `i == 0`)
- Track `last` index (when `i == size - 1`)
- Width = `last - first + 1`
- Update global `ans` with the max width seen

---

### Dry Run

```
        1           index 0
       / \
      3   2         index 1, 2
     /     \
    5       9       index 3, 6  (null nodes at 4,5 counted in width)
```

Level 0: first=0, last=0 → width = 1
Level 1: minInd=1, first=0, last=1 → width = 2
Level 2: minInd=3, indices after normalization → first=0, last=3 → width = 4

---

## 2. Code

```cpp
class Solution {
public:
    int widthOfBinaryTree(TreeNode* root) {
        if(!root) return 0;

        int ans = 0;

        // Queue stores {node, index} — index based on heap array numbering
        queue<pair<TreeNode*, long long>> todos;
        todos.push({root, 0});

        while(!todos.empty()) {
            int size = todos.size();

            // Normalize: subtract min index of this level to prevent overflow
            long long minInd = todos.front().second;
            long long first, last;

            for(int i = 0; i < size; i++) {
                // Normalized index for this node
                long long currInd = todos.front().second - minInd;

                TreeNode* node = todos.front().first;
                todos.pop();

                // Track first and last index of this level
                if(i == 0)        first = currInd;
                if(i == size - 1) last  = currInd;

                // Push children with heap-style indices
                if(node->left)
                    todos.push({node->left,  currInd * 2 + 1});
                if(node->right)
                    todos.push({node->right, currInd * 2 + 2});
            }

            // Width of this level
            ans = max(ans, (int)(last - first + 1));
        }

        return ans;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once during BFS |
| **Space** | `O(N)` | BFS queue holds at most one full level — up to `N/2` nodes at the last level of a complete tree |

> `N` = total number of nodes in the tree

---

## 4. Why `long long` even after normalization?

Even after subtracting `minInd` at each level, the **relative gap** between leftmost and rightmost nodes can still be very large (up to `2^depth`). For a tree of depth 32+, this exceeds `int` range — so `long long` is still necessary for the index values, even though the normalization prevents absolute overflow.
