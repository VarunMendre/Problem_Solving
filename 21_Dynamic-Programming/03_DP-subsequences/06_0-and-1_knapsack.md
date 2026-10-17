# 0/1 Knapsack

---

## 1. Problem Statement

Given `n` items, each with a weight `wt[i]` and value `val[i]`, and a knapsack with capacity `W`, find the **maximum value** you can carry. Each item can be used **at most once** (0/1 = either take it or leave it).

```
W=4
val = [1, 2, 3]
wt  = [4, 5, 1]

Item 0: weight=4, value=1
Item 1: weight=5, value=2  (too heavy for W=4, skip)
Item 2: weight=1, value=3

Options:
  {item0}: weight=4, value=1
  {item2}: weight=1, value=3
  {item0, item2}: weight=5 > 4, invalid
  
Best: take only item2, value=3
Answer: 3
```

```
W=5
val = [10, 40, 30, 50]
wt  = [5,  4,  6,  3]

Options:
  {item1, item3}: weight=4+3=7>5, no
  {item3}: weight=3, value=50
  {item1}: weight=4, value=40
  {item0}: weight=5, value=10
  {item2}: weight=6>5, no
  {item3+item0}: 3+5=8>5, no

Best: {item1}: value=40? or {item3}: value=50
Answer: 50
```

---

## 2. Intuition — The Pick / Not-Pick Pattern

At each item `ind`, with remaining capacity `cap`:

**Not pick item `ind`:** move to item `ind-1` with same capacity → `fn(ind-1, cap)`

**Pick item `ind`:** only if `wt[ind] <= cap`:
- Gain `val[ind]`
- Reduce capacity by `wt[ind]`
- Move to `fn(ind-1, cap - wt[ind])`

Answer = `max(notPick, pick)`

Base case: at `ind=0` (first item), take it if it fits, else take 0.

---

## 3. The Two Recursion Versions

### Version 1 (Your first recursion — with `totalScore`)

```cpp
fn(ind, totalScore, capacity, val, wt)
```

This carries accumulated score `totalScore` downward. At `ind==0`, it adds `val[0]` if it fits and returns max.

This formulation works but is **harder to memoize** — the state depends on three variables `(ind, totalScore, capacity)` instead of two.

### Version 2 (Standard — better for memoization)

```cpp
fn(ind, capacity, val, wt, dp)
```

Returns the **maximum additional value** from items `0..ind` with remaining capacity `cap`. No `totalScore` carried.

Base: `fn(0, cap) = val[0]` if `wt[0] <= cap`, else `0`.

This 2D state `(ind, capacity)` makes memoization clean and natural.

---

## 4. Approach 1 — Pure Recursion (Standard version)

```cpp
class Solution {
private:
    int fn(int ind, int capacity, vector<int>& val, vector<int>& wt) {
        if(ind == 0) {
            if(wt[0] <= capacity) return val[0];
            return 0;
        }

        int notPick = fn(ind-1, capacity, val, wt);

        int pick = 0;
        if(wt[ind] <= capacity)
            pick = val[ind] + fn(ind-1, capacity - wt[ind], val, wt);

        return max(notPick, pick);
    }

public:
    int knapsack(int W, vector<int>& val, vector<int>& wt) {
        int n = wt.size();
        return fn(n-1, W, val, wt);
    }
};
```

### Recursion Tree (3 items, W=4)

```
fn(2, 4)
├── notPick: fn(1, 4)
│   ├── notPick: fn(0, 4)
│   └── pick: fn(0, 4-wt[1]) [if fits]
└── pick: fn(1, 4-wt[2]) [if fits]
    ├── notPick: fn(0, ...)
    └── pick: fn(0, ...) ← REPEATED fn(0,...) calls
```

**Overlapping subproblems:** same `(ind, capacity)` computed multiple times.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(2ⁿ)` | Two branches per item |
| **Space** | `O(n)` | Recursion stack depth |

---

## 5. Approach 2 — Memoization

```cpp
class Solution {
private:
    int fn(int ind, int capacity, vector<int>& val,
           vector<int>& wt, vector<vector<int>>& dp) {
        if(ind == 0) {
            if(wt[0] <= capacity) return val[0];
            return 0;
        }

        if(dp[ind][capacity] != -1)
            return dp[ind][capacity];

        int notPick = fn(ind-1, capacity, val, wt, dp);

        int pick = 0;
        if(wt[ind] <= capacity)
            pick = val[ind] + fn(ind-1, capacity - wt[ind], val, wt, dp);

        return dp[ind][capacity] = max(notPick, pick);
    }

public:
    int knapsack(int W, vector<int>& val, vector<int>& wt) {
        int n = wt.size();
        vector<vector<int>> dp(n, vector<int>(W+1, -1));
        return fn(n-1, W, val, wt, dp);
    }
};
```

### dp Dimensions: `n × (W+1)`

`dp[ind][cap]` = max value from items `0..ind` with capacity `cap`.
Total states: `n × (W+1)`. Each computed once.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × W)` | n×W unique states, O(1) each |
| **Space** | `O(n × W) + O(n)` | dp table + stack |

---

## 6. Approach 3 — Tabulation

```cpp
class Solution {
public:
    int knapsack(int W, vector<int>& val, vector<int>& wt) {
        int n = wt.size();
        vector<vector<int>> dp(n, vector<int>(W+1, 0));

        // Base case: only item 0
        for(int cap = wt[0]; cap <= W; cap++)
            dp[0][cap] = val[0];

        for(int ind = 1; ind < n; ind++) {
            for(int cap = 1; cap <= W; cap++) {
                int notPick = dp[ind-1][cap];

                int pick = 0;
                if(wt[ind] <= cap)
                    pick = val[ind] + dp[ind-1][cap - wt[ind]];

                dp[ind][cap] = max(pick, notPick);
            }
        }

        return dp[n-1][W];
    }
};
```

### Base Case Explained — In Depth

```cpp
for(int cap = wt[0]; cap <= W; cap++)
    dp[0][cap] = val[0];
```

`dp[0][cap]` = max value using ONLY item 0 with knapsack capacity `cap`.

There are exactly two cases:

**Case 1 — `cap < wt[0]`:** Item 0 is too heavy to carry. Nothing can be taken. Value = 0. These entries stay at `0` from initialization — the loop doesn't touch them.

**Case 2 — `cap >= wt[0]`:** Item 0 fits. Take it for `val[0]`. You can't take it twice (0/1 knapsack), and there are no other items yet. Value = `val[0]` regardless of how much remaining capacity is left.

```
Example: val[0]=10, wt[0]=3, W=6

cap:    0   1   2   3   4   5   6
dp[0]:  0   0   0  10  10  10  10
        ↑___↑___↑   ↑___↑___↑___↑
        too heavy    fits → val[0]=10 always
```

**Key insight:** `dp[0][cap]` does NOT scale with `cap`. Whether `cap=3` or `cap=100`, the answer is the same `val[0]` — because you only have one item to take, and you either can or can't. More capacity doesn't help you when there's only one item.

This is why the loop starts at `wt[0]` (not 0) and sets every entry to the same fixed value `val[0]`:

```cpp
for(int cap = wt[0]; cap <= W; cap++)
    dp[0][cap] = val[0];   // flat line: same value for all valid caps
```

**What if you started the loop from `cap = 0`?**

```cpp
// WRONG — would overwrite valid 0s:
for(int cap = 0; cap <= W; cap++)
    dp[0][cap] = val[0];   // incorrectly sets dp[0][0]=val[0]
                           // but cap=0 < wt[0], so should be 0!
```

Starting at `wt[0]` naturally skips all under-capacity entries, leaving them at `0`.

**In the space-optimized versions (Approach 4 and 5):**

```cpp
vector<int> prev(W+1, 0);
for(int cap = wt[0]; cap <= W; cap++)
    prev[cap] = val[0];
```

Same logic — `prev` plays the role of `dp[0]`. Entries `cap < wt[0]` stay `0`, entries `cap >= wt[0]` are `val[0]`.

### Tabulation Trace (W=5, val=[10,40,50], wt=[5,4,3])

```
dp[0][cap]: val[0]=10, wt[0]=5
  cap=0..4: 0
  cap=5:    10

       cap: 0   1   2   3   4   5
ind=0:      0   0   0   0   0  10

ind=1 (val=40, wt=4):
  cap=1: notPick=dp[0][1]=0, pick(wt=4>1)=0 → 0
  cap=4: notPick=dp[0][4]=0, pick=40+dp[0][0]=40 → 40
  cap=5: notPick=dp[0][5]=10, pick=40+dp[0][1]=40 → max(10,40)=40

ind=1:     0   0   0   0  40  40

ind=2 (val=50, wt=3):
  cap=3: notPick=dp[1][3]=0, pick=50+dp[1][0]=50 → 50
  cap=4: notPick=dp[1][4]=40, pick=50+dp[1][1]=50 → max(40,50)=50
  cap=5: notPick=dp[1][5]=40, pick=50+dp[1][2]=50 → max(40,50)=50
       (Could do item1+item2? wt=4+3=7>5, no — correctly handled!)

ind=2:     0   0   0  50  50  50

Answer: dp[2][5] = 50 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × W)` | Two nested loops |
| **Space** | `O(n × W)` | 2D dp array |

---

## 7. Approach 4 — Space Optimization (Two 1D Arrays)

`dp[ind]` only reads from `dp[ind-1]` → keep only one previous row:

```cpp
class Solution {
public:
    int knapsack(int W, vector<int>& val, vector<int>& wt) {
        int n = wt.size();
        vector<int> prev(W+1, 0);

        for(int cap = wt[0]; cap <= W; cap++)
            prev[cap] = val[0];

        for(int ind = 1; ind < n; ind++) {
            vector<int> cur(W+1, 0);

            for(int cap = 1; cap <= W; cap++) {
                int notPick = prev[cap];

                int pick = 0;
                if(wt[ind] <= cap)
                    pick = val[ind] + prev[cap - wt[ind]];

                cur[cap] = max(pick, notPick);
            }

            prev = cur;
        }

        return prev[W];
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × W)` | Same loops |
| **Space** | `O(2W)` = `O(W)` | Two arrays of size W+1 |

---

## 8. Approach 5 — Super Space Optimization (One 1D Array, Right to Left)

### The Goal

Eliminate the `cur` array entirely. Update `prev` in-place.

```cpp
for(int ind = 1; ind < n; ind++) {
    for(int cap = W; cap >= 0; cap--) {   // ← RIGHT TO LEFT!
        int notPick = prev[cap];

        int pick = 0;
        if(wt[ind] <= cap)
            pick = val[ind] + prev[cap - wt[ind]];

        prev[cap] = max(pick, notPick);   // overwrite prev in-place
    }
}
```

### Why Right to Left? — The Deep Explanation

When we compute `prev[cap]` for the current item `ind`, the pick case reads:
```
prev[cap - wt[ind]]
```

`cap - wt[ind] < cap` — it reads from a **SMALLER** index.

**Left to Right (WRONG):**

If we iterate `cap` from 0 → W:
- When we reach `cap = 5`, we write `prev[5] = new value`
- Later when we reach `cap = 8`, and item weight = 3:
  - We read `prev[8 - 3] = prev[5]`
  - But `prev[5]` was ALREADY OVERWRITTEN to the new-item value!
  - We're reading the current item's updated value, not the previous item's

This means we'd be picking the SAME item twice — violating the 0/1 (each item once) constraint!

**Example of the bug:**

```
W=6, one item with wt=3, val=10

prev (before): [0, 0, 0, 10, 10, 10, 10]   (item 0 base)

Processing new item (wt=3, val=10) LEFT TO RIGHT:
  cap=3: prev[3] = max(notPick=prev[3]=10, pick=10+prev[0]=10) = 10
  cap=4: prev[4] = max(10, 10+prev[1]=10) = 10
  cap=5: prev[5] = max(10, 10+prev[2]=10) = 10
  cap=6: pick=10+prev[3] ← but prev[3] was UPDATED to use this item!
         = 10+10=20 ← WRONG! We used this item twice (weight 3+3=6)
```

**Right to Left (CORRECT):**

```
Processing new item (wt=3, val=10) RIGHT TO LEFT:
  cap=6: prev[6] = max(10, 10+prev[3]=10+10=20) = 20
         ↑ prev[3] still has OLD value (not yet overwritten) ✅
  cap=5: prev[5] = max(10, 10+prev[2]=10) = 10
  cap=4: prev[4] = max(10, 10+prev[1]=10) = 10
  cap=3: prev[3] = max(10, 10+prev[0]=10) = 10

Result: prev[6] = 20, but this is correct ONLY if we have two items
  both with wt=3. With one item: prev[6] should be 10.
  → RIGHT TO LEFT also gives 20 for cap=6 with one item!

Wait — let me reconsider. The point is: with 0/1 knapsack,
if we have item {wt=3,val=10} AND we had prev[6]=10 from item0,
then max(10, 10+prev[3]) = max(10, 20) = 20.
But prev[3] = 10 (from item 0, wt=3), NOT from taking this same item.
So 20 = take item0 (wt=3) + this item (wt=3) = two different items.
That's CORRECT!

The bug only occurs when prev[cap - wt] was already updated
to INCLUDE the current item. Right-to-left ensures prev[cap-wt]
still holds the OLD value (from previous row) since cap-wt < cap
and we process cap from high to low.
```

---

### The Critical Principle: Old Values Must Remain Until You Pass Them

```
Iteration right-to-left: cap = W, W-1, W-2, ..., 0

When processing cap:
  We READ  prev[cap - wt]      ← smaller index
  We WRITE prev[cap]           ← current index

Since cap-wt < cap, and we process cap from HIGH to LOW:
  When we write prev[cap], all indices SMALLER than cap are still UNMODIFIED
  → prev[cap-wt] still holds the previous item's value ✅

If we went LEFT to RIGHT (cap from 0 to W):
  When we write prev[cap], all indices SMALLER than cap were already UPDATED
  → prev[cap-wt] holds the CURRENT item's value ← double-counting ❌
```

---

### Visual: Left-to-Right vs Right-to-Left

```
Array: [_, _, _, _, _, _, _]  (indices 0..6)
New item: wt=3

LEFT TO RIGHT (WRONG):
cap=0: write prev[0]
cap=1: write prev[1]
cap=2: write prev[2]
cap=3: WRITE prev[3] ← updated!
cap=4: write prev[4]
cap=5: write prev[5]
cap=6: read prev[6-3]=prev[3] ← STALE! Already updated!
       → uses current item's contribution at index 3
       → picks this item TWICE ❌

RIGHT TO LEFT (CORRECT):
cap=6: read prev[6-3]=prev[3] ← FRESH! Not yet updated!
       → correctly uses previous item's value at index 3
       → picks this item once ✅
cap=5: write prev[5]
...
cap=3: WRITE prev[3] ← updated (but already used above ✅)
```

---

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × W)` | Same loops |
| **Space** | `O(W)` | Single array — optimal! |

---

## 9. All Approaches Compared

| Approach | Time | Space | Key Idea |
|---|---|---|---|
| **Recursion (v1 with score)** | `O(2ⁿ)` | `O(n)` | 3-param state, hard to memoize |
| **Recursion (v2 standard)** | `O(2ⁿ)` | `O(n)` | 2-param state, clean |
| **Memoization** | `O(n×W)` | `O(n×W+n)` | Cache `dp[ind][cap]` |
| **Tabulation** | `O(n×W)` | `O(n×W)` | Fill row by row |
| **Space Opt (2 arrays)** | `O(n×W)` | `O(2W)` | Keep only prev row |
| **Super Space Opt (1 array)** | `O(n×W)` | `O(W)` | In-place R→L update |

---

## 10. 0/1 Knapsack vs Subset Sum Family

| | Subset Sum | 0/1 Knapsack |
|---|---|---|
| **Operator** | `\|\|` / `+` | `max` |
| **Pick cost** | element value (sum) | element weight (constraint) |
| **Pick gain** | nothing (just sum tracks it) | `val[ind]` |
| **Base case** | `dp[i][0]=true` | `dp[0][cap]=val[0]` if fits |
| **Answer** | `dp[n][target]` | `dp[n-1][W]` |

The structural similarity is deep — both use the same pick/not-pick DP template, just with different semantics for what's being tracked and optimized.
