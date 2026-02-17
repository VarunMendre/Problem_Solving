# Minimum Average Waiting Time (Greedy Scheduling)

---

## 1️⃣ Problem Understanding

Given an array `bt[]` representing burst times of processes.

Goal:
Minimize the **average waiting time**.

Waiting time of a process =
Time spent waiting before its execution starts.

We must return:

```
Total Waiting Time / Number of Processes
```

---

## 2️⃣ Key Insight (Greedy Strategy)

To minimize average waiting time:

👉 Execute the process with the **smallest burst time first**.

This is known as:

**Shortest Job First (SJF)** Scheduling

Why does this work?
- Short jobs finish early.
- They contribute less waiting time to other processes.
- Long jobs executed early would increase waiting time for many processes.

So sorting ensures optimal order.

---

## 3️⃣ Step-by-Step Logic

1. Sort the burst time array.
2. Maintain:
   - `timer` → cumulative execution time
   - `waitingTime` → total waiting time
3. For each process:
   - Add current `timer` to `waitingTime`
   - Increase `timer` by current burst time
4. Return average waiting time.

---

## 4️⃣ Code

```cpp
class Solution {
  public:
    long long solve(vector<int>& bt) {
        int n = bt.size();
        
        sort(bt.begin(), bt.end());
        
        int timer = 0, waitingTime = 0;
        
        for(int i = 0; i < n; i++) {
            waitingTime += timer;
            timer += bt[i];
        }
        
        long long ans = waitingTime / n;
        
        return ans;
    }
};
```

---

## 5️⃣ Example

If burst times = [3, 1, 2]

After sorting → [1, 2, 3]

Waiting times:
- Process 1 → 0
- Process 2 → 1
- Process 3 → 1 + 2 = 3

Total waiting time = 0 + 1 + 3 = 4

Average = 4 / 3

---

## 6️⃣ Time and Space Complexity

- **Time Complexity:**
  ```
  O(n log n)
  ```
  (Due to sorting)

- **Space Complexity:**
  ```
  O(1)
  ```
  (Ignoring sorting space)

---

## ✅ Final Takeaway

- Pattern: Greedy + Sorting
- Concept: Shortest Job First (SJF)
- Sorting minimizes cumulative waiting impact

This is a fundamental greedy scheduling problem.

