# Climbing Stairs

---

## 1. Problem Statement

You are climbing a staircase with `n` steps. Each time you can climb either **1 or 2 steps**. In how many **distinct ways** can you climb to the top?

```
n=1: [1]                          → 1 way
n=2: [1+1], [2]                   → 2 ways
n=3: [1+1+1], [1+2], [2+1]        → 3 ways
n=4: [1+1+1+1],[1+1+2],[1+2+1],[2+1+1],[2+2] → 5 ways
```

Notice: 1, 1, 2, 3, 5, 8, ... — this is the **Fibonacci sequence**!

**Why?** To reach step `n`, you either:
- Came from step `n-1` (took 1 step) → `f(n-1)` ways
- Came from step `n-2` (took 2 steps) → `f(n-2)` ways

`f(n) = f(n-1) + f(n-2)`, with `f(0)=f(1)=1`.

---

## 2. The DP Journey — Four Approaches

This problem is the **"Hello World" of Dynamic Programming** — it perfectly demonstrates the evolution from naive recursion to optimized DP.

---

## 3. Approach 1 — Pure Recursion

### Code

```cpp
class Solution {
public:
    int f(int n) {
        if(n == 0) return 1;   // 0 steps left: 1 way (do nothing)
        if(n == 1) return 1;   // 1 step left: 1 way (take 1 step)

        int left  = f(n - 1);  // take 1 step
        int right = f(n - 2);  // take 2 steps

        return left + right;
    }

    int climbStairs(int n) {
        return f(n);
    }
};
```

### Recursion Tree for n=5

```
                    f(5)
                   /    \
              f(4)        f(3)
             /    \      /    \
          f(3)   f(2) f(2)   f(1)
          / \    / \   / \
       f(2) f(1) f(1)f(0) f(1)f(0)
       / \
    f(1) f(0)
```

**f(3) is computed TWICE. f(2) is computed THREE times. f(1) is computed FIVE times.**

This is the fundamental problem — **overlapping subproblems** → exponential redundant computation.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Each call branches into 2 → binary tree of depth n |
| **Space** | `O(n)` | Recursion stack depth = n |

> For `n=50`: 2⁵⁰ ≈ 10¹⁵ operations. Completely infeasible.

---

## 4. Approach 2 — Memoization (Top-Down DP)

### The Idea

**"Don't recompute what you've already computed."**

Add a `dp[]` array. Before computing `f(n)`, check if it's already stored. If yes, return immediately. If no, compute and store before returning.

### Code

```cpp
class Solution {
public:
    int f(int n, vector<int>& dp) {
        if(n == 0) return 1;
        if(n == 1) return 1;

        if(dp[n] != -1)
            return dp[n];   // ← already computed → return instantly

        int left  = f(n - 1, dp);
        int right = f(n - 2, dp);

        return dp[n] = left + right;   // ← store before returning
    }

    int climbStairs(int n) {
        vector<int> dp(n + 1, -1);   // -1 = "not computed yet"
        return f(n, dp);
    }
};
```

### What Changes

```
Without memo:              With memo (n=5):
f(3) computed 3 times      f(3) computed ONCE, then returned from dp[3]
f(2) computed 5 times      f(2) computed ONCE
f(1) computed 8 times      f(1) computed ONCE (base case)

Total calls: O(2ⁿ)         Total calls: O(n)
```

### Visualizing Memoization

```
f(5)
├── f(4)
│   ├── f(3)
│   │   ├── f(2)
│   │   │   ├── f(1) = 1  → dp[1]=1
│   │   │   └── f(0) = 1  → dp[0]=1
│   │   │   → dp[2] = 2
│   │   └── f(1) = 1  (dp[1] already set → instant!)
│   │   → dp[3] = 3
│   └── f(2) = 2  (dp[2] already set → instant!)
│   → dp[4] = 5
└── f(3) = 3  (dp[3] already set → instant!)
→ dp[5] = 8
```

Each value computed EXACTLY once. All subsequent calls → cache hit → `O(1)`.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Each subproblem `f(0)...f(n)` computed once |
| **Space** | `O(n)` + `O(n)` | `dp[]` array + recursion stack |
| **Total Space** | `O(2n)` = `O(n)` | |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

### The Idea

**"Build the solution from the ground up, no recursion."**

Instead of top-down (start at `f(n)`, recurse down to base cases), go **bottom-up**: start at base cases `dp[0]` and `dp[1]`, fill upward to `dp[n]`.

### Why Bottom-Up?

Memoization has two hidden costs:
1. **Recursion stack:** `O(n)` stack frames — risk of stack overflow for large `n`
2. **Function call overhead:** Each recursive call has OS-level cost

Tabulation eliminates both — it's pure iteration.

### Code

```cpp
class Solution {
public:
    int climbStairs(int n) {
        vector<int> dp(n + 1, -1);

        dp[0] = 1;   // base case: 0 steps → 1 way
        dp[1] = 1;   // base case: 1 step → 1 way

        for(int i = 2; i <= n; i++) {
            dp[i] = dp[i-1] + dp[i-2];   // build forward
        }

        return dp[n];
    }
};
```

### Filling the Table (n=6)

```
i:    0    1    2    3    4    5    6
dp:   1    1    2    3    5    8   13

dp[2] = dp[1] + dp[0] = 1+1 = 2
dp[3] = dp[2] + dp[1] = 2+1 = 3
dp[4] = dp[3] + dp[2] = 3+2 = 5
dp[5] = dp[4] + dp[3] = 5+3 = 8
dp[6] = dp[5] + dp[4] = 8+5 = 13
```

Each value depends only on the previous two — built left to right.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | One loop from 2 to n |
| **Space** | `O(n)` | `dp[]` array of size n+1 |

> Same time as memoization. But NO recursion stack → better in practice (no stack overflow risk).

---

## 6. Approach 4 — Space-Optimized (Two Variables)

### The Key Observation

```
dp[i] = dp[i-1] + dp[i-2]
```

To compute `dp[i]`, we only need **the previous two values** — not the entire array! We're maintaining `n+1` values but only ever using 2 at a time.

**Eliminate the array. Keep only two variables.**

### Code

```cpp
class Solution {
public:
    int climbStairs(int n) {
        int prev1 = 1;   // dp[1] (or dp[i-1])
        int prev2 = 1;   // dp[0] (or dp[i-2])

        for(int i = 2; i <= n; i++) {
            int currentI = prev1 + prev2;   // dp[i] = dp[i-1] + dp[i-2]
            prev2 = prev1;                  // slide: dp[i-2] ← dp[i-1]
            prev1 = currentI;               // slide: dp[i-1] ← dp[i]
        }

        return prev1;   // prev1 = dp[n]
    }
};
```

### The "Sliding Window" of Two Variables

```
i=2: currentI = 1+1 = 2,  prev2=1, prev1=2
i=3: currentI = 2+1 = 3,  prev2=2, prev1=3
i=4: currentI = 3+2 = 5,  prev2=3, prev1=5
i=5: currentI = 5+3 = 8,  prev2=5, prev1=8
i=6: currentI = 8+5 = 13, prev2=8, prev1=13

return prev1 = 13 ✅
```

### Visualizing the Slide

```
Before i=2:  prev2=dp[0]=1, prev1=dp[1]=1
After i=2:   prev2=dp[1]=1, prev1=dp[2]=2   ← window slid right
After i=3:   prev2=dp[2]=2, prev1=dp[3]=3
After i=4:   prev2=dp[3]=3, prev1=dp[4]=5
...
After i=n:   prev2=dp[n-1],  prev1=dp[n]    ← answer
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Same loop, same iterations |
| **Space** | `O(1)` | Only two integer variables — no array! |

---

## 7. The DP Evolution — Complete Comparison

| Approach | Time | Space | Key Idea | Limitation |
|---|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` stack | Direct translation of recurrence | Exponential redundancy |
| **Memoization** | `O(n)` | `O(2n)` | Cache results, top-down | Stack overflow risk for huge n |
| **Tabulation** | `O(n)` | `O(n)` | Build bottom-up, no recursion | Uses O(n) array |
| **Space Optimized** | `O(n)` | `O(1)` | Keep only last 2 values | — (optimal!) |

---

## 8. Why This Problem is the Gateway to DP

This problem teaches the **4 key DP concepts** in one shot:

**1. Overlapping Subproblems:**  
`f(3)` appears multiple times in the recursion tree → a sign that DP can help.

**2. Optimal Substructure:**  
The best solution for `f(n)` is built from the best solutions of `f(n-1)` and `f(n-2)` — the sub-solutions don't interfere with each other.

**3. Top-Down vs Bottom-Up:**  
Both memoization (top-down) and tabulation (bottom-up) give the same `O(n)` result — just different traversal directions.

**4. Space Optimization:**  
Recognizing that `dp[i]` depends only on `dp[i-1]` and `dp[i-2]` → compress the array into two variables → `O(1)` space.

These four concepts appear in nearly every DP problem. Master them here, apply everywhere.

---

## 9. Identifying the Pattern

```
f(n) = f(n-1) + f(n-2)     ← Climbing Stairs
f(n) = f(n-1) + f(n-2)     ← Fibonacci Numbers
f(n) = f(n-1) + f(n-2)     ← Count ways to tile a 2×n board
```

All three are literally the same recurrence! DP problems that reduce to `f(n) = f(n-1) + f(n-2)` follow this exact 4-approach journey.
