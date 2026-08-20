# Accounts Merge

---

## 1. Problem Statement

You are given a list of `accounts` where `accounts[i][0]` is a person's name and `accounts[i][1..k]` are email addresses belonging to that account.

**Two accounts belong to the same person** if they share at least one common email address.

Merge all accounts belonging to the same person. Return the merged accounts where each entry contains the person's name followed by all their emails in **sorted order**.

```
accounts = [
  ["John","johnsmith@mail.com","john_newyork@mail.com"],
  ["John","johnsmith@mail.com","john00@mail.com"],
  ["Mary","mary@mail.com"],
  ["John","johnnybravo@mail.com"]
]

Account 0 and Account 1 both have "johnsmith@mail.com" → same person → merge

Output: [
  ["John","john00@mail.com","john_newyork@mail.com","johnsmith@mail.com"],
  ["Mary","mary@mail.com"],
  ["John","johnnybravo@mail.com"]
]
```

---

## 2. Intuition / Approach

### The Core Problem — Grouping by Shared Identity

Two accounts that share an email are the SAME person. This is a connectivity/grouping problem:
- Each **account** = a node
- Two accounts are **connected** if they share an email

This is exactly what DSU solves — grouping connected components efficiently.

---

### The Three-Phase Algorithm

**Phase 1 — Build connections via shared emails:**

```
For each account i:
  For each email in account i:
    If email seen before (in account j):
      union(i, j)   ← accounts i and j are the same person
    Else:
      record emailsToId[email] = i
```

After this phase: DSU components = groups of accounts belonging to the same person.

**Phase 2 — Assign each email to its component's root:**

```
For each email in emailsToId:
  root = findUParent(emailsToId[email])
  mergedMails[root].add(email)
```

All emails belonging to the same person now land in the same bucket (`mergedMails[root]`).

**Phase 3 — Build the answer:**

```
For each account i:
  If mergedMails[i] is non-empty:  ← i is a root of some component
    Sort mergedMails[i]
    Prepend accounts[i][0] (the person's name)
    Add to answer
```

---

### Why Account Index (Not Email) as DSU Node?

The DSU is indexed by **account number** (0 to n-1), not by email. This is because:
- n accounts → n DSU nodes (compact, predictable size)
- Emails are the "bridge" that tells us WHICH accounts to union
- After merging, we bucket emails BY ACCOUNT COMPONENT

If we indexed by email, we'd need a DSU of potentially millions of nodes (one per email character combination) — wasteful and complex.

---

### The `emailsToId` Map

```cpp
unordered_map<string, int> emailsToId;
// email → account index that first claimed this email
```

When processing account `i` and email `mail`:
- **Email not seen before:** `emailsToId[mail] = i` (account `i` owns this email)
- **Email seen before (owned by account `j`):** `ds.unionBySize(i, j)` — accounts `i` and `j` share this email → same person

This map has two purposes:
1. Detect shared emails (trigger unions)
2. Later, map each email back to its account (for bucketing in Phase 2)

---

### The `mergedMails[n]` Array

```cpp
vector<string> mergedMails[n];   // n buckets, indexed by account number
```

In Phase 2, we use `ds.findUParent(emailsToId[mail])` to find the ROOT account for each email, then put the email in that root's bucket.

After Phase 2, `mergedMails[i]` is non-empty ONLY for `i` values that are roots of real components. All other `mergedMails[i]` remain empty → skipped in Phase 3.

---

### Getting the Person's Name

```cpp
temp.push_back(accounts[i][0]);
```

`accounts[i][0]` is the name field. Since `i` is the root of a merged group, and all accounts in the group have the same person's name, `accounts[i][0]` gives the correct name.

---

## 3. Dry Run

```
accounts = [
  0: ["John","a@m.com","b@m.com"],
  1: ["John","c@m.com"],
  2: ["Mary","b@m.com"],   ← shares b@m.com with account 0!
  3: ["John","d@m.com"]
]
n = 4
```

---

### Phase 1 — Build DSU via shared emails

**Account 0 (John):**
```
email "a@m.com": not seen → emailsToId["a@m.com"] = 0
email "b@m.com": not seen → emailsToId["b@m.com"] = 0
```

**Account 1 (John):**
```
email "c@m.com": not seen → emailsToId["c@m.com"] = 1
```

**Account 2 (Mary):**
```
email "b@m.com": SEEN! emailsToId["b@m.com"] = 0
  → unionBySize(2, 0): merge account 2 and account 0
  → parent[2]=0, size[0]=2

DSU: {0,2} {1} {3}
emailsToId["b@m.com"] still = 0 (not updated, stays as first owner)
```

**Account 3 (John):**
```
email "d@m.com": not seen → emailsToId["d@m.com"] = 3
```

```
Final emailsToId: {"a@m.com":0, "b@m.com":0, "c@m.com":1, "d@m.com":3}
DSU components: {0,2} {1} {3}
```

---

### Phase 2 — Assign emails to component roots

```
For each email in emailsToId:

"a@m.com" → account 0 → findUParent(0) = 0 → mergedMails[0].push("a@m.com")
"b@m.com" → account 0 → findUParent(0) = 0 → mergedMails[0].push("b@m.com")
"c@m.com" → account 1 → findUParent(1) = 1 → mergedMails[1].push("c@m.com")
"d@m.com" → account 3 → findUParent(3) = 3 → mergedMails[3].push("d@m.com")

mergedMails:
  [0]: ["a@m.com", "b@m.com"]   ← root of {account 0, account 2}
  [1]: ["c@m.com"]              ← root of {account 1}
  [2]: []                       ← not a root (merged into 0)
  [3]: ["d@m.com"]              ← root of {account 3}
```

---

### Phase 3 — Build answer

```
i=0: mergedMails[0] not empty
  sort → ["a@m.com","b@m.com"]
  name = accounts[0][0] = "John"
  → ["John","a@m.com","b@m.com"]

i=1: mergedMails[1] not empty
  sort → ["c@m.com"]
  name = accounts[1][0] = "John"
  → ["John","c@m.com"]

i=2: mergedMails[2] empty → skip

i=3: mergedMails[3] not empty
  sort → ["d@m.com"]
  name = accounts[3][0] = "John"
  → ["John","d@m.com"]

Answer: [
  ["John","a@m.com","b@m.com"],   ← John + Mary's accounts merged!
  ["John","c@m.com"],
  ["John","d@m.com"]
]
```

Note: "Mary" is account 2 — it merged INTO account 0. The name comes from `accounts[0][0] = "John"`, which is correct since both John (account 0) and Mary (account 2) contributed `b@m.com`. Wait — this reveals the name collision issue.

> **Important:** This problem guarantees that if two accounts share an email, they belong to the SAME person with the SAME name. "Mary" sharing an email with "John" won't happen in valid input. The problem statement guarantees name consistency within merged groups.

---

## 4. Story Points

---

**Story Point 1 — "Accounts are nodes, emails are bridges"**

The mental model: think of accounts as islands and emails as bridges connecting them. When two islands (accounts) share a bridge (email), they're really the same land mass (person). DSU's job: find all land masses after all bridges are known.

---

**Story Point 2 — "`emailsToId` serves double duty"**

During Phase 1: "Have I seen this email before? If yes, which account owned it → union those accounts."

During Phase 2: "For each email, which account originally owned it? → find that account's root → bucket the email there."

The same map handles both detection and assignment. No need for a separate data structure.

---

**Story Point 3 — "Why `mergedMails[i]` is empty for non-root accounts"**

In Phase 2, emails are placed in `mergedMails[root]` where `root = findUParent(account_that_owns_email)`. Non-root accounts (those merged INTO another) have `findUParent(i) != i`, so no email ever lands in their bucket. The `mergedMails[i].size() == 0` check in Phase 3 elegantly skips these merged accounts.

---

**Story Point 4 — "Sort is applied to emails within each merged group"**

The problem requires emails in sorted order. Sorting `mergedMails[i]` before building the output achieves this. The name (`accounts[i][0]`) is prepended AFTER sorting so it stays at position 0.

---

**Story Point 5 — "The person's name comes from the ROOT account"**

When multiple accounts merge, we use `accounts[i][0]` where `i` is the DSU root. The problem guarantees all merged accounts have the same person's name, so it doesn't matter which account's name we use — they're all the same.

---

## 5. Code

```cpp
class Solution {
public:
    vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
        int n = accounts.size();

        // email → account index that first registered this email
        unordered_map<string, int> emailsToId;

        DisjointSet ds(n);   // n DSU nodes, one per account

        // Phase 1: Connect accounts that share emails
        for(int i = 0; i < n; i++) {
            for(int j = 1; j < (int)accounts[i].size(); j++) {
                string mail = accounts[i][j];

                if(emailsToId.find(mail) == emailsToId.end()) {
                    // First time seeing this email → register to account i
                    emailsToId[mail] = i;
                } else {
                    // Email already seen in account j → same person → merge
                    ds.unionBySize(i, emailsToId[mail]);
                }
            }
        }

        // Phase 2: Assign each email to its component's root bucket
        vector<vector<string>> mergedMails(n);

        for(auto& it : emailsToId) {
            string mail = it.first;
            int root    = ds.findUParent(it.second);   // find root of this email's account
            mergedMails[root].push_back(mail);
        }

        // Phase 3: Build answer — only for non-empty buckets (roots of components)
        vector<vector<string>> ans;

        for(int i = 0; i < n; i++) {
            if(mergedMails[i].empty()) continue;   // not a root account → skip

            sort(mergedMails[i].begin(), mergedMails[i].end());   // sort emails

            vector<string> temp;
            temp.push_back(accounts[i][0]);   // add person's name first

            for(auto& mail : mergedMails[i])
                temp.push_back(mail);

            ans.push_back(temp);
        }

        return ans;
    }
};
```

---

## 6. Complexity Analysis

### Time Complexity — `O(N × M × α(N) + E log E)`

Where `N` = number of accounts, `M` = average emails per account, `E` = total unique emails.

| Step | Cost | Reason |
|---|---|---|
| Phase 1: Process all emails | `O(N × M × α(N))` | N×M emails, each `find`/`union` = `O(α(N))` |
| Phase 2: Assign emails to roots | `O(E × α(N))` | E emails, each `findUParent` = `O(α(N))` |
| Phase 3: Sort each bucket | `O(E log E)` | Sorting all emails across all buckets |
| Map operations | `O(N×M)` avg | Unordered map insert/lookup |

**Total: `O(N × M × α(N) + E log E)` ≈ `O(E log E)`**

> The sort dominates since `E log E` > `E × α(E)`.

---

### Space Complexity — `O(N + E)`

| Structure | Size | Reason |
|---|---|---|
| DSU `parent[]` + `size[]` | `O(N)` | One entry per account |
| `emailsToId` map | `O(E)` | One entry per unique email |
| `mergedMails[]` | `O(E)` | Stores all emails across all buckets |

**Total: `O(N + E)`**
