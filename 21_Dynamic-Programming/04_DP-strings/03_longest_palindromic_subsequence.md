# Longest Palindromic Subsequence (LPS)

---

## 1. The Core Insight — LPS = LCS(s, reverse(s))

### What is a Palindromic Subsequence?

A palindromic subsequence reads the same forward and backward.

```
s = "bbabcbcab"
LPS = "babcbab" (length 7) or "bacab" (length 5)... 
Actually "bbabbb"? Let's check: b-b-a-b-c-b-c-a-b
LPS = "babcbab" reading: b,a,b,c,b,a,b — yes palindrome! length 7
```

---

### The Reduction: Why LCS(s, reverse(s)) Gives LPS

**Key observation:** A palindrome reads the same forwards and backwards. So a subsequence P of string `s` is palindromic if and only if P is also a subsequence of `reverse(s)`.

**Think about it:**
- P is a subsequence of s (P's characters appear in s in order)
- P is a palindrome means P = reverse(P)
- reverse(P) is also a subsequence of reverse(s) (reversing both preserves order)
- Since P = reverse(P), P itself is a subsequence of both s AND reverse(s)

**Therefore:** P is a palindromic subsequence of s ↔ P is a common subsequence of s and reverse(s).

The LONGEST palindromic subsequence = the LONGEST common subsequence of s and reverse(s).

---

### Concrete Example

```
s = "agbcba"

reverse(s) = "abcbga"

LCS("agbcba", "abcbga"):
  Common subsequences: "a", "b", "c", "ab", "abc", "abcba" ...
  
  "abcba" = a-b-c-b-a
  In s:          a-g-b-c-b-a → a,b,c,b,a ✅ (subsequence of s)
  In reverse(s): a-b-c-b-g-a → a,b,c,b,a ✅ (subsequence of reverse(s))
  Is "abcba" a palindrome? a-b-c-b-a → yes! ✅

LCS("agbcba", "abcbga") = 5 = LPS("agbcba") ✅
```

---

### Why This Works Intuitively

```
s         = a  g  b  c  b  a
reverse(s)= a  b  c  b  g  a

When we find a character that matches between s and reverse(s)
at some position, we're finding a character that appears
in the original string AND in the reversed string at corresponding positions.

This ensures the matched characters form a palindrome:
  Reading forward in s = reading backward in s (since reverse(s) is backward s)
```

---

## 2. Problem Statement

Given a string `s`, return the length of the longest subsequence of `s` that is a palindrome.

```
s = "bbbab"
LPS = "bbbb" (length 4) — but "bbbab" reversed = "babbb"
  LCS("bbbab", "babbb"):
    Matches: b-b-b-b → length 4 ✅

s = "cbbd"
LPS = "bb" (length 2) or "c", "b", "b", "d" ... "bb" is longest
```

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int lcs(int i, int j, string& s1, string& s2) {
        if(i == 0 || j == 0) return 0;

        if(s1[i-1] == s2[j-1])
            return 1 + lcs(i-1, j-1, s1, s2);

        return max(lcs(i-1, j, s1, s2), lcs(i, j-1, s1, s2));
    }

public:
    int longestPalindromeSubseq(string s) {
        string s1 = s;
        string s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size(), m = s2.size();
        return lcs(n, m, s1, s2);
    }
};
```

### TC Correction

**Your stated TC: `O(2^n × 2^m)`** — this is not quite right.

The recursion has two strings both of length `n` (since s2 = reverse(s1) has same length). At each call, we either match and recurse once (O(1) branches) or mismatch and recurse twice. The tree depth is `n + m = 2n` in the worst case.

**Correct TC: `O(2^(n+m))` = `O(2^(2n))` = `O(4^n)`** since n=m here.

**Correct SC: `O(n + m)` = `O(2n)` = `O(n)`** — recursion stack depth is bounded by `i + j` going from `n+m` down to 0. Not `O(n+m)` separately but the combined path length.

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(4^n)` (both strings length n) |
| **Space** | `O(2n)` = `O(n)` (stack depth) |

---

## 4. Approach 2 — Memoization

```cpp
class Solution {
private:
    int lcs(int i, int j, string& s1, string& s2, vector<vector<int>>& dp) {
        if(i == 0 || j == 0) return 0;

        if(dp[i][j] != -1) return dp[i][j];

        if(s1[i-1] == s2[j-1])
            return dp[i][j] = 1 + lcs(i-1, j-1, s1, s2, dp);

        return dp[i][j] = max(lcs(i-1, j, s1, s2, dp),
                               lcs(i, j-1, s1, s2, dp));
    }

public:
    int longestPalindromeSubseq(string s) {
        string s1 = s, s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size(), m = s2.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1, -1));
        return lcs(n, m, s1, s2, dp);
    }
};
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` = `O(n²)` (both strings length n) |
| **Space** | `O(n²) + O(n)` (dp table + stack) |

---

## 5. Approach 3 — Tabulation

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        string s1 = s, s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size(), m = s2.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1, 0));

        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(s1[i-1] == s2[j-1])
                    dp[i][j] = 1 + dp[i-1][j-1];
                else
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }

        return dp[n][m];
    }
};
```

### Tabulation Trace for s = "bbbab"

```
s1 = "bbbab"
s2 = "babbb"  (reversed)

        ""  b  a  b  b  b
    ""  [0  0  0  0  0  0]
    b   [0  1  1  1  1  1]
    b   [0  1  1  2  2  2]
    b   [0  1  1  2  3  3]
    a   [0  1  2  2  3  3]
    b   [0  1  2  3  3  4]

dp[5][5] = 4 ✅ (LPS = "bbbb")
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n × m)` = `O(n²)` |
| **Space** | `O(n²)` |

---

## 6. Approach 4 — Space Optimization (Two Arrays)

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        string s1 = s, s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size(), m = s2.size();
        vector<int> prev(m+1, 0);

        for(int i = 1; i <= n; i++) {
            vector<int> cur(m+1, 0);
            for(int j = 1; j <= m; j++) {
                if(s1[i-1] == s2[j-1])
                    cur[j] = 1 + prev[j-1];         // diagonal
                else
                    cur[j] = max(prev[j], cur[j-1]); // up, left
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
| **Time** | `O(n²)` |
| **Space** | `O(2m)` = `O(n)` (m = n here) |

---

## 7. Approach 5 — Super Space Optimization (One Array + Diagonal)

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        string s1 = s, s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size(), m = s2.size();
        vector<int> prev(m+1, 0);

        for(int i = 1; i <= n; i++) {
            int diagonal = 0;
            for(int j = 1; j <= m; j++) {
                int temp = prev[j];   // save before overwrite

                if(s1[i-1] == s2[j-1])
                    prev[j] = 1 + diagonal;
                else
                    prev[j] = max(prev[j], prev[j-1]);

                diagonal = temp;   // carry forward for next j
            }
        }

        return prev[m];
    }
};
```

Identical to LCS super space opt — same diagonal variable trick. See LCS notes for deep explanation.

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n²)` |
| **Space** | `O(n)` (one array of size n) |

---

## 8. All Approaches Compared

| Approach | Time | Space |
|---|---|---|
| **Recursion** | `O(4^n)` | `O(n)` |
| **Memoization** | `O(n²)` | `O(n²)` |
| **Tabulation** | `O(n²)` | `O(n²)` |
| **Space Opt** | `O(n²)` | `O(n)` |
| **Super Space Opt** | `O(n²)` | `O(n)` |

Since s1 and s2 are both of length `n`, all complexities use `n²` instead of `n×m`.

---

## 9. Story Points

---

**Story Point 1 — "Reversing reduces a NEW problem to a KNOWN one"**

LPS is a new problem. LCS is a solved problem. The reversal insight eliminates the need to write a fresh DP — you reuse LCS completely. This is the power of problem reduction: recognize that a new problem is a familiar one in disguise.

---

**Story Point 2 — "Palindrome = matches with its own mirror"**

A string is a palindrome if it equals its reverse. So a palindromic subsequence of `s` is a subsequence that appears in BOTH `s` (forward) AND `reverse(s)` (backward). The LCS of `s` and `reverse(s)` finds exactly the longest such overlap.

---

**Story Point 3 — "The LCS DP code is unchanged — only the inputs change"**

```cpp
// LCS: given text1, text2
return lcs(text1, text2);

// LPS: compute s2 = reverse(s), then same LCS
string s2 = s; reverse(s2);
return lcs(s, s2);
```

Zero algorithmic change. The transformation is purely in how you construct the inputs.

---

## 10. LPS in the String DP Family

| Problem | Relation |
|---|---|
| LCS | Foundation |
| Longest Common Substring | LCS + reset on mismatch |
| **Longest Palindromic Subsequence** | LCS(s, reverse(s)) |
| Minimum Insertions to Make Palindrome | n - LPS(s) |
| Shortest Common Supersequence | n + m - LCS |
| Minimum Edit Distance | LCS generalization with costs |
