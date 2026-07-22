# Alien Dictionary (Alien Language Order)

---

## 1. Problem Statement

You're given a list of words from an alien language's dictionary, sorted in the alien language's lexicographic order. You're also given `k` — the number of distinct characters in the alien alphabet.

Determine the **order of characters** in the alien language.

```
dictionary = ["baa", "abcd", "abca", "cab", "cad"]
k = 4

From the dictionary order, we can infer:
'b' comes before 'a'   (baa vs abcd → first chars differ)
'a' comes before 'c'   (abcd vs cab → first chars differ... wait)

Actually comparing adjacent words:
"baa" vs "abcd" → 'b' before 'a'
"abcd" vs "abca" → 'd' before 'a'
"abca" vs "cab"  → 'a' before 'c'
"cab" vs "cad"   → 'b' before 'd'

Ordering: b < d < a < c  OR  d < b < a < c
One valid output: "bdac"
```

---

## 2. Intuition / Approach

### The Core Insight

The dictionary is sorted in alien lexicographic order. By **comparing adjacent words**, we can extract information about which character comes before which.

**Key Rule:** When comparing two adjacent words, find the **first position where they differ** — that character from the first word comes BEFORE that character from the second word in the alien alphabet.

```
"abcd" vs "abca"
Compare char by char:
  pos 0: 'a' == 'a' → no info
  pos 1: 'b' == 'b' → no info
  pos 2: 'c' == 'c' → no info
  pos 3: 'd' ≠ 'a' → 'd' comes before 'a' in alien order
  STOP after first difference
```

This gives us a **directed edge**: `d → a` (d must come before a).

### Building a DAG

Each character is a node (0 to k-1, where `'a'` = 0, `'b'` = 1, etc.).
Each ordering constraint `x before y` → directed edge `x → y`.

Once we have all constraints as a DAG, **topological sort** gives us the alien character order.

---

### Why Only Compare ADJACENT Words?

Because the dictionary is globally sorted. If "abc" < "abd" < "xyz" in alien order, then:
- "abc" vs "abd" → 'c' before 'd' (local constraint)
- "abd" vs "xyz" → 'a' before 'x' (local constraint)

By transitivity, all global ordering information is captured through adjacent pair comparisons.

---

### Why Stop at First Differing Character?

```
"abcd" vs "abca":
pos 3: 'd' ≠ 'a'
```

Once we find the first difference, we KNOW 'd' comes before 'a' because of how lexicographic ordering works — the first differing position determines the word order. Characters at positions beyond the first difference tell us NOTHING new about the ordering.

---

### The Algorithm

```
Step 1: Compare each adjacent pair of words in the dictionary
        Find first differing character → add directed edge
        Stop comparing after first difference (break)

Step 2: Run Kahn's topological sort on the character graph
        (characters = nodes, ordering constraints = edges)

Step 3: Convert topological order (0..k-1) back to characters ('a'..'z')
```

---

## 3. Dry Run

```
dictionary = ["baa", "abcd", "abca", "cab", "cad"]
k = 4  (characters: a=0, b=1, c=2, d=3)

adjLs = [[], [], [], []]  (4 nodes, no edges yet)
```

---

**Compare "baa" vs "abcd":**
```
p=0: 'b' ≠ 'a' → edge b→a (1→0)
break
adjLs[1].push_back(0)  →  adjLs: [[], [0], [], []]
```

**Compare "abcd" vs "abca":**
```
p=0: 'a'=='a' skip
p=1: 'b'=='b' skip
p=2: 'c'=='c' skip
p=3: 'd' ≠ 'a' → edge d→a (3→0)
break
adjLs[3].push_back(0)  →  adjLs: [[], [0], [], [0]]
```

**Compare "abca" vs "cab":**
```
p=0: 'a' ≠ 'c' → edge a→c (0→2)
break
adjLs[0].push_back(2)  →  adjLs: [[2], [0], [], [0]]
```

**Compare "cab" vs "cad":**
```
p=0: 'c'=='c' skip
p=1: 'a'=='a' skip
p=2: 'b' ≠ 'd' → edge b→d (1→3)
break
adjLs[1].push_back(3)  →  adjLs: [[2], [0,3], [], [0]]
```

**Graph built:**
```
Edges: b→a (1→0), d→a (3→0), a→c (0→2), b→d (1→3)

adjLs:
0(a) → [2(c)]
1(b) → [0(a), 3(d)]
2(c) → []
3(d) → [0(a)]
```

---

**Kahn's Topological Sort:**

```
Compute indegree:
  0(a): incoming from 1(b→a) and 3(d→a) → indegree=2
  1(b): no incoming → indegree=0
  2(c): incoming from 0(a→c) → indegree=1
  3(d): incoming from 1(b→d) → indegree=1

indegree = [2, 0, 1, 1]

Seed queue: indegree=0 → push 1(b)
queue=[1], result=[]

Pop 1(b): result=[1]
  neighbor 0(a): indegree[0]--→1
  neighbor 3(d): indegree[3]--→0 → push 3
queue=[3]

Pop 3(d): result=[1,3]
  neighbor 0(a): indegree[0]--→0 → push 0
queue=[0]

Pop 0(a): result=[1,3,0]
  neighbor 2(c): indegree[2]--→0 → push 2
queue=[2]

Pop 2(c): result=[1,3,0,2]
queue=[]

Topo order (indices): [1, 3, 0, 2]
Convert: [b, d, a, c]
Answer: "bdac" ✅
```

**Verify:**
```
b < d: b before d ✅ (from b→d edge)
d < a: d before a ✅ (from d→a edge)
a < c: a before c ✅ (from a→c edge)
b < a: b before a ✅ (from b→a edge)
```

---

## 4. Story Points

---

**Story Point 1 — "Only adjacent word comparisons — not all pairs"**

We don't compare every word against every other word. We ONLY compare adjacent pairs. This works because the dictionary is sorted — all ordering information is encoded in adjacent relationships. Comparing "baa" to "cad" directly gives no extra info beyond what's already captured by "baa"→"abcd"→...→"cad" chain comparisons.

---

**Story Point 2 — "Stop at first differing character (the `break`)"**

```cpp
for(int p = 0; p < len; p++) {
    if(s1[p] != s2[p]) {
        adjLs[s1[p]-'a'].push_back(s2[p]-'a');
        break;   // ← CRITICAL
    }
}
```

After finding the first differing character, we know the ordering for that position. Characters at later positions in these two words have NO ordering relationship implied — only the first difference determines which word comes first lexicographically. Without `break`, you'd add false edges.

---

**Story Point 3 — "Characters map to integers 0..k-1 via `char - 'a'`"**

```cpp
adjLs[s1[p] - 'a'].push_back(s2[p] - 'a');
```

`'a' - 'a' = 0`, `'b' - 'a' = 1`, `'c' - 'a' = 2`...

This maps characters directly to node indices. The reverse conversion: `char(it + 'a')` maps indices back to characters for the answer string.

Wait — the code uses `char(it - 'a')` which would give control characters. The correct conversion should be `char(it + 'a')`. This is a bug in the original code — the output should be:

```cpp
ans += char(it + 'a');   // correct: 0+'a'='a', 1+'a'='b', etc.
```

---

**Story Point 4 — "The result is a topological order of CHARACTERS not words"**

We sort characters, not words. The character ordering we derive IS the alien alphabet order. Topological sort on the character dependency graph gives the alien alphabet sequence.

---

**Story Point 5 — "Invalid input cases (not handled here)"**

Two edge cases this code doesn't handle:
1. **Same prefix, second word shorter**: `["abc", "ab"]` — "ab" can't come after "abc" in valid lex order → invalid dictionary → should return `""`
2. **No unique order**: Multiple valid orderings exist (character appears in no comparison) → any valid topo order is acceptable

---

## 5. Code

```cpp
// Kahn's topological sort helper
vector<int> topoSort(int V, vector<vector<int>>& adjLs) {
    vector<int> indegree(V, 0);

    // Compute indegrees
    for(int i = 0; i < V; i++) {
        for(auto& it : adjLs[i]) {
            indegree[it]++;
        }
    }

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

        for(auto& neighbour : adjLs[node]) {
            indegree[neighbour]--;
            if(indegree[neighbour] == 0)
                q.push(neighbour);
        }
    }

    return result;
}

string getAlienLanguage(vector<string>& dictionary, int k) {
    // k characters → k nodes (a=0, b=1, ..., z=25)
    vector<vector<int>> adjLs(k);

    // Step 1: Extract ordering constraints from adjacent word pairs
    for(int i = 0; i < (int)dictionary.size() - 1; i++) {
        string s1 = dictionary[i];
        string s2 = dictionary[i + 1];
        int len = min(s1.size(), s2.size());

        for(int p = 0; p < len; p++) {
            if(s1[p] != s2[p]) {
                // s1[p] comes before s2[p] in alien order
                adjLs[s1[p] - 'a'].push_back(s2[p] - 'a');
                break;   // only first differing char matters
            }
        }
    }

    // Step 2: Topological sort on character graph
    vector<int> result = topoSort(k, adjLs);

    // Step 3: Convert indices back to characters
    string ans = "";
    for(auto& it : result) {
        ans += char(it + 'a');   // 0→'a', 1→'b', 2→'c', etc.
    }

    return ans;
}
```

> **Bug fix:** The original code used `char(it - 'a')` which gives control characters. The correct conversion is `char(it + 'a')`.

---

## 6. Complexity Analysis

### Time Complexity — `O(N × L + K)`

Where:
- `N` = number of words in dictionary
- `L` = average word length (for comparisons)
- `K` = number of distinct characters (nodes in graph)

| Step | Cost | Reason |
|---|---|---|
| Extract edges (adjacent word comparisons) | `O(N × L)` | N-1 pairs, each comparison up to L characters |
| Build indegree | `O(K + E)` | K nodes, E edges (at most N-1 edges from N-1 pairs) |
| Kahn's BFS | `O(K + E)` | K character nodes + E edges |
| Build answer string | `O(K)` | At most K characters |

**Total: `O(N × L + K)`**

> `E` (number of edges) ≤ `N-1` since each adjacent pair contributes at most one edge. So `E` ≤ `N`, making the graph part `O(K + N)`.

---

### Space Complexity — `O(K + E)` = `O(K + N)`

| Structure | Size | Reason |
|---|---|---|
| `adjLs` | `O(K + E)` | K character nodes + E edges |
| `indegree[]` | `O(K)` | One per character |
| BFS queue | `O(K)` | At most K characters |
| `result` | `O(K)` | K characters in topo order |

**Total: `O(K + N)`**
