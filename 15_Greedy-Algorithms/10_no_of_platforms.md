# Minimum Platforms Problem — Code Explanation (Brute Force + Optimal)

---

# 🔹 1️⃣ Brute Force Approach (O(n²))

```cpp
class Solution {
  public:
    int minPlatform(vector<int>& arr, vector<int>& dep) {
        int maxCount = 1;
        int n = arr.size();
        
        for(int i = 0; i < n; i++) {
            int count = 1;
            
            for(int j = i + 1; j < n; j++) {
                if(arr[i] >= arr[j] && arr[i] <= dep[j] 
                || arr[j] >= arr[i] && arr[j] <= dep[i]) {
                    count += 1;
                }
                
                maxCount = max(maxCount, count);
            }
        }
        
        return maxCount;
    }
};
```

---

## 1️⃣ Idea

For every train, count how many trains overlap with it.

The maximum overlap at any point = minimum platforms required.

---

## 2️⃣ Outer Loop

```cpp
for(int i = 0; i < n; i++)
```

Treat each train as the base train.

---

## 3️⃣ Inner Loop

```cpp
for(int j = i + 1; j < n; j++)
```

Compare current train with all other trains.

---

## 4️⃣ Overlap Condition

```cpp
arr[i] >= arr[j] && arr[i] <= dep[j]
|| arr[j] >= arr[i] && arr[j] <= dep[i]
```

This checks:
- If train i arrives during train j’s stay
OR
- Train j arrives during train i’s stay

If true → they need separate platforms.

---

## 5️⃣ Counting Platforms

- `count` tracks overlaps for current train
- `maxCount` stores global maximum overlap

---

## 6️⃣ Complexity

Time Complexity:

O(n²)

Space Complexity:

O(1)

---

# 🔹 2️⃣ Optimal Approach (Sorting + Two Pointers)

```cpp
class Solution {
  public:
    int minPlatform(vector<int>& arr, vector<int>& dep) {
        int n = arr.size();
        
        sort(dep.begin(), dep.end());
        sort(arr.begin(), arr.end());
        
        int i = 0, j = 0, maxPlatform = 0, count = 0;
        
        while(i < n) {
            if(arr[i] <= dep[j]) {
                count += 1;
                i += 1;
            }
            else {
                count -= 1;
                j += 1;
            }

            maxPlatform = max(maxPlatform, count);
        }
        
        return maxPlatform;
    }
};
```

---

## 1️⃣ Core Idea

Instead of checking pairwise overlap,
we treat arrivals and departures separately.

We simulate timeline processing.

---

## 2️⃣ Sorting

```cpp
sort(arr.begin(), arr.end());
sort(dep.begin(), dep.end());
```

- Process trains in order of arrival
- Track earliest departure

---

## 3️⃣ Two Pointers

- `i` → arrival pointer
- `j` → departure pointer
- `count` → current platforms needed
- `maxPlatform` → maximum platforms required

---

## 4️⃣ Logic Inside While Loop

```cpp
if(arr[i] <= dep[j])
```

If next train arrives before earliest departure:
- Need new platform
- `count++`
- Move arrival pointer

Else:
- One train departs
- Free one platform
- `count--`
- Move departure pointer

---

## 5️⃣ Why This Works

This simulates events in chronological order.

At any moment:

`count` = number of trains currently at station

Maximum value of `count` = minimum platforms required.

---

## 6️⃣ Complexity

Time Complexity:

O(n log n)

(Sorting dominates)

Space Complexity:

O(1)

---

# ✅ Final Comparison

Brute Force:
- Simple logic
- Checks all overlaps
- O(n²)

Optimal:
- Uses sorting + two pointers
- Processes timeline once
- O(n log n)

The optimal approach is the standard and interview-preferred solution.

