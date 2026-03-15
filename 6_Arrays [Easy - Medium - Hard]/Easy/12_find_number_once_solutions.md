# 🧠 Find the Number that Appears Only Once

Given an array where every element appears twice except for one, find the element that appears only once.

---

## 🔸 1. Brute Force Approach

### ✅ Code:
```cpp
#include <bits/stdc++.h>
using namespace std;

int onceNumber(vector<int> arr) {
    int n = arr.size();

    for (int i = 0; i < n; i++) {
        int num = arr[i];
        int cnt = 0;

        for (int j = 0; j < n; j++) {
            if (arr[j] == num) cnt++;
        }

        if (cnt == 1) return num;
    }

    return -1;
}

int main() {
    vector<int> arr = {1, 1, 2, 2, 3, 4, 4};
    int ans = onceNumber(arr);
    cout << ans;
}
```

### 💡 Explanation:
- For every element, we count how many times it appears in the array.
- If a number appears exactly once, we return it.

### ⏱ Time Complexity:
- Outer loop runs `n` times, and for each iteration, the inner loop also runs `n` times.
- Total time = `O(n * n)` = `O(n²)`

### 💾 Space Complexity:
- No extra space is used.
- **SC = O(1)**

---

## 🔸 2. Better Approach (Using Hashing with Array)

### ✅ Code:
```cpp
int onceNumber(vector<int> arr) {
    int n = arr.size();
    int maxi = arr[0];

    // Find max element to define hash array size
    for (int i = 0; i < n; i++) {
        maxi = max(maxi, arr[i]);
    }

    vector<int> hash(maxi + 1, 0);

    for (int i = 0; i < n; i++) {
        hash[arr[i]]++;
    }

    for (int i = 0; i < n; i++) {
        if (hash[arr[i]] == 1)
            return arr[i];
    }

    return -1;
}
```

### 💡 Explanation:
- Use a fixed-size array (hash) to store the frequency of each number.
- After counting, check which element occurred only once.

### ⏱ Time Complexity:
- Finding max: `O(N)`
- Counting frequencies: `O(N)`
- Searching for element with frequency 1: `O(N)`
- Total: `O(3N)` → **O(N)**

### 💾 Space Complexity:
- **O(max element in array)** → `O(maxi)`

---

## 🔸 3. Better Approach (Using Map)

### ✅ Code:
```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int n = nums.size();
        map<int, int> mpp;

        for (int i = 0; i < n; i++) {
            mpp[nums[i]]++;
        }

        for (auto it : mpp) {
            if (it.second == 1) {
                return it.first;
            }
        }

        return -1;
    }
};
```

### 💡 Explanation:
- Use a map to count occurrences of each number.
- Then iterate through the map to find the one with count 1.

### ⏱ Time Complexity:
- Insertion in map takes `O(log N)` time.
- Inserting `N` elements: `O(N log N)`
- Traversing map: ~`O(N)`
- Total: **O(N log N)**

### 💾 Space Complexity:
- Storing unique elements: **O(N/2)** (if half are duplicates)
- **SC = O(N)** in the worst case.

---

## 🔸 4. Optimal Approach (Using XOR)

### ✅ Code:
```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int n = nums.size();
        int xorr = 0;

        for (int i = 0; i < n; i++) {
            xorr ^= nums[i];
        }

        return xorr;
    }
};
```

### 💡 Explanation:
- The XOR of two same numbers is 0 (i.e., `a ^ a = 0`).
- XOR of any number with 0 is the number itself.
- So, all duplicate elements cancel each other and we get the unique one.

### ⏱ Time Complexity:
- Only one pass through the array → **O(N)**

### 💾 Space Complexity:
- No extra space used → **O(1)**

---

## ✅ Summary

| Approach       | Time Complexity | Space Complexity | Method     |
|----------------|------------------|-------------------|------------|
| Brute Force    | O(n²)            | O(1)              | Nested Loops |
| Hashing Array  | O(n)             | O(max element)    | Frequency Array |
| Map            | O(n log n)       | O(n)              | Hash Map |
| XOR (Optimal)  | O(n)             | O(1)              | Bitwise XOR |