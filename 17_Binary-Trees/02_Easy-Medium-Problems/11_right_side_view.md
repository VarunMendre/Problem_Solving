# Right Side View of a Binary Tree

---

## 1. Intuition / Approach

The **right side view** returns the nodes visible when the tree is viewed from the **right side** — i.e., the **rightmost node at each level**.

### Key Insight — Right Before Left DFS

Instead of BFS (level-by-level), we use a **DFS that always visits the right child first**.

The trick lies in this condition:
```cpp
if (level == ans.size())
    ans.push_back(root->val);
```

- `ans.size()` tells us how many levels have already been recorded.
- `level == ans.size()` is only `true` the **first time we visit a new level**.
- Since we go **right before left**, the first node we encounter at any level is always the **rightmost node**.

### Dry Run Example

```
        1          level 0
       / \
      2   3        level 1
       \   \
        5   4      level 2
```

DFS (right first):
```
visit 1  → level=0, ans=[]   → 0 == 0 ✅ → ans=[1]
visit 3  → level=1, ans=[1]  → 1 == 1 ✅ → ans=[1,3]
visit 4  → level=2, ans=[1,3]→ 2 == 2 ✅ → ans=[1,3,4]
visit 2  → level=1, ans=[1,3,4] → 1 != 3 ❌ skip
visit 5  → level=2, ans=[1,3,4] → 2 != 3 ❌ skip
```

Output: `[1, 3, 4]` ✅

---

### Why this works cleanly

- No need to track levels explicitly in a map or queue.
- `ans.size()` acts as a **level counter** — it grows by 1 each time a new level is first reached.
- Visiting right before left **guarantees** the first node at each level is the rightmost visible one.

---

## 2. Code

```cpp
class Solution {
public:
    void customTraversal(TreeNode* root, int level, vector<int>& ans) {
        if (root == NULL)
            return;

        // First time reaching this level → rightmost node due to right-first DFS
        if (level == ans.size())
            ans.push_back(root->val);

        // Visit right child first to prioritize rightmost nodes
        customTraversal(root->right, level + 1, ans);

        // Visit left child — will only add to ans if its level hasn't been seen yet
        customTraversal(root->left, level + 1, ans);
    }

    vector<int> rightSideView(TreeNode* root) {
        vector<int> ans;
        customTraversal(root, 0, ans);
        return ans;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space (output)** | `O(N)` | Result vector stores one node per level — at most `N` levels in a skewed tree |
| **Space (auxiliary)** | `O(H)` | Recursive call stack depth equals the height of the tree |

> `N` = total number of nodes | `H` = height of the tree  
> `H` is `O(log N)` for a balanced tree and `O(N)` for a skewed tree
