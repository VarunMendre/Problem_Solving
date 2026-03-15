
# 11. Subarray Sum Equals K

---

## 🧩 Problem Statement

Given an array of integers `nums` and an integer `k`, return the total number of **continuous subarrays** whose sum equals to `k`.

### Example:
```cpp
Input: nums = [1,1,1], k = 2  
Output: 2
```
The subarrays [1,1] at index 0-1 and index 1-2 both have sum = 2.

---

## 🧠 1. Brute Force

### 🧭 Intuition:
We can try out all possible subarrays and check whether the sum equals `k`.

### 🛠 Approach:
- Use three nested loops:
  - Outer loop to fix the starting point.
  - Middle loop to fix the ending point.
  - Inner loop to calculate the sum of elements between start and end.

### 🔍 Algorithm:
1. Initialize count = 0.
2. Run `i` from 0 to n-1:
   - Run `j` from `i` to n-1:
     - Initialize `sum = 0`.
     - Run `k` from `i` to `j` and compute sum.
     - If sum equals `k`, increment count.
3. Return count.

### 🧪 Dry Run:
```cpp
nums = [1, 2, 3], k = 3
Subarrays:
[1] => 1
[1,2] => 3 ✅
[1,2,3] => 6
[2] => 2
[2,3] => 5
[3] => 3 ✅
Output: 2
```

### 💻 Code:
```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        int cnt = 0;
        for(int i = 0; i < n; i++) {
            for(int j = i; j < n; j++) {
                int sum = 0;
                for(int l = i; l <= j; l++) {
                    sum += nums[l];
                }
                if(sum == k) {
                    cnt++;
                }
            }
        }
        return cnt;
    }
};
```

### ⏱ Time & Space Complexity:
- **Time Complexity:** O(N³) — three nested loops
- **Space Complexity:** O(1) — constant space used

---

## ⚙️ 2. Better Approach

### 🧭 Intuition:
Instead of using a third loop for summation, maintain a running sum in the inner loop to avoid redundant operations.

### 🛠 Approach:
- Two nested loops:
  - Outer loop for starting index.
  - Inner loop to move forward and keep a running sum.
  - If running sum equals `k`, increment count.

### 🔍 Algorithm:
1. Initialize count = 0.
2. Loop `i` from 0 to n-1:
   - Initialize `sum = 0`.
   - Loop `j` from `i` to n-1:
     - Add `nums[j]` to `sum`.
     - If `sum == k`, increment count.
3. Return count.

### 🧪 Dry Run:
```cpp
nums = [1,2,3], k = 3
i = 0, sum = 0
  j = 0 → sum = 1
  j = 1 → sum = 3 ✅
  j = 2 → sum = 6
i = 1, sum = 0
  j = 1 → sum = 2
  j = 2 → sum = 5
i = 2, sum = 0
  j = 2 → sum = 3 ✅
Output: 2
```

### 💻 Code:
```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        int cnt = 0;
        for(int i = 0; i < n; i++) {
            int sum = 0;
            for(int j = i; j < n; j++) {
                sum += nums[j];
                if(sum == k) {
                    cnt++;
                }
            }
        }
        return cnt;
    }
};
```

### ⏱ Time & Space Complexity:
- **Time Complexity:** O(N²) — two nested loops
- **Space Complexity:** O(1)

---

## 🚀 3. Optimal Approach (Prefix Sum + HashMap)

### 🧭 Intuition:
We can use a **prefix sum technique** with a hashmap to store how many times a particular prefix sum has occurred.

If at index `i`, the prefix sum is `preSum`, and we want the subarray sum to be `k`, then:
```
preSum - k should have occurred before
```

### 🛠 Approach:
- Maintain a prefix sum `preSum` and a hashmap `mpp` to store frequency of each prefix sum.
- For each element:
  - Update `preSum` with current element.
  - Check if `(preSum - k)` exists in hashmap.
    - If yes, add its frequency to count.
  - Increment the frequency of `preSum` in hashmap.

### 🔍 Algorithm:
1. Initialize map with `{0: 1}` to handle case where prefix sum equals `k`.
2. Initialize `preSum = 0`, `count = 0`.
3. Loop over each element:
   - Add it to `preSum`.
   - Check if `preSum - k` exists in map.
     - If yes, add frequency to count.
   - Update `preSum` frequency in map.
4. Return count.

### 🧪 Dry Run:
```cpp
nums = [1,2,3], k = 3
preSum = 0, mpp = {0:1}, count = 0

i = 0: preSum = 1, remove = -2 → not in map
       mpp = {0:1, 1:1}
i = 1: preSum = 3, remove = 0 ✅ → count = 1
       mpp = {0:1, 1:1, 3:1}
i = 2: preSum = 6, remove = 3 ✅ → count = 2
       mpp = {0:1, 1:1, 3:1, 6:1}
```

### 💻 Code:
```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        unordered_map<int, int> mpp;
        mpp[0] = 1;

        int preSum = 0, cnt = 0;

        for(int i = 0; i < n; i++) {
            preSum += nums[i];
            int remove = preSum - k;

            cnt += mpp[remove];
            mpp[preSum]++;
        }

        return cnt;
    }
};
```

### ⏱ Time & Space Complexity:
- **Time Complexity:**
  - **O(N)** using `unordered_map`
  - **O(N log N)** if using `map` (ordered)
- **Space Complexity:**
  - **O(N)** in worst case (all prefix sums unique)

---

✅ Choose:
- **Brute Force** for conceptual understanding.
- **Better** if memory isn't a constraint and array size is small.
- **Optimal** for interviews and large inputs.

---
