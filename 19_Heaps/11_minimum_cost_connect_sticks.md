# Minimum Cost to Connect Sticks

---

## 1. Intuition / Approach

Every time we combine two sticks, the cost is `x + y`. The combined stick of length `x + y` is then used in future combinations — so **longer sticks get counted in the cost multiple times**.

### Key Insight — Greedy: Always Combine the Two Shortest

To minimize total cost, we want the sticks that get reused the most (combined earliest) to be as **small as possible**.

Think of it like Huffman Encoding — elements combined earlier contribute to more future costs, so we should combine the smallest ones first.

**Example:**
```
sticks = [1, 2, 3, 4]

Greedy (smallest first):
  combine 1+2=3,  cost=3,   sticks=[3,3,4]
  combine 3+3=6,  cost=6,   sticks=[4,6]
  combine 4+6=10, cost=10,  sticks=[10]
  Total = 3+6+10 = 19 ✅

Non-greedy (largest first):
  combine 3+4=7,  cost=7,   sticks=[1,2,7]
  combine 2+7=9,  cost=9,   sticks=[1,9]
  combine 1+9=10, cost=10,  sticks=[10]
  Total = 7+9+10 = 26 ❌ (worse)
```

### Why Min Heap?

A **min heap** always gives us the two smallest sticks in `O(log N)`. After combining, we push the new stick back — it might be the next smallest for a future combination.

---

### Dry Run

```
arr = [4, 3, 2, 6], n = 4
pq (min heap) = [2, 3, 4, 6]
```

```
Iteration 1: pop 2, pop 3 → cost=5, totalCost=5,  push 5  → pq=[4,5,6]
Iteration 2: pop 4, pop 5 → cost=9, totalCost=14, push 9  → pq=[6,9]
Iteration 3: pop 6, pop 9 → cost=15,totalCost=29, push 15 → pq=[15]

pq.size()==1 → stop
return 29 ✅
```

---

## 2. Code

```cpp
#include <bits/stdc++.h>

long long int minimumCostToConnectSticks(vector<int>& arr) {
    // Min heap — always gives the two shortest sticks
    priority_queue<int, vector<int>, greater<int>> pq;

    long long totalCost = 0;

    // Build the heap
    for(int& el : arr)
        pq.push(el);

    // Keep combining the two smallest sticks until one remains
    while(pq.size() > 1) {
        int el1 = pq.top(); pq.pop();   // shortest stick
        int el2 = pq.top(); pq.pop();   // second shortest

        int cost = el1 + el2;           // cost of this combination
        totalCost += cost;

        pq.push(cost);                  // new stick goes back into the pool
    }

    return totalCost;
}
```

---

## 3. Time & Space Complexity

### Time Complexity — `O(N log N)`

| Step | Cost | Reason |
|---|---|---|
| Insert all N sticks into heap | `O(N log N)` | N pushes, each `O(log N)` |
| Combine sticks (N-1 iterations) | `O(N log N)` | Each iteration: 2 pops + 1 push = `O(log N)`; runs N-1 times |

**Total: `O(N log N) + O(N log N)` = `O(N log N)`**

> The loop runs exactly **N-1 times** because each iteration reduces the number of sticks by 1 (two removed, one added). Starting with N sticks → N-1 merges to get to 1.

---

### Space Complexity — `O(N)`

| Structure | Size | Reason |
|---|---|---|
| Priority queue | `O(N)` | Holds all N sticks initially; size reduces as merges happen |

> No other auxiliary structure grows with N.

---

## 4. Connection to Huffman Encoding

This problem is essentially **Huffman Encoding** — the same greedy algorithm is used to build an optimal prefix-free code. In both:
- Combine the two smallest elements
- The result participates in future combinations
- Greedy minimum ensures each element's "depth" (how many times it's counted) is minimized

The min heap is the standard data structure for both.
