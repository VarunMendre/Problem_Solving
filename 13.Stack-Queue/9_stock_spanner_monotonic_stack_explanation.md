# Stock Spanner Problem (Monotonic Stack)

This document explains the **Stock Spanner** solution exactly as implemented in the given code, broken down into:
1. Intuition / Approach  
2. Code  
3. Time & Space Complexity  

---

## 1. Intuition / Approach

The **Stock Spanner** problem asks us to compute, for each day's stock price, the number of consecutive days (including today) the price was **less than or equal to today’s price**.

### Key Idea

Instead of checking prices backward every time (which would be slow), we use a **monotonic decreasing stack**.

### What the stack stores

The stack stores pairs:
```
(price, index)
```
- `price` → stock price on that day  
- `index` → day number (0-based)

### Why a monotonic decreasing stack?

- Prices in the stack are kept **strictly decreasing** from bottom to top.
- If the current price is **greater than or equal to** the top price, that top price can never affect future spans → so it is popped.

### How span is calculated

- `ind` keeps track of the current day index.
- After popping smaller or equal prices:
  - If the stack is empty → no greater price exists on the left → span = `ind + 1`
  - Else → nearest greater element is at `st.top().second`

Formula used in code:
```
span = current_index - previous_greater_index
```

This ensures each element is pushed and popped **only once**, giving an efficient solution.

---

## 2. Code

```cpp
class StockSpanner {
public:
    stack<pair<int, int>> st;
    int ind = -1;
    StockSpanner() {
        int ind = -1;
        stack<pair<int, int>> st;

            while (!st.empty()) {
            st.pop();
        }
    }

    int next(int price) {
        ind += 1;

        while (!st.empty() && price >= st.top().first) {
            st.pop();
        }

        int ans = ind - (st.empty() ? -1 : st.top().second);
        st.push({price, ind});
        return ans;
    }
};

/**
 * Your StockSpanner object will be instantiated and called as such:
 * StockSpanner* obj = new StockSpanner();
 * int param_1 = obj->next(price);
 */
```

---

## 3. Time & Space Complexity

### Time Complexity: **O(2N) ≈ O(N)**

- Each stock price is:
  - **Pushed once** onto the stack
  - **Popped at most once** from the stack

So total stack operations:
```
N pushes + N pops = 2N operations
```

Even though a `while` loop is used inside `next()`, the total work across all calls is linear.

✅ **Amortized Time Complexity:** `O(N)`

---

### Space Complexity: **O(N)**

- In the worst case (strictly decreasing prices), all elements stay in the stack.
- Stack stores up to `N` elements.

✅ **Space Complexity:** `O(N)`

---

### Final Notes

- This is a classic **Monotonic Stack + Previous Greater Element** problem.
- Efficient and interview-friendly approach.
- Much better than brute-force `O(N²)` solutions.

---

Happy coding 🚀

