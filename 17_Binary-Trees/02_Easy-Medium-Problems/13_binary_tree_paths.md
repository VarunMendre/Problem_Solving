# Binary Tree Paths (Root to Leaf)

---

## 1. Intuition / Approach

The goal is to return **all paths from root to every leaf node** as strings in the format `"1->2->5"`.

### Core Idea — DFS with string building

We do a simple **DFS (preorder)** — visit the node first, then recurse left and right. At each step, we **append the current node's value** to the running string `str`.

- If a **left or right child exists** → recurse deeper, appending `"->"` before the next value.
- If a node is a **leaf** (no left, no right) → the path is complete, push `str` into `answer`.

### Why pass `str` by value (not reference)?

```cpp
rootToLeaf(root->left, answer, str + "->");
rootToLeaf(root->right, answer, str + "->");
```

`str + "->"` creates a **new string** for each recursive call. This means the left and right branches get their own independent copies of the path — no backtracking needed.

If `str` were passed by reference, appending on the left branch would corrupt the path for the right branch.

### Dry Run

```
        1
       / \
      2   3
       \
        5
```

```
visit 1  → str = "1"
  visit 2  → str = "1->2"
    visit 5  → str = "1->2->5" → leaf → push "1->2->5"
  visit 3  → str = "1->3" → leaf → push "1->3"
```

Output: `["1->2->5", "1->3"]` ✅

---

## 2. Code

```cpp
class Solution {
private:
    void rootToLeaf(TreeNode* root, vector<string>& answer, string str) {
        // Append current node's value to the path
        str += to_string(root->val);

        // Recurse left — pass a new copy of str with "->" appended
        if (root->left)
            rootToLeaf(root->left, answer, str + "->");

        // Recurse right — pass a new copy of str with "->" appended
        if (root->right)
            rootToLeaf(root->right, answer, str + "->");

        // Leaf node → path is complete, store it
        if (!root->left && !root->right)
            answer.push_back(str);
    }

public:
    vector<string> binaryTreePaths(TreeNode* root) {
        vector<string> answer;
        if (!root || root == nullptr) {
            return answer;
        }
        rootToLeaf(root, answer, "");
        return answer;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node is visited exactly once |
| **Space (output)** | `O(N * H)` | Each path string is of length `O(H)`, and there can be `O(N)` leaf nodes |
| **Space (auxiliary)** | `O(H)` | Recursive call stack depth equals tree height |

> `N` = total number of nodes | `H` = height of the tree  
> `H` is `O(log N)` for a balanced tree and `O(N)` for a skewed tree
