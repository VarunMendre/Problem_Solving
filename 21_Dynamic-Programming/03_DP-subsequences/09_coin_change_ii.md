# Coin Change II — Count Ways to Make Change

---

## 1. Problem Statement

Given an array of coin denominations `coins[]` and an integer `amount`, count the **number of distinct combinations** to make up `amount`. Each coin can be used **unlimited times** (unbounded). Order does NOT matter — `[1,2]` and `[2,1]` are the same combination.

```
coins = [1, 2, 5], amount = 5

Ways:
  5         (one 5-coin)
  2+2+1     (two 2s and one 1)
  2+1+1+1   (one 2 and three 1s)
  1+1+1+1+1 (five 1s)

Answer: 4
```

---

## 2. Coin Change I vs Coin Change II

| | Coin Change I (Min Coins) | Coin Change II (Count Ways) |
|---|---|---|
| **Goal** | Minimize coin count | Count combinations |
| **Operator** | `min` | `+` (add) |
| **Impossible sentinel** | `1e9` | `0` (no ways) |
| **Base case** | `target/coins[0]` if divisible | `1` if divisible, `0` otherwise |
| **Take branch** | `1 + fn(ind, target-coin)` | `fn(ind, target-coin)` (no +1) |

Both are **unbounded** — the `take` branch stays at the same `ind`.

---

## 3. Base Case Deep Dive

```cpp
if(target == 0)
    return 1;   // ← found a valid combination (empty addition = one way)

if(ind == 0) {
    if(target % coins[0] == 0) {
        if((target / coins[0]) * coins[0] == target)
            return 1;
    } else {
        return 0;
    }
}
```

### `target == 0` → return 1

When target becomes 0, we've found exactly one valid combination (the set of coins chosen so far). Return 1 to count it.

This base case must come BEFORE the `ind == 0` check — if both `target == 0` AND `ind == 0`, `target == 0` fires first and returns 1 correctly.

### `ind == 0` — Only Coin `coins[0]` Available

With only one coin type, the target is achievable in exactly 1 way if `target` is a multiple of `coins[0]`, otherwise 0 ways.

```cpp
if(target % coins[0] == 0)
    return 1;   // exactly one way: use target/coins[0] copies
else
    return 0;   // impossible
```

**The redundant double check:**
```cpp
if(target % coins[0] == 0) {
    if((target / coins[0]) * coins[0] == target)   // ← this is always true when % == 0!
        return 1;
}
```

`(target / coins[0]) * coins[0] == target` is identical to `target % coins[0] == 0` in integer math. The inner check is redundant — it always evaluates to `true` when the outer check passes. A simplified version:

```cpp
if(ind == 0)
    return (target % coins[0] == 0) ? 1 : 0;
```

---

## 4. Why `take = f(ind, ...)` Not `f(ind-1, ...)`

Just like Coin Change I — coins are unbounded. After taking `coins[ind]` once, you can take it again → stay at `ind`.

```
0/1 style:  take = f(ind-1, target - coins[ind])  ← can't reuse
Unbounded:  take = f(ind,   target - coins[ind])  ← can reuse ✅
```

---

## 5. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int f(int ind, int target, vector<int>& coins) {
        if(target == 0) return 1;

        if(ind == 0)
            return (target % coins[0] == 0) ? 1 : 0;

        int notTake = f(ind-1, target, coins);
        int take    = 0;
        if(coins[ind] <= target)
            take = f(ind, target - coins[ind], coins);

        return notTake + take;
    }

public:
    int change(int amount, vector<int>& coins) {
        return f(coins.size()-1, amount, coins);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(amount^n)` rough | Unbounded take — same ind repeated |
| **Space** | `O(amount)` | Stack depth up to `amount/min_coin` |

---

## 6. Approach 2 — Memoization

```cpp
class Solution {
private:
    int f(int ind, int target, vector<int>& coins, vector<vector<int>>& dp) {
        if(target == 0) return 1;

        if(ind == 0)
            return (target % coins[0] == 0) ? 1 : 0;

        if(dp[ind][target] != -1)
            return dp[ind][target];

        int notTake = f(ind-1, target, coins, dp);
        int take    = 0;
        if(coins[ind] <= target)
            take = f(ind, target - coins[ind], coins, dp);

        return dp[ind][target] = notTake + take;
    }

public:
    int change(int amount, vector<int>& coins) {
        int n = coins.size();
        vector<vector<int>> dp(n, vector<int>(amount+1, -1));
        return f(n-1, amount, coins, dp);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | n×amount states, O(1) each |
| **Space** | `O(n × amount) + O(amount)` | dp + stack |

---

## 7. Approach 3 — Tabulation

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        int n = coins.size();
        vector<vector<int>> dp(n, vector<int>(amount+1, 0));

        // Base: row 0 — only coins[0]
        for(int target = 0; target <= amount; target++)
            dp[0][target] = (target % coins[0] == 0) ? 1 : 0;
        // dp[0][0] = 1 because 0%coins[0]=0 ✅ (one way: take nothing)

        for(int ind = 1; ind < n; ind++) {
            for(int target = 0; target <= amount; target++) {
                int notTake = dp[ind-1][target];
                int take    = 0;
                if(coins[ind] <= target)
                    take = dp[ind][target - coins[ind]];   // same row! (unbounded)

                dp[ind][target] = take + notTake;
            }
        }

        return dp[n-1][amount];
    }
};
```

### Tabulation Trace (coins=[1,2], amount=4)

```
Row 0 (coins[0]=1): dp[0][t] = 1 for all t (t%1==0 always)
  [1, 1, 1, 1, 1]

Row 1 (coins[1]=2):
  t=0: notTake=dp[0][0]=1, take(2>0)=0 → 1
  t=1: notTake=dp[0][1]=1, take(2>1)=0 → 1
  t=2: notTake=dp[0][2]=1, take=dp[1][0]=1 → 2
  t=3: notTake=dp[0][3]=1, take=dp[1][1]=1 → 2
  t=4: notTake=dp[0][4]=1, take=dp[1][2]=2 → 3

Row 1: [1, 1, 2, 2, 3]

dp[1][4] = 3 ✅
Verify: {1,1,1,1}, {1,1,2}, {2,2} = 3 combinations
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | Two nested loops |
| **Space** | `O(n × amount)` | 2D dp array |

---

## 8. Approach 4 — Space Optimization

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        int n = coins.size();
        vector<long long> prev(amount+1, 0);

        // Base: only coins[0]
        for(int target = 0; target <= amount; target++)
            if(target % coins[0] == 0)
                prev[target] = 1;

        for(int ind = 1; ind < n; ind++) {
            vector<long long> cur(amount+1, 0);

            for(int target = 0; target <= amount; target++) {
                long long notTake = prev[target];
                long long take    = 0;
                if(coins[ind] <= target)
                    take = cur[target - coins[ind]];   // cur, not prev! (unbounded)

                cur[target] = min((long long)INT_MAX, take + notTake);
            }

            prev = cur;
        }

        return (int)prev[amount];
    }
};
```

### Why `long long` and `min(INT_MAX, ...)`?

The number of combinations can be very large — for small coins and large amount, the count can exceed `INT_MAX`. Using `long long` prevents overflow during computation. The `min(INT_MAX, ...)` clamp ensures the result fits back into `int` for the return.

### `cur[target - coins[ind]]` Not `prev[...]]`

```
cur  = currently being filled row (current coin ind)
prev = completed previous row (previous coin ind-1)

take reads cur because: coin ind is unbounded.
dp[ind][target - coins[ind]] = how many ways to reach (target - coin) using coins 0..ind
→ coin ind may have been used again → same row → cur
```

This mirrors the tabulation's `dp[ind][target-coins[ind]]` read.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | Same loops |
| **Space** | `O(2 × amount)` = `O(amount)` | `prev` + `cur` |

---

## 9. All Approaches Compared

| Approach | Time | Space |
|---|---|---|
| **Recursion** | `O(amount^n)` rough | `O(amount)` |
| **Memoization** | `O(n × amount)` | `O(n × amount + amount)` |
| **Tabulation** | `O(n × amount)` | `O(n × amount)` |
| **Space Optimized** | `O(n × amount)` | `O(amount)` |

---

## 10. Coin Change I vs II vs 0/1 Knapsack — Full Comparison

| | 0/1 Knapsack | Coin Change I (Min) | Coin Change II (Count) |
|---|---|---|---|
| **Reuse item?** | No | Yes (unbounded) | Yes (unbounded) |
| **Goal** | Maximize value | Minimize count | Count combinations |
| **Take branch** | `fn(ind-1, cap-wt)` | `1 + fn(ind, target-coin)` | `fn(ind, target-coin)` |
| **Operator** | `max` | `min` | `+` |
| **Space opt take** | `prev[cap-wt]` | `cur[target-coin]` | `cur[target-coin]` |
| **Space opt direction** | Right → Left | Left → Right | Left → Right |
| **Impossible sentinel** | N/A | `1e9` | `0` |
| **Base case** | `val[0]` if fits | `target/coins[0]` | `1` if divisible |
