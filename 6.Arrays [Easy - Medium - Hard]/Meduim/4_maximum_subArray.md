# 4. Maximum Subarray

## 🧠 Problem Statement

Given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

---

## 1️⃣ Brute Force Approach

### 🔍 Intuition:
We can try **every possible subarray** and calculate their sum. The subarray which has the maximum sum will be our answer.

### 🧾 Algorithm:
1. Initialize `maxi = nums[0]` to track the maximum subarray sum.
2. Loop `i` from `0` to `n-1` (start of subarray).
3. Loop `j` from `i` to `n-1` (end of subarray).
4. Loop `k` from `i` to `j` to calculate sum of current subarray.
5. If this sum is greater than `maxi`, update `maxi`.
6. Return `maxi`.

### 🧪 Dry Run:
For `nums = [1, -2, 3]`
- Subarrays:  
  - [1] → 1  
  - [1, -2] → -1  
  - [1, -2, 3] → 2  
  - [-2] → -2  
  - [-2, 3] → 1  
  - [3] → 3 ✅ Max

### 💻 Code:
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        long long maxi = nums[0];

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                int sum = 0;
                for (int k = i; k <= j; k++) {
                    sum += nums[k];
                }
                maxi = max(maxi, sum);
            }
        }
        return maxi;
    }
};
```

### ⏱️ Time Complexity:
- `O(N³)` — Three nested loops.

### 📦 Space Complexity:
- `O(1)` — No extra space used.

---

## 2️⃣ Better Approach

### 🔍 Intuition:
Instead of calculating the sum of each subarray using an inner loop, we can **accumulate the sum on-the-fly** while expanding the subarray.

### 🧾 Algorithm:
1. Initialize `maxi = nums[0]`.
2. Loop `i` from `0` to `n-1`.
3. Initialize `sum = 0` for each `i`.
4. Loop `j` from `i` to `n-1`.
   - Add `nums[j]` to `sum`.
   - Update `maxi = max(maxi, sum)`.
5. Return `maxi`.

### 🧪 Dry Run:
For `nums = [1, -2, 3]`
- i = 0:
  - sum = 1 → maxi = 1
  - sum = -1 → maxi = 1
  - sum = 2 → maxi = 2
- i = 1:
  - sum = -2 → maxi = 2
  - sum = 1 → maxi = 2
- i = 2:
  - sum = 3 → maxi = 3 ✅

### 💻 Code:
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        long long maxi = nums[0];

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                maxi = max(maxi, sum);
            }
        }
        return maxi;
    }
};
```

### ⏱️ Time Complexity:
- `O(N²)` — Two nested loops.

### 📦 Space Complexity:
- `O(1)` — Constant extra space.

---

## 3️⃣ Optimal Approach (Kadane's Algorithm)

### 🔍 Intuition:
We can keep a **running sum** and **reset it to 0** whenever it becomes negative. A negative sum will only reduce the total sum of any future subarray.

### 🧾 Algorithm:
1. Initialize `sum = 0`, `maxi = LONG_MIN`.
2. Loop `i` from `0` to `n-1`:
   - Add `nums[i]` to `sum`.
   - If `sum > maxi`, update `maxi`.
   - If `sum < 0`, reset `sum = 0`.
3. Return `maxi`.

### 🧪 Dry Run:
For `nums = [-2,1,-3,4,-1,2,1,-5,4]`  
We track `sum` and `maxi`:
- i=0: sum=-2 → reset to 0, maxi=-2
- i=1: sum=1 → maxi=1
- i=2: sum=-2 → reset, maxi=1
- i=3: sum=4 → maxi=4
- i=4: sum=3 → maxi=4
- i=5: sum=5 → maxi=5
- i=6: sum=6 → maxi=6 ✅
- i=7: sum=1
- i=8: sum=5 → maxi=6

### 💻 Code:
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        long long sum = 0, maxi = LONG_MIN;

        for (int i = 0; i < n; i++) {
            sum += nums[i];
            if (sum > maxi) {
                maxi = sum;
            }
            if (sum < 0) {
                sum = 0;
            }
        }
        return maxi;
    }
};
```

### ⏱️ Time Complexity:
- `O(N)` — Single pass through the array.

### 📦 Space Complexity:
- `O(1)` — Constant space used.

---

## ✅ Summary Table

| Approach      | Time Complexity | Space Complexity | Notes                       |
|---------------|------------------|-------------------|-----------------------------|
| Brute Force   | O(N³)            | O(1)              | Triple nested loops         |
| Better        | O(N²)            | O(1)              | Running sum optimization    |
| Optimal (Kadane's) | O(N)       | O(1)              | Best possible solution ✅   |

---

📌 **Tip:** Kadane’s Algorithm also helps in identifying the actual subarray if needed by tracking start and end indices!