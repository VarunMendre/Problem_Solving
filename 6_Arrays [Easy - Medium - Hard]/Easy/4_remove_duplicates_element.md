
# 4. Remove Duplicates from Sorted Array

### 🧾 Problem Statement

Given an integer array `nums` sorted in non-decreasing order, remove the duplicates **in-place** such that each unique element appears only once. The relative order of the elements should be kept the same. Then return the number of unique elements in `nums`.

Consider the number of unique elements of `nums` to be `k`. To get accepted, you need to do the following:

- Change the array `nums` such that the first `k` elements of `nums` contain the unique elements in the order they were present initially.
- The remaining elements of `nums` are not important as well as the size of `nums`.
- Return `k`.

---

### 🧠 Understanding the Problem

You are given a sorted integer array `arr` of size `n`.

You need to remove the duplicates from the array such that each element appears only once and return the length of the updated array.

**Note:**  
- Do not allocate extra space for another array.
- Modify the given input array in place using **O(1)** extra memory.

---

### 💡 Example

Input:
```
n = 5  
arr = [1, 2, 2, 2, 3]
```

Output:
```
New array = [1, 2, 3, _, _] (the last two values are irrelevant)  
Return: 3
```

---

## 🔸 Brute Force Approach — Using Set (in-place idea)

### ✔️ Approach:

- A `set` automatically removes duplicates and maintains the elements in sorted order.
- Insert all elements into the set.
- Traverse the set and copy its elements back into the original array from the beginning.

### 🧩 Code:
```cpp
set<int> st;
for(int i = 0; i < n; i++) {
    st.insert(arr[i]);
}

int index = 0;
for(auto it : st) {
    arr[index] = it;
    index++;
}
```

### ⏱️ Time Complexity:
- `O(N log N)` for inserting all `N` elements into the set.
- `O(N)` for copying elements back.
- **Total: O(N log N + N)**

### 📦 Space Complexity:
- `O(N)` for the set.

---

## 🔹 Optimal Approach — Two Pointers

### ✔️ Approach:

- Since the array is **already sorted**, duplicates will always be **adjacent**.
- Use **two pointers**:
  - `i` will mark the position of the last unique element.
  - `j` will traverse the array.
- When `nums[j]` is not equal to `nums[i]`, increment `i` and set `nums[i] = nums[j]`.

### 🧩 Code:
```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int n = nums.size();
        int i = 0;

        for(int j = 1; j < n; j++) {
            if(nums[j] != nums[i]) {
                nums[i + 1] = nums[j];
                i++;
            }
        }
        return i + 1;
    }
};
```

### ⏱️ Time Complexity:
- `O(N)` — Single pass through the array.

### 📦 Space Complexity:
- `O(1)` — In-place, no extra space used.
