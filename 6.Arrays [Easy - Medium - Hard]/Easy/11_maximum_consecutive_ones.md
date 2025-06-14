# 11. Maximum Consecutive Ones

## 🧩 Problem Statement

Given a binary array `nums`, return the **maximum number of consecutive 1s** in the array.

### 🔍 Example:
```cpp
Input: nums = [1,1,0,1,1,1]
Output: 3
Explanation: The first two 1s are followed by a 0, then three consecutive 1s follow, which is the maximum.
```

---

## ✅ Approach: Linear Traversal

We iterate through the array and count the number of consecutive 1s. Every time we encounter a 0, we reset the counter. Throughout the iteration, we maintain a variable to track the maximum number of consecutive 1s encountered so far.

---

## 🔑 Intuition

- A variable `cnt` counts the current streak of 1s.
- A variable `maxi` stores the **maximum** streak seen.
- Whenever we see a `1`, increment `cnt`.
- If a `0` is encountered, reset `cnt` to `0`.
- At each step, update `maxi` as the maximum of `maxi` and `cnt`.

---

## 💻 Code (C++)

```cpp
class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
        int maxi = 0;
        int cnt = 0;

        for(int i = 0; i < nums.size(); i++) {
            if(nums[i] == 1) {
                cnt++;
                maxi = max(maxi, cnt);
            } else {
                cnt = 0;
            }
        }
        return maxi;
    }
};
```

---

## ⏱️ Time Complexity

- **O(n)**: We traverse the array once where `n` is the size of the input array.

## 📦 Space Complexity

- **O(1)**: No extra space is used apart from a few variables.

---

## 🧠 Dry Run Example

Input: `[1, 1, 0, 1, 1, 1]`

| Index | Value | cnt | maxi |
|-------|-------|-----|------|
| 0     | 1     | 1   | 1    |
| 1     | 1     | 2   | 2    |
| 2     | 0     | 0   | 2    |
| 3     | 1     | 1   | 2    |
| 4     | 1     | 2   | 2    |
| 5     | 1     | 3   | 3    |

Final Output: `3`
