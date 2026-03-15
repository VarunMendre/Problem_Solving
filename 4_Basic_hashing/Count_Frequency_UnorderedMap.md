# 🔢 Count Frequency of Each Element in the Array (Using `unordered_map`)

## 🚀 Why Use `unordered_map`?

- `unordered_map` is a **hash table-based container** that provides:
  - 🟢 **Average Case Time Complexity:** `O(1)` for insert, delete, and lookup.
  - 🟢 **Best Case Time Complexity:** `O(1)` (when no or few collisions).
  - 🔴 **Worst Case Time Complexity:** `O(N)` (in rare cases due to **hash collisions**).

## 🛠️ How Collisions are Handled Internally (Briefly):

- It uses **division method**: `hash(key) % table_size` to compute bucket index.
- **Collisions** (when two keys hash to the same bucket) are handled using **chaining** (linked list or similar data structure per bucket).
- These are low-level implementations handled internally by STL; no extra code is required from the programmer.

---

## ✅ Example Code

```cpp
#include <bits/stdc++.h>
using namespace std;

void Frequency(int arr[], int n)
{
    unordered_map<int, int> map;

    for (int i = 0; i < n; i++)
        map[arr[i]]++;  // Increment frequency of each element

    // Traverse through map and print frequencies
    for (auto x : map)
        cout << x.first << " -> " << x.second << endl;
}

int main()
{
    int arr[] = {10, 5, 10, 15, 10, 5};
    int n = sizeof(arr) / sizeof(arr[0]);
    Frequency(arr, n);
    return 0;
}
```

## 🧾 Output

```
15 -> 1
5 -> 2
10 -> 3
```

> Note: The output order may vary due to unordered nature of the map.