# Kth Largest Element in an Array

---

## 1. Intuition / Approach

We want the **kth largest** element. The key insight:

> If we maintain a **min heap of size K**, the top of the heap is always the **smallest among the K largest elements seen so far** — which is exactly the **kth largest**.

### Why a Min Heap?

A min heap of size K acts as a **sliding window of the K largest elements**:
- When a new element is **larger than the heap's top** → it deserves a spot in the top-K → pop the smallest, push the new one
- When a new element is **smaller or equal** → it can't be in the top-K → ignore it

At the end, the heap contains the **K largest elements**, and `pq.top()` (the minimum of those K) = the **Kth largest**.

---

### Step by Step

1. Push the first `K` elements into the min heap
2. For every remaining element:
   - If it's **greater than heap top** → pop top, push new element (maintain size K)
   - Otherwise → skip
3. Return `pq.top()`

---

### Dry Run

```
nums = [3, 2, 1, 5, 6, 4], k = 2
```

**Step 1 — Fill first K=2 elements:**
```
push 3 → pq = [3]
push 2 → pq = [2, 3]  (min heap, 2 on top)
```

**Step 2 — Process remaining:**
```
i=2: nums[2]=1 → 1 > pq.top()=2? NO → skip    pq = [2, 3]
i=3: nums[3]=5 → 5 > pq.top()=2? YES → pop 2, push 5  pq = [3, 5]
i=4: nums[4]=6 → 6 > pq.top()=3? YES → pop 3, push 6  pq = [5, 6]
i=5: nums[5]=4 → 4 > pq.top()=5? NO → skip    pq = [5, 6]
```

**Result:** `pq.top() = 5` ✅ (2nd largest in [3,2,1,5,6,4] is 5)

---

## 2. Code

```cpp
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        // Min heap of size K — top is the smallest of the K largest elements
        priority_queue<int, vector<int>, greater<int>> pq;

        // Step 1: fill the heap with the first K elements
        for(int i = 0; i < k; i++) {
            pq.push(nums[i]);
        }

        // Step 2: for remaining elements, maintain K largest
        for(int i = k; i < nums.size(); i++) {
            if(nums[i] > pq.top()) {
                // Current element is larger than smallest in top-K
                // → replace it
                pq.pop();
                pq.push(nums[i]);
            }
            // else: nums[i] can't be in top-K → skip
        }

        // Top of min heap = Kth largest
        return pq.top();
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log K)` | First K pushes cost `O(K log K)`; remaining `N-K` elements each cost `O(log K)` for push/pop → total `O(N log K)` |
| **Space** | `O(K)` | Min heap holds exactly K elements at all times |

> **Correction on your TC:** You wrote `O(N log N)` — that would be the cost if we used the full heap of size N. Since our heap is capped at size **K**, each push/pop is `O(log K)`, not `O(log N)`. So the correct TC is **`O(N log K)`**.  
> When `K ≈ N` these are equivalent, but for small K (e.g. K=3 in a million elements), `O(N log K)` is significantly better.

---

## 4. Why Not a Max Heap?

With a max heap you'd push all N elements (`O(N log N)`) and pop K times (`O(K log N)`) → total `O(N log N)`. That's worse than the min heap approach when K is small.

| Approach | TC | SC |
|---|---|---|
| Max heap (push all, pop K) | `O(N log N)` | `O(N)` |
| **Min heap of size K** | **`O(N log K)`** | **`O(K)`** |
