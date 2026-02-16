# Assign Cookies (Greedy Algorithm)

---

# 1️⃣ Problem Understanding

We are given:

- `g[i]` → greed factor of each child
- `s[j]` → size of each cookie

A child can be satisfied only if:

```
cookie_size >= greed_factor
```

Goal:
Maximize the number of content (satisfied) children.

---

# 2️⃣ Approach / Intuition (Greedy Strategy)

Core Greedy Idea:

👉 Always try to satisfy the least greedy child first using the smallest available cookie that works.

Why?
- If we give a large cookie to a less greedy child unnecessarily,
- We might fail to satisfy a more greedy child later.

So we:
1. Sort the greed array.
2. Sort the cookie sizes.
3. Use two pointers:
   - `greedInd` → tracks children
   - `cookieInd` → tracks cookies

Logic:
- If current cookie can satisfy current child:
  - Move to next child.
- Always move to next cookie.

This ensures optimal matching.

---

# 3️⃣ Code

```cpp
class Solution {
public:
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());

        int greedInd = 0, cookieInd = 0;
        int n = g.size(), m = s.size();

        while(greedInd < n && cookieInd < m) {
            if(s[cookieInd] >= g[greedInd]) {
                greedInd++;
            }
            cookieInd++;
        }
        return greedInd;
    }
};
```

---

# 4️⃣ Why This Works (Greedy Proof Intuition)

- Sorting ensures smallest greed is matched first.
- Smallest sufficient cookie is used.
- No cookie is wasted on a child who doesn't need that large size.
- Each step makes a locally optimal decision.
- This local optimal choice leads to global optimal result.

---

# 5️⃣ Time and Space Complexity

- **Time Complexity:**
  - Sorting greed array → O(n log n)
  - Sorting cookies → O(m log m)
  - Two-pointer traversal → O(m)

  Overall:
  ```
  O(n log n + m log m + m)
  ```

- **Space Complexity:**
  ```
  O(1)
  ```
  (Ignoring sorting space if done in-place)

---

# ✅ Final Summary

| Concept | Type |
|----------|--------|
| Technique | Greedy + Sorting |
| Pattern | Two Pointers |
| Key Idea | Match smallest valid pair first |

---

This is a classic greedy problem and a very common interview favorite.
Understanding why sorting works here is more important than memorizing the code.

