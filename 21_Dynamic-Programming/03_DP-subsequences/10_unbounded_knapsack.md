# Unbounded Knapsack

---

## 1. Problem Statement

Given `n` items each with weight `wt[i]` and value `val[i]`, and a knapsack of capacity `W`, find the **maximum value** you can carry. Each item can be used **unlimited times** (unbounded).

```
W=8
val = [1, 4, 5]
wt  = [1, 3, 4]

Options:
  {item1, item1}: 3+3=6 weight, 4+4=8 value
  {item1, item2}: 3+4=7 weight, 4+5=9 value
  {item2, item2}: 4+4=8 weight, 5+5=10 value ← max

Answer: 10
```

---

## 2. The One Line That Separates 0/1 from Unbounded

```
0/1 Knapsack:   pick = val[ind] + f(ind-1, cap-wt[ind])  ← move to ind-1
Unbounded:      pick = val[ind] + f(ind,   cap-wt[ind])  ← stay at ind
```

Staying at `ind` means the same item can be picked again on the next call. That's the entire structural difference.

---

## 3. Base Case: `capacity / wt[0] * val[0]`

```cpp
if(ind == 0)
    return (capacity / wt[0]) * val[0];
```

With only item 0 available (unbounded):
- How many times can we fit `wt[0]` into `capacity`? → `capacity / wt[0]` times (integer division)
- Each fit gives `val[0]`
- Total = `(capacity / wt[0]) * val[0]`

```
cap=7, wt[0]=3, val[0]=4:
  7/3 = 2 (integer) → take item 0 twice → 2×4 = 8
  Remaining capacity 7-6=1 < wt[0]=3 → wasted

cap=6, wt[0]=3, val[0]=4:
  6/3 = 2 → 2×4 = 8

cap=2, wt[0]=3, val[0]=4:
  2/3 = 0 → 0×4 = 0 (can't fit even once)
```

This is different from 0/1 knapsack where the base case was:
```cpp
// 0/1 base:
if(wt[0] <= cap) return val[0];   // either take once or not at all
else return 0;
```

In unbounded, we can take item 0 as many times as it fits.

---

## 4. Approach 1 — Pure Recursion

```cpp
class Solution {
private:
    int f(int ind, int capacity, vector<int>& val, vector<int>& wt) {
        if(ind == 0)
            return (capacity / wt[0]) * val[0];

        int notPick = f(ind-1, capacity, val, wt);
        int pick    = 0;

        if(wt[ind] <= capacity)
            pick = val[ind] + f(ind, capacity - wt[ind], val, wt);  // stay at ind

        return max(notPick, pick);
    }

public:
    int knapSack(vector<int>& val, vector<int>& wt, int capacity) {
        int n = wt.size();
        return f(n-1, capacity, val, wt);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(capacity^n)` rough | Stay at ind → can take item `capacity/wt` times |
| **Space** | `O(capacity)` | Stack depth up to `capacity/min_wt` |

---

## 5. Approach 2 — Memoization

```cpp
class Solution {
private:
    int f(int ind, int capacity, vector<int>& val, vector<int>& wt,
          vector<vector<int>>& dp) {
        if(ind == 0)
            return (capacity / wt[0]) * val[0];

        if(dp[ind][capacity] != -1)
            return dp[ind][capacity];

        int notPick = f(ind-1, capacity, val, wt, dp);
        int pick    = 0;

        if(wt[ind] <= capacity)
            pick = val[ind] + f(ind, capacity - wt[ind], val, wt, dp);

        return dp[ind][capacity] = max(notPick, pick);
    }

public:
    int knapSack(vector<int>& val, vector<int>& wt, int capacity) {
        int n = wt.size();
        vector<vector<int>> dp(n, vector<int>(capacity+1, -1));
        return f(n-1, capacity, val, wt, dp);
    }
};
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × capacity)` | n×capacity states, O(1) each |
| **Space** | `O(n × capacity) + O(capacity)` | dp + stack |

---

## 6. Approach 3 — Tabulation

```cpp
class Solution {
public:
    int knapSack(vector<int>& val, vector<int>& wt, int capacity) {
        int n = wt.size();
        vector<vector<int>> dp(n, vector<int>(capacity+1, 0));

        // Base: row 0 — only item 0, take as many times as it fits
        for(int cap = 0; cap <= capacity; cap++)
            dp[0][cap] = (cap / wt[0]) * val[0];

        for(int ind = 1; ind < n; ind++) {
            for(int cap = 0; cap <= capacity; cap++) {
                int notPick = dp[ind-1][cap];
                int pick    = 0;

                if(wt[ind] <= cap)
                    pick = val[ind] + dp[ind][cap - wt[ind]];  // same row (unbounded)

                dp[ind][cap] = max(pick, notPick);
            }
        }

        return dp[n-1][capacity];
    }
};
```

### `dp[ind][cap - wt[ind]]` — Same Row Read

The pick case reads `dp[ind][...]` — current row, not previous row. This is the unbounded property: using item `ind` and then potentially using it again is captured by reading from the currently-being-filled row.

Left-to-right iteration is correct here (and intentional) because reading `dp[ind][cap - wt[ind]]` where `cap - wt[ind] < cap` reads an ALREADY-COMPUTED current-row entry, which correctly includes item `ind` being picked multiple times.

### Tabulation Trace (val=[1,4,5], wt=[1,3,4], W=8)

```
Row 0 (wt[0]=1, val[0]=1): dp[0][cap] = cap/1 * 1 = cap
  [0, 1, 2, 3, 4, 5, 6, 7, 8]

Row 1 (wt[1]=3, val[1]=4):
  cap=0: notPick=0, take(3>0)=0 → 0
  cap=1: notPick=1, take=0 → 1
  cap=2: notPick=2, take=0 → 2
  cap=3: notPick=3, take=4+dp[1][0]=4 → max(3,4)=4
  cap=4: notPick=4, take=4+dp[1][1]=5 → max(4,5)=5
  cap=5: notPick=5, take=4+dp[1][2]=6 → max(5,6)=6
  cap=6: notPick=6, take=4+dp[1][3]=8 → max(6,8)=8   (two item1s!)
  cap=7: notPick=7, take=4+dp[1][4]=9 → max(7,9)=9
  cap=8: notPick=8, take=4+dp[1][5]=10 → max(8,10)=10

Row 2 (wt[2]=4, val[2]=5):
  cap=4: notPick=dp[1][4]=5, take=5+dp[2][0]=5 → max(5,5)=5
  cap=8: notPick=dp[1][8]=10, take=5+dp[2][4]=10 → max(10,10)=10

dp[2][8] = 10 ✅ (two item2s: 4+4=8 weight, 5+5=10 value)
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × capacity)` | Two nested loops |
| **Space** | `O(n × capacity)` | 2D dp array |

---

## 7. Approach 4 — Space Optimization (Two Arrays)

```cpp
class Solution {
public:
    int knapSack(vector<int>& val, vector<int>& wt, int capacity) {
        int n = wt.size();
        vector<int> prev(capacity+1, 0);

        for(int cap = 0; cap <= capacity; cap++)
            prev[cap] = (cap / wt[0]) * val[0];

        for(int ind = 1; ind < n; ind++) {
            vector<int> cur(capacity+1, 0);

            for(int cap = 0; cap <= capacity; cap++) {
                int notPick = prev[cap];
                int pick    = 0;

                if(wt[ind] <= cap)
                    pick = val[ind] + cur[cap - wt[ind]];  // cur not prev! (unbounded)

                cur[cap] = max(pick, notPick);
            }

            prev = cur;
        }

        return prev[capacity];
    }
};
```

### `cur[cap - wt[ind]]` Not `prev[cap - wt[ind]]`

`cur` = currently being filled (item `ind`). Reading `cur[cap - wt[ind]]` = "best value using items `0..ind` with smaller cap." Since item `ind` is included in `cur`, this allows picking item `ind` multiple times.

Reading `prev` instead would only allow picking item `ind` once (0/1 knapsack behavior).

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × capacity)` | Same loops |
| **Space** | `O(2 × capacity)` = `O(capacity)` | `prev` + `cur` |

---

## 8. Approach 5 — Super Space Optimization (One Array, Left to Right)

```cpp
class Solution {
public:
    int knapSack(vector<int>& val, vector<int>& wt, int capacity) {
        int n = wt.size();
        vector<int> prev(capacity+1, 0);

        for(int cap = 0; cap <= capacity; cap++)
            prev[cap] = (cap / wt[0]) * val[0];

        for(int ind = 1; ind < n; ind++) {
            for(int cap = 0; cap <= capacity; cap++) {   // LEFT TO RIGHT ← key!
                int notPick = prev[cap];
                int pick    = 0;

                if(wt[ind] <= cap)
                    pick = val[ind] + prev[cap - wt[ind]];  // reads already-updated prev!

                prev[cap] = max(pick, notPick);
            }
        }

        return prev[capacity];
    }
};
```

### Why LEFT TO RIGHT Works Here

In 0/1 knapsack super space opt, we went **right to left** to prevent reusing the same item. Here we go **left to right** — and that's EXACTLY what we want.

When we process `cap` left-to-right:
1. We WRITE `prev[cap]` with the updated value for item `ind`
2. Later, when processing a larger `cap'` where `cap' - wt[ind] = cap`:
   - We READ `prev[cap' - wt[ind]] = prev[cap]` which was ALREADY UPDATED to include item `ind`
   - → We're allowed to pick item `ind` again! ✅

This is the in-place implementation of reading `cur[cap - wt[ind]]` — by the time we need `prev[cap - wt[ind]]`, it already contains the value "with item ind possibly used" because we updated it earlier in the same pass.

```
Example: wt[ind]=3, val[ind]=4, capacity=9

prev (before): [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]  (from previous ind)

LEFT TO RIGHT update:
  cap=0,1,2: no change (3 > cap)
  cap=3: prev[3] = max(3, 4+prev[0]) = max(3,4) = 4
  cap=4: prev[4] = max(4, 4+prev[1]) = max(4,5) = 5
  cap=5: prev[5] = max(5, 4+prev[2]) = max(5,6) = 6
  cap=6: prev[6] = max(6, 4+prev[3]=4+4=8) = 8  ← prev[3] was just updated to 4!
         means: took item ind at cap=3 (prev[3]=4), and took it AGAIN at cap=6 ✅
  cap=9: prev[9] = max(9, 4+prev[6]=4+8=12) = 12 ← three times! ✅
```

### Left-to-Right vs Right-to-Left — The Complete Picture

| | Direction | Reads | Effect |
|---|---|---|---|
| **0/1 Knapsack** | Right → Left | Old `prev` values (not yet updated) | Each item used at most once |
| **Unbounded Knapsack** | Left → Right | Already-updated `prev` values | Same item can be used multiple times |

The direction of iteration is what switches between 0/1 and unbounded behavior. The same array, the same formula — only the loop direction changes.

---

## 9. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(capacity^n)` | `O(capacity)` | Unbounded take |
| **Memoization** | `O(n × W)` | `O(n × W + W)` | |
| **Tabulation** | `O(n × W)` | `O(n × W)` | Same-row read (L→R) |
| **Space Opt** | `O(n × W)` | `O(W)` | `cur[...]` for take |
| **Super Space Opt** | `O(n × W)` | `O(W)` | One array, L→R |

---

## 10. 0/1 vs Unbounded Knapsack — Complete Side-by-Side

| | 0/1 Knapsack | Unbounded Knapsack |
|---|---|---|
| **Item reuse** | Once | Unlimited |
| **Recursion take** | `f(ind-1, cap-wt)` | `f(ind, cap-wt)` |
| **Tabulation take** | `dp[ind-1][cap-wt]` | `dp[ind][cap-wt]` |
| **Space opt take** | `prev[cap-wt]` | `cur[cap-wt]` |
| **Super space direction** | Right → Left | Left → Right |
| **Base case** | `val[0]` if `wt[0]<=cap` | `(cap/wt[0])*val[0]` |
| **Recursion SC** | `O(n)` | `O(capacity)` |
