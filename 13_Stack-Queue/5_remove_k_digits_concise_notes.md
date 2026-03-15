# Remove K Digits

---

## 1. Intuition / Approach

- Goal: Remove exactly `k` digits from the number so that the **resulting number is the smallest possible**.
- To minimize the number, we want **smaller digits to appear as early as possible**.
- Use a **monotonic increasing stack** (digits in increasing order from bottom to top).

Approach:
- Traverse digits from left to right.
- For each digit:
  - While the stack is not empty, `k > 0`, and the top of the stack is **greater than the current digit**, pop the stack (removing a larger digit earlier reduces the number).
  - Push the current digit.
- If `k` is still greater than 0 after traversal, remove digits from the end (largest remaining digits).
- Build the result from the stack and remove **leading zeros**.
- If the result becomes empty, return `"0"`.

---

## 2. Code

```cpp
class Solution {
public:
    string removeKdigits(string num, int k) {
        int n = num.size();

        stack<char> st;
        // Traverse on the given string
        for (int i = 0; i < n; i++) {

            char digit = num[i];

            // pop last digit is smaller found
            while (!st.empty() && k > 0 && st.top() > digit) {
                st.pop();
                k -= 1;
            }
            st.push(digit);
        }

        // if k is still 0
        while (k > 0) {
            st.pop();
            k--;
        }

        // if stack is empty return
        if (st.empty())
            return "0";

        string res = "";

        // popout all stack elements and store in res
        while (!st.empty()) {
            res += st.top();
            st.pop();
        }

        // remove zero's from last
        while (res.size() != 0 && res.back() == '0') {
            res.pop_back();
        }

        reverse(res.begin(), res.end());

        if (res.size() == 0)
            return "0";

        return res;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity
- Each digit is pushed and popped **at most once**.

```
TC = O(2N) ≈ O(N)
```

### Space Complexity
- Stack can hold up to `N` digits.

```
SC =