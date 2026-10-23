# Rod Cutting — Complete DP Notes (Every Approach Explained Separately)

## Problem statement

Given a rod of length **`n` inches** and an array `price[]`, where `price[i]` gives the value of a piece of length `i + 1`, find the maximum value obtainable by cutting the rod into pieces and selling them.

**Note:** `n == price.size()`. The array is 0-indexed, but piece lengths are 1-indexed.

### Examples

**Example 1**

- **Input:** `price = [1, 5, 8, 9, 10, 17, 17, 20]`
- **Output:** `22`
- **Explanation:** Choose lengths 2 and 6, worth `5 + 17 = 22`.

**Example 2**

- **Input:** `price = [3, 5, 8, 9, 10, 17, 17, 20]`
- **Output:** `24`
- **Explanation:** Choose eight pieces of length 1, worth `8 * 3 = 24`.

**Example 3**

- **Input:** `price = [3]`
- **Output:** `3`
- **Explanation:** The rod has length 1, so the only possible piece has length 1.

### Constraints

- `1 <= price.size() <= 10^3`
- `1 <= price[i] <= 10^6`

---

# A shared foundation: understand the problem before each solution

This section establishes the reasoning used throughout the solutions. Each approach below then has its own intuition, algorithm/dry run, story points, exact code, and complexity analysis.

## How I can turn a hard problem into a familiar one

Instead of immediately trying to enumerate every way to cut a rod, ask: **What decision can I make now, and what smaller problem remains after that decision?**

For a rod with remaining length `N`, suppose the largest piece length currently being considered is `ind + 1`.

There are two choices:

1. **Not take** a piece of length `ind + 1`. Move to the previous piece length, `ind - 1`, while the rod length remains `N`.
2. **Take** a piece of length `ind + 1`, earn `price[ind]`, and solve the remaining rod of length `N - (ind + 1)`.

The important detail is that after taking a piece, we keep the same `ind`. We are allowed to cut multiple pieces of the same length.

## The connection to 0/1 knapsack

Rod Cutting resembles knapsack because each possible piece length has:

- a **weight**: its length;
- a **value**: its selling price;
- a **capacity limit**: the total rod length `n`.

We can map the problem as follows:

| Knapsack concept | Rod Cutting equivalent |
|---|---|
| Capacity | Remaining rod length `N` |
| Item type | A piece length |
| Item weight | Piece length `ind + 1` |
| Item value | `price[ind]` |
| Take | Cut and sell one piece of that length |
| Not take | Stop considering that piece length |

In **0/1 knapsack**, taking an item normally moves to the previous item index, because that item cannot be reused. Rod Cutting is instead **unbounded knapsack**: we may use a piece length repeatedly. That is why the take transition keeps the same index.

This is a useful way to break down a difficult problem: identify the choices, compare them with a known pattern, and reuse the known recurrence.

## State and recurrence

Let `f(ind, N)` mean:

> The maximum value obtainable from a rod of remaining length `N`, using piece lengths from `1` through `ind + 1`.

For `ind > 0`:

\[
\text{notTake}=f(ind-1,N)
\]

If `ind + 1 <= N`:

\[
\text{take}=price[ind]+f(ind,N-(ind+1))
\]

Then:

\[
f(ind,N)=\max(\text{notTake},\text{take})
\]

The take option is only valid when the piece fits.

### Base case

When `ind == 0`, only length-1 pieces are available. A rod of length `N` can be sold as `N` pieces of length 1:

\[
f(0,N)=N\times price[0]
\]

This base case is important: it must use `N * price[0]`, not just `price[0]`.

---

# Solution 1 — Recursion

## 1. Intuition / thought process

Start with the full rod, whose length is `n`. Initially, all piece lengths from 1 to `n` are available, so the first call is `f(n - 1, n, price)`.

At each state, consider the current piece length `ind + 1`.

- If we **do not take** it, we cannot use that piece length again, so we move to `ind - 1`. The remaining rod length stays `N`.
- If we **take** it, we add `price[ind]` to our value and reduce the remaining rod length by `ind + 1`. We keep `ind` unchanged because another piece of the same length may still be used.

We calculate both possibilities and return the larger value.

This is the most direct expression of the decision-making process. It is easy to reason about, but the same state can be evaluated many times. That repeated work is the reason to introduce memoization next.

## 2. Algorithm / dry run

### Algorithm

1. Set `rodLength = ind + 1`.
2. If `ind == 0`, return `N * price[0]`.
3. Compute `notTake = f(ind - 1, N, price)`.
4. Initialize `take` to `INT_MIN` so an impossible take is not selected.
5. If `rodLength <= N`, compute `take = price[ind] + f(ind, N - rodLength, price)`.
6. Return `max(notTake, take)`.
7. Start from `f(n - 1, n, price)`.

### Dry run

Use `price = [2, 5, 7, 8]`, with `n = 4`. The best answer is 10, obtained by cutting the rod into two pieces of length 2.

Consider state `f(1, 4)`:

- `ind = 1`, so `rodLength = 2` and `price[1] = 5`.
- **Not take:** `f(0, 4) = 4 * 2 = 8`, because only length-1 pieces remain available.
- **Take:** take a length-2 piece for value 5. The remaining length is 2, and we keep `ind = 1`: `5 + f(1, 2)`.
- For `f(1, 2)`, not taking gives `f(0, 2) = 4`; taking gives `5 + f(1, 0) = 5`.
- Therefore `f(1, 2) = 5`, and the take option for `f(1, 4)` is `5 + 5 = 10`.
- Choose `max(8, 10) = 10`.

This shows how keeping the same index after taking allows the length-2 piece to be used twice.

## 3. Story points

- The state needs two changing quantities: the current piece index `ind` and remaining rod length `N`.
- `ind + 1` converts the 0-based array index into the actual piece length.
- The **not-take** branch changes the index but keeps the capacity unchanged.
- The **take** branch reduces capacity but keeps the index unchanged.
- Keeping the same index in the take branch is the unbounded-knapsack clue.
- The base case handles the situation where only length-1 pieces are available.
- Pure recursion is a good first implementation because it mirrors the recurrence, even though it is not fast enough for large inputs.

## 4. Code

```cpp
class Solution {
private:
    int f(int ind, int N, vector<int>& price) {
        int rodLength = ind + 1;

        if (ind == 0)
            return N * price[0];

        int notTake = 0 + f(ind - 1, N, price);

        int take = INT_MIN;

        if (rodLength <= N)
            take = price[ind] + f(ind, N - rodLength, price);

        return max(notTake, take);
    }

public:
    int cutRod(vector<int>& price) {
        int n = price.size();

        return f(n - 1, n, price);
    }
};
```

## 5. Time and space complexity analysis

### Time: exponential

The recursion branches into take and not-take choices. Those branches can lead to repeated calculations of the same `(ind, N)` state. A rough upper-bound description is exponential, often written as `O(2^n)` for this branching recursion. The exact number of calls depends on the states and the take conditions, but the important point is that the running time grows exponentially in the worst case.

### Auxiliary space: `O(n)` recursion stack

The recursion stack grows along a path. The index can decrease at most `n - 1` times, and taking a piece decreases the remaining length by at least 1. Thus the maximum stack depth is linear in `n`, giving `O(n)` auxiliary stack space.

---

# Solution 2 — Memoization (top-down DP)

## 1. Intuition / thought process

The recursive solution has the right logic, but it can solve the same state repeatedly. For example, different cutting decisions can lead to the same pair `(ind, N)`.

The key observation is:

> If the answer for a particular `(ind, N)` is already known, there is no need to calculate it again.

Create a 2D array:

`dp[ind][N]`

It stores the maximum value for that state. Initialize every entry to `-1` to mean “not calculated yet.”

When `f(ind, N)` is called:

1. Handle the base case.
2. If `dp[ind][N] != -1`, return the saved answer.
3. Otherwise calculate take and not-take as before.
4. Store the maximum in `dp[ind][N]` and return it.

The recurrence has not changed. Memoization changes how often each state is evaluated.

## 2. Algorithm / dry run

### Algorithm

1. Create `dp` with `n` rows and `n + 1` columns, filled with `-1`.
2. Call `f(n - 1, n, price, dp)`.
3. For each state, return `N * price[0]` when `ind == 0`.
4. If the state is already stored, return `dp[ind][N]`.
5. Otherwise compute not-take and, if it fits, take.
6. Store `max(notTake, take)` in `dp[ind][N]`.
7. Return the stored result.

### Dry run

Use the same example: `price = [2, 5, 7, 8]`, `n = 4`.

Suppose the function reaches `f(1, 2)`:

- It checks `dp[1][2]`.
- If the value is `-1`, the state has not been computed, so it evaluates the recurrence.
- Not take gives `f(0, 2) = 4`.
- Take gives `5 + f(1, 0) = 5`.
- The maximum is 5, so it stores `dp[1][2] = 5`.

If another recursive path later asks for `f(1, 2)`, the function sees `dp[1][2] == 5` and returns 5 immediately. It does not solve the subproblem again.

The final answer remains 10. The improvement is avoiding duplicate work, not changing the answer.

## 3. Story points

- Memoization is **top-down DP** because it starts from the original problem and recursively visits the required subproblems.
- The state remains `(ind, N)`.
- `dp` has `n` rows for indices `0` through `n - 1`.
- It has `n + 1` columns because capacities range from `0` through `n`, inclusive.
- `-1` is a safe “not computed” marker here because all prices are positive and all achievable values are nonnegative.
- Each state is calculated at most once, so the number of state calculations is `O(n^2)`.
- Recursion is still used, so there is an additional call-stack cost.

## 4. Code

```cpp
class Solution {
private:
    int f(int ind, int N, vector<int>& price, vector<vector<int>>& dp) {
        int rodLength = ind + 1;

        if (ind == 0)
            return N * price[0];

        if (dp[ind][N] != -1)
            return dp[ind][N];

        int notTake = 0 + f(ind - 1, N, price, dp);

        int take = INT_MIN;

        if (rodLength <= N)
            take = price[ind] + f(ind, N - rodLength, price, dp);

        return dp[ind][N] = max(notTake, take);
    }

public:
    int cutRod(vector<int>& price) {
        int n = price.size();

        vector<vector<int>> dp(n, vector<int>(n + 1, -1));
        return f(n - 1, n, price, dp);
    }
};
```

## 5. Time and space complexity analysis

### Time: `O(n^2)`

There are `n * (n + 1)` possible states. Each state is computed at most once, and each calculation performs constant work besides recursive calls. Therefore the time complexity is `O(n^2)`.

### Auxiliary space: `O(n^2)` table + `O(n)` stack

- Memoization table: `O(n^2)`.
- Maximum recursion stack depth: `O(n)`.

Total auxiliary space is `O(n^2)`, since the table dominates the stack asymptotically.

---

# Solution 3 — Tabulation (bottom-up DP)

## 1. Intuition / thought process

Memoization avoids repeated work, but it still relies on recursive calls. Tabulation removes recursion by calculating the states in an order that guarantees the needed smaller answers are already available.

Use the same state meaning:

`dp[ind][N]` = maximum value for capacity `N` using piece lengths up to `ind + 1`.

The base row is known directly. When only length-1 pieces are allowed, a rod of length `N` earns `N * price[0]`:

`dp[0][N] = N * price[0]`

Then build each subsequent row:

- `dp[ind - 1][N]` is the not-take answer from the previous row.
- `dp[ind][N - rodLength]` is the take answer from the current row.

The current row is necessary for the take option because this is unbounded knapsack: we may reuse the current piece length. Capacities are processed in ascending order so `dp[ind][N - rodLength]` is already available.

## 2. Algorithm / dry run

### Algorithm

1. Create an `n x (n + 1)` table.
2. Initialize the base row: `dp[0][rodLen] = rodLen * price[0]`.
3. For every `ind` from `1` to `n - 1`:
   - For every capacity `N` from `0` to `n`:
   - Set `notTake = dp[ind - 1][N]`.
   - If `ind + 1 <= N`, set `take = price[ind] + dp[ind][N - (ind + 1)]`.
   - Store the maximum in `dp[ind][N]`.
4. Return `dp[n - 1][n]`.

### Dry run

Use `price = [2, 5, 7, 8]`, `n = 4`.

The base row for length-1 pieces is:

| Capacity `N` | 0 | 1 | 2 | 3 | 4 |
|---|---:|---:|---:|---:|---:|
| `dp[0][N]` | 0 | 2 | 4 | 6 | 8 |

Now process `ind = 1`, which represents pieces of length 2 and price 5:

- Capacity 0: length 2 does not fit, so value is 0.
- Capacity 1: length 2 does not fit, so value stays 2 from the previous row.
- Capacity 2: `max(dp[0][2], 5 + dp[1][0]) = max(4, 5) = 5`.
- Capacity 3: `max(dp[0][3], 5 + dp[1][1]) = max(6, 7) = 7`.
- Capacity 4: `max(dp[0][4], 5 + dp[1][2]) = max(8, 10) = 10`.

The take expression for capacity 4 reads `dp[1][2]` from the same row. Since capacity 2 was computed earlier, it already contains the best value using any number of length-2 pieces.

The final state `dp[3][4]` is 10, so the maximum value for the full rod is 10.

## 3. Story points

- Tabulation is **bottom-up DP**: no recursive calls are needed.
- The base row is initialized before the remaining states are computed.
- The order of loops matters because a state can depend on an earlier capacity in the same row.
- `dp[ind - 1][N]` represents skipping the current piece length.
- `dp[ind][N - rodLength]` represents taking it and potentially taking it again.
- The final answer is `dp[n - 1][n]`, not `dp[n - 1][n + 1]`; valid columns are `0` through `n`.
- The 2D table is easy to understand, but it stores every state even though only the previous row and part of the current row are needed.

## 4. Code

```cpp
class Solution {
public:
    int cutRod(vector<int>& price) {
        int n = price.size();

        vector<vector<int>> dp(n, vector<int>(n + 1, 0));

        for (int rodLen = 0; rodLen <= n; rodLen++)
            dp[0][rodLen] = rodLen * price[0];

        for (int ind = 1; ind < n; ind++) {
            for (int N = 0; N <= n; N++) {
                int notTake = 0 + dp[ind - 1][N];

                int take = INT_MIN;
                int rodLength = ind + 1;

                if (rodLength <= N)
                    take = price[ind] + dp[ind][N - rodLength];

                dp[ind][N] = max(notTake, take);
            }
        }

        return dp[n - 1][n];
    }
};
```

## 5. Time and space complexity analysis

### Time: `O(n^2)`

The outer loop visits `n - 1` rows after initialization, and the inner loop visits `n + 1` capacities. Each state takes constant work, giving `O(n^2)` time.

### Auxiliary space: `O(n^2)`

The DP table has `n * (n + 1)` entries, so its space requirement is `O(n^2)`. There is no recursive stack.

---

# Solution 4 — Space optimization with `prev` and `cur`

## 1. Intuition / thought process

Look at the tabulation recurrence:

- Not take reads `dp[ind - 1][N]`, from the previous row.
- Take reads `dp[ind][N - rodLength]`, from the current row at a smaller capacity.

We do not need to preserve every old row. We only need:

- `prev[N]`: the previous row's answer for capacity `N`;
- `cur[N]`: the current row's answer for capacity `N`.

The base row is stored in `prev`. For each new piece length, calculate the new row in `cur`, then copy it to `prev` with `prev = cur`.

The current-row dependency explains why the take expression must use `cur[N - rodLength]`. Since `N` is processed from low to high, that smaller capacity has already been computed.

## 2. Algorithm / dry run

### Algorithm

1. Create `prev` with `n + 1` entries.
2. Initialize `prev[N] = N * price[0]`.
3. For each `ind` from `1` to `n - 1`, create a new `cur` array.
4. Process capacities `N` in ascending order:
   - `notTake = prev[N]`.
   - If the piece fits, `take = price[ind] + cur[N - rodLength]`.
   - Set `cur[N] = max(notTake, take)`.
5. After completing the row, assign `prev = cur`.
6. Return `prev[n]`.

### Dry run

Use `price = [2, 5, 7, 8]`, `n = 4`.

Initially, `prev = [0, 2, 4, 6, 8]`, representing the base row where only length-1 pieces are available.

For `ind = 1`, piece length is 2 and price is 5. A new array `cur` is filled left to right:

- `N = 0`: length 2 does not fit, so `cur[0] = prev[0] = 0`.
- `N = 1`: length 2 does not fit, so `cur[1] = prev[1] = 2`.
- `N = 2`: take gives `5 + cur[0] = 5`; not take gives 4. So `cur[2] = 5`.
- `N = 3`: take gives `5 + cur[1] = 7`; not take gives 6. So `cur[3] = 7`.
- `N = 4`: take gives `5 + cur[2] = 10`; not take gives 8. So `cur[4] = 10`.

After this row, `prev = cur`. The next piece length is processed the same way. At the end, `prev[4] = 10`.

## 3. Story points

- This optimization uses the dependency structure of the 2D table.
- The previous row is needed for not-take; the current row is needed for take.
- Two arrays are enough, so the space drops from `O(n^2)` to `O(n)`.
- `cur[N - rodLength]` must be from the current row, not `prev`, because the recurrence uses the current item type repeatedly.
- The ascending capacity order makes that current-row value available.
- `prev = cur` moves the completed row forward for the next piece length.

## 4. Code

```cpp
class Solution {
public:
    int cutRod(vector<int>& price) {
        int n = price.size();

        vector<int> prev(n + 1, 0);

        for (int rodLen = 0; rodLen <= n; rodLen++)
            prev[rodLen] = rodLen * price[0];

        for (int ind = 1; ind < n; ind++) {
            vector<int> cur(n + 1, 0);

            for (int N = 0; N <= n; N++) {
                int notTake = 0 + prev[N];

                int take = INT_MIN;
                int rodLength = ind + 1;

                if (rodLength <= N)
                    take = price[ind] + cur[N - rodLength];

                cur[N] = max(notTake, take);
            }

            prev = cur;
        }

        return prev[n];
    }
};
```

## 5. Time and space complexity analysis

### Time: `O(n^2)`

The same number of DP states is computed as in tabulation. Reducing stored rows does not reduce the number of transitions.

### Auxiliary space: `O(n)`

Two arrays of length `n + 1` are stored. This is `O(2n)`, which simplifies to `O(n)`. A new `cur` vector is created for each outer iteration, but only two length-`n + 1` arrays are needed at a time.

---

# Solution 5 — One-array (super space optimization)

## 1. Intuition / thought process

The two-array solution stores both `prev` and `cur`, but perhaps even `cur` can be removed.

In the two-array solution, when calculating a capacity `N`:

- `notTake` comes from `prev[N]`;
- `take` comes from `cur[N - rodLength]`.

Because capacities are processed in ascending order, `cur[N - rodLength]` is already calculated. In a single array, we can let `prev` represent both roles:

- `prev[N]` still contains the previous row's value until this capacity is updated, so it can be saved as `notTake`;
- `prev[N - rodLength]` has already been updated for the current piece length, so it can be used for `take`.

That means the same array can represent the previous row at untouched capacities and the current row at already-processed capacities.

**The loop direction is essential.** Ascending capacity order makes the lower-capacity entry contain the current-row result, which permits repeated use of the current piece length.

## 2. Algorithm / dry run

### Algorithm

1. Create one array `prev` of length `n + 1`.
2. Initialize `prev[N] = N * price[0]`.
3. For each `ind` from `1` to `n - 1`:
   - Set `rodLength = ind + 1`.
   - Iterate `N` from `0` to `n` in ascending order.
   - Save `notTake = prev[N]` before overwriting it.
   - If the piece fits, calculate `take = price[ind] + prev[N - rodLength]`.
   - Overwrite `prev[N]` with the maximum of take and not-take.
4. Return `prev[n]`.

### Dry run

Use `price = [2, 5, 7, 8]`, `n = 4`.

Initially:

`prev = [0, 2, 4, 6, 8]`

Process `ind = 1`, where `rodLength = 2` and `price[ind] = 5`:

- `N = 0`: `notTake = 0`, length 2 does not fit. `prev[0]` stays 0.
- `N = 1`: `notTake = 2`, length 2 does not fit. `prev[1]` stays 2.
- `N = 2`: save `notTake = 4`. `prev[0]` is 0, so `take = 5 + 0 = 5`. Update `prev[2] = 5`.
- `N = 3`: save `notTake = 6`. `prev[1]` is already updated for this row and equals 2, so `take = 5 + 2 = 7`. Update `prev[3] = 7`.
- `N = 4`: save `notTake = 8`. `prev[2]` was already updated to 5, so `take = 5 + 5 = 10`. Update `prev[4] = 10`.

After this iteration, the array contains the best values for capacities using piece lengths 1 and 2. The remaining piece lengths are processed in the same way. The final answer is `prev[4] = 10`.

### Why ascending order matters

At capacity 4, the take transition needs the result for capacity `4 - 2 = 2` using the current piece length. With ascending order, capacity 2 has already been updated to 5, so capacity 4 can use it to make `5 + 5 = 10`.

If capacities were processed in descending order, `prev[2]` would still hold the previous-row value when capacity 4 was calculated. That would fail to represent the current-row unbounded recurrence and could prevent reusing the same piece length correctly.

## 3. Story points

- The one-array optimization comes from carefully examining which cells are needed, not from changing the recurrence.
- Save `notTake` before overwriting `prev[N]`.
- Ascending capacity order lets `prev[N - rodLength]` act like the current row's smaller-capacity value.
- This is the standard in-place update pattern for **unbounded knapsack**.
- In **0/1 knapsack**, a one-array capacity loop is generally descending, so the current item is not reused in the same iteration.
- The time complexity remains `O(n^2)`; only auxiliary space improves.
- The answer remains in `prev[n]`.

## 4. Code

```cpp
class Solution {
public:
    int cutRod(vector<int>& price) {
        int n = price.size();

        vector<int> prev(n + 1, 0);

        for (int rodLen = 0; rodLen <= n; rodLen++)
            prev[rodLen] = rodLen * price[0];

        for (int ind = 1; ind < n; ind++) {
            for (int N = 0; N <= n; N++) {
                int notTake = 0 + prev[N];

                int take = INT_MIN;
                int rodLength = ind + 1;

                if (rodLength <= N)
                    take = price[ind] + prev[N - rodLength];

                prev[N] = max(notTake, take);
            }
        }

        return prev[n];
    }
};
```

## 5. Time and space complexity analysis

### Time: `O(n^2)`

There are `n` piece-length iterations and `n + 1` capacity iterations. Each state is updated in constant time, giving `O(n^2)` time.

### Auxiliary space: `O(n)`

Only one vector of length `n + 1` is used, so the extra space is `O(n)`. This is the lowest space usage among the listed implementations while preserving the same `O(n^2)` time complexity.

---

# Final comparison

| Approach | Main idea | Time | Auxiliary space |
|---|---|---:|---:|
| Recursion | Explore take and not-take choices directly | Exponential worst case | `O(n)` stack |
| Memoization | Cache recursive state answers | `O(n^2)` | `O(n^2)` table + `O(n)` stack |
| Tabulation | Compute every state bottom-up | `O(n^2)` | `O(n^2)` |
| Two arrays | Keep only previous and current rows | `O(n^2)` | `O(n)` |
| One array | Update capacities in place, ascending | `O(n^2)` | `O(n)` |

# Final mental checklist for solving similar DP problems

1. **Find the decisions.** What can I take, skip, choose, or change?
2. **Define the state.** What information fully describes a subproblem?
3. **Write the recurrence.** Express the state using smaller states.
4. **Identify the pattern.** Here, capacity + value + reusable choices gives unbounded knapsack.
5. **Choose the correct transition.** Reuse allowed means the take branch stays at the same index.
6. **Write the base case.** Ensure it accounts for the entire remaining capacity.
7. **Start with recursion.** It reveals the logical structure.
8. **Add memoization.** Cache repeated states.
9. **Convert to tabulation.** Find an order where dependencies are already computed.
10. **Optimize space last.** Inspect dependencies and choose loop direction carefully.

The main lesson is that a hard-looking problem becomes more manageable when it is reduced to familiar decisions and states. Recognizing Rod Cutting as unbounded knapsack gives you a reusable framework for reasoning about it and many other dynamic-programming problems.
