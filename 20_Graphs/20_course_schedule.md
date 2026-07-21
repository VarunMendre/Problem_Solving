# Course Schedule I & II

---

## The Connection to Topological Sort

Both problems are **direct applications of Kahn's Algorithm** (BFS topological sort). Before diving in, internalize this mapping:

| Problem concept | Graph concept |
|---|---|
| Course | Node / vertex |
| Prerequisite `[a, b]` | Directed edge |
| "Can finish all courses?" | Does a valid topological order exist? (Is it a DAG?) |
| "Return order to finish courses" | Return the topological order itself |

A valid topological order EXISTS if and only if the graph has **no cycle**. With Kahn's: `result.size() == V` → no cycle → valid order exists.

---

---

## Problem 1 — Course Schedule (Can You Finish?)

### Problem Statement

There are `V` courses labeled `0` to `V-1`. You're given a list of `prerequisites` where `prerequisites[i] = [a, b]` means **course `b` must be taken before course `a`**.

Return `true` if you can finish all courses, `false` otherwise.

```
V = 2, prerequisites = [[1,0]]
"Course 0 must be taken before Course 1"
Possible order: 0 → 1 → true ✅

V = 2, prerequisites = [[1,0], [0,1]]
"0 before 1" AND "1 before 0" → cycle → false ❌
```

---

### Intuition

This is a cycle detection problem in disguise.

- If all courses can be completed → there's a valid order to take them → no cycles in the dependency graph → valid DAG
- If a cycle exists (A requires B requires A) → impossible → `false`

Kahn's algorithm gives us cycle detection for free: if `result.size() == V` after BFS, every course was processed → no cycle → return `true`.

---

### The Edge Direction

```cpp
adjLs[it[0]].push_back(it[1]);
indegree[it[1]]++;
```

`prerequisites[i] = [a, b]` means `b` must come before `a` → edge is `a → b` in our adjacency list.

Wait — why `adjLs[it[0]] → it[1]` and `indegree[it[1]]++`?

`it[0] = a` (the course that needs a prerequisite), `it[1] = b` (the prerequisite).  
Edge direction: `a → b` means "a points to b (a depends on b)".

In Kahn's, `indegree[b]++` because `b` now has something pointing INTO it. Nodes with no incoming edges (no prerequisites) start the BFS.

> Actually this edge goes `a → b` meaning "a has a dependency on b". Nodes with indegree 0 are those with no prerequisites. This is consistent with `indegree[it[1]]++` — `it[1] = b` gets an incoming edge from `a`.

---

### Dry Run

```
V = 4
prerequisites = [[1,0], [2,0], [3,1], [3,2]]
"Take 0 before 1, 0 before 2, 1 before 3, 2 before 3"

Build graph (edge a→b means a depends on b):
adjLs: 1→[0], 2→[0], 3→[1], 3→[2]
indegree: [0:2, 1:1, 2:1, 3:0]
         (0 has 2 incoming, 3 has 0 incoming)
```

Wait — let me reread: `adjLs[it[0]].push_back(it[1])` → `adjLs[1].push_back(0)`, `adjLs[2].push_back(0)`, etc.

```
indegree[it[1]]++ → indegree[0]++, indegree[0]++, indegree[1]++, indegree[2]++

adjLs: 1→[0], 2→[0], 3→[1,2]
indegree: [2, 1, 1, 0]

Seed queue: indegree[3]==0 → push 3
queue = [3], result = []

Pop 3: result=[3]
  neighbor 1: indegree[1]--→0 → push 1
  neighbor 2: indegree[2]--→0 → push 2
queue = [1, 2]

Pop 1: result=[3,1]
  neighbor 0: indegree[0]--→1 (not 0 yet)
queue = [2]

Pop 2: result=[3,1,2]
  neighbor 0: indegree[0]--→0 → push 0
queue = [0]

Pop 0: result=[3,1,2,0]
  no neighbors
queue = []

result.size()=4 == V=4 → return true ✅
```

---

### Code

```cpp
class Solution {
public:
    bool canFinish(int V, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adjLs(V);
        vector<int> indegree(V, 0);

        // prerequisites[i] = [a, b]: a depends on b → edge a→b
        for(auto it : prerequisites) {
            adjLs[it[0]].push_back(it[1]);
            indegree[it[1]]++;
        }

        // Seed: courses with no prerequisites
        queue<int> q;
        for(int i = 0; i < V; i++) {
            if(indegree[i] == 0)
                q.push(i);
        }

        vector<int> result;
        while(!q.empty()) {
            int node = q.front();
            q.pop();
            result.push_back(node);

            for(auto& it : adjLs[node]) {
                indegree[it]--;
                if(indegree[it] == 0)
                    q.push(it);
            }
        }

        // All courses processed → no cycle → can finish
        return result.size() == V;
    }
};
```

---

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(V + E)` — V nodes + E edges in BFS |
| **Space** | `O(V + E)` — adjacency list + indegree + queue |

---

---

## Problem 2 — Course Schedule II (Return the Order)

### Problem Statement

Same setup as Problem 1, but instead of returning `true/false`, **return the order** in which you should take the courses. Return an empty array if impossible (cycle exists).

```
V = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: [0,2,1,3] or [0,1,2,3] (any valid topological order)

V = 2, prerequisites = [[1,0],[0,1]]  ← cycle
Output: []
```

---

### The Critical Difference From Problem 1 — Edge Direction Flipped

This is the most important thing to notice.

**Problem 1:**
```cpp
adjLs[it[0]].push_back(it[1]);
indegree[it[1]]++;
```

**Problem 2:**
```cpp
adjLs[it[1]].push_back(it[0]);   // ← FLIPPED
indegree[it[0]]++;                // ← FLIPPED
```

**Why?**

`prerequisites[i] = [a, b]` means "take `b` before `a`".

- In Problem 1: we modeled `a → b` (a depends on b) → nodes with no dependencies (b-type nodes) have indegree = 0 → come first
- In Problem 2: we model `b → a` (b must be taken → then a becomes available) → this is the **natural course order** → b's indegree is 0 → b is taken first → then a's indegree drops

The edge direction in Problem 2 represents: "completing this course unlocks that course." This gives the topological order in the natural "take course b, which unlocks course a" sequence.

### Equivalent Result

Both directions give correct answers — they're just modeling the dependency graph differently. Problem 2's direction gives the order you'd actually take the courses (prerequisites come first naturally).

---

### Dry Run

```
V = 4
prerequisites = [[1,0],[2,0],[3,1],[3,2]]
"[1,0]: take 0 before 1" etc.

Problem 2 edge direction: adjLs[it[1]].push_back(it[0])
→ adjLs[0].push_back(1), adjLs[0].push_back(2)
   adjLs[1].push_back(3), adjLs[2].push_back(3)

indegree[it[0]]++
→ indegree[1]++, indegree[2]++, indegree[3]++, indegree[3]++

adjLs: 0→[1,2], 1→[3], 2→[3], 3→[]
indegree: [0, 1, 1, 2]

Seed: indegree[0]==0 → push 0
queue=[0], result=[]

Pop 0: result=[0]
  neighbor 1: indegree[1]--→0 → push 1
  neighbor 2: indegree[2]--→0 → push 2
queue=[1,2]

Pop 1: result=[0,1]
  neighbor 3: indegree[3]--→1 (not 0)
queue=[2]

Pop 2: result=[0,1,2]
  neighbor 3: indegree[3]--→0 → push 3
queue=[3]

Pop 3: result=[0,1,2,3]
queue=[]

result.size()==4==V → return [0,1,2,3] ✅

Verify: 0 before 1 ✅, 0 before 2 ✅, 1 before 3 ✅, 2 before 3 ✅
```

---

### Code

```cpp
class Solution {
public:
    vector<int> findOrder(int V, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adjLs(V);
        vector<int> indegree(V, 0);

        // prerequisites[i] = [a, b]: b before a → edge b→a (b unlocks a)
        for(auto it : prerequisites) {
            adjLs[it[1]].push_back(it[0]);   // b → a
            indegree[it[0]]++;                // a gets one more prerequisite
        }

        // Seed: courses with no prerequisites
        queue<int> q;
        for(int i = 0; i < V; i++) {
            if(indegree[i] == 0)
                q.push(i);
        }

        vector<int> result;
        while(!q.empty()) {
            int node = q.front();
            q.pop();
            result.push_back(node);

            for(auto& it : adjLs[node]) {
                indegree[it]--;
                if(indegree[it] == 0)
                    q.push(it);
            }
        }

        // Return order if all courses processed, else empty (cycle)
        if(result.size() == V)
            return result;

        return {};
    }
};
```

---

### Complexity

| | Complexity |
|---|---|
| **Time** | `O(V + E)` |
| **Space** | `O(V + E)` |

---

---

## Story Points

---

**Story Point 1 — "Both problems are just cycle detection + topological sort"**

Problem 1 asks: does a valid topological order exist? (= no cycle?)
Problem 2 asks: what IS that valid topological order?

Kahn's answers both simultaneously. The only difference is what you return.

---

**Story Point 2 — "The edge direction flip between Problem 1 and 2"**

This is the biggest trap. Both problems use the same `prerequisites[i] = [a, b]` format. But:

| | Edge added | What it models |
|---|---|---|
| Problem 1 | `a → b` (`adjLs[it[0]] → it[1]`) | "a depends on b" (dependency) |
| Problem 2 | `b → a` (`adjLs[it[1]] → it[0]`) | "b unlocks a" (sequence) |

Both give valid topological orderings, just modeled from different perspectives. Problem 2's direction happens to give the natural "take b first, then a" course order directly.

---

**Story Point 3 — "`result.size() == V` is the cycle detector"**

In both problems, this single check tells you if a cycle exists. Nodes in a cycle never reach indegree = 0 → never enter the queue → never enter `result`. If `result.size() < V`, some courses are stuck in a cycle → impossible → return `false` or `{}`.

---

**Story Point 4 — "Kahn's is perfect for prerequisite-style problems"**

Any problem that has "X must come before Y" constraints maps directly to Kahn's algorithm. The indegree model naturally captures "how many unmet prerequisites remain." When that hits zero, the node is ready — no complex tracking needed.

---

## Final Comparison

| | Problem 1 | Problem 2 |
|---|---|---|
| **Output** | `bool` | `vector<int>` |
| **Edge direction** | `adjLs[a].push_back(b)` | `adjLs[b].push_back(a)` |
| **Indegree incremented** | `indegree[b]++` | `indegree[a]++` |
| **Return condition** | `result.size() == V` | `result.size() == V ? result : {}` |
| **Core algorithm** | Kahn's BFS | Kahn's BFS (identical) |
| **TC** | `O(V + E)` | `O(V + E)` |
