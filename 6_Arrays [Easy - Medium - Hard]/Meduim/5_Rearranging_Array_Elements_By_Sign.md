
# 5. Rearranging Array Elements By Sign

## 1. Problem Statement

Given an array `nums` containing equal numbers of positive and negative integers, rearrange the elements such that:

- The elements are placed in alternating order starting with a positive number.
- The relative order of positive and negative numbers should be preserved.

**Constraints:**
- The array always contains equal numbers of positive and negative integers.

---

## 2. Approach That We're Using

We’ll go over **two approaches**:
- **Brute Force Approach:** Separate positive and negative numbers, then merge them in alternating order.
- **Optimal Approach:** Do the same while maintaining positions using an auxiliary array without extra passes.

---

## 3. Initiation

Before we begin solving:
- Create two vectors to hold positive and negative elements.
- For optimal, directly place the elements at correct indices in a result array.

---

## 4. Algorithm

### Brute Force Approach:
1. Traverse the array and store positive numbers in one array and negative in another.
2. Place elements in alternating positions using the two arrays.
   - Place positive[i] at index `2*i`
   - Place negative[i] at index `2*i + 1`

### Optimal Approach:
1. Create a result array of the same size.
2. Use two pointers: `posIndex = 0` and `negaIndex = 1`
3. Traverse the input array:
   - If the element is positive, place it at `result[posIndex]` and increment `posIndex` by 2.
   - If the element is negative, place it at `result[negaIndex]` and increment `negaIndex` by 2.

---

## 5. Dry Run

### Example Input:
```cpp
nums = [3,1,-2,-5,2,-4]
```

### Brute Force:
- positive = [3, 1, 2]
- negative = [-2, -5, -4]

Final rearranged:
- [3, -2, 1, -5, 2, -4]

### Optimal:
- Initial result: [0,0,0,0,0,0]
- 3 → result[0] = 3 → posIndex = 2
- 1 → result[2] = 1 → posIndex = 4
- -2 → result[1] = -2 → negaIndex = 3
- -5 → result[3] = -5 → negaIndex = 5
- 2 → result[4] = 2 → posIndex = 6
- -4 → result[5] = -4

Result = [3, -2, 1, -5, 2, -4]

---

## 6. Code

### Brute Force Code:
```cpp
class Solution {
public:
    vector<int> rearrangeArray(vector<int>& nums) {
        int n = nums.size();
        vector<int> positive;
        vector<int> negative;

        for(int i=0;i<n;i++) {
            if(nums[i] > 0) {
                positive.push_back(nums[i]);
            } else {
                negative.push_back(nums[i]);
            }
        }
        for(int i=0;i<n/2;i++) {
            nums[2 * i] = positive[i];
            nums[2 * i + 1] = negative[i];
        }
        return nums;
    }
};
```

### Optimal Code:
```cpp
class Solution {
public:
    vector<int> rearrangeArray(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, 0);
        int posIndex = 0, negaIndex = 1; 
        for(int i = 0; i < n; i++) {
            if(nums[i] < 0) {
                result[negaIndex] = nums[i];
                negaIndex += 2;
            } else {
                result[posIndex] = nums[i];
                posIndex += 2;
            }
        }
        return result;
    }
};
```

---

## 7. Time & Space Complexity

### Brute Force:
- **Time Complexity:** O(N) for separating + O(N) for merging → **O(2N)**
- **Space Complexity:** O(N) for positive + O(N) for negative → **O(N)**

### Optimal:
- **Time Complexity:** O(N)
- **Space Complexity:** O(N) (result array)
