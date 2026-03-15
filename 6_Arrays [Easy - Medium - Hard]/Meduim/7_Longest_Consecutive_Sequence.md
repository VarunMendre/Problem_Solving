# Longest Consecutive Sequence

## 🧊 Brute Force Approach

### 1. Problem
Given an unsorted array of integers, find the length of the longest sequence of consecutive elements.

### 2. Intuition
We can pick each element and then check how far a consecutive sequence starting from that element goes.

### 3. Approach & Algorithm
- For every element in the array, use a helper function (`linearSearch`) to check if the next consecutive element exists.
- Count the length of such a sequence.
- Maintain and update the longest length seen so far.

### 4. Dry Run
**Input**: [100, 4, 200, 1, 3, 2]  
Check each number:
- Start with 100 → next: 101 (not in array) → length: 1
- Start with 4 → next: 5 (not in array) → length: 1
- Start with 200 → length: 1
- Start with 1 → 2 → 3 → 4 → length: 4 → update longest

### 5. Code
```cpp
#include <bits/stdc++.h>
bool linearSearch(vector<int>&a, int num) {
    int n = a.size();
    for (int i = 0; i < n; i++) {
        if (a[i] == num)
            return true;
    }
    return false;
}
int lengthOfLongestConsecutiveSequence(vector<int> &a, int n) {
    int longest = 1;
    for (int i = 0; i < n; i++) {
        int x = a[i];
        int cnt = 1;
        while (linearSearch(a, x + 1) == true) {
            x += 1;
            cnt += 1;
        }

        longest = max(longest, cnt);
    }
    return longest;
}
```

### 6. Time & Space Complexity
- **Time Complexity**: O(N²) in the worst case because each `linearSearch` takes O(N) and is called N times.
- **Space Complexity**: O(1), constant space usage.

---

## 🚀 Better Approach

### 1. Problem
Same as above: find the longest consecutive sequence length.

### 2. Intuition
Sorting helps group consecutive elements together.

### 3. Approach & Algorithm
- Sort the array first.
- Use a single pass to count lengths of consecutive elements.
- Avoid duplicates in sequence counting.

### 4. Dry Run
**Input**: [100, 4, 200, 1, 3, 2]  
After sorting: [1, 2, 3, 4, 100, 200]  
Consecutive 1-2-3-4 → length: 4

### 5. Code
```cpp
#include <bits/stdc++.h>

int lengthOfLongestConsecutiveSequence(vector<int> &arr, int n) {
    if(n == 0) return 0;
    sort(arr.begin(),arr.end());
    int currCnt = 0;
    int lastSmaller = INT_MIN;
    int longest = 1;
    for(int i=0;i<n;i++) {
        if(arr[i] - 1 == lastSmaller) {
            currCnt += 1;
            lastSmaller = arr[i];
        }
        else if(lastSmaller != arr[i]){
            currCnt =1;
            lastSmaller = arr[i];
        }
        longest = max(longest, currCnt);
    }
    return longest;
}
```

### 6. Time & Space Complexity
- **Time Complexity**: O(N log N) for sorting + O(N) for traversal → total O(N log N)
- **Space Complexity**: O(1) (excluding sort's space cost, assuming in-place)

---

## ⚡ Optimal Approach

### 1. Problem
Same as before: find the length of the longest consecutive sequence.

### 2. Intuition
Use a set to store elements and start counting only from elements that could be the beginning of a sequence.

### 3. Approach & Algorithm
- Add all elements into a hash set (unordered_set).
- For each element, check if it's the start of a sequence (i.e., no element before it).
- Count consecutive elements from it.
- Update max length accordingly.

### 4. Dry Run
**Input**: [100, 4, 200, 1, 3, 2]  
Set = {1, 2, 3, 4, 100, 200}  
Start from 1 → 2 → 3 → 4 → count = 4

### 5. Code
```cpp
#include <bits/stdc++.h>

int lengthOfLongestConsecutiveSequence(vector<int> &arr, int n) {
    if(n==0)return 0;
    int longest = 0;
    unordered_set< int >st;
    for(int i=0;i<n;i++) {
        st.insert(arr[i]);
    }
    for(auto it : st) {
        if(st.find(it - 1) == st.end()) {
            int cnt = 1;
            int x = it;
            while(st.find(x+1) != st.end()){
                cnt += 1;
                x += 1;
            }
            longest = max(longest, cnt);
        }
    }
    return longest;
}
```

### 6. Time & Space Complexity
- **Time Complexity**: 
  - O(N) to insert all elements into the set
  - O(N) to iterate through set elements
  - O(N) for worst case of counting up sequences → total = O(3N) = O(N)
- **Space Complexity**: O(N) for the set to store unique elements.