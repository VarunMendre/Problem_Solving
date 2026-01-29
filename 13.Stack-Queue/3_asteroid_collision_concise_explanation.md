# Asteroid Collision

---

## 1. Intuition / Approach

- A collision is possible **only when a right-moving asteroid (`+`) meets a left-moving asteroid (`-`)**.
- Use a **stack-like structure** to keep track of asteroids that are still alive.
- Traverse asteroids from left to right:
  - Push all **right-moving asteroids** directly into the stack.
  - For a **left-moving asteroid**, compare it with the stack’s top:
    - Pop smaller right-moving asteroids (they explode).
    - If both asteroids have the same size, both explode.
    - If the stack becomes empty or the top is also moving left, the current asteroid survives.
- Each asteroid is pushed and popped at most once, making the approach efficient.

---

## 2. Code

```cpp
class Solution {
public:
    vector<int> asteroidCollision(vector<int>& asteroids) {
        vector<int> st;
        int n = asteroids.size();
        
        for (int i = 0; i < n; i++) {
            if (asteroids[i] > 0)
                st.push_back(asteroids[i]);

            else {
                while (!st.empty() && st.back() > 0 &&
                       st.back() < abs(asteroids[i])) {
                       st.pop_back();
                }

                if(!st.empty() && st.back() == abs((asteroids[i]))){
                    st.pop_back();
                }
                else if(st.empty() || st.back() < 0) {
                    st.push_back(asteroids[i]);
                }
            }
        }

        return st;
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity
- Each asteroid is pushed and popped **at most once**.

```
TC = O(2N) ≈ O(N)
```

### Space Complexity
- Stack can store up to `N` asteroids in the worst case.

```
SC = O(N)
```

---

