# Serialize and Deserialize a Binary Tree

---

## 1. Intuition / Approach

**Serialization** = converting a tree into a string (so it can be stored or transmitted)  
**Deserialization** = reconstructing the original tree from that string

### Why BFS (Level Order)?

We use **BFS** because it naturally produces a level-by-level string that's easy to reconstruct. NULL children are encoded as `#` so we know exactly where a subtree ends — no ambiguity.

---

### Serialize — BFS with NULL markers

Do a standard BFS. For each node:
- If the node is **not NULL** → append `val,` and push both children (even if NULL)
- If the node is **NULL** → append `#,` and push nothing

```
        1
       / \
      2   3
         / \
        4   5

Serialized: "1,2,3,#,#,4,5,#,#,#,#,"
```

The `#` markers preserve structural information — they tell us exactly where left/right children are missing.

---

### Deserialize — Rebuild using the same BFS order

1. Read the first token → create `root`
2. Use a queue of nodes **waiting for their children to be assigned**
3. For each node in the queue:
   - Read next token → assign as **left child** (or NULL if `#`)
   - Read next token → assign as **right child** (or NULL if `#`)
   - Push non-NULL children into the queue for future processing

The BFS order of serialization guarantees that tokens arrive in the exact order we need them during deserialization.

---

### `stringstream` + `getline` for parsing

```cpp
stringstream s(data);
string str;
getline(s, str, ',');   // reads next token split by ','
```

This cleanly splits the serialized string by commas one token at a time — no manual index tracking needed.

---

### Dry Run

```
Tree:       1
           / \
          2   3

serialize:
  queue: [1]
  pop 1 → s = "1,"  → push left(2), right(3)
  pop 2 → s = "1,2," → push left(NULL), right(NULL)
  pop 3 → s = "1,2,3," → push left(NULL), right(NULL)
  pop NULL → s = "1,2,3,#,"
  pop NULL → s = "1,2,3,#,#,"
  pop NULL → s = "1,2,3,#,#,#,"
  pop NULL → s = "1,2,3,#,#,#,#,"

deserialize("1,2,3,#,#,#,#,"):
  read "1" → root = node(1), queue = [1]
  pop 1:
    read "2" → left = node(2), queue = [2]
    read "3" → right = node(3), queue = [2, 3]
  pop 2:
    read "#" → left = NULL
    read "#" → right = NULL
  pop 3:
    read "#" → left = NULL
    read "#" → right = NULL

Tree reconstructed ✅
```

---

## 2. Code

```cpp
class Codec {
public:
    // Encodes a tree to a single string using BFS
    string serialize(TreeNode* root) {
        if(root == NULL) return "";

        queue<TreeNode*> q;
        q.push(root);
        string s = "";

        while(!q.empty()) {
            auto node = q.front();
            q.pop();

            if(node == NULL) {
                s.append("#,");             // NULL marker
            } else {
                s.append(to_string(node->val) + ',');
                q.push(node->left);         // push even if NULL — handled in next iteration
                q.push(node->right);
            }
        }

        return s;
    }

    // Decodes the string back to the original tree using BFS
    TreeNode* deserialize(string data) {
        if(data.size() == 0) return NULL;

        stringstream s(data);
        string str;

        // First token is always the root
        getline(s, str, ',');
        TreeNode* root = new TreeNode(stoi(str));

        queue<TreeNode*> q;
        q.push(root);

        while(!q.empty()) {
            auto node = q.front();
            q.pop();

            // Read left child token
            getline(s, str, ',');
            if(str == "#") {
                node->left = NULL;
            } else {
                TreeNode* leftNode = new TreeNode(stoi(str));
                node->left = leftNode;
                q.push(leftNode);           // will need its own children assigned later
            }

            // Read right child token
            getline(s, str, ',');
            if(str == "#") {
                node->right = NULL;
            } else {
                TreeNode* rightNode = new TreeNode(stoi(str));
                node->right = rightNode;
                q.push(rightNode);
            }
        }

        return root;
    }
};
```

---

## 3. Time & Space Complexity

### Serialize

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Every node visited exactly once in BFS |
| **Space** | `O(N)` | Queue holds at most one level (up to `N/2` nodes); output string has `O(N)` tokens |

### Deserialize

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Each token parsed once; each node created once |
| **Space** | `O(N)` | Queue holds at most one level at a time; `N` nodes created |

> `N` = total number of nodes in the tree
