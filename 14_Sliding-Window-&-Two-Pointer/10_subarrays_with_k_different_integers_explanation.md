# Subarrays With K Different Integers

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

Goal:
Count the number of subarrays that contain exactly `k` distinct integers.

Brute Force Idea:
- Fix a starting index `i`.
- Extend the subarray using index `j`.
- Maintain a hashmap to store frequency of elements.
- If number of distinct elements == k → increment count.
- If distinct elements > k → break (further extension will only increase distinct count).

We check all possible subarrays.

---

## 2. Code

```cpp
class Solution {
public:
    int subarraysWithKDistinct(vector<int>& nums, int k) {
        int n  = nums.size();
        int count = 0;

        for(int i = 0; i < n; i++) {
            unordered_map<int , int> mpp;

            for(int j = i; j < n; j++) {
                mpp[nums[j]]++;

                if(mpp.size() == k) {
                    count++;
                }
                if(mpp.size() > k) break;
            }
        } 

        return count;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
- **Space Complexity:** O(N)
  - In worst case, hashmap may store all distinct elements

---

# 2️⃣ Optimal Approach (Sliding Window + AtMost Trick)

## 1. Approach / Intuition

Key Insight:

Number of subarrays with exactly k distinct elements =

Subarrays with at most k distinct − Subarrays with at most (k - 1) distinct

Why this works?
Because "exactly k" can be derived by subtracting the smaller constraint from the larger one.

We create a helper function:

Helper(nums, k):
- Counts subarrays with at most k distinct elements
- Use sliding window

Sliding Window Logic:
1. Expand `r`
2. Add element to hashmap
3. If distinct elements > k:
   - Shrink from left
   - Reduce frequency
   - Remove element if frequency becomes 0
4. Add `(r - l + 1)` to count
   - This counts all valid subarrays ending at `r`

Final Answer:

```
helper(k) - helper(k - 1)
```

---

## 2. Code

```cpp
class Solution {
private: 
    int helper(vector<int>& nums, int k) {
        if(k == 0) return 0;
        int n = nums.size();

        int l = 0, r = 0, count = 0;
        unordered_map<int , int> mpp;

        while(r < n) {
            mpp[nums[r]]++;

            while(mpp.size() > k) {
                mpp[nums[l]]--;

                if(mpp[nums[l]] == 0) {
                    mpp.erase(nums[l]);
                }
                
                l++;
            }

            count = count + (r - l + 1);
            r += 1;
        }

        return count;
    }

public:
    int subarraysWithKDistinct(vector<int>& nums, int k) {
       return helper(nums, k) - helper(nums, k - 1); 
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(2 × N) ≈ O(N)
  - Helper runs twice
  - Each element processed at most twice per run

- **Space Complexity:** O(N)
  - Hashmap may store up to N distinct elements

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Technique |
|-----------|-----------------|--------|-----------|
| Brute | O(N²) | O(N) | Nested loops |
| Optimal | O(N) | O(N) | Sliding Window + AtMost Trick |

---

Structured clearly for:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Perfect for interview revision and sliding window m