# Majority Element

## 🧾 Problem Statement

Given an array `nums` of size `n`, return the **majority element**.

The **majority element** is the element that appears more than `⌊n / 2⌋` times. You may assume that the majority element **always exists** in the array.

### 🧪 Examples

**Example 1:**
```
Input: nums = [3,2,3]  
Output: 3
```

**Example 2:**
```
Input: nums = [2,2,1,1,1,2,2]  
Output: 2
```

---

## ✅ Constraints

- `n == nums.length`
- `1 <= n <= 5 * 10⁴`
- `-10⁹ <= nums[i] <= 10⁹`

---

# 🧠 Solutions

---

## 🚩 Brute Force Approach

### 🔹 Algorithm
1. Loop through each element of the array.
2. For every element, count how many times it appears in the array.
3. If count is greater than `n / 2`, return that element.
4. If no such element is found (which shouldn't happen as per problem constraints), return -1.

### 🔍 Intuition
We're checking every element and counting its frequency using nested loops.

### 🧪 Dry Run
For `nums = [2,2,1,1,1,2,2]`  
- Check 2 → appears 4 times → 4 > 7/2 → return 2

### 💻 Code
```cpp
for(int i = 0; i < n; i++) {
    int cnt = 0;
    for(int j = 0; j < n; j++) {
        if(nums[i] == nums[j]) {
            cnt++;
        }
    }
    if(cnt > n / 2) return nums[i];
}
return -1;
```

### ⏱️ Time and Space Complexity
- Time Complexity: **O(N²)** — Nested iteration.
- Space Complexity: **O(1)** — No extra space used.

---

## 🚀 Better Approach (Using Hash Map)

### 🔹 Algorithm
1. Create a hash map to store the frequency of each element.
2. Traverse the array and populate the map.
3. Traverse the map to find the element with frequency > `n / 2`.

### 🔍 Intuition
Using a map reduces the nested iteration to a single pass plus a map lookup.

### 🧪 Dry Run
For `nums = [2,2,1,1,1,2,2]`,  
Map becomes: `{2: 4, 1: 3}`  
4 > 7/2 → return 2

### 💻 Code
```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int n = nums.size();
        map<int, int> mpp;

        for(int i = 0; i < n; i++) {
            mpp[nums[i]]++;
        }

        for(auto it : mpp) {
            if(it.second > n / 2) {
                return it.first;
            }
        }

        return -1;
    }
};
```

### ⏱️ Time and Space Complexity
- Time Complexity: **O(N log N)** for `map` insertion + **O(N)** for second loop.
- Space Complexity: **O(N)** — In worst case if all elements are unique.

---

## 🥇 Optimal Approach (Moore's Voting Algorithm)

---

### 🧠 Moore's Voting Algorithm – Explained

The key idea is:  
If we cancel out each occurrence of an element `x` with all other elements that are **not** `x`, then `x` will still be the majority element.

### 🔹 Steps:
1. **First Pass**: Identify a *candidate* that could be the majority element.
   - Use a counter.
   - If counter is `0`, pick current element as candidate.
   - If current element == candidate → increment counter
   - Else → decrement counter
2. **Second Pass**: Verify if the candidate is truly a majority.

---

### 🔹 Algorithm
1. Initialize `cnt = 0`, `ele = 0`.
2. Traverse array:
   - If `cnt == 0`, set `ele = nums[i]`.
   - If `nums[i] == ele`, `cnt++`.
   - Else, `cnt--`.
3. After loop, `ele` is a candidate.
4. Count its actual frequency.
5. If frequency > n/2, return `ele`, else return -1.

---

### 🔍 Intuition
The majority element always survives because it cancels out all minority elements and still remains at the end.

---

### 🧪 Dry Run
`nums = [2,2,1,1,1,2,2]`

Pass 1:
- i=0: ele=2, cnt=1  
- i=1: nums[i]=2 → cnt=2  
- i=2: nums[i]=1 → cnt=1  
- i=3: nums[i]=1 → cnt=0  
- i=4: cnt=0 → ele=1, cnt=1  
- i=5: nums[i]=2 → cnt=0  
- i=6: cnt=0 → ele=2, cnt=1  

Candidate = 2  
Count in array = 4 → valid majority → return 2

---

### 💻 Code
```cpp
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int n = nums.size();
        int ele, cnt1 = 0;

        for(int i = 0; i < n; i++) {
            if(cnt1 == 0) {
                ele = nums[i];
                cnt1 = 1;
            }
            else if(nums[i] == ele) {
                cnt1++;
            }
            else {
                cnt1--;
            }
        }

        int cnt2 = 0;
        for(int i = 0; i < n; i++) {
            if(nums[i] == ele) cnt2++;
        }

        if(cnt2 > n/2) return ele;
        return -1;
    }
};
```

---

### ⏱️ Time and Space Complexity

- **Time Complexity**:  
  - First pass: O(N)  
  - Second pass: O(N)  
  - **Total**: O(N)

- **Space Complexity**:  
  - **O(1)** — Constant space used.

---

✅ This is the most efficient approach when the majority element is guaranteed to exist.
