# Max Consecutive Ones III

---

## 🔹 Brute Force Approach

### 1. Approach / Intuition
- Try **all possible subarrays** starting from every index `i`.
- For each start index, extend the subarray using index `j`.
- Count the number of `0`s in the current subarray.
- If the number of `0`s is **≤ k**, we can flip them, so the subarray is valid.
- Keep updating the maximum length.
- As soon as `0`s exceed `k`, break (no need to extend further).

This approach directly checks every possible window, hence it is simple but inefficient.

### 2. Code
```cpp
class Solution {
public:
    int longestOnes(vector<int>& nums, int k) {
        int n = nums.size();
        int maxi = 0;

        for (int i = 0; i < n; i++) {
            int zeros = 0;
            for (int j = i; j < n; j++) {
                if (nums[j] == 0)
                    zeros++;

                if (zeros <= k) {
                    int len = j - i + 1;
                    maxi = max(maxi, len);
                } else {
                    break;
                }
            }
        }
        return maxi;
    }
};
```

### 3. Time & Space Complexity
- **Time Complexity:** `O(n²)`
- **Space Complexity:** `O(1)`

---

## 🔹 Better Approach (Sliding Window)

### 1. Approach / Intuition
- Use the **sliding window** technique with two pointers `left` and `right`.
- Expand the window by moving `right`.
- Count how many `0`s are inside the window.
- If `0`s exceed `k`, shrink the window from the left until it becomes valid again.
- At every step, update the maximum window length.

This avoids recomputing for each subarray and improves efficiency.

### 2. Code
```cpp
class Solution {
public:
    int longestOnes(vector<int>& nums, int k) {
        int n = nums.size();
        int left = 0, maxi = 0, zeros = 0;

        for (int right = 0; right < n; right++) {
            if (nums[right] == 0)
                zeros++;

            while (zeros > k) {
                if (nums[left] == 0)
                    zeros--;
                left++;
            }

            maxi = max(maxi, right - left + 1);
        }
        return maxi;
    }
};
```

### 3. Time & Space Complexity
- **Time Complexity:** `O(n)`
- **Space Complexity:** `O(1)`

---

## 🔹 Optimal Approach (Clean Sliding Window)

### 1. Approach / Intuition
- This is a **refined sliding window** version.
- Use two pointers `l` and `r`.
- Expand the window using `r` and count `0`s.
- If `0`s exceed `k`, move `l` to reduce them.
- Update the maximum length only when the window is valid.

This version is clean, readable, and interview-ready.

### 2. Code
```cpp
class Solution {
public:
    int longestOnes(vector<int>& nums, int k) {
        int n = nums.size();
        int maxLen = 0, l = 0, r = 0, zeros = 0;

        while (r < n) {
            if (nums[r] == 0) zeros++;

            if (zeros > k) {
                if (nums[l] == 0) zeros--;
                l++;
            }

            if (zeros <= k) {
                maxLen = max(maxLen, r - l + 1);
            }
            r++;
        }
        return maxLen;
    }
};
```

### 3. Time & Space Complexity
- **Time Complexity:** `O(n)`
- **Space Complexity:** `O(1)`

---

## ✅ Key Takeaway
- Brute force helps in **understanding the problem**.
- Sliding Window is the **core concept** behind optimization.
- Once sliding window clicks, many array problems become easy 🚀

