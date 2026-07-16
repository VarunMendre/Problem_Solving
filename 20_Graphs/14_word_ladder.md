# Word Ladder

---

## 1. Problem Statement

Given a `beginWord`, an `endWord`, and a `wordList`, find the **length of the shortest transformation sequence** from `beginWord` to `endWord` such that:

- Only **one letter** can be changed at a time
- Every intermediate word must exist in `wordList`

Return the **number of words** in the shortest sequence. If no such sequence exists, return `0`.

```
beginWord = "hit"
endWord   = "cog"
wordList  = ["hot","dot","dog","lot","log","cog"]

Shortest path:
hit → hot → dot → dog → cog
     (1)   (2)   (3)   (4)   (5)

Return: 5
```

---

## 2. Intuition / Approach

### Modelling as a Graph Problem

Think of every word as a **node**. Draw an edge between two words if they differ by exactly **one character**.

```
hit — hot — dot — dog — cog
             |         |
            lot — log —+
```

Now the problem becomes: **find the shortest path from `beginWord` to `endWord`** in this implicit graph.

**Shortest path in an unweighted graph → BFS.**

BFS guarantees that the first time we reach `endWord`, it's via the minimum number of steps.

---

### Why Not Build the Graph Explicitly?

Building the full graph would require comparing every pair of words — `O(N² × L)` where N = wordList size, L = word length. For large word lists this is expensive.

Instead, we use a smarter approach: **for each word, try all possible one-character mutations** and check if the mutated word exists in the word set. This avoids building the graph and lets us discover edges on-the-fly.

---

### The Mutation Trick

For a word of length `L`, there are `L × 25` possible one-character mutations (each position can become any of the 25 other letters).

```
word = "hit"
Position 0: ait, bit, cit ... zit  (25 options)
Position 1: hat, hbt, hct ... hzt  (25 options)
Position 2: hia, hib, hic ... hiz  (25 options)
```

For each mutation, check if it's in the word set → if yes, it's a valid next step.

---

### Why `unordered_set` Instead of `vector`?

After building the set from `wordList`, each lookup (checking if a mutated word exists) costs `O(1)` average vs `O(N)` for a vector search.

Also, we **erase words from the set once they're enqueued** — this serves as the "visited" mechanism. A word removed from the set can never be re-enqueued, preventing revisiting and infinite loops.

**Why erase = visited?**  
If a word is already reached at step `k`, reaching it again at step `k+1` or later would only give a longer path. We never want to re-visit.

---

### Why Erase `beginWord` Immediately?

```cpp
st.erase(beginWord);
```

`beginWord` is our starting point. If `beginWord` itself appears in `wordList`, and some mutation leads back to it, we'd form a cycle. Erasing it prevents the BFS from ever returning to the start.

---

### Early Exit — `endWord` Not in List

```cpp
if(st.find(endWord) == st.end())
    return 0;
```

If `endWord` is not in `wordList`, it's impossible to reach it (we can only step to words in the list). Return `0` immediately.

---

### BFS Queue Format

```
queue stores: {word, steps}
```

`steps` = number of words in the transformation sequence so far (including `beginWord`). When we reach `endWord`, `steps` is our answer.

Note: the sequence length includes both endpoints. That's why we start with `{beginWord, 1}` — beginWord itself counts as step 1.

---

## 3. Dry Run

```
beginWord = "hit"
endWord   = "cog"
wordList  = ["hot","dot","dog","lot","log","cog"]

st = {hot, dot, dog, lot, log, cog}
q  = [("hit", 1)]
st.erase("hit") → "hit" not in set anyway, no change
```

---

**Pop ("hit", 1):**

```
Try all mutations of "hit":
  pos 0 'h':
    ait...zit: none in st
  pos 1 'i':
    hat → not in st
    hbt... → not in st
    hot → IN ST ✅ → push ("hot",2), erase "hot"
    hut, hvt... → not in st
  pos 2 't':
    hia...hiz → none in st

queue = [("hot",2)]
st = {dot, dog, lot, log, cog}
```

---

**Pop ("hot", 2):**

```
Mutations of "hot":
  pos 0 'h':
    aot...zot: dot ✅ → push ("dot",3), erase "dot"
               lot ✅ → push ("lot",3), erase "lot"
  pos 1 'o':
    hat,hbt...hut → none in st
  pos 2 't':
    hoa...hoz → none in st

queue = [("dot",3), ("lot",3)]
st = {dog, log, cog}
```

---

**Pop ("dot", 3):**

```
Mutations:
  pos 0: aot...got: none | lot→erased | ...zot→none
  pos 1: dat,dbt...dzt: none
  pos 2: doa...doz: dog ✅ → push ("dog",4), erase "dog"

queue = [("lot",3), ("dog",4)]
st = {log, cog}
```

---

**Pop ("lot", 3):**

```
Mutations:
  pos 0: aot...zot: dot→erased
  pos 1: lat...lzt: none
  pos 2: loa...loz: log ✅ → push ("log",4), erase "log"

queue = [("dog",4), ("log",4)]
st = {cog}
```

---

**Pop ("dog", 4):**

```
Mutations:
  pos 0: aog...zog: cog ✅ → push ("cog",5), erase "cog"
  pos 1: dag...dzg: none
  pos 2: doa...doz: dot→erased

queue = [("log",4), ("cog",5)]
st = {}
```

---

**Pop ("log", 4):**

```
Mutations:
  pos 0: aog...zog: cog → already erased → not in st
  pos 1,2: none in st

queue = [("cog",5)]
```

---

**Pop ("cog", 5):**

```
word == endWord ✅ → return 5
```

**Answer: 5** ✅

Path: `hit → hot → dot → dog → cog` (5 words)

---

## 4. Story Points

---

**Story Point 1 — "Graph is implicit — we never build it"**

We never explicitly store edges. Instead, we generate potential edges on-the-fly by trying all 25 mutations per character position. The `unordered_set` lookup tells us instantly if that edge is valid. This saves `O(N²)` graph building time.

---

**Story Point 2 — "Erase from set = visited marker"**

Using the set itself as the visited structure is elegant. Once a word is enqueued (and erased), it's gone — no separate `vis[]` array needed. And it's correct: we never want to revisit a word via a longer path.

---

**Story Point 3 — "Why BFS and not DFS?"**

DFS would find A path, but not necessarily the SHORTEST one. It could go `hit→hot→lot→log→dog→cog` (6 steps) before finding the shorter 5-step path.

BFS guarantees: the FIRST time we reach `endWord`, it's via the minimum number of steps. This is the fundamental guarantee of BFS on unweighted graphs.

---

**Story Point 4 — "The `word[i] = original` restore step is crucial"**

```cpp
char original = word[i];
for(char ch = 'a'; ch <= 'z'; ch++) {
    word[i] = ch;
    // ... check and maybe push
}
word[i] = original;   // RESTORE before next position
```

We mutate `word` in-place to avoid creating N×L copies. But we must restore the original character after trying all mutations at position `i`, before moving to position `i+1`. Without this restore, position `i+1` would be working with a corrupted word.

---

**Story Point 5 — "Steps start at 1, not 0"**

The problem asks for the number of words in the transformation sequence (including both `beginWord` and `endWord`). That's why we push `{beginWord, 1}` — beginWord itself is step 1. When we reach endWord, steps = total words in the sequence.

---

## 5. Code

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {

        // Build word set for O(1) lookups — also serves as visited tracker
        unordered_set<string> st(wordList.begin(), wordList.end());

        // Early exit: if endWord not reachable
        if(st.find(endWord) == st.end())
            return 0;

        // BFS queue: {word, steps}
        queue<pair<string,int>> q;
        q.push({beginWord, 1});
        st.erase(beginWord);   // prevent cycling back to start

        while(!q.empty()) {
            string word  = q.front().first;
            int steps    = q.front().second;
            q.pop();

            // Reached destination → return sequence length
            if(word == endWord)
                return steps;

            // Try all one-character mutations
            for(int i = 0; i < word.size(); i++) {
                char original = word[i];

                for(char ch = 'a'; ch <= 'z'; ch++) {
                    if(ch == original) continue;   // skip no-change

                    word[i] = ch;   // mutate

                    if(st.find(word) != st.end()) {
                        q.push({word, steps + 1});
                        st.erase(word);            // mark visited by erasing
                    }
                }

                word[i] = original;   // restore before next position
            }
        }

        return 0;   // no path found
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × L × 26)`

| Step | Cost | Reason |
|---|---|---|
| Build `unordered_set` | `O(N × L)` | N words of length L each |
| BFS — each word dequeued once | `O(N)` | At most N words processed |
| Per word: try all mutations | `O(L × 26)` | L positions × 26 letters |
| Each set lookup | `O(L)` average | String hash computation is O(L) |

**Total: `O(N × L × 26)` = `O(N × L)`** (26 is a constant)

> Where `N` = size of wordList, `L` = length of each word (all assumed equal length).

---

### Space Complexity — `O(N × L)`

| Structure | Size | Reason |
|---|---|---|
| `unordered_set` | `O(N × L)` | Stores all N words of length L |
| BFS queue | `O(N × L)` | At most N words in queue, each of length L |

**Total: `O(N × L)`**

---

---

## 7. Optimal — Bidirectional BFS

### Why Bidirectional BFS?

In standard BFS, the frontier expands outward from `beginWord` in all directions. In the worst case, BFS explores the entire reachable graph before finding `endWord`.

**Bidirectional BFS** runs two BFS searches simultaneously:
- **Forward frontier** — expanding from `beginWord`
- **Backward frontier** — expanding from `endWord`

When the two frontiers **meet** (a word from one frontier appears in the other), we've found the shortest path.

---

### Why is it Faster?

Standard BFS frontier at depth `d` has roughly `B^d` nodes (B = branching factor).

- Single BFS: explores `B^d` nodes
- Bidirectional BFS: each side explores `B^(d/2)` nodes → total `2 × B^(d/2)`

For `B=26, d=10`: Single = `26^10` ≈ 1.4 × 10¹⁴, Bidirectional = `2 × 26^5` ≈ 2.4 × 10⁷.

The savings are **exponential**.

---

### Always Expand the Smaller Frontier

```cpp
if(beginSet.size() > endSet.size())
    swap(beginSet, endSet);
```

At each step, we always expand whichever frontier is currently smaller. This keeps both frontiers balanced — preventing one side from exploding while the other stays tiny.

After `swap`, `beginSet` is always the smaller one → we expand it → generate `nextLevel`.

---

### How the Meeting is Detected

```cpp
if(endSet.find(word) != endSet.end())
    return steps + 1;
```

While generating mutations of words in `beginSet`, if a mutated word exists in `endSet` — the two frontiers have met. The path length is `steps + 1`:
- `steps` = levels expanded so far
- `+1` = the current expansion that bridged the gap

---

### `wordSet` Still Acts as Visited

Words found in `wordSet` are added to `nextLevel` and **erased from `wordSet`** — same visited mechanism as the standard BFS. This prevents words from being revisited from either frontier.

---

### Code

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord,
                     vector<string>& wordList) {

        unordered_set<string> wordSet(wordList.begin(), wordList.end());

        // If endWord doesn't exist, transformation is impossible
        if(wordSet.find(endWord) == wordSet.end())
            return 0;

        // Two frontiers — one from each end
        unordered_set<string> beginSet, endSet;
        beginSet.insert(beginWord);
        endSet.insert(endWord);

        int steps = 1;
        int len = beginWord.size();

        while(!beginSet.empty() && !endSet.empty()) {

            // Always expand the smaller frontier for optimal performance
            if(beginSet.size() > endSet.size())
                swap(beginSet, endSet);

            unordered_set<string> nextLevel;

            for(string word : beginSet) {
                for(int i = 0; i < len; i++) {
                    char original = word[i];

                    for(char ch = 'a'; ch <= 'z'; ch++) {
                        if(ch == original) continue;

                        word[i] = ch;

                        // Frontiers meet → shortest path found
                        if(endSet.find(word) != endSet.end())
                            return steps + 1;

                        // Valid unvisited word → add to next level
                        if(wordSet.find(word) != wordSet.end()) {
                            nextLevel.insert(word);
                            wordSet.erase(word);   // mark visited
                        }
                    }

                    word[i] = original;   // restore before next position
                }
            }

            // Move to next level
            beginSet = move(nextLevel);
            steps++;
        }

        return 0;   // no path found
    }
};
```

---

### Complexity Analysis

### Time Complexity — `O(N × L × 26)` but with much smaller constant

| | Standard BFS | Bidirectional BFS |
|---|---|---|
| Nodes explored | `O(B^d)` | `O(2 × B^(d/2))` |
| Worst case TC | `O(N × L × 26)` | `O(N × L × 26)` same asymptotic |
| Practical speedup | — | Exponentially faster for large graphs |

> Asymptotically both are `O(N × L × 26)` = `O(N × L)`, but in practice bidirectional BFS explores far fewer nodes due to the exponential frontier reduction.

### Space Complexity — `O(N × L)`

| Structure | Size |
|---|---|
| `wordSet` | `O(N × L)` |
| `beginSet` + `endSet` | `O(N × L)` combined |
| `nextLevel` | `O(N × L)` worst case |

**Total: `O(N × L)`**

---

### Standard BFS vs Bidirectional BFS

| | Standard BFS | Bidirectional BFS |
|---|---|---|
| **Frontiers** | One (from `beginWord`) | Two (from both ends) |
| **Meeting condition** | Reach `endWord` | Frontiers intersect |
| **Expansion strategy** | Always expand forward | Always expand smaller frontier |
| **Practical speed** | Slower for large graphs | Exponentially faster |
| **Code complexity** | Simpler | Slightly more complex |
| **Asymptotic TC** | `O(N × L)` | `O(N × L)` |
