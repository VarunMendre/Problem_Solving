# Best Time to Buy and Sell Stock III

## Problem Statement

You are given an array `prices` where `prices[i]` is the price of a given stock on the $i^{\text{th}}$ day.

Find the maximum profit you can achieve. You may complete **at most two transactions**.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

### Example

**Input:** `prices = [3, 3, 5, 0, 0, 3, 1, 4]`

**Output:** `6`

**Explanation:**

* Buy on day 4 (price = 0) and sell on day 6 (price = 3), profit = $3 - 0 = 3$.

* Buy on day 7 (price = 1) and sell on day 8 (price = 4), profit = $4 - 1 = 3$.

* Total profit = $3 + 3 = 6$.

A detailed breakdown of five approaches to solve the **Best Time to Buy and Sell Stock III** problem:

1. Pure Recursion

2. Top-Down Dynamic Programming (Memoization)

3. Bottom-Up Dynamic Programming (Tabulation)

4. Space Optimization (Two 2D Grids: `prev` & `cur`)

5. Space Optimization In-Place (Single 2D Grid: `prev`)

---

## 1. Recursion

### 1. Approach / Intuition

To enforce at most two transactions, we maintain three state variables:

1. `ind`: Current day index ($0 \le ind < n$).

2. `buy`: $1$ if we can buy, $0$ if we hold a stock and must sell.

3. `cap`: Remaining transaction capacity ($2$, $1$, or $0$).

When selling a stock (`buy = 0`), a transaction is completed, so we decrement `cap` by 1 (`cap - 1`).

### 2. Why We Think About This

Since we have decisions at each step (buy/skip or sell/skip) bounded by a capacity limit of 2 completed transactions, recursive branching allows us to express all valid state paths easily.

### 3. Dry-Run

For `prices = [3, 5, 0, 3]`, starting at `fun(0, 1, 2, 4, prices)`:

* At `ind = 0` (`price = 3`, `buy = 1`, `cap = 2`):

  * Option A (Buy): $-3 + \text{fun}(1, 0, 2)$

  * Option B (Skip): $0 + \text{fun}(1, 1, 2)$

* Following Option A at `ind = 1` (`price = 5`, `buy = 0`, `cap = 2`):

  * Option A1 (Sell): $+5 + \text{fun}(2, 1, 1)$ $\rightarrow$ `cap` becomes $1$.

  * Option A2 (Skip): $0 + \text{fun}(2, 0, 2)$

We continue exploring decisions until `ind == n` or `cap == 0`, returning the overall maximum profit.

### 4. Story Points

* **3D State Space:** Tracks index (`ind`), buying eligibility (`buy`), and remaining transaction capacity (`cap`).

* **Transaction Completion:** Decrement `cap` only when a stock is **sold**.

* **Base Conditions:** Stop when out of days (`ind == n`) or out of allowed transactions (`cap == 0`).

### 5. Code

```cpp
class Solution {
private:
    int fun(int ind, int buy, int cap, int n, vector<int>& prices) {
        // Base case: No transactions left or reached end of prices array
        if (cap == 0)
            return 0;
        if (ind == n)
            return 0;

        int profit = 0;

        if (buy) {
            // Option 1: Buy stock (-prices[ind]) and change state to buy = 0
            // Option 2: Skip buying (0) and keep state buy = 1
            profit = max(-prices[ind] + fun(ind + 1, 0, cap, n, prices),
                         0 + fun(ind + 1, 1, cap, n, prices));
        } else {
            // Option 1: Sell stock (+prices[ind]) and decrement capacity (cap - 1)
            // Option 2: Skip selling (0) and keep state buy = 0
            profit = max(prices[ind] + fun(ind + 1, 1, cap - 1, n, prices),
                         0 + fun(ind + 1, 0, cap, n, prices));
        }

        return profit;
    }

public:
    int maxProfit(vector<int>& prices) {
        return fun(0, 1, 2, prices.size(), prices);
    }
};
```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(2^N)$ — Exponential due to 2 recursive options per step over $N$ days.

* **Space Complexity:** $\mathcal{O}(N)$ — Recursion call stack depth up to $N$.

---

## 2. Memoization (Top-Down DP)

### 1. Approach / Intuition

Overlapping subproblems recur for identical `(ind, buy, cap)` states across different recursive paths. We initialize a 3D table `dp[n][2][3]` with `-1` to cache subproblem results.

### 2. Why We Think About This

Caching visited `(ind, buy, cap)` configurations reduces exponential time to linear time relative to the number of unique states.

### 3. Dry-Run

When `fun(ind, buy, cap)` is executed, it first checks `dp[ind][buy][cap]`. If it is not `-1`, the cached answer is returned in $\mathcal{O}(1)$ time, skipping recursive branch evaluation.

### 4. Story Points

* **3D DP Table Size:** `n x 2 x 3` (Indices $0 \dots n-1$, Buy $0 \dots 1$, Capacity $0 \dots 2$).

* **Lookup Before Branching:** Check for cached values to eliminate redundant calculations.

* **Store Before Returning:** Assign `dp[ind][buy][cap] = profit`.

### 5. Code

```cpp
class Solution {
private:
    int fun(int ind, int buy, int cap, int n, vector<int>& prices,
            vector<vector<vector<int>>>& dp) {
        if (cap == 0)
            return 0;
        if (ind == n)
            return 0;

        // Return memoized result if available
        if (dp[ind][buy][cap] != -1)
            return dp[ind][buy][cap];

        int profit = 0;

        if (buy) {
            profit = max(-prices[ind] + fun(ind + 1, 0, cap, n, prices, dp),
                         0 + fun(ind + 1, 1, cap, n, prices, dp));
        } else {
            profit = max(prices[ind] + fun(ind + 1, 1, cap - 1, n, prices, dp),
                         0 + fun(ind + 1, 0, cap, n, prices, dp));
        }

        return dp[ind][buy][cap] = profit;
    }

public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        // 3D vector for state caching initialized to -1
        vector<vector<vector<int>>> dp(
            n, vector<vector<int>>(2, vector<int>(3, -1)));

        return fun(0, 1, 2, n, prices, dp);
    }
};
```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N \times 2 \times 3) = \mathcal{O}(N)$ — Unique states equal $6N$.

* **Space Complexity:** $\mathcal{O}(N \times 2 \times 3) + \mathcal{O}(N) = \mathcal{O}(N)$ — 3D DP table storage plus call stack depth.

---

## 3. Tabulation (Bottom-Up DP)

### 1. Approach / Intuition

We build a 3D table `dp[n+1][2][3]` iteratively. Base cases (`ind = n` or `cap = 0`) remain `0`. We loop `ind` backwards from `n-1` to `0`, `buy` from `0` to `1`, and `cap` from `1` to `2`.

### 2. Why We Think About This

Tabulation avoids call stack limits and removes recursion overhead by populating values iteratively using past calculated states (`ind + 1`).

### 3. Dry-Run

* Initialize `dp[n+1][2][3]` with `0`.

* Outer loop iterates from `ind = n-1` down to `0`.

* Inner loops check possibilities for `buy` ($0$ or $1$) and `cap` ($1$ or $2$).

* Final output is read from `dp[0][1][2]` (Day 0, Can Buy, 2 Transactions available).

### 4. Story Points

* **Iterative Loops:** Reverse loop order for `ind` ($n-1 \to 0$).

* **Base Initializations:** Multi-dimensional grid pre-filled with zero handles base conditions implicitly.

* **Result Location:** Answer is stored at `dp[0][1][2]`.

### 5. Code

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        // 3D vector initialized to 0 (base cases cap=0 and ind=n are implicitly handled)
        vector<vector<vector<int>>> dp(
            n + 1, vector<vector<int>>(2, vector<int>(3, 0)));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= 2; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit = max(-prices[ind] + dp[ind + 1][0][cap],
                                     0 + dp[ind + 1][1][cap]);
                    } else {
                        profit = max(prices[ind] + dp[ind + 1][1][cap - 1],
                                     0 + dp[ind + 1][0][cap]);
                    }

                    dp[ind][buy][cap] = profit;
                }
            }
        }

        return dp[0][1][2];
    }
};
```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$ — Iterates through $N \times 2 \times 2$ iterations.

* **Space Complexity:** $\mathcal{O}(N)$ — Stores a $(N + 1) \times 2 \times 3$ 3D table.

---

## 4. Space Optimization (Two 2D Grids)

### 1. Approach / Intuition

`dp[ind]` only depends on entries from `dp[ind + 1]`. Therefore, we replace the 3D table with two 2D grids of size $2 \times 3$: `prev` (representing day `ind + 1`) and `cur` (representing day `ind`).

### 2. Why We Think About This

Reduces space complexity from $\mathcal{O}(N)$ down to $\mathcal{O}(1)$ since state requirements only span adjacent days.

### 3. Dry-Run

* `prev` represents index `ind + 1`.

* `cur` calculates values for index `ind`.

* After evaluating all `buy` and `cap` choices for day `ind`, we update `prev = cur`.

### 4. Story Points

* **Two 2D Arrays:** `prev[2][3]` and `cur[2][3]`.

* **Row Transition:** `prev` gets overwritten by `cur` after each day's calculations.

* **Final Result:** Stored in `prev[1][2]`.

### 5. Code

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        // 2D grids representing next day (prev) and current day (cur)
        vector<vector<int>> prev(2, vector<int>(3, 0));
        vector<vector<int>> cur(2, vector<int>(3, 0));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= 2; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit =
                            max(-prices[ind] + prev[0][cap], 0 + prev[1][cap]);
                    } else {
                        profit = max(prices[ind] + prev[1][cap - 1],
                                     0 + prev[0][cap]);
                    }

                    cur[buy][cap] = profit;
                }
            }

            prev = cur;
        }

        return prev[1][2];
    }
};
```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$

* **Space Complexity:** $\mathcal{O}(1)$ — Uses two fixed-size $2 \times 3$ matrices.

---

## 5. Space Optimization In-Place (Single 2D Grid)

### 1. Approach / Intuition

We can optimize further by maintaining only a single 2D vector `prev[2][3]`. Direct updates to `prev[buy][cap]` do not corrupt required values during the loop iterations for a given day.

### 2. Why We Think About This

It removes the allocation and assignment overhead of a secondary `cur` 2D array, achieving clean, in-place state transitions.

### 3. Dry-Run

For each index `ind`, `prev[buy][cap]` is updated in-place using current values in `prev`, correctly referencing values corresponding to `ind + 1`.

### 4. Story Points

* **Single 2D Vector:** Memory footprint consists of a single $2 \times 3$ array `prev`.

* **In-Place Updates:** Assignments write directly into `prev[buy][cap]`.

### 5. Code

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();

        // Single 2D grid updated in-place
        vector<vector<int>> prev(2, vector<int>(3, 0));

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int buy = 0; buy <= 1; buy++) {
                for (int cap = 1; cap <= 2; cap++) {
                    int profit = 0;

                    if (buy) {
                        profit =
                            max(-prices[ind] + prev[0][cap], 0 + prev[1][cap]);
                    } else {
                        profit = max(prices[ind] + prev[1][cap - 1],
                                     0 + prev[0][cap]);
                    }

                    prev[buy][cap] = profit;
                }
            }
        }

        return prev[1][2];
    } 
};
```

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N)$

* **Space Complexity:** $\mathcal{O}(1)$ — Uses only a single fixed-size $2 \times 3$ matrix.