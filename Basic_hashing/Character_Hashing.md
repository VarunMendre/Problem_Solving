
# Character Hashing

## Problem Statement

Given a string, for example: `"abcdabefc"` and a list of queries `['a', 'c', 'z']`, the task is to find the frequency of each queried character in the string.

**Example:**
- For query `'a'`, output → `2`
- For query `'c'`, output → `2`
- For query `'z'`, output → `0`

---

## 🔸 Approach 1: Brute Force

### 🔹 Logic

For every query character, iterate over the entire string and count its occurrences.

### 🔹 Code

```cpp
int countChar(char c, string s) {
    int cnt = 0;
    for (int i = 0; i < s.size(); i++) {
        if (s[i] == c) {
            cnt++;
        }
    }
    return cnt;
}
```

### 🔹 Time Complexity

- For `Q` queries and string length `N`, the time complexity is:  
  **O(Q * N)**

---

## 🔸 Approach 2: Optimized using Character Hashing

### 🔹 Logic

Precompute the frequency of all characters in a **hash array**, and use it to answer queries in constant time.

We can use different mapping strategies depending on the character types:

---

### ✅ Case 1: Lowercase Letters Only

- Map: `'a' -> 0, 'b' -> 1, ..., 'z' -> 25`  
- Array size: `26`

#### 🔹 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;

    // Precompute frequency
    int hash[26] = {0};
    for (int i = 0; i < s.size(); i++) {
        hash[s[i] - 'a']++;
    }

    int q;
    cin >> q;
    while (q--) {
        char ch;
        cin >> ch;
        cout << hash[ch - 'a'] << endl;
    }
    return 0;
}
```

#### 🔹 Time Complexity

- Preprocessing: **O(N)**
- Each Query: **O(1)**
- Total: **O(N + Q)**

---

### ✅ Case 2: Uppercase Letters Only

- Map: `'A' -> 0, 'B' -> 1, ..., 'Z' -> 25`  
- Array size: `26`

#### 🔹 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;

    int hash[26] = {0};
    for (int i = 0; i < s.size(); i++) {
        hash[s[i] - 'A']++;
    }

    int q;
    cin >> q;
    while (q--) {
        char ch;
        cin >> ch;
        cout << hash[ch - 'A'] << endl;
    }
    return 0;
}
```

---

### ✅ Case 3: Mixed Case or All ASCII Characters

- Array size: `256` (covers all ASCII characters)

#### 🔹 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    string s;
    cin >> s;

    int hash[256] = {0};
    for (int i = 0; i < s.size(); i++) {
        hash[s[i]]++;
    }

    int q;
    cin >> q;
    while (q--) {
        char ch;
        cin >> ch;
        cout << hash[ch] << endl;
    }
    return 0;
}
```

#### 🔹 Time Complexity

- Preprocessing: **O(N)**
- Each Query: **O(1)**
- Total: **O(N + Q)**

---

## ✅ Summary

| Approach               | Preprocessing Time | Query Time | Total Time     | Space Complexity |
|------------------------|--------------------|------------|----------------|------------------|
| Brute Force            | O(1)               | O(N)       | O(Q * N)       | O(1)             |
| Hashing (Lowercase)    | O(N)               | O(1)       | O(N + Q)       | O(26)            |
| Hashing (Uppercase)    | O(N)               | O(1)       | O(N + Q)       | O(26)            |
| Hashing (Full ASCII)   | O(N)               | O(1)       | O(N + Q)       | O(256)           |

---

> ✅ **Recommendation:** Use **Case 3 (256-sized hash array)** in practice to handle all cases (uppercase, lowercase, symbols, etc.) reliably without error.
