# Ninja Training

---

## 1. Problem Statement

A ninja needs to train for `n` days. Each day, they can do one of **3 tasks** (0, 1, or 2). Each task on each day gives certain merit points `matrix[day][task]`.

**Constraint:** The ninja CANNOT do the same task on two **consecutive** days.

Find the **maximum total merit points** the ninja can earn over `n` days.

```
matrix = [[10, 40, 70],
           [20, 50, 80],
           [30, 60, 90]]

Day 0: tasks [10, 40, 70]
Day 1: tasks [20, 50, 80]
Day 2: tasks [30, 60, 90]

Best: Day 0 → task 2 (70), Day 1 → task 0 or 1 (can't do task 2 again)
      Day 1 → task 1 (50) (skip task 2=80 since prev was 2), Day 2 → task 2 (90)
      Total: 70 + 50 + 90 = 210

Or: Day 0 → task 0 (10), Day 1 → task 2 (80), Day 2 → task 1 (60) = 150

Best answer: 210
```

---

## 2. Intuition — The 2D State

This problem has two changing variables per step:
1. **Day** — which day we're on
2. **Last task** — which task we did yesterday (to enforce the no-repeat constraint)

So the state is `f(day, prevTask)` = maximum points from day `0` to `day`, given that we did `prevTask` on `day+1` (the day after).

**Why track `prevTask` and not `currentTask`?**

The constraint is "can't repeat CONSECUTIVE tasks." When deciding what to do on `day`, we need to know what we did on `day+1` (the next day, since we recurse top-down from the last day). Tracking `prevTask` (the task of the day we're coming FROM) lets us exclude it from today's choices.

---

### The `prevTask = 3` Encoding

We use `prevTask = 3` (or `-1` in recursion) to mean "no task was done before" — i.e., the very first call (starting from the last day with no constraint from the future).

Why `3`? The dp array has 4 columns: indices `0,1,2` for tasks, and `3` for "no previous task." This way, the dp table is `n × 4`.

---

## 3. Approach 1 — Pure Recursion

### Intuition

`f(day, prevTask)` = max points from day `0` to `day`, where on day `day+1` (already decided) we did `prevTask`.

Try every task `i` (0, 1, 2) for `day`, skip if `i == prevTask`:
- Points = `matrix[day][i] + f(day-1, i)`

Take the maximum.

```cpp
class Solution {
private:
    int fn(int day, int prevTask, vector<vector<int>>& matrix) {
        if(day == 0) {
            int maxi = INT_MIN;
            for(int i = 0; i < 3; i++) {
                if(i != prevTask)
                    maxi = max(maxi, matrix[day][i]);
            }
            return maxi;
        }

        int maxi = INT_MIN;
        for(int i = 0; i < 3; i++) {
            if(i != prevTask) {
                int points = matrix[day][i] + fn(day-1, i, matrix);
                maxi = max(maxi, points);
            }
        }
        return maxi;
    }

public:
    int ninjaTraining(vector<vector<int>>& matrix) {
        int n = matrix.size();
        return fn(n-1, 3, matrix);   // 3 = no previous task constraint
    }
};
```

### Recursion Tree (3 days, 3 tasks)

Each call branches into at most 2 sub-calls (skip the previous task → 2 of 3 remaining). Many subproblems repeat.

```
f(2, 3)
├── task=0: f(1, 0) + matrix[2][0]
│   ├── task=1: f(0,1) + matrix[1][1]
│   └── task=2: f(0,2) + matrix[1][2]
├── task=1: f(1, 1) + matrix[2][1]
│   ├── task=0: f(0,0) + matrix[1][0]   ← f(0,0) REPEATED
│   └── task=2: f(0,2) + matrix[1][2]   ← f(0,2) REPEATED
└── task=2: f(1, 2) + matrix[2][2]
    ...
```

**Overlapping subproblems:** `f(0, 0)`, `f(0, 1)`, `f(0, 2)` all computed multiple times.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(3ⁿ)` | Each call has up to 3 sub-calls (or 2 after excluding prev) |
| **Space** | `O(n)` | Recursion stack depth |

---

## 4. Approach 2 — Memoization (Top-Down DP)

Add `dp[day][prevTask]` to cache:

```cpp
class Solution {
private:
    int fn(int day, int prevTask, vector<vector<int>>& dp, vector<vector<int>>& matrix) {
        if(day == 0) {
            int maxi = INT_MIN;
            for(int i = 0; i < 3; i++) {
                if(i != prevTask)
                    maxi = max(maxi, matrix[day][i]);
            }
            return dp[day][prevTask] = maxi;
        }

        if(dp[day][prevTask] != -1)
            return dp[day][prevTask];

        int maxi = INT_MIN;
        for(int i = 0; i < 3; i++) {
            if(i != prevTask) {
                int points = matrix[day][i] + fn(day-1, i, dp, matrix);
                maxi = max(maxi, points);
            }
        }
        return dp[day][prevTask] = maxi;
    }

public:
    int ninjaTraining(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<vector<int>> dp(n, vector<int>(4, -1));
        return fn(n-1, 3, dp, matrix);
    }
};
```

### dp Table Dimensions

`dp[n][4]`:
- `n` rows = n days
- `4` columns = prevTask ∈ {0, 1, 2, 3} (3 tasks + "no constraint")

Total states = `n × 4`. Each computed once → `O(4n)` = `O(n)` time.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × 4 × 3)` = `O(12n)` = `O(n)` | n days × 4 prev states × 3 task choices |
| **Space** | `O(4n) + O(n)` | dp table + recursion stack |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

### Base Cases for Day 0

On day 0, there's no "previous day" constraint — except from what the next day chose. We precompute all 4 cases:

```cpp
dp[0][0] = max(matrix[0][1], matrix[0][2]);   // prevTask=0 → can do task 1 or 2
dp[0][1] = max(matrix[0][0], matrix[0][2]);   // prevTask=1 → can do task 0 or 2
dp[0][2] = max(matrix[0][0], matrix[0][1]);   // prevTask=2 → can do task 0 or 1
dp[0][3] = max({matrix[0][0], matrix[0][1], matrix[0][2]});  // no constraint → best of all
```

`dp[0][last]` = best points on day 0 given that on day 1 we did task `last`.

### Main Loop

```cpp
for(int day = 1; day < n; day++) {
    for(int last = 0; last < 4; last++) {
        for(int task = 0; task < 3; task++) {
            if(last != task) {
                int points = matrix[day][task] + dp[day-1][task];
                dp[day][last] = max(dp[day][last], points);
            }
        }
    }
}
```

For each `(day, last)` pair, try all tasks that aren't `last`, find max.

### Why `dp[day-1][task]` Not `dp[day-1][last]`?

```cpp
int points = matrix[day][task] + dp[day-1][task];
```

We're doing `task` today. Yesterday's best, given that TODAY (which comes AFTER yesterday) we did `task`, is `dp[day-1][task]`. The second dimension is "what the NEXT day did" — and today is the next day relative to yesterday.

Confusing at first: the `last` variable represents "the task done on the day after the current day." For day `d`, `last` = "what day `d+1` did." Since we're building upward, `dp[day][last]` = "best points through day `day`, given that day `day+1` does task `last`."

### Tabulation Trace (matrix=[[10,40,70],[20,50,80],[30,60,90]])

**Day 0 base cases:**
```
dp[0][0] = max(40,70) = 70       (skip task 0)
dp[0][1] = max(10,70) = 70       (skip task 1)
dp[0][2] = max(10,40) = 40       (skip task 2)
dp[0][3] = max(10,40,70) = 70    (no constraint)
```

**Day 1:**
```
For last=0 (day 1 can't do task 0):
  task=1: matrix[1][1]+dp[0][1]=50+70=120
  task=2: matrix[1][2]+dp[0][2]=80+40=120
  dp[1][0] = 120

For last=1:
  task=0: 20+70=90
  task=2: 80+40=120
  dp[1][1] = 120

For last=2:
  task=0: 20+70=90
  task=1: 50+70=120
  dp[1][2] = 120

For last=3 (no constraint):
  task=0: 20+70=90
  task=1: 50+70=120
  task=2: 80+40=120
  dp[1][3] = 120
```

**Day 2:**
```
For last=3 (no constraint on day 3 which doesn't exist):
  task=0: matrix[2][0]+dp[1][0]=30+120=150
  task=1: matrix[2][1]+dp[1][1]=60+120=180
  task=2: matrix[2][2]+dp[1][2]=90+120=210
  dp[2][3] = 210

return dp[2][3] = 210 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × 4 × 3)` = `O(n)` | Triple nested loop, all constants |
| **Space** | `O(n × 4)` = `O(n)` | 2D dp table |

---

## 6. Approach 4 — Space Optimization

`dp[day]` only needs `dp[day-1]` → replace the 2D table with one 1D `prev` array:

```cpp
class Solution {
public:
    int ninjaTraining(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Base: day 0
        vector<int> prev(4, 0);
        prev[0] = max(matrix[0][1], matrix[0][2]);
        prev[1] = max(matrix[0][0], matrix[0][2]);
        prev[2] = max(matrix[0][0], matrix[0][1]);
        prev[3] = max(matrix[0][0], max(matrix[0][1], matrix[0][2]));

        for(int day = 1; day < n; day++) {
            vector<int> curr(4, 0);

            for(int last = 0; last < 4; last++) {
                for(int task = 0; task < 3; task++) {
                    if(last != task) {
                        int points = matrix[day][task] + prev[task];
                        curr[last] = max(curr[last], points);
                    }
                }
            }

            prev = curr;   // slide forward
        }

        return prev[3];
    }
};
```

### Why `prev[task]` Not `prev[last]`?

```cpp
int points = matrix[day][task] + prev[task];
```

`prev[task]` = best points through day `day-1`, given that on day `day` (current) we do `task`.

We're saying: "If today I do `task`, yesterday's best (subject to today's task being `task`) is `prev[task]`."

The second index is always "what tomorrow (the next day) does." Today is tomorrow's yesterday. So `prev[task]` correctly reads "yesterday's best, knowing I'll do `task` today."

### Space Optimization Trace (same matrix)

```
Initial prev = [70, 70, 40, 70]

Day 1:
  curr[0]: task=1→50+prev[1]=50+70=120, task=2→80+prev[2]=80+40=120 → curr[0]=120
  curr[1]: task=0→20+prev[0]=20+70=90,  task=2→80+40=120             → curr[1]=120
  curr[2]: task=0→20+70=90, task=1→50+70=120                          → curr[2]=120
  curr[3]: task=0→90, task=1→120, task=2→120                          → curr[3]=120

  prev = [120,120,120,120]

Day 2:
  curr[3]: task=0→30+prev[0]=30+120=150
           task=1→60+prev[1]=60+120=180
           task=2→90+prev[2]=90+120=210
           curr[3]=210

  prev[3]=210 → return 210 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × 4 × 3)` = `O(n)` | Same triple loop |
| **Space** | `O(4) + O(4)` = `O(1)` | Two vectors of fixed size 4 |

---

## 7. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(3ⁿ)` | `O(n)` | Direct try all tasks, recurse |
| **Memoization** | `O(n × 12)` = `O(n)` | `O(4n + n)` | Cache `dp[day][prevTask]` |
| **Tabulation** | `O(n × 12)` = `O(n)` | `O(4n)` | Fill 2D table bottom-up |
| **Space Optimized** | `O(n × 12)` = `O(n)` | `O(8)` = `O(1)` | One row at a time |

---

## 8. The 2D DP Pattern

This problem introduces the concept of **2D state DP** where two parameters change:

```
1D DP (Climbing Stairs, House Robber):
  State: f(index)
  Depends on: f(index-1), f(index-2)

2D DP (Ninja Training):
  State: f(day, prevTask)
  Depends on: f(day-1, task) for all valid tasks
```

The pattern:
- If you have **two changing variables** → likely 2D DP
- If you have **one changing variable** → likely 1D DP
- Space optimization: reduce `dp[day][]` to `prev[]` when each row only depends on the previous row

This 2D DP pattern appears in many problems: grid DP, string DP, stock buying, and more.
