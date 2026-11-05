# Wildcard Matching

---

## 1. Problem Statement

Given an input string `s` and a pattern `p`, return `true` if the pattern matches the **entire** string.

- `?` matches **any single character**
- `*` matches **any sequence of characters** (including the empty sequence)

```
s = "aa",    p = "a"      → false   (pattern too short; must cover the whole string)
s = "aa",    p = "*"      → true    ('*' matches "aa")
s = "cb",    p = "?a"     → false   ('?' matches 'c', but 'a' ≠ 'b')
s = "adceb", p = "*a*b"   → true    (* = "", a, * = "dce", b)
```

---

## 2. Intuition: Two Pointers From the End

```
f(i, j) = can s[0..i] be matched by p[0..j] ?
```

We compare the last characters `s[i]` and `p[j]`. There are exactly three situations.

### Case 1: `s[i] == p[j]` or `p[j] == '?'` (a single-character match)

The two last characters are consumed together:

```
f(i, j) = f(i-1, j-1)
```

### Case 2: `p[j] == '*'` (two choices)

A star is flexible, so we try both ways and take the **OR**:

```
f(i, j) = f(i, j-1)  ||  f(i-1, j)
          ↑ star matches EMPTY    ↑ star ABSORBS s[i]
```

| Choice | What it means | Pointer move |
|---|---|---|
| `f(i, j-1)` | The star matches the **empty sequence**. We are done with this star, `s[i]` stays to be matched by whatever comes before the star in `p` | only `j` moves |
| `f(i-1, j)` | The star **absorbs `s[i]`**. The star is **not** consumed yet, because it may also absorb more characters to the left | only `i` moves |

The star matching exactly one character is covered automatically: absorb it (`f(i-1, j)`), then let the star match empty (`f(i-1, j-1)`).

### Case 3: anything else (a plain letter that differs)

```
return false;
```

### Base cases

**`j < 0` (pattern exhausted):**

```cpp
if (j < 0) {
    if (i < 0) return true;    // both finished together → perfect match
    else       return false;   // pattern over, but string has leftover characters
}
```

**`i < 0` (string exhausted, pattern still has `p[0..j]` left):**

```cpp
// The leftover pattern can only match the EMPTY string if every character is '*'
for (int seq = 0; seq <= j; seq++)
    if (p[seq] != '*') return false;
return true;
```

Each star can match the empty sequence, but a `?` or a letter needs a real character, and there are none left.

The check `j < 0` comes **first**, so the state `(i<0, j<0)` correctly returns `true`.

---

## 3. Approach 1: Recursion

```cpp
class Solution {
private:
    bool fn(int i, int j, string& s1, string& s2) {
        if (j < 0) {
            if (i < 0) return true;
            else       return false;
        }

        if (i < 0) {
            int flag = 0;
            for (int seq = 0; seq <= j; seq++) {
                if (s2[seq] != '*')
                    flag = 1;
            }
            return flag == 1 ? false : true;
        }

        if (s1[i] == s2[j] || s2[j] == '?')
            return fn(i - 1, j - 1, s1, s2);

        if (s2[j] == '*')
            return fn(i, j - 1, s1, s2) || fn(i - 1, j, s1, s2);

        return false;
    }

public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();
        return fn(n - 1, m - 1, s, p);
    }
};
```

### Dry run: `s = "ab"`, `p = "*b"`

```
fn(1,1): s[1]='b' == p[1]='b'  → fn(0,0)
  fn(0,0): p[0]='*'  → fn(0,-1) || fn(-1,0)
     fn(0,-1):  j<0 and i=0 ≥ 0 → false
     fn(-1,0):  i<0, check p[0..0] = "*" all stars → true
  = false || true = true
→ true ✅   ('*' absorbed "a", then 'b' matched 'b')
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2^(n+m))` | Branching only on `*`; each call lowers `i + j` by at least 1 |
| **Space** | `O(n + m)` | Recursion depth |

---

## 4. Approach 2: Memoization (1-based indexing)

### Why memoize?

Different routes reach the same `(i, j)`. With stars, "absorb then empty" and "empty then absorb" lead to identical states, so the same subproblem is recomputed again and again.

### Why shift to 1-based indexing?

Same reason as in the earlier string DP problems: the base cases at `i = -1` and `j = -1` can't be stored in an array. After shifting by `+1`, `i` and `j` mean **prefix lengths**:

```
i = 0 → empty prefix of s          real character at prefix length i is s[i-1]
j = 0 → empty prefix of p          real character at prefix length j is p[j-1]
```

| Old | New |
|---|---|
| `j < 0` | `j == 0` |
| `i < 0` | `i == 0` |
| `s[i]`, `p[j]` | `s[i-1]`, `p[j-1]` |
| loop `seq = 0..j` over `p[seq]` | loop `seq = 1..j` over `p[seq-1]` |
| call `fn(n-1, m-1)` | call `fn(n, m)` |
| table `n × m` | table `(n+1) × (m+1)` |

```cpp
class Solution {
private:
    bool fn(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if (j == 0) {
            if (i == 0) return true;
            else        return false;
        }

        if (i == 0) {
            int flag = 0;
            for (int seq = 1; seq <= j; seq++) {
                if (s2[seq - 1] != '*')
                    flag = 1;
            }
            return flag == 1 ? false : true;
        }

        if (dp[i][j] != -1)
            return dp[i][j];

        if (s1[i - 1] == s2[j - 1] || s2[j - 1] == '?')
            return dp[i][j] = fn(i - 1, j - 1, s1, s2, dp);

        if (s2[j - 1] == '*')
            return dp[i][j] = fn(i, j - 1, s1, s2, dp) || fn(i - 1, j, s1, s2, dp);

        return dp[i][j] = false;
    }

public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, -1));
        return fn(n, m, s, p, dp);
    }
};
```

`dp[i][j]` = "can the first `i` characters of `s` be matched by the first `j` characters of `p`?"

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` states | Each state does `O(1)` work |
| **Space** | `O(n × m) + O(n + m)` | dp table + recursion stack |

> **A small hidden cost:** the `i == 0` base case runs a loop of up to `j` steps and is **not cached** (it returns before the `dp` check). In the worst case this adds up to `O(m²)` extra work. The tabulation below removes it by computing the same information once for every `j` (see the next section).

---

## 5. Approach 3: Tabulation: The Initialization Explained

```
dp[i][j] = can the first i chars of s be matched by the first j chars of p ?
```

The two base cases of the recursion become the **first column** and the **first row** of the table.

### Initialization loop 1: the first column (`j = 0`, empty pattern)

```cpp
for (int i = 0; i <= n; i++) {
    if (i == 0) dp[i][0] = true;
    else        dp[i][0] = false;
}
dp[0][0] = 1;
```

An **empty pattern** matches only an **empty string**:

- `dp[0][0] = true` (empty string vs empty pattern: match)
- `dp[i][0] = false` for `i ≥ 1` (a non-empty string can never be matched by an empty pattern)

This is the `j == 0` base case of the recursion. (Because `dp` is initialized with `0`, the `false` entries are already there. The explicit `dp[0][0] = 1` after the loop is redundant, since the loop's `i == 0` branch already sets it, but it is harmless.)

### Initialization loop 2: the first row (`i = 0`, empty string)

```cpp
for (int j = 1; j <= m; j++) {
    if (p[j - 1] == '*')
        dp[0][j] = dp[0][j - 1];
    else
        dp[0][j] = false;
}
```

An **empty string** can only be matched by a pattern made **entirely of stars** (each star matches the empty sequence).

The recursion checked this with a `for seq` scan that costs `O(j)` every time. The table does it in `O(1)` per cell by **reusing the previous cell**:

```
dp[0][j] = (p[j-1] == '*')  AND  dp[0][j-1]
```

- If `p[j-1]` is a star, the prefix of length `j` is all-stars **iff** the prefix of length `j-1` was all-stars → copy `dp[0][j-1]`.
- If `p[j-1]` is anything else, the prefix contains a non-star → `false`. And every later cell stays `false`, because `dp[0][j-1]` is `false` from then on.

```
p = "**a*"

j=0: dp[0][0] = true
j=1: p[0]='*'  → dp[0][1] = dp[0][0] = true
j=2: p[1]='*'  → dp[0][2] = dp[0][1] = true
j=3: p[2]='a'  → dp[0][3] = false          ← first non-star kills it
j=4: p[3]='*'  → dp[0][4] = dp[0][3] = false   ← stays false forever
```

### The main loops

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= m; j++) {

        if (s[i - 1] == p[j - 1] || p[j - 1] == '?')
            dp[i][j] = dp[i - 1][j - 1];              // single-char match → diagonal

        else if (p[j - 1] == '*')
            dp[i][j] = dp[i][j - 1] || dp[i - 1][j];  // left (empty) OR up (absorb)

        // else: letters differ → stays 0 (false) from the initialization
    }
}
```

How the star case reads on the table:

```
              j-1      j
     i-1   [   ?   ][ up ]       up   = dp[i-1][j]   star absorbs s[i-1]
      i    [ left ][ ME ]        left = dp[i][j-1]   star matches empty
```

### Complete code

```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

        // first column: empty pattern matches only empty string
        for (int i = 0; i <= n; i++) {
            if (i == 0) dp[i][0] = true;
            else        dp[i][0] = false;
        }
        dp[0][0] = 1;

        // first row: empty string matches only an all-star pattern prefix
        for (int j = 1; j <= m; j++) {
            if (p[j - 1] == '*') dp[0][j] = dp[0][j - 1];
            else                 dp[0][j] = false;
        }

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (s[i - 1] == p[j - 1] || p[j - 1] == '?')
                    dp[i][j] = dp[i - 1][j - 1];
                else if (p[j - 1] == '*')
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
            }
        }

        return dp[n][m];
    }
};
```

### Full table: `s = "adceb"`, `p = "*a*b"` (answer `true`)

```
           ""   *   a   *   b          (j = 0..4)
     ""  [  T   T   F   F   F ]        i = 0   (first row: only leading stars)
     a   [  F   T   T   T   F ]        i = 1
     d   [  F   T   F   T   F ]        i = 2
     c   [  F   T   F   T   F ]        i = 3
     e   [  F   T   F   T   F ]        i = 4
     b   [  F   T   F   T   T ]        i = 5

dp[5][4] = T ✅
```

Spot checks:

```
dp[1][1]: p[0]='*' → dp[1][0] || dp[0][1] = F || T = T       (star absorbs 'a', or matches empty)
dp[1][2]: 'a' == 'a' → dp[0][1] = T                          (first star matched empty, 'a' matched 'a')
dp[3][3]: p[2]='*' → dp[3][2] || dp[2][3] = F || T = T       (second star absorbed 'c' on top of "d")
dp[5][4]: 'b' == 'b' → dp[4][3] = T                          (final 'b' matches, second star took "dce")
```

### A failing example: `s = "cb"`, `p = "?a"` (answer `false`)

```
           ""   ?   a
     ""  [  T   F   F ]       first row: '?' is not a star → false from j=1 on
     c   [  F   T   F ]       '?' matches 'c' → dp[0][0]=T; 'a' ≠ 'c' → F
     b   [  F   F   F ]       '?' needs dp[1][0]=F → F; 'a' ≠ 'b' → F

dp[2][2] = F ✅
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` (the first-row init is `O(m)`, no hidden scans) |
| **Space** | `O(n × m)` |

---

## 6. Approach 4: Space Optimization (Two Arrays)

Each cell reads only the previous row (`dp[i-1][*]`) and the current row's left neighbor (`dp[i][j-1]`), so two rows are enough.

### How the base cases are stored

```cpp
prev[0] = 1;                                   // row 0, column 0: empty vs empty
for (int j = 1; j <= m; j++)                   // rest of row 0: leading stars only
    prev[j] = (p[j - 1] == '*') ? prev[j - 1] : false;
...
cur[0] = false;                                // column 0 of every row i ≥ 1
```

- **`prev` starts as row 0** of the table.
- **`cur[0] = false`** must be set at the start of every row, because `dp[i][0]` is `false` for all `i ≥ 1`. The inner loop starts at `j = 1`, so nothing else would reset it.

### Why the explicit `else cur[j] = false`

In the 2D version the default `0` quietly covers the "letters differ" case. Here `cur` is **reused for every row**, so if we don't write `false` explicitly, `cur[j]` would keep the value left over from the previous row.

```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.size();
        int m = p.size();

        vector<int> prev(m + 1, 0);
        vector<int> cur(m + 1, 0);

        prev[0] = 1;
        for (int j = 1; j <= m; j++) {
            if (p[j - 1] == '*') prev[j] = prev[j - 1];
            else                 prev[j] = false;
        }

        for (int i = 1; i <= n; i++) {
            cur[0] = false;
            for (int j = 1; j <= m; j++) {
                if (s[i - 1] == p[j - 1] || p[j - 1] == '?')
                    cur[j] = prev[j - 1];                // diagonal
                else if (p[j - 1] == '*')
                    cur[j] = cur[j - 1] || prev[j];      // left (new) OR up (old)
                else
                    cur[j] = false;                      // must overwrite stale value
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
| **Space** | `O(m)` |

> **Going to a single array:** each cell needs the diagonal, the up cell **and** the left cell of the current row, so a pure direction trick fails and a saved `diagonal` variable is needed, exactly as in LCS and Edit Distance.

---

## 7. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(2^(n+m))` | `O(n + m)` | Branch only on `*` |
| **Memoization** | `O(n·m)` | `O(n·m) + O(n+m)` | `i == 0` base loop is not cached |
| **Tabulation** | `O(n·m)` | `O(n·m)` | First column = false except `dp[0][0]`; first row = leading stars |
| **Two arrays** | `O(n·m)` | `O(m)` | `cur[0] = false` every row; explicit `else` |

---

## 8. Story Points

---

**Story Point 1: "A star has two lives: empty, or absorb one more"**

`f(i, j-1)` retires the star (it matched nothing), while `f(i-1, j)` keeps the star alive after eating `s[i]`. Together they cover every possible length of the sequence a star can match.

---

**Story Point 2: "The first row is the only place where a pattern can match an empty string"**

`dp[0][j]` is true exactly for the leading all-star prefixes of `p`. The recurrence `dp[0][j] = (p[j-1]=='*') && dp[0][j-1]` computes this for every `j` in one pass.

---

**Story Point 3: "The first column is almost entirely false"**

A non-empty string can never be matched by an empty pattern, so `dp[i][0]` is false for every `i ≥ 1`. That is why `cur[0] = false` has to be re-applied each row in the space-optimized version.

---

**Story Point 4: "A mismatching letter needs no code in the 2D table, but needs an `else` when a row is reused"**

The default `0` handles it in the full table. With a reused `cur` array, a stale value from the previous row would survive unless it is overwritten.

---

## 9. Wildcard Matching in the String DP Family

| Problem | Match | Mismatch / special | Combine |
|---|---|---|---|
| LCS | `1 + diag` | `max(up, left)` | `max` |
| Distinct Subsequences | `diag + up` | `up` | `+` |
| Edit Distance | `diag` | `1 + min(left, up, diag)` | `min` |
| **Wildcard Matching** | `diag` (`?` or equal) | `*` → `left OR up`, letter mismatch → `false` | `OR` (boolean) |
