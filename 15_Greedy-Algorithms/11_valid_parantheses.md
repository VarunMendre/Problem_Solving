# Check Valid String (Parenthesis with *) — Brute Force + Optimal

---

# 🔹 1️⃣ Recursive Backtracking Approach

```cpp
class Solution {
private:
    bool isValid(string& s, int i, int open) {
        if (i == s.size())
            return open == 0;
        if (open < 0)
            return false;

        if (s[i] == '(') {
            return isValid(s, i + 1, open + 1);
        }
        else if (s[i] == ')') {
            return isValid(s, i + 1, open - 1);
        }
        else {
            return isValid(s, i + 1, open + 1) ||
                   isValid(s, i + 1, open - 1) ||
                   isValid(s, i + 1, open);
        }
    }

public:
    bool checkValidString(string s) {
        return isValid(s, 0, 0);
    }
};
```

---

## 1️⃣ Idea

At every position:

- '(' → increase open count
- ')' → decrease open count
- '*' → can act as:
  - '('
  - ')'
  - empty

So we explore all possibilities using recursion.

---

## 2️⃣ Base Case

```cpp
if (i == s.size())
    return open == 0;
```

If we reach end:
- Valid only if no unmatched '(' remain.

---

## 3️⃣ Invalid Case

```cpp
if (open < 0)
    return false;
```

More ')' than '(' → invalid immediately.

---

## 4️⃣ Recursive Choices

For '*', three branches are explored:

- Treat as '('
- Treat as ')'
- Treat as empty

This causes exponential branching.

---

## 5️⃣ Complexity

Time Complexity:

O(3^n)

(Each '*' creates 3 recursive branches)

Space Complexity:

O(n)

(Recursion stack depth)

---

# 🔹 2️⃣ Greedy Optimal Approach

```cpp
class Solution {
public:
    bool checkValidString(string s) {
        int n = s.size();

        int min = 0, max = 0;

        for(int i = 0; i < n; i++) {
            if(s[i] == '(') {
                min += 1;
                max += 1;
            }
            else if(s[i] == ')') {
                min -= 1;
                max -= 1; 
            }
            else {
                min -= 1;
                max += 1;
            }

            if(min < 0) min = 0;
            if(max < 0) return false;
        }

        return (min == 0);
    }
};
```

---

## 1️⃣ Core Idea

Instead of trying all possibilities,
we maintain a RANGE of possible open brackets.

- `min` → minimum possible open brackets
- `max` → maximum possible open brackets

---

## 2️⃣ How Range Works

For each character:

### If '(':
- min++
- max++

### If ')':
- min--
- max--

### If '*':
- It could be ')'
  → min--
- It could be '('
  → max++

So range expands.

---

## 3️⃣ Important Conditions

```cpp
if(min < 0) min = 0;
```

Minimum open cannot go below 0.

```cpp
if(max < 0) return false;
```

If maximum becomes negative:
Even treating all '*' as '(' cannot fix it.
So invalid.

---

## 4️⃣ Final Check

```cpp
return (min == 0);
```

If minimum possible open is zero,
we can form a valid sequence.

---

## 5️⃣ Complexity

Time Complexity:

O(n)

Single traversal.

Space Complexity:

O(1)

Only two variables used.

---

# ✅ Final Comparison

Recursive: