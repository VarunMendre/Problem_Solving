# House Robber II (Circular Street)

---

## 1. Problem Statement

Same as House Robber, but now the houses are arranged in a **circle** — the first and last houses are adjacent. You cannot rob two adjacent houses.

Return the **maximum amount** you can rob.

```
nums = [2, 3, 2]

Circle: house 0 — house 1 — house 2 — house 0 (wraps around)

Can't rob 0 and 2 together (they're adjacent via the circle).
Options: rob {0}: 2, rob {1}: 3, rob {2}: 2
Answer: 3
```

```
nums = [1, 2, 3, 1]

Options:
  Rob 0,2: 1+3 = 4 ✅
  Rob 1,3: 2+1 = 3
  Rob 0:   1
  Rob 1:   2
  Rob 2:   3

Answer: 4
```

---

## 2. Why House Robber I's Solution Fails Here

In House Robber I, we could freely pick `nums[0]` and `nums[n-1]` together — they weren't adjacent.

In the circular version, they **are adjacent** (the street wraps around). If we apply House Robber I directly:

```
nums = [2, 3, 2]

House Robber I gives:
  dp[0]=2, dp[1]=3, dp[2]=max(2+2, 3)=max(4,3)=4 → returns 4 ❌

But robbing houses 0 and 2 is INVALID (they're adjacent in the circle)!
Correct answer: 3
```

---

## 3. The Key Insight — Two Non-Overlapping Subproblems

The circular constraint boils down to one simple fact:

> **Houses `0` and `n-1` cannot BOTH be robbed.**

So exactly one of them must be excluded from any valid solution. This gives us two cases:

**Case 1 — Exclude house `0`:** Run House Robber I on `nums[1..n-1]`

**Case 2 — Exclude house `n-1`:** Run House Robber I on `nums[0..n-2]`

The answer is `max(Case 1, Case 2)`.

**Why is this correct?**

- Any valid robbery sequence either includes house `0` or it doesn't.
  - If it includes house `0`: house `n-1` is excluded → Case 2 covers this
  - If it doesn't include house `0`: Case 1 covers this

- Both cases consider ALL possible valid subsets. The maximum of both = global optimum.

---

## 4. Why Not Three Cases?

You might think we also need a case where NEITHER house `0` nor house `n-1` is robbed. But this is already covered:

- Case 1 (exclude house 0) includes solutions where house `n-1` is also not robbed
- Case 2 (exclude house n-1) includes solutions where house `0` is also not robbed

Both cases are supersets — neither case forces you to rob the "included" endpoint.

```
Case 1 = run House Robber on [1..n-1]
  → can choose to rob or not rob house n-1
  → covers all solutions that don't include house 0

Case 2 = run House Robber on [0..n-2]
  → can choose to rob or not rob house 0
  → covers all solutions that don't include house n-1

Union of Case 1 and Case 2 = ALL valid solutions
```

---

## 5. The `fn(startInd, endInd)` Helper

```cpp
int fn(int startInd, int endInd, vector<int>& nums) {
    int prev = nums[startInd], prev1 = 0;

    for(int i = startInd + 1; i < endInd; i++) {
        int pick = nums[i];
        if(i > 1)
            pick += prev1;

        int notPick = 0 + prev;

        int curr = max(pick, notPick);

        prev1 = prev;
        prev  = curr;
    }

    return prev;
}
```

This is the space-optimized House Robber I, generalized to work on a subarray `[startInd, endInd)` (half-open interval — includes `startInd`, excludes `endInd`).

**Two calls:**
- `fn(1, n, nums)` → range `[1, n-1]` → excludes house `0`
- `fn(0, n-1, nums)` → range `[0, n-2]` → excludes house `n-1`

---

### Bug in the Helper: `if(i > 1)` Should Be `if(i > startInd)`

```cpp
if(i > 1)
    pick += prev1;
```

This condition checks `i > 1` (global index > 1), but the intent is: "are there at least 2 houses before `i` in the SUBARRAY?"

When `startInd = 1` and `i = 2`:
- `i > 1` is TRUE → adds `prev1`
- `prev1` = 0 (initialized, never updated yet because this is only the second iteration)
- Adding 0 is harmless → code works correctly by coincidence

When `startInd = 0` and `i = 1`:
- `i > 1` is FALSE → correct (can't look back 2 from index 1 in 0-indexed)

The code happens to work correctly for these specific two subarray ranges, but the correct general condition would be `if(i > startInd + 1)` or `if(i > startInd)` ... actually:

For `i = startInd + 1` (second element of subarray): picking means just `nums[i]` (no prev1 needed)
For `i > startInd + 1`: picking means `nums[i] + prev1`

The condition should be `if(i > startInd + 1)`. The code uses `i > 1` which happens to be:
- For `fn(0, n-1)`: startInd=0, so `startInd+1=1`, condition `i>1` ≡ `i>startInd+1` ✅
- For `fn(1, n)`: startInd=1, so `startInd+1=2`, condition `i>1` means `i≥2`, but we need `i>2`. At `i=2`, prev1=0 (unset), adding 0 is harmless ✅ (works by accident)

---

## 6. Dry Run

```
nums = [2, 3, 2]   n=3
```

**Case 1: excludeFirst = fn(1, 3, nums)**
```
startInd=1, endInd=3
prev = nums[1] = 3, prev1 = 0

Loop: i from startInd+1=2 to endInd-1=2 (just i=2):
  pick = nums[2] = 2
  i=2 > 1 → pick += prev1=0 → pick=2
  notPick = prev=3
  curr = max(2,3) = 3
  prev1=3, prev=3

return prev=3
```

**Case 2: excludeLast = fn(0, 2, nums)**
```
startInd=0, endInd=2
prev = nums[0] = 2, prev1 = 0

Loop: i from startInd+1=1 to endInd-1=1 (just i=1):
  pick = nums[1] = 3
  i=1 > 1? NO → pick stays 3
  notPick = prev=2
  curr = max(3,2) = 3
  prev1=2, prev=3

return prev=3
```

**Answer: max(excludeFirst=3, excludeLast=3) = 3 ✅**

---

```
nums = [1, 2, 3, 1]   n=4
```

**Case 1: fn(1, 4, nums) → range [1,2,3]**
```
prev=nums[1]=2, prev1=0

i=2: pick=3+(i>1? prev1=0)=3, notPick=2, curr=3, prev1=2,prev=3
i=3: pick=1+(i>1? prev1=2)=3, notPick=3, curr=3, prev1=3,prev=3

return 3
```

**Case 2: fn(0, 3, nums) → range [0,1,2]**
```
prev=nums[0]=1, prev1=0

i=1: pick=2+(i>1? NO)=2, notPick=1, curr=2, prev1=1,prev=2
i=2: pick=3+(i>1? prev1=1)=4, notPick=2, curr=4, prev1=2,prev=4

return 4
```

**Answer: max(3, 4) = 4 ✅**

---

## 7. Complete Code

```cpp
class Solution {
private:
    // Space-optimized House Robber I on subarray [startInd, endInd)
    int fn(int startInd, int endInd, vector<int>& nums) {
        int prev  = nums[startInd];   // best amount after first house of subarray
        int prev1 = 0;                 // best amount two houses before current

        for(int i = startInd + 1; i < endInd; i++) {
            int pick = nums[i];
            if(i > 1)             // has a house two steps back in subarray
                pick += prev1;    // nums[i] + best two houses ago

            int notPick = prev;   // skip current house

            int curr = max(pick, notPick);

            prev1 = prev;   // slide window
            prev  = curr;
        }

        return prev;
    }

public:
    int rob(vector<int>& nums) {
        int n = nums.size();

        // Edge case: only one house
        if(n == 1) return nums[0];

        // Case 1: exclude house 0 → run on nums[1..n-1]
        int excludeFirst = fn(1, n, nums);

        // Case 2: exclude house n-1 → run on nums[0..n-2]
        int excludeLast  = fn(0, n-1, nums);

        return max(excludeFirst, excludeLast);
    }
};
```

---

## 8. Complexity Analysis

### Time Complexity — `O(n)`

| Step | Cost |
|---|---|
| `fn(1, n, nums)` | `O(n-1)` |
| `fn(0, n-1, nums)` | `O(n-1)` |
| **Total** | `O(2n)` = `O(n)` |

### Space Complexity — `O(1)`

Only `prev`, `prev1`, `curr` — no arrays. `O(1)` auxiliary space.

---

## 9. House Robber I vs II — Side by Side

| | House Robber I | House Robber II |
|---|---|---|
| **Layout** | Linear (no wrap) | Circular (0 and n-1 adjacent) |
| **Constraint** | No adjacent houses | No adjacent houses + no 0 and n-1 together |
| **Solution** | One pass of DP | Two passes of DP (exclude first OR last) |
| **TC** | `O(n)` | `O(2n)` = `O(n)` |
| **SC** | `O(1)` | `O(1)` |
| **Edge case** | n=1: return nums[0] | n=1: return nums[0] (handled separately) |

The circular version is solved by **reducing it to two instances of the linear version**. This "split into cases" technique appears in many circular DP problems.
