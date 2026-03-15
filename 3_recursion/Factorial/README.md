
# Factorial Calculation Using Recursion in C++

This repository contains two implementations for computing the **factorial** of a given number `n` using **recursion** in C++. Each approach is explained with code, output, and a recursion tree.

---

## 🧮 Problem Statement

Given a number `n`, compute its factorial.

**Example:**

```
Input: 4  
Output: 24  
Explanation: 4! = 4 × 3 × 2 × 1 = 24
```

---

## 🔁 Method 1: Functional Way

In this approach, the function returns the result of the recursive call directly, without using extra parameters.

### 🔧 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int fact(int n){
  if(n == 0) return 1;
  return n * fact(n - 1);
}

int main() {
  int n;
  cin >> n;
  cout << fact(n);
}
```

### 🌳 Recursion Tree

```
fact(4)
|
--> 4 * fact(3)
         |
         --> 3 * fact(2)
                  |
                  --> 2 * fact(1)
                           |
                           --> 1 * fact(0)
                                    |
                                    --> 1
```

### ✅ Output for Input `4`

```
24
```

---

## 📥 Method 2: Parameterised Way (Tail Recursion)

This version uses a helper parameter (`sum`) to accumulate the result, making it suitable for **tail recursion** optimization.

### 🔧 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

void fact(int i , int sum){
  if(i < 1){
    cout << sum;
    return;
  }
  fact(i - 1, sum * i);
}

int main() {
  int n;
  cin >> n;
  fact(n, 1);
}
```

### 🌳 Recursion Tree

```
fact(4, 1)
   |
   └── fact(3, 4)
         |
         └── fact(2, 12)
               |
               └── fact(1, 24)
                     |
                     └── fact(0, 24)
                            |
                            └── print 24
```

### ✅ Output for Input `4`

```
24
```

---

## 📌 Conclusion

- The **Functional Method** is simple and elegant.
- The **Parameterised Method** is better for large input values because it can be optimized via **tail recursion**.

Both approaches ultimately return the same result: the factorial of a number.

---

## 🔗 Author

Created by Varun Mendre
