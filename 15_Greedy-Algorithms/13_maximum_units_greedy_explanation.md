# 🚚 Maximum Units on a Truck (Greedy Approach)

---

## 🔎 Problem Understanding

Each element in `boxTypes` contains:

```
boxTypes[i] = { numberOfBoxes, unitsPerBox }
```

- `numberOfBoxes` → how many boxes of this type are available
- `unitsPerBox` → how many units each box contains
- `truckSize` → maximum number of boxes truck can carry

Goal:

👉 Maximize the total units loaded onto the truck.

---

# 💡 Core Idea (Greedy Strategy)

To maximize total units:

> Always pick boxes that give **more units per box first**.

This is similar to the **Fractional Knapsack intuition** — prioritize higher value items first.

---

# 🧠 Step-by-Step Code Explanation

## 1️⃣ Custom Comparator

```cpp
static bool comp(vector<int>& a, vector<int>& b) {
    return a[1] > b[1];
}
```

- `a[1]` → units per box of type A
- `b[1]` → units per box of type B
- Sorting in **descending order**

So highest unit boxes come first.

---

## 2️⃣ Sorting

```cpp
sort(boxTypes.begin(), boxTypes.end(), comp);
```

After sorting:

```
Highest units per box  →  First
Lowest units per box   →  Last
```

Now we process box types greedily from best to worst.

---

## 3️⃣ Iterating Through Box Types

```cpp
for (int i = 0; i < boxTypes.size(); i++) {
```

For each type:

```cpp
int numberOfBoxes = boxTypes[i][0];
int unitsPerBox = boxTypes[i][1];
```

---

## 4️⃣ Decide How Many Boxes to Take

```cpp
int boxesToTake = min(numberOfBoxes, truckSize);
```

Why?

- If truck has enough space → take all available boxes
- If truck has limited space → take only remaining capacity

---

## 5️⃣ Update Total Units

Contribution from current selection:

```
totalUnits += boxesToTake * unitsPerBox;
```

Then reduce truck capacity:

```cpp
truckSize -= boxesToTake;
```

---

## 6️⃣ Early Stopping

```cpp
if (truckSize == 0)
    break;
```

Once truck is full → stop immediately.

This avoids unnecessary iterations.

---

# 🔥 Why Greedy Works Here

This works because:

- Each box contributes independently
- There are no future dependencies
- Choosing higher unit boxes first never harms optimality

Local optimal choice → Global optimal solution.

---

# ⏱ Time & Space Complexity

### Time Complexity

- Sorting → O(n log n)
- Traversal → O(n)

Final:

```
O(n log n)
```

---

### Space Complexity

```
O(1)
```

(ignoring sorting space)

---

# 📌 Final Summary

1. Sort box types by units per box (descending).
2. Pick maximum possible boxes greedily.
3. Stop when truck becomes full.

👉 Classic Greedy Optimization Problem.

