# Task Scheduler

---

## 1. Intuition / Approach

We need to schedule tasks such that **the same task must have at least `n` intervals apart**. We want to minimize total CPU time (including idle slots).

### Greedy Strategy — Most Frequent Task First

At every scheduling "chunk" of `n+1` slots:
- Execute the **most frequent remaining tasks** (up to `n+1` of them)
- If fewer than `n+1` distinct tasks are left → the remaining slots are **idle** (only when more tasks are pending)
- If no tasks are left after the chunk → add only the actual tasks scheduled (no idle needed for the last chunk)

**Why `n+1` as the chunk size?**  
After executing a task, we must wait `n` intervals before using it again. So in one "round", we can fit `n+1` distinct tasks (1 execution + n cooldown slots for other tasks/idle).

---

### Why Max Heap?

We always want to schedule the **most frequent task** first — it's the most "urgent" bottleneck. A max heap gives us the highest-frequency task in `O(log K)` where K ≤ 26.

---

### Scheduling Logic

Each iteration:
1. Pop up to `n+1` tasks from the max heap, decrement their frequency, store in `temp`
2. Push back tasks with remaining frequency > 0
3. If heap is **not empty** after the round → full chunk ran → `time += n+1`
4. If heap is **empty** after the round → last chunk → `time += temp.size()` (only actual tasks, no idle)

---

### Dry Run

```
tasks = ["A","A","A","B","B","B"], n = 2
freq: A=3, B=3
pq = [3, 3]  (max heap)
```

```
Round 1: chunk size = n+1 = 3
  pop 3(A) → freq=2, pop 3(B) → freq=2, pq empty → temp=[2,2]
  push back 2(A), 2(B) → pq=[2,2]
  pq not empty → time += n+1 = 3   (schedule: A B idle)  time=3

Round 2:
  pop 2(A) → freq=1, pop 2(B) → freq=1, pq empty → temp=[1,1]
  push back 1(A), 1(B) → pq=[1,1]
  pq not empty → time += 3   (schedule: A B idle)  time=6

Round 3:
  pop 1(A) → freq=0, pop 1(B) → freq=0, pq empty → temp=[0,0]
  nothing pushed back → pq empty
  pq empty → time += temp.size() = 2   (schedule: A B)  time=8

return 8 ✅
```

---

### Example 3 Dry Run

```
tasks = ["A","A","A","B","B","B"], n = 3
pq = [3, 3]
```

```
Round 1: chunk = 4
  pop 3(A)→2, pop 3(B)→2, pq now empty, i goes to 3,4 but pq empty → temp=[2,2]
  push back → pq=[2,2]
  pq not empty → time += n+1 = 4   (A B idle idle)  time=4

Round 2:
  pop 2(A)→1, pop 2(B)→1 → temp=[1,1]
  push back → pq=[1,1]
  pq not empty → time += 4   (A B idle idle)  time=8

Round 3:
  pop 1(A)→0, pop 1(B)→0 → temp=[0,0]
  pq empty → time += temp.size() = 2   (A B)  time=10

return 10 ✅
```

---

## 2. Code

```cpp
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        // Count frequency of each task
        vector<int> mpp(26, 0);
        for(char& ch : tasks)
            mpp[ch - 'A']++;

        int time = 0;

        // Max heap — always schedule most frequent task first
        priority_queue<int> pq;
        for(int i = 0; i < 26; i++) {
            if(mpp[i] > 0)
                pq.push(mpp[i]);
        }

        while(!pq.empty()) {
            vector<int> temp;

            // Schedule up to n+1 tasks in this round
            for(int i = 1; i <= n + 1; i++) {
                if(!pq.empty()) {
                    int freq = pq.top();
                    pq.pop();
                    freq--;
                    temp.push_back(freq);
                }
            }

            // Push back tasks that still have remaining frequency
            for(int& f : temp) {
                if(f > 0)
                    pq.push(f);
            }

            if(pq.empty()) {
                // Last chunk — no idle needed, count only actual tasks
                time += temp.size();
            } else {
                // More tasks remain — full chunk of n+1 (includes idles if needed)
                time += n + 1;
            }
        }

        return time;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity — `O(N)`

| Step | Cost | Reason |
|---|---|---|
| Count frequencies | `O(N)` | Single pass over tasks array |
| Build heap | `O(26 log 26)` = `O(1)` | At most 26 distinct tasks |
| Schedule all tasks | `O(N log 26)` = `O(N)` | Each task popped and pushed at most once; heap size ≤ 26 → `log 26` = constant |

**Total: `O(N)`**

> The heap never exceeds 26 elements (only 26 possible tasks A–Z), so all heap operations are `O(log 26)` = `O(1)`. This is what keeps the overall complexity linear in N.

---

### Space Complexity — `O(1)`

| Structure | Size |
|---|---|
| `mpp` frequency array | 26 |
| Priority queue | At most 26 elements |
| `temp` vector per round | At most 26 elements |

**Total: `O(1)`** (all bounded by the constant 26)
