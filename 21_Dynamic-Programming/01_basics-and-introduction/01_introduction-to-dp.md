# Introduction to Dynamic Programming (DP)

---

## 1. What is Dynamic Programming?

**Dynamic Programming (DP)** is an algorithmic technique for solving problems by breaking them down into smaller **overlapping subproblems**, solving each subproblem only **once**, and **storing** its result so it can be reused later instead of recomputed.

The term "dynamic programming" has nothing to do with dynamic memory or programming languages — it's just a historical name. Think of it as:

> **DP = Recursion + Remembering answers you already computed (caching/storing)**

A problem is a good candidate for DP if it has two properties:

1. **Optimal Substructure**
   The optimal solution to the problem can be constructed from optimal solutions of its subproblems.
   Example: `fib(n) = fib(n-1) + fib(n-2)` — the answer for `n` depends directly on answers for smaller `n`.

2. **Overlapping Subproblems**
   The same subproblems are solved again and again if you use plain recursion.
   Example: Computing `fib(5)` calls `fib(3)` twice, `fib(2)` three times, etc.

If a problem has optimal substructure but **no** overlapping subproblems (e.g., simple binary search, merge sort), plain **Divide and Conquer** is enough — DP won't give any extra benefit.

---

## 2. Why is DP Required?

Without DP, many recursive problems recompute the same subproblem exponentially many times.

Take Fibonacci as an example. The plain recursive recursion tree for `fib(5)` looks like this:

```
                              f(5)
                          /          \
                      f(4)            f(3)
                    /      \          /    \
                f(3)       f(2)    f(2)    f(1)
               /    \      /   \   /   \
            f(2)   f(1) f(1) f(0) f(1) f(0)
           /    \
        f(1)   f(0)
```

Notice `f(3)` is computed **twice**, `f(2)` is computed **three times**, `f(1)` is computed **five times**. As `n` grows, this blows up **exponentially**:

- Time Complexity (plain recursion) = **O(2^n)**
- Space Complexity = **O(n)** (recursion stack depth)

For `n = 50`, this would take billions of operations — completely impractical.

**DP fixes this** by storing the result of each subproblem the first time it's computed, so the next time the same subproblem is needed, it's fetched in O(1) instead of being recomputed. This brings Fibonacci down from **O(2^n) → O(n)**.

**In short, DP is required to:**
- Avoid redundant recomputation of overlapping subproblems
- Convert exponential-time brute-force recursive solutions into polynomial-time solutions
- Systematically solve optimization problems (min/max/count/feasibility) built from smaller decisions

---

## 3. Types of DP

There are broadly two implementation styles of DP, plus one further optimization step:

| Type | Also Called | Direction | How it Works |
|---|---|---|---|
| **1. Memoization** | Top-Down DP | Starts from the original problem `n` and goes down to base cases | Recursion + a lookup table (array/map) to cache results |
| **2. Tabulation** | Bottom-Up DP | Starts from base cases and builds up to `n` | Iterative loop filling a DP table in order |
| **3. Space-Optimized DP** | Bottom-Up with rolling variables | Same direction as tabulation | Instead of a full array, keep only the last few states needed in variables |

Beyond these implementation styles, DP problems are also commonly **categorized by structure**, such as:

- **1D DP** — state depends on one index (e.g., Fibonacci, Climbing Stairs, House Robber)
- **2D DP** — state depends on two indices (e.g., LCS, Edit Distance, Grid Unique Paths)
- **DP on Subsets / Bitmask DP** — state includes a subset of elements (e.g., Travelling Salesman Problem)
- **DP on Trees** — state computed via DFS on a tree (e.g., Diameter of Tree, House Robber III)
- **DP on Intervals** — state represents a range `[i, j]` (e.g., Matrix Chain Multiplication, Burst Balloons)
- **Knapsack-type DP** — include/exclude decisions with a capacity constraint (0/1 Knapsack, Unbounded Knapsack, Subset Sum)
- **DP on Strings** — two-pointer style state over two strings (LCS, Wildcard Matching)
- **Digit DP** — state built digit-by-digit for counting problems over number ranges

---

## 4. When to Use What?

### When to even think of DP?
Ask yourself while solving a recursive problem:
- Am I asked to find a **minimum / maximum / count of ways / true-false feasibility**?
- Does the recursive brute force involve **choices** at each step (take / not take, go left / go right, etc.)?
- Are the **same recursive calls repeating** with the same parameters (overlapping subproblems)?

If yes to these → it's a DP problem.

### Memoization vs Tabulation vs Space Optimization

| Situation | Recommended Approach |
|---|---|
| You already have a working recursive brute-force solution and just want to optimize it quickly | **Memoization** (easiest to convert from recursion, minimal code change) |
| You want to avoid recursion stack overflow for large `n` (e.g., n = 10^6) | **Tabulation** (iterative, no stack overflow risk) |
| You need the fastest possible runtime and want to avoid function call overhead | **Tabulation** |
| The current DP state only depends on the last 1, 2, or a few previous states (not the entire table) | **Space Optimization** (reduce O(n) space to O(1) or O(k)) |
| You need to reconstruct the actual path / actual subset / actual sequence (not just the final answer) | **Tabulation** (Space Optimization won't work here since you lose old rows) |
| The order of subproblem evaluation is complex/unclear | **Memoization** (recursion naturally figures out the right order for you) |

**General advice for beginners:** always solve in this order →
1. Write plain **recursion** (brute force) first.
2. Convert to **Memoization** (top-down DP) — just add a `dp[]` array/cache.
3. Convert to **Tabulation** (bottom-up DP) — remove recursion, use a loop, careful with base cases.
4. Convert to **Space Optimization** — if only a fixed few previous states are needed.

---

## 5. Patterns of DP

### i. Memoization (Top-Down)
- Start with the recursive solution for the **original problem** (`f(n)`).
- Identify the **changing parameters** in the recursive calls — these become the "state" of your DP.
- Create a `dp` array/map sized according to the state, initialized with a sentinel value (commonly `-1`) meaning "not computed yet."
- Before doing the recursive work, **check** if `dp[state]` is already computed. If yes, return it directly.
- Otherwise, compute the result recursively, **store** it in `dp[state]`, then return it.

**Template:**
```cpp
int f(int n, vector<int>& dp) {
    if (/* base case */) return baseValue;
    if (dp[n] != -1) return dp[n];          // already computed → return cached
    return dp[n] = /* recursive relation */; // compute, store, return
}
```

### ii. Tabulation (Bottom-Up)
- Identify the same "state" as memoization, but now think **iteratively**.
- Create a `dp` array and manually fill in the **base cases** first (these are the "seeds").
- Loop from the smallest state up to the target state `n`, filling `dp[i]` using the **same recurrence relation** as memoization, just written iteratively.
- Answer sits at `dp[n]`.

**Template:**
```cpp
int fT(int n, vector<int>& dp) {
    dp[0] = base0;
    dp[1] = base1;
    for (int i = 2; i <= n; i++) {
        dp[i] = /* recurrence relation, using dp[i-1], dp[i-2], etc. */;
    }
    return dp[n];
}
```

### iii. Space Optimization (Rolling Variables)
- Look at the tabulation loop — check **how many previous states** `dp[i]` actually depends on.
- If it only depends on the last `k` states (commonly `k = 1` or `2`), you don't need the whole array — just keep `k` variables.
- After computing the current value, **shift** the variables forward (roll them) for the next iteration.
- This reduces space from **O(n) → O(1)** (or O(k) for k variables).

**Template (when only last 2 states are needed):**
```cpp
int fO(int n) {
    int prev2 = base0, prev = base1;
    for (int i = 2; i <= n; i++) {
        int curr = /* recurrence using prev, prev2 */;
        prev2 = prev;
        prev = curr;
    }
    return prev;
}
```

---

## 6. Example Walkthrough: Nth Fibonacci Number

We will fully explain the 3 given implementations **separately**: Memoization, Tabulation, and Space Optimization.

### Problem & Formula
Find the `n`-th Fibonacci number, defined as:

```
f(0) = 0
f(1) = 1
f(n) = f(n-1) + f(n-2)     for n >= 2
```

This is our **recurrence relation**, and `f(0) = 0`, `f(1) = 1` are our **base cases**.

---

### 6.1 Memoization (Top-Down DP)

```cpp
int f(int n, vector<int>& dp) {
    if(n <= 1)
        return n;

    if(dp[n] != -1)
        return dp[n];

    return dp[n] = f(n-1, dp) + f(n-2, dp);
}
```

**Line-by-line explanation:**
- `if(n <= 1) return n;` → **Base case**. If `n` is 0 or 1, `f(0)=0` and `f(1)=1`, and conveniently `return n` handles both in one line.
- `if(dp[n] != -1) return dp[n];` → **Memo check**. `dp` array was initialized with `-1` meaning "not yet computed." If `dp[n]` already has a real value, we return it immediately instead of recomputing.
- `return dp[n] = f(n-1, dp) + f(n-2, dp);` → **Recursive case**. We calculate `f(n-1) + f(n-2)`, and **store** it into `dp[n]` before returning, so future calls for the same `n` are instant.

**Dry Run for `n = 5`, `dp = [-1,-1,-1,-1,-1,-1]` initially:**

```
f(5, dp)
 dp[5] == -1 → compute f(4) + f(3)
 ├── f(4, dp)
 │    dp[4] == -1 → compute f(3) + f(2)
 │    ├── f(3, dp)
 │    │    dp[3] == -1 → compute f(2) + f(1)
 │    │    ├── f(2, dp)
 │    │    │    dp[2] == -1 → compute f(1) + f(0)
 │    │    │    ├── f(1, dp) → base case → return 1
 │    │    │    └── f(0, dp) → base case → return 0
 │    │    │    dp[2] = 1 + 0 = 1 → return 1
 │    │    └── f(1, dp) → base case → return 1
 │    │    dp[3] = 1 + 1 = 2 → return 2
 │    └── f(2, dp) → dp[2] != -1 (=1) → return 1 directly (CACHE HIT, no recomputation!)
 │    dp[4] = 2 + 1 = 3 → return 3
 └── f(3, dp) → dp[3] != -1 (=2) → return 2 directly (CACHE HIT!)
 dp[5] = 3 + 2 = 5 → return 5

Final dp array: [0, 1, 1, 2, 3, 5]
Answer: f(5) = 5
```

Notice how `f(3)` and `f(2)` were each **computed only once**, and reused via cache hits — this is exactly what avoids the exponential blow-up seen in plain recursion.

**Complexity:**
- **Time Complexity: O(n)** — each state `0` to `n` is computed exactly once (thanks to memoization); each computation does O(1) work.
- **Space Complexity: O(n) + O(n)**
  - O(n) for the `dp` array
  - O(n) for the recursion call stack (depth of recursion goes up to `n` before hitting base cases)

---

### 6.2 Tabulation (Bottom-Up DP)

```cpp
int fT(int n, vector<int>& dp) {
    dp[0] = 0;
    dp[1] = 1;

    for(int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }

    return dp[n];
}
```

**Line-by-line explanation:**
- `dp[0] = 0; dp[1] = 1;` → We directly **seed the base cases** into the table (no recursion needed to figure these out — we already know them).
- `for(int i = 2; i <= n; i++)` → We iterate **forward**, from the smallest unknown state (`i=2`) up to `n`.
- `dp[i] = dp[i-1] + dp[i-2];` → Same recurrence relation as memoization, but now we're **building up** the answer instead of breaking it down.
- `return dp[n];` → By the time the loop ends, `dp[n]` holds the final answer.

**Dry Run for `n = 5`:**

```
dp[0] = 0
dp[1] = 1

i = 2: dp[2] = dp[1] + dp[0] = 1 + 0 = 1
i = 3: dp[3] = dp[2] + dp[1] = 1 + 1 = 2
i = 4: dp[4] = dp[3] + dp[2] = 2 + 1 = 3
i = 5: dp[5] = dp[4] + dp[3] = 3 + 2 = 5

Final dp array: [0, 1, 1, 2, 3, 5]
Answer: dp[5] = 5
```

There's **no recursion tree** here at all — it's a simple straight-line loop filling the table left to right. This is the key structural difference from memoization.

**Complexity:**
- **Time Complexity: O(n)** — a single loop from `2` to `n`, O(1) work per iteration.
- **Space Complexity: O(n)** — only the `dp` array; **no recursion stack** this time (that's the main advantage over memoization — no stack overflow risk for very large `n`, e.g. n = 10^7).

---

### 6.3 Space Optimization

```cpp
int fO(int n) {
    int prev2 = 0;
    int prev = 1;

    for(int i = 2; i <= n; i++) {
        int curr = prev + prev2;
        prev2 = prev;
        prev = curr;
    }

    return prev;
}
```

**Line-by-line explanation:**
- Looking at the tabulation loop, `dp[i]` only ever depends on `dp[i-1]` and `dp[i-2]` — **never anything further back**. So we don't need to store the *entire* array; we only need to remember the **last two values**.
- `prev2 = 0;` → represents `dp[0]` (i.e., `dp[i-2]` initially, when `i=2`)
- `prev = 1;` → represents `dp[1]` (i.e., `dp[i-1]` initially)
- Inside the loop:
  - `int curr = prev + prev2;` → this is exactly `dp[i] = dp[i-1] + dp[i-2]`, just using variables instead of array indices.
  - `prev2 = prev;` → **shift**: what was `dp[i-1]` becomes the new `dp[i-2]` for the next iteration.
  - `prev = curr;` → **shift**: what was just computed (`dp[i]`) becomes the new `dp[i-1]` for the next iteration.
- After the loop, `prev` holds the most recently computed value, which is `dp[n]`.

**Dry Run for `n = 5`:**

```
Initial: prev2 = 0 (dp[0]),  prev = 1 (dp[1])

i = 2: curr = prev + prev2 = 1 + 0 = 1   → prev2 = 1, prev = 1
i = 3: curr = prev + prev2 = 1 + 1 = 2   → prev2 = 1, prev = 2
i = 4: curr = prev + prev2 = 2 + 1 = 3   → prev2 = 2, prev = 3
i = 5: curr = prev + prev2 = 3 + 2 = 5   → prev2 = 3, prev = 5

Loop ends (i becomes 6 > n=5)
Return prev = 5
```

Same final answer (`5`), but observe: at no point did we ever store an array of size `n+1` — just **two integer variables** getting recycled every iteration.

**Complexity:**
- **Time Complexity: O(n)** — still a single loop from `2` to `n`.
- **Space Complexity: O(1)** — only a constant number of variables (`prev`, `prev2`, `curr`) regardless of how large `n` is. This is the whole point of this optimization step — from O(n) array space down to O(1).

---

## 7. Summary Comparison Table

| Approach | Time Complexity | Space Complexity | Extra Stack Space | Can Reconstruct Path? | Notes |
|---|---|---|---|---|---|
| Plain Recursion | O(2^n) | O(n) | O(n) | Yes | Impractical for large n |
| Memoization (Top-Down) | O(n) | O(n) [dp array] + O(n) [stack] | Yes | Yes | Easiest to derive from recursion |
| Tabulation (Bottom-Up) | O(n) | O(n) [dp array] | No | Yes | No stack overflow risk |
| Space Optimized | O(n) | O(1) | No | No (values overwritten) | Best when only limited history is needed |

---

## 8. Key Takeaways

- DP is just **"recursion with memory"** — solve each unique subproblem once, remember it, reuse it.
- Always identify the **state** (changing parameters) and the **recurrence relation** first — everything else follows from that.
- Progression path: **Recursion → Memoization → Tabulation → Space Optimization**.
- Space Optimization is only possible when the current state depends on a **small, fixed window** of previous states — not the entire history.
- If you need to **reconstruct the actual solution** (not just an optimal value), avoid space optimization and stick to tabulation with the full table.
