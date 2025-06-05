# 7. Move 0's to End

## 🧠 Problem Statement

Given an array of integers, move all the 0's to the end of the array while maintaining the relative order of the non-zero elements.

---

## 🔹 Brute Force Approach

### ✅ Approach:

1. **Create a temporary vector** to store all non-zero elements.
2. **Iterate through the array**, and push each non-zero element to the temporary vector.
3. After collecting all non-zero elements:
   - **Copy them back** to the original array starting from index 0.
4. **Fill the remaining indices** of the array with zeros.

This method preserves the order of non-zero elements.

### 🧾 Code:
```cpp
void pushZerosAtEnd(vector<int> &arr) 
{
    int n = arr.size();
    vector<int> temp;

    for(int i = 0; i < n; i++) {
        if(arr[i] != 0) {
            temp.push_back(arr[i]);
        }
    }

    int nz = temp.size();

    for(int i = 0; i < nz; i++) {
        arr[i] = temp[i];
    }

    for(int i = nz; i < n; i++) {
        arr[i] = 0;
    }

    return;
}
```

### ⏱️ Time Complexity:
- O(N) for traversing the array and storing non-zero elements.
- O(X) to copy non-zero elements back.
- O(N - X) to fill remaining positions with zeros.
- **Total: O(2N) → O(N)**

### 💾 Space Complexity:
- **O(X)**: where X = number of non-zero elements (not the full array).

---

## 🔹 Optimal Approach (Two Pointers)

### ✅ Approach:

1. Use two pointers:
   - `i` traverses through the array.
   - `j` points to the **first 0** found in the array.
2. Once `j` is found, start from `j + 1`:
   - If a non-zero is found at `i`, **swap** it with the element at `j`, and increment `j`.

This way, every time a non-zero is found, it's **moved left** to the position of the first zero.

### 🧪 Dry Run:

For array `a = [1, 0, 2, 3, 0, 4, 0]`

```
Initial: a = [1, 0, 2, 3, 0, 4, 0]
j = 1 (first 0)

i = 2 → a[2] = 2 ≠ 0 → swap a[2] and a[1] → a = [1, 2, 0, 3, 0, 4, 0], j++
i = 3 → a[3] = 3 ≠ 0 → swap a[3] and a[2] → a = [1, 2, 3, 0, 0, 4, 0], j++
i = 4 → a[4] = 0 → do nothing
i = 5 → a[5] = 4 ≠ 0 → swap a[5] and a[3] → a = [1, 2, 3, 4, 0, 0, 0], j++
i = 6 → a[6] = 0 → do nothing

Final Output: [1, 2, 3, 4, 0, 0, 0]
```

### 🧾 Code:
```cpp
vector<int> moveZeros(int n, vector<int> a) {
    int j = -1;

    // Step 1: Find the first 0
    for(int i = 0; i < n; i++) {
        if(a[i] == 0) {
            j = i;
            break;
        }
    }

    // If no 0 found, array is already fine
    if(j == -1) return a;

    // Step 2: Move non-zero elements to the front
    for(int i = j + 1; i < n; i++) {
        if(a[i] != 0) {
            swap(a[i], a[j]);
            j++;
        }
    }

    return a;
}
```

### ⏱️ Time Complexity:
- O(N): One pass to find the first 0.
- O(N): Second pass to process and swap elements.
- **Total: O(N)**

### 💾 Space Complexity:
- **O(1)**: In-place swapping, no extra space used.
