# Design Twitter

---

## 1. Intuition / Approach

We need to support 4 operations efficiently:
- **postTweet** — store a tweet with a timestamp
- **getNewsFeed** — get 10 most recent tweets from self + followed users
- **follow / unfollow** — manage relationships

### Data Structures

| Structure | Purpose |
|---|---|
| `unordered_map<int, vector<pair<int,int>>> tweets` | Maps `userId → [(timestamp, tweetId)]` — tweet history per user |
| `unordered_map<int, unordered_set<int>> following` | Maps `userId → {set of followees}` — who each user follows |
| `int time` | Global counter — increments on every tweet to establish recency order |

### Why a Timestamp?

We need to compare tweets across multiple users by recency. A global `time` counter acts as a timestamp — higher value = more recent. This lets us easily compare tweets from different users.

---

### Core Logic — `getNewsFeed`

We need the **10 most recent** tweets from self + all followed users.

**Strategy: Min Heap of size 10**

Use a min heap (min = oldest among top-10). As we iterate through all relevant tweets:
- Push each tweet into the heap
- If heap size exceeds 10 → pop the oldest (smallest timestamp)

At the end, the heap contains exactly the 10 most recent tweets — but in ascending order. Reverse to get newest-first.

This is the same **sliding window of K largest** pattern from Kth Largest Element.

---

### Bug in the Code

```cpp
Twitter() { int time = 0; }  // ❌ declares a LOCAL variable
```

This creates a local `time` instead of initializing the member `time`. The member `time` is left uninitialized (undefined behavior).

**Fix:**
```cpp
Twitter() : time(0) {}       // ✅ initializer list
// OR
Twitter() { this->time = 0; } // ✅ explicit member access
```

---

### Dry Run

```
Twitter obj;
obj.postTweet(1, 5)    → tweets[1] = [(0, 5)]       time=1
obj.postTweet(1, 3)    → tweets[1] = [(0,5),(1,3)]   time=2
obj.postTweet(2, 7)    → tweets[2] = [(2, 7)]        time=3
obj.follow(1, 2)       → following[1] = {2}

obj.getNewsFeed(1):
  Own tweets: push (0,5) → pq=[(0,5)], push (1,3) → pq=[(0,5),(1,3)]
  Followed user 2: push (2,7) → pq=[(0,5),(1,3),(2,7)]
  Size ≤ 10 → no pops

  pq (min heap by timestamp): top=(0,5)
  pop order: (0,5) → (1,3) → (2,7)
  res = [5, 3, 7]
  reverse → [7, 3, 5]

return [7, 3, 5]   ← tweet 7 is most recent ✅
```

---

## 2. Code

```cpp
class Twitter {
private:
    // userId → [(timestamp, tweetId)] — tweet history
    unordered_map<int, vector<pair<int,int>>> tweets;

    // userId → {set of followees}
    unordered_map<int, unordered_set<int>> following;

    int time;   // global counter for recency ordering

public:
    Twitter() : time(0) {}   // ✅ correctly initialize member variable

    void postTweet(int userId, int tweetId) {
        // Store tweet with current timestamp, then increment
        tweets[userId].push_back({time++, tweetId});
    }

    vector<int> getNewsFeed(int userId) {
        // Min heap — keeps the 10 most recent tweets (oldest on top for eviction)
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;

        // Add own tweets
        for(auto& t : tweets[userId]) {
            pq.push(t);
            if(pq.size() > 10) pq.pop();    // evict oldest if exceeds 10
        }

        // Add tweets from followed users
        for(auto followee : following[userId]) {
            for(auto& t : tweets[followee]) {
                pq.push(t);
                if(pq.size() > 10) pq.pop();
            }
        }

        // Extract in ascending order, then reverse for newest-first
        vector<int> res;
        while(!pq.empty()) {
            res.push_back(pq.top().second);
            pq.pop();
        }
        reverse(res.begin(), res.end());

        return res;
    }

    void follow(int followerId, int followeeId) {
        following[followerId].insert(followeeId);
    }

    void unfollow(int followerId, int followeeId) {
        following[followerId].erase(followeeId);
    }
};
```

---

## 3. Time & Space Complexity

### Per-Operation Complexity

| Operation | Time | Reason |
|---|---|---|
| `postTweet` | `O(1)` | Append to vector + increment counter |
| `follow` | `O(1)` avg | Insert into unordered_set |
| `unfollow` | `O(1)` avg | Erase from unordered_set |
| `getNewsFeed` | `O(T log 10)` = `O(T)` | T = total tweets from self + followees; heap size capped at 10 → `log 10` = constant |

### Space Complexity

| Structure | Space | Reason |
|---|---|---|
| `tweets` | `O(total tweets)` | Stores all tweets ever posted |
| `following` | `O(total follow relationships)` | Stores all follow pairs |
| `pq` in getNewsFeed | `O(10)` = `O(1)` | Capped at 10 elements |
