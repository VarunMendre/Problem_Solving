
# 🔑 Basic Hashing – Introduction

Hashing is a technique used to **optimize lookups and frequency counts**. It is widely used when we want to reduce time complexity by avoiding repetitive loops.

---

## 📘 Problem Example

Given an array:  
`[1, 2, 1, 3, 2]`  
And queries:  
`[1, 3, 4, 2, 10]`  
We must find the number of times each query element appears in the array.

Expected Output:  
`2 1 0 2 0`

---

## ❌ Brute Force Approach

We iterate over the array for each query and count occurrences manually.

### 🔗 Code
```cpp
int F(int number, int arr[], int n) {
    int cnt = 0;
    for(int i = 0; i < n; i++) {
        if(arr[i] == number)
            cnt++;
    }
    return cnt;
}
```

### 🧮 Time Complexity:  
- O(N) for each query  
- Total: **O(Q × N)**  
If both `N` and `Q` are large (e.g., 1e5), this results in 1e10 operations — **too slow!**

---

## ✅ Optimized Approach using Hashing

### ✨ Hashing = Precompute + Fetch

We store the frequency of each element once, then fetch it instantly for each query.

### 🔗 Code
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;
    int arr[n];
    for(int i = 0; i < n; i++) cin >> arr[i];

    // Precompute
    int hash[13] = {0};
    for(int i = 0; i < n; i++) {
        hash[arr[i]] += 1;
    }

    // Fetch
    int q;
    cin >> q;
    while(q--) {
        int num;
        cin >> num;
        cout << hash[num] << " ";
    }
}
```

### ⏱️ Time Complexity:
- Precompute: O(N)  
- Each Query: O(1)  
- Total: **O(N + Q)** → Highly efficient!

---

## ⚠️ Note on Limits

If elements are large (e.g., up to 1e9), arrays are **not suitable** due to memory constraints.  
We will use **maps** or **unordered_maps** in such cases (covered later).

---

## 🧩 Next Module: Character Hashing

Now that you've learned **number hashing**, we'll explore how to use similar logic for characters and strings.
