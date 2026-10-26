# Longest Common Subsequence (LCS)

---

## 1. Problem Statement

Given two strings `text1` and `text2`, return the length of their **Longest Common Subsequence (LCS)**.

A **subsequence** is derived by deleting some (possibly zero) characters from the string without changing the relative order of the remaining characters.

```
text1 = "abcde", text2 = "ace"

LCS = "ace" (length 3)
  a-b-c-d-e
  a---c---e

text1 = "abc", text2 = "abc"
LCS = "abc" (length 3)

text1 = "abc", text2 = "def"
LCS = "" (length 0)
```

**Subsequence vs Substring:** A substring must be contiguous. A subsequence doesn't have to be.
```
"ace" is a subsequence of "abcde" ✅
"ace" is NOT a substring of "abcde" ❌ (not contiguous)
"bcd" IS a substring of "abcde" ✅
```

---

## 2. Intuition — The Two-Pointer Recursion

Walk both strings from the end simultaneously using indices `ind1` (into s1) and `ind2` (into s2).

**Case 1 — Characters match: `s1[ind1] == s2[ind2]`**
```
Both characters are part of the LCS.
Count 1 + LCS of remaining prefixes.
LCS("...a", "...a") = 1 + LCS("...", "...")
```

**Case 2 — Characters don't match:**
```
At least one character is NOT in the LCS.
Try removing from s1: LCS(ind1-1, ind2)
Try removing from s2: LCS(ind1, ind2-1)
Take the maximum of both.
```

Base case: either index goes below 0 → no characters left → LCS = 0.

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int f(int ind1, int ind2, string& s1, string& s2) {
        if(ind1 < 0 || ind2 < 0) return 0;

        if(s1[ind1] == s2[ind2])
            return 1 + f(ind1-1, ind2-1, s1, s2);

        return max(f(ind1-1, ind2, s1, s2),
                   f(ind1, ind2-1, s1, s2));
    }

public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        return f(n-1, m-1, text1, text2);
    }
};
```

### Recursion Tree for "ab", "ab"

```
f(1,1): s1[1]='b'==s2[1]='b' → 1+f(0,0)
  f(0,0): s1[0]='a'==s2[0]='a' → 1+f(-1,-1)
    f(-1,-1): return 0
  → 1+0=1
→ 1+1=2
```

For mismatched characters, branching occurs → overlapping subproblems.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2^(n+m))` | Two branches at every mismatch |
| **Space** | `O(n+m)` | Recursion stack depth |

---

## 4. Approach 2 — Memoization (Original, with negative index base case)

```cpp
class Solution {
private:
    int f(int ind1, int ind2, string& s1, string& s2, vector<vector<int>>& dp) {
        if(ind1 < 0 || ind2 < 0) return 0;

        if(dp[ind1][ind2] != -1)
            return dp[ind1][ind2];

        if(s1[ind1] == s2[ind2])
            return dp[ind1][ind2] = 1 + f(ind1-1, ind2-1, s1, s2, dp);

        return dp[ind1][ind2] = max(f(ind1-1, ind2, s1, s2, dp),
                                     f(ind1, ind2-1, s1, s2, dp));
    }

public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        vector<vector<int>> dp(n, vector<int>(m, -1));
        return f(n-1, m-1, text1, text2, dp);
    }
};
```

**Problem:** Base case `ind1 < 0 || ind2 < 0` cannot be cached in the dp array (no negative indices). This memoization works but the base case is handled outside the dp table.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | n×m unique states |
| **Space** | `O(n × m) + O(n+m)` | dp + stack |

---

## 5. The Index Shifting Trick — Why We Do It for Tabulation

### The Problem with Negative Indices

The recursion base case is `ind1 < 0 || ind2 < 0 → return 0`. In tabulation, we need to fill a dp table. But we can't have negative indices in an array — `dp[-1][j]` is undefined.

**The common workaround: shift all indices by +1.**

Instead of tracking "index in string 0-indexed," we track "how many characters we've considered 1-indexed":

```
Old: ind1 ∈ [0, n-1],  base: ind1 < 0
New: ind1 ∈ [1, n],    base: ind1 == 0

Mapping: new_ind1 = old_ind1 + 1
         s1[ind1-1] in new notation = s1[old_ind1] in old notation
```

### What Changes in the Code

```cpp
// OLD (0-indexed, negative base):
if(ind1 < 0 || ind2 < 0) return 0;
if(s1[ind1] == s2[ind2])  ...

// NEW (1-indexed, zero base):
if(ind1 == 0 || ind2 == 0) return 0;   ← zero is the new "before start"
if(s1[ind1-1] == s2[ind2-1]) ...       ← shift back by 1 to access actual character
```

`ind1 = 0` means "0 characters of s1 considered" → LCS = 0.
`ind1 = 3` means "3 characters of s1 considered" → `s1[0..2]` → access via `s1[ind1-1] = s1[2]`.

### The Shifted Memoization (Bridge to Tabulation)

```cpp
class Solution {
private:
    int f(int ind1, int ind2, string& s1, string& s2, vector<vector<int>>& dp) {
        if(ind1 == 0 || ind2 == 0) return 0;   // ← zero, not negative

        if(dp[ind1][ind2] != -1) return dp[ind1][ind2];

        if(s1[ind1-1] == s2[ind2-1])            // ← shifted access
            return dp[ind1][ind2] = 1 + f(ind1-1, ind2-1, s1, s2, dp);

        return dp[ind1][ind2] = max(f(ind1-1, ind2, s1, s2, dp),
                                     f(ind1, ind2-1, s1, s2, dp));
    }

public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1, -1));  // ← +1 for index 0
        return f(n, m, text1, text2, dp);                    // ← start at n,m not n-1,m-1
    }
};
```

**Why `dp(n+1, m+1)`?** Indices now go from `0` to `n` (size n+1) and `0` to `m` (size m+1). The extra row and column of 0s at `ind1=0` and `ind2=0` represent the base case — naturally initialized to 0 in tabulation.

---

## 6. Approach 3 — Tabulation

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();

        // dp[i][j] = LCS of text1[0..i-1] and text2[0..j-1]
        vector<vector<int>> dp(n+1, vector<int>(m+1, 0));
        // dp[0][j] = 0 (empty text1) and dp[i][0] = 0 (empty text2) — auto from init

        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(text1[i-1] == text2[j-1])      // actual chars at positions i-1, j-1
                    dp[i][j] = 1 + dp[i-1][j-1];
                else
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }

        return dp[n][m];
    }
};
```

### Why `text1[i-1]` and `text2[j-1]`?

`i` represents "considering i characters of text1." The last character in those `i` characters is at index `i-1` (0-indexed). Same for `j`.

### Tabulation Trace for text1="abcd", text2="abdc"

```
     ""  a  b  d  c
  "" [0  0  0  0  0]
  a  [0  1  1  1  1]
  b  [0  1  2  2  2]
  c  [0  1  2  2  3]
  d  [0  1  2  3  3]

dp[4][4] = 3 → LCS = "abd" or "abc" ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Fill every cell |
| **Space** | `O(n × m)` | 2D dp array |

---

## 7. Approach 4 — Space Optimization (Two Arrays)

`dp[i][j]` reads from `dp[i-1][j]`, `dp[i-1][j-1]`, and `dp[i][j-1]` → only the previous row needed:

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        vector<int> prev(m+1, 0);

        for(int i = 1; i <= n; i++) {
            vector<int> cur(m+1, 0);
            for(int j = 1; j <= m; j++) {
                if(text1[i-1] == text2[j-1])
                    cur[j] = 1 + prev[j-1];   // dp[i-1][j-1]
                else
                    cur[j] = max(prev[j],       // dp[i-1][j]
                                 cur[j-1]);     // dp[i][j-1]
            }
            prev = cur;
        }

        return prev[m];
    }
};
```

### Three Values Needed Per Cell

```
dp[i][j] depends on:
  dp[i-1][j-1] → prev[j-1]  (diagonal: previous row, previous column)
  dp[i-1][j]   → prev[j]    (above: previous row, same column)
  dp[i][j-1]   → cur[j-1]   (left: same row, previous column — already computed)
```

All three are safely accessible from `prev` and the current `cur` being filled left-to-right.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Same loops |
| **Space** | `O(2m)` = `O(m)` | Two arrays of size m+1 |

---

## 8. Approach 5 — Super Space Optimization (One Array + Diagonal Variable)

### Is It Necessary?

**Practically: No.** The two-array solution already runs in `O(m)` space which is optimal for this problem class. The super space optimization saves one array (`O(m)` vs `O(2m)`) — a constant factor, not an asymptotic improvement.

It's a clever trick worth knowing, but in interviews and production code, the two-array version is cleaner and preferred.

### The Problem with In-Place Update

If we try to update `prev` in-place (no `cur`):

```
When computing prev[j] (in-place), we need prev[j-1] as the DIAGONAL (old value)
But prev[j-1] was ALREADY UPDATED in this same pass!
→ We'd use the new prev[j-1] (current row) instead of the old prev[j-1] (previous row)
```

We need to save the old `prev[j-1]` BEFORE overwriting it — that's exactly what `diagonal` does.

### The Code

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        vector<int> prev(m+1, 0);

        for(int i = 1; i <= n; i++) {
            int diagonal = 0;   // holds old prev[j-1] (= dp[i-1][j-1] before update)

            for(int j = 1; j <= m; j++) {
                int temp = prev[j];   // save old prev[j] BEFORE overwriting

                if(text1[i-1] == text2[j-1])
                    prev[j] = 1 + diagonal;          // use saved diagonal
                else
                    prev[j] = max(prev[j], prev[j-1]); // prev[j]=above, prev[j-1]=left

                diagonal = temp;   // next iteration's diagonal = this iteration's "above"
            }
        }

        return prev[m];
    }
};
```

### Variable Roles — Step by Step

Before processing `j`:
- `prev[j]` = `dp[i-1][j]` (above — still old)
- `prev[j-1]` = `dp[i][j-1]` (left — ALREADY UPDATED in this row)
- `diagonal` = `dp[i-1][j-1]` (the true diagonal — saved from previous iteration)

After `int temp = prev[j]`:
- `temp` holds `dp[i-1][j]` safely before we overwrite `prev[j]`

After updating `prev[j]`:
- `prev[j]` = `dp[i][j]` (current row result)

`diagonal = temp` sets up the next iteration:
- Next iteration's `j` becomes current `j+1`
- The diagonal for `j+1` is `dp[i-1][j]` = what `temp` held ✅

### Visualization

```
Before row i starts:
  prev = [dp[i-1][0], dp[i-1][1], dp[i-1][2], ..., dp[i-1][m]]

j=1:
  diagonal=0 (= dp[i-1][0])
  temp = prev[1] (= dp[i-1][1])
  update prev[1] → dp[i][1]
  diagonal = temp = dp[i-1][1]

j=2:
  diagonal = dp[i-1][1] ← correct diagonal for j=2
  temp = prev[2] (= dp[i-1][2])
  update prev[2] → dp[i][2]
  diagonal = temp = dp[i-1][2]

j=3:
  diagonal = dp[i-1][2] ← correct ✅
  ...
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Same loops |
| **Space** | `O(m)` | One array + 2 integers |

---

## 9. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(2^(n+m))` | `O(n+m)` | Two branches on mismatch |
| **Memoization (original)** | `O(nm)` | `O(nm + n+m)` | Negative index base case |
| **Memoization (shifted)** | `O(nm)` | `O(nm + n+m)` | Zero base case, prep for tabulation |
| **Tabulation** | `O(nm)` | `O(nm)` | Standard, clean |
| **Space Opt** | `O(nm)` | `O(m)` | Two rows, preferred |
| **Super Space Opt** | `O(nm)` | `O(m)` | One row + diagonal var, clever but not needed |

---

## 10. LCS is the Gateway to Many String DP Problems

LCS is the foundation for:

| Problem | Relation to LCS |
|---|---|
| **Longest Common Substring** | Same DP, reset to 0 on mismatch (no carry) |
| **Shortest Common Supersequence** | Length = n + m - LCS |
| **Minimum Insertions/Deletions to convert s1 → s2** | Deletions = n - LCS, Insertions = m - LCS |
| **Longest Palindromic Subsequence** | LCS(s, reverse(s)) |
| **Edit Distance** | Generalization of LCS with costs |
| **Sequence Pattern Matching** | Check if LCS(s1,s2) == min(len(s1), len(s2)) |

Master LCS and these all follow naturally.
