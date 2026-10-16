# Partitions with Given Difference

---

## 1. The Full Mathematical Derivation — Step by Step

### Start With the Original Problem

Split array into two subsets S1 and S2 such that `S1 - S2 = diff`.

Define:
- `S1` = sum of first subset (the larger one)
- `S2` = sum of second subset
- `T` = totalSum of all elements
- `D` = required difference (`diff`)

We have two simultaneous equations:
```
S1 - S2 = D    ... (equation 1 — the constraint we want)
S1 + S2 = T    ... (equation 2 — every element in exactly one subset)
```

---

### The Substitution

From equation 2: `S1 = T - S2`

Now substitute into equation 1 — **replace S1 everywhere** with `(T - S2)`:

```
      S1    -  S2  = D
(T - S2)  -  S2  = D      ← substituted S1 = T - S2
 T - 2×S2       = D      ← simplified
     2×S2       = T - D
       S2       = (T - D) / 2
```

**S1 didn't vanish.** It was replaced by its equivalent expression `(T - S2)`. We have one equation with one unknown (S2) and solved it directly.

---

### Why This Works — The Substitution Intuition

Think of it like a simple algebra problem:

```
x - y = 5    and    x + y = 11

From second: x = 11 - y
Substitute into first: (11 - y) - y = 5
                        11 - 2y = 5
                             y = 3
                             x = 8
```

We didn't "remove" x. We replaced it with what we know it equals — `(11-y)`. Same thing here.

| Before substitution | After substitution |
|---|---|
| `S1 - S2 = D` | `(T - S2) - S2 = D` |
| S1 | Replaced by `T - S2` |
| S2 | Unchanged |
| D | Unchanged |

---

### The Result

```
S2 = (T - D) / 2
```

So the problem becomes:

> **Count subsets whose sum equals `S2 = (T - D) / 2`.**

Once we find any subset with that sum, the remaining elements automatically form S1 with sum `T - S2`.

---

### Concrete Example

```
arr = [5, 2, 6, 4], diff = 3

T = 17, D = 3
S2 = (17 - 3) / 2 = 14 / 2 = 7

Find subsets of {5,2,6,4} with sum = 7:
  {5, 2} = 7 ✅

If S2 = {5,2} = 7, then S1 = {6,4} = 10
S1 - S2 = 10 - 7 = 3 = diff ✅

Answer: 1 (only one such subset)
```

---

## 2. When is the Answer Automatically 0?

Before running any DP, two checks:

**Check 1 — `diff > totalSum`:**
```
The maximum possible difference is totalSum itself (one subset takes everything,
the other is empty with sum 0). If diff > T, impossible.
return 0;
```

**Check 2 — `(T - D)` is odd:**
```
S2 = (T - D) / 2

If T - D is odd, dividing by 2 gives a non-integer.
Subset sums are always integers. Impossible.
return 0;

Example: arr=[3,2,5,1], diff=4
T = 11, T-D = 7 (odd) → S2 = 7/2 = 3.5 → impossible → return 0
```

---

## 3. Problem Reduction Summary

```
Partitions with difference D
        ↓
Count subsets with sum = (T - D) / 2
        ↓
Count Subsets with Sum K  ← already solved!
```

This problem is a **direct application** of Count Subsets with Sum K. The only new work is the 3-line reduction:
1. Compute `T = totalSum`
2. Check `diff > T` or `(T-D)%2 != 0` → return 0
3. Set `target = (T-D)/2`

---

## 4. Problem Statement

Given an array `arr[]` and a value `diff`, count the number of ways to split the array into two subsets such that their difference equals `diff`.

```
arr = [5, 2, 6, 4], diff = 3
Answer: 1

arr = [1, 1, 2, 3], diff = 1
T=7, target=(7-1)/2=3
Count subsets with sum 3: {1,2},{1,2},{3} → 3
Answer: 3
```

---

## 5. Code (Space-Optimized)

```cpp
class Solution {
public:
    int countPartitions(vector<int>& arr, int diff) {
        int n = arr.size();
        int totalSum = accumulate(arr.begin(), arr.end(), 0);

        // Check 1: impossible if diff > totalSum
        if(diff > totalSum) return 0;

        // T - D must be even (so S2 is an integer)
        int remain = totalSum - diff;
        if(remain % 2 != 0) return 0;

        int target = remain / 2;   // S2 = (T - D) / 2

        // Count Subsets with Sum = target
        // Using reverse-direction DP (from previous problem)
        vector<int> prev(target+1, 0);
        prev[target] = 1;   // base: accumulated target → 1 way

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

---

## 6. Dry Run

```
arr = [1, 1, 2, 3], diff = 1
T = 7, remain = 7-1 = 6 (even), target = 3

We need: count subsets summing to 3.

prev = [0, 0, 0, 1]   (prev[3]=1, target=3)

ind=3 (arr[3]=3):
  sum=0: take=prev[3]=1, notTake=prev[0]=0 → cur[0]=1
  sum=1: take(1+3=4>3→0), notTake=prev[1]=0 → cur[1]=0
  sum=2: take(2+3=5>3→0), notTake=prev[2]=0 → cur[2]=0
  sum=3: take(3+3=6>3→0), notTake=prev[3]=1 → cur[3]=1

prev = [1, 0, 0, 1]

ind=2 (arr[2]=2):
  sum=0: take=prev[2]=0, notTake=prev[0]=1 → 1
  sum=1: take=prev[3]=1, notTake=prev[1]=0 → 1
  sum=2: take(2+2=4>3→0), notTake=prev[2]=0 → 0
  sum=3: take(3+2=5>3→0), notTake=prev[3]=1 → 1

prev = [1, 1, 0, 1]

ind=1 (arr[1]=1):
  sum=0: take=prev[1]=1, notTake=prev[0]=1 → 2
  sum=1: take=prev[2]=0, notTake=prev[1]=1 → 1
  sum=2: take=prev[3]=1, notTake=prev[2]=0 → 1
  sum=3: take(3+1=4>3→0), notTake=prev[3]=1 → 1

prev = [2, 1, 1, 1]

ind=0 (arr[0]=1):
  sum=0: take=prev[1]=1, notTake=prev[0]=2 → 3
  sum=1: take=prev[2]=1, notTake=prev[1]=1 → 2
  sum=2: take=prev[3]=1, notTake=prev[2]=1 → 2
  sum=3: take(3+1=4>3→0), notTake=prev[3]=1 → 1

prev = [3, 2, 2, 1]

return prev[0] = 3 ✅

Verify:
  {1(ind0), 2} = 3 ✅ → S1={1,3}=4, diff=4-3=1 ✅
  {1(ind1), 2} = 3 ✅ → S1={1,3}=4, diff=4-3=1 ✅ (different index)
  {3}          = 3 ✅ → S1={1,1,2}=4, diff=4-3=1 ✅
```

---

## 7. Story Points

---

**Story Point 1 — "The substitution makes two unknowns into one"**

We started with two unknowns (S1, S2) and two equations. Substituting `S1 = T - S2` into the difference equation eliminates S1 entirely, leaving one equation in one unknown (S2). This is standard algebra — the same technique used to solve any 2×2 system of equations.

---

**Story Point 2 — "We only need to find S2 — S1 is determined automatically"**

Once we know which elements form S2 (sum = `(T-D)/2`), S1 is everything else. We never explicitly search for S1. This is why the problem reduces to "count subsets with sum `target`" rather than "count pairs of subsets."

---

**Story Point 3 — "Two early-exit checks before any DP"**

```cpp
if(diff > totalSum) return 0;    // physically impossible
if(remain % 2 != 0) return 0;    // non-integer target
```

These prevent running expensive DP on provably impossible inputs. Both checks are `O(1)`.

---

**Story Point 4 — "Zeros in the array create multiple valid subsets with the same sum"**

If the array contains zeros (e.g., `arr=[0,1,2]`, target=3):
- Subset `{3}` = 3 (without zero)
- Subset `{0,3}` = 3 (with zero)

These are DIFFERENT subsets (different elements) even though they have the same sum. The DP correctly counts both because zeros contribute to the "take" branch, branching the count even when `sum + 0 = sum`.

---

**Story Point 5 — "This completes the subset DP chain"**

```
Subset Sum (exist?)
    ↓ change || to +
Count Subsets with Sum K
    ↓ derive target algebraically
Partitions with Given Difference
    ↓ use Count Subsets
```

Three problems, one DP. Each builds on the previous by changing one thing: the aggregation operator, then the target derivation.

---

## 8. Complexity Analysis

### Time Complexity — `O(n × target)` = `O(n × (T-D)/2)`

| Step | Cost |
|---|---|
| Compute totalSum | `O(n)` |
| DP loops | `O(n × target)` |

**Total: `O(n × (T-D)/2)`**

---

### Space Complexity — `O(target)` = `O((T-D)/2)`

| Structure | Size |
|---|---|
| `prev[]` + `cur[]` | `O(2 × target)` |

**Total: `O(target)`**

---

## 9. The Subset DP Family — Complete

| Problem | Given | Derive | DP operator | Answer |
|---|---|---|---|---|
| Subset Sum | `arr`, `target` | — | `\|\|` | `dp[n][target]` |
| Count Subsets Sum K | `arr`, `target` | — | `+` | `dp[n][target]` |
| Partition Equal Subset | `arr` | `target = T/2` | `\|\|` | `dp[n][T/2]` |
| Min Subset Difference | `arr` | scan `dp[n][0..T/2]` | `\|\|` then scan | `min(T-2*s1)` |
| **Partitions with Diff D** | `arr`, `diff` | `target = (T-D)/2` | `+` | Count subsets summing to target |
