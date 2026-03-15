# Largest Rectangle in a Histogram

---

## 1. Intuition / Approach

- Each bar in the histogram can act as the **minimum height** of a rectangle.
- For a bar at index `i`, the maximum rectangle using this bar as height extends:
  - Left until a **previous smaller element (PSE)**
  - Right until a **next smaller element (NSE)**

- Width of the rectangle:

```
width = NSE - PSE - 1
```

- Area contributed by that bar:

```
area = height[i] × width
```

- Use a **monotonic increasing stack (by height)** to efficiently find NSE and PSE in one pass.
- When a smaller height is encountered, it means the current bar is the **NSE** for stack top elements.
- After traversal, remaining stack elements use `n` as their NSE.

This avoids brute force and ensures optimal performance.

---

## 2. Code

```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        int maxArea = 0;

        stack<int> st;

        for (int i = 0; i < n; i++) {
            while (!st.empty() && heights[st.top()] > heights[i]) {
                int element = st.top();
                st.pop();

                int nse = i, pse = st.empty() ? -1 : st.top();

                maxArea = max(heights[element] * (nse - pse - 1), maxArea);
            }
            st.push(i);
        }

        while (!st.empty()) {
            int nse = n;
            int element = st.top();
            st.pop();

            int pse = st.empty() ? -1 : st.top();

            maxArea = max(maxArea, heights[element] * (nse - pse - 1));
        }

        return maxArea;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity
- Each index is pushed and popped **at most once** from the stack.

```
TC = O(2N) ≈ O(N)
```

### Space Complexity
- Stack stores indices of bars.

```
SC = O(N)
```

---

