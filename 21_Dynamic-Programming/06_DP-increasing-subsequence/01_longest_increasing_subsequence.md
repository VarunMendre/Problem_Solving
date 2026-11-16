# Longest Increasing Subsequence (LIS)

Problem link: [LeetCode 300: Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

---

## 1. Problem Statement

Given an integer array `nums`, return the length of the **longest strictly increasing subsequence**.

A subsequence keeps the original order but may skip elements (it does not have to be contiguous).

```
nums = [10, 9, 2, 5, 3, 7, 101, 18]

One LIS: [2, 3, 7, 101]  (also [2, 5, 7, 18], [2, 3, 7, 18], ...)

Answer: 4
```

```
nums = [0, 1, 0, 3, 2, 3]   → 4   ([0, 1, 2, 3])
nums = [7, 7, 7, 7]         → 1   (strictly increasing: equal values don't count)
```

---

## 2. Thought Process: Pick / Not-Pick With a Memory of the Last Pick

LIS is a **subsequence** problem, so the usual template applies: at every index, either **take** the element or **skip** it.

```
take    → length grows by 1
notTake → length stays the same
```

But there is a catch that earlier subsequence problems (Subset Sum, Knapsack) did not have. **Whether we are allowed to take `nums[ind]` depends on the last element we took:** it must be **larger** than that element.

So the state needs to remember the last chosen element:

```
f(ind, prev) = length of the longest increasing subsequence obtainable from
               nums[ind .. n-1], given that the last element picked so far is nums[prev]
```

- `ind` is the current index we are deciding about.
- `prev` is the **index** of the last picked element. `prev = -1` means **nothing has been picked yet**, so any element is allowed.

### Transitions

```
notTake = 0 + f(ind + 1, prev)                       // skip nums[ind]; the last pick stays the same

take    = 1 + f(ind + 1, ind)                        // only if prev == -1 || nums[ind] > nums[prev]
                                                     // now nums[ind] becomes the last pick

f(ind, prev) = max(take, notTake)
```

Base case:

```
ind == n  →  return 0        // no elements left
```

---

## 3. Approach 1: Recursion

```cpp
class Solution {
private:
    int f(int curInd, int prevInd, vector<int>& nums) {
        if (curInd == nums.size())
            return 0;

        int notTake = 0 + f(curInd + 1, prevInd, nums);
        int take = 0;

        if (prevInd == -1 || nums[curInd] > nums[prevInd])
            take = 1 + f(curInd + 1, curInd, nums);

        return max(take, notTake);
    }

public:
    int lengthOfLIS(vector<int>& nums) {
        return f(0, -1, nums);
    }
};
```

### Dry run: `nums = [1, 3, 2]` (answer 2)

```
f(0,-1):  nothing picked yet
  take 1  → 1 + f(1, 0)
     f(1,0): nums[1]=3 > nums[0]=1 → can take
        take 3  → 1 + f(2, 1)
           f(2,1): nums[2]=2 > nums[1]=3? NO → take = 0; notTake = f(3,1) = 0   → 0
           = 1
        notTake → f(2, 0)
           f(2,0): nums[2]=2 > nums[0]=1 → take = 1 + f(3,2) = 1;  notTake = 0 → 1
        f(1,0) = max(1, 1) = 1
     take total = 1 + 1 = 2

  notTake → f(1,-1)  = 1   (picking a single element at best)

f(0,-1) = max(2, 1) = 2 ✅     (e.g. [1,3] or [1,2])
```

### Story points

- **Strictly increasing:** the condition is `>`, not `>=`. For a non-decreasing subsequence, change it to `>=`.
- **`prev = -1` is a sentinel**, not a real index. It means "no previous element", and the `prevInd == -1 ||` check short-circuits **before** `nums[prevInd]` is read, so there is no out-of-bounds access.
- **Why `take` moves `prev` to `ind` but `notTake` keeps `prev`:** skipping an element does not change what the last picked element was.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Two choices per index |
| **Space** | `O(n)` | Recursion depth |

---

## 4. Approach 2: Memoization

The state `(ind, prev)` repeats across branches, and there are only `n × (n + 1)` distinct pairs.

### The coordinate shift: `prev + 1`

`prev` ranges over `-1, 0, 1, ..., n-1`. An array cannot have index `-1`, so we **shift** it by one and store at `dp[ind][prev + 1]`:

```
prev = -1  →  column 0
prev =  0  →  column 1
...
prev = n-1 →  column n
```

That is why the table is `n × (n + 1)`: `n` rows for `ind`, and `n + 1` columns for `prev + 1`.

```cpp
class Solution {
private:
    int f(int ind, int prev, vector<int>& nums, vector<vector<int>>& dp) {
        if (ind == nums.size())
            return 0;

        if (dp[ind][prev + 1] != -1)
            return dp[ind][prev + 1];

        int notTake = 0 + f(ind + 1, prev, nums, dp);
        int take = 0;

        if (prev == -1 || nums[ind] > nums[prev])
            take = 1 + f(ind + 1, ind, nums, dp);

        return dp[ind][prev + 1] = max(take, notTake);
    }

public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> dp(n, vector<int>(n + 1, -1));
        return f(0, -1, nums, dp);
    }
};
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n²)` (`n × (n+1)` states, `O(1)` each) |
| **Space** | `O(n²)` dp + `O(n)` stack |

---

## 5. Approach 3: Tabulation

`f(ind, ·)` depends only on `f(ind + 1, ·)`, so we fill from the last index backwards.

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();

        vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));   // dp[n][*] = 0 is the base case

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int prev = ind - 1; prev >= -1; prev--) {

                int notTake = dp[ind + 1][prev + 1];
                int take = 0;

                if (prev == -1 || nums[ind] > nums[prev])
                    take = 1 + dp[ind + 1][ind + 1];

                dp[ind][prev + 1] = max(take, notTake);
            }
        }

        return dp[0][0];
    }
};
```

### Why does `prev` loop from `ind - 1` down to `-1`?

The last picked index must always come **before** the current index (`prev < ind`). So the only meaningful `prev` values at row `ind` are `-1, 0, ..., ind - 1`. Values `prev ≥ ind` can never occur, and the loop skips them.

### How the base case and the answer are stored

- `dp[n][*] = 0` (the `ind == n` base case) comes from the zero initialization.
- The answer is `dp[0][0]`: `ind = 0`, `prev = -1` → column `prev + 1 = 0`.

### How the transitions map to the table

```
notTake = dp[ind + 1][prev + 1]      same last pick, next row
take    = 1 + dp[ind + 1][ind + 1]   last pick becomes `ind` → column ind + 1 in the next row
```

### Table for `nums = [1, 3, 2, 4]` (answer 3)

```
                 column (prev + 1):   0    1    2    3    4
                      prev value:    -1    0    1    2    3
ind = 4 (base)                        0    0    0    0    0
ind = 3  (nums = 4)                   1    1    1    1    ·
ind = 2  (nums = 2)                   2    2    1    ·    ·
ind = 1  (nums = 3)                   2    2    ·    ·    ·
ind = 0  (nums = 1)                   3    ·    ·    ·    ·

Answer = dp[0][0] = 3 ✅     ([1, 3, 4] or [1, 2, 4])
```

(`·` marks cells with `prev ≥ ind`, which never occur.)

Spot checks:

```
dp[2][1]  (ind=2 → value 2, prev=0 → value 1):
          2 > 1 → take = 1 + dp[3][3] = 1 + 1 = 2;   notTake = dp[3][1] = 1   → 2

dp[2][2]  (ind=2 → value 2, prev=1 → value 3):
          2 > 3? NO → take = 0;   notTake = dp[3][2] = 1                      → 1

dp[0][0]  (ind=0 → value 1, nothing picked):
          take = 1 + dp[1][1] = 1 + 2 = 3;   notTake = dp[1][0] = 2          → 3
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n²)` |
| **Space** | `O(n²)` |

---

## 6. Approach 4: Space Optimization

Row `ind` reads only row `ind + 1`, so two arrays are enough: `dp` (row `ind + 1`) and `cur` (row `ind`).

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();

        vector<int> dp(n + 1, 0);     // row ind + 1   (starts as the base row of zeros)
        vector<int> cur(n + 1, 0);    // row ind

        for (int ind = n - 1; ind >= 0; ind--) {
            for (int prev = ind - 1; prev >= -1; prev--) {

                int notTake = dp[prev + 1];                 // dp[ind+1][prev+1]
                int take = 0;

                if (prev == -1 || nums[ind] > nums[prev])
                    take = 1 + dp[ind + 1];                 // dp[ind+1][ind+1]

                cur[prev + 1] = max(take, notTake);
            }
            dp = cur;                                        // row ind becomes the "next" row
        }

        return dp[0];                                        // dp[0][0]
    }
};
```

### Why this is safe

- In iteration `ind`, only columns `0 .. ind` of `cur` are written (`prev` from `-1` to `ind - 1`).
- In the next iteration (`ind - 1`), the code reads `dp[prev + 1]` for `prev ≤ ind - 2` (columns `≤ ind - 1`) and `dp[ind]` (column `ind`). All of those columns were written in the iteration just completed.
- Columns beyond `ind` in `cur` may hold stale values from earlier rows, but they are **never read**.

### Trace on `nums = [1, 3, 2, 4]`

```
start:   dp = [0, 0, 0, 0, 0]                              (row ind = 4)

ind=3 (4):  cur[3]=1 (prev=2), cur[2]=1 (prev=1), cur[1]=1 (prev=0), cur[0]=1 (prev=-1)
            dp = [1, 1, 1, 1, 0]

ind=2 (2):  prev=1: 2>3? no → max(0, dp[2]=1) = 1            → cur[2] = 1
            prev=0: 2>1 → max(1 + dp[3]=2, dp[1]=1) = 2      → cur[1] = 2
            prev=-1:      max(1 + dp[3]=2, dp[0]=1) = 2      → cur[0] = 2
            dp = [2, 2, 1, 1, 0]

ind=1 (3):  prev=0: 3>1 → max(1 + dp[2]=2, dp[1]=2) = 2      → cur[1] = 2
            prev=-1:      max(1 + dp[2]=2, dp[0]=2) = 2      → cur[0] = 2
            dp = [2, 2, 1, 1, 0]

ind=0 (1):  prev=-1:      max(1 + dp[1]=3, dp[0]=2) = 3      → cur[0] = 3
            dp[0] = 3

return 3 ✅
```

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(n²)` |
| **Space** | `O(n)` (two arrays of size `n + 1`) |

---

## 7. All Approaches Compared

| Approach | Time | Space |
|---|---|---|
| Recursion | `O(2ⁿ)` | `O(n)` |
| Memoization | `O(n²)` | `O(n²)` + stack |
| Tabulation | `O(n²)` | `O(n²)` |
| Space optimized | `O(n²)` | `O(n)` |

---

## 8. Where This Leads: Better LIS Formulations

The `(ind, prev)` DP is the direct "pick / not-pick" translation. Two other formulations are very common.

### 8.1 `dp[i]` = length of the LIS **ending at** index `i` (`O(n²)` time, `O(n)` space)

```cpp
vector<int> dp(n, 1);
int ans = 1;
for (int i = 0; i < n; i++) {
    for (int j = 0; j < i; j++)
        if (nums[j] < nums[i])
            dp[i] = max(dp[i], 1 + dp[j]);
    ans = max(ans, dp[i]);
}
return ans;
```

It changes the state from "last pick is `prev`" to "the subsequence **ends** at `i`", which removes one dimension from the start. It is also the form you need to **print** an LIS (store a parent index for each `i`).

### 8.2 Binary search on "tails" (`O(n log n)` time, `O(n)` space)

```cpp
vector<int> tails;
for (int x : nums) {
    auto it = lower_bound(tails.begin(), tails.end(), x);
    if (it == tails.end()) tails.push_back(x);
    else *it = x;
}
return tails.size();
```

`tails[len - 1]` holds the **smallest possible last element** of any increasing subsequence of length `len`. For each new number, find where it fits and either extend or improve a length. (`lower_bound` gives a strict LIS; the array `tails` itself is not necessarily an LIS, only its length is the answer.)

---

## 9. Link to LCS

```
LIS(nums) = LCS(nums, sorted unique values of nums)
```

A strictly increasing subsequence of `nums` is exactly a common subsequence of `nums` and its sorted, de-duplicated version. This is the same trick as LPS = LCS(s, reverse(s)): reduce a new problem to LCS by choosing the second string cleverly. It runs in `O(n²)`, the same as the DP above.

---

## 10. Subsequence DP Family

| Problem | State | Constraint on "take" |
|---|---|---|
| Subset Sum | `(ind, remaining sum)` | element ≤ remaining sum |
| 0/1 Knapsack | `(ind, capacity)` | weight ≤ capacity |
| **LIS** | **`(ind, last picked index)`** | **`nums[ind] > nums[prev]`** |
| LCS | `(i, j)` | `s1[i] == s2[j]` |
| Distinct Subsequences | `(i, j)` | `s[i] == t[j]` (add both choices) |

LIS is the first problem in this series where the "can I take it?" test depends on **what was picked earlier**, which is why the previous pick has to be part of the state.
