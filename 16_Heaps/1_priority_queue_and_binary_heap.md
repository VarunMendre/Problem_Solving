# Priority Queue & Binary Heap Notes

---

## What is a Priority Queue?

A **Priority Queue** is a special type of queue where each element is assigned a priority. Instead of being processed in the order they arrive (like a normal queue - FIFO), the element with the **highest priority** is processed first.

If two elements have the same priority, they are handled based on their insertion order.

### Real-Life Example
Think of an emergency room in a hospital:
- Patients are not treated based only on arrival time.
- A patient with a heart attack is treated before someone with a mild cold.

### Applications
- Task Scheduling
- Pathfinding Algorithms (Dijkstra’s Algorithm)
- Real-Time Systems
- CPU Scheduling

---

# Binary Heap

A **Binary Heap** is a Binary Tree that satisfies:

1. It must be a **Complete Binary Tree**.
2. It must satisfy the **Heap Property**.

---

## Complete Binary Tree

A binary tree in which:
- All levels are completely filled except possibly the last level.
- The last level is filled from left to right.

This ensures efficient array representation.

---

## Heap Property

A Binary Heap can be:

### 1️⃣ Min Heap
For every node:

```
Parent <= Left Child
Parent <= Right Child
```

- Root contains the **minimum** element.

### 2️⃣ Max Heap
For every node:

```
Parent >= Left Child
Parent >= Right Child
```

- Root contains the **maximum** element.

---

# Representation of Binary Heap (Array Representation)

A Binary Heap is represented using an array.

- Root → `arr[0]`

### Index Relationships

| Node Index (i) | Left Child | Right Child |
|---------------|------------|-------------|
| i             | 2*i + 1    | 2*i + 2     |

### Parent Index

| Child Index (i) | Parent Index |
|------------------|--------------|
| i                | (i - 1) / 2  |

---

# Operations in Min Heap

- Insert()
- Heapify()
- getMin()
- ExtractMin()
- DecreaseKey()
- Delete()

> Note: A Binary Heap must always maintain:
> - Complete Binary Tree property
> - Heap Property (Min/Max)

---

## 1️⃣ Insert(x)

### Steps:
1. Insert the element at the first vacant position (last level, leftmost).
2. Complete Binary Tree property is maintained automatically.
3. Compare with parent.
4. If parent > child → Swap.
5. Repeat until heap property is restored.

### Time Complexity
```
O(log N)
```

---

## 2️⃣ Heapify(i)

Used when heap property is violated at index `i`.

### Steps:
1. Find smallest among:
   - Current node
   - Left child
   - Right child
2. If smallest is not the current node:
   - Swap
   - Recursively call Heapify()
3. Stop when property is satisfied.

### Time Complexity
```
O(log N)
```

---

## 3️⃣ getMin()

- Returns `arr[0]`
- Root always contains minimum element.

### Time Complexity
```
O(1)
```

---

## 4️⃣ ExtractMin()

Removes minimum element.

### Steps:
1. Store root value.
2. Replace root with last element.
3. Reduce size by 1.
4. Call Heapify(0).

### Time Complexity
```
O(log N)
```

---

## 5️⃣ DecreaseKey(i, new_val)

### Steps:
1. Update `arr[i] = new_val`.
2. Compare with parent.
3. If parent > current → Swap.
4. Repeat until heap property is satisfied.

### Time Complexity
```
O(log N)
```

---

## 6️⃣ Delete(i)

### Steps:
1. Replace value at index `i` with `INT_MIN`.
2. Call `DecreaseKey(i, INT_MIN)`.
3. Call `ExtractMin()`.

### Time Complexity
```
O(log N)
```

---

# C++ Implementation (Min Heap)

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

    int parent(int i) { return (i - 1) / 2; }
    int left(int i) { return 2 * i + 1; }
    int right(int i) { return 2 * i + 2; }

    void Insert(int x) {
        if (size == capacity) {
            cout << "Heap Overflow" << endl;
            return;
        }

        arr[size] = x;
        int k = size;
        size++;

        while (k != 0 && arr[parent(k)] > arr[k]) {
            swap(arr[parent(k)], arr[k]);
            k = parent(k);
        }
    }

    void Heapify(int ind) {
        int li = left(ind);
        int ri = right(ind);
        int smallest = ind;

        if (li < size && arr[li] < arr[smallest])
            smallest = li;

        if (ri < size && arr[ri] < arr[smallest])
            smallest = ri;

        if (smallest != ind) {
            swap(arr[ind], arr[smallest]);
            Heapify(smallest);
        }
    }

    int getMin() {
        return arr[0];
    }

    int ExtractMin() {
        if (size <= 0) return INT_MAX;

        if (size == 1) {
            size--;
            return arr[0];
        }

        int root = arr[0];
        arr[0] = arr[size - 1];
        size--;
        Heapify(0);
        return root;
    }

    void DecreaseKey(int i, int val) {
        arr[i] = val;
        while (i != 0 && arr[parent(i)] > arr[i]) {
            swap(arr[parent(i)], arr[i]);
            i = parent(i);
        }
    }

    void Delete(int i) {
        DecreaseKey(i, INT_MIN);
        ExtractMin();
    }
};
```

---

# Complexity Summary

| Operation     | Time Complexity |
|--------------|-----------------|
| Insert       | O(log N)        |
| Heapify      | O(log N)        |
| getMin       | O(1)            |
| ExtractMin   | O(log N)        |
| DecreaseKey  | O(log N)        |
| Delete       | O(log N)        |

---

# Final Notes

- Priority Queue is commonly implemented using a Binary Heap.
- Min Heap → Used when smallest element priority matters.
- Max Heap → Used when largest element priority matters.
- Heap ensures efficient insertion and deletion compared to sorted arrays.

---

These notes are GitHub-ready and structured for revision before interviews or DSA practice.

