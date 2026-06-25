# Priority Queues using Binary Heaps — Revision Notes

---

## 1. What is a Priority Queue?

A **Priority Queue** is an abstract data type similar to a regular queue or stack, but where **each element has a priority associated with it**. Elements are served based on their priority rather than their insertion order.

Key characteristics:
- The element with the **highest (or lowest) priority** is dequeued first.
- If two elements share the same priority, they are served in insertion order.
- Commonly implemented using a **Binary Heap** for efficiency.

**Use cases:** Dijkstra's shortest path, Huffman coding, CPU scheduling, A* algorithm.

---

## 2. Binary Heap

### 2.1 Complete Binary Tree

A **Complete Binary Tree** is a binary tree where:
- All levels are completely filled **except possibly the last level**.
- The last level has all nodes as **far left as possible**.

```
        1
      /   \
     2     3
    / \   /
   4   5 6
```
This is a valid complete binary tree.

---

### 2.2 Heap Property

There are two types of heaps:

| Type | Property |
|------|----------|
| **Min Heap** | Parent node ≤ both children → root is the **minimum** element |
| **Max Heap** | Parent node ≥ both children → root is the **maximum** element |

**Min Heap example:**
```
        1
      /   \
     3     2
    / \   / \
   6   5 4   8
```
Every parent is smaller than or equal to its children. ✅

---

## 3. Representation of Binary Heap

A binary heap is efficiently stored as a **1D array** (0-indexed):

```
Index:  0   1   2   3   4   5   6
Array: [1,  3,  2,  6,  5,  4,  8]
```

**Index relationships for node at index `i`:**

| Relation | Formula |
|----------|---------|
| Parent   | `(i - 1) / 2` |
| Left Child | `2*i + 1` |
| Right Child | `2*i + 2` |

This array representation avoids pointer overhead and gives O(1) access to parent/children.

---

## 4. Operations on Min Heap

### 4.1 `insert(x)` — Insert an element

**Idea:** Add the new element at the end of the array and "bubble up" (sift up) until the heap property is restored.

**Steps:**
1. Place the new element at `arr[size]`.
2. Increment size.
3. Compare with parent; if smaller, swap.
4. Repeat until root or parent ≤ current node.

**Example:** Insert `1` into `[4]`
```
Before: [4]
Place:  [4, 1]   → 1 < parent(4), swap
After:  [1, 4]   ✅
```

```cpp
void insert(int x) {
    if (size == capacity) { cout << "Overflow"; return; }
    arr[size] = x;
    int k = size;
    size++;
    while (k != 0 && arr[parent(k)] > arr[k]) {
        swap(&arr[parent(k)], &arr[k]);
        k = parent(k);
    }
}
```

---

### 4.2 `heapify(ind)` — Restore Heap Property (Top-Down)

**Idea:** Given a node that may violate the heap property, push it **down** to its correct position by comparing with children.

**Steps:**
1. Find the smallest among `arr[ind]`, left child, right child.
2. If smallest is not `ind`, swap and recursively heapify at the swapped index.

```cpp
void heapify(int ind) {
    int li = left(ind), ri = right(ind), smallest = ind;
    if (li < size && arr[li] < arr[smallest]) smallest = li;
    if (ri < size && arr[ri] < arr[smallest]) smallest = ri;
    if (smallest != ind) {
        swap(&arr[ind], &arr[smallest]);
        heapify(smallest);
    }
}
```

---

### 4.3 `getMin()` — Get Minimum Element

**Idea:** In a Min Heap, the root (`arr[0]`) is always the minimum.

```cpp
int getMin() {
    if (size <= 0) return INT_MIN;
    return arr[0];   // Always O(1)
}
```

---

### 4.4 `extractMin()` — Remove and Return Minimum

**Idea:** Remove the root (minimum), replace it with the last element, reduce size, and heapify down from root.

**Steps:**
1. Store `arr[0]` (the minimum).
2. Move `arr[size-1]` to `arr[0]`.
3. Decrease size.
4. Call `heapify(0)` to restore heap property.
5. Return the stored minimum.

```
Before: [1, 3, 2, 6, 5, 4, 8]
Step 1: store min = 1
Step 2: [8, 3, 2, 6, 5, 4]    ← last element to root
Step 3: heapify(0)
After:  [2, 3, 4, 6, 5, 8]    ✅
```

```cpp
int extractMin() {
    if (size <= 0) return INT_MIN;
    if (size == 1) { size--; return arr[0]; }
    int mini = arr[0];
    arr[0] = arr[size - 1];
    size--;
    heapify(0);
    return mini;
}
```

---

### 4.5 `decreaseKey(i, val)` — Decrease Value at Index `i`

**Idea:** Decrease the value at index `i` to `val`, then bubble it up since a smaller value may violate the heap property with its parent.

**Steps:**
1. Set `arr[i] = val`.
2. Bubble up: while `arr[i] < arr[parent(i)]`, swap and move up.

```cpp
void decreaseKey(int i, int val) {
    if (i >= size) return;
    arr[i] = val;
    while (i != 0 && arr[parent(i)] > arr[i]) {
        swap(&arr[i], &arr[parent(i)]);
        i = parent(i);
    }
}
```

---

### 4.6 `deleteKey(i)` — Delete Element at Index `i`

**Idea:** Use a trick — decrease the key to `INT_MIN` (making it the smallest), then extract the minimum.

**Steps:**
1. Call `decreaseKey(i, INT_MIN)` → element bubbles to root.
2. Call `extractMin()` → removes the root.

```cpp
void deleteKey(int i) {
    if (i >= size) return;
    decreaseKey(i, INT_MIN);
    extractMin();
}
```

---

## 5. Time Complexities

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `insert()` | **O(log N)** | Bubble up traverses height of tree = log N |
| `heapify()` | **O(log N)** | Sift down traverses height of tree = log N |
| `getMin()` | **O(1)** | Root element is always the minimum |
| `extractMin()` | **O(log N)** | Replace root + heapify = O(log N) |
| `decreaseKey()` | **O(log N)** | Bubble up traverses height of tree = log N |
| `delete()` | **O(log N)** | decreaseKey + extractMin = O(log N) + O(log N) |

> **Why log N?** A complete binary tree with N nodes has height **⌊log₂N⌋**. All sift-up and sift-down operations traverse at most this many levels.

---

## 6. Full C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

class BinaryHeap {
public:
    int capacity;
    int size;
    int* arr;

    BinaryHeap(int cap) {
        capacity = cap;
        size = 0;
        arr = new int[capacity];
    }

    ~BinaryHeap() {
        delete[] arr;
    }

    // Index helpers
    int parent(int i) { return (i - 1) / 2; }
    int left(int i)   { return 2 * i + 1; }
    int right(int i)  { return 2 * i + 2; }

    void swap(int* x, int* y) {
        int temp = *x; *x = *y; *y = temp;
    }

    // Insert: add at end, bubble up
    void insert(int x) {
        if (size == capacity) { cout << "Binary Heap Overflow" << endl; return; }
        arr[size] = x;
        int k = size;
        size++;
        while (k != 0 && arr[parent(k)] > arr[k]) {
            swap(&arr[parent(k)], &arr[k]);
            k = parent(k);
        }
    }

    // Heapify: sift down from index ind
    void heapify(int ind) {
        int li = left(ind), ri = right(ind), smallest = ind;
        if (li < size && arr[li] < arr[smallest]) smallest = li;
        if (ri < size && arr[ri] < arr[smallest]) smallest = ri;
        if (smallest != ind) {
            swap(&arr[ind], &arr[smallest]);
            heapify(smallest);
        }
    }

    // Get minimum (root)
    int getMin() {
        if (size <= 0) return INT_MIN;
        return arr[0];
    }

    // Extract minimum: remove root, restore heap
    int extractMin() {
        if (size <= 0) return INT_MIN;
        if (size == 1) { size--; return arr[0]; }
        int mini = arr[0];
        arr[0] = arr[size - 1];
        size--;
        heapify(0);
        return mini;
    }

    // Decrease key at index i to val, then bubble up
    void decreaseKey(int i, int val) {
        if (i >= size) return;
        arr[i] = val;
        while (i != 0 && arr[parent(i)] > arr[i]) {
            swap(&arr[i], &arr[parent(i)]);
            i = parent(i);
        }
    }

    // Delete key at index i using decreaseKey trick
    void deleteKey(int i) {
        if (i >= size) return;
        decreaseKey(i, INT_MIN);
        extractMin();
    }

    // Print heap array
    void print() {
        for (int i = 0; i < size; i++) cout << arr[i] << " ";
        cout << endl;
    }
};

int main() {
    BinaryHeap h(20);

    // Insert elements: 4, 1, 2, 6, 7, 3, 8, 5
    h.insert(4); h.insert(1); h.insert(2); h.insert(6);
    h.insert(7); h.insert(3); h.insert(8); h.insert(5);

    cout << "Min value is " << h.getMin() << endl;   // 1

    h.insert(-1);
    cout << "Min value is " << h.getMin() << endl;   // -1

    h.decreaseKey(3, -2);
    cout << "Min value is " << h.getMin() << endl;   // -2

    h.extractMin();
    cout << "Min value is " << h.getMin() << endl;   // -1

    h.deleteKey(0);
    cout << "Min value is " << h.getMin() << endl;   // next min

    h.print();
    return 0;
}
```

---

### Step-by-Step Trace of `main()`

| Step | Operation | Heap State (array) | Min |
|------|-----------|--------------------|-----|
| 1–8 | Insert 4,1,2,6,7,3,8,5 | `[1, 4, 2, 6, 7, 3, 8, 5]` | 1 |
| 9 | Insert -1 | `[-1, 1, 2, 6, 4, 3, 8, 5, 7]` | -1 |
| 10 | decreaseKey(3, -2) | `[-2, -1, 2, 1, 4, 3, 8, 5, 7, 6]` | -2 |
| 11 | extractMin() | removes -2 | -1 |
| 12 | deleteKey(0) | removes -1 | next smallest |

---

## 7. Quick Revision Summary

```
Priority Queue → serves by priority, not insertion order
Binary Heap   → complete binary tree + heap property
Min Heap      → parent ≤ children, root = minimum
Array rep     → parent(i) = (i-1)/2, left = 2i+1, right = 2i+2

insert()      → add at end, bubble UP        → O(log N)
heapify()     → compare with children, push DOWN → O(log N)
getMin()      → return arr[0]                → O(1)
extractMin()  → remove root, last→root, heapify → O(log N)
decreaseKey() → update value, bubble UP      → O(log N)
deleteKey()   → decreaseKey(INT_MIN) + extractMin → O(log N)
```

---

*Last revised: June 2026*
