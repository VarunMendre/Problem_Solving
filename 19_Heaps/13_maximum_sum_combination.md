# Top K Sum Pairs (Max Combinations)

---

## 1. Brute Force — Generate All Pairs + Sort

### Approach

1. Generate **all possible pair sums** `nums1[i] + nums2[j]` → `N²` total sums
2. Sort in descending order
3. Return first K elements

```cpp
class Solution {
public:
    vector<int> maxCombinations(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<int> allSums;

        // Generate all N² pair sums
        for(int i = 0; i < nums1.size(); i++) {
            for(int j = 0; j < nums2.size(); j++) {
                allSums.push_back(nums1[i] + nums2[j]);
            }
        }

        // Sort descending
        sort(allSums.begin(), allSums.end(), greater<int>());

        // Return top K
        return vector<int>(allSums.begin(), allSums.begin() + k);
    }
};
```

### Time & Space Complexity

|           | Complexity                      | Reason                                                |
| --------- | ------------------------------- | ----------------------------------------------------- |
| **Time**  | `O(N²  log N²)` = `O(N² log N)` | Generating all pairs `O(N²)` + sorting `O(N² log N²)` |
| **Space** | `O(N²)`                         | Stores all N² pair sums                               |

> Works correctly but completely wasteful — we compute all `N²` pairs just to return K of them.

---

---

## 2. Optimal — Max Heap + Visited Set

## 2.1. Intuition / Approach

Given two arrays `a` and `b`, find the **K largest possible sums** `a[i] + b[j]` (one element from each array).

### Brute Force Idea (Why We Don't Do It)

Generate all `N²` pairs, sort by sum, take top K → `O(N² log N)` time, `O(N²)` space. Too slow for large N.

---

### Optimal — Max Heap + Visited Set

**Key Observation:**  
If both arrays are sorted in **descending order**, the largest sum is always `a[0] + b[0]`. The next largest candidates can only come from:

- Moving `i` forward → `a[i+1] + b[j]`
- Moving `j` forward → `a[i] + b[j+1]`

This is exactly the same pattern as **merging K sorted arrays** or **finding Kth smallest in a sorted matrix**.

---

### Algorithm

1. Sort both arrays in **descending order**
2. Push `(a[0]+b[0], 0, 0)` into a **max heap** (sum, i, j)
3. Mark `(0,0)` as visited
4. Repeat K times:
   - Pop the max sum `(sum, i, j)`
   - Add `sum` to result
   - Push `(a[i+1]+b[j], i+1, j)` if not visited
   - Push `(a[i]+b[j+1], i, j+1)` if not visited
   - Mark both as visited

The **`visited` set** prevents pushing duplicate pairs — without it, `(i+1,j)` and `(i,j+1)` from different parent pairs could both try to push `(i+1,j+1)` twice.

---

### Dry Run

```
a = [4, 2, 1],  b = [3, 1, 0],  k = 3
After sorting descending: a=[4,2,1], b=[3,1,0]
```

```
Initial: pq=[(7,0,0)], visited={(0,0)}

--- k=3 (iteration 1) ---
pop (7,0,0) → result=[7]
  push (i+1=1,j=0): a[1]+b[0]=2+3=5 → pq=[(5,1,0)], visited+={(1,0)}
  push (i=0,j+1=1): a[0]+b[1]=4+1=5 → pq=[(5,1,0),(5,0,1)], visited+={(0,1)}

--- k=2 (iteration 2) ---
pop (5,1,0) → result=[7,5]   (tie broken arbitrarily by heap)
  push (i+1=2,j=0): a[2]+b[0]=1+3=4 → pq=[(5,0,1),(4,2,0)], visited+={(2,0)}
  push (i=1,j+1=1): a[1]+b[1]=2+1=3 → pq=[(5,0,1),(4,2,0),(3,1,1)], visited+={(1,1)}

--- k=1 (iteration 3) ---
pop (5,0,1) → result=[7,5,5]
  push (1,1): already visited ❌
  push (0,2): a[0]+b[2]=4+0=4 → push (4,0,2), visited+={(0,2)}

return [7, 5, 5] ✅
```

---

## 2.2. Code

```cpp
class Solution {
public:
    vector<int> topKSumPairs(vector<int>& a, vector<int>& b, int k) {
        int n = a.size();

        // Sort both descending so largest sums are at front
        sort(a.begin(), a.end(), greater<int>());
        sort(b.begin(), b.end(), greater<int>());

        // Max heap: {sum, i, j}
        priority_queue<tuple<int,int,int>> pq;

        // Track visited (i,j) pairs to avoid duplicates
        set<pair<int,int>> visited;

        // Seed with the globally largest pair
        pq.push({a[0]+b[0], 0, 0});
        visited.insert({0, 0});

        vector<int> result;

        while(k-- && !pq.empty()) {
            auto [sum, i, j] = pq.top();
            pq.pop();

            result.push_back(sum);

            // Next candidate: advance i
            if(i+1 < n && !visited.count({i+1, j})) {
                pq.push({a[i+1]+b[j], i+1, j});
                visited.insert({i+1, j});
            }

            // Next candidate: advance j
            if(j+1 < n && !visited.count({i, j+1})) {
                pq.push({a[i]+b[j+1], i, j+1});
                visited.insert({i, j+1});
            }
        }

        return result;
    }
};
```

---

## 2.3. Time & Space Complexity

### Time Complexity — `O(N log N + K log K)`

| Step              | Cost         | Reason                                                                    |
| ----------------- | ------------ | ------------------------------------------------------------------------- |
| Sort both arrays  | `O(N log N)` | Standard sort                                                             |
| K heap operations | `O(K log K)` | Each pop + up to 2 pushes; heap grows to at most `2K` → `O(log K)` per op |
| K set operations  | `O(K log K)` | Each insert/lookup in `set` is `O(log K)`                                 |

**Total: `O(N log N + K log K)`**

---

### Space Complexity — `O(K)`

| Structure     | Size   | Reason                                                                         |
| ------------- | ------ | ------------------------------------------------------------------------------ |
| Max heap `pq` | `O(K)` | At most 2 new pairs pushed per iteration → heap grows by at most K pairs total |
| `visited` set | `O(K)` | At most 2 entries added per iteration → K iterations → `O(K)` entries          |

---

## 2.4. Why the `visited` Set is Necessary

Without `visited`, consider this scenario:

```
Pop (a[0]+b[0], 0, 0)
  Push (1, 0) and (0, 1)

Pop (a[1]+b[0], 1, 0)
  Push (2, 0) and (1, 1) ← pushes (1,1)

Pop (a[0]+b[1], 0, 1)
  Push (1, 1) ← pushes (1,1) AGAIN → duplicate!
```

`(1,1)` would be pushed twice, leading to the same pair counted twice in results. The `visited` set prevents this.

## 3. Brute vs Optimal

|                  | Brute                    | Optimal                                             |
| ---------------- | ------------------------ | --------------------------------------------------- |
| **Time**         | `O(N² log N)`            | `O(N log N + K log K)`                              |
| **Space**        | `O(N²)`                  | `O(K)`                                              |
| **Key idea**     | Generate all pairs, sort | Lazily explore only promising candidates using heap |
| **When K << N²** | Wasteful                 | Huge win — only K iterations needed                 |

> For N=1000, K=10: Brute does `10⁶` operations, Optimal does roughly `10 × log(10)` ≈ 33 heap operations after sorting.
