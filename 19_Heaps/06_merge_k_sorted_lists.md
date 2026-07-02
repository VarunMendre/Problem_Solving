# Merge K Sorted Linked Lists

---

## 1. Brute Force — Collect All Values + Sort

### Approach

1. Traverse all K lists and collect every node's value into a single vector
2. Sort the vector
3. Build a new linked list from the sorted values

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        vector<int> allVals;

        // Step 1: collect all values from all K lists
        for(auto list : lists) {
            while(list != NULL) {
                allVals.push_back(list->val);
                list = list->next;
            }
        }

        // Step 2: sort
        sort(allVals.begin(), allVals.end());

        // Step 3: build new sorted linked list
        ListNode* dummy = new ListNode(0);
        ListNode* cur = dummy;

        for(auto val : allVals) {
            cur->next = new ListNode(val);
            cur = cur->next;
        }

        return dummy->next;
    }
};
```

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log N)` | Collect all N values `O(N)` + sort `O(N log N)` + rebuild `O(N)` → dominated by sort |
| **Space** | `O(N)` | `allVals` vector stores all N node values |

> Simple but throws away the fact that **each individual list is already sorted** — the optimal approach uses this.

---

---

## 2. Optimal — Min Heap (Priority Queue)

### Approach

Since each of the K lists is **already sorted**, the next smallest element globally must be the **minimum of the K current front nodes**. A **min heap** gives us this minimum in `O(log K)`.

**Algorithm:**
1. Push the **head of each list** into the min heap
2. Repeatedly:
   - Pop the minimum node
   - Append it to the result list
   - Push that node's `next` into the heap (if it exists)
3. Heap always has **at most K elements** (one per list)

```cpp
class Compare {
public:
    bool operator()(ListNode* a, ListNode* b) {
        return a->val > b->val;     // min heap — smaller value has higher priority
    }
};

class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*, vector<ListNode*>, Compare> pq;

        // Step 1: push head of each non-null list
        for(auto list : lists) {
            if(list != NULL)
                pq.push(list);
        }

        ListNode* dummy = new ListNode(0);
        ListNode* curr = dummy;

        while(!pq.empty()) {
            auto p = pq.top();
            pq.pop();

            // Append minimum node to result
            curr->next = new ListNode(p->val);
            curr = curr->next;

            // Push next node from the same list
            if(p->next != NULL)
                pq.push(p->next);
        }

        return dummy->next;
    }
};
```

---

### Dry Run

```
lists = [1→4→5, 1→3→4, 2→6]
K = 3
```

```
Initial heap: [(1,L1), (1,L2), (2,L3)]  (min on top)

Pop (1,L1) → result: 1  push 4 → heap: [(1,L2),(2,L3),(4,L1)]
Pop (1,L2) → result: 1→1  push 3 → heap: [(2,L3),(4,L1),(3,L2)]
Pop (2,L3) → result: 1→1→2  push 6 → heap: [(3,L2),(4,L1),(6,L3)]
Pop (3,L2) → result: 1→1→2→3  push 4 → heap: [(4,L1),(4,L2),(6,L3)]
Pop (4,L1) → result: 1→1→2→3→4  push 5 → heap: [(4,L2),(5,L1),(6,L3)]
Pop (4,L2) → result: 1→1→2→3→4→4  push NULL → heap: [(5,L1),(6,L3)]
Pop (5,L1) → result: 1→1→2→3→4→4→5  push NULL → heap: [(6,L3)]
Pop (6,L3) → result: 1→1→2→3→4→4→5→6  push NULL → heap: []

Final: 1→1→2→3→4→4→5→6 ✅
```

---

### Time & Space Complexity

| | Complexity | Reason |
|---|---|---|
| **Time** | `O(N log K)` | N total nodes, each pushed and popped once from a heap of size at most K → `O(log K)` per operation |
| **Space** | `O(K)` | Heap holds at most K nodes at any time (one per list) |

---

---

## 3. Brute vs Optimal

| | Brute | Optimal |
|---|---|---|
| **Time** | `O(N log N)` | `O(N log K)` |
| **Space** | `O(N)` | `O(K)` |
| **Uses sorted property?** | ❌ No | ✅ Yes |
| **Key idea** | Collect everything, sort | Always pick global minimum from K candidates using a heap |

> When `K << N` (few lists, many nodes), `O(N log K)` is significantly better than `O(N log N)`. E.g., merging 4 lists of 1000 nodes: `log K = 2` vs `log N ≈ 12`.
