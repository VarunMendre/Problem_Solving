# Target Sum

---

## 1. Problem Statement

Given an integer array `nums` and an integer `target`, assign either `+` or `-` to each element. Return the number of ways to assign signs such that the resulting expression evaluates to `target`.

```
nums = [1, 1, 1, 1, 1], target = 3

+1+1+1+1-1 = 3 ✅
+1+1+1-1+1 = 3 ✅
+1+1-1+1+1 = 3 ✅
+1-1+1+1+1 = 3 ✅
-1+1+1+1+1 = 3 ✅

Answer: 5
```

---

## 2. The Key Reduction — This Is "Partitions with Given Difference"

### How to Think About It

Every element gets a `+` or `-` sign. This naturally splits `nums` into two subsets:
- **S1** = elements assigned `+` (positive subset)
- **S2** = elements assigned `-` (negative subset)

Every element belongs to exactly one of S1 or S2.

The expression evaluates to:
```
S1 - S2 = target
```

And since every element is in either S1 or S2:
```
S1 + S2 = totalSum
```

This is **exactly** the "Partitions with Given Difference" problem with `diff = target`!

---

### The Algebra (Quick Reference)

> For the full step-by-step derivation of how `S1 - S2 = D` and `S1 + S2 = T` leads to `S2 = (T - D) / 2`, refer to the Partitions with Given Difference notes: [partitions_with_given_diff.md](https://github.com/VarunMendre/Problem_Solving/blob/main/21_Dynamic-Programming/03_DP-subsequences/05_partitions_with_given_diff.md)

**Bottom line:** we need to count subsets with sum:
```
S2 = (totalSum - target) / 2
```

---

### Concrete Example

```
nums = [1, 1, 1, 1, 1], target = 3

totalSum = 5
S2 = (5 - 3) / 2 = 1

We need: how many subsets of nums sum to 1?
  {1} (index 0) ✅
  {1} (index 1) ✅
  {1} (index 2) ✅
  {1} (index 3) ✅
  {1} (index 4) ✅

5 subsets → 5 ways ✅

For each such S2={one element with value 1}:
  S1 = remaining 4 elements = sum 4
  S1 - S2 = 4 - 1 = 3 = target ✅
```

---

### The Two Early-Exit Checks

**Check 1 — `target > totalSum`:** The maximum achievable by assigning all `+` is `totalSum`. If `target > totalSum`, impossible.

**Check 2 — `(totalSum - target)` is odd:** `S2 = (totalSum - target) / 2` must be an integer. If odd, no valid partition exists → return 0.

---

## 3. Code

```cpp
class Solution {
private:
    int countPartitions(vector<int>& arr, int diff) {
        int n = arr.size();
        int totalSum = accumulate(arr.begin(), arr.end(), 0);

        if(diff > totalSum) return 0;

        int remain = totalSum - diff;
        if(remain % 2 != 0) return 0;

        int target = remain / 2;   // S2 = (totalSum - diff) / 2

        // Count subsets with sum = target
        vector<int> prev(target+1, 0);
        prev[target] = 1;   // base: achieved target → 1 way

        for(int ind = n-1; ind >= 0; ind--) {
            vector<int> cur(target+1, 0);

            for(int sum = 0; sum <= target; sum++) {
                int notTake = prev[sum];
                int take    = (sum + arr[ind] <= target) ? prev[sum + arr[ind]] : 0;
                cur[sum]    = take + notTake;
            }

            prev = cur;
        }

        return prev[0];
    }

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        return countPartitions(nums, target);
    }
};
```

---

## 4. The Mapping in One Line

```
findTargetSumWays(nums, target)
        ↓
countPartitions(nums, diff=target)
        ↓
countSubsets(nums, sum = (totalSum - target) / 2)
```

The `findTargetSumWays` → `countPartitions` call is the entire problem. The wrapper just renames `target` to `diff` and passes through.

---

## 5. Complexity

### Time — `O(n × target)`

| Step | Cost |
|---|---|
| Compute totalSum | `O(n)` |
| DP loops | `O(n × (totalSum - target)/2)` |

**Total: `O(n × totalSum)`** in the worst case.

### Space — `O(target)` = `O((totalSum - target)/2)`

Two 1D arrays of size `target+1`.

---

## 6. The Full Subset DP Family

| Problem | Derivation | DP type | Answer |
|---|---|---|---|
| Subset Sum | Given target directly | `\|\|` | `dp[n][target]` |
| Count Subsets Sum K | Given target directly | `+` | count |
| Partition Equal Subset | `target = totalSum/2` | `\|\|` | `dp[n][target]` |
| Min Subset Difference | scan `dp[0..S/2]` | `\|\|` then scan | `min(S-2*s1)` |
| Partitions with Diff D | `target = (T-D)/2` | `+` | count |
| **Target Sum** | `target = (T-diff)/2` | `+` | count (same as above!) |

Target Sum and Partitions with Given Difference are **the same problem** — one assigns `+`/`-` signs, the other splits into two subsets. The underlying math is identical.
