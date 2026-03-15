
# 10. Find Missing Number

### 🔍 Problem Statement

Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return the only number in the range that is missing from the array.

---

## ✅ Approaches

---

### 1. Brute Force

#### 🔸 Algorithm (Step-by-Step):
1. Loop from `i = 0` to `n`.
2. For each `i`, check if it's present in the array.
3. Use a nested loop to compare `i` with every element in the array.
4. If `i` is not found, return `i` as the missing number.

#### 💻 Code:
```cpp
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int n = nums.size();

        for(int i = 0; i <= n; i++) {
            int flag = 0;
            for(int j = 0; j < n; j++) {
                if(nums[j] == i) {
                    flag = 1;
                    break;
                }
            }
            if(flag == 0) return i;
        }
        return -1;
    }
};
```

#### ⏱️ Time Complexity:
- **O(N²)** — for each number from `0 to n`, we loop through the array.

#### 🧠 Space Complexity:
- **O(1)** — constant space, no extra storage used.

---

### 2. Better Approach (Using Hashing)

#### 🔸 Algorithm (Step-by-Step):
1. Create a hash array of size `n + 1` initialized to 0.
2. Traverse the input array and mark the elements present in the hash.
3. Loop through `1 to n` and find the index that has 0 in the hash array.
4. That index will be the missing number.

#### 💻 Code:
```cpp
int missingNumber(vector<int>& a, int N) {
    int hash[N + 1] = {0};

    for (int i = 0; i < N - 1; i++)
        hash[a[i]]++;

    for (int i = 1; i <= N; i++) {
        if (hash[i] == 0) {
            return i;
        }
    }
    return -1;
}
```

#### ⏱️ Time Complexity:
- **O(N)** — one loop to fill the hash and one to find the missing number.

#### 🧠 Space Complexity:
- **O(N)** — due to extra hash array.

---

### 3. Optimal Solution (Using Sum Formula)

#### 🔸 Algorithm (Step-by-Step):
1. Calculate the expected sum of first `n` numbers using formula `n*(n+1)/2`.
2. Calculate the actual sum of the array.
3. Subtract the actual sum from expected sum.
4. The result is the missing number.

#### 💻 Code:
```cpp
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int n = nums.size();

        int sum1 = (n * (n + 1)) / 2;
        int sum2 = 0;

        for(int i = 0; i < n; i++) {
            sum2 += nums[i];
        }

        return sum1 - sum2;
    }
};
```

#### ⏱️ Time Complexity:
- **O(N)** — single pass to compute the sum.

#### 🧠 Space Complexity:
- **O(1)** — no extra space used.

---

## 📌 Conclusion

| Approach        | Time Complexity | Space Complexity | Notes                        |
|----------------|------------------|------------------|------------------------------|
| Brute Force     | O(N²)            | O(1)             | Simple but inefficient       |
| Better (Hashing)| O(N)             | O(N)             | Faster, uses extra memory    |
| Optimal         | O(N)             | O(1)             | Best solution using math     |

---
