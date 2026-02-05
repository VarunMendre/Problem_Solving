# 🧠 Longest Substring Without Repeating Characters

---

## 📌 Problem Statement

Given a string `s`, find the **length of the longest substring** without repeating characters.

---

## 1️⃣ Intuition / Approach

This problem is a classic example of **substring + frequency tracking** and is best understood by comparing **Brute Force vs Sliding Window**.

---

## 🐌 Brute Force Approach

### 💡 Intuition

- Fix a starting index `i`
- Extend the substring character by character from `i` to `j`
- Stop when a duplicate character appears
- Track the maximum length

---

### 🔍 How Duplication is Detected

- Use a `hash[256]` array
- Each character is marked when visited
- If already marked → break

---

### 🔁 Visual Flow

```
for i = 0 → n-1
  reset hash
  for j = i → n-1
     if char already seen → break
     update answer
```

---

## 2️⃣ Code (Brute Force)

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int n = s.size();
        int maxi = 0;

        for (int i = 0; i < n; i++) {
            string sub;
            vector<int> hashChar(256, 0);

            for (int j = i; j < n; i++) {

                if (hashChar[j] == 1)
                    break;

                int len = j - i + 1;
                maxi = max(maxi, len);
                hashChar[s[j]] = 1;
            }
        }

        return maxi;
    }
};
```

---

### ⏱ Time & Space Complexity (Brute Force)

| Metric | Complexity |
| ------ | ---------- |
| Time   | **O(N²)**  |
| Space  | **O(256)** |

---

## ⚡ Optimal Approach (Sliding Window)

---

### 💡 Intuition

Instead of restarting from scratch:

- Maintain a **window [left, right]**
- Expand `right`
- If duplicate appears, **move ****************************left**************************** forward**
- Store the **last index** of every character

---

### 🧠 Key Idea

```
hash[c] = last index where character c appeared
```

If a character repeats **inside the window**, shift `left`.

---

### 🔁 Visual Sliding Window

```
[left ...... right]

No duplicate → expand window
Duplicate → shrink from left
```

---

## 3️⃣ Code (Optimal)

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int left = 0;
        int right = 0;

        int n = s.size();
        int maxLen = 0;

        vector<int>hash(256, -1);

        while(right < n) {
            if(hash[s[right]] != -1) {
                if(hash[s[right]] >= left) {
                    left = hash[s[right]] + 1;
                }
            }

            int len = right - left + 1;
            maxLen = max(maxLen, len);

            hash[s[right]] = right;
            right++;
        }   

        return maxLen;
    }
};
```

---

## 4️⃣ Time & Space Complexity (Optimal)

| Metric | Complexity |
| ------ | ---------- |
| Time   | **O(N)**   |
| Space  | **O(256)** |

---

## 🆚 Brute Force vs Optimal

| Feature            | Brute Force  | Sliding Window |
| ------------------ | ------------ | -------------- |
| Time               | O(N²)        | O(N)           |
| Space              | O(256)       | O(256)         |
| Technique          | Nested loops | Two pointers   |
| Interview Friendly | ❌            | ✅              |

