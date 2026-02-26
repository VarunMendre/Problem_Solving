# Introduction to Priority Queues using Binary Heaps

## What is a Priority Queue?

A Priority Queue is a special type of queue where each element is
assigned a priority, and the element with the highest priority is
processed first.

If two elements have the same priority, they are handled based on
insertion order.

### Real-Life Example

Think of a hospital emergency room: Patients are treated based on how
critical their condition is --- not based on arrival time.

### Applications

-   Task Scheduling
-   Dijkstra's Algorithm
-   Prim's Algorithm
-   Event-driven systems
-   Top K Problems

------------------------------------------------------------------------

# Binary Heap

A Binary Heap is a Binary Tree that satisfies:

1.  Complete Binary Tree Property
2.  Heap Property

------------------------------------------------------------------------

## Complete Binary Tree

-   All levels are completely filled
-   Except possibly the last level
-   The last level is filled from left to right

------------------------------------------------------------------------

## Heap Property

### Min Heap

-   Parent node is smaller than its children
-   Root contains the minimum element

### Max Heap

-   Parent node is greater than its children
-   Root contains the maximum element

------------------------------------------------------------------------

# Array Representation of Binary Heap

-   Root â†’ arr\[0\]

For any index i:

Left Child = 2*i + 1\
Right Child = 2*i + 2\
Parent = (i - 1) / 2

------------------------------------------------------------------------

# Operations in Min Heap

-   Insert()
-   Heapify()
-   getMin()
-   ExtractMin()
-   DecreaseKey()
-   Delete()

------------------------------------------------------------------------

## Insert()

1.  Insert at last vacant position
2.  Compare with parent
3.  Swap if parent is greater
4.  Repeat until heap property is restored

Time Complexity: O(log N)

------------------------------------------------------------------------

## Heapify()

1.  Find smallest among node and its children
2.  Swap with smallest
3.  Recursively call heapify

Time Complexity: O(log N)

------------------------------------------------------------------------

## getMin()

-   Return arr\[0\]

Time Complexity: O(1)

------------------------------------------------------------------------

## ExtractMin()

1.  Replace root with last element
2.  Reduce size
3.  Call heapify(0)

Time Complexity: O(log N)

------------------------------------------------------------------------

## DecreaseKey()

1.  Update value
2.  Swap upward if parent is greater

Time Complexity: O(log N)

------------------------------------------------------------------------

## Delete()

1.  Set value to INT_MIN
2.  Call DecreaseKey()
3.  Call ExtractMin()

Time Complexity: O(log N)

------------------------------------------------------------------------

# C++ Implementation of Min Heap

``` cpp
#include <bits/stdc++.h>
using namespace std;

class MinHeap {
    int *arr;
    int capacity;
    int size;

public:
    MinHeap(int cap) {
        capacity = cap;
        size = 0;
        arr = new int[cap];
    }

    int parent(int i) { return (i - 1) / 2; }
    int left(int i) { return 2 * i + 1; }
    int right(int i) { return 2 * i + 2; }

    void insert(int x) {
        if (size == capacity) {
            cout << "Overflow\n";
            return;
        }

        int i = size++;
        arr[i] = x;

        while (i != 0 && arr[parent(i)] > arr[i]) {
            swap(arr[i], arr[parent(i)]);
            i = parent(i);
        }
    }

    void heapify(int i) {
        int l = left(i);
        int r = right(i);
        int smallest = i;

        if (l < size && arr[l] < arr[smallest])
            smallest = l;

        if (r < size && arr[r] < arr[smallest])
            smallest = r;

        if (smallest != i) {
            swap(arr[i], arr[smallest]);
            heapify(smallest);
        }
    }

    int getMin() {
        if (size <= 0) return INT_MAX;
        return arr[0];
    }

    int extractMin() {
        if (size <= 0) return INT_MAX;
        if (size == 1) {
            size--;
            return arr[0];
        }

        int root = arr[0];
        arr[0] = arr[size - 1];
        size--;
        heapify(0);

        return root;
    }

    void decreaseKey(int i, int new_val) {
        arr[i] = new_val;
        while (i != 0 && arr[parent(i)] > arr[i]) {
            swap(arr[i], arr[parent(i)]);
            i = parent(i);
        }
    }

    void deleteKey(int i) {
        decreaseKey(i, INT_MIN);
        extractMin();
    }
};
```

------------------------------------------------------------------------

# Time Complexity Summary

  Operation       Time Complexity
  --------------- -----------------
  Insert()        O(log N)
  Heapify()       O(log N)
  getMin()        O(1)
  ExtractMin()    O(log N)
  DecreaseKey()   O(log N)
  Delete()        O(log N)

------------------------------------------------------------------------

# Final Notes

Binary Heap is the backbone of Priority Queue and is very important for
DSA interviews and competitive programming.