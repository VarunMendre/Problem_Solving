# Longest Common Substring

---

## 1. Problem Statement

Given two strings `s1` and `s2`, return the length of their **Longest Common Substring**.

A **substring** is a contiguous sequence of characters (unlike a subsequence which can skip characters).

```
s1 = "abcde", s2 = "abfce"

Common substrings: "ab" (length 2), "c" (length 1), "e" (length 1)
Longest: "ab" → length 2

s1 = "GeeksforGeeks", s2 = "GeeksQuiz"
Common: "Geeks" → length 5
```

---

## 2. LCS vs Longest Common Substring — The Critical Difference

| | LCS (Subsequence) | Longest Common Substring |
|---|---|---|
| **Must be contiguous?** | No (can skip) | Yes |
| **On mismatch** | Take max of two branches | RESET to 0 |
| **Track answer** | `dp[n][m]` (final cell) | `max(dp[i][j])` (any cell) |

The DP table structure is **identical**. The only difference is what happens when characters DON'T match.

---

## 3. The Three Cases — In Depth

### Case 1: Characters Match (`s1[i-1] == s2[j-1]`)

```cpp
dp[i][j] = 1 + dp[i-1][j-1];
```

**What `dp[i][j]` means:** the length of the common substring that ENDS at `s1[i-1]` and `s2[j-1]`.

When `s1[i-1] == s2[j-1]`:
- These two characters extend any common substring that ended at `s1[i-2]` and `s2[j-2]`
- `dp[i-1][j-1]` = length of common substring ending at the PREVIOUS characters (the diagonal)
- So `dp[i][j] = 1 + dp[i-1][j-1]` = extend that common substring by 1

```
s1 = "abcd"
s2 = "abcd"

i=1,j=1: s1[0]='a'==s2[0]='a' → dp[1][1] = 1 + dp[0][0] = 1+0 = 1
          (substring "a" of length 1 ending here)

i=2,j=2: s1[1]='b'==s2[1]='b' → dp[2][2] = 1 + dp[1][1] = 1+1 = 2
          (substring "ab" of length 2 ending here — "a" was already matched at dp[1][1])

i=3,j=3: s1[2]='c'==s2[2]='c' → dp[3][3] = 1 + dp[2][2] = 1+2 = 3
          (substring "abc" of length 3 — building on "ab" at dp[2][2])
```

**Why only the diagonal (`dp[i-1][j-1]`) and not left/up?**

Because substrings must be CONTIGUOUS. The diagonal `dp[i-1][j-1]` means: "the common substring ending at the character IMMEDIATELY BEFORE both current characters." That's the only position that can be directly extended when both current characters match.

Using `dp[i-1][j]` (up) or `dp[i][j-1]` (left) would allow gaps — making it a subsequence, not a substring.

---

### Case 2: Characters Don't Match

```cpp
dp[i][j] = 0;
```

**Why 0 and not `max(dp[i-1][j], dp[i][j-1])`?**

This is the KEY difference from LCS.

In LCS, when characters don't match, we take `max(dp[i-1][j], dp[i][j-1])` — we can SKIP one character and carry forward the best LCS seen so far.

In Longest Common Substring, skipping is NOT allowed. A substring must be contiguous. If `s1[i-1] != s2[j-1]`, then there is NO common substring ending at BOTH `s1[i-1]` and `s2[j-1]`. The substring "chain" is BROKEN → reset to 0.

```
s1 = "abXcd"
s2 = "abYcd"

dp fills fine until X vs Y:
  i=3,j=3: 'X' != 'Y' → dp[3][3] = 0  ← RESET, chain broken!
  i=4,j=4: 'c'=='c' → dp[4][4] = 1+dp[3][3] = 1+0 = 1 ← starts fresh

LCS would carry 2 from "ab" forward here. Substring resets to 0.
```

---

### Case 3: Track the Maximum Separately

```cpp
ans = max(ans, dp[i][j]);
```

In LCS, `dp[n][m]` is always the answer — the LCS considering ALL characters of both strings.

In Longest Common Substring, the longest substring might END anywhere in the middle of either string. `dp[i][j]` = length of the LONGEST substring ending at `s1[i-1]` and `s2[j-1]`. We track the global maximum across ALL cells.

```
s1 = "abcdef", s2 = "bcde"

dp table:
      ""  b  c  d  e
  ""  [0  0  0  0  0]
  a   [0  0  0  0  0]
  b   [0  1  0  0  0]
  c   [0  0  2  0  0]
  d   [0  0  0  3  0]
  e   [0  0  0  0  4]   ← "bcde" ends here
  f   [0  0  0  0  0]

dp[5][4]=4 → "bcde", length 4

Final dp[n][m] = dp[6][4] = 0 ← WRONG if we returned this!
max across all cells = 4 ✅
```

---

## 4. Approach 1 — Tabulation

```cpp
class Solution {
public:
    int longCommSubstr(string& s1, string& s2) {
        int n = s1.size(), m = s2.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1, 0));

        int ans = 0;

        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                if(s1[i-1] == s2[j-1]) {
                    dp[i][j] = 1 + dp[i-1][j-1];   // extend diagonal
                    ans = max(ans, dp[i][j]);          // track maximum
                }
                else
                    dp[i][j] = 0;   // mismatch → reset (substring broken)
            }
        }

        return ans;
    }
};
```

### Full Table Trace for s1="abcde", s2="abfce"

```
     ""  a  b  f  c  e
  "" [0  0  0  0  0  0]
  a  [0  1  0  0  0  0]    a==a → 1+dp[0][0]=1
  b  [0  0  2  0  0  0]    b==b → 1+dp[1][1]=2 ("ab")
  c  [0  0  0  0  1  0]    c!=f,c!=b; c==c → 1+dp[2][3]=1 ("c")
  d  [0  0  0  0  0  0]    no match
  e  [0  0  0  0  0  1]    e==e → 1+dp[4][4]=1

ans tracks: max(1,2,1,1) = 2 ✅ ("ab")
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Fill every cell |
| **Space** | `O(n × m)` | 2D dp array |

---

## 5. Approach 2 — Space Optimization (Two Arrays)

```cpp
class Solution {
public:
    int longCommSubstr(string& s1, string& s2) {
        int n = s1.size(), m = s2.size();
        vector<int> prev(m+1, 0);
        int ans = 0;

        for(int i = 1; i <= n; i++) {
            vector<int> cur(m+1, 0);
            for(int j = 1; j <= m; j++) {
                if(s1[i-1] == s2[j-1]) {
                    cur[j] = 1 + prev[j-1];   // prev[j-1] = dp[i-1][j-1] (diagonal)
                    ans = max(ans, cur[j]);
                }
                else
                    cur[j] = 0;
            }
            prev = cur;
        }

        return ans;
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Same |
| **Space** | `O(2m)` = `O(m)` | Two arrays |

---

## 6. Approach 3 — Super Space Optimization (One Array + Diagonal)

```cpp
class Solution {
public:
    int longCommSubstr(string& s1, string& s2) {
        int n = s1.size(), m = s2.size();
        vector<int> prev(m+1, 0);
        int ans = 0;

        for(int i = 1; i <= n; i++) {
            int diagonal = 0;   // holds old prev[j-1] = dp[i-1][j-1]

            for(int j = 1; j <= m; j++) {
                int temp = prev[j];   // save before overwrite (needed as next diagonal)

                if(s1[i-1] == s2[j-1]) {
                    prev[j] = 1 + diagonal;
                    ans = max(ans, prev[j]);
                }
                else
                    prev[j] = 0;   // reset

                diagonal = temp;   // next j's diagonal = this j's "above"
            }
        }

        return ans;
    }
};
```

### Why Only One Diagonal Variable (Unlike LCS Which Needs the Same)?

LCS needs three values: diagonal, up, left.
- `diagonal` = `dp[i-1][j-1]` → saved in variable
- `up` = `dp[i-1][j]` → `prev[j]` (not yet overwritten at start of inner j)
- `left` = `dp[i][j-1]` → `prev[j-1]` (already updated in this pass)

Substring only needs ONE value: `diagonal` = `dp[i-1][j-1]`.
- On match: `prev[j] = 1 + diagonal`
- On mismatch: `prev[j] = 0`

No up or left needed at all — no max to compute on mismatch. This makes the diagonal management simpler.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × m)` | Same |
| **Space** | `O(m)` | One array + 2 integers |

---

## 7. All Approaches Compared

| Approach | Time | Space |
|---|---|---|
| **Tabulation** | `O(n × m)` | `O(n × m)` |
| **Space Opt** | `O(n × m)` | `O(m)` |
| **Super Space Opt** | `O(n × m)` | `O(m)` |

No recursion/memoization shown here because the "reset to 0 on mismatch" behavior makes the problem straightforward as a tabulation — there's no benefit to top-down memoization since the recurrence is trivially a table fill.

---

## 8. LCS vs Longest Common Substring — Final Comparison

| | LCS (Subsequence) | Longest Common Substring |
|---|---|---|
| **Contiguous?** | No | Yes |
| **Match** | `dp[i][j] = 1 + dp[i-1][j-1]` | `dp[i][j] = 1 + dp[i-1][j-1]` |
| **Mismatch** | `dp[i][j] = max(dp[i-1][j], dp[i][j-1])` | `dp[i][j] = 0` |
| **Answer** | `dp[n][m]` (final cell) | `max over all dp[i][j]` |
| **"Memory"** | Carries forward best seen | Resets on every break |
| **Diagonal needed?** | Yes | Yes (same) |
| **Up/left needed?** | Yes (on mismatch) | No (always 0) |
