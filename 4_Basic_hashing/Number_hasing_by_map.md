
# 📌 Number Hashing using `map` in C++

Hashing is a very useful technique when we want to **store and retrieve information efficiently**, especially when the problem involves **frequency counting** or **existence checks**. Let’s explore how this can be done using `map` in C++.

---

## 💡 What is `map` in C++?

A `map` is a type of **associative container** in C++ STL which stores data in **key-value** pairs. Think of it as a dictionary where:

- **Key** ➤ The item we want to store information about.
- **Value** ➤ The information itself (like frequency, position, etc.).

---

## 🧠 Why use a `map` for number hashing?

Let’s say we are given an array:
```
[1, 2, 3, 1, 3, 2, 12]
```
We want to **store the frequency of each number** in the array.

Here’s how we can do it using a `map<int, int>`:
```cpp
map<int, int> mp;
for (int i = 0; i < n; i++) {
    mp[arr[i]]++;
}
```

Now we can fetch the frequency of any number in constant time (logarithmic under the hood).

---

## 📊 Difference between **array hashing** and **map hashing**

Let’s consider the same array:  
```
[1, 2, 3, 1, 3, 2, 12]
```

- **Array Hashing**: You need to declare an array of size `13` (as 12 is the largest element).
- **Map Hashing**: You don’t need to worry about the maximum value; it dynamically stores only the keys that exist.

So, map is more **space-efficient**, especially when the data is sparse or keys are very large numbers.

---

## ✅ Key Point

If you try to access a key that doesn't exist in the map:

- C++ `map` returns `0`
- Java `HashMap` returns `null`

This is handy when checking frequencies or existence.

---

## 🧪 Example Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;
    int arr[n];

    for (int i = 0; i < n; i++) {
        cin >> arr[i];
    }

    map<int, int> mp;
    for (int i = 0; i < n; i++) {
        mp[arr[i]]++;
    }

    int q;
    cin >> q;
    while (q--) {
        int number;
        cin >> number;
        cout << mp[number] << endl;
    }

    return 0;
}
```

### 🔹 Input:
```
7
1 2 3 1 3 2 12
5
1 2 3 4 12
```

### 🔹 Output:
```
2
2
2
0
1
```

---

## ⏱️ Time Complexity

- Insertion and Retrieval in `map`: **O(log N)**  
  (Because it's implemented as a Red-Black Tree under the hood)

So, for N elements:  
**Overall Time Complexity** = `O(N * log N)`

---

## 🚀 Better Option? `unordered_map`

- `unordered_map` (in C++) or `HashMap` (in Java) provides **O(1)** average time complexity for insertion and lookup.
- But in **worst-case scenarios**, due to **hash collisions**, the time complexity can degrade to **O(N)**.

---

## 🧯 What’s Next?

> In our next topic, we’ll explore `unordered_map` in detail and understand how **collisions** happen, how they’re handled, and when to use `unordered_map` vs `map`.

**Stay tuned! 🔥**
