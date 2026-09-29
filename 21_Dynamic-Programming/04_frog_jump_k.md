# Frog Jump with K Steps

---

## 1. Problem Statement

Same as the classic frog jump, but now the frog can jump **1 to K steps** instead of just 1 or 2. From stair `i`, the frog can jump to any stair `i+1, i+2, ..., i+K`. The cost is the absolute height difference. Find the minimum total cost to reach the last stair.

```
heights = [10, 30, 40, 50, 20], K=3

From stair 0, can jump to 1, 2, or 3.
From stair 1, can jump to 2, 3, or 4.
...

Optimal path: 0→2→4: |40-10|+|20-40| = 30+20 = 50
Or: 0→3→4: |50-10|+|20-50| = 40+30 = 70
Or: 0→1→4: |30-10|+|20-30| = 20+10 = 30 ✅

Answer: 30
```

---

## 2. How This Extends Frog Jump (K=2)

The classic frog jump had exactly 2 choices at each step (jump 1 or jump 2). That allowed a simple `min(left, right)`.

With K choices, we need a **loop over all valid predecessors**:

```
f(i) = min over j in [1..K] of:
          f(i-j) + |heights[i] - heights[i-j]|     (if i-j >= 0)
```

The recurrence is the same idea — look back — but now you look back `K` positions instead of just 2.

---

## 3. Approach 1 — Pure Recursion

### Intuition

`f(ind)` = minimum cost to reach stair `ind` from stair `0`.

For each stair `ind`, try every valid jump size `j` from 1 to K:
- Come from stair `ind-j` (if it exists)
- Cost = `f(ind-j) + |heights[ind] - heights[ind-j]|`
- Track the minimum across all valid jumps

```cpp
class Solution {
public:
    int f(int ind, int k, vector<int>& heights) {
        if(ind == 0) return 0;   // at start, cost = 0

        int minJumps = INT_MAX;

        for(int j = 1; j <= k; j++) {
            int prevInd = ind - j;

            if(prevInd >= 0) {
                int difference = abs(heights[ind] - heights[prevInd]);
                int cost = f(prevInd, k, heights) + difference;
                minJumps = min(minJumps, cost);
            }
        }

        return minJumps;
    }

    int frogJump(vector<int>& heights, int k) {
        int n = heights.size();
        return f(n-1, k, heights);
    }
};
```

### Recursion Tree (K=3, n=4)

```
f(3)
├── f(2) + cost(3,2)
│   ├── f(1) + cost(2,1)
│   │   ├── f(0) + cost(1,0)
│   │   ├── (f(-1) invalid)
│   └── f(0) + cost(2,0)
├── f(1) + cost(3,1)
│   ├── f(0) + cost(1,0)   ← RECOMPUTED
└── f(0) + cost(3,0)
```

**Overlapping subproblems:** `f(1)` and `f(0)` computed multiple times. With K large, this is severely exponential.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(Kⁿ)` | Each call has K branches |
| **Space** | `O(n)` | Recursion stack depth |

---

## 4. Approach 2 — Memoization (Top-Down DP)

Same recursion + `dp[]` cache:

```cpp
class Solution {
public:
    int f(int ind, int k, vector<int>& dp, vector<int>& heights) {
        if(ind == 0) return 0;

        if(dp[ind] != -1)
            return dp[ind];   // already computed

        int minJumps = INT_MAX;

        for(int j = 1; j <= k; j++) {
            int prevInd = ind - j;

            if(prevInd >= 0) {
                int difference = abs(heights[ind] - heights[prevInd]);
                int cost = f(prevInd, k, dp, heights) + difference;
                minJumps = min(minJumps, cost);
            }
        }

        return dp[ind] = minJumps;   // cache result
    }

    int frogJump(vector<int>& heights, int k) {
        int n = heights.size();
        vector<int> dp(n, -1);
        return f(n-1, k, dp, heights);
    }
};
```

### What Changes vs Recursion

```
Without memo: f(2) called multiple times → recomputed each time
With memo:    f(2) computed once → dp[2] = result → future calls return dp[2] instantly
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × K)` | n subproblems × K iterations each |
| **Space** | `O(n) + O(n)` | dp[] + recursion stack |

---

## 5. Approach 3 — Tabulation (Bottom-Up DP)

No recursion. Fill `dp[]` from `dp[0]` upward:

```cpp
class Solution {
public:
    int frogJump(vector<int>& heights, int k) {
        int n = heights.size();
        vector<int> dp(n, 0);   // dp[0]=0 by default

        for(int index = 1; index <= n-1; index++) {
            int minJumps = INT_MAX;

            for(int jump = 1; jump <= k; jump++) {
                int prevIndex = index - jump;

                if(prevIndex >= 0) {
                    int difference = abs(heights[index] - heights[prevIndex]);
                    int newJump = dp[prevIndex] + difference;
                    minJumps = min(minJumps, newJump);
                }
            }

            dp[index] = minJumps;
        }

        return dp[n-1];
    }
};
```

### Tabulation Trace (heights=[10,30,40,50,20], K=3)

```
dp[0] = 0

index=1:
  jump=1: prev=0, cost=|30-10|=20, dp[0]+20=20
  (jump=2,3: prevIndex<0)
  dp[1] = 20

index=2:
  jump=1: prev=1, cost=|40-30|=10, dp[1]+10=30
  jump=2: prev=0, cost=|40-10|=30, dp[0]+30=30
  (jump=3: prevIndex<0)
  dp[2] = min(30,30) = 30

index=3:
  jump=1: prev=2, cost=|50-40|=10, dp[2]+10=40
  jump=2: prev=1, cost=|50-30|=20, dp[1]+20=40
  jump=3: prev=0, cost=|50-10|=40, dp[0]+40=40
  dp[3] = 40

index=4:
  jump=1: prev=3, cost=|20-50|=30, dp[3]+30=70
  jump=2: prev=2, cost=|20-40|=20, dp[2]+20=50
  jump=3: prev=1, cost=|20-30|=10, dp[1]+10=30   ← minimum!
  dp[4] = 30

Answer: dp[4] = 30 ✅
```

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × K)` | n positions × K jumps each |
| **Space** | `O(n)` | dp[] array |

---

## 6. Approach 4 — Circular DP Window (Space Optimization)

### The Key Observation

`dp[index]` depends on at most the last `K` entries: `dp[index-1], dp[index-2], ..., dp[index-K]`.

Instead of storing all `n` entries, we only need a **circular window of size K+1**.

### Why K+1 (Not K)?

We need entries at distances 1 through K from the current index. At index `i`, we access `dp[i-1]` through `dp[i-K]`. That's K entries. But we also need to write `dp[i]` without overwriting an entry we still need. So the window size is `K+1` to safely hold all K predecessors AND the current write target.

### Why Read Before Write?

**Critical:** we must read ALL predecessor values before writing `dp[currentSlot]`.

```
windowSize = K+1 = 4 (for K=3)
index=4: currentSlot = 4%4 = 0

Predecessors:
  index-1=3: slot 3%4=3 → read dp[3]
  index-2=2: slot 2%4=2 → read dp[2]
  index-3=1: slot 1%4=1 → read dp[1]

Then write: dp[0] = result   ← slot 0, which we haven't read (dp[0] isn't needed for index=4 with K=3)
```

The modulo arithmetic ensures: when we write `dp[index % windowSize]`, the slot we're overwriting is always the entry at `index - windowSize = index - (K+1)`, which is too old to be needed (any jump from `index` only looks back at most K steps, not K+1).

### Code

```cpp
class Solution {
public:
    int frogJump(vector<int>& heights, int k) {
        int n = heights.size();
        int windowSize = k + 1;
        vector<int> dp(windowSize, 0);   // circular window, all initialized to 0

        for(int index = 1; index < n; index++) {
            int current = INT_MAX;

            // READ all predecessors FIRST (before any write)
            for(int jump = 1; jump <= k; jump++) {
                int previousIndex = index - jump;

                if(previousIndex >= 0) {
                    int previousSlot = previousIndex % windowSize;
                    int difference = abs(heights[index] - heights[previousIndex]);
                    int jumpEnergy = dp[previousSlot] + difference;
                    current = min(current, jumpEnergy);
                }
            }

            // WRITE after all reads (to avoid overwriting needed values)
            int currentSlot = index % windowSize;
            dp[currentSlot] = current;
        }

        return dp[(n-1) % windowSize];
    }
};
```

### Circular Window Visualization (K=3, windowSize=4)

```
Indices: 0  1  2  3  4  5  6  7
Slots:   0  1  2  3  0  1  2  3    (index % 4)

When writing index=4 (slot=0):
  We read slots: 3(index=3), 2(index=2), 1(index=1)
  We write slot: 0 (overwrites dp[0]=dp[index=0], which is no longer needed!)

When writing index=5 (slot=1):
  We read slots: 0(index=4), 3(index=3), 2(index=2)
  We write slot: 1 (overwrites dp[1]=dp[index=1], no longer needed)
```

The window slides forward, always keeping the last K+1 values, discarding values older than K steps ago.

### Why Not Space-Optimize to `O(1)` (Two Variables)?

Unlike the K=2 case (where we only needed `prev1` and `prev2`), with general K we need up to K previous values. Two variables aren't enough. The circular window is the minimal storage: `O(K+1) = O(K)`.

### Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(n × K)` | Same loop structure |
| **Space** | `O(K)` | Circular window of size K+1 |

---

## 7. All Approaches Compared

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Recursion** | `O(Kⁿ)` | `O(n)` | Exponential — unusable for large K,n |
| **Memoization** | `O(n × K)` | `O(2n)` | Top-down, clean code |
| **Tabulation** | `O(n × K)` | `O(n)` | Bottom-up, no recursion overhead |
| **Circular Window** | `O(n × K)` | `O(K)` | Best space, clever modular indexing |

> The circular window approach is the theoretical optimum in space — you can't do better than `O(K)` since you always need the last K dp values.

---

## 8. Frog Jump (K=2) vs Frog Jump (K Steps)

| | K=2 | General K |
|---|---|---|
| **Choices per stair** | 2 (fixed) | K (variable) |
| **Recurrence** | `min(dp[i-1]+cost, dp[i-2]+cost)` | `min over j=1..K of dp[i-j]+cost` |
| **Space optimize to O(1)?** | ✅ Yes (2 variables) | ❌ Need O(K) |
| **Time** | `O(n)` | `O(n × K)` |
| **Space (optimal)** | `O(1)` | `O(K)` |

The general K case is strictly harder — the inner loop over K jumps adds a factor of K to both time and the minimum required space.
