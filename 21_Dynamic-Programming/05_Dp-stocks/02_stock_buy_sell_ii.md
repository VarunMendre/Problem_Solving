# Best Time to Buy and Sell Stock II

## Problem Statement

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i^{\text{th}}$ day.

On each day, you may decide to buy and/or sell the stock. You can only hold **at most one** share of stock at any time. However, you can buy it then immediately sell it on the same day.

Find and return the **maximum profit** you can achieve.

### Example

**Input:** `prices = [7, 1, 5, 3, 6, 4]`

**Output:** `7`

**Explanation:**

* Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = $5 - 1 = 4$.

* Buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = $6 - 3 = 3$.
  Total profit is $4 + 3 = 7$.

A detailed breakdown of all approaches to solve the **Best Time to Buy and Sell Stock II** problem:

1. Pure Recursion

2. Top-Down Dynamic Programming (Memoization)

3. Bottom-Up Dynamic Programming (Tabulation)

4. Space Optimization (Two Vectors Variant)

5. Space Optimization (Single Vector In-Place Variant)

## 1. Recursion

### 1. Approach / Intuition

At every day `ind`, we have two primary states:

1. `buy = 1`: We are allowed to buy a stock. We can either:

   * **Buy** the stock at the current price: `-prices[ind]` and transition to state `buy = 0`.

   * **Skip** buying: `0` and remain in state `buy = 1`.

2. `buy = 0`: We hold a stock and are allowed to sell it. We can either:

   * **Sell** the stock at the current price: `+prices[ind]` and transition to state `buy = 1`.

   * **Skip** selling: `0` and remain in state `buy = 0`.

We recursively explore all choices to maximize our total profit.

### 2. Why We Think About This

Since we can make multiple transactions and have choices at every single day (buy, sell, or skip), modeling this as a decision tree with a boolean flag for whether we can buy captures all valid trading strategies.

### 3. Dry-Run

For `prices = [1, 2]`:

* Start at `fn(0, 1)`: We can buy.

  * Option A (Buy): `-1 + fn(1, 0)` $\rightarrow$ At day 1, we sell: `+2 + fn(2, 1)` $\rightarrow 0$. Total = $-1 + 2 = 1$.

  * Option B (Skip): `0 + fn(1, 1)` $\rightarrow$ Buy at day 1: `-2 + fn(2, 0)` $\rightarrow -2$. Total = $-2$.

* Maximum profit = $\max(1, -2) = 1$.

### 4. Story Points

* **Day-by-Day Decisions:** Walk through every index from day $0$ to $n$.

* **State Flag (`buy`):** $1$ means we can buy; $0$ means we must sell our current holding.

* **Base Case:** Reaching index $n$ means no more days are left, returning a profit of $0$.

### 5. Code

```
class Solution {
private:
    int fn(int ind, int buy, int n, vector<int>& prices) {
        // Base case: no more days left to trade
        if (ind == n)
            return 0;

        int profit = 0;

        if (buy) {
            // Choice 1: Buy stock (-prices[ind]) or Skip (0)
            profit = max(-prices[ind] + fn(ind + 1, 0, n, prices),
                         0 + fn(ind + 1, 1, n, prices));
        } else {
            // Choice 1: Sell stock (+prices[ind]) or Skip (0)
            profit = max(prices[ind] + fn(ind + 1, 1, n, prices),
                         0 + fn(ind + 1, 0, n, prices));
        }

        return profit;
    }

public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        return fn(0, 1, n, prices);
    }
};

```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(2^N)$ — At each day, we branch into 2 choices across $N$ days.

* **Space Complexity:** $\mathcal{O}(N)$ — Recursion call stack depth goes up to $N$.

## 2. Memoization (Top-Down DP)

### 1. Approach / Intuition

The pure recursion tree recalculates identical states `(ind, buy)` multiple times. By introducing a 2D memoization table `dp[ind][buy]`, we cache results of subproblems so they can be retrieved in $\mathcal{O}(1)$ time.

### 2. Why We Think About This

Overlapping subproblems exist because the same day index and buy status can be reached via different choice paths. Memoization cuts redundant recursive branches.

### 3. Dry-Run

When `fn(ind, buy)` is called, it checks if `dp[ind][buy]` is already computed (not `-1`). If so, it returns the cached profit instantly, preventing redundant execution trees.

### 4. Story Points

* **Cache Check:** Look up `dp[ind][buy]` before executing choices.

* **Store Result:** Save calculated profits in `dp[ind][buy]` before returning.

### 5. Code

```
class Solution {
private:
    int fn(int ind, int buy, int n, vector<int>& prices,
           vector<vector<int>>& dp) {
        if (ind == n)
            return 0;

        // Return cached result if already computed
        if (dp[ind][buy] != -1)
            return dp[ind][buy];

        int profit = 0;

        if (buy) {
            profit = max(-prices[ind] + fn(ind + 1, 0, n, prices, dp),
                         0 + fn(ind + 1, 1, n, prices, dp));
        } else {
            profit = max(prices[ind] + fn(ind + 1, 1, n, prices, dp),
                         0 + fn(ind + 1, 0, n, prices, dp));
        }

        return dp[ind][buy] = profit;
    }

public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        // DP table of size n x 2 initialized to -1
        vector<vector<int>> dp(n, vector<int>(2, -1));
        return fn(0, 1, n, prices, dp);
    }
};

```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N \times 2) = \mathcal{O}(N)$ — Total states are $N \times 2$.

* **Space Complexity:** $\mathcal{O}(N) + \mathcal{O}(N) = \mathcal{O}(N)$ — For the 2D DP table and recursion call stack.

## 3. Tabulation (Bottom-Up DP)

### 1. Approach / Intuition

We transition from top-down recursion to a bottom-up iterative table. We build a `dp` table of size $(n + 1) \times 2$, iterating backwards from day $n-1$ down to $0$, filling in values based on future states `dp[ind + 1]`.

### 2. Why We Think About This

Tabulation completely removes recursion overhead and stack space limits by filling out states iteratively.

### 3. Dry-Run

Base cases at index $n$ are initialized to $0$. We iterate backwards, calculating max profits for `buy = 1` and `buy = 0` at each day until we reach `dp[0][1]`.

### 4. Story Points

* **Bottom-Up Loop:** Start from day $n-1$ down to $0$.

* **State Table:** Fill out `buy` states $0$ and $1$ for every day.

* **Final Answer:** `dp[0][1]` holds the maximum profit starting from day 0 with buying capability.

### 5. Code

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        // DP table with n+1 rows and 2 columns, initialized to 0
        vector<vector<int>> dp(n + 1, vector<int>(2, 0));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                int profit = 0;

                if (buy) {
                    profit =
                        max(-prices[ind] + dp[ind + 1][0], 0 + dp[ind + 1][1]);
                } else {
                    profit = max(prices[ind] + dp[ind + 1][1],
                                 0 + dp[ind + 1][0]);
                }

                dp[ind][buy] = profit;
            }
        }

        return dp[0][1];
    }
};

```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$ — Iterates through all $N$ days and 2 buy states.

* **Space Complexity:** $\mathcal{O}(N)$ — To store the $(N + 1) \times 2$ DP table.

## 4. Space Optimization (Two Vectors Variant)

### 1. Approach / Intuition

Since any day's calculation only depends on the results of the **next day** (`ind + 1`), we do not need to store the entire $N \times 2$ table. We can maintain a `prev` vector for the next day and a `cur` vector for the current day.

### 2. Why We Think About This

Optimizes space complexity from $\mathcal{O}(N)$ down to $\mathcal{O}(1)$ auxiliary storage by discarding history older than the immediate next day.

### 3. Dry-Run

As we step backwards from $n-1$ to $0$, `cur` computes profits using values from `prev`, and then `prev` is updated to `cur`.

### 4. Story Points

* **Row Rolling:** Keep track of only `prev` and `cur` state vectors of size 2.

* **Vector Assignment:** Assign `prev = cur` at the end of each day loop.

### 5. Code

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        // Represents dp[ind + 1]
        vector<int> prev(2, 0);

        for (int ind = n - 1; ind >= 0; ind--) {
            vector<int> cur(2, 0);

            for (int buy = 0; buy <= 1; buy++) {
                int profit = 0;

                if (buy) {
                    profit = max(-prices[ind] + prev[0], 0 + prev[1]);
                } else {
                    profit = max(prices[ind] + prev[1], 0 + prev[0]);
                }

                cur[buy] = profit;
            }

            prev = cur;
        }

        return prev[1];
    }
};

```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$

* **Space Complexity:** $\mathcal{O}(1)$ — Only two elements are stored in `prev` and `cur`.

## 5. Space Optimization (Single Vector In-Place Variant)

### 1. Approach / Intuition

Instead of creating a new `cur` vector on each iteration, we can update a single `prev` vector **in-place** as we iterate through the buy states ($0$ and $1$). Because updating one state doesn't instantly invalidate the other needed prior value in a size-2 array, we can safely overwrite it.

### 2. Why We Think About This

Further simplifies code by eliminating the secondary `cur` vector declaration inside the loop while maintaining optimal constant space.

### 3. Dry-Run

For each day, `prev[1]` and `prev[0]` are updated directly using the values from the previous day stored inside the same vector.

### 4. Story Points

* **Single Buffer:** Reuse the `prev` vector of size 2 across loop iterations.

* **In-Place Update:** Directly assign computed profits back to `prev[buy]`.

### 5. Code

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        // Single vector storing the previous day's states
        vector<int> prev(2, 0);

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                int profit = 0;

                if (buy) {
                    profit = max(-prices[ind] + prev[0], 0 + prev[1]);
                } else {
                    profit = max(prices[ind] + prev[1], 0 + prev[0]);
                }

                prev[buy] = profit;
            }
        }

        return prev[1];
    }
};

```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$

* **Space Complexity:** $\mathcal{O}(1)$ — Uses a single vector of fixed size 2.