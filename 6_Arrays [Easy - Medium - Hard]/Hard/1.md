# 1_Pascals_Triangle

## 1. Problem Statement
Given a non-negative integer `numRows`, generate the first `numRows` of Pascal's triangle. Each row of Pascal's triangle contains binomial coefficients. For example, for `numRows = 5`, the triangle is:
```
[
  [1],
  [1,1],
  [1,2,1],
  [1,3,3,1],
  [1,4,6,4,1]
]
```

## 2. Intuition
Pascal's triangle is built from binomial coefficients. The value at row `r` and column `c` (0-indexed) is `C(r, c)` — the number of ways to choose `c` items from `r`. Each element (except the edges which are `1`) can be computed from the previous element of the same row using the relation:
```
C(r, c) = C(r, c-1) * (r - c + 1) / c
```
This recurrence lets us compute a row left-to-right using a single running variable that stores the previous binomial coefficient. That avoids computing factorials or using heavy arithmetic repeatedly.

## 3. Approach / Thought Process
Two common methods to generate Pascal's triangle:

**A. Build each row from the previous row**  
- Start from row `[1]`. For each next row, prepend and append `1`, and for each internal position take the sum of two elements above from the previous row.
- This method uses values from the previous row directly and is intuitive.

**B. Compute each row using binomial coefficients incrementally (used in the provided code)**  
- For row `r` (1-indexed in code), start with `ans = 1` (which is `C(r-1, 0)`).
- For each next column `c` from `1` to `r-1`, compute `ans = ans * (r - c) / c`. Push `ans` to the row.
- This computes exact binomial coefficients without repeated addition and is numerically efficient for moderate `r` when using a type large enough for intermediate results (e.g., `long long`).

The provided solution implements approach B and constructs rows using a combinatorial incremental formula.

## 4. Algorithm (Pseudocode)
```
function generate(numRows):
    if numRows <= 0: return empty list
    result = empty list of lists
    for rowIndex in 1..numRows:               // 1-indexed for clarity
        append generateRow(rowIndex) to result
    return result

function generateRow(row):                    // row is 1-indexed
    ans = 1
    rowVec = [1]
    for col from 1 to row-1:
        ans = ans * (row - col)
        ans = ans / col
        append ans to rowVec
    return rowVec
```

## 5. Dry Run
Let's dry run for `numRows = 4`:

- `i = 1` -> generateRow(1):
  - ans = 1, rowVec = [1] -> no loop (col < row), return `[1]`

- `i = 2` -> generateRow(2):
  - ans = 1, rowVec = [1]
  - col = 1: ans = 1 * (2 - 1) / 1 = 1 -> rowVec = [1, 1]
  - return `[1, 1]`

- `i = 3` -> generateRow(3):
  - ans = 1, rowVec = [1]
  - col = 1: ans = 1 * (3 - 1) / 1 = 2 -> rowVec = [1, 2]
  - col = 2: ans = 2 * (3 - 2) / 2 = 1 -> rowVec = [1, 2, 1]
  - return `[1, 2, 1]`

- `i = 4` -> generateRow(4):
  - ans = 1, rowVec = [1]
  - col = 1: ans = 1 * (4 - 1) / 1 = 3 -> [1, 3]
  - col = 2: ans = 3 * (4 - 2) / 2 = 3 -> [1, 3, 3]
  - col = 3: ans = 3 * (4 - 3) / 3 = 1 -> [1, 3, 3, 1]
  - return `[1, 3, 3, 1]`

Final result: `[[1], [1,1], [1,2,1], [1,3,3,1]]`

## 6. Code (C++)
```cpp
#include <vector>
using namespace std;

class Solution {
private:
    vector<int> generateRows(int row) {
        long long ans = 1;                 // initialize to 1
        vector<int> ansRow;
        ansRow.push_back(1);

        for (int col = 1; col < row; col++) {
            ans = ans * (row - col);
            ans = ans / col;
            ansRow.push_back(static_cast<int>(ans));
        }
        return ansRow;
    }

public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int>> ans;
        if (numRows <= 0) return ans;

        for (int i = 1; i <= numRows; i++) {
            ans.push_back(generateRows(i));
        }
        return ans;
    }
};
```

### Small notes about the code
- `ans` must be initialized to `1`. Using an uninitialized variable leads to undefined behavior.
- We use `long long` to reduce risk of overflow when computing intermediate binomial coefficients; final values are cast to `int` when pushed into the vector (LeetCode expects `int`). For very large rows, values may still overflow `long long`.
- Alternative approach builds next row directly from previous row using addition, avoiding multiplicative intermediate values; both methods are valid.

## 7. Time & Space Complexity
- **Time complexity:** `O(numRows^2)` — you must compute every entry in the triangle (sum of 1..numRows = numRows*(numRows+1)/2 entries), and each entry is produced in constant time.
- **Space complexity:** `O(numRows^2)` for the output storage holding all rows. If only one row is required at a time (streaming), additional space can be reduced to `O(numRows)` for the largest row.

---
**Edge cases and recommendations**
- If `numRows == 0`, return an empty vector (handled in code).
- If you expect `numRows` > ~30 on platforms with 32-bit ints, intermediate values can overflow `long long`. Consider using big-integer libraries or generating rows using previous-row addition and storing results in a larger integer type if accurate large coefficients are required.
- For memory-constrained settings where you only need the last row, generate rows iteratively and discard previous rows, keeping `O(row_length)` memory.
