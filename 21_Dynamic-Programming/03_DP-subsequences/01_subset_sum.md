# Subset Sum

---

## 1. Problem Statement

Given an array `arr[]` of non-negative integers and a target `sum`, determine if there exists a **subset** of `arr` whose elements sum to exactly `sum`. Return `true` if such a subset exists, `false` otherwise.

```
arr = [3, 34, 4, 12, 5, 2], sum = 9

Subsets that sum to 9: {3, 4, 2} = 9 ✅, {4, 5} = 9 ✅
Answer: true

arr = [3, 34, 4, 12, 5, 2], sum = 30
No subset sums to 30.
Answer: false
```

---

## 2. The Subset DP Category

This problem opens the **"Subsequence / Subset DP"** category — one of the most important DP families. The core idea:

> For every element, you have exactly two choices: **take it** (include in subset) or **not take it** (skip it). Explore all possibilities via DP.

This pick/not-pick pattern extends to:
- Subset Sum → does a subset summing to `target` exist?
- Count Subsets with Sum K → how many such subsets?
- 0/1 Knapsack → maximize value within weight limit
- Partition Equal Subset Sum → can array be split into two equal halves?

---

## 3. Intuition

Define `f(ind, curSum)` = "can we form `target - curSum` using elements from index `ind` onward?"

Or equivalently: `f(ind, target)` = "can we form exactly `target` using elements from index `0` to `ind`?"

At each index, we either:
- **Take `arr[ind]`:** reduces the remaining target by `arr[ind]`
- **Not take `arr[ind]`:** target stays the same

Base cases:
- `curSum == target` → found a valid subset → `true`
- `ind >= n` → no elements left, and target not met → `false`

---

## 4. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    bool fn(int ind, int n, int curSum, int target,
            vector<int>& arr) {
        if(curSum == target) return true;
        if(ind >= n)         return false;
        if(curSum > target)  return false;   // pruning: sum exceeded

        bool take    = fn(ind+1, n, curSum + arr[ind], target, arr);
        bool notTake = fn(ind+1, n, curSum,             target, arr);

        return take || notTake;
    }

public:
    bool isSubsetSum(vector<int>& arr, int sum) {
        int n = arr.size();
        return fn(0, n, 0, sum, arr);
    }
};
```

### Recursion Tree for arr=[3,4,2], target=6

```
fn(0, curSum=0)
├── take 3: fn(1, curSum=3)
│   ├── take 4: fn(2, curSum=7) → 7>6 → false
│   └── notTake: fn(2, curSum=3)
│       ├── take 2: fn(3, curSum=5) → ind>=n, sum≠6 → false
│       └── notTake: fn(3, curSum=3) → false
└── notTake: fn(1, curSum=0)
    ├── take 4: fn(2, curSum=4)
    │   ├── take 2: fn(3, curSum=6) → curSum==target → TRUE ✅
    │   └── notTake: fn(3, curSum=4) → false
    └── notTake: fn(2, curSum=0)
        ...
```

### Two Equivalent Formulations

The code tracks `curSum` and checks `curSum == target`. An alternative formulation tracks remaining target:
```
f(ind, remTarget): can we form remTarget from arr[ind..n-1]?
  take:    f(ind+1, remTarget - arr[ind])
  notTake: f(ind+1, remTarget)
  base: remTarget == 0 → true
```

Both are equivalent. The tabulation (approach 3) uses the second formulation via `dp[i][s]`.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Two branches per element |
| **Space** | `O(n)` | Recursion stack depth |

---

## 5. Approach 2 — Memoization (Top-Down DP)

The state is `(ind, curSum)` — two changing variables → 2D dp:

```cpp
class Solution {
private:
    bool fn(int ind, int n, int curSum, int target,
            vector<vector<int>>& dp, vector<int>& arr) {
        if(curSum == target) return true;
        if(curSum > target)  return false;
        if(ind >= n)         return false;

        if(dp[ind][curSum] != -1)
            return dp[ind][curSum];

        bool take    = fn(ind+1, n, curSum + arr[ind], target, dp, arr);
        bool notTake = fn(ind+1, n, curSum,             target, dp, arr);

        return dp[ind][curSum] = take || notTake;
    }

public:
    bool isSubsetSum(vector<int>& arr, int sum) {
        int n = arr.size();
        vector<vector<int>> dp(n, vector<int>(sum+1, -1));
        return fn(0, n, 0, sum, dp, arr);
    }
};
```

### dp Dimensions: `n × (sum+1)`

- Rows: index `ind` (0 to n-1)
- Columns: `curSum` (0 to sum)
- Value: -1 (uncomputed), 0 (false), 1 (true)

Total unique states: `n × (sum+1)`. Each computed once.

### Why `curSum > target` Early Return?

```cpp
if(curSum > target) return false;
```

Once `curSum` exceeds the target, no way to reduce it (all elements ≥ 0). This prunes the tree early. Without this, we'd cache unnecessarily large curSum values that can never succeed.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × sum)` | n × sum unique states, O(1) each |
| **Space** | `O(n × sum) + O(n)` | dp table + recursion stack |

---

## 6. Approach 3 — Tabulation (Bottom-Up DP)

### State Redefinition for Tabulation

Shift to: `dp[i][s]` = "can we form sum `s` using elements from the first `i` elements (`arr[0..i-1]`)?"

This is slightly different from the memoization state (which tracked `curSum`). Here `s` is the **target remaining** and `i` is how many elements we've considered.

### Base Cases

```cpp
// Sum 0 is always achievable (empty subset)
for(int i = 0; i <= n; i++)
    dp[i][0] = true;

// With 0 elements, only sum=0 achievable (all others false by default)
// dp[0][s] = false for s > 0  ← initialized to false already
```

### Recurrence

```cpp
dp[i][s] = dp[i-1][s]                          // notTake arr[i-1]
         || (s >= arr[i-1] && dp[i-1][s-arr[i-1]])  // take arr[i-1]
```

```cpp
class Solution {
public:
    bool isSubsetSum(vector<int>& arr, int sum) {
        int n = arr.size();
        vector<vector<bool>> dp(n+1, vector<bool>(sum+1, false));

        // Base case: sum=0 always achievable
        for(int i = 0; i <= n; i++)
            dp[i][0] = true;

        for(int i = 1; i <= n; i++) {
            for(int s = 1; s <= sum; s++) {
                bool notTake = dp[i-1][s];

                bool take = false;
                if(s >= arr[i-1])
                    take = dp[i-1][s - arr[i-1]];

                dp[i][s] = take || notTake;
            }
        }

        return dp[n][sum];
    }
};
```

### Tabulation Trace for arr=[3,4,2], sum=6

```
dp table: rows=items (0..3), cols=sum (0..6)

Initial (dp[i][0]=true for all i):
     s:  0  1  2  3  4  5  6
i=0:    T  F  F  F  F  F  F
i=1:    T  ?  ?  ?  ?  ?  ?
i=2:    T  ?  ?  ?  ?  ?  ?
i=3:    T  ?  ?  ?  ?  ?  ?

Item 1 = arr[0] = 3 (i=1):
  s=1: notTake=dp[0][1]=F, take(s<3)=F → F
  s=2: notTake=F, take(s<3)=F → F
  s=3: notTake=dp[0][3]=F, take=dp[0][0]=T → T
  s=4: notTake=F, take=dp[0][1]=F → F
  s=5: notTake=F, take=dp[0][2]=F → F
  s=6: notTake=F, take=dp[0][3]=F → F

     s:  0  1  2  3  4  5  6
i=1:    T  F  F  T  F  F  F    ← only sum=3 achievable with {3}

Item 2 = arr[1] = 4 (i=2):
  s=1: F
  s=2: F
  s=3: notTake=dp[1][3]=T → T
  s=4: notTake=F, take=dp[1][0]=T → T
  s=5: notTake=F, take=dp[1][1]=F → F
  s=6: notTake=F, take=dp[1][2]=F → F
  s=7: would be T (3+4) but s only goes to 6

     s:  0  1  2  3  4  5  6
i=2:    T  F  F  T  T  F  F    ← {3} or {4} achievable

Item 3 = arr[2] = 2 (i=3):
  s=1: F
  s=2: notTake=F, take=dp[2][0]=T → T
  s=3: notTake=T → T
  s=4: notTake=T → T
  s=5: notTake=F, take=dp[2][3]=T → T
  s=6: notTake=F, take=dp[2][4]=T → T ← {4,2}=6! ✅

     s:  0  1  2  3  4  5  6
i=3:    T  F  T  T  T  T  T

return dp[3][6] = true ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × sum)` | Two nested loops |
| **Space** | `O(n × sum)` | 2D boolean array |

---

## 7. Approach 4 — Space Optimization

`dp[i]` only depends on `dp[i-1]` → keep two 1D arrays:

```cpp
class Solution {
public:
    bool isSubsetSum(vector<int>& arr, int sum) {
        int n = arr.size();

        vector<bool> prev(sum+1, false);
        prev[0] = true;   // base: empty subset → sum=0 achievable

        for(int i = 1; i <= n; i++) {
            vector<bool> cur(sum+1, false);
            cur[0] = true;   // empty subset always achieves sum=0

            for(int s = 1; s <= sum; s++) {
                bool notTake = prev[s];
                bool take    = (s >= arr[i-1]) ? prev[s - arr[i-1]] : false;
                cur[s] = take || notTake;
            }

            prev = cur;
        }

        return prev[sum];
    }
};
```

### Why `cur[0] = true` in Every Iteration?

`dp[i][0] = true` for all `i` (empty subset always sums to 0). In the space-optimized version, `cur` is freshly allocated with `false`. We must explicitly set `cur[0] = true` each iteration to maintain this invariant.

Alternatively, initialize `cur` as a copy of `prev` or set `cur[0] = true` explicitly — both work.

### Space Optimization Trace

```
prev = [T, F, F, F, F, F, F]   (0 items)

i=1 (arr[0]=3):
  cur = [T, F, F, T, F, F, F]
  prev = [T, F, F, T, F, F, F]

i=2 (arr[1]=4):
  cur = [T, F, F, T, T, F, F]
  prev = [T, F, F, T, T, F, F]

i=3 (arr[2]=2):
  cur = [T, F, T, T, T, T, T]
  prev = [T, F, T, T, T, T, T]

return prev[6] = true ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × sum)` | Same loops |
| **Space** | `O(2 × sum)` = `O(sum)` | Two boolean arrays |

---

## 8. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` | Pick/not-pick tree |
| **Memoization** | `O(n × sum)` | `O(n × sum + n)` | Cache `(ind, curSum)` |
| **Tabulation** | `O(n × sum)` | `O(n × sum)` | Build `dp[i][s]` bottom-up |
| **Space Optimized** | `O(n × sum)` | `O(sum)` | Slide two 1D arrays |

---

## 9. The Subset DP Template

This problem establishes the **subset/knapsack DP template**:

```
dp[i][s] = can we achieve sum s using first i elements?

Recurrence:
  dp[i][s] = dp[i-1][s]                       // skip element i
           OR dp[i-1][s - arr[i-1]]            // include element i (if s >= arr[i-1])

Base:
  dp[i][0] = true    // empty subset achieves sum 0
  dp[0][s] = false   // no elements → can't achieve s > 0
```

Nearly identical pattern appears in:
- **Count subsets with sum K:** replace `||` with `+`
- **0/1 Knapsack:** replace `bool` with `int` (max value), `||` with `max`
- **Partition Equal Subset Sum:** check if `dp[n][totalSum/2]` is true

The only change is the "aggregation operator" (`||` for existence, `+` for count, `max` for optimization).
