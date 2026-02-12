# Maximum Points You Can Obtain From Cards

---

# 1️⃣ Optimal Approach (Sliding Window on Ends)

## 1. Approach / Intuition

Problem Understanding:
- We can pick exactly `k` cards.
- Cards can be taken only from the beginning or the end of the array.

Key Insight:
Instead of thinking about which `k` cards to take,
think about how many cards to take from the left and how many from the right.

Possible combinations:
- Take all `k` from left
- Take `k-1` from left and 1 from right
- Take `k-2` from left and 2 from right
- ...
- Take all `k` from right

So total possibilities = `k + 1`

Algorithm:
1. First, take first `k` elements → this is initial sum.
2. Then gradually:
   - Remove one element from the left portion
   - Add one element from the right portion
3. Keep updating maximum sum.

This way we check all valid left-right combinations efficiently.

---

## 2. Code

```cpp
class Solution {
public:
    int maxScore(vector<int>& cardPoints, int k) {
        int n = cardPoints.size();
        int total = 0;

        // Take first k cards
        for (int i = 0; i < k; i++)
            total += cardPoints[i];

        int maxTotal = total;
        
        // Slide window: remove from left, add from right
        for (int i = 0; i < k; ++i) {
            total -= cardPoints[k - 1 - i];
            total += cardPoints[n - 1 - i];

            maxTotal = max(maxTotal, total);
        }

        return maxTotal;
    }
};
```

---

## 3. Time and Space Complexity

- **Time Complexity:** O(k)
  - Initial sum → O(k)
  - Sliding adjustments → O(k)

- **Space Complexity:** O(1)
  - No extra data structures used

---

# ✅ Why This Works

We are effectively evaluating all possible splits of `k` picks between:
- Left side
- Right side

Instead of generating combinations explicitly,
we maintain a rolling sum.

This is a classic example of:
👉 Fixed-size sliding window over two ends of array.

---

Structured for revision:
1. Approach / Intuition
2. Code
3. Time and Space Complexity

Clean, interview-ready explanation.

