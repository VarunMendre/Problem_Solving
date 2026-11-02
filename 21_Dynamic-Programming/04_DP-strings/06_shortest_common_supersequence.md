# Shortest Common Supersequence (SCS)

---

## 1. Problem Statement

Given two strings `str1` and `str2`, return the **shortest string** that has both `str1` and `str2` as **subsequences**. If multiple answers exist, return any one.

```
str1 = "geek", str2 = "eke"
SCS  = "geeke" (length 5)

  geeke
  g e e k .   ← str1 = "geek" is a subsequence
  . e . k e   ← str2 = "eke"  is a subsequence

str1 = "abac", str2 = "cab"
SCS  = "cabac" (length 5)
```

---

## 2. Part 1: Finding Only the Length

### Thinking about it

If the two strings shared **nothing**, the shortest string containing both would be `str1 + str2`, with length `n + m`.

But if they share characters, we don't need to write the shared ones twice. The best we can share is the **LCS**: those characters are written **once** and serve both strings.

```
Length of SCS = n + m - LCS
```

### The breakdown (where your formula fits)

Split the characters of the supersequence into three groups:

```
(n - LCS)   characters only in str1   (not part of the LCS)
(m - LCS)   characters only in str2   (not part of the LCS)
   LCS      characters shared, written ONCE
```

```
SCS length = (n - LCS) + (m - LCS) + LCS
           = n + m - LCS
```

> **Careful:** `(n - LCS) + (m - LCS)` alone is **not** the SCS length. It counts only the non-shared characters. That expression is the number of insertions + deletions needed to convert `str1` into `str2`. For SCS you must add the shared `LCS` characters back once.

### Example

```
str1 = "geek"  (n = 4)
str2 = "eke"   (m = 3)
LCS  = "ek"    (length 2)

(n - LCS) = 2   → 'g', 'e'  (from str1)
(m - LCS) = 1   → 'e'       (from str2)
   LCS    = 2   → 'e', 'k'  (shared)

SCS length = 2 + 1 + 2 = 5 = 4 + 3 - 2 ✅
```

---

## 3. Part 2: Building the Actual String

The length only needs `dp[n][m]`. To **construct** the string, we walk the DP table **backwards** from `dp[n][m]` to `dp[0][0]` and decide, at each cell, which character to emit.

### Step 1: Build the LCS table

`dp[i][j]` = LCS length of `str1[0..i-1]` and `str2[0..j-1]`.

```cpp
if (s1[i-1] == s2[j-1])
    dp[i][j] = 1 + dp[i-1][j-1];
else
    dp[i][j] = max(dp[i][j-1], dp[i-1][j]);
```

This table remembers **where each LCS value came from**. We now replay that history in reverse.

---

## 4. The Main `while (i > 0 && j > 0)` Loop, Explained

We start at `i = n, j = m` (bottom-right) and move toward the top-left. At every cell there are exactly three situations.

### Case A: `str1[i-1] == str2[j-1]` (characters match)

```cpp
result += str1[i - 1];
i--;
j--;
```

**What it means:** this character is part of the LCS. Both strings need it, and one copy serves both.

- Emit the character **once**
- Move **both** pointers back

This is the "shared" character, written once.

---

### Case B: mismatch, and `dp[i][j-1] < dp[i-1][j]` (shrink `str1`)

```cpp
result += str1[i - 1];
i--;
```

**Why compare these two cells?**

On a mismatch, the LCS recurrence was:

```
dp[i][j] = max( dp[i-1][j],     dp[i][j-1] )
                ↑ drop str1[i-1]   ↑ drop str2[j-1]
```

So `dp[i][j]` **came from** whichever neighbor is larger. We follow that same neighbor backward.

- `dp[i-1][j]` is larger → the best LCS **does not use** `str1[i-1]` (dropping it loses nothing).
- That means `str1[i-1]` is **not shared** with `str2`. It exists only in `str1`.

**Why we add it to the result:** the supersequence must still contain `str1` as a subsequence. A character that is not part of the LCS has no partner in `str2`, so it must be written explicitly. If we just moved `i--` without emitting it, that character would be lost from the answer.

> **Rule:** when a pointer shrinks (a character is dropped from the LCS path), the shrunk string's character is emitted into the result.

---

### Case C: mismatch, and `dp[i][j-1] >= dp[i-1][j]` (shrink `str2`)

```cpp
result += str2[j - 1];
j--;
```

Mirror image of Case B:

- `dp[i][j-1]` is larger (or equal) → the best LCS does not use `str2[j-1]`
- `str2[j-1]` exists only in `str2`, so emit it and move `j--`

### What about ties (`dp[i][j-1] == dp[i-1][j]`)?

Both neighbors give the same LCS length, so **either** choice leads to a valid shortest supersequence. The code's `else` picks Case C on ties. Different tie-breaking produces different, equally short answers. That is why the problem says "return any one."

---

## 5. Recovering the Exhausted Pointer

The main loop stops when `i == 0` **or** `j == 0`. At that moment one string is fully consumed, but the other may still have characters left.

```cpp
while (i > 0) { result += str1[i - 1]; i--; }
while (j > 0) { result += str2[j - 1]; j--; }
```

**Why we must append them:**

- Say `j` reached 0 first. `str2` is fully handled, but `str1[0..i-1]` still remains.
- Those leftover characters have **no partner** in `str2` (nothing left to match against).
- They are not part of the LCS, so by the rule above they must be emitted explicitly.

Only one of the two loops actually runs (the pointer that is still positive). If both are 0, neither runs.

This also handles the edge case where one string is empty: the main loop never runs, and the leftover loop copies the entire other string. The SCS of `""` and `"abc"` is `"abc"` ✅

---

## 6. Why We Reverse at the End

We walk the table from **bottom-right to top-left**, so we discover the characters of the answer **from the last to the first**.

```
Discovery order:  last char ... first char
Result string:    built in this reversed order
```

So after the loops, `result` is the answer **backwards**. One `reverse(result.begin(), result.end())` fixes it.

**Why not insert at the front?** `result = ch + result` costs `O(length)` per insertion, giving `O(L²)` overall. Appending with `+=` is amortized `O(1)`, followed by a single `O(L)` reverse.

---

## 7. Full Dry Run: `str1 = "geek"`, `str2 = "eke"`

### Step 1: LCS table

```
str1 = g e e k  (n=4)
str2 = e k e    (m=3)

          ""  e  k  e
     ""  [ 0  0  0  0 ]
     g   [ 0  0  0  0 ]
     e   [ 0  1  1  1 ]
     e   [ 0  1  1  2 ]
     k   [ 0  1  2  2 ]

LCS = dp[4][3] = 2   →   SCS length = 4 + 3 - 2 = 5
```

### Step 2: Backtrack from `(i=4, j=3)`

```
(4,3): str1[3]='k', str2[2]='e'  → mismatch
       dp[4][2]=2, dp[3][3]=2    → tie, not (2 < 2) → else: shrink str2
       emit str2[2]='e', j=2                         result = "e"

(4,2): str1[3]='k', str2[1]='k'  → MATCH
       emit 'k', i=3, j=1                            result = "ek"

(3,1): str1[2]='e', str2[0]='e'  → MATCH
       emit 'e', i=2, j=0                            result = "eke"

j == 0 → main loop ends.
```

### Step 3: Recover the exhausted pointer

```
i = 2 remaining:
  emit str1[1]='e', i=1                              result = "ekee"
  emit str1[0]='g', i=0                              result = "ekeeg"
```

### Step 4: Reverse

```
"ekeeg"  →  "geeke"
```

### Verification

```
"geeke" contains "geek" as a subsequence?  g(0) e(1) e(2) k(3)        ✅
"geeke" contains "eke"  as a subsequence?  e(1) k(3) e(4)             ✅
Length = 5 = n + m - LCS                                              ✅
```

---

## 8. Second Dry Run: the "shrink `str1`" branch

`str1 = "abac"`, `str2 = "cab"` (n=4, m=3)

```
          ""  c  a  b
     ""  [ 0  0  0  0 ]
     a   [ 0  0  1  1 ]
     b   [ 0  0  1  2 ]
     a   [ 0  0  1  2 ]
     c   [ 0  1  1  2 ]

LCS = 2, SCS length = 4 + 3 - 2 = 5
```

```
(4,3): 'c' vs 'b' mismatch. dp[4][2]=1 < dp[3][3]=2 → shrink str1
       emit str1[3]='c', i=3                         result = "c"

(3,3): 'a' vs 'b' mismatch. dp[3][2]=1 < dp[2][3]=2 → shrink str1
       emit str1[2]='a', i=2                         result = "ca"

(2,3): 'b' vs 'b' MATCH → emit 'b', i=1, j=2         result = "cab"

(1,2): 'a' vs 'a' MATCH → emit 'a', i=0, j=1         result = "caba"

i == 0 → leftover: j=1 → emit str2[0]='c'            result = "cabac"

reverse → "cabac" ✅
```

---

## 9. Code

```cpp
class Solution {
private:
    // Fills dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
    void lcs(string& s1, string& s2, int n, int m, vector<vector<int>>& dp) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (s1[i-1] == s2[j-1])
                    dp[i][j] = 1 + dp[i-1][j-1];
                else
                    dp[i][j] = max(dp[i][j-1], dp[i-1][j]);
            }
        }
    }

public:
    string shortestCommonSupersequence(string str1, string str2) {
        int n = str1.size();
        int m = str2.size();

        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
        lcs(str1, str2, n, m, dp);          // Step 1: build LCS table

        string result = "";
        int i = n, j = m;

        // Step 2: walk back from dp[n][m], following where each value came from
        while (i > 0 && j > 0) {
            if (str1[i-1] == str2[j-1]) {
                result += str1[i-1];        // shared char: emit ONCE
                i--; j--;
            } else if (dp[i][j-1] < dp[i-1][j]) {
                result += str1[i-1];        // str1 shrinks: emit its char
                i--;
            } else {
                result += str2[j-1];        // str2 shrinks: emit its char
                j--;
            }
        }

        // Step 3: one string is exhausted; copy the leftover of the other
        while (i > 0) { result += str1[i-1]; i--; }
        while (j > 0) { result += str2[j-1]; j--; }

        // Step 4: we built it back-to-front, so reverse
        reverse(result.begin(), result.end());
        return result;
    }
};
```

---

## 10. Story Points

---

**Story Point 1: "Shrinking a pointer means that character is NOT in the LCS, so it must be emitted"**

Only a match (Case A) moves both pointers and emits one shared character. Every other step drops one character from the LCS path, and a dropped character has no partner, so it goes straight into the supersequence. Nothing is ever skipped silently.

---

**Story Point 2: "The comparison `dp[i][j-1]` vs `dp[i-1][j]` just replays the LCS recurrence"**

The mismatch branch of the LCS table used `max(up, left)`. Backtracking asks the same question in reverse: "which neighbor did this value come from?" We always step toward the larger neighbor, so we stay on an optimal LCS path.

---

**Story Point 3: "Every character ends up in the answer exactly once, with shared ones counted once"**

```
characters emitted = (matches) + (shrinks of str1) + (shrinks of str2) + (leftovers)
                   = LCS + (n - LCS) + (m - LCS)
                   = n + m - LCS
```

This confirms the length formula from Part 1.

---

**Story Point 4: "The full 2D table is required for reconstruction"**

For LCS **length** we could compress to one or two rows. For SCS **construction** we must look at `dp[i-1][j]`, `dp[i][j-1]` and arbitrary earlier rows while backtracking, so the whole table must be kept.

---

## 11. Complexity Analysis

### Time Complexity: `O(n × m)`

| Step | Cost |
|---|---|
| Build LCS table | `O(n × m)` |
| Backtrack (pointers only decrease) | `O(n + m)` |
| Reverse result | `O(n + m)` |

**Total: `O(n × m)`**

### Space Complexity: `O(n × m)`

| Structure | Size |
|---|---|
| `dp` table | `O(n × m)` |
| `result` string | `O(n + m)` |

**Total: `O(n × m)`**

> If you only need the **length**, use the space-optimized LCS and return `n + m - LCS` in `O(m)` space. The reconstruction is what forces the full table.

---

## 12. The String DP Family: Updated

| Problem | Formula / Idea |
|---|---|
| LCS | `dp[n][m]` |
| Longest Common Substring | `max dp[i][j]`, reset to 0 on mismatch |
| Longest Palindromic Subsequence | `LCS(s, reverse(s))` |
| Min Insertions to Make Palindrome | `n - LPS` |
| Min insertions + deletions to convert `s1 → s2` | `(n - LCS) + (m - LCS)` |
| SCS length | `n + m - LCS` |
| **SCS string** | **LCS table + backtrack: emit on match once, emit shrunk char otherwise, copy leftovers, reverse** |
