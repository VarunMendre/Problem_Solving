# Jump Game (Greedy)

---

## 1️⃣ Problem Understanding

Given an array `nums[]` where:
- `nums[i]` represents the maximum jump length from index `i`.

Goal:
Return `true` if you can reach the last index starting from index `0`.
Return `false` otherwise.

---

## 2️⃣ Key Insight (Greedy Thinking)

Instead of trying all possible jumps,
👉 Track the **maximum index reachable so far**.

If at any point:

```
current_index > maximum_reachable_index
```

Then we are stuck → return `false`.

Otherwise:
Keep updating the maximum reachable index.

This avoids recursion and dynamic programming.

---

## 3️⃣ Step-by-Step Logic

1. Initialize:
   - `maxInd = 0` (maximum reachable index)

2. Traverse the array:
   - If `i > maxInd` → cannot reach this index → return `false`
   - Otherwise:
     - Update:
       ```
       maxInd = max(maxInd, i + nums[i])
       ```

3. If loop completes → return `true`

---

## 4️⃣ Code

```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int n = nums.size();

        int maxInd = 0;
        for(int i = 0; i < n; i++) {
            if(i > maxInd) return false;
            maxInd = max(maxInd, nums[i] + i);
        }

        return true;
    }
};
```

---

## 5️⃣ Why This Greedy Works

- We always maintain the farthest reachable index.
- If we can reach index `i`, we check how far we can extend from it.
- We never need to explore every path.
- A single pass is enough.

This works because:
If a position is unreachable, no future position can fix it.

---

## 6️⃣ Time and Space Complexity

- **Time Complexity:**
  ```
  O(N)
  ```
  (Single pass)

- **Space Complexity:**
  ```
  O(1)
  ```
  (Only one variable used)

---

## ✅ Final Takeaway

- Pattern: Greedy
- Idea: Track maximum reachable index
- If you ever get stuck → return false

This is one of the most important greedy interview problems.

