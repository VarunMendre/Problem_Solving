# Top K Frequent Elements — Deep Dive: All Approaches

## Problem Recap
Given an array `nums` and an integer `k`, return the `k` most frequent elements.

---

# Approach 1: Brute Force (Sort by Frequency)

## Core Idea

The most straightforward idea: count how often every element appears, then **sort the unique elements by their frequency** in descending order, and simply take the first `k`. No clever tricks — just count, sort, slice.

## Step-by-Step Walkthrough

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> mpp;

        // Step 1: Count frequency of every element
        for (int& num : nums) {
            mpp[num]++;
        }

        // Step 2: Copy (value, freq) pairs into a vector so we can sort them
        vector<pair<int,int>> freqList; // {value, freq}
        for (auto& it : mpp) {
            freqList.push_back({it.first, it.second});
        }

        // Step 3: Sort by frequency, descending
        sort(freqList.begin(), freqList.end(),
             [](const pair<int,int>& a, const pair<int,int>& b) {
                 return a.second > b.second;
             });

        // Step 4: Take the first k elements
        vector<int> result;
        for (int i = 0; i < k; i++) {
            result.push_back(freqList[i].first);
        }

        return result;
    }
};
```

### Trace Example
`nums = [1,1,1,2,2,3]`, `k = 2`

1. **Frequency map:** `{1:3, 2:2, 3:1}`
2. **Convert to list:** `[(1,3), (2,2), (3,1)]`
3. **Sort descending by frequency:** `[(1,3), (2,2), (3,1)]` (already sorted in this case)
4. **Take first k=2:** `[1, 2]` ✅

## Complexity Breakdown

Let `n` = size of `nums`, `m` = number of **unique** elements (`m ≤ n`).

| Step                         | Time         | Why                                                        |
|-------------------------------|--------------|-------------------------------------------------------------|
| Build frequency map            | O(n)         | One pass, O(1) average hashmap insert                       |
| Copy map into a vector         | O(m)         | One entry per unique element                                |
| Sort the vector                | O(m log m)   | Comparison-based sort (e.g. `std::sort`, typically introsort) |
| Take first k                   | O(k)         | Simple slice                                                 |
| **Total Time**                 | **O(n + m log m) ≈ O(n log n)** | Worst case, all elements are unique so m = n |

**Space Complexity:**
- Frequency map: O(m) ≈ O(n)
- Vector of pairs for sorting: O(m) ≈ O(n)
- **Total: O(n)**

## Why This Is the Weakest Approach

The sort step fully orders **all `m` unique elements** by frequency — even though we only actually care about the top `k`. If `m` is large and `k` is small, we're doing a lot of unnecessary work: sorting the entire list just to throw away everything past index `k-1`.

This is exactly the inefficiency that Approaches 2 and 3 are designed to eliminate:
- The **heap** approach avoids fully sorting by only maintaining the top `k` candidates at any time.
- The **bucket sort** approach avoids comparison-based sorting altogether by exploiting the bounded range of frequency values.

## When Is This the Right Choice?

- When code simplicity/readability matters more than optimality (e.g., a quick script, or `k` is expected to be close to `m` anyway, in which case sorting everything isn't much extra work).
- As the natural **first solution to mention in an interview** — it shows you understand the problem correctly before you optimize.

## Common Pitfalls

- Forgetting to specify a **descending** comparator — `std::sort`'s default is ascending, so you'd get the *least* frequent elements instead of the most frequent.
- Not accounting for `k` possibly being larger than `m` (number of unique elements) — accessing `freqList[i]` out of bounds. (The problem constraints usually guarantee `k ≤ number of unique elements`, but it's worth a defensive check in production code.)

---

# Approach 2: Min-Heap of Size K

## Core Idea

Instead of sorting **all** unique elements by frequency (which does more work than needed), we maintain a **min-heap capped at size k**. The heap always holds our "current best k candidates." Whenever we see something better than the worst candidate in the heap, we swap it in.

Think of it like a leaderboard with only k slots — you don't need to fully rank everyone, you just need to make sure the top k are correct, and the heap does that cheaply.

## Why a MIN-Heap (not max-heap)?

This trips people up, so let's be precise:

- We want to **keep** the k elements with the **highest** frequency.
- We want to **discard** the element with the **lowest** frequency among our current candidates, the moment we find something better.
- A min-heap puts the smallest element at the top (`O(1)` access).
- So: if heap size exceeds k, we pop the top of a min-heap → we pop the **smallest frequency** → exactly the one we wanted to discard.

If we used a max-heap instead, the top would be the *largest* frequency, and popping would throw away our best candidate — the opposite of what we want.

## Step-by-Step Walkthrough

```cpp
class Solution {
public:
    typedef pair<int, int> P;   // {frequency, value}
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> mpp;

        // Step 1: Count frequency of every element
        for (int& num : nums) {
            mpp[num]++;
        }

        // Step 2: Min-heap ordered by frequency (pair.first)
        priority_queue<P, vector<P>, greater<P>> pq;

        for (auto& it : mpp) {
            auto value = it.first;
            auto freq  = it.second;

            pq.push({freq, value});

            if (pq.size() > k)   // heap grew too big
                pq.pop();        // remove smallest-frequency element
        }

        // Step 3: Whatever remains in the heap is our answer
        vector<int> result;
        while (!pq.empty()) {
            result.push_back(pq.top().second);
            pq.pop();
        }

        return result;
    }
};
```

### Trace Example
`nums = [1,1,1,2,2,3]`, `k = 2`

1. **Frequency map:** `{1:3, 2:2, 3:1}`
2. **Process map entries into heap (heap capped at size 2):**

| Insert       | Heap contents (min at top)         | Action                       |
|--------------|-------------------------------------|-------------------------------|
| push {3,1}   | `[{3,1}]`                            | size = 1, OK                  |
| push {2,2}   | `[{2,2}, {3,1}]`                     | size = 2, OK                  |
| push {1,3}   | `[{1,3}, {3,1}, {2,2}]`              | size = 3 > k → pop top `{1,3}` |
| after pop    | `[{2,2}, {3,1}]`                     | size = 2, OK                  |

3. **Final heap:** `{2,2}` and `{3,1}` → elements `2` and `1` → these are the top-2 frequent elements. ✅

Notice how element `3` (frequency 1) got pushed in but was immediately evicted because it was the weakest candidate once the heap exceeded size k.

## Complexity Breakdown

Let `n` = size of `nums`, `m` = number of **unique** elements (`m ≤ n`).

| Step                              | Time              | Why                                                                 |
|------------------------------------|-------------------|----------------------------------------------------------------------|
| Build frequency map                | O(n)              | One pass, O(1) average hashmap insert                                |
| Insert each of m entries into heap | O(m log k)        | Heap size is capped at k, so each push/pop costs O(log k), done m times |
| Extract k elements at the end      | O(k log k)        | Popping from a heap of size ≤ k, k times                             |
| **Total Time**                     | **O(n + m log k) ≈ O(n log k)** | Since m ≤ n, and log k dominates when k is meaningfully smaller than n |

**Space Complexity:**
- Frequency map: O(m) ≈ O(n)
- Heap: O(k)
- **Total: O(n)** (map dominates)

## When Is This the Right Choice?

- When **k is small relative to n** — the O(log k) factor is cheap, much cheaper than O(log n) from a full sort.
- When you're dealing with a **stream** of data or don't want to materialize a large auxiliary array (bucket sort needs an array of size n+1).
- It's the **generalizable** technique — the same heap pattern extends to problems like "k closest points," "k largest elements in a stream," merge k sorted lists, etc. Interviewers often want to see you reach for this pattern.

## Common Pitfalls

- Mixing up min-heap vs max-heap logic (explained above — this is the #1 mistake).
- Forgetting that `priority_queue<P, vector<P>, greater<P>>` in C++ is what makes it a **min**-heap (default `priority_queue` is a max-heap).
- The final result is **not necessarily in sorted order of frequency** — the heap is only guaranteed to *contain* the correct top-k elements, not to pop them in a particular order beyond "smallest frequency first." If the problem needs the result sorted by frequency, you'd need an extra step.

---

# Approach 3: Bucket Sort

## Core Idea

This is the key insight that unlocks true **O(n)** performance:

> **The frequency of any single element can never exceed `n`** (the total size of the array).

Since frequencies are bounded integers in the range `[0, n]`, we don't need comparison-based sorting (which costs at least O(m log m)) — we can use **counting/bucket sort**, placing elements directly into indexed "buckets" based on their frequency.

This is the same trick used in **Counting Sort**: when your sort key is a small bounded integer rather than an arbitrary comparable value, you can sort in linear time by using the key itself as an array index.

## Step-by-Step Walkthrough

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        int n = nums.size();

        unordered_map<int, int> mpp;
        for (int& num : nums) {
            mpp[num]++;
        }

        // bucket[i] = list of elements whose frequency is exactly i
        vector<vector<int>> bucket(n + 1);

        for (auto& it : mpp) {
            int element = it.first;
            int freq    = it.second;

            bucket[freq].push_back(element);
        }

        vector<int> result;

        // Walk buckets from highest frequency to lowest
        for (int i = n; i >= 0; i--) {
            if (bucket[i].empty())
                continue;

            while (k > 0 && !bucket[i].empty()) {
                result.push_back(bucket[i].back());
                bucket[i].pop_back();
                k--;
            }
        }

        return result;
    }
};
```

### Trace Example
`nums = [1,1,1,2,2,3]`, `n = 6`, `k = 2`

1. **Frequency map:** `{1:3, 2:2, 3:1}`
2. **Bucket array** (size `n+1 = 7`, indices `0..6`):

| Index (freq) | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Bucket contents | `[]` | `[3]` | `[2]` | `[1]` | `[]` | `[]` | `[]` |

3. **Scan from index `n=6` down to `0`:**
   - `i=6,5,4`: empty, skip
   - `i=3`: bucket has `[1]` → take `1` → result = `[1]`, `k=1`
   - `i=2`: bucket has `[2]` → take `2` → result = `[1,2]`, `k=0`
   - `k` is now 0, loop effectively does nothing more (or you can `break` early)

4. **Result:** `[1, 2]` ✅ — same answer as the heap approach, and notably it comes out **already sorted by descending frequency** as a side benefit.

## Why the Bucket Array Is Sized `n + 1`

- Minimum possible frequency: `0` (though in practice every element that appears has freq ≥ 1)
- Maximum possible frequency: `n` (if every element in the array is identical)
- So valid frequency values range over `[0, n]` → that's `n + 1` distinct slots → hence `vector<vector<int>> bucket(n + 1)`.

This bound is exactly what makes bucket sort applicable — we're exploiting a known, small range for the sort key.

## Complexity Breakdown

| Step                                   | Time    | Why                                                                 |
|------------------------------------------|---------|----------------------------------------------------------------------|
| Build frequency map                      | O(n)    | Single pass over nums                                                |
| Populate bucket array                    | O(m)    | One insert per unique element, m ≤ n                                 |
| Scan buckets from n down to 0, collect k | O(n)    | Worst case, you scan through all n+1 buckets (most are empty)        |
| **Total Time**                           | **O(n)** | Every step is linear — no log factor anywhere!                       |

**Space Complexity:**
- Frequency map: O(m) ≈ O(n)
- Bucket array: O(n + 1) buckets, but the **total number of elements across all buckets is exactly m** (each unique element appears in exactly one bucket)
- **Total: O(n)**

## Why This Beats the Heap Approach Asymptotically

- Heap approach: **O(n log k)**
- Bucket sort: **O(n)**

No matter how small k is, `log k ≥ 1` for `k ≥ 2`, so bucket sort's pure linear time is asymptotically at least as good, and strictly better whenever `k > 1`. There are **no comparisons and no logarithms anywhere** in bucket sort — it sidesteps the fundamental `O(n log n)` lower bound of comparison sorting by using **array indexing** instead of **comparisons** to determine order.

## When Is This the Right Choice?

- **Always**, when frequency (or any sort key) is bounded by a small, known range — this is the textbook-optimal solution for *this exact problem*.
- Great when n is not astronomically large (the `O(n)` bucket array must be allocated, so if n were, say, in the billions with very few unique elements, this could waste more memory than the heap approach — a rare edge case worth mentioning in an interview).

## Common Pitfalls

- Off-by-one on bucket array size — must be `n + 1`, not `n`, since frequency `n` is a valid (if rare) value.
- Forgetting to `break`/stop early once `k` elements are collected — not fixing this doesn't break correctness (the `k > 0` check guards it) but does waste a few cycles scanning remaining buckets. Adding `if (result.size() == k) break;` outside the inner while loop is a minor optimization.
- Iterating buckets in the **wrong direction** — must go from `n` down to `0` (descending frequency) to get the **most** frequent elements first.

---

# Side-by-Side Comparison

| Aspect | Brute Force (Approach 1) | Heap (Approach 2) | Bucket Sort (Approach 3) |
|---|---|---|---|
| **Time Complexity** | O(n log n) | O(n log k) | O(n) |
| **Space Complexity** | O(n) | O(n) | O(n) |
| **Core Technique** | Full comparison-based sort of all unique elements | Comparison-based, size-capped priority queue | Index-based counting sort |
| **Result Order** | Fully sorted by descending frequency | Not fully sorted by frequency | Naturally comes out sorted by descending frequency |
| **Wasted Work** | Sorts *all* m unique elements even though only top k are needed | None — heap size stays capped at k throughout | None — every bucket scan step is necessary |
| **Best When** | Simplicity matters, or k is close to m anyway | k is small; data is streamed; pattern reused elsewhere (k-way merge, k closest points, etc.) | Frequency (sort key) is bounded by a small known range — true optimal for this problem |
| **Generalizes To** | Any "sort then slice" problem | Many "top-k from a stream" problems | Only problems where the sort key is a small bounded integer |

## Progression of Optimization

It's worth seeing these three as a natural progression rather than three unrelated solutions:

1. **Brute force** asks: "What's the most obvious correct thing to do?" → sort everything, take the top k. Correct, but does unnecessary work (fully orders elements we'll discard).
2. **Heap** asks: "Do I really need to sort *everything*?" → No — I only need to *track* the top k at all times, discarding the weakest candidate whenever a better one shows up. This shrinks the sorting cost from `O(m log m)` down to `O(m log k)`.
3. **Bucket sort** asks: "Do I even need comparisons at all?" → No — since frequency is bounded by `n`, I can use frequency values directly as array indices, sidestepping comparison-based sorting entirely and achieving true `O(n)`.

**Interview tip:** Start with brute force to show correctness and establish a baseline. Move to the heap solution as the versatile, widely-applicable pattern (useful for many "top-k" problems beyond this one). Finish with bucket sort as the problem-specific optimal trick — showing you understand *why* frequency being bounded by `n` is the special property that unlocks linear time.
