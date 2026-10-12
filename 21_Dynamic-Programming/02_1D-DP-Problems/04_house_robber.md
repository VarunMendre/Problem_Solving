# House Robber

---

## 1. Problem Statement

You are a robber planning to rob houses along a street. Each house has some money `nums[i]`. The constraint: **you cannot rob two adjacent houses** (the alarm connects them).

Return the **maximum amount of money** you can rob without triggering any alarms.

```
nums = [2, 7, 9, 3, 1]

Options:
  Rob 0,2,4: 2+9+1 = 12
  Rob 0,2:   2+9   = 11
  Rob 1,3:   7+3   = 10
  Rob 0,3:   2+3   = 5
  Rob 1,4:   7+1   = 8
  Rob 0,2,4: 2+9+1 = 12  ← maximum

Answer: 12
```

---

## 2. Intuition — The Pick / Not-Pick Pattern

This is a foundational DP pattern. At every index `i`, you face exactly two choices:

**Pick house `i`:**
- Gain `nums[i]`
- Must skip `i-1` (adjacent) → move to `i-2`
- Total = `nums[i] + f(i-2)`

**Not Pick house `i`:**
- Gain nothing from `i`
- Can consider `i-1` → move to `i-1`
- Total = `f(i-1)`

Answer = `max(pick, notPick)`

This is optimal substructure: the best decision at index `i` depends only on the best decisions at `i-1` and `i-2` — making DP perfect.

---

## 3. The "Pick or Not Pick" Choice Tree

```
nums = [2, 7, 9, 3, 1]

f(4):
  pick:    1 + f(2)
           f(2): pick 9 + f(0)=2 → 11
                 notpick f(1)=7
                 max=11
  notpick: f(3)
           f(3): pick 3+f(1)=7 → 10
                 notpick f(2)=11
                 max=11

f(4) = max(1+11, 11) = max(12, 11) = 12 ✅
```

---

## 4. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int fn(int ind, vector<int>& nums) {
        if(ind == 0) return nums[ind];   // only one house: rob it
        if(ind < 0)  return 0;           // no houses: nothing to rob

        int pick    = nums[ind] + fn(ind - 2, nums);   // rob this + skip one
        int notPick = 0 + fn(ind - 1, nums);            // skip this

        return max(pick, notPick);
    }

public:
    int rob(vector<int>& nums) {
        return fn(nums.size() - 1, nums);
    }
};
```

### Why `ind < 0` Check?

When `ind == 1` and we pick, we call `fn(ind-2) = fn(-1)`. No house at index -1 → return 0. This base case prevents out-of-bounds access.

### Recursion Tree (n=4)

```
              f(3)
            /       \
         f(2)         f(2)    ← f(2) computed TWICE
        /    \       /    \
      f(1)  f(0)  f(1)  f(0)  ← f(1),f(0) computed multiple times
```

**Overlapping subproblems → exponential redundancy.**

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Each call branches into at most 2 subproblems |
| **Space** | `O(n)` | Recursion stack depth |

---

## 5. Approach 2 — Memoization (Top-Down DP)

```cpp
class Solution {
private:
    int fn(int ind, vector<int>& dp, vector<int>& nums) {
        if(ind == 0) return nums[ind];
        if(ind < 0)  return 0;

        if(dp[ind] != -1)
            return dp[ind];   // already solved → return instantly

        int pick    = nums[ind] + fn(ind - 2, dp, nums);
        int notPick = 0 + fn(ind - 1, dp, nums);

        return dp[ind] = max(pick, notPick);   // store before return
    }

public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n, -1);
        return fn(n-1, dp, nums);
    }
};
```

### What Changes

```
Recursion: f(2) computed 2+ times → exponential
Memo:      f(2) computed once → dp[2] = result
           Second call to f(2) → hits dp[2] → returns instantly (O(1))
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Each index 0..n-1 computed exactly once |
| **Space** | `O(n) + O(n)` | dp[] array + recursion call stack = `O(2n)` |

---

## 6. Approach 3 — Tabulation (Bottom-Up DP)

No recursion. Fill from `dp[0]` upward:

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n, 0);

        dp[0] = nums[0];   // base case: one house, rob it

        for(int i = 1; i < n; i++) {
            int pick = nums[i];
            if(i > 1)
                pick += dp[i-2];   // add best from two steps back

            int notPick = dp[i-1];   // best from one step back

            dp[i] = max(pick, notPick);
        }

        return dp[n-1];
    }
};
```

### Why `if(i > 1)` for pick?

When `i == 1`, `i-2 = -1` — out of bounds. The `if(i > 1)` guard ensures we only access `dp[i-2]` when `i ≥ 2`. When `i == 1`, picking just means taking `nums[1]` with no prior houses (so no addition).

### Tabulation Trace (nums=[2,7,9,3,1])

```
dp[0] = 2

i=1:
  pick    = nums[1] = 7  (i≤1, so no dp[-1])
  notPick = dp[0]  = 2
  dp[1]   = max(7, 2) = 7

i=2:
  pick    = nums[2] + dp[0] = 9+2 = 11
  notPick = dp[1] = 7
  dp[2]   = max(11, 7) = 11

i=3:
  pick    = nums[3] + dp[1] = 3+7 = 10
  notPick = dp[2] = 11
  dp[3]   = max(10, 11) = 11

i=4:
  pick    = nums[4] + dp[2] = 1+11 = 12
  notPick = dp[3] = 11
  dp[4]   = max(12, 11) = 12

Answer: dp[4] = 12 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Single loop |
| **Space** | `O(n)` | dp[] array |

---

## 7. Approach 4 — Space Optimization

`dp[i]` only needs `dp[i-1]` and `dp[i-2]` → replace array with two variables:

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();

        int prev  = nums[0];   // dp[i-1] — best up to previous house
        int prev1 = 0;         // dp[i-2] — best up to two houses ago

        for(int i = 1; i < n; i++) {
            int pick = nums[i];
            if(i > 1)
                pick += prev1;   // nums[i] + dp[i-2]

            int notPick = prev;  // dp[i-1]

            int curr = max(pick, notPick);

            prev1 = prev;   // slide: dp[i-2] ← dp[i-1]
            prev  = curr;   // slide: dp[i-1] ← dp[i]
        }

        return prev;   // prev = dp[n-1]
    }
};
```

### Variable Roles

```
prev  = dp[i-1] = "best amount robbing houses 0..i-1"
prev1 = dp[i-2] = "best amount robbing houses 0..i-2"
curr  = dp[i]   = "best amount robbing houses 0..i"
```

After each iteration, we shift: `prev1 ← prev`, `prev ← curr`. This "slides" the window forward by one position.

### Space Optimization Trace (nums=[2,7,9,3,1])

```
Init:  prev=2(dp[0]), prev1=0

i=1:
  pick    = 7  (i≤1, no prev1)
  notPick = prev=2
  curr    = max(7,2) = 7
  prev1=2, prev=7

i=2:
  pick    = 9 + prev1=2 → 11
  notPick = prev=7
  curr    = max(11,7) = 11
  prev1=7, prev=11

i=3:
  pick    = 3 + prev1=7 → 10
  notPick = prev=11
  curr    = max(10,11) = 11
  prev1=11, prev=11

i=4:
  pick    = 1 + prev1=11 → 12
  notPick = prev=11
  curr    = max(12,11) = 12
  prev1=11, prev=12

return prev=12 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n)` | Same loop |
| **Space** | `O(1)` | Only two integer variables |

---

## 8. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion** | `O(2ⁿ)` | `O(n)` | Direct pick/not-pick branches |
| **Memoization** | `O(n)` | `O(2n)` | Cache to avoid recomputation |
| **Tabulation** | `O(n)` | `O(n)` | Fill dp[] bottom-up |
| **Space Optimized** | `O(n)` | `O(1)` | Two variables slide forward |

---

## 9. The Pick / Not-Pick Template

House Robber establishes the fundamental **pick/not-pick** DP template. Variants of this appear throughout DP:

```
At index i:
  pick    = value[i] + dp[i-skip]    (take + skip adjacent)
  notPick = dp[i-1]                   (skip this)
  dp[i]   = max(pick, notPick)
```

| Problem | What changes |
|---|---|
| House Robber | skip = 2 (adjacent constraint) |
| House Robber II (circular) | Run the algorithm twice (exclude first or last house) |
| Delete and Earn | Sort coins into "houses" by value first |

Once you recognize the pick/not-pick pattern, all three collapse to the same DP.

---

## 10. Edge Cases

**Single house:** `n=1`
- Loop doesn't execute
- Return `prev = nums[0]` ✅

**Two houses:** `n=2`
- `i=1`: `pick = nums[1]` (i≤1, no prev1), `notPick = nums[0]`
- Return `max(nums[0], nums[1])` ✅

**All zeros:** nums=[0,0,0]
- dp[0]=0, all dp[i]=0
- Return 0 ✅
