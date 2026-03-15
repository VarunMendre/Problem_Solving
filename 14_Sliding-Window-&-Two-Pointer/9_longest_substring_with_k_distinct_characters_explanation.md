# Longest Substring With K Distinct Characters

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

Goal:
Find the length of the longest substring that contains at most `k` distinct characters.

Brute Force Idea:
- Try every possible starting index `i`.
- For each `i`, extend substring using `j`.
- Maintain a hashmap `freq` to store character frequency.
- If number of distinct characters exceeds `k` → break.
- Otherwise update maximum length.

We explore all substrings until they become invalid.

---

## 2. Code

```cpp
#include<bits/stdc++.h>
int kDistinctChars(int k, string &s)
{
    int maxLength = 0;

    for (int i = 0; i < s.size(); i++) {
        unordered_map<char, int> freq;

        for (int j = i; j < s.size(); j++) {
            freq[s[j]]++;

            if (freq.size() > k) break;

            maxLength = max(maxLength, j - i + 1);
        }
    }
    return maxLength;
}
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
  - Two nested loops
- **Space Complexity:** O(K)
  - At most `k` distinct characters stored in hashmap

---

# 2️⃣ Optimal Approach (Sliding Window)

## 1. Approach / Intuition

Key Insight:
Instead of restarting for every `i`, we use two pointers:
- `l` → left pointer
- `r` → right pointer

Steps:
1. Expand window by moving `r`.
2. Insert/update character frequency in hashmap.
3. If distinct characters > k:
   - Shrink window from left
   - Decrease frequency
   - Remove character if frequency becomes 0
4. Update maximum length whenever window is valid.

This ensures:
- Each character is processed at most twice.
- No unnecessary recomputation.

---

## 2. Code

```cpp
#include<bits/stdc++.h>
int kDistinctChars(int k, string &s)
{
    if (k == 0 || s.empty()) return 0;
    int n = s.size();
    int maxLen = 0, r = 0, l = 0;

    unordered_map<char, int> mpp;

    while(r < n) {
        mpp[s[r]]++;

        if(mpp.size() > k) {
            mpp[s[l]]--;
            if(mpp[s[l]] == 0){
                mpp.erase(s[l]);
            }
            l += 1;
        }

        if(mpp.size() <= k) {
            maxLen = max(maxLen, r - l + 1);
        }

        r += 1;
    }

    return maxLen;
}
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N)
  - Each character enters and leaves the window at most once

- **Space Complexity:** O(K)
  - Hashmap stores at most `k` distinct characters

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Technique |
|-----------|-----------------|--------|-----------|
| Brute | O(N²) | O(K) | Check all substrings |
| Optimal | O(N) | O(K) | Sliding Window |

---

Structured clearly for:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Perfect for interview revision and sliding window pattern mastery.

