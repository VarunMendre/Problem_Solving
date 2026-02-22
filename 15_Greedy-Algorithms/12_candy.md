# Candy Problem — Naive, Better, Optimal (Point Wise Explanation)

---

# 🔹 1️⃣ Naive Approach (Left + Right Arrays)

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();

        vector<int> left(n, 0);
        vector<int> right(n, 0);

        left[0] = 1;
        right[n - 1] = 1;

        for(int i = 1; i < n; i++) {
            if(ratings[i] > ratings[i - 1]) {
                left[i] = left[i - 1] + 1;
            }else {
                left[i] = 1;
            }
        }  

        for(int i = n - 2; i >= 0; i--) {
            if(ratings[i] > ratings[i + 1]) {
                right[i] = right[i + 1] + 1;
            }else{ 
                right[i] = 1;
            }
        }    

        int sum = 0;
        for(int i = 0; i < n; i++) {
            sum += max(left[i], right[i]);
        }

        return sum;
    }
};
```

---

## 1️⃣ Idea

Each child must:
- Get at least 1 candy
- If rating is higher than neighbor → get more candies

We check this in two directions:

- Left → Right
- Right → Left

---

## 2️⃣ Left Pass

If current rating > previous rating:

```
left[i] = left[i-1] + 1
```

Else:

```
left[i] = 1
```

Ensures left neighbor condition is satisfied.

---

## 3️⃣ Right Pass

If rating[i] > rating[i+1]:

```
right[i] = right[i+1] + 1
```

Else:

```
right[i] = 1
```

Ensures right neighbor condition.

---

## 4️⃣ Final Calculation

For each child:

```
max(left[i], right[i])
```

Because child must satisfy both conditions.

---

## 5️⃣ Complexity

Time Complexity:
O(3n)

Space Complexity:
O(2n)

---

# 🔹 2️⃣ Better Approach (Single Left Array + Right Traversal)

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();

        vector<int> left(n, 1);

        for(int i = 1; i < n; i++) {
            if(ratings[i] > ratings[i - 1]) {
                left[i] = left[i - 1] + 1;
            }
        }  

        int curr = 1 , right = 1, sum = max(1, left[n - 1]);

        for(int i = n - 2; i >= 0; i--) {
            if(ratings[i] > ratings[i + 1]) {
                curr = right + 1;
                right = curr;
            }else{ 
                curr = 1;
                right = 1;
            }

            sum += max(left[i], curr);
        }    

        return sum;
    }
};
```

---

## 1️⃣ Idea

Instead of storing full right array,
we compute right side on the fly.

---

## 2️⃣ Left Pass

Same as naive.
Stores increasing sequence info.

---

## 3️⃣ Right Traversal

We track:

- `right` → candies required for decreasing slope
- `curr` → current right-side candy count

If rating decreases:

```
curr = right + 1
```

Else reset to 1.

---

## 4️⃣ Add to Sum

At each step:

```
sum += max(left[i], curr)
```

---

## 5️⃣ Complexity

Time Complexity:
O(2n)

Space Complexity:
O(n)

---

# 🔹 3️⃣ Optimal Approach (Slope / Peak Method)

```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();

        int sum = 1;
        int i = 1;

        while(i < n) {
            if(ratings[i] == ratings[i - 1]) {
                sum += 1;
                i++;
                continue;
            }

            int peak = 1;

            while(i < n && ratings[i] > ratings[i - 1]) {
                peak += 1;
                sum += peak;
                i++;
            }

            int down = 1;

            while(i < n && ratings[i] < ratings[i - 1]) {
                sum += down;
                down += 1;
                i++;
            }

            if(down > peak) {
                sum += (down - peak);
            }
        }

        return sum;
    }
};
```

---

## 1️⃣ Core Idea

Treat ratings as slopes:

- Increasing slope
- Decreasing slope

We count candies using slope lengths.

---

## 2️⃣ Increasing Sequence

For strictly increasing ratings:

Candies form:
1, 2, 3, ..., peak

We accumulate directly.

---

## 3️⃣ Decreasing Sequence

For strictly decreasing ratings:

Candies form:
1, 2, 3, ..., down

But peak element must satisfy both sides.

---

## 4️⃣ Peak Adjustment

If decreasing length > increasing length:

```
sum += (down - peak)
```

Ensures peak gets enough candies.

---

## 5️⃣ Why This Works

Instead of storing arrays,
we calculate lengths of slopes and use math logic.

No extra arrays needed.

---

## 6️⃣ Complexity

Time Complexity:
O(n)

Space Complexity:
O(1)

---

# ✅ Final Comparison

Naive → Two arrays (easy to understand)
Better → One array
Optimal → Pure slope counting, constant space

The optimal slope approach is the most efficient and interview preferred.

