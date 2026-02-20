# Maximum Meetings in One Room (Activity Selection - Greedy)

---

## 1️⃣ Problem Understanding

We are given:
- `start[i]` → Start time of meeting i
- `end[i]` → End time of meeting i

Rules:
- Only one meeting can be held in the room at a time.
- A meeting can be selected only if its start time is strictly greater than the ending time of the last selected meeting.

Goal:
Maximize the number of meetings that can be performed.

---

## 2️⃣ Key Insight (Greedy Strategy)

👉 Always select the meeting that finishes earliest.

Why?
- A meeting that ends earlier leaves more room for future meetings.
- If we select a meeting that ends late, it may block many smaller meetings.

This is the classic **Activity Selection Problem**.

Greedy Rule:
1. Sort meetings by ending time (ascending).
2. Always pick the first meeting.
3. Then select the next meeting whose start time is greater than the last selected meeting's ending time.

---

## 3️⃣ Step-by-Step Logic

### Step 1: Store Meeting Data

We create a struct to store:
- start time
- ending time
- position (index)

```cpp
struct Data {
    int start;
    int ending;
    int pos;
};
```

---

### Step 2: Sort by Ending Time

```cpp
static bool comp(Data p1, Data p2) {
    return p1.ending < p2.ending;
}
```

Sorting ensures earliest finishing meetings are considered first.

---

### Step 3: Select Meetings

1. Select the first meeting (it finishes earliest).
2. Track its ending time as `freeTime`.
3. For every next meeting:
   - If `meeting.start > freeTime`
     - Select it
     - Update `freeTime`

---

## 4️⃣ Code

```cpp
class Solution {
    
    struct Data {
        int start;
        int ending;
        int pos;
    };
    
    static bool comp(Data p1, Data p2) {
        return p1.ending < p2.ending;
    }
    
public:
    int maxMeetings(vector<int>& start, vector<int>& end) {
        
        int n = start.size();
        vector<Data> meetings;
        
        for(int i = 0; i < n; i++) {
            meetings.push_back({start[i], end[i], i+1});
        }
        
        sort(meetings.begin(), meetings.end(), comp);
        
        int cnt = 1;
        int freeTime = meetings[0].ending;
        
        for(int i = 1; i < n; i++) {
            if(meetings[i].start > freeTime) {
                cnt++;
                freeTime = meetings[i].ending;
            }
        }
        
        return cnt;
    }
};
```

---

## 5️⃣ Why This Greedy Works

- Choosing the earliest finishing meeting maximizes remaining time.
- This allows more meetings to fit.
- Every step makes a locally optimal decision.
- That leads to a globally optimal solution.

This is one of the most important greedy scheduling problems.

---

## 6️⃣ Time and Space Complexity

### Time Complexity:

- Storing meetings → O(N)
- Sorting → O(N log N)
- Traversal → O(N)

Overall:

```
O(N log N + 2N)
```

---

### Space Complexity:

```
O(3N)
```

- Storing start, end, and position inside vector

---

## ✅ Final Takeaway

Pattern: Greedy + Sorting

Key Rule:
- Sort by earliest finishing time
- Select compatible meetings greedily

This problem builds the foundation for interval scheduling problems.

