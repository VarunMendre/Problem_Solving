# Longest Increasing Subsequence: `dp[i]` Formulation and Printing the LIS

Problem link: [LeetCode 300: Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

> This file continues the earlier LIS notes (`longest_increasing_subsequence.md`), which solved the problem with the pick / not-pick state `(ind, prev)`. Here we use a different, more compact formulation, and then extend it to **print an actual LIS**.

---

## 1. Problem Statement

Given an integer array `nums`:

- **Part 1:** return the **length** of the longest strictly increasing subsequence.
- **Part 2:** return **one actual LIS** (the elements themselves).

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

Length: 4
One LIS: [2, 5, 7, 101]       (others: [2, 3, 7, 101], [2, 5, 7, 18], [2, 3, 7, 18])
```

---

## 2. Part 1: The `dp[i]` Formulation (Length Only)

### Redefining the state

The earlier DP used `f(ind, prev)`, remembering the **last picked** element as an extra dimension. A neater idea: **fix the last element to be `nums[i]` itself.**

```
dp[i] = length of the longest increasing subsequence that ENDS exactly at index i
```

Because the subsequence **ends at `i`**, the "last picked element" is already known (it is `i`), so the extra `prev` dimension disappears.

### The transition

A subsequence ending at `i` is either:

1. just `nums[i]` alone → length `1`, or
2. some increasing subsequence ending at an earlier index `prev` (with `nums[prev] < nums[i]`), **extended by** `nums[i]` → length `dp[prev] + 1`.

Take the best option:

```
dp[i] = max( 1,  1 + dp[prev]   for every prev < i with nums[prev] < nums[i] )
```

### The answer

The LIS can **end anywhere**, not necessarily at the last index. So the answer is:

```
answer = max over all i of dp[i]
```

### Code

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();

        vector<int> dp(n, 1);              // every element alone is an LIS of length 1
        int maxi = 0;

        for (int ind = 0; ind < n; ind++) {
            for (int prev = 0; prev <= ind; prev++) {
                if (nums[prev] < nums[ind])
                    dp[ind] = max(1 + dp[prev], dp[ind]);
            }

            maxi = max(maxi, dp[ind]);     // the LIS may end at any index
        }

        return maxi;
    }
};
```

> **Small cleanup:** the inner loop runs to `prev <= ind`. The case `prev == ind` can never pass `nums[prev] < nums[ind]` (a number is not less than itself), so it is harmless, but `prev < ind` is the precise bound.

### `dp` table for `nums = [10, 9, 2, 5, 3, 7, 101, 18]`

```
index   0    1    2    3    4    5     6     7
nums   10    9    2    5    3    7   101    18
dp      1    1    1    2    2    3     4     4

dp[3] = 2   → [2, 5]
dp[5] = 3   → [2, 5, 7]  or  [2, 3, 7]
dp[6] = 4   → [2, 5, 7, 101]
dp[7] = 4   → [2, 5, 7, 18]

answer = max(dp) = 4 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n²)` | Two nested loops |
| **Space** | `O(n)` | One `dp` array |

---

## 3. Part 2: How to Print the LIS

### The problem: `dp` only stores lengths

`dp[i]` tells us **how long** the best subsequence ending at `i` is, but not **which elements** form it. To rebuild the sequence we need to remember, for every `i`, **which `prev` produced the best `dp[i]`**.

### The parent (a.k.a. `hash`) array

```
hash[i] = index of the element that comes right BEFORE nums[i]
          in the best subsequence ending at i
```

Think of it as a chain of back-pointers:

```
hash[6] = 5   →   101 is preceded by 7  (index 5)
hash[5] = 3   →     7 is preceded by 5  (index 3)
hash[3] = 2   →     5 is preceded by 2  (index 2)
hash[2] = 2   →     2 points to ITSELF  → it is the start, nothing before it
```

### The sentinel: `hash[i] = i`

We initialize `hash[ind] = ind` for every index. It means "**no predecessor**: this element starts its own subsequence". If a better predecessor is found, `hash[ind]` is overwritten. While walking backwards, a node pointing at itself tells us the chain is finished.

### Step 1: Build `dp` and `hash` together

```cpp
for (int ind = 0; ind < n; ind++) {
    hash[ind] = ind;                                           // default: starts alone
    for (int prev = 0; prev < ind; prev++) {
        if (nums[prev] < nums[ind] && 1 + dp[prev] > dp[ind]) {
            dp[ind] = 1 + dp[prev];                            // better length found
            hash[ind] = prev;                                  // remember who gave it
        }
    }
    ...
}
```

**Why the extra condition `1 + dp[prev] > dp[ind]`?**
In the length-only version, `max(...)` hides this. Now `dp[ind]` and `hash[ind]` must always change **together**: we only update the parent when `dp[ind]` **strictly improves**. Otherwise `hash[ind]` could point to a predecessor that does not match the stored length.

### Step 2: Remember where the LIS ends

```cpp
if (dp[ind] > maxi) {
    maxi = dp[ind];
    lastInd = ind;           // the index at which the longest subsequence ends
}
```

The LIS ends at an index that holds the maximum `dp` value. We store **that index** (not only the length) so we know where to start walking backwards.

### Step 3: Walk the chain backwards

```cpp
vector<int> temp;
temp.push_back(nums[lastInd]);

while (hash[lastInd] != lastInd) {          // stop when a node points to itself
    lastInd = hash[lastInd];                // jump to the predecessor
    temp.push_back(nums[lastInd]);
}
```

### Step 4: Reverse

```cpp
reverse(temp.begin(), temp.end());
```

We walked from the **end** of the subsequence to its **start**, so `temp` is in reverse order. One `reverse` puts it in the right order.

---

## 4. Complete Working Code

The submitted printing code builds `temp` but never returns it (the function has no `return`, which is undefined behavior for a non-void function). Here is the complete version, returning the LIS itself:

```cpp
class Solution {
public:
    vector<int> printLIS(vector<int>& nums) {
        int n = nums.size();
        if (n == 0) return {};

        vector<int> dp(n, 1), hash(n);
        int maxi = 0;
        int lastInd = 0;

        for (int ind = 0; ind < n; ind++) {
            hash[ind] = ind;                                       // starts alone by default

            for (int prev = 0; prev < ind; prev++) {
                if (nums[prev] < nums[ind] && 1 + dp[prev] > dp[ind]) {
                    dp[ind] = 1 + dp[prev];
                    hash[ind] = prev;
                }
            }

            if (dp[ind] > maxi) {
                maxi = dp[ind];
                lastInd = ind;
            }
        }

        vector<int> temp;
        temp.push_back(nums[lastInd]);

        while (hash[lastInd] != lastInd) {
            lastInd = hash[lastInd];
            temp.push_back(nums[lastInd]);
        }

        reverse(temp.begin(), temp.end());

        return temp;                                               // the LIS itself
    }
};
```

(For the LeetCode problem, which asks only for the length, return `maxi`. Use `temp.size()` or `maxi` if you need both.)

---

## 5. Full Dry Run: `nums = [10, 9, 2, 5, 3, 7, 101, 18]`

### Building `dp` and `hash`

```
ind=0 (10):   no prev                                         dp=1  hash=0
ind=1 (9):    10<9? no                                        dp=1  hash=1
ind=2 (2):    10<2? no, 9<2? no                               dp=1  hash=2
ind=3 (5):    prev=2 (2<5): 1+1=2 > 1 ✓                       dp=2  hash=2
ind=4 (3):    prev=2 (2<3): 1+1=2 > 1 ✓                       dp=2  hash=2
ind=5 (7):    prev=2 (2<7): 2 > 1 ✓  → dp=2, hash=2
              prev=3 (5<7): 1+2=3 > 2 ✓ → dp=3, hash=3
              prev=4 (3<7): 1+2=3 > 3? NO (not strictly better)  hash stays 3
                                                              dp=3  hash=3
ind=6 (101):  prev=0 (10): 2 > 1 ✓ → dp=2, hash=0
              prev=3 (5):  3 > 2 ✓ → dp=3, hash=3
              prev=5 (7):  4 > 3 ✓ → dp=4, hash=5            dp=4  hash=5
ind=7 (18):   prev=5 (7):  4 > 3 ✓ → dp=4, hash=5             dp=4  hash=5
              prev=6 (101<18)? no
```

```
index   0    1    2    3    4    5     6     7
nums   10    9    2    5    3    7   101    18
dp      1    1    1    2    2    3     4     4
hash    0    1    2    2    2    3     5     5
```

### Tracking `lastInd` (strict `>`)

```
ind=0: dp=1 > 0 → maxi=1, lastInd=0
ind=3: dp=2 > 1 → maxi=2, lastInd=3
ind=5: dp=3 > 2 → maxi=3, lastInd=5
ind=6: dp=4 > 3 → maxi=4, lastInd=6
ind=7: dp=4 > 4? NO → lastInd stays 6
```

### Walking back from `lastInd = 6`

```
temp = [101]
hash[6]=5  ≠ 6 → lastInd=5  → push 7     temp = [101, 7]
hash[5]=3  ≠ 5 → lastInd=3  → push 5     temp = [101, 7, 5]
hash[3]=2  ≠ 3 → lastInd=2  → push 2     temp = [101, 7, 5, 2]
hash[2]=2  == 2 → STOP

reverse → [2, 5, 7, 101]  ✅
```

Check: `2 < 5 < 7 < 101`, length 4 = `maxi` ✅.

---

## 6. Why This Particular LIS (and Not Another)?

Several LIS exist: `[2,5,7,101]`, `[2,3,7,101]`, `[2,5,7,18]`, `[2,3,7,18]`. Any is accepted. The code's tie-breaking decides which one you get:

- **`1 + dp[prev] > dp[ind]` (strict):** the **first** `prev` that reaches the best length wins. At `ind=5`, both `prev=3` (value 5) and `prev=4` (value 3) give length 3, and `prev=3` is kept.
- **`dp[ind] > maxi` (strict):** the **first** index that reaches the maximum length is kept as `lastInd`. Both indices 6 (value 101) and 7 (value 18) have `dp = 4`, and index 6 wins.

Changing either `>` to `>=` would pick the **last** candidate instead and print a different (still valid) LIS.

---

## 7. Common Mistakes

| Mistake | What goes wrong |
|---|---|
| Updating `hash[ind]` without checking `1 + dp[prev] > dp[ind]` | The parent may not match the stored length, so the printed sequence can be shorter or wrong |
| Assuming the LIS ends at index `n - 1` | The LIS may end anywhere, so track `lastInd` |
| Forgetting to initialize `hash[ind] = ind` | The backward walk never finds a stopping point |
| Forgetting `reverse` | The sequence is printed from end to start |
| Missing `return` | Undefined behavior in a non-void function |
| Empty array | `nums[lastInd]` is out of bounds, so guard with `if (n == 0) return {}` |

---

## 8. Complexity Analysis

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n²)` | Two nested loops; backtracking adds `O(n)` |
| **Space** | `O(n)` | `dp` and `hash` arrays (plus the `O(n)` output) |

---

## 9. How This Compares With the Other LIS Approaches

| Approach | Time | Space | Can print an LIS? |
|---|---|---|---|
| `(ind, prev)` pick / not-pick DP | `O(n²)` | `O(n²)` or `O(n)` | Awkward (needs table backtracking) |
| **`dp[i]` ending at `i` + `hash`** | **`O(n²)`** | **`O(n)`** | **Yes, simply (this file)** |
| Binary search on `tails` | `O(n log n)` | `O(n)` | Not directly: `tails` is **not** an LIS (needs extra bookkeeping) |

This `dp[i]` + parent-array pattern is exactly what the **printing** variants of other DP problems use (largest divisible subset, longest string chain, and so on): compute a `dp` of lengths, keep a parent pointer, remember the best end, then walk back and reverse.
