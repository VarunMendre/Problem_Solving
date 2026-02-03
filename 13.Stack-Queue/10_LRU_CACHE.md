# 🧠 LRU Cache (Least Recently Used)

---

## 📌 Problem Statement
Design a data structure that follows **LRU (Least Recently Used)** eviction policy:
- If the cache exceeds its capacity, remove the **least recently used** item.
- Support `get()` and `put()` operations in **O(1)** time.

---

## 1️⃣ Intuition / Approach

To achieve **O(1)** operations, we combine **two data structures**:

### 🔹 1. Doubly Linked List
- Maintains the **order of usage**.
- **Most Recently Used (MRU)** → placed near the `head`.
- **Least Recently Used (LRU)** → placed near the `tail`.

```
HEAD <-> [MRU] <-> ... <-> [LRU] <-> TAIL
```

### 🔹 2. Hash Map (`unordered_map`)
- Maps `key → Node*`.
- Allows **direct access** to nodes in O(1).

---

### 🔁 Core Operations (Visual Model)

#### ▶️ `get(key)`
```
Access key
   ↓
Move node to front (MRU)
```

#### ▶️ `put(key, value)`
```
If key exists → delete old node
If capacity full → remove LRU (tail->prev)
Insert new node at front (MRU)
```

---

### 🧩 Why Dummy Head & Tail?
- Simplifies insertion & deletion.
- No need to handle edge cases like empty list.

```
HEAD <-> node1 <-> node2 <-> TAIL
```

---

## 2️⃣ Code (As It Is)

```cpp
class LRUCache {
public:
    class Node {
    public:
        int key;
        int val;
        Node* prev;
        Node* next;

        Node(int _key, int _val) {
            key = _key;
            val = _val;
        }
    };

    Node* head = new Node(-1, -1);
    Node* tail = new Node(-1, -1);

    int cap;

    unordered_map<int, Node*> mpp;
    LRUCache(int capacity) {
        cap = capacity;
        head->next = tail;
        tail->prev = head;
    }   

    void addNode(Node *newNode) {
        Node* temp = head->next;
        newNode->next = temp;
        newNode->prev = head;
        head->next = newNode;
        temp->prev = newNode;
    }

    void deleteNode(Node *delNode) {
        Node* delPrev = delNode->prev;
        Node* delNext = delNode->next;

        delPrev->next = delNext;
        delNext->prev = delPrev;

        /*delNode->next = null;
        delNode->prev = null;
        delete(delNode);*/
    }

    int get(int key) {
        if(mpp.find(key) != mpp.end()) {
            Node* resNode = mpp[key];
            int res = resNode->val;

            mpp.erase(key);

            deleteNode(resNode);
            addNode(resNode);

            mpp[key] = head->next;
            return res;
        }
        return -1;
    }

    void put(int key, int value) {
        if(mpp.find(key) != mpp.end()) {
            Node* existingNode = mpp[key];
            mpp.erase(key);
            deleteNode(existingNode);
        }

        if(mpp.size() == cap) {
            mpp.erase(tail->prev->key);
            deleteNode(tail->prev);
        }

        addNode(new Node(key, value));
        mpp[key] = head->next;
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

---

## 3️⃣ Time & Space Complexity

### ⏱ Time Complexity
| Operation | Complexity |
|---------|------------|
| `get()` | **O(1)** |
| `put()` | **O(1)** |

All operations are constant time due to:
- Hash map lookup
- Constant-time linked list insertion & deletion

---

### 💾 Space Complexity

```
O(capacity)
```

- Hash map stores up to `capacity` nodes
- Doubly linked list stores same nodes

---

## ✅ Final Takeaway
- **Hash Map** → fast access
- **Doubly Linked List** → tracks usage order
- Dummy nodes make implementation clean & bug-free

This pattern is **interview gold** 🚀 and shows strong data-structure design skills.

