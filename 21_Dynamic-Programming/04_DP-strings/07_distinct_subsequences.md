# Distinct Subsequences

---

## 1. Problem Statement

Given two strings `s` and `t`, return the **number of distinct subsequences of `s` that equal `t`**.

Two subsequences are different if they use **different index positions** in `s`, even if the characters look identical.

```
s = "babgbag", t = "bag"

index:  0 1 2 3 4 5 6
s    :  b a b g b a g

The 5 ways to pick "bag":
  (0, 1, 3)  →  b a . g . . .
  (0, 1, 6)  →  b a . . . . g
  (0, 5, 6)  →  b . . . . a g
  (2, 5, 6)  →  . . b . . a g
  (4, 5, 6)  →  . . . . b a g

Answer: 5
```

---

## 2. The Thought Process Behind the Recursion

This follows the four steps from your notes: express in indexes, explore all possibilities, combine, base cases.

### Step 1: Express the problem in terms of indexes

```
f(i, j) = number of distinct subsequences of s[0..i] that equal t[0..j]
```

We want `f(n-1, m-1)`. Two pointers walk from the **end** of both strings toward the front: `i` on `s`, `j` on `t`.

### Step 2: Explore all possibilities

Look at the last characters `s[i]` and `t[j]`. There are two situations.

#### Situation A: `s[i] == t[j]` (match)

We have **two valid choices**, and both lead to real answers:

**Choice 1: use `s[i]` to match `t[j]`.**
This pair is settled, so both strings shrink: `f(i-1, j-1)`.

**Choice 2: do NOT use `s[i]`.**
Your note says it well: *"we don't want that g, we have another occurrence of g which could be in the remaining `s`."* The same `t[j]` might be matched by an **earlier** occurrence of that character further left in `s`. So only `s` shrinks, `t` stays: `f(i-1, j)`.

```
s = b a b g b a g        t = b a g
                  ↑ i                  ↑ j      s[i]='g' == t[j]='g'

Choice 1: use this last 'g'      → f(i-1, j-1)   (now find "ba" in "babgba")
Choice 2: skip it, use an earlier 'g'  → f(i-1, j)  (find "bag" in "babgba")
```

These two choices cover **disjoint** sets of subsequences (one uses index `i`, the other doesn't), so nothing is counted twice and nothing is missed.

```
if (s[i] == t[j])
    return f(i-1, j-1) + f(i-1, j);
```

#### Situation B: `s[i] != t[j]` (mismatch)

`s[i]` cannot serve as `t[j]`, so it is useless for this position. We must look further left in `s` to find a match for `t[j]`:

```
else
    return f(i-1, j);
```

Only `s` shrinks.

### Why does only `s` ever shrink on its own, never `t`?

`t` is the **pattern** that must be matched in full. Every character of `t` has to be found somewhere. `s` is the **source** we are allowed to skip characters from. So:

- skipping a character of `s` is always legal
- skipping a character of `t` is never legal

`j` only moves when a real match happens.

### Step 3: Return the summation of all possibilities

This is a **counting** problem, so when there are two valid choices we **add** their counts (compare with an optimization problem, where we would take `max`/`min`).

### Step 4: Base cases

```cpp
if (j < 0) return 1;   // t is fully matched
if (i < 0) return 0;   // s is exhausted but t still has characters left
```

- **`j < 0` → return 1:** every character of `t` has been matched. This path is exactly **one valid way**.
- **`i < 0` → return 0:** no characters of `s` are left, but `t` still has unmatched characters. This path failed.

**The order matters.** The `j < 0` check comes first. If both `i < 0` and `j < 0`, then `t` is fully matched and we must return `1`. Checking `i < 0` first would wrongly return `0`.

```
Example: s = "b", t = "b"
f(0,0): match → f(-1,-1) + f(-1,0)
   f(-1,-1): j<0 first → 1  ✅ (t fully matched using the single 'b')
   f(-1, 0): j=0, i<0 → 0
= 1 + 0 = 1 ✅
```

### How this differs from LCS

| | LCS | Distinct Subsequences |
|---|---|---|
| **On match** | `1 + f(i-1, j-1)` (only the diagonal) | `f(i-1, j-1) + f(i-1, j)` (diagonal **and** skip) |
| **Why** | Matching is always at least as good | Both ways are valid distinct subsequences, so both are counted |
| **Combine** | `max` | `+` |

---

## 3. Approach 1: Recursion

```cpp
class Solution {
private:
    int f(int i, int j, string& s1, string& s2) {
        if (j < 0) return 1;   // t fully matched
        if (i < 0) return 0;   // s exhausted, t not finished

        if (s1[i] == s2[j])
            return f(i - 1, j - 1, s1, s2) + f(i - 1, j, s1, s2);
        else
            return f(i - 1, j, s1, s2);
    }

public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();
        return f(n - 1, m - 1, s, t);
    }
};
```

### Dry run: `s = "bbag"`, `t = "bag"` (answer 2)

```
f(3,2): s[3]='g' == t[2]='g'  →  f(2,1) + f(2,2)

  f(2,1): s[2]='a' == t[1]='a'  →  f(1,0) + f(1,1)

     f(1,0): s[1]='b' == t[0]='b'  →  f(0,-1) + f(0,0)
        f(0,-1): j<0 → 1
        f(0, 0): s[0]='b' == t[0]='b' → f(-1,-1) + f(-1,0)
                 f(-1,-1): j<0 → 1
                 f(-1, 0): i<0 → 0
                 = 1
        = 1 + 1 = 2        (use b@1, or use b@0)

     f(1,1): s[1]='b' != t[1]='a' → f(0,1) → 'b' != 'a' → f(-1,1) → 0
        = 0

     f(2,1) = 2 + 0 = 2

  f(2,2): 'a' != 'g' → f(1,2) → 'b' != 'g' → f(0,2) → 'b' != 'g' → f(-1,2) → 0
        = 0

f(3,2) = 2 + 0 = 2 ✅   ( b@0 a@2 g@3 and b@1 a@2 g@3 )
```

### Complexity (a correction to the notes)

Your notes mention `2ⁿ × 2ᵐ` for time and `O(n+m)` for space. These can be tightened:

- **Every call decreases `i` by 1.** So the recursion depth is at most `n + 1`.
- A call branches into **two** only on a match, and `j` decreases only along with `i`. So `m` does not add a separate exponential factor.

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Binary branching on matches, depth bounded by `n` |
| **Space** | `O(n)` | Stack depth ≤ `n + 1` (not `n + m`) |

---

## 4. Approach 2: Memoization

### Why memoization works here

The recursion has **overlapping subproblems**. The same `(i, j)` is reached through different paths. Take a match at `(i, j)`:

```
f(i, j) → f(i-1, j-1)   and   f(i-1, j)

From f(i-1, j)   (if it matches too) → f(i-2, j-1) and f(i-2, j)
From f(i-1, j-1) (skip branch)       → f(i-2, j-1)  ...

f(i-2, j-1) is reached from TWO different parents → computed twice.
```

With many matching characters, this duplication multiplies exponentially.

### How to apply it

The state is fully described by two changing parameters `(i, j)`, so a 2D table is enough.

1. Create `dp` filled with `-1` ("not computed"). `-1` is a safe sentinel because counts are never negative.
2. After the base cases, **check** `dp[i][j]` first.
3. **Store** the result in `dp[i][j]` before returning.

The base cases return **before** the `dp` check. They cannot be stored in the table anyway, because `i` or `j` is `-1` there (no negative index).

```cpp
class Solution {
private:
    int f(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if (j < 0) return 1;
        if (i < 0) return 0;

        if (dp[i][j] != -1)
            return dp[i][j];                     // already solved

        if (s1[i] == s2[j])
            return dp[i][j] =
                       f(i - 1, j - 1, s1, s2, dp) + f(i - 1, j, s1, s2, dp);
        else
            return dp[i][j] = f(i - 1, j, s1, s2, dp);
    }

public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, -1));
        return f(n - 1, m - 1, s, t, dp);
    }
};
```

> The table is `(n+1) × (m+1)` here, one larger than needed. Valid indices are `0..n-1` and `0..m-1`, so `n × m` would be enough. The extra row/column is harmless.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | `n × m` states, `O(1)` work each |
| **Space** | `O(n × m) + O(n)` | dp table + recursion stack (depth ≤ `n + 1`) |

---

## 5. Why We Shift to 1-Based Indexing Before Tabulation

### The problem with the current base cases

The recursion base cases are `j < 0` and `i < 0`. In a table we want to **store** base cases as ordinary cells, but an array has no index `-1`. We cannot write `dp[-1][...]`.

### The fix: shift every index by +1

Change the meaning of `i` and `j` from *"position in the string"* to *"how many characters have been considered"* (prefix length):

```
Old:  i ∈ [0, n-1],  base case when i = -1
New:  i ∈ [1, n],    base case when i =  0

i = 0  →  zero characters of s considered (empty prefix)
i = 3  →  first 3 characters of s considered → s[0..2]
```

### What changes in the code

| Old (0-based) | New (1-based) |
|---|---|
| `if (j < 0) return 1` | `if (j == 0) return 1` |
| `if (i < 0) return 0` | `if (i == 0) return 0` |
| `s1[i] == s2[j]` | `s1[i-1] == s2[j-1]` (shift back to read the real character) |
| call `f(n-1, m-1)` | call `f(n, m)` |
| table `n × m` is enough | table **must be** `(n+1) × (m+1)` (index 0 is a real row/column now) |

### The shifted memoization (the bridge to tabulation)

```cpp
class Solution {
private:
    int f(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if (j == 0) return 1;          // empty t: exactly one way (pick nothing)
        if (i == 0) return 0;          // empty s, non-empty t: impossible

        if (dp[i][j] != -1)
            return dp[i][j];

        if (s1[i - 1] == s2[j - 1])    // i-1, j-1: real character positions
            return dp[i][j] =
                       f(i - 1, j - 1, s1, s2, dp) + f(i - 1, j, s1, s2, dp);
        else
            return dp[i][j] = f(i - 1, j, s1, s2, dp);
    }

public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, -1));
        return f(n, m, s, t, dp);
    }
};
```

Now `dp[i][j]` has a clean meaning:

```
dp[i][j] = number of distinct subsequences of s[0..i-1] that equal t[0..j-1]
```

and both base cases are real cells of the table (`dp[*][0]` and `dp[0][*]`).

---

## 6. Approach 3: Tabulation

### How the base cases are stored

The two base-case `if`s of the recursion become the **first column** and the **first row**:

```
j == 0 → return 1   becomes   dp[i][0] = 1  for every i
i == 0 → return 0   becomes   dp[0][j] = 0  for every j ≥ 1
```

```cpp
vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));   // everything starts at 0

for (int i = 0; i <= n; i++)
    dp[i][0] = 1;                                         // first column = 1
```

- **First column = 1:** an empty `t` can be formed from **any** prefix of `s` in exactly one way, by picking nothing.
- **First row (`j ≥ 1`) = 0:** an empty `s` cannot form a non-empty `t`. These cells are already `0` from the initialization, so no extra code is needed.
- **`dp[0][0] = 1`** (empty from empty, one way). The loop starts at `i = 0`, so it is set. This mirrors the recursion, where the `j == 0` check comes first.

### Direction of filling

`dp[i][j]` depends on `dp[i-1][j-1]` and `dp[i-1][j]`, both in the **previous row**. So we fill row by row, from `i = 1` to `n`.

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

        for (int i = 0; i <= n; i++)
            dp[i][0] = 1;                                   // base: empty t

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (s[i - 1] == t[j - 1])
                    dp[i][j] = (long long)dp[i - 1][j - 1] + dp[i - 1][j];
                else
                    dp[i][j] = dp[i - 1][j];
            }
        }

        return dp[n][m];
    }
};
```

### Table for `s = "babgbag"`, `t = "bag"`

```
          ""   b   a   g          (j = 0..3)
    ""  [  1   0   0   0 ]        i = 0   (base row)
    b   [  1   1   0   0 ]        i = 1
    a   [  1   1   1   0 ]        i = 2
    b   [  1   2   1   0 ]        i = 3
    g   [  1   2   1   1 ]        i = 4
    b   [  1   3   1   1 ]        i = 5
    a   [  1   3   4   1 ]        i = 6
    g   [  1   3   4   5 ]        i = 7

dp[7][3] = 5 ✅
```

Spot checks:

```
dp[3][1]: s[2]='b' == t[0]='b' → dp[2][0] + dp[2][1] = 1 + 1 = 2   (b@0 or b@2)
dp[6][2]: s[5]='a' == t[1]='a' → dp[5][1] + dp[5][2] = 3 + 1 = 4
dp[7][3]: s[6]='g' == t[2]='g' → dp[6][2] + dp[6][3] = 4 + 1 = 5
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(n × m)` |

---

## 7. Approach 4: Space Optimization (Two Arrays)

Row `i` depends only on row `i-1`, so two rows are enough: `prev` (row `i-1`) and `cur` (row `i`).

### How the base cases are stored

```cpp
vector<int> prev(m + 1, 0), cur(m + 1, 0);
prev[0] = 1;
cur[0]  = 1;
```

- **`prev[0] = 1`:** `prev` starts as **row 0** of the table, `[1, 0, 0, ..., 0]`. Only the first cell is `1` (empty from empty). Every other cell of row 0 is `0` from initialization.
- **`cur[0] = 1`:** every row needs `dp[i][0] = 1` (the first column). The inner loop starts at `j = 1`, so **nothing ever overwrites `cur[0]`**. Setting it once is enough. After `prev = cur`, `prev[0]` is still `1` for the next row.

Every `cur[j]` with `j ≥ 1` is assigned in **both** branches of the `if/else`, so reusing the same `cur` vector across rows leaves no stale data.

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();

        vector<int> prev(m + 1, 0), cur(m + 1, 0);

        prev[0] = 1;                    // row 0: dp[0][0] = 1
        cur[0]  = 1;                    // column 0 stays 1 for every row

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (s[i - 1] == t[j - 1])
                    cur[j] = (long long)prev[j - 1] + prev[j];
                else
                    cur[j] = prev[j];
            }
            prev = cur;
        }

        return prev[m];
    }
};
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(2m)` = `O(m)` |

---

## 8. Approach 5: 1D Space Optimization (Single Array)

### What the recurrence needs

```
dp[i][j] needs:
    dp[i-1][j]     (same column, previous row)       ← "above"
    dp[i-1][j-1]   (previous column, previous row)   ← "diagonal"
```

It **never** needs `dp[i][j-1]` (the left neighbor in the current row).

### How we can use one array

Keep one array `prev`. Before processing row `i`, `prev[j]` holds `dp[i-1][j]`. After processing, we overwrite it with `dp[i][j]`.

- **Mismatch:** `dp[i][j] = dp[i-1][j]`, which is exactly what `prev[j]` already holds. **Nothing to do**, and that is why the code has no `else`.
- **Match:** `dp[i][j] = dp[i-1][j-1] + dp[i-1][j]`, i.e. `prev[j] = prev[j-1] + prev[j]` using the **old** values of both.

The only danger: `prev[j-1]` must still hold **last row's** value (`dp[i-1][j-1]`) when we read it.

### Why `j` must go from `m` down to `1`

#### Going right to left (`j = m → 1`) ✅

When we process column `j`, we have only updated columns **greater than** `j`. Columns `0..j-1` still contain row `i-1` values. So `prev[j-1]` is the true `dp[i-1][j-1]`.

```
Before row i:   prev = [ r0, r1, r2, r3 ]      (all from row i-1)

process j=3:  reads prev[2] (old ✅)   writes prev[3]
process j=2:  reads prev[1] (old ✅)   writes prev[2]
process j=1:  reads prev[0] (old ✅)   writes prev[1]
```

#### Going left to right (`j = 1 → m`) ❌

When we process column `j`, column `j-1` was **already overwritten** with row `i` data. So `prev[j-1]` would be `dp[i][j-1]`, not `dp[i-1][j-1]`.

```
process j=1:  reads prev[0], writes prev[1]    (now prev[1] = dp[i][1])
process j=2:  reads prev[1] ← ALREADY the NEW value ❌
```

Using `dp[i][j-1]` means the **same character `s[i-1]`** can be used for two different positions of `t` in one subsequence, which is illegal.

#### Concrete failing example: `s = "b"`, `t = "bb"` (correct answer: **0**)

```
Only one 'b' in s, but t needs two b's → impossible.

prev = [1, 0, 0]  (row 0)

LEFT → RIGHT (wrong):
  i=1, s[0]='b'
  j=1: t[0]='b' match → prev[1] = prev[0] + prev[1] = 1 + 0 = 1
  j=2: t[1]='b' match → prev[2] = prev[1] + prev[2] = 1 + 0 = 1   ← prev[1] is NEW!
  answer = prev[2] = 1   ❌   (the single 'b' was used twice)

RIGHT → LEFT (correct):
  i=1, s[0]='b'
  j=2: t[1]='b' match → prev[2] = prev[1] + prev[2] = 0 + 0 = 0   ← prev[1] still old
  j=1: t[0]='b' match → prev[1] = prev[0] + prev[1] = 1 + 0 = 1
  answer = prev[2] = 0   ✅
```

This is the same principle as the **0/1 knapsack** in-place optimization: each item (here, each character `s[i-1]`) may be used **at most once**, so we update from the right, so the values we read are still from the previous row.

### Why LCS needed a `diagonal` variable but this does not

LCS needs three neighbors, including the left one from the **current** row (`cur[j-1]`) **and** the old diagonal. Going left to right, the old diagonal gets overwritten, so LCS saves it in a temp variable.

Here we don't need the current-row left neighbor at all. We only need the old diagonal and the old above, so going right to left solves everything with no extra variable.

### Code

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int n = s.size();
        int m = t.size();

        vector<int> prev(m + 1, 0);
        prev[0] = 1;                           // empty t: always 1 way

        for (int i = 1; i <= n; i++) {
            for (int j = m; j >= 1; j--) {     // RIGHT → LEFT
                if (s[i - 1] == t[j - 1])
                    prev[j] = (long long)prev[j - 1] + prev[j];
                // mismatch: prev[j] already equals dp[i-1][j], nothing to do
            }
        }

        return prev[m];
    }
};
```

`prev[0]` is never touched (`j ≥ 1`), so the base case stays `1` throughout.

### Trace on `s = "babgbag"`, `t = "bag"`

```
start (row 0):   [1, 0, 0, 0]

i=1  'b': j=1 match  → prev[1] = 1+0 = 1                [1, 1, 0, 0]
i=2  'a': j=2 match  → prev[2] = prev[1]+prev[2] = 1    [1, 1, 1, 0]
i=3  'b': j=1 match  → prev[1] = 1+1 = 2                [1, 2, 1, 0]
i=4  'g': j=3 match  → prev[3] = prev[2]+prev[3] = 1    [1, 2, 1, 1]
i=5  'b': j=1 match  → prev[1] = 1+2 = 3                [1, 3, 1, 1]
i=6  'a': j=2 match  → prev[2] = prev[1]+prev[2] = 4    [1, 3, 4, 1]
i=7  'g': j=3 match  → prev[3] = prev[2]+prev[3] = 5    [1, 3, 4, 5]

answer = prev[3] = 5 ✅   (each row equals the matching row of the 2D table)
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(m)` (one array) |

---

## 9. Overflow Note

All versions only **add** counts. The problem guarantees the **final** answer fits in a 32-bit signed integer, but some intermediate cells can be larger. In the code above:

- `(long long)prev[j - 1] + prev[j]` makes the **addition** itself safe (no signed-overflow undefined behavior).
- The result is stored back into an `int`, so a value above `INT_MAX` wraps around.

Because only additions are involved, wrapping modulo `2³²` does not corrupt the final answer, and the final answer is guaranteed to fit. For clarity, the safest choice is `unsigned int` or `unsigned long long` for the dp values, and the same applies to the `int` returns in the recursive versions.

---

## 10. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` | Match: add both choices; mismatch: shrink `s` |
| **Memoization** | `O(n·m)` | `O(n·m) + O(n)` | Cache `(i, j)`; base cases return before the check |
| **Tabulation** | `O(n·m)` | `O(n·m)` | 1-based indexing; column 0 = 1, row 0 = 0 |
| **Two arrays** | `O(n·m)` | `O(m)` | `prev[0] = cur[0] = 1` |
| **1D array** | `O(n·m)` | `O(m)` | Iterate `j` from `m` to `1` so old values survive |

---

## 11. Where This Sits in the String DP Family

| Problem | On match | Combine |
|---|---|---|
| LCS | `1 + f(i-1, j-1)` | `max` |
| Longest Common Substring | `1 + f(i-1, j-1)`, reset on mismatch | track `max` |
| **Distinct Subsequences** | `f(i-1, j-1) + f(i-1, j)` | `+` (counting) |
| Edit Distance | `f(i-1, j-1)` (free) | `min` + 1 on edit |

Any problem that **counts** the ways to build a pattern from a source follows the same shape as this one: two choices on a match (use it or skip it), added together, with the pattern pointer moving only when a match is used.
