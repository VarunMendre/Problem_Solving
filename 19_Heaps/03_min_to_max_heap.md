# Convert Min Heap to Max Heap

---

## 1. Brute Force — Sort in Descending Order

### Approach

A key observation: if an array is **sorted in descending order**, it automatically satisfies the max-heap property.

Why? In a descending sorted array, for every index `i`:
```
arr[i] >= arr[2*i+1]   (left child)
arr[i] >= arr[2*i+2]   (right child)
```

This holds simply because every element to the right (higher index) is smaller or equal. So **sorting descending = valid max heap array representation**.

```cpp
vector<int> MinToMaxHeap(int n, vector<int>& arr) {
    sort(arr.rbegin(), arr.rend());   // sort descending
    return arr;
}
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Sorting the array dominates |
| **Space** | `O(1)` | In-place sort, no extra structures |

> Works correctly but **discards the existing heap structure entirely** — it doesn't use the fact that we already have a (min) heap to begin with.

---

---

## 2. Optimal — Heapify (Build Max Heap)

### Approach

We don't need to sort. Instead, treat the input array as a tree and **build a max heap from scratch** using the classic **heapify** technique — same approach used in Heap Sort's build phase.

### Core Idea — `heapifyUpward` (sift-down)

For a node at index `i`, ensure it's larger than both its children. If not, swap with the larger child and recursively fix the subtree that was disturbed.

```cpp
void heapifyUpward(vector<int>& arr, int i, int n) {
    int li = 2*i + 1, ri = 2*i + 2;
    int largest = i;

    if(li < n && arr[li] > arr[i])      largest = li;
    if(ri < n && arr[ri] > arr[largest]) largest = ri;

    if(largest != i) {
        swap(arr[i], arr[largest]);
        heapifyUpward(arr, largest, n);  // fix the subtree we just disturbed
    }
}
```

> Despite the name `heapifyUpward`, this is actually a **sift-down** operation — it pushes a potentially-too-small value down the tree until heap property is restored. (Naming is a bit misleading, but the logic is correct.)

---

### Why Start from `(n-2)/2` and Go Backward?

`(n-2)/2` is the **last non-leaf node** (0-indexed). We only need to heapify non-leaf nodes — leaves are trivially valid heaps by themselves (no children to compare).

**Why backward (from last non-leaf to root)?**

We must fix **subtrees before their parents**. If we heapified top-down, a parent might be fixed using children that haven't been heapified yet — leading to incorrect results. Bottom-up guarantees that by the time we heapify node `i`, both its subtrees are already valid max heaps.

```cpp
for(int i = (n-2)/2; i >= 0; i--) {
    heapifyUpward(arr, i, n);
}
```

---

### Dry Run

```
Min Heap: [1, 3, 6, 5, 9, 8]

        1
       / \
      3   6
     / \  /
    5  9 8

n = 6, last non-leaf = (6-2)/2 = 2

i=2: node=6, li=5(8) → 8>6 → swap → [1,3,8,5,9,6]
     heapify(largest=5): leaf, no children → stop

i=1: node=3, li=3(5), ri=4(9) → largest=4(9)
     swap(3,9) → [1,9,8,5,3,6]
     heapify(4): leaf → stop

i=0: node=1, li=1(9), ri=2(8) → largest=1(9)
     swap(1,9) → [9,1,8,5,3,6]
     heapify(1): node=1, li=3(5), ri=4(3) → largest=3(5)
       swap(1,5) → [9,5,8,1,3,6]
       heapify(3): leaf → stop

Final: [9, 5, 8, 1, 3, 6]

Verify max heap:
        9
       / \
      5   8
     / \  /
    1  3 6

9>5 ✅, 9>8 ✅, 5>1 ✅, 5>3 ✅, 8>6 ✅  → Valid max heap ✅
```

---

## 2. Code

```cpp
// Sift-down: fix the subtree rooted at index i
void heapifyUpward(vector<int>& arr, int i, int n) {
    int li = 2 * i + 1;
    int ri = 2 * i + 2;
    int largest = i;

    // Find the largest among node, left child, right child
    if(li < n && arr[li] > arr[i])
        largest = li;
    if(ri < n && arr[ri] > arr[largest])
        largest = ri;

    // If a child is larger, swap and continue fixing downward
    if(largest != i) {
        swap(arr[i], arr[largest]);
        heapifyUpward(arr, largest, n);   // recursively fix disturbed subtree
    }
}

vector<int> MinToMaxHeap(int n, vector<int>& arr) {
    // Start from last non-leaf node, go up to root
    for(int i = (n - 2) / 2; i >= 0; i--) {
        heapifyUpward(arr, i, n);
    }

    return arr;
}
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Building a heap from an unordered array is `O(N)`, **not** `O(N log N)` — even though each `heapifyUpward` call is `O(log N)`, most nodes are near the bottom and require very little work; the math works out to a tight `O(N)` bound |
| **Space** | `O(log N)` auxiliary | Recursive call stack depth = height of heap = `O(log N)` |

---

---

## 4. Brute vs Optimal

| | Brute (Sort) | Optimal (Heapify) |
|---|---|---|
| **Time** | `O(N log N)` | `O(N)` |
| **Space** | `O(1)` | `O(log N)` auxiliary |
| **Uses existing heap structure?** | ❌ No, discards it | ✅ Yes, builds on the array tree structure |
| **Key idea** | Sort descending = valid max heap | Bottom-up heapify in `O(N)` |
