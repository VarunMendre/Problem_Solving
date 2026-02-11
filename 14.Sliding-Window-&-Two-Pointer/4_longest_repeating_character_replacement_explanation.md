# Longest Repeating Character Replacement

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

- We try every possible starting index `i`.
- For each `i`, we extend the substring using another pointer `j`.
- We maintain a frequency array of size 26 (since only uppercase letters).
- For every substring `[i...j]`:
  - Track the maximum frequency character (`maxFreq`).
  - Number of changes required = (window length - maxFreq).
- If changes ≤ k → valid substring.
- If changes > k → break (no need to extend further from this `i`).
- Keep updating maximum length.

This checks all possible substrings.

---

## 2. Code

```cpp
class Solution {
public:
    int characterReplacement(string s, int k) {
        int n = s.size();

        int maxLen = 0;

        for(int i = 0; i < n; i++) {
            vector<int>mpp(26, 0);
            int maxFreq = 0;

            for(int j = i; j < n; j++) {
                mpp[s[j] - 'A']++;

                maxFreq = max(maxFreq, mpp[s[j] - 'A']);
                int changes = (j - i + 1) - maxFreq;

                if(changes <= k) {
                    maxLen = max(maxLen, j - i + 1);
                }else {
                    break;
                }
            }
        }

        return maxLen;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
  - Outer loop → N
  - Inner loop → up to N
- **Space Complexity:** O(26)
  - Frequency array of 26 characters

---

# 2️⃣ Better Approach (Sliding Window with Recalculation)

## 1. Approach / Intuition

- Use Sliding Window (Two Pointers).
- Expand window using `r` pointer.
- Maintain:
  - Frequency array
  - `maxFreq` inside window
- If window becomes invalid:
  - (window size - maxFreq) > k
  - Shrink from left
  - Recalculate `maxFreq` by scanning all 26 characters

Why recalculate?
Because when we shrink, previous `maxFreq` might not be valid anymore.

---

## 2. Code

```cpp
class Solution {
public:
    int characterReplacement(string s, int k) {
        int n = s.size();
        int l = 0, r = 0;
        int maxLen = 0, maxFreq = 0;

        vector<int>mpp(26, 0);
        
        while(r < n) {
            mpp[s[r] - 'A']++;

            maxFreq = max(maxFreq, mpp[s[r] - 'A']);

            while((r - l + 1) - maxFreq > k) {
                mpp[s[l] - 'A']--;
                maxFreq = 0;
                for(int i = 0; i < 26; i++)
                    maxFreq = max(maxFreq, mpp[i]);
                l+=1;
            }

            if((r - l + 1) - maxFreq <= k) {
                maxLen = max(maxLen, r - l + 1);
            }

            r++;
        }

        return maxLen;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:**
  - O(N) for sliding window
  - Each shrink recalculates maxFreq → O(26)
  - Overall: O(N × 26) ≈ O(N)

- **Space Complexity:** O(26)

---

# 3️⃣ Optimal Approach (Best Sliding Window)

## 1. Approach / Intuition

Key Observation:
We do NOT need to decrease `maxFreq` when shrinking.

Why?
Because even if `maxFreq` becomes slightly outdated, the window size will automatically adjust.

We:
- Expand window using `r`
- Update `maxFreq`
- If window invalid → shrink once from left
- No need to recompute `maxFreq`

This works because we are only interested in the maximum valid window length.

This is the most efficient solution.

---

## 2. Code

```cpp
class Solution {
public:
    int characterReplacement(string s, int k) {
        int n = s.size();
        int l = 0, r = 0;
        int maxLen = 0, maxFreq = 0;

        vector<int>mpp(26, 0);
        
        while(r < n) {
            mpp[s[r] - 'A']++;

            maxFreq = max(maxFreq, mpp[s[r] - 'A']);

            if((r - l + 1) - maxFreq > k) {
                mpp[s[l] - 'A']--;
                l+=1;
            }

            if((r - l + 1) - maxFreq <= k) {
                maxLen = max(maxLen, r - l + 1);
            }

            r++;
        }

        return maxLen;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N)
  - Each character processed at most twice
- **Space Complexity:** O(26)

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Idea |
|-----------|-----------------|--------|------|
| Brute | O(N²) | O(26) | Try all substrings |
| Better | O(N × 26) | O(26) | Sliding window + recompute |
| Optimal | O(N) | O(26) | Sliding window without recompute |

---

This structure follows:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Perfect for revision and interview preparation.

