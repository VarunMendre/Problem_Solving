# Coin Change — Minimum Coins

---

## 1. Problem Statement

Given an array of coin denominations `coins[]` and an integer `amount`, find the **minimum number of coins** needed to make up `amount`. You can use each coin denomination **unlimited times** (unbounded). Return `-1` if it's impossible.

```
coins = [1, 5, 6, 9], amount = 11

Options:
  11 = 1+1+9 = 3 coins
  11 = 5+6   = 2 coins ← minimum
  11 = 1+1+1+1+1+6 = 6 coins

Answer: 2
```

---

## 2. 0/1 Knapsack vs Coin Change — The Unbounded Difference

### 0/1 Knapsack
Each item can be used **at most once**. Once you pick item `ind`, you move to `ind-1`.

### Coin Change (Unbounded Knapsack)
Each coin can be used **unlimited times**. Once you pick coin `ind`, you **stay at `ind`** — the same coin can be picked again.

```
0/1 Knapsack take:   1 + fn(ind - 1, target - wt[ind])  ← move to ind-1
Coin Change take:    1 + fn(ind,     target - coins[ind]) ← STAY at ind
```

**This is the entire difference.** Not moving to `ind-1` means the same coin can be reused on the next recursive call.

---

## 3. Why `fn(ind, target - coins[ind])` NOT `fn(ind-1, ...)`

When you take a coin of denomination `coins[ind]`:
- You add 1 to your coin count
- You reduce the remaining target by `coins[ind]`
- **You can pick this same coin again** → stay at `ind`

If you wrote `fn(ind-1, target - coins[ind])`:
- After picking coin `ind`, you'd move to coin `ind-1`
- You could never pick coin `ind` a second time
- That's 0/1 knapsack, not coin change

```
Example: coins=[2], amount=6

fn(0, 6):
  take: 1 + fn(0, 4)   ← still at ind=0 (can take coin 2 again!)
    take: 1 + fn(0, 2)
      take: 1 + fn(0, 0)
        base: 0%2==0 → 0/2=0 coins → return 0
      → 1+0 = 1
    → 1+1 = 2
  → 1+2 = 3

Answer: 3 coins of value 2 = 6 ✅

With fn(ind-1,...): would be wrong — can't reuse same coin
```

---

## 4. TC is NOT `O(2^n)` — It's More

### Why 0/1 Knapsack was `O(2^n)`

In 0/1 knapsack, each item is considered once — you either take it (move to `ind-1`) or skip it (move to `ind-1`). The recursion tree has depth `n` with 2 branches → `O(2^n)`.

### Why Coin Change is Worse

The `take` branch does NOT decrease `ind`. You can take the same coin multiple times until the target reaches 0. The recursion tree has:
- `notTake` branch: depth reduces by 1 (moves to `ind-1`)
- `take` branch: depth does NOT reduce! Repeats at `ind` for `target / coins[ind]` steps

The number of take-steps for coin `ind` can be `target / coins[ind]` in the worst case. For a coin of value 1 and target `T`: you can take it `T` times before the target reaches 0.

```
Worst case: coins=[1], amount=T

fn(0, T) → take → fn(0, T-1) → take → fn(0, T-2) → ... → fn(0, 0)

This is O(T) = O(amount) deep, not O(n)!
```

For the general case with `n` coins and target `T`, the recursion can branch into `target/coins[i]` sub-take-calls per coin, making the recursion tree exponential in `T` — much worse than `O(2^n)` for large amounts.

**TC without memo: `O(amount^n)` in the worst case** (very rough upper bound). With memo: `O(n × amount)`.

---

## 5. SC is NOT `O(n)` — It's `O(target)` = `O(amount)`

### Why 0/1 Knapsack Stack Depth Was `O(n)`

Each recursive call decreased `ind` by 1 → depth = n.

### Why Coin Change Stack Depth Is `O(amount)`

The `take` branch calls `fn(ind, target - coins[ind])` — `ind` doesn't change. The recursion only terminates when either:
1. `ind` reaches 0 (from `notTake` branches decreasing `ind`)
2. `target` reaches 0 (from `take` branches decreasing `target`)

In the worst case (coin of value 1), `target` decreases by 1 per call — giving a stack depth of `O(target)` = `O(amount)`.

```
Worst case: coins=[1,5], amount=T
fn(1, T): take → fn(1, T-5) → take → fn(1, T-10) → ... → fn(1, T%5) → notTake → fn(0, T%5)

Stack depth from notTake path: bounded by n (number of coins)
Stack depth from take path: bounded by T/coins[ind]

For coins[ind]=1: stack depth = T = amount
```

**SC without memo: `O(amount)` for the recursion stack** (not `O(n)`).

---

## 6. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int ind, int target, vector<int>& coins) {
        // Base: only coin[0] available — use as many as needed
        if(ind == 0) {
            if(target % coins[0] == 0)
                return target / coins[0];   // exactly divisible → this many coins
            else
                return 1e9;                  // impossible → infinity
        }

        int notTake = fn(ind-1, target, coins);   // skip this coin

        int take = 1e9;
        if(coins[ind] <= target)
            take = 1 + fn(ind, target - coins[ind]);  // ← STAY at ind (unbounded!)

        return min(take, notTake);
    }

public:
    int coinChange(vector<int>& coins, int amount) {
        int result = fn(coins.size()-1, amount, coins);
        return result != (int)1e9 ? result : -1;
    }
};
```

### Why `1e9` as Infinity?

`INT_MAX + 1` overflows. `1e9` is large enough to represent "impossible" while staying safe when adding 1 to it (`1e9 + 1` doesn't overflow). The final check `result != 1e9` converts it back to `-1`.

### The Base Case Explained

```cpp
if(target % coins[0] == 0)
    return target / coins[0];
```

When only `coins[0]` is available (unbounded), the minimum coins = `target / coins[0]` if divisible, else impossible (`1e9`).

```
coins[0]=5, target=15: 15/5=3 coins ✅
coins[0]=5, target=13: 13%5≠0 → impossible → 1e9
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(amount^n)` rough | Can take same coin `amount` times |
| **Space** | `O(amount)` | Stack depth up to `amount` steps |

---

## 7. Approach 2 — Memoization

```cpp
class Solution {
private:
    int fn(int ind, int target, vector<int>& coins, vector<vector<int>>& dp) {
        if(ind == 0) {
            if(target % coins[0] == 0) return target / coins[0];
            return 1e9;
        }

        if(dp[ind][target] != -1)
            return dp[ind][target];

        int notTake = fn(ind-1, target, coins, dp);

        int take = 1e9;
        if(coins[ind] <= target)
            take = 1 + fn(ind, target - coins[ind], coins, dp);

        return dp[ind][target] = min(take, notTake);
    }

public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        vector<vector<int>> dp(n, vector<int>(amount+1, -1));
        int result = fn(n-1, amount, coins, dp);
        return result != (int)1e9 ? result : -1;
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | n × amount unique states, O(1) each |
| **Space** | `O(n × amount) + O(amount)` | dp table + stack |

---

## 8. Approach 3 — Tabulation

### Key Observation in Tabulation

In the `take` case:
```cpp
take = 1 + dp[ind][target - coins[ind]];   // ← reads dp[ind][...], NOT dp[ind-1][...]
```

This reads from the **SAME ROW** (`dp[ind]`), not the previous row. Why?

Because coin `ind` is unbounded — `dp[ind][target - coins[ind]]` means "best result using coins `0..ind` for a smaller target." Since we can use coin `ind` again, we stay in the current row.

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        vector<vector<int>> dp(n, vector<int>(amount+1, 0));

        // Base: row 0 — only coins[0] available
        for(int target = 0; target <= amount; target++) {
            dp[0][target] = (target % coins[0] == 0) ? target/coins[0] : (int)1e9;
        }

        for(int ind = 1; ind < n; ind++) {
            for(int target = 0; target <= amount; target++) {
                int notTake = dp[ind-1][target];

                int take = (int)1e9;
                if(coins[ind] <= target)
                    take = 1 + dp[ind][target - coins[ind]];   // same row!

                dp[ind][target] = min(take, notTake);
            }
        }

        return dp[n-1][amount] != (int)1e9 ? dp[n-1][amount] : -1;
    }
};
```

### Why Same Row Is Safe Here (Left-to-Right OK!)

In 0/1 knapsack (super space optimization), we had to go **right-to-left** to avoid using the same item twice. Here we go **left-to-right** deliberately:

```
When computing dp[ind][target], take reads dp[ind][target - coins[ind]]
where target - coins[ind] < target (smaller index, already computed in this iteration)
→ that value already includes the option of using coin[ind] again ← DESIRED!
```

This is the unbounded property in action: reading from the already-updated same row allows coin reuse.

### Tabulation Trace (coins=[1,2,5], amount=5)

```
Row 0 (coins[0]=1): dp[0][t] = t (t coins of value 1)
  [0, 1, 2, 3, 4, 5]

Row 1 (coins[1]=2):
  t=0: notTake=0, take(2>0)=no → 0
  t=1: notTake=dp[0][1]=1, take(2>1)=no → 1
  t=2: notTake=dp[0][2]=2, take=1+dp[1][0]=1 → min(2,1)=1
  t=3: notTake=3, take=1+dp[1][1]=2 → min(3,2)=2
  t=4: notTake=4, take=1+dp[1][2]=2 → min(4,2)=2
  t=5: notTake=5, take=1+dp[1][3]=3 → min(5,3)=3

Row 1: [0, 1, 1, 2, 2, 3]

Row 2 (coins[2]=5):
  t=0..4: notTake=row1, take(5>t)=no
  t=5: notTake=dp[1][5]=3, take=1+dp[2][0]=1 → min(3,1)=1

Row 2: [0, 1, 1, 2, 2, 1]

Answer: dp[2][5] = 1 ✅ (one coin of value 5)
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | Two nested loops |
| **Space** | `O(n × amount)` | 2D dp array |

---

## 9. Approach 4 — Space Optimization

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        vector<int> prev(amount+1, 0);

        for(int target = 0; target <= amount; target++)
            prev[target] = (target % coins[0] == 0) ? target/coins[0] : (int)1e9;

        for(int ind = 1; ind < n; ind++) {
            vector<int> cur(amount+1, 0);

            for(int target = 0; target <= amount; target++) {
                int notTake = prev[target];

                int take = (int)1e9;
                if(coins[ind] <= target)
                    take = 1 + cur[target - coins[ind]];   // cur not prev! (same row)

                cur[target] = min(take, notTake);
            }

            prev = cur;
        }

        return prev[amount] >= (int)1e9 ? -1 : prev[amount];
    }
};
```

### `cur[target - coins[ind]]` Not `prev[target - coins[ind]]`

This is the crucial difference from 0/1 knapsack space optimization:

```
0/1 Knapsack take: pick = val[ind] + prev[cap - wt[ind]]  ← prev (previous item)
Coin Change take:  take = 1 + cur[target - coins[ind]]    ← cur  (current coin, reuse!)
```

Reading from `cur` (currently being filled, left-to-right) means we can pick coin `ind` multiple times — the unbounded property.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × amount)` | Same loops |
| **Space** | `O(2 × amount)` = `O(amount)` | `prev` + `cur` arrays |

---

## 10. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(amount^n)` rough | `O(amount)` stack | Not O(2^n), not O(n) |
| **Memoization** | `O(n × amount)` | `O(n × amount + amount)` | |
| **Tabulation** | `O(n × amount)` | `O(n × amount)` | take reads same row |
| **Space Opt** | `O(n × amount)` | `O(amount)` | cur not prev for take |

---

## 11. 0/1 Knapsack vs Coin Change (Unbounded) — Complete Comparison

| | 0/1 Knapsack | Coin Change (Unbounded) |
|---|---|---|
| **Each item usable** | Once | Unlimited |
| **Take branch** | `fn(ind-1, target-wt)` | `fn(ind, target-coin)` ← same ind |
| **Tabulation take** | `dp[ind-1][target-wt]` | `dp[ind][target-coin]` ← same row |
| **Space opt take** | `prev[cap - wt]` | `cur[target - coin]` |
| **Space opt direction** | Right → Left | Left → Right |
| **Recursion SC** | `O(n)` | `O(amount)` |
| **Recursion TC** | `O(2^n)` | `O(amount^n)` rough |
