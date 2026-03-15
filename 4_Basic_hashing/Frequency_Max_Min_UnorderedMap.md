# 🔢 Count Frequency of Each Element in the Array (Using `unordered_map`) – With Max & Min Frequency

## 🚀 Why Use `unordered_map`?

- `unordered_map` is a **hash table-based container** that provides:
  - 🟢 **Average Case Time Complexity:** `O(1)` for insert, delete, and lookup.
  - 🟢 **Best Case Time Complexity:** `O(1)` (when no or few collisions).
  - 🔴 **Worst Case Time Complexity:** `O(N)` (in rare cases due to **hash collisions**).

## 🛠️ How Collisions are Handled Internally (Briefly):

- Uses **division method**: `hash(key) % table_size` to compute bucket index.
- **Collisions** are handled using **chaining** (linked list or similar per bucket).
- No need to implement these manually — handled by the STL.

---

## ✅ Example Code – Count Frequencies & Identify Max/Min Frequency Elements

```cpp
#include <bits/stdc++.h>
using namespace std;

void Frequency(int arr[], int n)
{
    unordered_map<int, int> map;

    // Count frequency of each element
    for (int i = 0; i < n; i++)
        map[arr[i]]++;

    int maxFeq = 0, minFeq = n;
    int maxEle = 0, minEle = 0;

    // Find elements with highest and lowest frequency
    for (auto it : map) {
        int count = it.second;
        int element = it.first;

        if (count > maxFeq) {
            maxFeq = count;
            maxEle = element;
        }
        if (count < minFeq) {
            minFeq = count;
            minEle = element;
        }
    }

    cout << "The highest frequency element is: " << maxEle << " (Frequency: " << maxFeq << ")\n";
    cout << "The lowest frequency element is: " << minEle << " (Frequency: " << minFeq << ")\n";
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
The highest frequency element is: 10 (Frequency: 3)
The lowest frequency element is: 15 (Frequency: 1)
```

## 📘 Explanation

- First, we create an `unordered_map` to count how many times each element appears in the array.
- Then we loop through the map to determine:
  - The **maximum frequency element** (appears most times).
  - The **minimum frequency element** (appears least times).
- This helps in frequency analysis which is useful in problems like **mode calculation**, **data compression**, or **majority element detection**.

> Note: Output order of elements in `unordered_map` is **not guaranteed** because it's based on hashing.