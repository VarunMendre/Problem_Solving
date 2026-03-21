# Bottom View of a Binary Tree

---

## 1. Intuition / Approach

The **bottom view** of a binary tree is the set of nodes visible when the tree is viewed from **directly below**. For each vertical column, only the **last node encountered** (bottommost) is visible.

### How it differs from Top View

The logic is **identical to Top View** with just one change:

| | Top View | Bottom View |
|---|---|---|
| Which node to keep? | First seen (topmost) | Last seen (bottommost) |
| Map insert condition | Only if key **not present** | **Always overwrite** |

In Top View we had:
```cpp
if(mpp.find(vertical) == mpp.end()) {   // insert only once
    mpp[vertical] = node->data;
}
```

In Bottom View we simply do:
```cpp
mpp[vertical] = node->data;   // always overwrite
```

Since BFS processes nodes **top to bottom**, by the time the loop ends, the map holds the **last (bottommost) node** for every vertical column.

---

## 2. Code

```cpp
class Solution {
  public:
    vector<int> bottomView(Node *root) {
        vector<int> ans;
        if(root == nullptr) return ans;

        // vertical column → bottommost node's value
        map<int, int> mpp;

        // BFS queue: {node, vertical}
        queue<pair<Node*, int>> todo;
        todo.push({root, 0});

        while(!todo.empty()) {
            auto it = todo.front();
            todo.pop();

            Node* node = it.first;
            int vertical = it.second;

            // Always overwrite — BFS goes top to bottom,
            // so the last write for any vertical is always the bottommost node
            mpp[vertical] = node->data;

            // Left child: column - 1
            if(node->left) {
                todo.push({node->left, vertical - 1});
            }

            // Right child: column + 1
            if(node->right) {
                todo.push({node->right, vertical + 1});
            }
        }

        // map is sorted by key (vertical), so result is left to right
        for(auto it : mpp) {
            ans.push_back(it.second);
        }

        return ans;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Each node processed once in BFS — `O(N)`, each `map` insertion costs `O(log N)` |
| **Space** | `O(N)` | Map stores at most `N` entries; BFS queue holds at most one level at a time |

> `N` = total number of nodes in the tree
