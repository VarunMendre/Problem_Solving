# Find Median from Data Stream

---

## 1. Brute Force — Sort on Every Query

### Approach

Store all numbers in a vector. When `findMedian()` is called, sort the vector and return the middle element(s).

```cpp
class MedianFinder {
private:
    vector<int> nums;

public:
    MedianFinder() {}

    void addNum(int num) {
        nums.push_back(num);    // just append — O(1)
    }

    double findMedian() {
        sort(nums.begin(), nums.end());     // O(N log N) every call
        int n = nums.size();

        if(n % 2 == 1)
            return nums[n / 2];             // odd → middle element

        return (nums[n/2 - 1] + nums[n/2]) / 2.0;  // even → avg of two middles
    }
};
```

### Time & Space Complexity

| Operation | Time | Reason |
|---|---|---|
| `addNum` | `O(1)` | Simple vector append |
| `findMedian` | `O(N log N)` | Sorts entire array every call |
| **Space** | `O(N)` | Vector stores all N elements |

> Fine for few queries, but if `findMedian` is called frequently on a large stream it's very expensive.

---

---

## 2. Optimal — Two Heaps

### Core Idea

The **median** sits at the boundary between the smaller half and the larger half of all numbers seen so far. If we maintain:

- **Max heap** (`maxHeap`) → stores the **smaller half** (top = largest of the small half)
- **Min heap** (`minHeap`) → stores the **larger half** (top = smallest of the large half)

Then:
- **Odd** total elements → `maxHeap` has one extra → median = `maxHeap.top()`
- **Even** total elements → both heaps equal size → median = `(maxHeap.top() + minHeap.top()) / 2.0`

---

### Invariants Maintained

1. Every element in `maxHeap` ≤ every element in `minHeap`
2. `maxHeap.size() == minHeap.size()` OR `maxHeap.size() == minHeap.size() + 1`

---

### `addNum` Logic — 3 Steps

```cpp
// Step 1: push to maxHeap first
maxHeap.push(num);

// Step 2: balance by moving maxHeap's top to minHeap
// → ensures all of minHeap is ≥ all of maxHeap
minHeap.push(maxHeap.top());
maxHeap.pop();

// Step 3: if minHeap got too big, move one back to maxHeap
// → ensures maxHeap is always >= minHeap in size
if(minHeap.size() > maxHeap.size()) {
    maxHeap.push(minHeap.top());
    minHeap.pop();
}
```

**Why push to maxHeap first then transfer?**  
We can't push `num` directly to `minHeap` without checking — `num` might be smaller than the current median and belong in `maxHeap`. Pushing to `maxHeap` first and then moving its top to `minHeap` elegantly handles both cases with correct ordering.

---

### Dry Run

```
Stream: 1, 7, 3
```

**addNum(1):**
```
maxHeap.push(1)  → maxH=[1]
minHeap.push(1), maxHeap.pop() → minH=[1], maxH=[]
minH.size(1) > maxH.size(0) → maxH.push(1), minH.pop()
→ maxH=[1], minH=[]
```

**findMedian():** maxH.size > minH.size → return `maxH.top() = 1` ✅

---

**addNum(7):**
```
maxHeap.push(7)  → maxH=[7,1]
minHeap.push(7), maxHeap.pop() → minH=[7], maxH=[1]
minH.size(1) == maxH.size(1) → no rebalance
→ maxH=[1], minH=[7]
```

**findMedian():** equal sizes → `(1+7)/2.0 = 4.0` ✅

---

**addNum(3):**
```
maxHeap.push(3)  → maxH=[3,1]
minHeap.push(3), maxHeap.pop() → minH=[3,7], maxH=[1]
minH.size(2) > maxH.size(1) → maxH.push(3), minH.pop()
→ maxH=[3,1], minH=[7]
```

**findMedian():** maxH.size > minH.size → return `maxH.top() = 3` ✅

---

## 2. Code

```cpp
class MedianFinder {
public:
    priority_queue<int> maxHeap;                                // smaller half — max on top
    priority_queue<int, vector<int>, greater<int>> minHeap;    // larger half — min on top

    MedianFinder() {}

    void addNum(int num) {
        // Always push to maxHeap first
        maxHeap.push(num);

        // Transfer maxHeap's top to minHeap to maintain ordering invariant
        // (ensures every element in maxHeap ≤ every element in minHeap)
        minHeap.push(maxHeap.top());
        maxHeap.pop();

        // Rebalance: maxHeap must always be >= minHeap in size
        if(minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }

    double findMedian() {
        // Equal sizes → even count → average of two middles
        if(minHeap.size() == maxHeap.size())
            return (maxHeap.top() + minHeap.top()) / 2.0;

        // maxHeap has one extra → odd count → its top is the median
        return maxHeap.top();
    }
};
```

---

## 3. Time & Space Complexity

### Per-Operation

| Operation | Time | Reason |
|---|---|---|
| `addNum` | `O(log N)` | At most 2 pushes + 2 pops on heaps of size N/2 each → `O(log N)` |
| `findMedian` | `O(1)` | Just peek tops of both heaps |
| **Space** | `O(N)` | Both heaps together store all N elements |

---

## 4. Brute vs Optimal

| | Brute | Optimal |
|---|---|---|
| `addNum` | `O(1)` | `O(log N)` |
| `findMedian` | `O(N log N)` | `O(1)` |
| **Space** | `O(N)` | `O(N)` |
| **Best when** | Few median queries | Frequent median queries on large streams |

> The optimal trades a slightly slower `addNum` (`O(log N)` vs `O(1)`) for a dramatically faster `findMedian` (`O(1)` vs `O(N log N)`). For a data *stream* where both operations are called frequently, this is a huge win.
