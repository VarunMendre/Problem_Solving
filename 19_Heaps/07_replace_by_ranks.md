# Replace Elements by Ranks

---

## 1. Brute Force — Count Smaller Elements

### Approach

For each element `arr[i]`, its **rank** = number of elements strictly smaller than it + number of equal elements that appeared before it (to handle duplicates with stable ranking).

Iterate over the entire array for every element to count this.

```cpp
class Solution {
public:
    void replaceWithRank(vector<int>& arr) {
        int n = arr.size();
        vector<int> ans(n);

        for(int i = 0; i < n; i++) {
            int rank = 0;

            for(int j = 0; j < n; j++) {
                if(arr[j] < arr[i])
                    rank++;                     // strictly smaller → contributes to rank
                else if(arr[j] == arr[i] && j < i)
                    rank++;                     // equal but appeared earlier → contributes
            }

            ans[i] = rank;                      // rank is 0-indexed here
        }

        arr = ans;
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N²)` | For each of N elements, scan all N elements to compute rank |
| **Space** | `O(N)` | `ans` vector stores the result |

---

---

## 2. Optimal — Sort with Index Tracking

### Approach

The rank of an element in sorted order is simply its **position in the sorted array** (0-indexed).

1. Create pairs of `{value, original_index}` for every element
2. Sort the pairs by value → the position `i` in sorted order = rank of that element
3. Write `rank = i` back to the original index: `arr[v[i].second] = i`

This works because after sorting, the first element (index 0) is the smallest (rank 0), the second (index 1) is the next smallest (rank 1), and so on.

```cpp
void convert(int arr[], int n) {
    // Step 1: pair each value with its original index
    vector<pair<int,int>> v;
    for(int i = 0; i < n; i++) {
        v.push_back({arr[i], i});
    }

    // Step 2: sort by value → sorted position = rank
    sort(v.begin(), v.end());

    // Step 3: assign rank back to original position
    for(int i = 0; i < n; i++) {
        arr[v[i].second] = i;       // element at original index gets rank i
    }
}
```

---

### Dry Run

```
arr = [20, 15, 26, 2, 98, 6]
```

**Step 1 — Pair with indices:**
```
v = [(20,0), (15,1), (26,2), (2,3), (98,4), (6,5)]
```

**Step 2 — Sort by value:**
```
v = [(2,3), (6,5), (15,1), (20,0), (26,2), (98,4)]
```

**Step 3 — Write rank back:**
```
i=0: arr[3] = 0   (2 is rank 0)
i=1: arr[5] = 1   (6 is rank 1)
i=2: arr[1] = 2   (15 is rank 2)
i=3: arr[0] = 3   (20 is rank 3)
i=4: arr[2] = 4   (26 is rank 4)
i=5: arr[4] = 5   (98 is rank 5)

arr = [3, 2, 4, 0, 5, 1] ✅
```

---

## 3. Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | `O(N)` to build pairs + `O(N log N)` to sort + `O(N)` to write back → dominated by sort |
| **Space** | `O(N)` | Vector of pairs stores all N elements |

---

---

## 4. Brute vs Optimal

| | Brute | Optimal |
|---|---|---|
| **Time** | `O(N²)` | `O(N log N)` |
| **Space** | `O(N)` | `O(N)` |
| **Key idea** | Count smaller elements per item | Sort once, use position as rank |
| **Handles duplicates?** | ✅ (via `j < i` check) | ✅ (stable sort preserves order) |
