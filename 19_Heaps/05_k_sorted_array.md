# Check if an Array is K-Sorted

---

## 1. Intuition / Approach

A **K-sorted array** is one where every element is at most **K positions away** from its correct (sorted) position.

In other words, if element at index `i` in the original array should be at index `j` in the sorted array, then `|i - j| <= k`.

---

### Key Insight — Min Heap of Size K+1

In a K-sorted array, the **smallest unplaced element** is always within the first `K+1` elements of the remaining unsorted portion. This is the same logic used to **sort** a K-sorted array.

We maintain a **min heap of size K+1** and process elements in order:
- Push elements into the heap
- When heap size exceeds `K+1`, extract the minimum (the element that **must** be placed next in sorted order)
- Check if this element's **original index** is within K distance from its **expected sorted position** `j`
- If `|j - original_index| > k` → not K-sorted → return `"No"`

`j` tracks the **expected sorted position** (0, 1, 2, ... in order).

---

### Why `K+1` as the window size?

In a K-sorted array, each element can be displaced by at most K positions. So the element that belongs at sorted position `j` must be within indices `[j-k, j+k]` of the original array. Keeping a window of `K+1` elements ensures we always have the correct minimum available to place next.

---

### Dry Run

```
arr = [3, 2, 1, 5, 4, 7, 6, 5], k = 2
```

```
j = 0 (next expected sorted position)

i=0: push (3,0) → pq=[(3,0)]            size=1 ≤ 3
i=1: push (2,1) → pq=[(2,1),(3,0)]      size=2 ≤ 3
i=2: push (1,2) → pq=[(1,2),(3,0),(2,1)] size=3 ≤ 3
i=3: push (5,3) → size=4 > 3
     pop (1,2) → |j=0 - idx=2| = 2 ≤ k=2 ✅  j=1
i=4: push (4,4) → size=4 > 3
     pop (2,1) → |j=1 - idx=1| = 0 ≤ 2 ✅  j=2
i=5: push (7,5) → size=4 > 3
     pop (3,0) → |j=2 - idx=0| = 2 ≤ 2 ✅  j=3
i=6: push (6,6) → size=4 > 3
     pop (4,4) → |j=3 - idx=4| = 1 ≤ 2 ✅  j=4
i=7: push (5,7) → size=4 > 3
     pop (5,3) → |j=4 - idx=3| = 1 ≤ 2 ✅  j=5

Drain remaining pq:
     pop (5,7) → |j=5 - idx=7| = 2 ≤ 2 ✅  j=6
     pop (6,6) → |j=6 - idx=6| = 0 ≤ 2 ✅  j=7
     pop (7,5) → |j=7 - idx=5| = 2 ≤ 2 ✅  j=8

return "Yes" ✅
```

---

## 2. Code

```cpp
class Solution {
public:
    string isKSortedArray(int arr[], int n, int k) {
        // Min heap of {value, original_index}
        priority_queue<pair<int,int>,
                       vector<pair<int,int>>,
                       greater<pair<int,int>>> pq;

        int j = 0;  // next expected sorted position

        for(int i = 0; i < n; i++) {
            pq.push({arr[i], i});

            // Window exceeds K+1 → extract the minimum (it must be placed now)
            if(pq.size() > k + 1) {
                auto p = pq.top();
                pq.pop();

                // Check: original index must be within K of sorted position j
                if(abs(j - p.second) > k)
                    return "No";

                j++;  // next element will go to sorted position j+1
            }
        }

        // Drain remaining elements from heap
        while(!pq.empty()) {
            auto p = pq.top();
            pq.pop();

            if(abs(j - p.second) > k)
                return "No";

            j++;
        }

        return "Yes";
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log K)` | Each of the N elements is pushed and popped from a heap of size `K+1` → each operation is `O(log K)` |
| **Space** | `O(K)` | Heap holds at most `K+1` elements at any time |

> `N` = size of array | `K` = max displacement allowed
