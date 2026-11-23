# Longest String Chain

Problem link: [LeetCode 1048: Longest String Chain](https://leetcode.com/problems/longest-string-chain/)

---

## 1. Problem Statement

`wordA` is a **predecessor** of `wordB` if you can obtain `wordB` by **inserting exactly one letter** anywhere into `wordA` (without reordering the other letters).

A **word chain** is a sequence `[w1, w2, ..., wk]` where each word is a predecessor of the next. Given a list `words`, return the **length of the longest possible chain** using words from the list.

```
words = ["xbc", "pcxbcf", "xb", "cxbc", "pcxbc"]

Chain: "xb" → "xbc" → "cxbc" → "pcxbc" → "pcxbcf"

  xb       → xbc      (insert 'c' at the end)
  xbc      → cxbc     (insert 'c' at the front)
  cxbc     → pcxbc    (insert 'p' at the front)
  pcxbc    → pcxbcf   (insert 'f' at the end)

Answer: 5
```

```
words = ["a", "b", "ba", "bca", "bda", "bdca"]   → 4   (a → ba → bda → bdca)
words = ["abcd", "dbqca"]                         → 1   (no word is a predecessor of another)
```

---

## 2. How This Problem Relates to LIS

### The LIS recurrence (recap)

```
dp[i] = length of the longest increasing subsequence ENDING at index i

for every prev < i:
    if nums[prev] < nums[i]:
        dp[i] = max(dp[i], 1 + dp[prev])

answer = max over all dp[i]
```

### The same idea for words

```
dp[i] = length of the longest word chain ENDING at words[i]

for every prev before i:
    if words[prev] is a predecessor of words[i]:
        dp[i] = max(dp[i], 1 + dp[prev])

answer = max over all dp[i]
```

The structure is **identical**. Only two things change:

| | LIS | Longest String Chain |
|---|---|---|
| **Element** | number | word |
| **"`prev` can come right before `i`" test** | `nums[prev] < nums[i]` | `compare(words[i], words[prev])`: is `words[prev]` a predecessor of `words[i]`? |
| **Order of processing** | the array's own order | **sort the words by length first** |
| **`dp[i]` means** | longest increasing subsequence ending at `i` | longest chain ending at `words[i]` |
| **Answer** | `max(dp)` | `max(dp)` |

So "we've just changed a few things": the **comparison function** (what counts as a valid step) and a **sort** (to make the iteration order meaningful). Everything else, including `dp` initialized to `1` (every word alone is a chain of length 1) and tracking `maxi`, is copied from LIS.

### A unifying view: longest path in a DAG

Think of each element as a node and draw an edge `prev → i` whenever `prev` can directly precede `i`. Both problems ask for the **longest path** in that graph. DP works when we process nodes in an order where every edge goes **forward**:

- LIS: edges go from a smaller index to a larger index automatically (the array order is the order).
- String chain: edges go from a shorter word to a longer word, so we sort by length to make every edge go forward.

---

## 3. The `compare` Function Explained

```cpp
bool compare(string& s1, string& s2) {
    if (s1.size() != s2.size() + 1)
        return false;

    int ptr1 = 0;
    int ptr2 = 0;

    while (ptr1 < s1.size()) {
        if (s1[ptr1] == s2[ptr2] && ptr2 < s2.size()) {
            ptr1++, ptr2++;
        } else {
            ptr1++;
        }
    }

    return (ptr1 == s1.size() && ptr2 == s2.size()) ? true : false;
}
```

**Meaning:** `compare(s1, s2)` returns `true` if **`s2` is a predecessor of `s1`**, i.e. `s1` is `s2` with exactly one extra letter. `s1` is the longer word.

### Step 1: the length check

```cpp
if (s1.size() != s2.size() + 1) return false;
```

A predecessor is exactly **one letter shorter**. Any other length difference can be rejected immediately.

### Step 2: two pointers

- `ptr1` walks through the **longer** word `s1`.
- `ptr2` walks through the **shorter** word `s2`.

At each step:

| Situation | Action | Meaning |
|---|---|---|
| `s1[ptr1] == s2[ptr2]` | advance **both** pointers | this letter of `s1` is matched with the next letter of `s2` |
| letters differ | advance **only `ptr1`** | this letter of `s1` is the **extra (inserted)** letter, skip it |

### Step 3: the final check

If every letter of `s2` was matched (`ptr2 == s2.size()`), then `s2` is a subsequence of `s1`. Because `s1` is exactly one letter longer, there is exactly **one** unmatched letter in `s1`, the inserted one. So `s1` is `s2` plus one letter.

> `ptr1 == s1.size()` is always true after the loop ends (the loop runs until it is), so the condition that actually decides the result is `ptr2 == s2.size()`.

### Traces

**Success: `compare("cxbc", "xbc")`** (`"xbc"` is a predecessor of `"cxbc"`)

```
s1 = c x b c      s2 = x b c        sizes: 4 == 3 + 1 ✓

ptr1=0 'c' vs ptr2=0 'x'  → differ  → skip 'c'          ptr1=1
ptr1=1 'x' vs ptr2=0 'x'  → match   → both advance        ptr1=2, ptr2=1
ptr1=2 'b' vs ptr2=1 'b'  → match                          ptr1=3, ptr2=2
ptr1=3 'c' vs ptr2=2 'c'  → match                          ptr1=4, ptr2=3

ptr2 == 3 == s2.size()  → true ✅   (the inserted letter was the leading 'c')
```

**Failure: `compare("abd", "abc")`** (same length difference, but the letters differ)

```
a=a (1,1)    b=b (2,2)    'd' vs 'c' → differ → skip 'd'    ptr1=3

ptr2 = 2, but s2.size() = 3  → false ✅
```

**Failure: `compare("abc", "xb")`**

```
'a' vs 'x' → skip     'b' vs 'x' → skip     'c' vs 'x' → skip
ptr2 = 0 ≠ 2 → false ✅
```

**Inserted letter at the very end: `compare("pcxbcf", "pcxbc")`**

```
p,c,x,b,c all match   → ptr1=5, ptr2=5
'f' vs s2[5] (end of string) → no match → skip 'f'   ptr1=6

ptr2 = 5 == s2.size() → true ✅
```

### A small safety cleanup

In `s1[ptr1] == s2[ptr2] && ptr2 < s2.size()` the array access happens **before** the bounds check. For `std::string`, `s2[s2.size()]` is a valid read of the terminating `'\0'`, so this does not crash, but the clean order is to check bounds first:

```cpp
if (ptr2 < s2.size() && s1[ptr1] == s2[ptr2])
```

---

## 4. Why Sorting Is Necessary

### The key difference from LIS

In LIS, the answer is a **subsequence**, so the original array order is part of the problem: you may only pick elements left to right.

In a word chain, the answer is a **set of words arranged by length**. The order in which the words appear in the input is **irrelevant**. So we are free to reorder the array however we like, and we choose an order that makes the DP work.

### What goes wrong without sorting

The DP only looks at **earlier** indices (`prev < ind`). That is only valid if every predecessor of a word **appears before it** in the array, and its `dp` value is already final.

Take the test case:

```
words = ["xbc", "pcxbcf", "xb", "cxbc", "pcxbc"]
idx:       0        1       2      3        4
```

**Without sorting (original order):**

```
idx  word      predecessor among EARLIER indices           dp
 0   xbc       none                                         1
 1   pcxbcf    none  ("pcxbc" is at idx 4, which is LATER)  1
 2   xb        none                                         1
 3   cxbc      xbc (idx 0)                                  2
 4   pcxbc     cxbc (idx 3)                                 3

maxi = 3   ✗   (expected 5)
```

Two things break:

1. **`"xb"` is the predecessor of `"xbc"`, but it appears later** (idx 2 vs idx 0). When `dp[0]` is computed, `"xb"` is not among the earlier words, so `dp[0]` stays `1` instead of `2`. That error then propagates: `cxbc`, `pcxbc` and `pcxbcf` are all one short of their true values.
2. **`"pcxbcf"` is processed before its predecessor `"pcxbc"`** (idx 1 vs idx 4). It cannot see `"pcxbc"`, so `dp[1]` stays `1`.

Even if we let `prev` range over **all** indices instead of only earlier ones, it would not help: `dp[prev]` would be read **before** it is final (it would still hold its initial `1`).

> **Rule:** `dp[i]` may only be computed **after** the `dp` of all its predecessors is final.

### With sorting by length

```
sorted: ["xb", "xbc", "cxbc", "pcxbc", "pcxbcf"]      lengths: 2, 3, 4, 5, 6
```

```
idx  word      predecessor found     dp
 0   xb        none                   1
 1   xbc       xb                     2
 2   cxbc      xbc                    3
 3   pcxbc     cxbc                   4
 4   pcxbcf    pcxbc                  5

maxi = 5   ✅
```

Every word now appears **after** its predecessor, so each `dp` value is final by the time it is used.

### Why sorting by **length** is exactly the right order

- A predecessor is **exactly one letter shorter**, so it always lands **strictly earlier** in a length-sorted array.
- Words of the **same length** can never be predecessors of each other, so their relative order does not matter (stability is unnecessary).
- The comparator is `s1.size() < s2.size()`, a valid strict weak ordering.

Sorting does not lose any information: the problem's chains are determined by word lengths and letters, not by input positions.

---

## 5. The Complete Code

```cpp
class Solution {
private:
    // true if s2 is a predecessor of s1 (s1 = s2 with exactly one extra letter)
    bool compare(string& s1, string& s2) {
        if (s1.size() != s2.size() + 1)
            return false;

        int ptr1 = 0;
        int ptr2 = 0;

        while (ptr1 < s1.size()) {
            if (s1[ptr1] == s2[ptr2] && ptr2 < s2.size()) {
                ptr1++, ptr2++;
            } else {
                ptr1++;
            }
        }

        return (ptr1 == s1.size() && ptr2 == s2.size()) ? true : false;
    }

public:
    static bool comp(string& s1, string& s2) {
        return s1.size() < s2.size();
    }

    int longestStrChain(vector<string>& words) {
        int n = words.size();

        sort(words.begin(), words.end(), comp);        // shorter words first

        vector<int> dp(n, 1);                          // every word alone is a chain of length 1
        int maxi = 0;

        for (int ind = 0; ind < n; ind++) {
            for (int prev = 0; prev <= ind; prev++) {
                if (compare(words[ind], words[prev]) &&
                    dp[prev] + 1 > dp[ind])
                    dp[ind] = max(1 + dp[prev], dp[ind]);
            }

            maxi = max(maxi, dp[ind]);
        }

        return maxi;
    }
};
```

### Small cleanups (optional)

- The inner loop `prev <= ind` includes `prev == ind`. `compare(w, w)` is always `false` (equal lengths), so it is harmless, but `prev < ind` is the precise bound.
- The condition `dp[prev] + 1 > dp[ind]` already guarantees the update is an improvement, so `max(1 + dp[prev], dp[ind])` simply equals `1 + dp[prev]`.
- Take parameters as `const string&` so the code works with temporaries and states the intent.

---

## 6. Complexity Analysis

Let `n` = number of words and `L` = maximum word length.

| Step | Cost |
|---|---|
| Sort by length | `O(n log n)` |
| Pairs `(ind, prev)` | `O(n²)` |
| Each `compare` | `O(L)` |

| | Complexity |
|---|---|
| **Time** | `O(n log n + n² · L)` |
| **Space** | `O(n)` for `dp` (sorting may use extra stack space) |

With `n ≤ 1000` and `L ≤ 16` this is about `1.6 × 10⁷` operations, comfortably fast.

---

## 7. Alternative: Hash Map Over Deleted-Letter Predecessors

Instead of comparing **every pair** of words, generate the **predecessors of each word** directly: delete each letter once (at most `L` candidates) and look each one up in a map.

```cpp
class Solution {
public:
    int longestStrChain(vector<string>& words) {
        sort(words.begin(), words.end(),
             [](const string& a, const string& b) { return a.size() < b.size(); });

        unordered_map<string, int> dp;       // dp[word] = longest chain ending at word
        int best = 1;

        for (const string& w : words) {
            int cur = 1;
            for (int i = 0; i < (int)w.size(); i++) {
                string pred = w.substr(0, i) + w.substr(i + 1);   // delete letter i
                auto it = dp.find(pred);
                if (it != dp.end())
                    cur = max(cur, it->second + 1);
            }
            dp[w] = cur;
            best = max(best, cur);
        }

        return best;
    }
};
```

Sorting is still required for the same reason: a word's predecessors must be in the map **before** the word is processed.

| | Pair comparison (above) | Hash map |
|---|---|---|
| **Time** | `O(n² · L)` | `O(n · L²)` |
| **Space** | `O(n)` | `O(n · L)` |
| **Better when** | few words, long words | many words, short words |

---

## 8. The LIS Family

| Problem | Order trick | "Can `prev` precede `i`?" test |
|---|---|---|
| LIS | array order | `nums[prev] < nums[i]` |
| Print LIS | array order + parent array | same |
| LIS with binary search | tails array | `lower_bound` |
| **Longest String Chain** | **sort by length** | **`prev` is `i` with one letter removed** |
| Largest Divisible Subset | sort by value | `nums[i] % nums[prev] == 0` |
| Russian Doll Envelopes | sort by width | height and width both strictly larger |

All of them are the same pattern: **impose an order so predecessors come first, define what a valid step is, then take `dp[i] = 1 + max dp[prev]`.**
