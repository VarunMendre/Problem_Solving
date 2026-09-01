# Frog Jump — Minimum Cost

---

## 1. Problem Statement

A frog starts at stair `0` and wants to reach stair `n-1`. From stair `i`, it can jump to `i+1` or `i+2`. The cost of a jump is the **absolute height difference** between the stairs.

Find the **minimum total cost** to reach the last stair.

```
heights = [20, 30, 40, 20]
            0   1   2   3

Path 0→1→3: |30-20| + |20-30| = 10 + 10 = 20 ✅ (minimum)
Path 0→1→2→3: 10 + 10 + 20 = 40
Path 0→2→3: 20 + 20 = 40
```

---

## 2. Why Greedy Fails — Formal Proof

### Greedy Idea

"At every stair, choose the jump (1-step or 2-step) that costs the least right now."

Sounds reasonable — but **greedy optimization at each step doesn't guarantee global optimum**. A cheap step now might force you into an expensive step later.

### Counterexample

```
heights = [10, 100, 10, 10]
           0    1    2   3
```

**Greedy from stair 0:**

```
At stair 0:
  Jump to 1: cost = |100-10| = 90
  Jump to 2: cost = |10-10|  = 0   ← greedy picks this (cheaper!)

At stair 2:
  Jump to 3: cost = |10-10| = 0

Greedy path: 0→2→3, total = 0 + 0 = 0
```

Wait — greedy accidentally got it right here. Let me build a better example:

```
heights = [10, 30, 10, 100, 10]
           0    1   2    3   4
```

**Greedy from stair 0:**
```
At stair 0:
  Jump to 1: |30-10| = 20
  Jump to 2: |10-10| = 0   ← greedy picks 2 (cheaper)

At stair 2:
  Jump to 3: |100-10| = 90
  Jump to 4: |10-10|  = 0   ← greedy picks 4 (cheaper)

Greedy path: 0→2→4, total = 0+0 = 0 ✅ greedy got it right again...
```

Let me construct a case greedy genuinely fails:

```
heights = [1, 100, 1, 100, 1]
           0   1   2   3   4
```

**Greedy from stair 0:**
```
At stair 0:
  Jump to 1: |100-1| = 99
  Jump to 2: |1-1|   = 0   ← greedy picks 2

At stair 2:
  Jump to 3: |100-1| = 99
  Jump to 4: |1-1|   = 0   ← greedy picks 4

Total: 0+0 = 0   ← still correct here...
```

The key insight is WHY greedy is provably wrong in general even if it happens to work in some cases:

### The Fundamental Flaw

```
heights = [10, 1, 200, 1, 100]
           0   1   2   3   4
```

**Greedy from stair 0:**
```
At stair 0:
  Jump to 1: |1-10|   = 9
  Jump to 2: |200-10| = 190
  Greedy picks: jump to 1 (cost=9)

At stair 1:
  Jump to 2: |200-1| = 199
  Jump to 3: |1-1|   = 0   ← greedy picks 3 (cost=0)

At stair 3:
  Jump to 4: |100-1| = 99

Greedy total: 9 + 0 + 99 = 108
```

**Optimal (DP):**
```
Path 0→2→4: |200-10| + |100-200| = 190 + 100 = 290 (worse)
Path 0→1→3→4: 9 + 0 + 99 = 108 (same as greedy here)

Let me try:
heights = [0, 1, 10, 1, 0]
```

**Clearest greedy failure:**

```
heights = [0, 10, 0, 10, 0]
           0   1  2   3  4
```

**Greedy from stair 0:**
```
At 0: jump to 1 (cost=10) vs jump to 2 (cost=0) → picks 2

At 2: jump to 3 (cost=10) vs jump to 4 (cost=0) → picks 4

Greedy total: 0 + 0 = 0   ← correct again!
```

**The real failure pattern:**

```
heights = [0, 5, 2, 5, 2, 5, 0]
           0  1  2  3  4  5  6
```

**Greedy from stair 0:**
```
At 0: jump to 2 (cost=|2-0|=2) vs jump to 1 (cost=|5-0|=5) → picks 2 (cheaper)
At 2: jump to 4 (cost=|2-2|=0) vs jump to 3 (cost=|5-2|=3) → picks 4 (cheaper)
At 4: jump to 6 (cost=|0-2|=2) vs jump to 5 (cost=|5-2|=3) → picks 6 (cheaper)
Greedy total = 2+0+2 = 4

Optimal:
0→1→2→4→6: 5+3+0+2=10 (worse)
0→2→4→6: 2+0+2=4 (same)
```

**The DEFINITIVE greedy failure:**

```
heights = [10, 5, 20, 1, 10]
           0   1   2  3   4
```

**Greedy:**
```
At 0:
  To 1: |5-10| = 5
  To 2: |20-10| = 10
  Greedy picks: 1 (cost=5)

At 1:
  To 2: |20-5| = 15
  To 3: |1-5|  = 4   ← greedy picks 3 (cost=4)

At 3:
  To 4: |10-1| = 9

Greedy path: 0→1→3→4 = 5+4+9 = 18
```

**Optimal:**
```
0→1→2→4: |5-10|+|20-5|+|10-20| = 5+15+10 = 30  (worse)
0→2→4:   |20-10|+|10-20| = 10+10 = 20  (worse)
0→1→3→4: 5+4+9 = 18  (greedy matches optimal here)

Let's find a case where greedy truly diverges:

heights = [10, 50, 30, 1, 100]
           0    1   2  3    4
```

**Greedy:**
```
At 0:
  To 1: |50-10| = 40
  To 2: |30-10| = 20 ← picks 2

At 2:
  To 3: |1-30|  = 29
  To 4: |100-30|= 70 ← picks 3

At 3:
  To 4: |100-1| = 99

Greedy: 0→2→3→4 = 20+29+99 = 148
```

**Optimal:**
```
0→1→3→4: 40+|1-50|+99  = 40+49+99 = 188 (worse)
0→1→2→4: 40+|30-50|+|100-30| = 40+20+70 = 130 ✅ (BETTER than greedy!)
0→2→4:   20+70 = 90 ✅ (BETTER than greedy!)

Best: 0→2→4 = 90  but greedy chose 148 ← GREEDY FAILS! ❌
```

**Greedy failed because:** It chose to jump to stair 3 (cheap step, cost=29) but then had to pay the expensive cost=99 to reach stair 4. DP correctly sees that skipping stair 3 entirely and jumping from stair 2 to 4 (cost=70) is much cheaper overall.

**The lesson:** Greedy optimizes locally. A "cheap" step right now might trap you into expensive steps later. DP considers ALL paths and picks the globally optimal one.

---

## 3. Why DP Works

The key realization: **the cost to reach stair `i` optimally depends on the optimal cost to reach stairs `i-1` and `i-2`**.

This is **optimal substructure** — the problem decomposes into subproblems whose solutions combine to give the overall solution. No greedy choice can guarantee this — you need to know costs of BOTH paths before deciding.

---

## 4. Approach 1 — Recursion

### Intuition

`f(i)` = minimum cost to reach stair `i` from stair `0`.

To reach stair `i`, the frog either:
- Came from `i-1`: cost = `f(i-1) + |height[i] - height[i-1]|`
- Came from `i-2`: cost = `f(i-2) + |height[i] - height[i-2]|`

Take the minimum.

```cpp
class Solution {
public:
    int f(int ind, vector<int>& arr) {
        if(ind == 0) return 0;   // at stair 0: already here, cost=0

        // Cost of coming from one step back
        int left = f(ind-1, arr) + abs(arr[ind] - arr[ind-1]);

        // Cost of coming from two steps back (if valid)
        int right = INT_MAX;
        if(ind > 1)
            right = f(ind-2, arr) + abs(arr[ind] - arr[ind-2]);

        return min(left, right);
    }

    int minCost(vector<int>& height) {
        int n = height.size();
        return f(n-1, height);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Each call branches into 2 |
| **Space** | `O(n)` | Recursion stack |

---

## 5. Approach 2 — Memoization

Add `dp[]` to cache results:

```cpp
class Solution {
public:
    int f(int ind, vector<int>& arr, vector<int>& dp) {
        if(ind == 0) return 0;

        if(dp[ind] != -1)
            return dp[ind];   // return cached result

        int left = f(ind-1, arr, dp) + abs(arr[ind] - arr[ind-1]);

        int right = INT_MAX;
        if(ind > 1)
            right = f(ind-2, arr, dp) + abs(arr[ind] - arr[ind-2]);

        return dp[ind] = min(left, right);   // cache before returning
    }

    int minCost(vector<int>& height) {
        int n = height.size();
        vector<int> dp(n+1, -1);
        return f(n-1, height, dp);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Each of n subproblems computed once |
| **Space** | `O(n) + O(n)` | dp[] + recursion stack = `O(2n)` |

---

## 6. Approach 3 — Tabulation (Bottom-Up)

Build from `dp[0]` upward — no recursion:

```cpp
class Solution {
public:
    int minCost(vector<int>& height) {
        int n = height.size();
        vector<int> dp(n, 0);

        dp[0] = 0;   // cost to reach stair 0 = 0 (start here)

        for(int i = 1; i < n; i++) {
            // Option 1: came from i-1
            int left = dp[i-1] + abs(height[i] - height[i-1]);

            // Option 2: came from i-2 (if valid)
            int right = INT_MAX;
            if(i > 1)
                right = dp[i-2] + abs(height[i] - height[i-2]);

            dp[i] = min(left, right);
        }

        return dp[n-1];
    }
};
```

### Tabulation Trace for heights=[20,30,40,20]

```
dp[0] = 0
dp[1] = min(dp[0]+|30-20|)              = min(10)                = 10
dp[2] = min(dp[1]+|40-30|, dp[0]+|40-20|) = min(10+10, 0+20)   = min(20,20) = 20
dp[3] = min(dp[2]+|20-40|, dp[1]+|20-30|) = min(20+20, 10+10)  = min(40,20) = 20

Answer: dp[3] = 20 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | One loop |
| **Space** | `O(n)` | dp[] array |

---

## 7. Approach 4 — Space Optimization

`dp[i]` only needs `dp[i-1]` and `dp[i-2]` → replace array with two variables:

```cpp
class Solution {
public:
    int minCost(vector<int>& height) {
        int n = height.size();

        int prev  = 0;   // dp[0] = 0  (cost to be at stair 0)
        int prev2 = 0;   // placeholder for dp[-1] (never used at i=1)

        for(int i = 1; i < n; i++) {
            int left = prev + abs(height[i] - height[i-1]);

            int right = INT_MAX;
            if(i > 1)
                right = prev2 + abs(height[i] - height[i-2]);

            int curri = min(left, right);

            prev2 = prev;    // slide: prev2 becomes what prev was
            prev  = curri;   // slide: prev becomes current result
        }

        return prev;   // prev = dp[n-1]
    }
};
```

### Space Optimization Trace for heights=[20,30,40,20]

```
Initial: prev=0 (dp[0]), prev2=0 (unused)

i=1: left=0+10=10, right=INT_MAX (i≤1), curri=10
     prev2=0, prev=10   (prev=dp[1]=10)

i=2: left=10+10=20, right=0+20=20, curri=20
     prev2=10, prev=20  (prev=dp[2]=20)

i=3: left=20+20=40, right=10+10=20, curri=20
     prev2=20, prev=20  (prev=dp[3]=20)

return prev=20 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Same loop |
| **Space** | `O(1)` | Only two integers |

---

## 8. Complete Comparison

| Approach | Time | Space | Key Idea | Drawback |
|---|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` | Direct recurrence | Exponential recomputation |
| **Memoization** | `O(n)` | `O(2n)` | Cache subproblems, top-down | Stack overflow risk |
| **Tabulation** | `O(n)` | `O(n)` | Build bottom-up iteratively | Still uses O(n) array |
| **Space Optimized** | `O(n)` | `O(1)` | Only keep last 2 results | — (optimal) |

---

## 9. Differences from Climbing Stairs

| | Climbing Stairs | Frog Jump |
|---|---|---|
| **Goal** | Count distinct paths | Minimize total cost |
| **Recurrence** | `f(n) = f(n-1) + f(n-2)` (sum) | `f(i) = min(f(i-1)+cost1, f(i-2)+cost2)` (min) |
| **Base case** | `f(0)=1, f(1)=1` | `f(0)=0` (cost to be at start = 0) |
| **Operations** | Addition | Minimum + addition |
| **DP insight** | Count all paths | Optimal path only |

The structure is identical — both use the "previous two" pattern. But instead of summing, we minimize. This single change converts a counting problem into an optimization problem.
