# Minimum Subset Sum Difference

---

## 1. The Thinking Process — Read This First

### The Setup

We need to split array `nums` into two non-empty subsets S1 and S2 such that `|S1 - S2|` is minimized.

Let `S = totalSum = sum of all elements`.

Key relationship:
```
S1 + S2 = S        (together they contain all elements)
|S1 - S2| = ?      (what we want to minimize)
```

Since `S1 + S2 = S`, we can express S2 in terms of S1:
```
S2 = S - S1
|S1 - S2| = |S1 - (S - S1)| = |2×S1 - S|
```

So minimizing `|S1 - S2|` = minimizing `|2×S1 - S|`.

---

### S1 ≤ S/2 Is Enough to Check

If S1 ≤ S/2, then S2 ≥ S/2 → S2 ≥ S1 → difference is `S2 - S1 = (S - S1) - S1 = S - 2×S1`.

If S1 > S/2, the roles of S1 and S2 just swap — we'd be double-counting. So we only need to check all achievable S1 values in `[0, S/2]`.

```
For every valid S1 ≤ S/2:
  difference = S2 - S1 = (S - S1) - S1 = S - 2×S1

Minimize over all such S1.
```

---

### What the `prev[]` Array Gives Us

After running Subset Sum DP on the full range `[0, S]`, `prev[s1]` tells us:

> **`prev[s1] = true` if and only if there exists a subset of `nums` with sum exactly `s1`.**

This gives us ALL achievable S1 values at once — the entire row of the DP table.

```
nums = [1, 6, 11, 5], S = 23

After DP, prev[] might look like:
  prev[0]  = T  (empty subset)
  prev[1]  = T  ({1})
  prev[5]  = T  ({5})
  prev[6]  = T  ({6})
  prev[7]  = T  ({1,6})
  prev[11] = T  ({11})
  prev[12] = T  ({1,11})
  prev[16] = T  ({5,11})
  prev[17] = T  ({1,5,11} or {6,11})
  ...

For each valid s1 ≤ 23/2 = 11:
  s1=1:  |23-1-1|  = |23-2|  = 21
  s1=5:  |23-10|   = 13
  s1=6:  |23-12|   = 11
  s1=7:  |23-14|   = 9
  s1=11: |23-22|   = 1  ← minimum!

Answer: 1 (S1={11}, S2={1,6,5}=12, |12-11|=1)
```

---

### Why Only Check s1 ≤ S/2?

If we checked s1 > S/2:
```
s1=12: S - 2×12 = 23-24 = -1, abs = 1  ← same as s1=11!
s1=16: S - 2×16 = 23-32 = -9, abs = 9  ← same as s1=7!
```

Values above S/2 are mirror images of values below S/2. Checking only `[0, S/2]` covers all unique differences without redundancy.

---

## 2. Problem Statement

Given an integer array `nums`, split it into two subsets S1 and S2 (both non-empty) such that every element belongs to exactly one subset. Return the **minimum absolute difference** between the sums of the two subsets.

```
nums = [1, 6, 11, 5]

Best split: S1={11}, S2={1,6,5}=12
Difference: |11-12| = 1

Answer: 1
```

```
nums = [3, 9, 7, 3]

Possible splits:
  {3,9} vs {7,3}: |12-10| = 2
  {3,7} vs {9,3}: |10-12| = 2
  {3,3} vs {9,7}: |6-16| = 10
  {9} vs {3,7,3}: |9-13| = 4
  {3,9,7} vs {3}: |19-3| = 16

Answer: 2
```

---

## 3. The Algorithm

**Step 1:** Run Subset Sum DP on the full array with target = `totalSum`. This fills `prev[s]` = true for all achievable subset sums.

**Step 2:** For each achievable S1 ∈ [0, S/2]:
```
difference = (S - S1) - S1 = S - 2×S1
```
Track the minimum.

---

## 4. Code

```cpp
class Solution {
public:
    int minDifference(vector<int>& nums) {
        int n = nums.size();
        int totalSum = accumulate(nums.begin(), nums.end(), 0);

        // Step 1: Subset Sum DP — find all achievable subset sums
        vector<bool> prev(totalSum+1, false);
        prev[0] = true;   // empty subset achieves sum 0

        for(int ind = 1; ind <= n; ind++) {
            vector<bool> cur(totalSum+1, false);
            cur[0] = true;

            for(int sum = 1; sum <= totalSum; sum++) {
                bool notTake = prev[sum];
                bool take    = (sum >= nums[ind-1]) ? prev[sum - nums[ind-1]] : false;
                cur[sum]     = take | notTake;
            }

            prev = cur;
        }

        // Step 2: Find minimum |S1 - S2| = minimum (S - 2*S1)
        // Only check S1 in [0, S/2] — S1 > S/2 gives symmetric results
        int mini = 1e9;

        for(int s1 = 0; s1 <= totalSum/2; s1++) {
            if(prev[s1]) {
                // S2 = totalSum - s1
                // |S1 - S2| = |(s1) - (totalSum - s1)| = |2*s1 - totalSum| = totalSum - 2*s1
                mini = min(mini, abs(totalSum - s1) - s1);
                //                    ↑             ↑
                //               = S - s1           this is wrong — see fix below
            }
        }

        return mini;
    }
};
```

### Fix: The Formula `abs(totalSum - s1) - s1`

The code writes:
```cpp
mini = min(mini, abs(totalSum - s1) - s1);
```

Let's expand: `abs(S - s1) - s1`

Since `s1 ≤ S/2`, `S - s1 ≥ S/2 ≥ s1` → `S - s1 > 0`, so `abs(S - s1) = S - s1`.

```
abs(S - s1) - s1 = (S - s1) - s1 = S - 2×s1
```

Which equals `|S2 - S1|` = `|(S - s1) - s1|` = the difference. ✓

It's a compact but confusing way to write `S - 2*s1`. The clearer version:

```cpp
int s2 = totalSum - s1;
mini = min(mini, s2 - s1);   // s2 >= s1 since s1 <= totalSum/2
```

---

## 5. Dry Run

```
nums = [3, 9, 7, 3], totalSum = 22, S/2 = 11

Step 1: Subset Sum DP on range [0, 22]

After processing all 4 elements, prev[] = true for:
  0  (empty)
  3  ({3})
  3  ({3} — duplicate, same sum)
  6  ({3,3})
  7  ({7})
  9  ({9})
  10 ({7,3} or {3,7})
  12 ({9,3} or {3,9})
  13 ({3,7,3})
  16 ({7,9})
  19 ({3,9,7})
  22 (all)

Step 2: Check s1 ∈ [0, 11]:
  s1=0:  diff = 22-0  = 22
  s1=3:  diff = 22-6  = 16
  s1=6:  diff = 22-12 = 10
  s1=7:  diff = 22-14 = 8
  s1=9:  diff = 22-18 = 4
  s1=10: diff = 22-20 = 2  ← minimum!

Answer: 2 ✅ ({7,3} = 10, rest = {3,9} = 12, |12-10|=2)
```

---

## 6. Story Points

---

**Story Point 1 — "S2 = S - S1 collapses two unknowns into one"**

We have two free variables (S1, S2) but the constraint `S1 + S2 = S` means we only need to find S1. The whole DP becomes: find the best achievable S1 in [0, S/2]. S2 is always `S - S1`.

---

**Story Point 2 — "The `prev[]` array is the full subset-sum landscape"**

After the DP, `prev[]` is like a map saying "these are all the sums you can build from this array." We scan this map once to find the best S1. This is why Subset Sum DP is a building block — compute it once, answer many queries.

---

**Story Point 3 — "Only check [0, S/2] — the mirror argument"**

S1 = 10, S2 = 12 → difference = 2
S1 = 12, S2 = 10 → difference = 2 (same!)

Every pair above S/2 is the mirror of a pair below. Checking only [0, S/2] covers all unique differences. This is why we have `s1 <= totalSum/2` in the loop.

---

**Story Point 4 — "The formula `abs(totalSum - s1) - s1` = `S - 2*s1`"**

Since `s1 ≤ S/2`, `S - s1 ≥ 0` always, so `abs(S - s1) = S - s1`.
Then `(S - s1) - s1 = S - 2*s1 = |S1 - S2|`.

Write it as `totalSum - 2*s1` for clarity in interviews.

---

## 7. Complexity Analysis

### Time Complexity — `O(n × S)`

| Step | Cost | Reason |
|---|---|---|
| Subset Sum DP | `O(n × S)` | n items × S possible sums |
| Scan for minimum | `O(S/2)` ≈ `O(S)` | Check half the dp row |

**Total: `O(n × S)`** where S = sum of all elements.

---

### Space Complexity — `O(S)`

| Structure | Size |
|---|---|
| `prev[]` + `cur[]` | `O(2S)` |

**Total: `O(S)`**

---

## 8. The Subset Sum Family — Updated

| Problem | What changes vs Subset Sum | Answer |
|---|---|---|
| Subset Sum | — (base problem) | `dp[n][target]` |
| Partition Equal Subset | target = S/2, odd check | `dp[n][S/2]` |
| **Min Subset Diff** | Run full DP, scan [0,S/2] | min(S - 2*s1) for true s1 |
| Count Subsets Sum K | `+` instead of `\|\|` | `dp[n][K]` as count |
| 0/1 Knapsack | maximize value | `max(dp[n][W])` |

Min Subset Difference uses Subset Sum as a subroutine and then does a post-processing scan — a pattern that appears in many optimization problems.
