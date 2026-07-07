# Kth Largest Element in a Stream

---

## 1. Intuition / Approach

We need to maintain the **Kth largest element** as new values are continuously added to a stream.

### Core Idea — Min Heap of Size K

Keep a **min heap of exactly K elements** that always holds the **K largest values seen so far**.

- The **top** of the heap = smallest among the K largest = **Kth largest overall**
- When a new element comes in:
  - Push it into the heap
  - If heap size exceeds K → pop the smallest (it can't be in the top K)
  - Top of heap is now the Kth largest

This is the same "sliding K largest" pattern from **Kth Largest Element in an Array**, but applied to a stream dynamically.

---

### Why Min Heap (Not Max Heap)?

| | Max Heap | Min Heap of size K |
|---|---|---|
| What's on top | Largest element | Kth largest element |
| Finding Kth largest | Need K pops | Just peek top → `O(1)` |

A min heap of size K keeps the K largest candidates, and the smallest of them (heap top) is the Kth largest.

---

### Dry Run

```
k = 3, nums = [4, 5, 8, 2]
```

**Constructor:**
```
push 4 → pq=[4],    size=1 ≤ 3
push 5 → pq=[4,5],  size=2 ≤ 3
push 8 → pq=[4,5,8],size=3 ≤ 3
push 2 → pq=[2,4,5,8], size=4 > 3 → pop 2 → pq=[4,5,8]

pq = [4,5,8]  (min on top = 4 = 3rd largest) ✅
```

**add(3):**
```
push 3 → pq=[3,4,5,8], size=4 > 3 → pop 3 → pq=[4,5,8]
return pq.top() = 4 ✅  (3rd largest of {4,5,8,2,3} = 4)
```

**add(5):**
```
push 5 → pq=[4,5,5,8], size=4 > 3 → pop 4 → pq=[5,5,8]
return pq.top() = 5 ✅  (3rd largest of {4,5,8,2,3,5} = 5)
```

**add(10):**
```
push 10 → pq=[5,5,8,10], size=4 > 3 → pop 5 → pq=[5,8,10]
return pq.top() = 5 ✅  (3rd largest)
```

---

## 2. Code

```cpp
class KthLargest {
public:
    int K;
    priority_queue<int, vector<int>, greater<int>> pq;   // min heap

    KthLargest(int k, vector<int>& nums) {
        K = k;

        for(int& num : nums) {
            pq.push(num);           // push each element

            if(pq.size() > k)
                pq.pop();           // evict smallest if over K — keeps top K largest
        }
    }

    int add(int val) {
        pq.push(val);               // add new stream element

        if(pq.size() > K)
            pq.pop();               // evict if over K

        return pq.top();            // top of min heap = Kth largest
    }
};
```

---

## 3. Time & Space Complexity

### Constructor — `KthLargest(k, nums)`

| Step | Cost | Reason |
|---|---|---|
| Push each of N elements | `O(log K)` per push | Heap size capped at K → each operation is `O(log K)` |
| Pop when size > K | `O(log K)` per pop | Same reason |
| **Total** | **`O(N log K)`** | N elements processed, each costs `O(log K)` |

---

### `add(val)` — Single Call

| Step | Cost | Reason |
|---|---|---|
| `pq.push(val)` | `O(log K)` | Heap has at most K+1 elements temporarily |
| `pq.pop()` (if needed) | `O(log K)` | Same |
| `pq.top()` | `O(1)` | Just peek at root |
| **Total per call** | **`O(log K)`** | |

---

### Space Complexity

| Structure | Size | Reason |
|---|---|---|
| Min heap `pq` | `O(K)` | Always maintains exactly K elements after construction |

---

### Summary

| Operation | Time | Space |
|---|---|---|
| `KthLargest(k, nums)` | `O(N log K)` | `O(K)` |
| `add(val)` — single call | `O(log K)` | `O(1)` extra |
| `add(val)` — M calls total | `O(M log K)` | `O(1)` extra per call |

> **Key distinction:** It's `O(log K)` not `O(log N)` because the heap is **always capped at size K** — heap operations depend on heap size, not total elements processed.
