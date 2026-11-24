# Longest Bitonic Subsequence

---

## 1. Problem Statement

A **bitonic subsequence** first **strictly increases** up to a peak and then **strictly decreases**. Given an array `arr` of `n` integers, return the length of the **longest bitonic subsequence**.

```
arr = [1, 11, 2, 10, 4, 5, 2, 1]

One longest bitonic subsequence: [1, 2, 10, 5, 2, 1]
                                  ↑ increasing ↑  ↑ decreasing ↑
                                  1 < 2 < 10      10 > 5 > 2 > 1
Answer: 6
```

```
arr = [12, 4, 78, 90, 45, 23]   → 5   ([4, 78, 90, 45, 23])
arr = [1, 2, 3, 4, 5]           → 5   (purely increasing counts: the "decreasing" part is empty)
arr = [5, 4, 3, 2, 1]           → 5   (purely decreasing counts too)
```

---

## 2. How This Relates to LIS

A bitonic sequence is **two monotone pieces glued at a peak**:

```
 value
   ^
   |            *  ← peak
   |          *   *
   |        *       *
   |      *           *
   +--------------------------> index
     increasing   decreasing
```

For a fixed peak at index `i`:

- the **left part** is an **increasing subsequence ending at `i`** (uses indices `≤ i`)
- the **right part** is a **decreasing subsequence starting at `i`** (uses indices `≥ i`)

The two parts use **disjoint index ranges** (except for the shared peak), so they can be optimized **independently**:

```
best bitonic with peak i = (longest increasing subsequence ending at i)
                         + (longest decreasing subsequence starting at i)
                         − 1                                  ← the peak is counted twice
```

The answer is the maximum of this over all possible peaks `i`.

This is the standard pattern **"compute a prefix DP and a suffix DP, then combine them at every index."** The prefix DP is exactly the LIS `dp[i]` from the earlier notes.

---

## 3. `dp1`: Longest Increasing Subsequence Ending at `i`

```
dp1[i] = length of the longest strictly increasing subsequence ending at index i
```

Identical to the LIS DP:

```cpp
vector<int> dp1(n, 1);                       // every element alone has length 1

for (int i = 0; i < n; i++) {
    for (int j = 0; j < i; j++) {
        if (arr[i] > arr[j] && 1 + dp1[j] > dp1[i]) {
            dp1[i] = dp1[j] + 1;
        }
    }
}
```

`arr[j]` is a smaller element **before** `i`, so it can sit just before `arr[i]` in an increasing run. This pass moves **left to right** because `dp1[i]` depends on `dp1[j]` with `j < i`.

---

## 4. `dp2`: Longest Decreasing Subsequence Starting at `i`

```
dp2[i] = length of the longest strictly decreasing subsequence starting at index i
```

This is the mirror image. A decreasing run that **starts** at `i` continues to the **right** with smaller values:

```cpp
vector<int> dp2(n, 1);

for (int i = n - 1; i >= 0; i--) {
    for (int j = n - 1; j > i; j--) {
        if (arr[i] > arr[j] && 1 + dp2[j] > dp2[i]) {
            dp2[i] = dp2[j] + 1;
        }
    }
}
```

- `arr[j]` is a smaller element **after** `i`, so it can follow `arr[i]` in a decreasing run.
- The outer loop goes **right to left** (`i = n-1 → 0`) because `dp2[i]` depends on `dp2[j]` with `j > i`. By the time we process `i`, every `dp2[j]` with `j > i` is already final.
- The inner loop order (`j` descending or ascending) does not matter, since we take a maximum over all `j > i`.

### An equivalent way to think about it

`dp2[i]` is the same as the **LIS ending at position `n-1-i` of the reversed array**. Reading a decreasing run backwards gives an increasing run. So `dp2` is "run the LIS DP on the reversed array".

---

## 5. Combining: Why `dp1[i] + dp2[i] - 1`

```
peak at i:   [ ... increasing ... , arr[i] ]   +   [ arr[i] , ... decreasing ... ]
                 dp1[i] elements                      dp2[i] elements
```

`arr[i]` appears **in both**: as the last element of the increasing part (counted in `dp1[i]`) and as the first element of the decreasing part (counted in `dp2[i]`). Adding the two lengths counts it **twice**, so subtract 1.

```
answer = max over i of ( dp1[i] + dp2[i] - 1 )
```

### Why this is optimal

Any bitonic subsequence has some peak `i`. Its left part is an increasing subsequence ending at `i`, which has length `≤ dp1[i]`. Its right part is a decreasing subsequence starting at `i`, which has length `≤ dp2[i]`. So its total length is at most `dp1[i] + dp2[i] - 1`. And this bound is **achievable**, because the best left part and the best right part do not interfere with each other (left uses indices `< i`, right uses indices `> i`). Taking the maximum over all peaks gives the exact answer.

---

## 6. Code

```cpp
int longestBitonicSubsequence(vector<int>& arr, int n)
{
    vector<int> dp1(n, 1);      // LIS ending at i
    vector<int> dp2(n, 1);      // LDS starting at i

    // Pass 1 (left → right): longest increasing subsequence ending at i
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[i] > arr[j] && 1 + dp1[j] > dp1[i]) {
                dp1[i] = dp1[j] + 1;
            }
        }
    }

    int maxi = 0;

    // Pass 2 (right → left): longest decreasing subsequence starting at i,
    // and combine with dp1 right away (dp1 is already complete)
    for (int i = n - 1; i >= 0; i--) {
        for (int j = n - 1; j > i; j--) {
            if (arr[i] > arr[j] && 1 + dp2[j] > dp2[i]) {
                dp2[i] = dp2[j] + 1;
            }
        }
        maxi = max(maxi, (dp1[i] + dp2[i]) - 1);      // peak counted once
    }

    return maxi;
}
```

**Why the combine step can live inside the second loop:** `dp1` was fully computed in the first pass, and `dp2[i]` becomes final the moment the inner loop for `i` finishes. So `dp1[i] + dp2[i] - 1` is available right there, and no third pass is needed.

---

## 7. Dry Run: `arr = [1, 11, 2, 10, 4, 5, 2, 1]`

### Pass 1: `dp1` (increasing, ending at `i`)

```
i=0  (1):   no earlier smaller element                              dp1 = 1
i=1  (11):  1 < 11 → 1 + dp1[0] = 2                                 dp1 = 2
i=2  (2):   1 < 2  → 2                                              dp1 = 2
i=3  (10):  2 < 10 → 1 + dp1[2] = 3                                 dp1 = 3
i=4  (4):   2 < 4  → 3                                              dp1 = 3
i=5  (5):   4 < 5  → 1 + dp1[4] = 4                                 dp1 = 4
i=6  (2):   1 < 2  → 2                                              dp1 = 2
i=7  (1):   nothing smaller before it                               dp1 = 1
```

### Pass 2: `dp2` (decreasing, starting at `i`)

```
i=7  (1):   nothing after                                           dp2 = 1
i=6  (2):   2 > 1 → 1 + dp2[7] = 2                                  dp2 = 2
i=5  (5):   5 > 2 → 1 + dp2[6] = 3                                  dp2 = 3
i=4  (4):   4 > 2 → 1 + dp2[6] = 3                                  dp2 = 3   (4 > 5? no)
i=3  (10):  10 > 5 → 1 + dp2[5] = 4                                 dp2 = 4
i=2  (2):   2 > 1 → 1 + dp2[7] = 2                                  dp2 = 2
i=1  (11):  11 > 10 → 1 + dp2[3] = 5                                dp2 = 5
i=0  (1):   nothing smaller after it                                dp2 = 1
```

### Combine

```
idx      0    1    2    3    4    5    6    7
arr      1   11    2   10    4    5    2    1
dp1      1    2    2    3    3    4    2    1
dp2      1    5    2    4    3    3    2    1
total    1    6    3    6    5    6    3    1      (dp1 + dp2 − 1)

answer = max = 6 ✅
```

Three different peaks reach 6:

```
peak 11 (idx 1):  [1, 11]  +  [11, 10, 5, 2, 1]  →  1, 11, 10, 5, 2, 1
peak 10 (idx 3):  [1, 2, 10] + [10, 5, 2, 1]     →  1, 2, 10, 5, 2, 1
peak  5 (idx 5):  [1, 2, 4, 5] + [5, 2, 1]       →  1, 2, 4, 5, 2, 1
```

### Second example: `[12, 4, 78, 90, 45, 23]`

```
idx     0    1    2    3    4    5
arr    12    4   78   90   45   23
dp1     1    1    2    3    2    2
dp2     2    1    3    3    2    1
total   2    1    4    5    3    2

answer = 5   ([4, 78, 90, 45, 23]) ✅
```

---

## 8. Variants to Watch For

### Strictness

Both comparisons are strict (`arr[i] > arr[j]`), so equal neighbors cannot appear within the same run. For `[3, 3, 3]` the answer is `1`.

### Do purely increasing or purely decreasing sequences count?

With the code above, **yes**: a peak at the last index has `dp2 = 1` (empty decreasing part), and a peak at the first index has `dp1 = 1`. That matches problem statements that treat monotone sequences as bitonic.

If the problem wants a real **mountain** (both sides must be non-empty, as in LeetCode 1671, "Minimum Number of Removals to Make Mountain Array"), only count peaks that have something on **both** sides:

```cpp
if (dp1[i] > 1 && dp2[i] > 1)
    maxi = max(maxi, dp1[i] + dp2[i] - 1);
```

(For LeetCode 1671, the answer is then `n - maxi`.)

---

## 9. Bonus: `O(n log n)` Using the Binary Search LIS

Each of `dp1` and `dp2` is just an LIS computation, so each can use the binary search method from the earlier notes. When processing `arr[i]` and finding its position `p = lower_bound(tails, arr[i])`, the length of the longest increasing subsequence **ending at `i`** is `p + 1`:

```cpp
vector<int> lisEnding(const vector<int>& a) {
    vector<int> tails, res(a.size());
    for (int i = 0; i < (int)a.size(); i++) {
        int p = lower_bound(tails.begin(), tails.end(), a[i]) - tails.begin();
        if (p == (int)tails.size()) tails.push_back(a[i]);
        else                        tails[p] = a[i];
        res[i] = p + 1;                              // LIS length ending at i
    }
    return res;
}

int longestBitonicSubsequence(vector<int>& arr, int n) {
    vector<int> dp1 = lisEnding(arr);

    vector<int> rev(arr.rbegin(), arr.rend());
    vector<int> r = lisEnding(rev);                  // LIS ending at position n-1-i of rev
    // dp2[i] = r[n - 1 - i]

    int maxi = 0;
    for (int i = 0; i < n; i++)
        maxi = max(maxi, dp1[i] + r[n - 1 - i] - 1);

    return maxi;
}
```

This is the "reverse the array" view of `dp2` from section 4.

---

## 10. Complexity Analysis

| Version | Time | Space |
|---|---|---|
| **`dp1` / `dp2` (this code)** | **`O(n²)`** | **`O(n)`** (two arrays) |
| Binary search version | `O(n log n)` | `O(n)` |

---

## 11. Where This Fits in the LIS Family

| Problem | Idea |
|---|---|
| LIS | `dp[i]` = longest increasing subsequence ending at `i` |
| Print LIS | `dp[i]` + parent array |
| LIS with binary search | tails array |
| Longest String Chain | sort by length, then LIS-style DP |
| **Longest Bitonic Subsequence** | **LIS from the left + LDS from the right, combined at each peak: `dp1[i] + dp2[i] - 1`** |
| Min removals to make a mountain array | `n -` (longest mountain, with both sides non-empty) |

The combining trick (a left DP and a right DP merged at every index) shows up again in problems like "trapping rain water" and "maximum product / sum around a split point".
