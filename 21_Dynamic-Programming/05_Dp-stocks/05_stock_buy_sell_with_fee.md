# Best Time to Buy and Sell Stock with Transaction Fee

Problem link: [LeetCode 714: Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

---

> ## 📌 Reminder: read Stock II first
>
> This problem is **Stock II plus one extra term**. Everything below (the `(ind, buy)` state, the cash-flow accounting, the base case, the tabulation direction, and the in-place space optimization analysis) is explained in detail in the Stock II notes:
>
> **[02_stock_buy_sell_ii.md](https://github.com/VarunMendre/Problem_Solving/blob/main/21_Dynamic-Programming/05_Dp-stocks/02_stock_buy_sell_ii.md)**
>
> Here we only explain **what changes** because of the fee.

---

## 1. Problem Statement

You are given `prices[i]` (the price of a stock on day `i`) and an integer `fee` that is paid **once per transaction**. You may complete as many transactions as you like, but you can hold at most one share at a time. Return the maximum profit.

```
prices = [1, 3, 2, 8, 4, 9], fee = 2

Buy at 1, sell at 8 → profit = 8 - 1 - 2 = 5
Buy at 4, sell at 9 → profit = 9 - 4 - 2 = 3
Total = 8

Answer: 8
```

```
prices = [1, 3, 7, 5, 10, 3], fee = 3

Buy at 1, sell at 10 → profit = 10 - 1 - 3 = 6

Answer: 6
```

---

## 2. What Changes From Stock II: One Term

Stock II:

```
buy  = 1:  max( -p + f(ind+1, 0),   0 + f(ind+1, 1) )
buy  = 0:  max(  p + f(ind+1, 1),   0 + f(ind+1, 0) )
```

Stock with fee:

```
buy  = 1:  max( -p        + f(ind+1, 0),   0 + f(ind+1, 1) )
buy  = 0:  max( (p - fee) + f(ind+1, 1),   0 + f(ind+1, 0) )
                  ↑ the only change
```

The state `(ind, buy)`, the base case (`ind == n → 0`), the choices and the transitions are identical. A completed transaction now earns `sellPrice - buyPrice - fee`.

### Where do we charge the fee: on the buy or the sell?

The fee is paid **once per transaction** (one buy + one sell). We can charge it at either end:

| Charge on | Buy branch | Sell branch |
|---|---|---|
| **Sell** (this code) | `-p` | `p - fee` |
| Buy | `-p - fee` | `p` |

Both give the **same answer**. Charging on the **sell** is slightly cleaner: the fee is paid when the transaction is **completed**, and a stock that is bought but never sold (which is never optimal) doesn't carry a fee.

---

## 3. The Progression: Recursion → Memoization → Tabulation

These are the Stock II versions with `- fee` added in the sell branch.

### Recursion

```cpp
int fn(int ind, int buy, int n, vector<int>& prices, int fee) {
    if (ind == n) return 0;

    if (buy)
        return max(-prices[ind] + fn(ind + 1, 0, n, prices, fee),            // buy
                    0            + fn(ind + 1, 1, n, prices, fee));           // skip
    else
        return max((prices[ind] - fee) + fn(ind + 1, 1, n, prices, fee),     // sell (pay fee)
                    0                   + fn(ind + 1, 0, n, prices, fee));    // hold
}
// call: fn(0, 1, n, prices, fee)
```

`O(2ⁿ)` time, `O(n)` stack.

### Memoization

Add `dp[ind][buy]` (size `n × 2`) exactly as in Stock II: check the cache after the base case, store before returning. `O(n)` time, `O(n)` space.

### Tabulation

```cpp
vector<vector<int>> dp(n + 1, vector<int>(2, 0));      // dp[n][*] = 0 is the base case

for (int ind = n - 1; ind >= 0; ind--) {
    for (int buy = 0; buy <= 1; buy++) {
        if (buy)
            dp[ind][buy] = max(-prices[ind] + dp[ind + 1][0], 0 + dp[ind + 1][1]);
        else
            dp[ind][buy] = max((prices[ind] - fee) + dp[ind + 1][1], 0 + dp[ind + 1][0]);
    }
}
return dp[0][1];
```

`O(n)` time, `O(n)` space. Filled from the last day backwards because `dp[ind]` needs `dp[ind+1]`.

---

## 4. Space Optimization: The Submitted Code

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();

        vector<int> prev(2, 0);        // prev[0] = dp[ind+1][0] (holding), prev[1] = dp[ind+1][1] (can buy)

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                int profit = 0;

                if (buy) {
                    profit = max(-prices[ind] + prev[0], 0 + prev[1]);
                } else {
                    profit = max((prices[ind] - fee) + prev[1], 0 + prev[0]);
                }

                prev[buy] = profit;     // overwrite in place
            }
        }

        return prev[1];
    }
};
```

Only the previous day's two values are needed, so one array of size 2 replaces the whole table, updated **in place**.

### Is the in-place update safe? (Yes, and even more clearly than in Stock II)

The loop runs `buy = 0` first, which overwrites `prev[0]` with the new `dp[ind][0]`. Then `buy = 1` reads `prev[0]`, which is now the **new** value instead of `dp[ind+1][0]`. Let's see what that does (`p = prices[ind]`):

```
new prev[0] = max( (p - fee) + dp[ind+1][1],  dp[ind+1][0] )

-p + new prev[0] = max( -fee + dp[ind+1][1],  -p + dp[ind+1][0] )
```

The correct buy value is `max( -p + dp[ind+1][0],  dp[ind+1][1] )`. The only extra term is:

```
-fee + dp[ind+1][1]        ("sell today, pay the fee, and re-buy today")
```

Since `fee ≥ 0`:

```
-fee + dp[ind+1][1]  ≤  dp[ind+1][1]        ← the skip option already in the max
```

So the contaminated term can **never** beat an option that is already present, and the result is unchanged. With a fee the extra term is even **strictly worse** (it wastes `fee`), whereas in Stock II it was exactly equal. A cooldown rule would still break this trick.

### Dry run: `prices = [1, 3, 2, 8, 4, 9]`, `fee = 2` (state shown as `[prev[0], prev[1]]`)

```
start:         [0, 0]

ind=5 (9):  buy=0 → max(9-2+0, 0) = 7          buy=1 → max(-9+7, 0) = 0          [7, 0]
ind=4 (4):  buy=0 → max(4-2+0, 7) = 7          buy=1 → max(-4+7, 0) = 3          [7, 3]
ind=3 (8):  buy=0 → max(8-2+3, 7) = 9          buy=1 → max(-8+9, 3) = 3          [9, 3]
ind=2 (2):  buy=0 → max(2-2+3, 9) = 9          buy=1 → max(-2+9, 3) = 7          [9, 7]
ind=1 (3):  buy=0 → max(3-2+7, 9) = 9          buy=1 → max(-3+9, 7) = 7          [9, 7]
ind=0 (1):  buy=0 → max(1-2+7, 9) = 9          buy=1 → max(-1+9, 7) = 8          [9, 8]

return prev[1] = 8 ✅
```

At `ind=3`, the contaminated read gives `-8 + 9 = 1`, which equals the extra term `-fee + dp[4][1] = -2 + 3 = 1`. It loses to the skip option `3`, exactly as proven above.

### Second example: `prices = [1, 3, 7, 5, 10, 3]`, `fee = 3`

```
start:         [0, 0]
ind=5 (3):     [0, 0]
ind=4 (10):    [7, 0]      sell at 10: 10 - 3 = 7
ind=3 (5):     [7, 2]      buy at 5 then sell at 10: -5 + 7 = 2
ind=2 (7):     [7, 2]
ind=1 (3):     [7, 4]      buy at 3 → -3 + 7 = 4
ind=0 (1):     [7, 6]      buy at 1 → -1 + 7 = 6

return 6 ✅
```

---

## 5. Why the Stock II Greedy Doesn't Work Here

In Stock II we could sum every upward step, because splitting a long rise into daily trades cost nothing. With a fee, **every transaction costs `fee`**, so splitting a rise into many trades pays the fee many times.

```
prices = [1, 3, 5], fee = 1

Greedy (trade every rise):  (3 - 1 - 1) + (5 - 3 - 1) = 1 + 1 = 2
One long transaction:        5 - 1 - 1                = 3   ✅  (better)
```

The DP naturally handles this: it only pays the fee when it **chooses** to sell, and it can decide to keep holding through small dips.

---

## 6. Bonus: Forward Two-Variable Form

The same DP can be run forward with two variables, which many people find easiest to remember:

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int cash = 0;               // best profit while holding NO stock
        int hold = -prices[0];      // best profit while HOLDING a stock

        for (int i = 1; i < prices.size(); i++) {
            cash = max(cash, hold + prices[i] - fee);   // sell today (pay fee) or stay in cash
            hold = max(hold, cash - prices[i]);          // buy today or keep holding
        }

        return cash;
    }
};
```

Trace on `[1, 3, 2, 8, 4, 9]`, `fee = 2`:

```
start: cash = 0, hold = -1
i=1 (3):  cash = max(0, -1+3-2) = 0     hold = max(-1, 0-3) = -1
i=2 (2):  cash = max(0, -1+2-2) = 0     hold = max(-1, 0-2) = -1
i=3 (8):  cash = max(0, -1+8-2) = 5     hold = max(-1, 5-8) = -1
i=4 (4):  cash = max(5, -1+4-2) = 5     hold = max(-1, 5-4) = 1
i=5 (9):  cash = max(5,  1+9-2) = 8     hold = max(1, 8-9) = 1

return cash = 8 ✅
```

---

## 7. Complexity Analysis

| Approach | Time | Space |
|---|---|---|
| Recursion | `O(2ⁿ)` | `O(n)` |
| Memoization | `O(n)` | `O(n)` + stack |
| Tabulation | `O(n)` | `O(n)` |
| **Space-optimized (submitted)** | **`O(n)`** | **`O(1)`** |
| Forward two-variable | `O(n)` | `O(1)` |

---

## 8. Stock Series: Updated

| Problem | Rule | State | What changes |
|---|---|---|---|
| Stock I | At most 1 transaction | running minimum | — |
| Stock II | Unlimited | `(ind, buy)` | base template |
| **Stock with Fee** | **Unlimited, fee per transaction** | **`(ind, buy)`** | **`p - fee` on the sell** |
| Stock III | At most 2 | `(ind, buy, cap)` | add a counter |
| Stock IV | At most `k` | `(ind, buy, cap)` | general counter |
| With Cooldown | Unlimited + 1 rest day after a sell | `(ind, buy)` | jump to `ind + 2` on sell |
