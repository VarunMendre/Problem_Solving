# Edit Distance

---

## 1. Problem Statement

Given two strings `word1` and `word2`, return the **minimum number of operations** needed to convert `word1` into `word2`. Three operations are allowed on a word:

- **Insert** a character
- **Delete** a character
- **Replace** a character

```
word1 = "horse", word2 = "ros"

horse → rorse   (replace 'h' with 'r')
rorse → rose    (delete 'r')
rose  → ros     (delete 'e')

Answer: 3
```

---

## 2. Intuition: Three Operations Become Three Pointer Moves

Define:

```
f(i, j) = minimum operations to convert word1[0..i] into word2[0..j]
```

Compare the **last characters** `word1[i]` and `word2[j]`.

### Case 1: characters match: `word1[i] == word2[j]`

No operation is needed for this pair. Both strings shrink for free:

```
f(i, j) = 0 + f(i-1, j-1)
```

### Case 2: characters differ: we must pay 1 and pick the best operation

| Operation | What it does | Next state | Why |
|---|---|---|---|
| **Insert** | Add `word2[j]` after `word1[i]`, so the last characters now match | `f(i, j-1)` | `word2[j]` is settled, `j` shrinks. `word1[0..i]` is still untouched, so `i` stays |
| **Delete** | Remove `word1[i]` | `f(i-1, j)` | `word1[i]` is gone, `i` shrinks. `word2[j]` is still unmatched, so `j` stays |
| **Replace** | Change `word1[i]` into `word2[j]` | `f(i-1, j-1)` | Both last characters are now settled, both shrink |

```
f(i, j) = 1 + min( f(i, j-1),      // insert
                   f(i-1, j),      // delete
                   f(i-1, j-1) )   // replace
```

### Why we don't try the other options on a match

When the characters match, taking the free diagonal `f(i-1, j-1)` is never worse than paying 1 for insert/delete/replace. Moving to an adjacent cell in the table changes the answer by at most 1, so any alternative costs at least `1 + (something ≥ f(i-1, j-1) - 1)`, which can't beat the free move. So on a match we take the diagonal and stop.

### Base cases

```cpp
if (j < 0) return i + 1;   // word2 exhausted → delete the remaining i+1 chars of word1
if (i < 0) return j + 1;   // word1 exhausted → insert the remaining j+1 chars of word2
```

- **`j < 0`:** `word2` has been fully produced. `word1[0..i]` is left over (`i + 1` characters), so we delete them all.
- **`i < 0`:** `word1` is used up, but `word2[0..j]` is still missing (`j + 1` characters), so we insert them all.
- If both are `-1`, the first check returns `i + 1 = 0` ✅.

Notice that the base cases are **not** constants (unlike Distinct Subsequences, where they were `0` and `1`). They depend on how many characters remain.

---

## 3. Approach 1: Recursion

```cpp
class Solution {
private:
    int fn(int i, int j, string& s1, string& s2) {
        if (j < 0) return i + 1;      // delete leftover of s1
        if (i < 0) return j + 1;      // insert leftover of s2

        if (s1[i] == s2[j])
            return 0 + fn(i - 1, j - 1, s1, s2);       // free match

        return 1 + min({
            fn(i, j - 1, s1, s2),       // insertion
            fn(i - 1, j, s1, s2),       // deletion
            fn(i - 1, j - 1, s1, s2)    // replacement
        });
    }

public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();
        return fn(n - 1, m - 1, word1, word2);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(3^(n+m))` | Three branches per mismatch; every call lowers `i + j` by at least 1, so depth ≤ `n + m` |
| **Space** | `O(n + m)` | Recursion depth |

---

## 4. Approach 2: Memoization (0-based)

The state is just `(i, j)`, and the same pair is reached through many different operation sequences (for example "replace then delete" and "delete then replace" both end up at `(i-2, j-1)`). Cache it.

```cpp
class Solution {
private:
    int fn(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if (j < 0) return i + 1;
        if (i < 0) return j + 1;

        if (dp[i][j] != -1)
            return dp[i][j];

        if (s1[i] == s2[j])
            return dp[i][j] = 0 + fn(i - 1, j - 1, s1, s2, dp);

        return dp[i][j] = 1 + min({
            fn(i, j - 1, s1, s2, dp),      // insertion
            fn(i - 1, j, s1, s2, dp),      // deletion
            fn(i - 1, j - 1, s1, s2, dp)   // replacement
        });
    }

public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();
        vector<vector<int>> dp(n, vector<int>(m, -1));
        return fn(n - 1, m - 1, word1, word2, dp);
    }
};
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(n × m)` + `O(n + m)` stack |

---

## 5. Approach 3: Memoization with 1-based Indexing (the bridge to tabulation)

Base cases at `i = -1` or `j = -1` can't be stored in an array. Shift both indexes by `+1` so that `i` and `j` mean **how many characters** have been used (prefix lengths):

```
i = 0 → empty prefix of word1       j = 0 → empty prefix of word2
real character at prefix length i is word1[i-1]
```

The base cases become `j == 0 → return i` and `i == 0 → return j`:

- `dp[i][0] = i` → turning `i` characters into nothing = `i` deletions
- `dp[0][j] = j` → building `j` characters from nothing = `j` insertions

```cpp
class Solution {
private:
    int fn(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if (j == 0) return i;
        if (i == 0) return j;

        if (dp[i][j] != -1)
            return dp[i][j];

        if (s1[i - 1] == s2[j - 1])
            return dp[i][j] = 0 + fn(i - 1, j - 1, s1, s2, dp);

        return dp[i][j] = 1 + min({
            fn(i, j - 1, s1, s2, dp),      // insertion
            fn(i - 1, j, s1, s2, dp),      // deletion
            fn(i - 1, j - 1, s1, s2, dp)   // replacement
        });
    }

public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, -1));
        return fn(n, m, word1, word2, dp);
    }
};
```

Now `dp[i][j]` = minimum operations to turn the first `i` characters of `word1` into the first `j` characters of `word2`, with `(n+1) × (m+1)` cells.

---

## 6. Approach 4: Tabulation

### How the base cases are stored

```cpp
for (int j = 0; j <= m; j++) dp[0][j] = j;   // first row:    j insertions
for (int i = 0; i <= n; i++) dp[i][0] = i;   // first column: i deletions
```

Both loops set `dp[0][0] = 0`, so they agree.

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

        for (int j = 0; j <= m; j++) dp[0][j] = j;
        for (int i = 0; i <= n; i++) dp[i][0] = i;

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (word1[i - 1] == word2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1];
                else
                    dp[i][j] = 1 + min({dp[i][j - 1],       // insert
                                        dp[i - 1][j],       // delete
                                        dp[i - 1][j - 1]}); // replace
            }
        }

        return dp[n][m];
    }
};
```

### Table for `word1 = "horse"`, `word2 = "ros"`

```
            ""   r   o   s
     ""   [  0   1   2   3 ]
      h   [  1   1   2   3 ]
      o   [  2   2   1   2 ]
      r   [  3   2   2   2 ]
      s   [  4   3   3   2 ]
      e   [  5   4   4   3 ]

dp[5][3] = 3 ✅
```

Spot checks:

```
dp[2][2]: 'o' == 'o'  → dp[1][1] = 1                                  (free match)
dp[3][3]: 'r' != 's'  → 1 + min(dp[3][2]=2, dp[2][3]=2, dp[2][2]=1) = 2
dp[5][3]: 'e' != 's'  → 1 + min(dp[5][2]=4, dp[4][3]=2, dp[4][2]=3) = 3
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(n × m)` |

---

## 7. Approach 5: Space Optimization (Two Arrays)

Each cell needs the previous row (`dp[i-1][*]`) and the current row's left neighbor (`dp[i][j-1]`), so two rows suffice.

### How the base cases are stored

```cpp
for (int j = 0; j <= m; j++) prev[j] = j;   // row 0:  [0, 1, 2, ..., m]
...
cur[0] = i;                                  // column 0 of row i
```

Unlike Distinct Subsequences (where column 0 was always `1`), here the first column **changes with every row** (`dp[i][0] = i`). So `cur[0] = i` must be set **inside the `i` loop**, once per row.

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();

        vector<int> prev(m + 1, 0);
        vector<int> cur(m + 1, 0);

        for (int j = 0; j <= m; j++)
            prev[j] = j;                              // row 0

        for (int i = 1; i <= n; i++) {
            cur[0] = i;                               // column 0 of this row
            for (int j = 1; j <= m; j++) {
                if (word1[i - 1] == word2[j - 1])
                    cur[j] = prev[j - 1];                           // diagonal
                else
                    cur[j] = 1 + min({cur[j - 1],                   // left   (insert)
                                      prev[j],                      // up     (delete)
                                      prev[j - 1]});                // diag   (replace)
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
| **Space** | `O(m)` (two arrays) |

---

## 8. Why a Plain 1D Optimization Doesn't Work (and What Does)

> **Short answer:** a single array with just a **loop direction trick** is impossible for Edit Distance. A single array **plus one extra variable** (the saved diagonal) does work, exactly like the LCS super space optimization.

### What each cell needs

```
dp[i][j] needs three neighbors:

   diagonal = dp[i-1][j-1]   (previous row)
   up       = dp[i-1][j]     (previous row)
   left     = dp[i][j-1]     (CURRENT row)
```

With one array `dp`, slot `j` holds `dp[i-1][j]` before we process it and `dp[i][j]` after.

Here is the conflict. To compute slot `j`, we need slot `j-1` in **two different forms at once**:

| Needed value | Form of slot `j-1` |
|---|---|
| diagonal `dp[i-1][j-1]` | **OLD** (previous row) |
| left `dp[i][j-1]` | **NEW** (current row) |

One slot cannot hold both the old and the new value.

### Try right to left (`j = m → 1`)

```
Processing slot j: slots 0..j-1 are still OLD (previous row).

  diagonal = slot j-1 (old) ✅
  up       = slot j   (old) ✅
  left     = needs dp[i][j-1], the NEW value of slot j-1 ❌
```

The new value of slot `j-1` **hasn't been computed yet** (it is computed later, and it itself depends on cells further left). There is no way to get it. So right-to-left is **impossible**.

This is the key difference from Distinct Subsequences and 0/1 Knapsack, which needed only `up` and `diagonal` and never the left neighbor. That is why a pure right-to-left loop worked for them.

### Try left to right (`j = 1 → m`)

```
Processing slot j: slots 0..j-1 are already NEW (current row).

  left     = slot j-1 (new) ✅
  up       = slot j   (old, not yet overwritten) ✅
  diagonal = needs dp[i-1][j-1], the OLD value of slot j-1 ❌  (it was overwritten)
```

Here the missing value **was** known earlier, so we can simply **save it before overwriting**. That one saved number is the `diagonal` variable.

### The fix: one array + one `diagonal` variable (left to right)

Before overwriting slot `j`, stash its old value in `temp`. It becomes the diagonal for the **next** `j`.

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();

        vector<int> dp(m + 1);
        for (int j = 0; j <= m; j++)
            dp[j] = j;                          // row 0

        for (int i = 1; i <= n; i++) {
            int diagonal = dp[0];               // dp[i-1][0]
            dp[0] = i;                          // dp[i][0]

            for (int j = 1; j <= m; j++) {
                int temp = dp[j];               // save old dp[i-1][j] (next diagonal)

                if (word1[i - 1] == word2[j - 1])
                    dp[j] = diagonal;           // dp[i-1][j-1]
                else
                    dp[j] = 1 + min({dp[j - 1],     // left:  dp[i][j-1]   (new)
                                     dp[j],         // up:    dp[i-1][j]   (old)
                                     diagonal});    // diag:  dp[i-1][j-1] (saved)

                diagonal = temp;
            }
        }

        return dp[m];
    }
};
```

### What each variable holds when computing slot `j`

```
dp[j-1]   → dp[i][j-1]     already updated this row   (left)
dp[j]     → dp[i-1][j]     not yet overwritten        (up)
diagonal  → dp[i-1][j-1]   saved before it was overwritten
```

### Trace on `horse → ros` (first two rows)

```
start (row 0):  dp = [0, 1, 2, 3]

i=1  'h':  diagonal = dp[0] = 0,  dp[0] = 1
   j=1 'r': temp=1.  h≠r → 1 + min(left=1, up=1, diag=0) = 1.   diagonal=1
   j=2 'o': temp=2.  h≠o → 1 + min(left=1, up=2, diag=1) = 2.   diagonal=2
   j=3 's': temp=3.  h≠s → 1 + min(left=2, up=3, diag=2) = 3.
   dp = [1, 1, 2, 3]       ✅ row 1

i=2  'o':  diagonal = dp[0] = 1,  dp[0] = 2
   j=1 'r': temp=1.  o≠r → 1 + min(left=2, up=1, diag=1) = 2.   diagonal=1
   j=2 'o': temp=2.  o==o → dp[2] = diagonal = 1.               diagonal=2
   j=3 's': temp=3.  o≠s → 1 + min(left=1, up=3, diag=2) = 2.
   dp = [2, 2, 1, 2]       ✅ row 2
```

Both rows match the 2D table above.

### Which problems can use which trick?

| Problem | Neighbors needed | Pure direction trick? | Needs `diagonal` variable? |
|---|---|---|---|
| 0/1 Knapsack | up, shifted-up | ✅ right to left | No |
| Distinct Subsequences | up, diagonal | ✅ right to left | No |
| **LCS** | up, left, diagonal | ❌ | ✅ |
| **Edit Distance** | up, left, diagonal | ❌ | ✅ |

The rule: **if a cell needs its current-row left neighbor, a pure direction trick can't work**, because that neighbor must be new while the diagonal must be old.

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` |
| **Space** | `O(m)`: one array plus two integers |

Whether this counts as "1D" depends on how strict you are. Strictly one array and **nothing else** is not possible; one array plus the `diagonal` variable is.

---

## 9. All Approaches Compared

| Approach | Time | Space |
|---|---|---|
| **Recursion** | `O(3^(n+m))` | `O(n + m)` |
| **Memoization** | `O(n·m)` | `O(n·m) + O(n+m)` |
| **Tabulation** | `O(n·m)` | `O(n·m)` |
| **Two arrays** | `O(n·m)` | `O(m)` |
| **One array + diagonal** | `O(n·m)` | `O(m)` |

---

## 10. Edit Distance in the String DP Family

| Problem | On match | On mismatch | Combine |
|---|---|---|---|
| LCS | `1 + diag` | `max(up, left)` | `max` |
| Longest Common Substring | `1 + diag` | `0` | track `max` |
| Distinct Subsequences | `diag + up` | `up` | `+` |
| **Edit Distance** | `diag` (free) | `1 + min(left, up, diag)` | `min` |

**Link to LCS:** if only insert and delete were allowed (no replace), the answer would be `n + m - 2 × LCS`. Allowing replace lets two mismatching characters be fixed with one operation instead of two.
