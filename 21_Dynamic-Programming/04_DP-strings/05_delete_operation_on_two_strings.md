# Delete Operation for Two Strings

## Problem Statement

Given two strings `word1` and `word2`, return the *minimum number of steps* required to make `word1` and `word2` identical.

In one step, you can delete exactly one character in either string.

---

### Example

**Input:** `word1 = "sea"`, `word2 = "eat"`

**Output:** `2`

**Explanation:** 
* Step 1: Delete `'s'` from `"sea"` to get `"ea"`.
* Step 2: Delete `'t'` from `"eat"` to get `"ea"`.

Now both strings are identical (`"ea"`), which required **2 deletions**.

---

## Solution (LCS Reduction / Space-Optimized DP)

### 1. Approach / Intuition

To transform both strings into an identical string using the minimum number of deletions, we should preserve as many common characters as possible without changing their relative order. The longest sequence of characters present in both strings in the same order is their **Longest Common Subsequence (LCS)**.

Once we find the length of the LCS ($lcs$):
1. Number of deletions required in `word1` = $\text{length}(word1) - lcs$
2. Number of deletions required in `word2` = $\text{length}(word2) - lcs$

Therefore, total minimum deletions required:
$$ \text{Total Deletions} = (n - lcs) + (m - lcs) = n + m - 2 \cdot lcs $$

---

### 2. Why We Think About This

Instead of directly deleting characters and testing every combination (which would be exponential), we realize that both strings must eventually match at a common sequence. To minimize deletions, this common sequence must be as **long as possible**. Finding the longest common sequence between two strings is a classic dynamic programming problem: **Longest Common Subsequence (LCS)**.

---

### 3. Dry-Run

**Input:** `word1 = "sea"`, `word2 = "eat"`  
Length $n = 3$, $m = 3$.

#### Step 1: Compute LCS using Space-Optimized DP
Initialize `prev = [0, 0, 0, 0]` (for string `word2` length $m=3$).

* **$i = 1$ (`word1[0] = 's'`):**
  * $j = 1$ (`word2[0] = 'e'`): `'s' != 'e'` $\rightarrow \text{cur}[1] = \max(\text{prev}[1], \text{cur}[0]) = \max(0, 0) = 0$
  * $j = 2$ (`word2[1] = 'a'`): `'s' != 'a'` $\rightarrow \text{cur}[2] = \max(\text{prev}[2], \text{cur}[1]) = \max(0, 0) = 0$
  * $j = 3$ (`word2[2] = 't'`): `'s' != 't'` $\rightarrow \text{cur}[3] = \max(\text{prev}[3], \text{cur}[2]) = \max(0, 0) = 0$  
  `prev = [0, 0, 0, 0]`

* **$i = 2$ (`word1[1] = 'e'`):**
  * $j = 1$ (`word2[0] = 'e'`): `'e' == 'e'` $\rightarrow \text{cur}[1] = 1 + \text{prev}[0] = 1 + 0 = 1$
  * $j = 2$ (`word2[1] = 'a'`): `'e' != 'a'` $\rightarrow \text{cur}[2] = \max(\text{prev}[2], \text{cur}[1]) = \max(0, 1) = 1$
  * $j = 3$ (`word2[2] = 't'`): `'e' != 't'` $\rightarrow \text{cur}[3] = \max(\text{prev}[3], \text{cur}[2]) = \max(0, 1) = 1$  
  `prev = [0, 1, 1, 1]`

* **$i = 3$ (`word1[2] = 'a'`):**
  * $j = 1$ (`word2[0] = 'e'`): `'a' != 'e'` $\rightarrow \text{cur}[1] = \max(\text{prev}[1], \text{cur}[0]) = \max(1, 0) = 1$
  * $j = 2$ (`word2[1] = 'a'`): `'a' == 'a'` $\rightarrow \text{cur}[2] = 1 + \text{prev}[1] = 1 + 1 = 2$
  * $j = 3$ (`word2[2] = 't'`): `'a' != 't'` $\rightarrow \text{cur}[3] = \max(\text{prev}[3], \text{cur}[2]) = \max(1, 2) = 2$  
  `prev = [0, 1, 2, 2]`

**LCS result:** $lcs = \text{prev}[3] = 2$ (The common subsequence is `"ea"`).

#### Step 2: Calculate Min Deletions
* Deletions in `word1` = $3 - 2 = 1$
* Deletions in `word2` = $3 - 2 = 1$
* **Total Min Distance:** $1 + 1 = 2$.

---

### 4. Story Points

* **Identify the Core Goal:** Find the longest common part that both strings can keep.
* **LCS Helper Function:** Use dynamic programming to compute the length of the longest common subsequence efficiently.
* **Space Optimization:** Keep track of only two rows (`prev` and `cur`) instead of a full $(N+1) \times (M+1)$ matrix to optimize memory usage.
* **Math Formula:** Subtract the LCS length from each string's total length to get the required deletions for each, then sum them up.

---

### 5. Code

```cpp
class Solution {
private:
    // Helper function to find the length of the Longest Common Subsequence (LCS)
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size();
        int m = text2.size();

        // 1D vector to store DP state of previous row
        vector<int> prev(m + 1, 0);

        for (int i = 1; i <= n; i++) {
            // 1D vector to store DP state of current row
            vector<int> cur(m + 1, 0);
            for (int j = 1; j <= m; j++) {
                // If characters match, increment LCS length from previous diagonal
                if (text1[i - 1] == text2[j - 1])
                    cur[j] = 1 + prev[j - 1];
                // If characters don't match, take max by ignoring current char from either string
                else
                    cur[j] = max(prev[j], cur[j - 1]);
            }

            // Move current row to previous for the next iteration
            prev = cur;
        }

        return prev[m];
    }

public:
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();

        // Find the length of the longest sequence both strings share
        int lcs = longestCommonSubsequence(word1, word2);

        // Deletions needed = chars to remove from word1 + chars to remove from word2
        return (n - lcs) + (m - lcs);
    }
};
```

---

### 6. Time & Space Complexity

* **Time Complexity:** $\mathcal{O}(N \times M)$  
  We iterate through nested loops of size $N$ (`word1.length()`) and $M$ (`word2.length()`) to compute the LCS.
  
* **Space Complexity:** $\mathcal{O}(M)$  
  Using space optimization, we only maintain two 1D arrays (`prev` and `cur`) of size $(M + 1)$, where $M$ is the length of `word2`.