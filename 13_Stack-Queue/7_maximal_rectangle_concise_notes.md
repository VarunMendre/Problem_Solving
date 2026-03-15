# Maximal Rectangle

---

## 1. Intuition / Approach

- The problem asks for the **largest rectangle containing only `1`s** in a binary matrix.
- Treat **each row as the base of a histogram**:
  - For every column, compute the **height of consecutive `1`s ending at the current row**.
- This converts the 2D problem into multiple **Largest Rectangle in Histogram** problems.

Approach:
- Build a prefix-sum–like matrix (`pSum`) where:
  - `pSum[i][j]` = number of continuous `1`s vertically ending at cell `(i, j)`.
- For each row of `pSum`:
  - Treat it as a histogram.
  - Compute the largest rectangle using a **monotonic increasing stack**.
- The maximum over all rows is the final answer.

---

## 2. Code

```cpp
class Solution {
private:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        int maxArea = 0;

        stack<int> st;

        for (int i = 0; i < n; i++) {
            while (!st.empty() && heights[st.top()] > heights[i]) {
                int element = st.top();
                st.pop();

                int NGE = i, PSE = st.empty() ? -1 : st.top();

                maxArea = max(maxArea, heights[element] * (NGE - PSE - 1));
            }
            st.push(i);
        }

        while (!st.empty()) {
            int NGE = n;
            int element = st.top();
            st.pop();

            int PSE = st.empty() ? -1 : st.top();

            maxArea = max(maxArea, heights[element] * (NGE - PSE - 1));
        }

        return maxArea;
    }

public:
    int maximalRectangle(vector<vector<char>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        vector<vector<int>> pSum(n, vector<int>(m));

        for (int j = 0; j < m; j++) {
            int sum = 0;

            for (int i = 0; i < n; i++) {
                if (matrix[i][j] == '1') {
                    sum += 1;
                } else {
                    sum = 0;
                }

                pSum[i][j] = sum;
            }
        }

        int maxArea = 0;
        for (int i = 0; i < n; i++) {
            maxArea = max(maxArea, largestRectangleArea(pSum[i]));
        }

        return maxArea;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity
- Building prefix heights matrix: `O(N × M)`
- Largest Rectangle in Histogram for each row: `O(N × 2M)`

```
TC = O(N × M)
```

### Space Complexity
- Prefix matrix storage: `O(N × M)`
- Stack for histogram computation: `O(M)`

```
SC = O(N × M)
```

---
