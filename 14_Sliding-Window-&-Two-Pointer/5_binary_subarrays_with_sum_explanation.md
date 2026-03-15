# Binary Subarrays With Sum

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

- Try every possible starting index `i`.
- For each `i`, extend the subarray using pointer `j`.
- Maintain a running sum.
- If `sum == goal` → increment answer.
- If `sum > goal` → break early (since array contains only 0s and 1s).

Why break?
Because adding more elements (0 or 1) will never reduce the sum.

This checks all possible subarrays.

---

## 2. Code

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& nums, int goal) {
        int n = nums.size();
        int ans = 0;

        for(int i = 0; i < n; i++) {
            int sum = 0;

            for(int j = i; j < n; j++) {
                sum += nums[j];

                if(sum == goal) {
                    ans += 1;
                }

                if(sum > goal) break;
            }
        }

        return ans;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
- **Space Complexity:** O(1)

---

# 2️⃣ Better Approach (Prefix Sum + HashMap)

## 1. Approach / Intuition

Core Idea:

If `(prefixSum - goal)` existed before, then there exists a subarray ending at current index whose sum is equal to `goal`.

Steps:
- Maintain a running prefix sum.
- Use a hashmap to store frequency of prefix sums.
- For each element:
  - Update prefix sum.
  - Compute `remove = prefixSum - goal`.
  - Add frequency of `remove` to answer.
  - Store current prefixSum in map.

This converts the problem into the classic:
**"Count subarrays with sum = K"**.

---

## 2. Code

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& nums, int goal) {
        int n = nums.size();

        int preFixSum = 0;
        int cnt = 0;

        unordered_map<int, int> mpp;
        mpp[0] = 1;

        for(int i = 0; i < n; i++) {
            preFixSum += nums[i];
            int remove = preFixSum - goal;

            cnt += mpp[remove];
            mpp[preFixSum] += 1;
        }

        return cnt;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N)
- **Space Complexity:** O(N)
  - HashMap may store up to N prefix sums

---

# 3️⃣ Optimal Approach (Sliding Window – AtMost Trick)

## 1. Approach / Intuition

Important Observation (only works because array contains 0s and 1s):

Number of subarrays with sum = goal
=
Subarrays with sum ≤ goal − Subarrays with sum ≤ (goal - 1)

Why does this work?
Because counting "exactly goal" can be derived from two "at most" counts.

We create a helper function:
- Count subarrays with sum ≤ goal
- Use sliding window

Inside helper:
- Expand right pointer
- Shrink left pointer while sum > goal
- Add `(r - l + 1)` to count

This counts all valid subarrays ending at index `r`.

Final Answer:

```
count(goal) - count(goal - 1)
```

---

## 2. Code

```cpp
class Solution {
private:
    int sumLessThanEqualToGoal(vector<int>& nums, int goal) {
        if (goal < 0)
            return 0;
        int l = 0, r = 0, sum = 0, cnt = 0;
        int n = nums.size();

        while (r < n) {
            sum += nums[r];

            while (sum > goal) {
                sum -= nums[l];
                l += 1;
            }

            cnt += (r - l + 1);
            r += 1;
        }

        return cnt;
    }

public:
    int numSubarraysWithSum(vector<int>& nums, int goal) {
        return sumLessThanEqualToGoal(nums, goal) -
               sumLessThanEqualToGoal(nums, goal - 1);
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(2 × N) ≈ O(N)
  - Helper function runs twice
- **Space Complexity:** O(1)

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Technique |
|-----------|-----------------|--------|-----------|
| Brute | O(N²) | O(1) | Nested loops |
| Better | O(N) | O(N) | Prefix Sum + HashMap |
| Optimal | O(N) | O(1) | Sliding Window (AtMost trick) |

---

Structured exactly as required:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Clean for revision and interviews.

