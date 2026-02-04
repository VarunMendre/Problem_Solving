# 🧠 LFU Cache (Least Frequently Used)

---

## 📌 Problem Statement

Design a data structure that follows **LFU (Least Frequently Used)** eviction policy:

- Evict the key with the **lowest access frequency**.
- If multiple keys have the same frequency, evict the **Least Recently Used (LRU)** among them.
- Support `get()` and `put()` in **O(1)** time.

---

## 1️⃣ Intuition / Approach

LFU is more complex than LRU because we must track **frequency + recency**. To achieve **O(1)** operations, we use **three core components**.

---

### 🔹 1. Node (Key, Value, Frequency)

Each cache entry stores:

- `key`
- `value`
- `cnt` → number of times accessed

```
[key | value | freq]
```

---

### 🔹 2. Frequency → Doubly Linked List Map

We maintain **multiple DLLs**, one per frequency.

```
Frequency 1 → DLL (LRU order)
Frequency 2 → DLL (LRU order)
Frequency 3 → DLL (LRU order)
```

- Each DLL maintains **recency within same frequency**.
- Head → Most Recently Used
- Tail → Least Recently Used

---

### 🔹 3. Key → Node Hash Map

```
keyNodeMap[key] = Node*
```

- Direct access to nodes in **O(1)**

---

### 🔹 Important Variables

| Variable       | Purpose                           |
| -------------- | --------------------------------- |
| `minFreq`      | Tracks minimum frequency in cache |
| `currSize`     | Current cache size                |
| `maxCacheSize` | Maximum allowed capacity          |

---

## 🔁 Working Flow (Visual)

### ▶️ `get(key)`

```
key exists?
  ↓ yes
increase frequency
move node to next freq DLL
update minFreq if needed
```

---

### ▶️ `put(key, value)`

```
key exists?
  ↓ yes → update value + freq
  ↓ no
cache full?
  ↓ yes → evict (minFreq DLL, LRU node)
insert new node at freq = 1
set minFreq = 1
```

---

### 🧩 Why `minFreq` is Critical

- Always tells **which frequency list to evict from**.
- Ensures eviction is **O(1)** without scanning.

---

## 2️⃣ Code 

```cpp
#include <unordered_map>
using namespace std;

class Node {
public:
    int key, value, cnt;
    Node* prev;
    Node* next;

    Node(int _key, int _value) {
        key = _key;
        value = _value;
        cnt = 1;
        prev = next = nullptr;
    }
};

// DLL
struct List {
    int size;
    Node* head;
    Node* tail;

    List() {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head->next = tail;
        tail->prev = head;
        size = 0;
    }

    void addNode(Node* newNode) {
        Node* temp = head->next;

        newNode->next = temp;
        newNode->prev = head;

        head->next = newNode;
        temp->prev = newNode;

        size++;
    }

    void deleteNode(Node* delNode) {
        Node* prevNode = delNode->prev;
        Node* nextNode = delNode->next;

        prevNode->next = nextNode;
        nextNode->prev = prevNode;

        size--;
    }
};

class LFUCache {
private:
    unordered_map<int, Node*> keyNodeMap;
    unordered_map<int, List*> freqListMap;

    int maxCacheSize;
    int minFreq;
    int currSize;

public:
    LFUCache(int capacity) {
        maxCacheSize = capacity;
        minFreq = 0;
        currSize = 0;
    }

    void updateFreqListMap(Node* node) {
        int freq = node->cnt;

        // remove from current freq list
        freqListMap[freq]->deleteNode(node);

        if (freq == minFreq && freqListMap[freq]->size == 0) {
            minFreq++;
        }

        node->cnt++;

        List* nextList;
        if (freqListMap.find(node->cnt) != freqListMap.end()) {
            nextList = freqListMap[node->cnt];
        } else {
            nextList = new List();
            freqListMap[node->cnt] = nextList;
        }

        nextList->addNode(node);
        keyNodeMap[node->key] = node;
    }

    int get(int key) {
        if (keyNodeMap.find(key) == keyNodeMap.end()) {
            return -1;
        }

        Node* node = keyNodeMap[key];
        int val = node->value;

        updateFreqListMap(node);
        return val;
    }

    void put(int key, int value) {
        if (maxCacheSize == 0) return;

        if (keyNodeMap.find(key) != keyNodeMap.end()) {
            Node* node = keyNodeMap[key];
            node->value = value;
            updateFreqListMap(node);
            return;
        }

        if (currSize == maxCacheSize) {
            List* list = freqListMap[minFreq];
            Node* nodeToRemove = list->tail->prev;

            keyNodeMap.erase(nodeToRemove->key);
            list->deleteNode(nodeToRemove);

            currSize--;
        }

        currSize++;
        minFreq = 1;

        List* listFreq;
        if (freqListMap.find(minFreq) != freqListMap.end()) {
            listFreq = freqListMap[minFreq];
        } else {
            listFreq = new List();
            freqListMap[minFreq] = listFreq;
        }

        Node* node = new Node(key, value);
        listFreq->addNode(node);
        keyNodeMap[key] = node;
    }
};
```

---

## 3️⃣ Time & Space Complexity

### ⏱ Time Complexity

| Operation | Complexity |
| --------- | ---------- |
| `get()`   | **O(1)**   |
| `put()`   | **O(1)**   |

- Hash maps → constant access
- DLL operations → constant insertion/deletion

---

### 💾 Space Complexity

```
O(capacity)
```

- One node per cache entry
- Multiple frequency lists, total nodes ≤ capacity

---

## ✅ Final Takeaway

- **LFU = Frequency + Recency**
- `minFreq` avoids costly scans
- DLL per frequency guarantees LRU tie-breaking



