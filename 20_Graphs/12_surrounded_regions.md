# Surrounded Regions

---

## 1. Problem Statement

Given an `n × m` board containing `'X'` and `'O'`, capture all regions that are **surrounded by `'X'`**.

A region is captured by flipping all `'O'`s in that region to `'X'`.

**The rule:** An `'O'` is NOT captured if it is:
- On the **border** of the board, OR
- Connected (4-directionally) to an `'O'` that is on the border

Any `'O'` that has NO path to the border (completely surrounded by `'X'`) → gets flipped to `'X'`.

```
Input:                Output:
X X X X              X X X X
X O O X    →         X X X X
X X O X              X X X X
X O X X              X O X X
```

The `'O'` at `(3,1)` is on the border → safe. The `'O'`s at `(1,1),(1,2),(2,2)` are completely surrounded → flipped to `'X'`.

---

## 2. Intuition / Approach

### The Wrong Way — Find Surrounded Regions Directly

Trying to directly find `'O'`s that are surrounded is hard. We'd need to check if every `'O'` can reach the border — essentially running DFS/BFS from every `'O'` cell, which is expensive and complex to reason about.

---

### The Right Way — Find What is SAFE, Then Flip Everything Else

**Reverse the thinking entirely.**

Instead of asking *"which `'O'`s are surrounded?"*, ask *"which `'O'`s are NOT surrounded (i.e., safe)?"*

A safe `'O'` is any `'O'` that:
- Is on the border, OR
- Is connected to a border `'O'`

**Algorithm:**

```
Step 1: Find all 'O's on the border and run DFS from each.
        Mark every 'O' reachable from the border as "safe" (vis=1).

Step 2: Scan the entire board.
        - If a cell is 'O' and NOT marked safe → it's surrounded → flip to 'X'
        - If a cell is 'X' or marked safe → leave it unchanged
```

This is a classic **"complement" trick** — instead of finding the hard set (surrounded `'O'`s), find the easy set (border-connected `'O'`s) and flip everything else.

---

### Why DFS From the Borders?

The border is the only way an `'O'` can "escape" being surrounded. If an `'O'` has any 4-directional path to the border, it's safe. DFS naturally explores the entire connected component — so starting DFS from a border `'O'` marks ALL `'O'`s connected to it as safe, no matter how far inland they are.

---

### Why `vis[][]` Instead of Modifying the Board Directly?

We could use a third character (e.g. `'S'` for safe) to mark the board in-place, but using a separate `vis[][]` is cleaner — it never risks confusing the original data, and the final pass is straightforward.

---

## 3. Dry Run

```
Board (4×4):
X X X X
X O O X
X X O X
X O X X

n=4, m=4
vis = all 0s
```

---

**Step 1 — Scan first & last row:**

```
Row 0: (0,0)=X, (0,1)=X, (0,2)=X, (0,3)=X → no 'O's
Row 3: (3,0)=X, (3,1)=O ✅ → dfs(3,1), (3,2)=X, (3,3)=X
```

**dfs(3,1):**
```
vis[3][1] = 1
Neighbors:
  up (2,1): board='X' → skip
  down (4,1): OOB → skip
  left (3,0): board='X' → skip
  right (3,2): board='X' → skip
← return
```

```
vis:
[ 0  0  0  0 ]
[ 0  0  0  0 ]
[ 0  0  0  0 ]
[ 0  1  0  0 ]
```

---

**Step 2 — Scan first & last column:**

```
Col 0: (0,0)=X,(1,0)=X,(2,0)=X,(3,0)=X → no 'O's
Col 3: (0,3)=X,(1,3)=X,(2,3)=X,(3,3)=X → no 'O's
```

No new DFS calls.

---

**Step 3 — Final scan:**

```
For every cell where board=='O' && !vis:
  (1,1): O, vis=0 → flip to X
  (1,2): O, vis=0 → flip to X
  (2,2): O, vis=0 → flip to X
  (3,1): O, vis=1 → SAFE, leave as O
```

```
Final board:
X X X X
X X X X
X X X X
X O X X  ✅
```

---

## 4. Story Points

---

**Story Point 1 — "Reverse the problem"**

The hardest mental leap: don't look for surrounded `'O'`s. Look for SAFE `'O'`s (border-connected). Everything else is automatically surrounded. This turns a hard problem into a simple DFS from borders.

---

**Story Point 2 — "Only the border matters as the entry point"**

An `'O'` deep inside the grid can still be safe — if it has a chain of `'O'`s leading to the border. DFS from the border propagates the "safe" marking inward through connected `'O'`s, no matter how deep.

```
X X X X X
X O O O X
X O X O X
X O O O X
X X X X X

Even the inner 'O's at (1,1),(2,1),(3,1),etc. are ALL unsafe —
none of them connect to the border.
DFS from borders finds NO 'O' → every 'O' gets flipped.
```

---

**Story Point 3 — "Corner cells are checked twice — that's fine"**

Corner cells (e.g. `(0,0)`) are checked in BOTH the row scan and the column scan. The `!vis[row][col]` guard prevents running DFS twice from the same cell — efficient and correct.

---

**Story Point 4 — "The vis array doubles as a 'safe' marker"**

`vis[i][j] = 1` doesn't just mean "visited by DFS" — it means "this `'O'` is connected to the border and is SAFE". This dual meaning makes the final pass trivial: `!vis && board=='O'` → flip.

---

**Story Point 5 — "We never flip 'X' to 'O'"**

The final pass only flips `'O'` to `'X'`. `'X'` cells are never touched. This is guaranteed by the condition:

```cpp
if(!vis[i][j] && board[i][j] == 'O')
    board[i][j] = 'X';
```

---

## 5. Code

```cpp
class Solution {
private:
    void dfs(int row, int col, vector<vector<char>>& vis,
             const int dRow[], const int dCol[],
             vector<vector<char>>& board) {

        vis[row][col] = 1;   // mark this 'O' as safe (border-connected)

        int n = board.size();
        int m = board[0].size();

        for(int i = 0; i < 4; i++) {
            int nRow = row + dRow[i];
            int nCol = col + dCol[i];

            // In bounds + unvisited + is 'O' → part of same safe region
            if(nRow >= 0 && nRow < n &&
               nCol >= 0 && nCol < m &&
               !vis[nRow][nCol] &&
               board[nRow][nCol] == 'O') {
                dfs(nRow, nCol, vis, dRow, dCol, board);
            }
        }
    }

public:
    void solve(vector<vector<char>>& board) {
        int n = board.size();
        int m = board[0].size();

        vector<vector<char>> vis(n, vector<char>(m, 0));
        const int dRow[4] = {-1, +1, 0, 0};
        const int dCol[4] = { 0,  0,-1,+1};

        // Step 1: DFS from all 'O's on the first and last row
        for(int j = 0; j < m; j++) {
            if(board[0][j] == 'O' && !vis[0][j])
                dfs(0, j, vis, dRow, dCol, board);

            if(board[n-1][j] == 'O' && !vis[n-1][j])
                dfs(n-1, j, vis, dRow, dCol, board);
        }

        // Step 2: DFS from all 'O's on the first and last column
        for(int i = 0; i < n; i++) {
            if(board[i][0] == 'O' && !vis[i][0])
                dfs(i, 0, vis, dRow, dCol, board);

            if(board[i][m-1] == 'O' && !vis[i][m-1])
                dfs(i, m-1, vis, dRow, dCol, board);
        }

        // Step 3: Flip all unvisited 'O's → they are surrounded
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m; j++) {
                if(!vis[i][j] && board[i][j] == 'O')
                    board[i][j] = 'X';
            }
        }
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M)`

| Step | Cost | Reason |
|---|---|---|
| Border scan (rows) | `O(M)` | Two rows of length M |
| Border scan (cols) | `O(N)` | Two columns of length N |
| All DFS calls combined | `O(N × M)` | Each cell visited at most once across all DFS calls (vis prevents re-entry) |
| Final flip scan | `O(N × M)` | Full grid scan |

**Total: `O(N × M)`**

> The `vis[][]` array is global across all DFS calls. Once marked, a cell is never entered again. So cumulative DFS work = N×M total cell visits.

---

### Space Complexity — `O(N × M)`

| Structure | Size | Reason |
|---|---|---|
| `vis[][]` array | `O(N × M)` | One entry per cell |
| DFS recursion stack | `O(N × M)` worst case | Entire board could be 'O' — DFS could go N×M levels deep |

**Total: `O(N × M)`**

> Worst case for recursion stack: entire board is `'O'` — DFS from a corner visits all N×M cells recursively without backtracking until the very end.

---

### Comparison — Direct vs Reverse Approach

| Approach | Time | Complexity of reasoning |
|---|---|---|
| Direct: check each 'O' for border connection | `O((NM)²)` worst case | Hard — need to trace every 'O' to border |
| **Reverse: mark safe from border, flip rest** | **`O(NM)`** | **Easy — DFS from borders, rest is surrounded** |
