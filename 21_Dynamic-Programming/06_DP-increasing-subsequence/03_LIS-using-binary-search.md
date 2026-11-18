# Longest Increasing Subsequence: The Binary Search (`O(n log n)`) Approach

Problem link: [LeetCode 300: Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

> Earlier notes solved LIS with the pick / not-pick DP `(ind, prev)` and with `dp[i]` (LIS ending at `i`). Both are `O(n²)`. This file shows **how to arrive at** the `O(n log n)` binary search solution step by step, instead of just stating the trick.

---

## 1. Problem Statement

Given an integer array `nums`, return the **length** of the longest strictly increasing subsequence.

```
nums = [2, 5, 3, 7, 11, 8, 10, 13, 6]

LIS = [2, 3, 7, 8, 10, 13]   → length 6
```

We only need the **length**, not the subsequence itself. That detail is what makes the optimization possible.

---

## 2. How the Idea Develops (the Thought Process)

### Stage A: "Track every increasing subsequence"

The most natural plan: scan left to right and keep **all increasing subsequences found so far**. For each new number `x`:

- start a new sequence `[x]`
- for every existing sequence whose **last** element is `< x`, create a copy extended by `x`

```
nums = [2, 5, 3, 7, ...]

after 2:   [2]
after 5:   [2]  [5]  [2,5]
after 3:   [2]  [5]  [2,5]  [3]  [2,3]
after 7:   [2]  [5]  [2,5]  [3]  [2,3]  [7]  [2,7]  [5,7]  [2,5,7]  [3,7]  [2,3,7]
...
```

The answer would be the length of the longest list. But this is **hectic**: the number of increasing subsequences can be exponential (a sorted array of `n` numbers has `2ⁿ - 1` of them). Storing and extending all of them costs exponential time **and** space. We need to throw information away safely.

### Stage B: "What do we actually need from a sequence?"

To decide whether a new number `x` can extend a sequence, we only ask: **is `x` larger than its last element?** The inner elements never matter again.

So for each sequence the only facts that matter are:

```
(length, last element = "tail")
```

### Stage C: "For the same length, keep only the best tail"

Take two sequences of the **same length**: `[2, 5]` and `[2, 3]`. Their tails are `5` and `3`.

Which one is more useful for the future? The **smaller tail**, because every number that can extend `[2, 5]` (anything `> 5`) can also extend `[2, 3]` (anything `> 3`), and `[2, 3]` can additionally be extended by `4`.

```
future number 4:   [2,5] + 4 → not increasing ✗
                   [2,3] + 4 → [2,3,4]        ✓
```

So `[2, 3]` **dominates** `[2, 5]`. For each length we keep only **one** sequence: the one with the smallest tail.

```
after 2:   len1: [2]
after 5:   len1: [2]   len2: [2,5]
after 3:   len1: [2]   len2: [2,3]        ← [2,3] replaces [2,5] (tail 3 < 5)
after 7:   len1: [2]   len2: [2,3]   len3: [2,3,7]
```

### Stage D: "We don't even need the sequences, only the tails"

Since we only need the **length**, we don't have to store whole sequences. For each length `L` just store the **smallest tail** among all increasing subsequences of length `L`:

```
temp[L-1] = the smallest possible tail of an increasing subsequence of length L
```

```
after 2, 5, 3, 7:     temp = [2, 3, 7]
                              ↑  ↑  ↑
                          len1 len2 len3 tails
```

That is it. The "hectic" collection of sequences has shrunk to **one array of numbers**, and the length of the answer is simply `temp.size()`.

> **"Overriding an element in the sequence"** (your words) is exactly Stage C in action: when a new number gives a **smaller tail** for some length, we overwrite that slot.

### Stage E: "`temp` is sorted, so binary search works"

`temp` is always **strictly increasing**:

```
temp[0] < temp[1] < temp[2] < ...
```

**Why:** take the best subsequence of length `L + 1`, with tail `t`. Remove its last element. What is left is an increasing subsequence of length `L` whose tail is **smaller than `t`**. So the smallest tail for length `L` is smaller than the smallest tail for length `L + 1`.

Because `temp` is sorted, finding "the slot for `x`" can use **binary search** instead of a linear scan. That turns `O(n)` per number into `O(log n)`.

---

## 3. The Algorithm: Two Cases Per Number

For each number `x`:

### Case 1: `x` is larger than every tail (`x > temp.back()`)

`x` can extend the **longest** subsequence found so far.

```
temp.push_back(x);       // the LIS length grows by 1
```

### Case 2: otherwise, find the first tail `≥ x` and overwrite it

```
p = lower_bound(temp, x)      // first position with temp[p] >= x
temp[p] = x;
```

**Why this is correct:**

- Because `p` is the **first** tail `≥ x`, we know `temp[p-1] < x`. So we can take the length-`p` subsequence with tail `temp[p-1]` and add `x`, giving a **real** increasing subsequence of length `p + 1` that ends in `x`. (If `p = 0`, it is just `[x]`.)
- Since `x ≤ temp[p]`, this is a **better or equal** tail for length `p + 1`, so we replace `temp[p]`.
- The replacement is **safe**: anything that could extend the old tail (a future `y > temp[p]`) can also extend `x` (because `y > temp[p] ≥ x`). We never lose an option.
- The **length of `temp` does not change**, because we did not find a longer subsequence, only a better tail for an existing length.

---

## 4. Code

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();

        vector<int> temp;
        temp.push_back(nums[0]);
        int len = 1;

        for (int i = 1; i < n; i++) {
            if (nums[i] > temp.back()) {                  // Case 1: extends the longest
                temp.push_back(nums[i]);
                len++;
            } else {                                      // Case 2: improve a tail
                int ind = lower_bound(temp.begin(), temp.end(), nums[i]) -
                          temp.begin();
                temp[ind] = nums[i];
            }
        }

        return len;
    }
};
```

### Line-by-line notes

- `temp.push_back(nums[0])`: the first number alone is an increasing subsequence of length 1. (This line assumes `n ≥ 1`; add `if (n == 0) return 0;` for safety.)
- `nums[i] > temp.back()`: `temp.back()` is the **largest** tail (tail of the longest subsequence). Beating it means we can lengthen the LIS.
- `lower_bound(...)`: returns the first position whose value is **`≥ nums[i]`**. In the `else` branch we know `nums[i] ≤ temp.back()`, so a valid position always exists (the index is always `< temp.size()`).
- `temp[ind] = nums[i]`: overwrite with the smaller (or equal) tail.
- `len` always equals `temp.size()`; you can drop `len` and return `temp.size()`.

---

## 5. Dry Run: `nums = [2, 5, 3, 7, 11, 8, 10, 13, 6]`

```
x     action                                   temp                         len
2     empty → push                             [2]                           1
5     5 > 2    → push                          [2, 5]                        2
3     3 ≤ 5    → lower_bound(3) = pos 1        [2, 3]                        2    ← 5 replaced by 3
7     7 > 3    → push                          [2, 3, 7]                     3
11    11 > 7   → push                          [2, 3, 7, 11]                 4
8     8 ≤ 11   → lower_bound(8) = pos 3        [2, 3, 7, 8]                  4    ← 11 replaced by 8
10    10 > 8   → push                          [2, 3, 7, 8, 10]              5
13    13 > 10  → push                          [2, 3, 7, 8, 10, 13]          6
6     6 ≤ 13   → lower_bound(6) = pos 2        [2, 3, 6, 8, 10, 13]          6    ← 7 replaced by 6

Answer: 6 ✅
```

What each overwrite means:

- `3` replaced `5`: the best length-2 sequence now ends in `3` (`[2, 3]`) instead of `5`.
- `8` replaced `11`: the best length-4 sequence now ends in `8` (`[2, 3, 7, 8]`).
- `6` replaced `7`: the best length-3 sequence now ends in `6` (`[2, 3, 6]`). This did **not** change the length, but a future `7`-ish number would now have a better chance to extend length 3.

### Second example: `[10, 9, 2, 5, 3, 7, 101, 18]`

```
10  → [10]
9   → lower_bound(9)=0   → [9]
2   → lower_bound(2)=0   → [2]
5   → push               → [2, 5]
3   → lower_bound(3)=1   → [2, 3]
7   → push               → [2, 3, 7]
101 → push               → [2, 3, 7, 101]
18  → lower_bound(18)=3  → [2, 3, 7, 18]

Answer: 4 ✅
```

---

## 6. Important Caveat: `temp` Is NOT Necessarily an Actual LIS

Only the **size** of `temp` is meaningful as an answer. The values may come from **different** subsequences, and may not even appear in that order in `nums`.

```
nums = [1, 2, 3, 0]

1  → [1]
2  → [1, 2]
3  → [1, 2, 3]
0  → lower_bound(0) = 0 → [0, 2, 3]

len = 3 ✅  (correct: real LIS is [1, 2, 3])

But [0, 2, 3] is NOT a subsequence of nums: 0 comes AFTER 2 and 3.
```

In the main example the final `temp = [2, 3, 6, 8, 10, 13]` is also not a real subsequence (`6` appears after `13`), yet its length `6` is right. That is acceptable because this approach is only designed to find the **length**.

---

## 7. Why `lower_bound` and Not `upper_bound`?

The problem asks for a **strictly** increasing subsequence.

`lower_bound(x)` returns the first position with value `≥ x`. If `x` is already in `temp`, it points **at that same value**, and overwriting it with `x` changes nothing. So a duplicate never grows the length:

```
nums = [7, 7, 7, 7]

7 → [7]
7 → 7 > 7? no → lower_bound(7)=0 → [7]     (overwritten with itself)
...
len = 1 ✅
```

`upper_bound(x)` returns the first position with value `> x`; it would let a duplicate extend the sequence, giving the length of the longest **non-decreasing** subsequence. (In that version, the "push" condition also changes to `>=`.)

| Function | Counts | Equal values |
|---|---|---|
| `lower_bound` | strictly increasing | Do not extend |
| `upper_bound` | non-decreasing | Extend |

---

## 8. Bonus: Printing the LIS With This Approach

`temp` alone cannot be printed (section 6), but with a little bookkeeping we can rebuild a real LIS. Keep, for each slot, the **index in `nums`** of its tail, and for each element, a **parent** pointer to the tail of the previous length at the time it was placed:

```cpp
vector<int> printLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return {};

    vector<int> tailsVal;            // same as temp
    vector<int> tailsIdx;            // tailsIdx[k] = index in nums of the tail of length k+1
    vector<int> parent(n, -1);       // parent[i] = index of the element before nums[i]

    for (int i = 0; i < n; i++) {
        int p = lower_bound(tailsVal.begin(), tailsVal.end(), nums[i]) - tailsVal.begin();

        if (p == (int)tailsVal.size()) {       // extends the longest
            tailsVal.push_back(nums[i]);
            tailsIdx.push_back(i);
        } else {                                // improves a tail
            tailsVal[p] = nums[i];
            tailsIdx[p] = i;
        }

        parent[i] = (p > 0) ? tailsIdx[p - 1] : -1;   // who sits just before us
    }

    vector<int> lis;
    for (int i = tailsIdx.back(); i != -1; i = parent[i])
        lis.push_back(nums[i]);

    reverse(lis.begin(), lis.end());
    return lis;
}
```

Trace on `[2, 5, 3, 7, 11, 8, 10, 13, 6]`:

```
i  x    p   tailsIdx (after)        parent[i]
0  2    0   [0]                      -1
1  5    1   [0,1]                    0   (2)
2  3    1   [0,2]                    0   (2)
3  7    2   [0,2,3]                  2   (3)
4  11   3   [0,2,3,4]                3   (7)
5  8    3   [0,2,3,5]                3   (7)
6  10   4   [0,2,3,5,6]              5   (8)
7  13   5   [0,2,3,5,6,7]            6   (10)
8  6    2   [0,2,8,5,6,7]            2   (3)

Start at tailsIdx.back() = 7 (13) → 6 (10) → 5 (8) → 3 (7) → 2 (3) → 0 (2) → -1
Reversed:  [2, 3, 7, 8, 10, 13]  ✅  a real increasing subsequence of length 6
```

---

## 9. Complexity Analysis

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n log n)` | `n` numbers, each with one `O(log n)` binary search |
| **Space** | `O(n)` | The `temp` array (at most `n` entries) |

---

## 10. All LIS Approaches Compared

| Approach | Time | Space | Prints LIS? |
|---|---|---|---|
| `(ind, prev)` pick / not-pick DP | `O(n²)` | `O(n²)` / `O(n)` | Awkward |
| `dp[i]` ending at `i` + parent array | `O(n²)` | `O(n)` | Yes |
| **Binary search on tails (this file)** | **`O(n log n)`** | **`O(n)`** | Only with extra bookkeeping |

---

## 11. Common Mistakes

| Mistake | What goes wrong |
|---|---|
| Treating `temp` as the LIS | It may not be a real subsequence; only `temp.size()` is the answer |
| Using `upper_bound` for a strict LIS | Duplicates would wrongly extend the length |
| Forgetting that `temp` must stay sorted | Binary search would be invalid (it stays sorted automatically with this algorithm) |
| `nums[0]` on an empty array | Out-of-bounds read; guard with `if (n == 0) return 0` |
| Thinking the overwrite shortens the LIS | The size never decreases; overwriting only improves a tail |

---

## 12. One-Paragraph Summary

Tracking every increasing subsequence is exponential. We only need to know **how long** a subsequence can get, and whether a new number can extend it, which depends only on the subsequence's **last element**. For each length, only the **smallest tail** matters, because it can be extended by more future numbers. Storing just those tails gives a **sorted** array, so each new number can be placed with **binary search**: append it if it beats every tail (LIS grows), otherwise overwrite the first tail that is `≥` it (a better tail for that length). The answer is the size of the array.
