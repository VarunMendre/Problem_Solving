# 🧠 Recursion - Introduction  
> Understand Recursion by Printing Something N Times

---

## ✅ Pre-requisite  
You should be comfortable with:
- Writing a basic function in any language.
- Making function calls from the `main()` function.

---

## 📌 What is Recursion?  
Recursion is the process where a function calls itself until a certain condition (known as the base case) is met. It is often used to solve problems that can be broken down into smaller, similar subproblems.

### 🧠 Example Illustration:
Imagine a function calling itself without a stopping condition:

```cpp
void func() {
    func(); // keeps calling itself endlessly
}
```

This will result in a **Stack Overflow**.

---

## 💥 What is Stack Overflow in Recursion?

Each time a function calls itself, it is pushed onto the **call stack**.  
If no **base condition** is provided, the stack grows indefinitely until it exceeds memory, resulting in a **Stack Overflow** or **Segmentation Fault**.

---

## 🧱 Base Condition

A **base condition** is used to stop further recursive calls and allow the function to return.

### 🧾 Pseudocode:
```cpp
int count = 0;

void func() {
    if (count == 3) return;
    print(count);
    count++;
    func();
}
```

### ✅ Output:
```
0
1
2
```

---

## 💻 Code Implementation (C++)

```cpp
#include<bits/stdc++.h>
using namespace std;

int cnt = 0;

void print() {
    if(cnt == 3) return;   // Base Condition
    cout << cnt << endl;   // Action
    cnt++;                 // Progression
    print();               // Recursive Call
}

int main() {
    print();
    return 0;
}
```

---

## 🌳 Recursive Tree

A **recursion tree** helps visualize how recursive calls are made and how they return. Each level of the tree represents a function call and its return.

For example:
```
print(0)
  └── print(1)
        └── print(2)
              └── return
        └── return
  └── return
```

---

## 📚 Summary

- ✅ What is Recursion?
- ✅ Base Condition
- ✅ Stack Overflow / Stack Space
- ✅ Recursion Tree

---

_This is part of my DSA Learning Journey. Stay tuned for more recursion problems and solutions!_