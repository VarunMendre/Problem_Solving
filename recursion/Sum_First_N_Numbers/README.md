
# 📘 Problem Statement

**Title**: Sum of First N Natural Numbers using Recursion

**Objective**:  
Given a positive integer `N`, compute the sum of the first `N` natural numbers using recursion.

**Example**:
```
Input: N = 5
Output: 15
Explanation: 1 + 2 + 3 + 4 + 5 = 15
```

---

## 🧠 Approach to the Problem

We aim to use **recursion** to solve this problem. There are two common recursive approaches:

### 1. Parameterised Way
In this method, we pass the result (sum) as a parameter to the recursive function. The function builds the sum incrementally as it progresses.

### 2. Functional Way
In this approach, we return the result from the function rather than passing it as a parameter. The recursive call builds up the result using the return values.

---

## ✅ Solution 1: Parameterised Way

### 🔁 Logic
- Start with `i = N` and an initial `sum = 0`.
- Add `i` to `sum` and call the function with `i-1` and updated `sum`.
- When `i < 1`, print the accumulated sum.

### 📄 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

void F(int i, int sum) {
    if (i < 1) {
        cout << sum;
        return;
    }
    F(i - 1, sum + i);
}

int main() {
    int n;
    cin >> n;
    F(n, 0);
}
```

### 🧪 Example

Input:
```
5
```

Output:
```
15
```

---

## ✅ Solution 2: Functional Way

### 🔁 Logic
- Call the function recursively for `n-1`.
- Add `n` to the result returned by the recursive call.
- Base case: if `n == 0`, return 0.

### 📄 Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);
}

int main() {
    int n;
    cin >> n;
    cout << sum(n);
}
```

### 🧪 Example

Input:
```
5
```

Output:
```
15
```

---

## 📝 Conclusion

Both approaches use recursion but differ in how they handle the result:

| Approach        | Style          | Result Handling | Suitable for       |
|----------------|----------------|-----------------|--------------------|
| Parameterised   | Top-Down       | Uses parameters | When extra info is passed or tracked |
| Functional      | Bottom-Up      | Uses return value | When building result from subproblems |

Choose the approach that best suits the situation and your preference!
