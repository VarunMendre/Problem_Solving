# Job Sequencing with Deadlines (Greedy)

---

## 1️⃣ Problem Understanding

Each job has:
- `job[i][0]` → Job ID
- `job[i][1]` → Deadline
- `job[i][2]` → Profit

Rules:
- Each job takes **1 unit of time**.
- A job must be completed **on or before its deadline**.

Goal:
Maximize:
1. Number of jobs done
2. Total profit

---

## 2️⃣ Greedy Strategy

👉 Always pick the job with the **highest profit first**.

Why?
If we schedule lower-profit jobs first,
we may lose the chance to schedule a higher-profit job later.

So the greedy steps are:
1. Sort jobs in **descending order of profit**.
2. Try to schedule each job in the latest possible free slot before its deadline.

This ensures:
- High-profit jobs are prioritized.
- Earlier slots remain available for other jobs.

---

## 3️⃣ Step-by-Step Logic

### Step 1: Sort by Profit

```cpp
bool comp(vector<int> &a, vector<int> &b) {
    return a[2] > b[2];
}
```

This ensures higher profit jobs come first.

---

### Step 2: Find Maximum Deadline

We need enough time slots to accommodate jobs.

```cpp
int maxDeadline = 0;
for(int i = 0; i < n; i++) {
    maxDeadline = max(maxDeadline, jobs[i][1]);
}
```

---

### Step 3: Create Time Slots

```cpp
vector<int> slot(maxDeadline + 1, -1);
```

- Index represents time.
- `-1` means slot is free.

---

### Step 4: Assign Jobs

For each job (in profit order):

- Try to place it at its deadline.
- If occupied, try previous slot.
- Continue until slot 1.

```cpp
for(int i = 0; i < n; i++) {
    for(int j = jobs[i][1]; j > 0; j--) {
        if(slot[j] == -1) {
            slot[j] = jobs[i][0];
            count++;
            totalProfit += jobs[i][2];
            break;
        }
    }
}
```

Why place it as late as possible?
Because it leaves earlier slots free for other jobs.

---

## 4️⃣ Code

```cpp
bool comp(vector<int> &a, vector<int> &b) {
    return a[2] > b[2];
}

vector<int> jobScheduling(vector<vector<int>> &jobs)
{
    int n = jobs.size();

    sort(jobs.begin(), jobs.end(), comp);

    int totalProfit = 0, count = 0;

    int maxDeadline = 0;
    for(int i = 0; i < n; i++) {
        maxDeadline = max(maxDeadline, jobs[i][1]);
    }

    vector<int> slot(maxDeadline + 1, -1);

    for(int i = 0; i < n; i++) {
        for(int j = jobs[i][1]; j > 0; j--) {
            if(slot[j] == -1) {
                slot[j] = jobs[i][0];
                count++;
                totalProfit += jobs[i][2];
                break;
            }
        }
    }

    return {count, totalProfit};
}
```

---

## 5️⃣ Why This Greedy Works

- Highest profit jobs are handled first.
- Placing them at the latest available slot avoids blocking other jobs.
- Every decision maximizes profit locally.
- That leads to globally optimal total profit.

This is a classic greedy scheduling problem.

---

## 6️⃣ Time and Space Complexity

### Time Complexity:

- Sorting → `O(N log N)`
- Slot assignment → Worst case `O(N * D)`
  - D = maximum deadline

Overall:

```
O(N log N + N * D)
```

### Space Complexity:

```
O(D)
```

- For slot array

---

## ✅ Final Takeaway

Pattern: Greedy + Scheduling

Key Ideas:
- Sort by descending profit
- Schedule as late as possible
- Preserve early slots

This is one of the most important greedy interval scheduling problems.

