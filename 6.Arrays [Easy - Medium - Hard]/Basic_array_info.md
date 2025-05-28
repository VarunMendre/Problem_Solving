# 📘 Arrays - Basic Concepts in Data Structures

Welcome to the Arrays module in our DSA journey! This file contains a beginner-friendly and detailed explanation of arrays, including declaration, initialization, memory details, and usage.

---

## 1️⃣ Definition of Array

An **array** is a **linear data structure** that stores elements **of the same data type** in **contiguous memory locations**. It is used to store a **fixed-size** sequential collection of elements.

> Similar to a row of lockers where each locker stores a value and is accessible by its index (locker number).

---

## 2️⃣ How to Declare an Array

```cpp
// Syntax:
data_type array_name[size];

// Example:
int marks[5];  // Declares an array of 5 integers
```

---

## 3️⃣ How to Initialize an Array

```cpp
// Method 1: Static Initialization
int arr[5] = {10, 20, 30, 40, 50};

// Method 2: Partial Initialization (rest will be 0)
int arr[5] = {1, 2};  // arr = {1, 2, 0, 0, 0}

// Method 3: Initialization with size deduction
int arr[] = {5, 10, 15};  // Compiler sets size to 3
```

---

## 4️⃣ Maximum Size of Array

The maximum size you can declare depends on **where** the array is declared:

| Location     | Max Size           | Note                               |
|--------------|--------------------|------------------------------------|
| Inside `main()` (stack) | `10^6` elements (approx) | Stack memory is limited (~1MB)   |
| Global Scope (global/static) | `10^7` elements (approx) | Uses heap/data segment (~10MB)  |

> ⚠️ Larger sizes may cause segmentation faults due to memory overflow.

---

## 5️⃣ How to Access Elements in an Array

Array elements are accessed using the index starting from `0`.

```cpp
int arr[5] = {10, 20, 30, 40, 50};

// Accessing elements:
cout << arr[0];  // Output: 10
cout << arr[2];  // Output: 30

// Modifying element:
arr[1] = 100;    // arr becomes {10, 100, 30, 40, 50}
```

---

## 6️⃣ Where is Array Stored in Memory?

- When you declare an array, a **contiguous block** of memory is allocated.
- The array's **base address** (address of the first element) is assigned to the array variable.
- The array elements are stored at consecutive memory locations.

> For example:
```cpp
int arr[3] = {5, 10, 15};
```
Suppose `arr` starts at memory address `0x61fe00`, then:

| Index | Value | Memory Address |
|-------|-------|----------------|
| arr[0] | 5     | 0x61fe00       |
| arr[1] | 10    | 0x61fe04       |
| arr[2] | 15    | 0x61fe08       |

(Each `int` takes 4 bytes, hence the gap)

---

## ✅ Summary

- Arrays store data of the same type in a contiguous block of memory.
- Can be declared globally or locally with size limitations.
- Elements are accessed by index, starting from 0.
- Memory is allocated sequentially and can be accessed using pointers or index.

---

🚀 Keep practicing with examples and problems to build a strong base in arrays!
