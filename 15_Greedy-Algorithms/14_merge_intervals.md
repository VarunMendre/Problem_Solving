# Merge Intervals – Detailed Explanation

---

## 1️⃣ Approach / Intuition

### 🔹 Problem Understanding
We are given a list of intervals where each interval is represented as:

[start, end]

Our goal is to merge all overlapping intervals and return a list of non-overlapping intervals.

---

### 🔹 Key Observation
If intervals overlap, then:

current.start <= previous.end

So the idea is:

1. Sort all intervals based on starting time.
2. Traverse them one by one.
3. Merge overlapping intervals.

---

### 🔹 Step-by-Step Intuition

1. **Edge Case Handling**  
   If the input list is empty → return empty result.

2. **Sorting**  
   Sort intervals in ascending order.
   After sorting, overlapping intervals will always be adjacent.

3. **Initialize Temporary Interval**  
   Store the first interval as `tempInterval`.

4. **Traverse All Intervals**  
   For each interval:

   - If it overlaps with `tempInterval`:
     - Update the end of `tempInterval` to:
       max(current.end, tempInterval.end)
   
   - If it does NOT overlap:
     - Push `tempInterval` into result
     - Update `tempInterval` to current interval

5. **Final Step**  
   Push the last remaining `tempInterval` into result.

---

### 🔹 Why Sorting is Important?

Without sorting:
- Overlapping intervals may not be adjacent.
- We would need nested loops (O(N²)).

Sorting ensures we only need a single linear traversal after sorting.

---

## 2️⃣ Code

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        vector<vector<int>> mergedIntervals;

        // Edge case: if no intervals
        if(intervals.size() == 0) {
            return mergedIntervals;
        }

        // Step 1: Sort intervals
        sort(intervals.begin(), intervals.end());

        // Step 2: Take first interval as temporary
        vector<int> tempInterval = intervals[0];

        // Step 3: Traverse intervals
        for(auto it : intervals) {
            // If overlapping
            if(it[0] <= tempInterval[1]) {
                tempInterval[1] = max(it[1], tempInterval[1]);
            }
            // If not overlapping
            else {
                mergedIntervals.push_back(tempInterval);
                tempInterval = it;
            }
        }

        // Step 4: Push last interval
        mergedIntervals.push_back(tempInterval);

        return mergedIntervals;
    }
};
```

---

## 3️⃣ Time & Space Complexity

### 🔹 Time Complexity

1. Sorting → O(N log N)
2. Single traversal → O(N)

Total:

O(N log N + N)

Since N log N dominates:

Final Time Complexity:

O(N log N)

---

### 🔹 Space Complexity

🔸 Auxiliary Space (used to solve problem):

We only use:
- tempInterval (constant space)

So solving space = O(1)

🔸 Output Space:

We store merged intervals in a new vector.
In worst case (no overlaps), we store N intervals.

So output space = O(N)

---

## ✅ Final Summary

Time Complexity:  O(N log N)
Space Complexity: O(N) (because of output array)
Auxiliary Space:  O(1)

---

This is a classic Greedy + Sorting problem.
The core idea is:

"Sort first, then merge overlapping intervals in one pass."

