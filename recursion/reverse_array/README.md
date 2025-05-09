# 🔁 Recursive Array Reversal in C++

This repository contains two implementations to reverse an array using **recursion** in C++. Each version demonstrates a different approach:

- ✅ Using Two Pointers (`left`, `right`)
- ✅ Using a Single Index (`i`)

---

## 🧩 Problem Statement

You are given an array. The task is to reverse the array using recursion and print the reversed array.

---

## 📌 Example

**Input:**

```
N = 5  
arr = [5, 4, 3, 2, 1]
```

**Output:**

```
1 2 3 4 5
```

---

## 1️⃣ Solution 1: Two Pointer Approach

### 🔍 Approach

- Use two pointers: one from the beginning (`l`) and one from the end (`r`).
- Recursively swap `arr[l]` and `arr[r]`, then move the pointers inward (`l + 1`, `r - 1`) until `l >= r`.

### 📌 Code

```cpp
void f(int l, int r, int arr[], int n) {
    if (l >= r) return;
    swap(arr[l], arr[r]);
    f(l + 1, r - 1, arr, n);
}
```

### 🌳 Recursion Tree

```
f(0, 4)
├── f(1, 3)
│   └── f(2, 2) → base case
└── unwind and return
```

---

## 2️⃣ Solution 2: Single Pointer Approach

### 🔍 Approach

- Use a single index `i`.
- Swap `arr[i]` and `arr[n - i - 1]`.
- Continue recursively with `i + 1` until `i >= n/2`.

### 📌 Code

```cpp
void f(int i, int arr[], int n) {
    if (i >= n / 2) return;
    swap(arr[i], arr[n - i - 1]);
    f(i + 1, arr, n);
}
```

### 🌳 Recursion Tree

```
f(0) → swap arr[0] and arr[4]
├── f(1) → swap arr[1] and arr[3]
│   └── f(2) → base case
└── unwind and return
```

---

## 🧠 Note

In both methods:
- Only `n/2` swaps are performed.
- The array is reversed **in-place** with **O(n)** time and **O(n)** space due to recursion stack.

---

## 📁 Files

- `using_two_pointers.cpp` – Implements the two-pointer method.
- `using_single_pointer.cpp` – Implements the single-index method.

---

## 📌 Author

This repository is maintained by **Varun Mendre** ✨