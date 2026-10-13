# Partition Equal Subset Sum

---

## 1. The Problem-Reduction Thinking — Read This First

### What the Problem Actually Asks

Can we split the array into **two subsets with equal sum**?

```text
nums = [1, 5, 11, 5]
Split: {1, 5, 5} = 11 and {11} = 11
Answer: true

nums = [1, 2, 3, 5]
No way to split into two equal halves.
Answer: false
```

### Step 1 — The Odd Sum Elimination

Let `S = totalSum = sum of all elements`.

If we split into two subsets S1 and S2:

```text
S1 + S2 = S
S1 = S2

→ 2 × S1 = S
→ S1 = S/2
```

**If `S` is odd, return `false` immediately.**

```text
nums = [1, 2, 3, 5]
S = 11 (odd)
Answer: false
```

### Step 2 — What We Need

If `S` is even, we need:

```text
S1 = S2 = S/2
```

We only need to find one subset that sums to `S/2`. The remaining elements will automatically form the other subset with the same sum.

This reduces the problem to **Subset Sum**.

```text
nums = [1, 5, 11, 5]
S = 22
target = 11

Subset {11} sums to 11.
Answer: true
```

### The Complete Reduction

```text
Partition Equal Subset Sum
        ↓
Calculate totalSum
        ↓
If totalSum is odd → false
        ↓
target = totalSum / 2
        ↓
Subset Sum using pick/not-pick DP
```

---

## 2. Problem Statement

Given an integer array `nums`, return `true` if it can be partitioned into two subsets with equal sum; otherwise, return `false`.

```text
nums = [1, 5, 11, 5] → true
nums = [1, 2, 3, 5]  → false
nums = [1, 2, 5]     → false
```

---

## 3. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    bool findSubsetSum(int ind, int n, int curSum, int target,
                       vector<int>& nums) {
        if (curSum == target)
            return true;

        if (curSum > target || ind >= n)
            return false;

        bool take = findSubsetSum(
            ind + 1, n, curSum + nums[ind], target, nums
        );

        bool notTake = findSubsetSum(
            ind + 1, n, curSum, target, nums
        );

        return take || notTake;
    }

public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();
        int totalSum = accumulate(nums.begin(), nums.end(), 0);

        if (totalSum % 2 != 0)
            return false;

        int target = totalSum / 2;

        return findSubsetSum(0, n, 0, target, nums);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| Time | `O(2ⁿ)` | Two branches per element |
| Space | `O(n)` | Recursion stack |

---

## 4. Approach 2 — Memoization

```cpp
class Solution {
private:
    bool findSubsetSum(int ind, int n, int curSum, int target,
                       vector<vector<int>>& dp, vector<int>& nums) {
        if (curSum == target)
            return true;

        if (curSum > target || ind >= n)
            return false;

        if (dp[ind][curSum] != -1)
            return dp[ind][curSum];

        bool take = findSubsetSum(
            ind + 1, n, curSum + nums[ind], target, dp, nums
        );

        bool notTake = findSubsetSum(
            ind + 1, n, curSum, target, dp, nums
        );

        return dp[ind][curSum] = take || notTake;
    }

public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();
        int totalSum = accumulate(nums.begin(), nums.end(), 0);

        if (totalSum % 2 != 0)
            return false;

        int target = totalSum / 2;

        vector<vector<int>> dp(
            n, vector<int>(target + 1, -1)
        );

        return findSubsetSum(0, n, 0, target, dp, nums);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| Time | `O(n × target)` | Each state is computed once |
| Space | `O(n × target) + O(n)` | DP table and recursion stack |

---

## 5. Approach 3 — Tabulation

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();

        int totalSum = accumulate(nums.begin(), nums.end(), 0);

        if (totalSum % 2 != 0)
            return false;

        int target = totalSum / 2;

        // dp[i][s] = can we form sum s using the first i elements?
        vector<vector<bool>> dp(
            n + 1, vector<bool>(target + 1, false)
        );

        // Sum 0 is possible using an empty subset.
        for (int i = 0; i <= n; i++)
            dp[i][0] = true;

        for (int ind = 1; ind <= n; ind++) {
            for (int sum = 1; sum <= target; sum++) {
                bool notTake = dp[ind - 1][sum];
                bool take = false;

                if (sum >= nums[ind - 1])
                    take = dp[ind - 1][sum - nums[ind - 1]];

                dp[ind][sum] = take || notTake;
            }
        }

        return dp[n][target];
    }
};
```

### Tabulation Trace for `nums = [1, 5, 11, 5]`

Here, `target = 11`.

```text
dp[i][s] = Can the first i elements form sum s?

Initial:
dp[0][0] = true
All other sums in row 0 = false

After processing 1:
Possible sums: 0, 1

After processing 5:
Possible sums: 0, 1, 5, 6

After processing 11:
Possible sums: 0, 1, 5, 6, 11

After processing the final 5:
Sum 11 remains possible.

dp[4][11] = true
Answer: true
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| Time | `O(n × target)` | Two nested loops |
| Space | `O(n × target)` | Two-dimensional DP table |

---

## 6. Approach 4 — Space Optimization

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();

        int totalSum = accumulate(nums.begin(), nums.end(), 0);

        if (totalSum % 2 != 0)
            return false;

        int target = totalSum / 2;

        vector<bool> dp(target + 1, false);
        dp[0] = true;

        for (int ind = 0; ind < n; ind++) {
            // Traverse backwards to avoid reusing the same element.
            for (int sum = target; sum >= nums[ind]; sum--) {
                dp[sum] = dp[sum] || dp[sum - nums[ind]];
            }
        }

        return dp[target];
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| Time | `O(n × target)` | Same DP transitions |
| Space | `O(target)` | Single one-dimensional DP array |

---

## 7. All Approaches Compared

| Approach | Time | Auxiliary Space |
|---|---|---|
| Recursion | `O(2ⁿ)` | `O(n)` |
| Memoization | `O(n × S/2)` | `O(n × S/2) + O(n)` |
| Tabulation | `O(n × S/2)` | `O(n × S/2)` |
| Space Optimized | `O(n × S/2)` | `O(S/2)` |

Here, `S` is the total sum of all elements.

---

## 8. Subset Sum vs Partition Equal Subset Sum

| | Subset Sum | Partition Equal Subset Sum |
|---|---|---|
| Given | Array and target | Array |
| Target | Provided directly | `totalSum / 2` |
| Extra check | None | Odd total sum means `false` |
| DP | Pick/not-pick | Pick/not-pick |
| Objective | Find a subset with sum K | Find a subset with half the total sum |

The main reduction is:

```cpp
int totalSum = accumulate(nums.begin(), nums.end(), 0);

if (totalSum % 2 != 0)
    return false;

int target = totalSum / 2;
```

---

## 9. The Subset DP Family

| Problem | DP operation | Objective |
|---|---|---|
| Subset Sum | Logical OR | Determine whether a subset with sum K exists |
| Partition Equal Subset Sum | Logical OR | Determine whether two equal-sum subsets exist |
| Count Subsets with Sum K | Addition | Count subsets with sum K |
| 0/1 Knapsack | Maximum | Maximize value within a capacity |
| Minimum Subset Difference | Minimum difference | Minimize the difference between subset sums |

The central pattern is the same: **decide whether to take or skip each element**. The state, transition, and final result depend on what the problem asks you to calculate.