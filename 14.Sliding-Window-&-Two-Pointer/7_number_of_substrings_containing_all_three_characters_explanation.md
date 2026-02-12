# Number of Substrings Containing All Three Characters

Given a string `s` consisting only of characters `a`, `b`, and `c`.
Return the number of substrings containing at least one occurrence of all three characters.

---

# 1️⃣ Brute Force Approach

## 1. Approach / Intuition

- Try every possible starting index `i`.
- For each `i`, extend substring using `j`.
- Maintain a frequency array of size 3 (for `a`, `b`, `c`).
- For every substring `[i...j]`:
  - Update frequency.
  - Check if all three characters are present.
  - If yes → increment count.

This checks every possible substring.

---

## 2. Code

```cpp
class Solution {
public:
    int numberOfSubstrings(string s) {
        int n = s.size();
        int count = 0;

        for(int i = 0; i < n; i++) {
            vector<int> freq(3, 0);

            for(int j = i; j < n; j++) {
                freq[s[j] - 'a']++;

                if(freq[0] > 0 && freq[1] > 0 && freq[2] > 0) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
  - Two nested loops
- **Space Complexity:** O(3) = O(1)

---

# 2️⃣ Better Approach (Early Break Optimization)

## 1. Approach / Intuition

Observation:
Once substring `[i...j]` contains all three characters,
any extension beyond `j` will also contain all three.

So instead of checking further individually,
we can directly add:

```
(n - j)
```

Why?
Because all substrings from `[i...j]`, `[i...j+1]`, ..., `[i...n-1]`
will remain valid.

This reduces unnecessary checks.

---

## 2. Code

```cpp
class Solution {
public:
    int numberOfSubstrings(string s) {
        int n = s.size();
        int count = 0;

        for(int i = 0; i < n; i++) {
            vector<int> freq(3, 0);

            for(int j = i; j < n; j++) {
                freq[s[j] - 'a']++;

                if(freq[0] > 0 && freq[1] > 0 && freq[2] > 0) {
                    count += (n - j);
                    break;
                }
            }
        }

        return count;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N²)
  - Worst case still quadratic
- **Space Complexity:** O(3) = O(1)

---

# 3️⃣ Optimal Approach (Last Seen Index Trick)

## 1. Approach / Intuition

Key Insight:

Instead of checking every substring,
track the **last seen index** of each character.

Maintain array `freq[3]`:
- `freq[0]` → last index of 'a'
- `freq[1]` → last index of 'b'
- `freq[2]` → last index of 'c'

At each index `i`:
- Update last seen position of current character.
- If all three characters have appeared:
  - The smallest last seen index determines
    how many valid substrings end at `i`.

Why?
If minimum last index is `m`,
then all substrings starting from `0` to `m`
and ending at `i` are valid.

So we add:

```
1 + min(freq[0], freq[1], freq[2])
```

This gives total substrings ending at `i` that contain all three characters.

---

## 2. Code

```cpp
class Solution {
public:
    int numberOfSubstrings(string s) {
        int n = s.size();
        vector<int> freq(3, -1);
        int count = 0;

        for(int i = 0; i < n; i++) {
            freq[s[i] - 'a'] = i;

            if(freq[0] != -1 && freq[1] != -1 && freq[2] != -1) {
                count += (1 + min({freq[0], freq[1], freq[2]}));
            }
        }

        return count;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(N)
  - Single pass
- **Space Complexity:** O(3) = O(1)

---

# ✅ Final Comparison

| Approach | Time Complexity | Space | Idea |
|-----------|-----------------|--------|------|
| Brute | O(N²) | O(1) | Check all substrings |
| Better | O(N²) | O(1) | Early break + counting trick |
| Optimal | O(N) | O(1) | Last seen index technique |

---

Structured clearly for:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Perfect for c