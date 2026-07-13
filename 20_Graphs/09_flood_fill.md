# Flood Fill

Below is a ready-to-copy Markdown version of your explanation for `09_flood_fill.md`, written around your exact code and structured into the sections you asked for. The flood fill idea is to start from one cell, find all connected cells with the same original color, and repaint only that connected region [web:1][web:8].

## 1. Approach / Intuition

The intuition comes from the paint bucket tool in drawing apps: when you click one colored pixel, the tool changes that pixel and all nearby connected pixels of the same color into a new color [web:1][web:8].  
So instead of checking the whole image, we only expand from the starting cell `(sr, sc)` to its valid neighbors, as long as they belong to the same original color region [web:4][web:9].

The key idea is **connected component traversal**.  
If a neighbor has the same initial color, it is part of the same area, so we color it and continue exploring from there [web:4][web:9].

I used **BFS with a queue** here because it is a clean way to process cells level by level without recursion.  
The queue stores cells that still need to be explored, and each processed cell pushes its valid 4-directional neighbors into the queue [web:4][web:9].

## 2. How I Got This Intuition

First, I asked: “What changes after we color one cell?”  
Answer: only the cells that are connected to it and have the same original color should change too [web:1][web:6].

Then I noticed the problem is not about random cells in the matrix.  
It is about growing outward from one starting point, exactly like spreading paint across a connected region [web:4][web:8].

That leads naturally to graph thinking: each cell is a node, and each valid move to an adjacent cell is an edge.  
Once I thought in this way, BFS became a natural choice because it can explore all reachable same-colored cells systematically [web:4][web:5].

## 3. Story Point

1. Take the starting cell `(sr, sc)`.
2. Save its original color as `initialColor`.
3. If `initialColor` is already equal to the new `color`, return the image as it is.
4. Create a copy of the image in `result` so we can update it.
5. Put the starting cell into a queue and color it immediately.
6. Keep taking one cell from the queue.
7. Check its 4 neighbors: up, right, down, left.
8. For every neighbor inside the grid and still having `initialColor`, change it to `color`.
9. Push that neighbor into the queue so its neighbors can also be checked later.
10. Repeat until the queue becomes empty.
11. Return the final colored grid.

## 4. Dry Run

Take this example:

```cpp
image = [
 ,[1]
 ,[1]
[1]
]
sr = 1, sc = 1, color = 2
```

### Step 1
- `initialColor = image[1][1] = 1`
- `initialColor != color`, so continue.
- `result` becomes a copy of `image`.
- Push `(1,1)` into the queue.
- Change `result[1][1]` to `2`.

Queue:
```cpp
[(1,1)]
```

Result:
```cpp
[
 ,[1]
 ,[2][1]
[1]
]
```

### Step 2
Pop `(1,1)`.

Check 4 directions:
- Up → `(0,1)` has value `1`, so color it `2` and push it.
- Right → `(1,2)` has value `0`, ignore it.
- Down → `(2,1)` has value `0`, ignore it.
- Left → `(1,0)` has value `1`, so color it `2` and push it.

Queue:
```cpp
[(0,1), (1,0)]
```

Result:
```cpp
[
 ,[2][1]
 ,[2]
[1]
]
```

### Step 3
Pop `(0,1)`.

Check neighbors:
- Up → out of bounds.
- Right → `(0,2)` has value `1`, color it `2`, push it.
- Down → `(1,1)` is already `2`, ignore.
- Left → `(0,0)` has value `1`, color it `2`, push it.

Queue:
```cpp
[(1,0), (0,2), (0,0)]
```

Result:
```cpp
[
 ,[2]
 ,[2]
[1]
]
```

### Step 4
Pop `(1,0)`.

Check neighbors:
- Up → `(0,0)` already colored.
- Right → `(1,1)` already colored.
- Down → `(2,0)` has value `1`, color it `2`, push it.
- Left → out of bounds.

Queue:
```cpp
[(0,2), (0,0), (2,0)]
```

Result:
```cpp
[
 ,[2]
 ,[2]
[1][2]
]
```

### Step 5
Continue similarly for the remaining cells.  
In the end, all connected `1`s starting from `(1,1)` become `2`.

Final result:
```cpp
[
 ,[2]
 ,[2]
[1][2]
]
```

## 5. Code

```cpp
class Solution {
public:
    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
        int n = image.size();
        int m = image.size();


        int initialColor = image[sr][sc];


        if (initialColor == color)
            return image;


        vector<vector<int>> result = image;
        queue<pair<int, int>> q;


        q.push({sr, sc});
        result[sr][sc] = color;


        int dRow = {-1, 0, 1, 0};[3]
        int ddCol = {0, 1, 0, -1};[3]


        while (!q.empty()) {
            auto currentCell = q.front();
            q.pop();


            int row = currentCell.first;
            int col = currentCell.second;


            for (int i = 0; i < 4; i++) {
                int newRow = row + dRow[i];
                int newCol = col + ddCol[i];


                if (newRow >= 0 && newRow < n && newCol >= 0 && newCol < m &&
                    result[newRow][newCol] == initialColor) {


                    result[newRow][newCol] = color;
                    q.push({newRow, newCol});
                }
            }
        }


        return result;
    }
};
```

## 6. Complexity Analysis

Let `n` be the number of rows and `m` be the number of columns.  
In the worst case, every cell may be visited once, so the time complexity is `O(n * m)` [web:5].  

The queue can also store many cells in the worst case, so the extra space complexity is `O(n * m)` [web:5].  
The copied grid `result` also takes `O(n * m)` space, which is part of the implementation cost.

## 7. Why This Works

The algorithm only expands into neighbors that have the same `initialColor`, so it never crosses into a different region [web:4][web:6].  
That means every cell recolored is part of the same connected component as the starting cell [web:1][web:9].

The early return is also important.  
If the new color is already the same as the starting color, recoloring would do nothing, so we stop immediately [web:9].

## 8. Final Note

This is a BFS-based flood fill, and its main strength is that it is easy to understand and avoids recursive stack depth issues.  
The logic is simple: start from one cell, expand to valid neighbors, and continue until the region is fully painted [web:4][web:8].
