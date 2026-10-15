# Count Subsets with Sum K (Perfect Sum Problem)

---

## 1. Problem Statement

Given an array `arr[]` and a target, count the **number of subsets** whose elements sum to exactly `target`.

```
arr = [1, 1, 2, 3], target = 3

Subsets summing to 3:
  {1, 2} (using index 0, 2) = 3
  {1, 2} (using index 1, 2) = 3  ← different subset, same values
  {3}    (using index 3)    = 3

Count = 3
```

```
arr = [2, 3, 5, 6, 8, 10], target = 10

Subsets:
  {2, 8} = 10
  {2, 3, 5} = 10
  {10} = 10

Count = 3
```

---

## 2. From Subset Sum (Existence) to Counting

Subset Sum answers: **does** a subset exist? (`true`/`false`)
Count Subsets answers: **how many** subsets exist? (integer)

The ONLY change: replace the aggregation operator.

```
Subset Sum:     dp[i][s] = dp[i-1][s]  OR  dp[i-1][s-arr[i-1]]
                           ↑ boolean OR ↑

Count Subsets:  dp[i][s] = dp[i-1][s]  +   dp[i-1][s-arr[i-1]]
                           ↑ integer ADD ↑
```

**`||` becomes `+`.** The dp table structure, base cases, traversal order — all identical.

---

## 3. Traversal Direction: Why This Code Goes Bottom-Up in Recursion

The classic Subset Sum code we wrote earlier traversed **top-down** (from index 0 → n-1, building curSum upward toward target). This code uses the **other direction**: starts at index `n-1` and tracks `curSum` from `sum` → `target`.

**Why the different direction?**

The tabulation base case here:
```cpp
dp[n][target] = 1    // used all n items, curSum equals target → 1 way
dp[n][sum] = 0       // used all n items, curSum ≠ target → 0 ways
```

The recurrence reads `dp[ind+1]` to fill `dp[ind]` — iterating from `ind = n-1` downward to `0`. This is perfectly valid; it's just a different "direction" of the same pick/not-pick DP.

Both approaches are equivalent. The memoization here tracks `(ind, curSum)` where `curSum` is partial sum built so far, which travels in the "forward" direction (ind 0→n, curSum 0→target).

---

## 4. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int ind, int curSum, int count, int target, int n, vector<int>& arr) {
        if(ind >= n)
            return (curSum == target) ? ++count : count;   // count is unused here

        int take    = fn(ind+1, curSum + arr[ind], count, target, n, arr);
        int notTake = fn(ind+1, curSum,             count, target, n, arr);

        return take + notTake;
    }

public:
    int perfectSum(vector<int>& arr, int target) {
        int n = arr.size();
        return fn(0, 0, 0, target, n, arr);
    }
};
```

### The `count` Parameter — Redundant

The `count` parameter in `fn` is carried but never meaningfully used. The base case:
```cpp
return (curSum == target) ? ++count : count;
```

`count` is always `0` when passed in (never modified before the base case check). So this simplifies to:
```cpp
return (curSum == target) ? 1 : 0;
```

The `take + notTake` return aggregates counts from all branches — this is the correct counting logic. The `count` parameter is dead weight.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Two branches per element |
| **Space** | `O(n)` | Recursion stack |

---

## 5. Approach 2 — Memoization

```cpp
class Solution {
private:
    int fn(int ind, int curSum, int target, int n,
           vector<vector<int>>& dp, vector<int>& arr) {
        if(ind >= n)
            return (curSum == target) ? 1 : 0;

        if(curSum > target) return 0;   // pruning: over-budget

        if(dp[ind][curSum] != -1)
            return dp[ind][curSum];

        int take    = fn(ind+1, curSum + arr[ind], target, n, dp, arr);
        int notTake = fn(ind+1, curSum,             target, n, dp, arr);

        return dp[ind][curSum] = take + notTake;   // COUNT: sum, not OR
    }

public:
    int perfectSum(vector<int>& arr, int target) {
        int n = arr.size();
        vector<vector<int>> dp(n, vector<int>(target+1, -1));
        return fn(0, 0, target, n, dp, arr);
    }
};
```

### Key Change from Subset Sum

```cpp
// Subset Sum:
return dp[ind][curSum] = take || notTake;

// Count Subsets:
return dp[ind][curSum] = take + notTake;
```

`||` counts existence (0 or 1), `+` counts ALL paths.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × target)` | n × target states, O(1) each |
| **Space** | `O(n × target) + O(n)` | dp + stack |

---

## 6. Approach 3 — Tabulation

This uses the **reverse direction** — start from the last index, track `sum` from 0 upward:

```cpp
class Solution {
public:
    int perfectSum(vector<int>& arr, int target) {
        int n = arr.size();
        vector<vector<int>> dp(n+1, vector<int>(target+1, 0));

        // Base case: used all n elements, achieved target → 1 way
        dp[n][target] = 1;
        // dp[n][sum≠target] = 0 (initialized to 0)

        for(int ind = n-1; ind >= 0; ind--) {
            for(int sum = 0; sum <= target; sum++) {
                int notTake = dp[ind+1][sum];               // skip arr[ind]

                int take = 0;
                if(sum + arr[ind] <= target)
                    take = dp[ind+1][sum + arr[ind]];       // include arr[ind]

                dp[ind][sum] = take + notTake;
            }
        }

        return dp[0][0];   // start with 0 items used, curSum=0
    }
};
```

### The Reverse Direction Explained

**Classic direction (Subset Sum):** `dp[i][s]` = "can we form sum `s` using first `i` elements?"
- Read `dp[i-1][s]` and `dp[i-1][s - arr[i-1]]`
- Fill from `i=1` to `n`

**This direction:** `dp[ind][sum]` = "how many subsets summing to `target` can be formed using elements from index `ind` onward, if we've already accumulated `sum`?"
- Read `dp[ind+1][sum]` and `dp[ind+1][sum + arr[ind]]`
- Fill from `ind = n-1` down to `0`

Both are valid formulations. The base cases flip:
```
Classic:  dp[0][i]=true for all i (0 items, achieves sum 0)
          dp[n][target] is the answer

Reverse:  dp[n][target]=1, dp[n][others]=0
          dp[0][0] is the answer
```

### Tabulation Trace (arr=[1,1,2], target=3)

```
n=3, target=3
dp[3][3]=1, all others 0

ind=2 (arr[2]=2):
  sum=0: take=dp[3][2]=0, notTake=dp[3][0]=0 → 0
  sum=1: take=dp[3][3]=1, notTake=dp[3][1]=0 → 1
  sum=2: take(2+2=4>3→0), notTake=dp[3][2]=0 → 0
  sum=3: take(3+2=5>3→0), notTake=dp[3][3]=1 → 1

dp[2] = [0, 1, 0, 1]

ind=1 (arr[1]=1):
  sum=0: take=dp[2][1]=1, notTake=dp[2][0]=0 → 1
  sum=1: take=dp[2][2]=0, notTake=dp[2][1]=1 → 1
  sum=2: take=dp[2][3]=1, notTake=dp[2][2]=0 → 1
  sum=3: take(3+1=4>3→0), notTake=dp[2][3]=1 → 1

dp[1] = [1, 1, 1, 1]

ind=0 (arr[0]=1):
  sum=0: take=dp[1][1]=1, notTake=dp[1][0]=1 → 2
  sum=1: take=dp[1][2]=1, notTake=dp[1][1]=1 → 2
  sum=2: take=dp[1][3]=1, notTake=dp[1][2]=1 → 2
  sum=3: take(3+1=4>3→0), notTake=dp[1][3]=1 → 1

dp[0] = [2, 2, 2, 1]

return dp[0][0] = 2... 

Wait — arr=[1,1,2], target=3
Subsets: {1(ind0),2}=3 ✅, {1(ind1),2}=3 ✅
Count = 2... but we have 3 from example?

Let me re-check with arr=[1,1,2,3]:
No — arr=[1,1,2], target=3, count=2 ✅ (only {1,2} with first 1, and {1,2} with second 1)
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × target)` | Two nested loops |
| **Space** | `O(n × target)` | 2D dp array |

---

## 7. Approach 4 — Space Optimization

```cpp
class Solution {
public:
    int perfectSum(vector<int>& arr, int target) {
        int n = arr.size();

        vector<int> prev(target+1, 0);
        prev[target] = 1;   // base: achieved target with 0 more elements needed

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
};
```

### Why Return `prev[0]` and Not `prev[target]`?

We start from `ind=n-1` going backward to `ind=0`. At `ind=0`, `sum=0` means "we've accumulated 0 so far and are at the first element." `prev[0]` = number of subsets from element 0 onward that sum to `target` when starting with accumulated sum 0. That's our answer.

In the classic direction, `prev[target]` would be the answer (we built up to `target`). In this direction, `prev[0]` is the answer (we start from 0 and track how many ways reach `target`).

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × target)` | Same loops |
| **Space** | `O(2 × target)` = `O(target)` | Two 1D arrays |

---

## 8. All Approaches Compared

| Approach | Time | Space | Direction | Answer cell |
|---|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` | Forward (0→n) | return value |
| **Memoization** | `O(n × K)` | `O(n × K)` | Forward (0→n) | `fn(0,0)` |
| **Tabulation** | `O(n × K)` | `O(n × K)` | Reverse (n→0) | `dp[0][0]` |
| **Space Optimized** | `O(n × K)` | `O(K)` | Reverse (n→0) | `prev[0]` |

---

## 9. Subset Sum Family — Updated

| Problem | Operator | Base case | Answer |
|---|---|---|---|
| Subset Sum (exist?) | `\|\|` | `dp[i][0]=true` | `dp[n][target]` |
| Count Subsets | `+` | `dp[n][target]=1` | `dp[0][0]` or `dp[n][target]` |
| Partition Equal Subset | `\|\|` | same as Subset Sum | `dp[n][S/2]` |
| Min Subset Difference | `\|\|` then scan | same | `min(S-2*s1)` |
| 0/1 Knapsack | `max` | `dp[0][w]=0` | `dp[n][W]` |

The family is unified by one table structure with varying operators and base cases.
