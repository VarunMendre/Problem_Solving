# Insert Interval — Code Explanation (Point Wise)

---

## 1️⃣ Function Overview

This function inserts a new interval into an already **sorted** and **non-overlapping** list of intervals, and merges if necessary.

```cpp
vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval)
```

---

## 2️⃣ Initial Setup

```cpp
int n = intervals.size();
vector<vector<int>> result;
int i = 0;
```

- `n` → total number of intervals
- `result` → stores final merged answer
- `i` → pointer to traverse intervals

---

## 3️⃣ Left Part (Non-Overlapping Before New Interval)

```cpp
while(i < n && intervals[i][1] < newInterval[0]) {
    result.push_back(intervals[i]);
    i++;
}
```

### Condition Meaning:

```
intervals[i][1] < newInterval[0]
```

This means:
- Current interval ends before new interval starts
- No overlap possible

### Action:
- Directly push into result
- Move pointer forward

These intervals remain unchanged.

---

## 4️⃣ Merging Overlapping Part

```cpp
while(i < n && intervals[i][0] <= newInterval[1]){
    newInterval[0] = min(newInterval[0], intervals[i][0]);
    newInterval[1] = max(newInterval[1], intervals[i][1]);
    i++;
}
```

### Overlap Condition:

```
intervals[i][0] <= newInterval[1]
```

This means:
- Current interval starts before new interval ends
- Overlap exists

### Merging Logic:

- Start becomes minimum of both starts
- End becomes maximum of both ends

So `newInterval` keeps expanding to absorb all overlapping intervals.

---

## 5️⃣ Insert Merged Interval

```cpp
result.push_back(newInterval);
```

After finishing overlap merging:
- The final merged interval is added once.

---

## 6️⃣ Right Part (Remaining Intervals)

```cpp
while(i < n) {
    result.push_back(intervals[i]);
    i++;
}
```

These intervals:
- Start after merged interval ends
- No overlap

So we directly append them.

---

## 7️⃣ Return Result

```cpp
return result;
```

Final list contains:

Left Non-Overlapping + Merged Interval + Right Non-Overlapping

---

## 8️⃣ Time and Space Complexity

### Time Complexity:

O(n)

Each interval is visited once.

### Space Complexity:

O(n)

Result vector stores up to n+1 intervals.

---

## ✅ Final Structure of Algorithm

The code works in exactly **three zones**:

1. Left (no overlap)
2. Merge (overlapping part)
3. Right (no overlap)

This is the standard template for Insert Interval problems.

