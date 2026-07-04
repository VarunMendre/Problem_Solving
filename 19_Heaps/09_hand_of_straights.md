# Hand of Straights

---

## 1. Intuition / Approach

We need to check if all cards can be grouped into sets of `groupSize` **consecutive** integers.

### Greedy Strategy — Always Start from the Smallest

At each step, take the **smallest remaining card** and try to form a consecutive group of length `groupSize` starting from it.

**Why greedy works:**  
The smallest card can only ever be the **start** of a group — it can never be in the middle or end of a valid consecutive group that hasn't been formed yet. So if we can't form a group starting from the smallest, it's impossible.

### Why `map` (not `unordered_map`)?

`map` keeps keys in **sorted order**, so `mpp.begin()->first` always gives us the current smallest card in `O(1)`. This is essential for the greedy strategy.

---

### Algorithm

1. If `n % groupSize != 0` → impossible to partition evenly → return `false`
2. Build a frequency map of all cards
3. While the map is not empty:
   - Get the smallest card `curr = mpp.begin()->first`
   - Try to form a group: `curr, curr+1, curr+2, ..., curr+groupSize-1`
   - For each card in the group, decrement its frequency; if it hits 0, remove it from the map
   - If any card in the group is missing (`freq == 0`) → return `false`
4. If all groups formed successfully → return `true`

---

### Dry Run

```
hand = [1,2,3,6,2,3,4,7,8], groupSize = 3
```

**Build frequency map:**
```
mpp = {1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}
```

**Iteration 1:** curr = 1
```
Group: 1, 2, 3
  mpp[1]-- → 0 → erase  → mpp = {2:1, 3:2, 4:1, 6:1, 7:1, 8:1}
  mpp[2]-- → 1            mpp = {2:1, 3:1, 4:1, 6:1, 7:1, 8:1}
  mpp[3]-- → 1            mpp = {2:1, 3:1, 4:1, 6:1, 7:1, 8:1}
```

Wait — let me retrace:
```
mpp = {1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}
Group: 1,2,3 → mpp[1]=0(erase), mpp[2]=1, mpp[3]=1
mpp = {2:1, 3:1, 4:1, 6:1, 7:1, 8:1}
```

**Iteration 2:** curr = 2
```
Group: 2,3,4 → mpp[2]=0(erase), mpp[3]=0(erase), mpp[4]=0(erase)
mpp = {6:1, 7:1, 8:1}
```

**Iteration 3:** curr = 6
```
Group: 6,7,8 → mpp[6]=0(erase), mpp[7]=0(erase), mpp[8]=0(erase)
mpp = {}
```

Map empty → return `true` ✅

---

**Failure case:**
```
hand = [1,2,3,4,5], groupSize = 4
n=5, 5 % 4 = 1 ≠ 0 → return false immediately ✅
```

```
hand = [1,2,4,5,6], groupSize = 3
mpp = {1:1, 2:1, 4:1, 5:1, 6:1}
curr = 1: try group 1,2,3 → mpp[3] == 0 → return false ✅
```

---

## 2. Code

```cpp
class Solution {
public:
    bool isNStraightHand(vector<int>& hand, int groupSize) {
        int n = hand.size();

        // Total cards must be divisible by groupSize
        if(n % groupSize)
            return false;

        // Build frequency map — sorted by key for greedy smallest-first access
        map<int, int> mpp;
        for(int& it : hand)
            mpp[it]++;      // O(N log N)

        while(!mpp.empty()) {
            // Always start group from the smallest remaining card
            int curr = mpp.begin()->first;

            // Try to form consecutive group: curr, curr+1, ..., curr+groupSize-1
            for(int i = 0; i < groupSize; i++) {
                if(mpp[curr + i] == 0)
                    return false;       // required card missing → impossible

                mpp[curr + i]--;        // use one card of this value

                if(mpp[curr + i] < 1)
                    mpp.erase(curr + i); // remove exhausted cards from map
            }
        }

        return true;
    }
};
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Building the map costs `O(N log N)`; each card is inserted and removed from the map once, each operation `O(log N)` → processing all cards is `O(N log N)` |
| **Space** | `O(N)` | Map stores at most N distinct values |

> The `while` loop runs at most `N/groupSize` times (one per group), and each inner loop runs `groupSize` times → total inner iterations = N. Each map operation is `O(log N)` → overall `O(N log N)`.
