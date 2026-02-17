# Lemonade Change (Greedy Explanation)

---

## 1️⃣ Problem Summary

- Each lemonade costs **$5**.
- Customers pay using: **$5, $10, or $20**.
- We must return correct change immediately.
- Return `true` if all customers can be served.
- Return `false` if at any point we cannot give proper change.

---

## 2️⃣ Key Observations

1. We only need to track:
   - Number of `$5` bills
   - Number of `$10` bills
2. `$20` bills are never used to give change.
3. Change requirements:
   - If customer pays `$5` → no change needed.
   - If customer pays `$10` → give `$5`.
   - If customer pays `$20` → give `$15`.

---

## 3️⃣ Greedy Strategy

👉 Always try to give change using larger denominations first.

For `$20` payment (needs `$15` change):

1. Prefer giving:
   - One `$10` + one `$5`
2. Otherwise:
   - Give three `$5` bills

### Why?
Because `$5` bills are more important for future transactions.
Preserving smaller bills increases flexibility.

This is a classic greedy decision:
- Make the best local choice at every step.
- That guarantees a globally correct result.

---

## 4️⃣ Step-by-Step Logic

For each bill:

### Case 1: bill == 5
- No change required.
- Increment count of `$5` bills.

### Case 2: bill == 10
- Need to give `$5` as change.
- If we have at least one `$5`:
  - Decrement `$5`
  - Increment `$10`
- Otherwise:
  - Return `false`

### Case 3: bill == 20
- Need to give `$15` as change.

Try in this order:
1. If we have `$10` and `$5`:
   - Use one of each
2. Else if we have at least three `$5`:
   - Use three `$5`
3. Else:
   - Return `false`

If we successfully process all customers → return `true`.

---

## 5️⃣ Code

```cpp
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        int five = 0; 
        int ten = 0;  

        for (int bill : bills) {
            if (bill == 5) {
                five++;
            } 
            else if (bill == 10) {
                if (five > 0) {
                    five--;
                    ten++;
                } else {
                    return false;
                }
            } 
            else { // bill == 20
                if (five > 0 && ten > 0) {
                    five--;
                    ten--;
                } 
                else if (five >= 3) {
                    five -= 3;
                } 
                else {
                    return false;
                }
            }
        }
        return true;
    }
};
```

---

## 6️⃣ Time and Space Complexity

- **Time Complexity:** `O(N)`
  - Single pass through the array

- **Space Complexity:** `O(1)`
  - Only two counters are used

---

## ✅ Final Takeaway

- This is a **Greedy + Simulation** problem.
- The trick is prioritizing `$10 + $5` over `three $5`.
- Preserving smaller denominations is the key insight.

