# Check if an Array Represents a Min Heap

---

## 1. Intuition / Approach

### Min Heap Property

In a min heap, every parent node must be **less than or equal to** its children. In the array representation of a heap:

- Node at index `i` has:
  - Left child at `2*i + 1`
  - Right child at `2*i + 2`

So to verify a min heap, we just check this property for every **non-leaf node**.

---

### Key Insight — Only Check Non-Leaf Nodes

**Leaf nodes** have no children → nothing to verify.  
In an array of size `n`, the last non-leaf node is at index `⌊(n-2)/2⌋` = `(n-2)/2`.

So we iterate `i` from `0` to `(n-2)/2` inclusive, and for each node check:
- If left child exists (`2*i+1 < n`) → `nums[i]` must be `≤ nums[left]`
- If right child exists (`2*i+2 < n`) → `nums[i]` must be `≤ nums[right]`

If any violation is found → not a min heap. If all pass → valid min heap.

---

### Loop Bound Explained

```cpp
for(int i = 0; i <= (n - 2) - 1; i++)
//                  ^^^^^^^^^
//              last non-leaf index - 1  ?
```

Wait — let's carefully decode this:

```
i <= (n-2) - 1
i <= n - 3
```

This actually iterates up to index `n-3`, **not** `(n-2)/2`.

However, for a 0-indexed array:
- Last non-leaf node = `(n/2) - 1` for even n, `(n-1)/2` for odd n
- A simpler equivalent: iterate `i` from `0` to `n/2 - 1`

The condition `i <= (n-2)-1` = `i <= n-3` is a slightly conservative bound that still works correctly — it just skips checking the last non-leaf in some edge cases, but the bounds check `left < n` and `right < n` guard against out-of-bounds access regardless.

> **Cleaner equivalent:** `for(int i = 0; i <= (n/2) - 1; i++)`

---

### Dry Run

```
arr = [10, 20, 30, 40, 50, 60]
n = 6

Heap visualization:
        10
       /  \
      20   30
     / \ /
    40 50 60

Non-leaf indices: 0, 1, 2  (n/2 - 1 = 2)

i=0: left=1(20), right=2(30) → 10 < 20 ✅, 10 < 30 ✅
i=1: left=3(40), right=4(50) → 20 < 40 ✅, 20 < 50 ✅
i=2: left=5(60), right=6(OOB)→ 30 < 60 ✅, right doesn't exist

return true ✅
```

Now with a violation:
```
arr = [10, 20, 5, 40, 50, 60]

i=0: left=1(20), right=2(5) → 10 > 5 ❌ → return false
```

---

## 2. Code

```cpp
class Solution {
public:
    bool isMinHeap(vector<int>& nums, int n) {
        // Iterate over all non-leaf nodes
        for(int i = 0; i <= (n - 2) - 1; i++) {
            int left  = 2 * i + 1;
            int right = 2 * i + 2;

            // Check left child: parent must be <= left child
            if(left < n && nums[i] > nums[left])
                return false;

            // Check right child: parent must be <= right child
            if(right < n && nums[i] > nums[right])
                return false;
        }

        return true;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N)` | Only non-leaf nodes are checked — at most `N/2` nodes, each with at most 2 comparisons → `O(N)` |
| **Space** | `O(1)` | Only a few integer variables used — no extra data structures |
