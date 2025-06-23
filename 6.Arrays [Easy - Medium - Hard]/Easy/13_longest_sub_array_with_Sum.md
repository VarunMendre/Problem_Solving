# 13. Longest Subarray with Given Sum

## 1. Brute Force Approach

### ✅ Approach (Explained in Steps):
1. Iterate over all possible starting indices `i` from `0` to `n-1`.
2. For each starting index `i`, iterate over all possible ending indices `j` from `i` to `n-1`.
3. For each pair `(i, j)`, calculate the sum of the subarray from index `i` to `j`.
4. If the sum is equal to the given `k`, calculate the length of this subarray as `(j - i + 1)`.
5. Track the maximum length found among all such subarrays.
6. Return the maximum length.

### 🧠 Algorithm:
```
- Initialize len = 0
- Loop i from 0 to n-1:
    - Loop j from i to n-1:
        - Initialize sum = 0
        - Loop p from i to j:
            - sum += arr[p]
        - If sum == k:
            - len = max(len, j - i + 1)
- Return len
```

### 💻 Code:
```cpp
int getLongestSubarray(vector<int>&arr ,long long target ){
  int n = arr.size();
  int len = 0;

  for(int i=0;i<n;i++){
     for(int j=i ; j<n ;j++){
       long long sum = 0;

       for(int p=i ; p<=j ;p++){
           sum += arr[p];
       }
        if(sum == target){
          len = max(len , j-i+1);
        }
     }
  }
  return len;
}
```

### ⏱️ Time & Space Complexity:
- Time Complexity: **O(N^3)** due to three nested loops
- Space Complexity: **O(1)** — no extra space used

---

## 2. Improved Brute Force (O(N^2))

### 💻 Code:
```cpp
int getLongestSubarray(vector<int>& a, long long k) {
    int n = a.size();
    int len = 0;
    for (int i = 0; i < n; i++) {
        long long s = 0;
        for (int j = i; j < n; j++) {
            s += a[j];
            if (s == k)
                len = max(len, j - i + 1);
        }
    }
    return len;
}
```

### ⏱️ Time & Space Complexity:
- Time Complexity: **O(N^2)** — inner loop removed the third loop
- Space Complexity: **O(1)**

---

## 3. Better Approach using HashMap (Prefix Sum)

### ✅ Approach:
1. Use a prefix sum technique.
2. Keep a hash map to store the first occurrence of each prefix sum.
3. For each element, update the running prefix sum.
4. If the prefix sum is equal to `k`, the subarray from `0` to `i` is valid.
5. Otherwise, check if `prefix_sum - k` exists in the map. If yes, calculate the length.
6. Store the prefix sum in the map only if it doesn’t already exist (to keep the earliest index).

### 🧠 Algorithm:
```
- Initialize map, sum = 0, maxLen = 0
- Loop i from 0 to n-1:
    - sum += a[i]
    - If sum == k:
        - maxLen = i+1
    - Else if (sum - k) exists in map:
        - len = i - map[sum - k]
        - maxLen = max(maxLen, len)
    - If sum not in map:
        - map[sum] = i
- Return maxLen
```

### 💻 Code:
```cpp
int getLongestSubarray(vector<int>& a, long long k) {
    int n = a.size();
    map<long long, int> preSumMap;
    long long sum = 0;
    int maxLen = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
        if (sum == k) {
            maxLen = max(maxLen, i + 1);
        }
        long long rem = sum - k;
        if (preSumMap.find(rem) != preSumMap.end()) {
            int len = i - preSumMap[rem];
            maxLen = max(maxLen, len);
        }
        if (preSumMap.find(sum) == preSumMap.end()) {
            preSumMap[sum] = i;
        }
    }
    return maxLen;
}
```

### ⏱️ Time & Space Complexity:
- Time Complexity: **O(N * log N)** (due to map operations)
- Space Complexity: **O(N)** (for the map)

---

## 4. Optimal Approach (Two Pointers / Sliding Window) [Only for **positive** numbers]

### ✅ Approach:
1. Use two pointers `left` and `right` to maintain a sliding window.
2. Keep a running sum of elements in the current window.
3. While the sum is greater than `k`, shrink the window from the left.
4. If sum equals `k`, update the maximum length.
5. Expand the window by moving the `right` pointer.

### 🧠 Algorithm:
```
- Initialize left = 0, right = 0, sum = a[0], maxLen = 0
- While right < n:
    - While sum > k and left <= right:
        - sum -= a[left]; left++
    - If sum == k:
        - maxLen = max(maxLen, right - left + 1)
    - right++
    - If right < n:
        - sum += a[right]
- Return maxLen
```

### 💻 Code:
```cpp
int longestSubarrayWithSumK(vector<int> a, long long k) {
    int left = 0, right = 0;
    int maxLen = 0;
    long long sum = a[0];
    int n = a.size();

    while(right < n){
        while(left<=right && sum > k){
            sum -= a[left];
            left++;
        }
        if(sum == k){
            maxLen = max(maxLen , right - left + 1);
        }
        right++;
        if(right < n)
            sum += a[right];
    }
    return maxLen;
}
```

### ⏱️ Time & Space Complexity:
- Time Complexity: **O(2N)** – Each element is visited at most twice
- Space Complexity: **O(1)** – No extra space used
