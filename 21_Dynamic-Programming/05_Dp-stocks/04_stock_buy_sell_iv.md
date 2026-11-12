# Best Time to Buy and Sell Stock IV

Problem link: [LeetCode 188: Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

---

## 1. Problem Statement

You are given an integer `k` and an array `prices` where `prices[i]` is the price of a stock on day `i`. You may complete **at most `k` transactions** (a transaction = one buy followed by one sell). You may not hold more than one share at a time. Return the maximum profit.

```
k = 2, prices = [3, 2, 6, 5, 0, 3]

Buy at 2, sell at 6  → profit 4
Buy at 0, sell at 3  → profit 3
Total = 7

Answer: 7
```

```
k = 2, prices = [2, 4, 1]      → 2    (buy 2, sell 4; a second transaction would lose money)
k = 1, prices = [3, 2, 6, 5, 0, 3]  → 4    (buy 2, sell 6)
```

---

## 2. From Stock III to Stock IV: One Change

Stock III hard-coded the limit `2`. Stock IV makes it a parameter:

```
Stock III:  f(ind, buy, cap)   with cap ≤ 2
Stock IV:   f(ind, buy, cap)   with cap ≤ k
```

The recurrence, base cases and transitions are **identical**. Only the size of the `cap` dimension changes from `3` to `k + 1`.

```
f(ind, buy, k) = maximum profit from day `ind` onward, where
    buy = 1 → we hold no stock and may buy
    buy = 0 → we hold a stock and may sell
    k       → number of transactions still allowed
```

| State | Choice | Profit | Next state |
|---|---|---|---|
| `buy = 1` | Buy | `-prices[ind]` | `f(ind+1, 0, k)` |
| `buy = 1` | Skip | `0` | `f(ind+1, 1, k)` |
| `buy = 0` | Sell | `+prices[ind]` | `f(ind+1, 1, k - 1)` |
| `buy = 0` | Hold | `0` | `f(ind+1, 0, k)` |

---

## 3. Approach 1: Recursion

```cpp
class Solution {
private:
    int f(int ind, int buy, int k, int n, vector<int>& prices) {
        if (k == 0)
            return 0;
        if (ind == n)
            return 0;

        int profit = 0;

        if (buy) {
            profit = max(-prices[ind] + f(ind + 1, 0, k, n, prices),      // buy
                          0           + f(ind + 1, 1, k, n, prices));     // skip
        } else {
            profit = max( prices[ind] + f(ind + 1, 1, k - 1, n, prices),  // sell (k-1)
                          0           + f(ind + 1, 0, k, n, prices));     // hold
        }

        return profit;
    }

public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();
        return f(0, 1, k, n, prices);
    }
};
```

### Story Points

---

**Story Point 1: "A transaction is counted on the SELL, not on the buy"**

A transaction is a buy **and** a sell. The counter `k` drops only when the pair is completed (`f(ind+1, 1, k - 1)` in the sell branch). A buy does not touch `k`.

---

**Story Point 2: "Why decrement on the sell: it keeps the `k == 0` base case simple"**

If `k` dropped on the **buy**, we could reach `k == 0` while still **holding a stock that needs to be sold**. Then `return 0` at `k == 0` would throw away a pending sale (a stock held and never sold is worth `-price`, not `0`). By decrementing on the sell, `k == 0` happens exactly when the last transaction has finished, so there is nothing left to do and `return 0` is correct.

---

**Story Point 3: "`k == 0` is checked before `ind == n`: both return 0, so the order is only a micro-optimization"**

Either check alone ends the recursion with `0`. Checking `k == 0` first just cuts off recursion the moment the last allowed sale is done, without walking through the rest of the days.

---

**Story Point 4: "`buy` is a permission flag, not an inventory count"**

`buy = 1` means "we hold nothing, so the next action is a buy"; `buy = 0` means "we hold one share, so the next action is a sell". Because we hold at most one share, one bit is enough.

---

**Story Point 5: "'At most k' is automatic"**

We never **have** to use all `k` transactions. At every state the **skip/hold** branch is available, so the `max` can pick a plan with fewer transactions when extra ones would lose money. No special handling for "unused" transactions is needed.

---

**Story Point 6: "k only prunes the recursion, it does not remove the exponential"**

At every day there are still two choices, so the recursion has `O(2ⁿ)` calls in the worst case. `k` can cut branches short (via `k == 0`), but it does not change the exponential bound.

---

**Story Point 7: "Stock II is the `k = ∞` special case; Stock III is `k = 2`"**

With `k` large enough, the `cap` dimension never constrains anything and the problem collapses to Stock II (see section 8).

---

### Trace: `k = 2`, `prices = [1, 4, 2, 5]` (answer 6)

```
f(0, buy, k=2)   buy at 1   → -1
f(1, sell, k=2)  sell at 4  → +4,  k becomes 1
f(2, buy, k=1)   buy at 2   → -2
f(3, sell, k=1)  sell at 5  → +5,  k becomes 0
f(4, ..., k=0)   base case  → 0

Total = -1 + 4 - 2 + 5 = 6 ✅
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(2ⁿ)` |
| **Space** | `O(n)` recursion depth |

---

## 4. Approach 2: Memoization

The state `(ind, buy, k)` repeats across branches. Cache it in a 3D table:

```cpp
class Solution {
private:
    int f(int ind, int buy, int k, int n, vector<int>& prices,
          vector<vector<vector<int>>>& dp) {
        if (k == 0) return 0;
        if (ind == n) return 0;

        if (dp[ind][buy][k] != -1)
            return dp[ind][buy][k];

        int profit = 0;

        if (buy) {
            profit = max(-prices[ind] + f(ind + 1, 0, k, n, prices, dp),
                          0           + f(ind + 1, 1, k, n, prices, dp));
        } else {
            profit = max( prices[ind] + f(ind + 1, 1, k - 1, n, prices, dp),
                          0           + f(ind + 1, 0, k, n, prices, dp));
        }

        return dp[ind][buy][k] = profit;
    }

public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();
        vector<vector<vector<int>>> dp(
            n, vector<vector<int>>(2, vector<int>(k + 1, -1)));   // k+1 so index k is valid

        return f(0, 1, k, n, prices, dp);
    }
};
```

The third dimension has size `k + 1` because `k` ranges over `0..k`. Index `0` is never read (the base case returns first), but it keeps the indexing natural.

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × 2 × k)` = `O(n·k)` |
| **Space** | `O(n × 2 × (k+1))` dp + `O(n)` stack |

---

## 5. Approach 3: Tabulation

`f(ind, ...)` depends on `f(ind + 1, ...)`, so fill from the last day backwards.

### How the base cases are stored

```cpp
vector<vector<vector<int>>> dp(n + 1, vector<vector<int>>(2, vector<int>(k + 1, 0)));
```

- **`ind == n`:** layer `dp[n][*][*] = 0`
- **`cap == 0`:** layer `dp[*][*][0] = 0`

Both come free from zero initialization, which is why the `cap` loop starts at `1`. In the sell branch, `cap - 1` may be `0` and lands in the zero layer, which is exactly the `k == 0 → return 0` base case.

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();
        vector<vector<vector<int>>> dp(
            n + 1, vector<vector<int>>(2, vector<int>(k + 1, 0)));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= k; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit = max(-prices[ind] + dp[ind + 1][0][cap],
                                      0           + dp[ind + 1][1][cap]);
                    } else {
                        profit = max( prices[ind] + dp[ind + 1][1][cap - 1],
                                      0           + dp[ind + 1][0][cap]);
                    }

                    dp[ind][buy][cap] = profit;
                }
            }
        }

        return dp[0][1][k];
    }
};
```

### Table for `k = 2`, `prices = [3, 2, 6, 5, 0, 3]`

```
ind  price | dp[ind][1][1]  dp[ind][0][1]  dp[ind][1][2]  dp[ind][0][2]
              (buy,cap=1)    (sell,cap=1)   (buy,cap=2)    (sell,cap=2)
 6     -   |      0              0              0              0        ← base row
 5     3   |      0              3              0              3
 4     0   |      3              3              3              3
 3     5   |      3              5              3              8
 2     6   |      3              6              3              9
 1     2   |      4              6              7              9
 0     3   |      4              6              7              9

Answer = dp[0][1][2] = 7 ✅     (and dp[0][1][1] = 4 is the answer for k = 1)
```

Spot checks:

```
dp[1][1][2] = max( -2 + dp[2][0][2], dp[2][1][2] ) = max(-2 + 9, 3) = 7
              buy at 2 on day 1, then sell at 6 and do one more transaction (0 → 3)

dp[3][0][2] = max( 5 + dp[4][1][1], dp[4][0][2] ) = max(5 + 3, 3) = 8
              sell at 5, then use the remaining 1 transaction (buy 0, sell 3)
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × 2 × k)` = `O(n·k)` |
| **Space** | `O(n × 2 × (k+1))` |

---

## 6. Approach 4: Space Optimization (Two Layers)

Layer `ind` depends only on layer `ind + 1`:

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();

        vector<vector<int>> prev(2, vector<int>(k + 1, 0));   // layer ind+1
        vector<vector<int>> cur(2, vector<int>(k + 1, 0));    // layer ind

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= k; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit = max(-prices[ind] + prev[0][cap], 0 + prev[1][cap]);
                    } else {
                        profit = max( prices[ind] + prev[1][cap - 1], 0 + prev[0][cap]);
                    }

                    cur[buy][cap] = profit;
                }
            }
            prev = cur;
        }

        return prev[1][k];
    }
};
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n·k)` |
| **Space** | `O(2 × (k+1))` = `O(k)` |

---

## 7. Approach 5: In-Place Update on One 2×(k+1) Array

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();

        vector<vector<int>> prev(2, vector<int>(k + 1, 0));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= k; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit = max(-prices[ind] + prev[0][cap], 0 + prev[1][cap]);
                    } else {
                        profit = max( prices[ind] + prev[1][cap - 1], 0 + prev[0][cap]);
                    }

                    prev[buy][cap] = profit;      // overwrite in place
                }
            }
        }

        return prev[1][k];
    }
};
```

The `cur` array is gone: the "2·k" real values (`buy ∈ {0,1}` × `cap ∈ {1..k}`) live in one table (column `0` is just the constant base case).

### Is overwriting safe?

For each day the loop runs **`buy = 0` for all caps first, then `buy = 1`.**

**`buy = 0` pass:** reads `prev[1][cap-1]` and `prev[0][cap]`. The `buy = 1` row has not been touched yet this day, so `prev[1][cap-1]` is still the old value ✅.

**`buy = 1` pass:** reads `prev[0][cap]`, which has already been **overwritten** with the new `dp[ind][0][cap]`. Expanding:

```
new prev[0][cap] = max( p + dp[ind+1][1][cap-1],  dp[ind+1][0][cap] )          (p = prices[ind])

-p + new prev[0][cap] = max( dp[ind+1][1][cap-1],  -p + dp[ind+1][0][cap] )
```

The correct buy value is `max( -p + dp[ind+1][0][cap], dp[ind+1][1][cap] )`. The only extra term is `dp[ind+1][1][cap-1]` ("sell today and re-buy today"). Since more allowed transactions never hurt:

```
dp[ind+1][1][cap-1]  ≤  dp[ind+1][1][cap]
```

it is dominated by the skip option already in the `max`, so the result is unchanged.

### Caveat

This relies on sell-then-rebuy on the same day being a no-op. With a cooldown rule it breaks, so use two layers or explicit temporaries there.

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n·k)` |
| **Space** | `O(k)` (one `2 × (k+1)` table) |

---

## 8. Important Optimization: When `k` Is Large

Each transaction needs a **buy day and a sell day**, so at most `⌊n / 2⌋` transactions can ever be useful. If `k ≥ n / 2`, the limit never binds and the answer equals Stock II (unlimited transactions), which has a simple greedy solution:

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();

        if (k >= n / 2) {                          // limit is irrelevant → Stock II greedy
            int profit = 0;
            for (int i = 1; i < n; i++)
                if (prices[i] > prices[i - 1])
                    profit += prices[i] - prices[i - 1];
            return profit;
        }

        // otherwise k < n/2 → run the DP above (space-optimized version)
        vector<vector<int>> prev(2, vector<int>(k + 1, 0));
        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= k; cap++) {
                    int profit = 0;
                    if (buy)
                        profit = max(-prices[ind] + prev[0][cap], 0 + prev[1][cap]);
                    else
                        profit = max( prices[ind] + prev[1][cap - 1], 0 + prev[0][cap]);
                    prev[buy][cap] = profit;
                }
            }
        }
        return prev[1][k];
    }
};
```

This also protects memory when `k` is huge, because the DP table size is `O(k)` and the shortcut makes the DP run only when `k < n/2`.

---

## 9. Alternative View: One Counter for All Buy/Sell Events

The `(buy, cap)` pair can be merged into a single number `t` counting how many buy/sell **events** have happened (`0 .. 2k-1`):

```
t even → next event is a BUY
t odd  → next event is a SELL
t == 2k → all transactions done
```

```cpp
f(ind, t):
    if (ind == n || t == 2*k) return 0;
    if (t % 2 == 0) return max(-prices[ind] + f(ind+1, t+1), f(ind+1, t));   // buy
    else            return max( prices[ind] + f(ind+1, t+1), f(ind+1, t));   // sell
```

It is the same DP with the state flattened to `dp[n][2k]`.

---

## 10. Complexity Summary

| Approach | Time | Space |
|---|---|---|
| Recursion | `O(2ⁿ)` | `O(n)` |
| Memoization | `O(n·k)` | `O(n·k)` + stack |
| Tabulation | `O(n·k)` | `O(n·k)` |
| Two layers | `O(n·k)` | `O(k)` |
| In-place `2 × (k+1)` | `O(n·k)` | `O(k)` |
| With `k ≥ n/2` greedy shortcut | `O(n)` for large `k` | `O(1)` |

---

## 11. The Whole Stock Series in One Table

| Problem | Rule | State | Notes |
|---|---|---|---|
| Stock I | At most 1 | running minimum | `O(n)` time, `O(1)` space |
| Stock II | Unlimited | `(ind, buy)` | or greedy over rises |
| Stock III | At most 2 | `(ind, buy, cap ≤ 2)` | |
| **Stock IV** | **At most `k`** | **`(ind, buy, cap ≤ k)`** | **`k ≥ n/2` ⇒ Stock II** |
| With Cooldown | Unlimited + 1 rest day after a sell | `(ind, buy)`, jump to `ind + 2` on sell | |
| With Fee | Unlimited + fee per transaction | `(ind, buy)`, subtract fee on sell | |

Stock IV is the general form: set `k = 1` to get Stock I, `k = 2` to get Stock III, and `k ≥ n/2` to get Stock II.
