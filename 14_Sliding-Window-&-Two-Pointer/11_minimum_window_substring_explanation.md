# Minimum Window Substring

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

Goal:
Find the smallest substring in `s` that contains all characters of string `t` (including duplicates).

Brute Force Idea:
- Fix a starting index `i`.
- For each `i`, build a fresh frequency array for string `t`.
- Expand the window using index `j`.
- Reduce frequency as characters are matched.
- Maintain a counter to track how many characters from `t` are matched.
- When all `m` characters are matched:
  - Update minimum length
  - Break (since we want smallest for that `i`)

This checks almost all possible substrings.

---

## 2. Code

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        int n = s.size(), m = t.size();
        int minLen = INT_MAX, sIndex = -1;

        for(int i = 0; i < n; i++) {

            vector<int> freq(256, 0);
            int count = 0;

            for(int j = 0; j < m; j++) freq[t[j]]++;

            for(int j = i; j < n; j++) {
                if(freq[s[j]] > 0) {
                    count++;
                }
                freq[s[j]]--;

                if(count == m) {
                    if((j - i + 1) < minLen) {
                        minLen = j - i + 1;
                        sIndex = i;
                        break;
                    }
                }
            }
        }

        return sIndex == -1 ? "" : s.substr(sIndex, minLen);
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
- **Space Complexity:** O(256)
  - Fixed character frequency array

---

# 2️⃣ Optimal Approach (Sliding Window)

## 1. Approach / Intuition

This is a classic variable-size sliding window problem.

Core Idea:
- Expand right pointer `r` to include characters.
- Keep track of required characters using a frequency map.
- Maintain a `count` of matched characters.
- When all characters of `t` are matched (`count == m`):
  - Try shrinking from the left to minimize the window.
  - Update minimum length whenever valid.

Important Concepts:
- If `freq[s[r]] > 0`, we matched a needed character.
- Decrement frequency when expanding.
- While window is valid, shrink from left.
- If removing a character makes it required again, reduce `count`.

This ensures each character is processed at most twice.

---

## 2. Code

```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        int n = s.size(), m = t.size();

        int minLen = 1e7, count = 0, sInd = -1;
        int l = 0, r = 0;

        map<char, int> freq;
        for(int i = 0; i < m; i++) freq[t[i]]++;

        while(r < n) {
            if(freq[s[r]] > 0) count++;
            freq[s[r]]--;

            while(count == m) {
                if((r - l + 1) < minLen) {
                    minLen = r - l + 1;
                    sInd = l;
                }

                freq[s[l]]++;
                if(freq[s[l]] > 0) count--;
                l++;
            }

            r++;
        }

        return sInd == -1 ? "" : s.substr(sInd, minLen);
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(2N + M) ≈ O(N)
  - Each character visited at most twice
  - Initial frequency build takes O(M)

- **Space Complexity:** O(256)
  - Frequency map stores at most all character types

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Technique |
|-----------|-----------------|--------|-----------|
| Brute | O(N²) | O(256) | Nested loops |
| Optimal | O(N) | O(256) | Sliding Window |

---

Structured clearly for revision:
1. Intuition
2. Code
3. Complexity

This is one of the most important sliding window interview problems.

