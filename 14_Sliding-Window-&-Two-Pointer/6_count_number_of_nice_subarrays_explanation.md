# Count Number of Nice Subarrays

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

- Try every possible starting index `start`.
- For each `start`, extend subarray using `end` pointer.
- Maintain a counter `oddCount` to track number of odd numbers in current subarray.
- If element is odd → increment `oddCount`.
- If `oddCount > k` → break (no need to extend further).
- If `oddCount == k` → increment result.

Why break early?
Because adding more elements will only increase the odd count further.

This checks all possible subarrays.

---

## 2. Code

```cpp
class Solution {
public:
    int numberOfSubarrays(vector<int>& nums, int k) {
        int count = 0;

        for (int start = 0; start < nums.size(); start++) {
            int oddCount = 0;

            for (int end = start; end < nums.size(); end++) {
                if (nums[end] % 2 != 0)
                    oddCount++;

                if (oddCount > k)
                    break;

                if (oddCount == k)
                    count++;
            }
        }

        return count;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
- **Space Complexity:** O(1)

---

# 2️⃣ Optimal Approach (Sliding Window – AtMost Trick)

## 1. Approach / Intuition

Important Observation:

We convert the array into a binary representation:
- Odd number → 1
- Even number → 0

Now the problem becomes:

Count subarrays with sum = k

Using the same trick:

Number of subarrays with exactly k odds =
Subarrays with sum ≤ k − Subarrays with sum ≤ (k - 1)

We create a helper function:
- Count subarrays with sum ≤ goal
- Use sliding window

Inside helper:
- Add `(nums[r] % 2)` to sum
- Shrink left pointer while sum > goal
- Add `(r - l + 1)` to count

This counts all valid subarrays ending at index `r`.

Final Answer:

```
count(k) - count(k - 1)
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
            sum += (nums[r] % 2);

            while (sum > goal) {
                sum -= (nums[l] % 2);
                l += 1;
            }

            cnt += (r - l + 1);
            r += 1;
        }

        return cnt;
    }
public:
    int numberOfSubarrays(vector<int>& nums, int k) {
        return sumLessThanEqualToGoal(nums, k) -
               sumLessThanEqualToGoal(nums, k - 1);
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
| Optimal | O(N) | O(1) | Sliding Window (AtMost trick) |

---

Structured cleanly for:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Perfect for revision and interview prep.

