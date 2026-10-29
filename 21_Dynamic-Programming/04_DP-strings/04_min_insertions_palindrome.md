# Minimum Insertions to Make a String Palindrome

---

## 1. Problem Statement

Given a string `s`, you can insert characters **at any position**. Return the **minimum number of insertions** needed to make `s` a palindrome.

```
s = "zzazz"   → already a palindrome → 0

s = "mbadm"   → insert 2 characters → "mbdadbm" (palindrome) → 2

s = "leetcode" → 5 insertions → "leetcodocteel" → 5
```

---

## 2. The Core Insight: `answer = n - LPS(s)`

### Step 1: What does a palindrome need?

In a palindrome, **every character has a mirror partner** on the opposite side (except possibly the middle one).

### Step 2: Which characters of `s` are already fine?

The characters that belong to the **Longest Palindromic Subsequence (LPS)** are already correctly paired with each other. They form a valid palindromic skeleton, and we never need to touch them.

### Step 3: Which characters are the problem?

Every character **not** in the LPS has no mirror partner. For each such character, we insert **one** matching copy on the opposite side.

```
Characters in LPS:      already mirrored → cost 0
Characters not in LPS:  need 1 inserted mirror each → cost 1 each

Total insertions = (characters not in LPS) = n - LPS
```

### Why can't we do better?

Suppose we used a palindromic subsequence **shorter** than the LPS as our skeleton. Then even more characters would be unpaired, requiring more insertions. The longest skeleton leaves the fewest characters unpaired, so `n - LPS` is the minimum.

---

## 3. Concrete Example: `s = "mbadm"`

```
s = m b a d m   (n = 5)

LPS candidates: "mam" (m, a, m)  → length 3
                "mbm"           → length 3
                "mdm"           → length 3

LPS = 3
Answer = 5 - 3 = 2
```

**Building the palindrome using skeleton "mam":**

```
Original:   m  b  a  d  m
Skeleton:   m  .  a  .  m      (b and d are unpaired)

Insert a mirror for b → place 'b' on the right side of 'a'
Insert a mirror for d → place 'd' on the left side of 'a'

Result:     m  b  d  a  d  b  m     → palindrome ✅ (length 7 = 5 + 2)
```

Check that the original is still a subsequence: `m(0) b(1) a(3) d(4) m(6)` ✅

---

## 4. Building on Previous Problems

```
Minimum Insertions to Make Palindrome
        ↓   answer = n - LPS
Longest Palindromic Subsequence
        ↓   LPS(s) = LCS(s, reverse(s))
Longest Common Subsequence
```

This is the **third level of a reduction chain**:

| Level | Problem | Reduces to |
|---|---|---|
| 1 | Min Insertions for Palindrome | `n - LPS` |
| 2 | LPS | `LCS(s, reverse(s))` |
| 3 | LCS | Base DP (pick/skip on two strings) |

No new DP is written here. The code reuses the LPS function (space-optimized LCS on `s` and `reverse(s)`) and does one subtraction.

---

## 5. Code

```cpp
class Solution {
private:
    // LPS = LCS(s, reverse(s)), space-optimized (two rows)
    int longestPalindromeSubseq(string s) {
        string s1 = s;
        string s2 = s;
        reverse(s2.begin(), s2.end());

        int n = s1.size();
        int m = s2.size();

        vector<int> prev(m + 1, 0);

        for (int i = 1; i <= n; i++) {
            vector<int> cur(m + 1, 0);
            for (int j = 1; j <= m; j++) {
                if (s1[i - 1] == s2[j - 1])
                    cur[j] = 1 + prev[j - 1];          // match → extend diagonal
                else
                    cur[j] = max(prev[j], cur[j - 1]); // skip from s1 or s2
            }
            prev = cur;
        }

        return prev[m];
    }

public:
    int minInsertions(string s) {
        int n = s.size();
        int lps = longestPalindromeSubseq(s);
        return n - lps;   // characters outside the LPS each need one insertion
    }
};
```

---

## 6. Dry Run: `s = "abca"`

```
s1 = "abca"
s2 = "acba"   (reverse of s)
n = m = 4
```

**DP table** (`dp[i][j]` = LCS of first `i` chars of s1 and first `j` chars of s2):

```
        ""  a  c  b  a
    ""  [0  0  0  0  0]
    a   [0  1  1  1  1]
    b   [0  1  1  2  2]
    c   [0  1  2  2  2]
    a   [0  1  2  2  3]
```

**Row by row (space-optimized view):**

```
i=1 (a): cur = [0, 1, 1, 1, 1]
  j=1: a==a → 1 + prev[0] = 1
  j=2: c≠a → max(prev[2]=0, cur[1]=1) = 1
  j=3: b≠a → max(0, 1) = 1
  j=4: a==a → 1 + prev[3] = 1 + 0 = 1

i=2 (b): cur = [0, 1, 1, 2, 2]
  j=1: a≠b → max(prev[1]=1, cur[0]=0) = 1
  j=2: c≠b → max(prev[2]=1, cur[1]=1) = 1
  j=3: b==b → 1 + prev[2] = 1 + 1 = 2
  j=4: a≠b → max(prev[4]=1, cur[3]=2) = 2

i=3 (c): cur = [0, 1, 2, 2, 2]
  j=2: c==c → 1 + prev[1] = 1 + 1 = 2
  j=3: b≠c → max(prev[3]=2, cur[2]=2) = 2
  j=4: a≠c → max(prev[4]=2, cur[3]=2) = 2

i=4 (a): cur = [0, 1, 2, 2, 3]
  j=4: a==a → 1 + prev[3] = 1 + 2 = 3
```

```
LPS = 3  ("aba" or "aca")
n = 4
Answer = 4 - 3 = 1
```

**Verification:** insert one `b` between `c` and `a` → `"abcba"` is a palindrome ✅

---

## 7. Story Points

---

**Story Point 1: "Each unpaired character costs exactly one insertion"**

Once the LPS skeleton is fixed, every character outside it needs exactly one mirror copy. Not zero (it would be unpaired), not two (one copy already completes the pair). That is why the formula is a plain subtraction.

---

**Story Point 2: "Insertions = Deletions"**

The minimum **deletions** to make `s` a palindrome is also `n - LPS`. Delete every character outside the LPS, and what remains is the LPS itself, a palindrome.

```
Min insertions to make palindrome = n - LPS
Min deletions  to make palindrome = n - LPS
```

They are always equal, because inserting a mirror for an unpaired character is the "dual" of deleting that character.

---

**Story Point 3: "The helper is reused as a black box"**

`minInsertions` does not know or care how LPS is computed. It could use memoization, tabulation, or any version. Swapping the helper changes only the complexity, not the logic. This is what a clean reduction looks like.

---

**Story Point 4: "Why the LCS with the reverse finds the pairing structure"**

Matching `s1[i-1] == s2[j-1]` where `s2` is the reverse of `s` pairs a character near the **front** of `s` with a character near the **back** of `s`. Chains of such matches form exactly the mirrored pairs of a palindromic subsequence. (Full proof in the LPS notes.)

---

## 8. Complexity Analysis

### Time Complexity: `O(n²)`

| Step | Cost |
|---|---|
| Reverse the string | `O(n)` |
| Fill DP (n rows × n columns, since m = n) | `O(n²)` |
| Subtraction | `O(1)` |

**Total: `O(n²)`**

### Space Complexity: `O(n)`

| Structure | Size |
|---|---|
| `s1`, `s2` copies | `O(n)` each |
| `prev` + `cur` | `O(n)` each |

**Total: `O(n)`** (the full 2D table is never stored)

---

## 9. The String DP Family: Updated

| Problem | Formula |
|---|---|
| LCS | `dp[n][m]` |
| Longest Common Substring | `max over all dp[i][j]` (reset to 0 on mismatch) |
| Longest Palindromic Subsequence | `LCS(s, reverse(s))` |
| **Min Insertions to Make Palindrome** | **`n - LPS(s)`** |
| Min Deletions to Make Palindrome | `n - LPS(s)` |
| Min Insertions + Deletions to convert `s1` → `s2` | `(n - LCS) + (m - LCS)` |
| Shortest Common Supersequence (length) | `n + m - LCS` |
| Edit Distance | Generalization of LCS with replace cost |
