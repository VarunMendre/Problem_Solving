# Non-Overlapping Intervals (Greedy)

---

## 1️⃣ Problem Understanding

We are given an array of intervals:

```
intervals[i] = [start, end]
```

Goal:
Remove the **minimum number of intervals** so that the remaining intervals do not overlap.

Return the number of intervals removed.

---

## 2️⃣ Key Insight (Greedy Strategy)

This problem is a variation of **Activity Selection**.

Instead of maximizing meetings,
we:

👉 Maximize the number of non-overlapping intervals.

Then:

```
Minimum removals = Total intervals - Non-overlapping intervals selected
```

Greedy Rule:
1. Sort intervals by ending time (ascending).
2. Always pick the interval that finishes earliest.
3. Select the next interval only if:
   ```
   current_start >= last_selected_end
   ```

---

## 3️⃣ Step-by-Step Logic

### Step 1: Sort by Ending Time

```cpp
bool static comp (vector<int> p1, vector<int> p2) {
    return p1[1] < p2[1];
}
```

Sorting ensures intervals that finish earliest are considered first.

---

### Step 2: Select Non-Overlapping Intervals

1. Select the first interval.
2. Track its ending time as `lastEndTime`.
3. Traverse remaining intervals:
   - If `intervals[i][0] >= lastEndTime`
     - Select it
     - Update `lastEndTime`

---

## 4️⃣ Code

```cpp
class Solution {

    bool static comp (vector<int> p1, vector<int> p2) {
        return p1[1] < p2[1];
    }
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        int n = intervals.size();

        sort(intervals.begin(), intervals.end(), comp);

        int count = 1;
        int lastEndTime = intervals[0][1];

        for(int i = 1; i < n; i++) {
            if(intervals[i][0] >= lastEndTime) {
                count += 1;
                lastEndTime = intervals[i][1];
            }
        }

        return n - count;
    }
};
```

---

## 5️⃣ Why This Greedy Works

- Picking the earliest finishing interval leaves maximum space for future intervals.
- This maximizes the number of non-overlapping intervals.
- Since we want minimum removals, we subtract selected count from total.

This is mathematically identical to Activity Selection.

---

## 6️⃣ Time and Space Complexity

### Time Complexity:

- Sorting → `O(N log N)`
- Traversal → `O(N)`

Overall:

```
O(N log N + N)
```

---

### Space Complexity:

```
O(1)
```

(If sorting in-place)

---

## ✅ Final Takeaway

Pattern: Greedy + Sorting

Key Idea:
- Sort by end time
- Select maximum non-overlapping intervals
- Answer = total - selected

Th