# Best Time to Buy and Sell Stock I

Problem link: [LeetCode 121: Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

---

## 1. Problem Statement

You are given an array `prices` where `prices[i]` is the price of a stock on day `i`. You may complete **at most one transaction**: choose one day to **buy** and a **later** day to **sell**. Return the maximum profit. If no profit is possible, return `0`.

```
prices = [7, 1, 5, 3, 6, 4]

Buy on day 1 (price 1), sell on day 4 (price 6) → profit = 5
(Buying at 7 and selling at 6 would be a loss; selling before buying is not allowed.)

Answer: 5
```

```
prices = [7, 6, 4, 3, 1]

Prices only fall, so every transaction loses money → don't trade.

Answer: 0
```

---

## 2. Brute Force and Why We Improve It

Try every pair `(buy day i, sell day j)` with `i < j`:

```cpp
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++)
        maxProfit = max(maxProfit, prices[j] - prices[i]);
```

This is `O(n²)`, which is too slow for `n = 10⁵`. The inner loop is repeating work: for every sell day `j` we re-scan all earlier days to find the cheapest buy price.

---

## 3. Intuition: The Best Buy for a Given Sell Day Is the Cheapest Day Before It

Fix a sell day `j`. The profit from selling on day `j` is:

```
prices[j] - (buy price)
```

To make this as large as possible, we want the **smallest buy price among the days before `j`** (or on day `j`).

So instead of searching for it every time, **carry it along** as we scan left to right:

```
mini = the minimum price seen so far
```

At each day:

1. Profit if we sell **today** = `prices[day] - mini`
2. Update `maxProfit` with it
3. Update `mini` with today's price (it may be a better buy for **later** days)

The answer is the best profit over all possible sell days.

---

## 4. Code

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        int mini = prices[0];      // cheapest price seen so far (day 0 is the first candidate)
        int maxProfit = 0;         // 0 = "don't trade at all"

        for (int day = 1; day < n; day++) {
            int transaction = prices[day] - mini;   // profit if we sell today

            maxProfit = max(maxProfit, transaction);
            mini = min(mini, prices[day]);          // update AFTER using it
        }

        return maxProfit;
    }
};
```

---

## 5. Dry Run: `prices = [7, 1, 5, 3, 6, 4]`

Start: `mini = 7`, `maxProfit = 0`.

```
day  price   transaction (price - mini)   maxProfit   mini after
 1     1          1 - 7 = -6                  0           1
 2     5          5 - 1 =  4                  4           1
 3     3          3 - 1 =  2                  4           1
 4     6          6 - 1 =  5                  5           1
 5     4          4 - 1 =  3                  5           1

Answer: 5 ✅  (buy at 1, sell at 6)
```

Notice that on day 1 the transaction is negative (`-6`), so `maxProfit` stays `0`, and `mini` drops to `1`, a better buy price for every later day.

### Falling prices: `[7, 6, 4, 3, 1]`

```
day 1: 6 - 7 = -1 → maxProfit = 0, mini = 6
day 2: 4 - 6 = -2 → maxProfit = 0, mini = 4
day 3: 3 - 4 = -1 → maxProfit = 0, mini = 3
day 4: 1 - 3 = -2 → maxProfit = 0, mini = 1

Answer: 0 ✅
```

---

## 6. Story Points

---

**Story Point 1: "`maxProfit` starts at 0 because not trading is allowed"**

If prices only fall, every transaction is negative. Starting from `0` means the best choice is to do nothing, which is exactly what the problem asks for.

---

**Story Point 2: "Compute the profit with the old `mini`, then update `mini`"**

On each day we first use `mini` (the cheapest price from **earlier** days), and only then fold today's price into it. This guarantees that we never sell before we buy. (If you update `mini` first, the same-day transaction is `0`, which is harmless here, but the order above keeps the logic clean.)

---

**Story Point 3: "We never need to know *which day* was the buy day, only its price"**

`mini` stores just the cheapest price, not its index. The profit only depends on the price difference, so the index is unnecessary. (If a problem asked you to return the days, you would store the index as well.)

---

**Story Point 4: "It is a max-subarray problem in disguise"**

Let `d[i] = prices[i] - prices[i-1]` (daily changes). Buying on day `a` and selling on day `b` earns exactly `d[a+1] + ... + d[b]`. So the best single transaction equals the **maximum subarray sum** of `d` (Kadane's algorithm), with `0` if every subarray is negative.

```
prices = [7, 1, 5, 3, 6, 4]
d      = [  -6, 4, -2, 3, -2]
best subarray = 4 + (-2) + 3 = 5 ✅
```

---

## 7. Complexity Analysis

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | One pass over the array |
| **Space** | `O(1)` | Two integer variables |

---

## 8. Where This Fits in the Stock Series

| Problem | Rule | Approach |
|---|---|---|
| **Stock I** | **At most 1** transaction | Track the minimum price so far (this file) |
| Stock II | Unlimited transactions | DP on `(day, canBuy)`, or add every upward step |
| Stock III | At most 2 transactions | DP on `(day, canBuy, transactionsLeft)` |
| Stock IV | At most `k` transactions | Same DP with `k` |
| With Cooldown | Unlimited, one day of rest after selling | DP, jump to `day + 2` after a sell |
| With Fee | Unlimited, fee per transaction | DP, subtract the fee on sell |

Stock I is the simplest member: with only one transaction allowed, a single running minimum is enough, so no DP table is needed.
