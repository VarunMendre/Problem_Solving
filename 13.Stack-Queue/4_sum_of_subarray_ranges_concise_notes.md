# Sum of Subarray Ranges

---

## 1. Intuition / Approach

- The **range of a subarray** is defined as:
  
  ```
  max(subarray) − min(subarray)
  ```

- So, instead of calculating ranges directly for every subarray (which would be expensive), we split the problem into two independent parts:
  1. **Sum of maximum elements of all subarrays**
  2. **Sum of minimum elements of all subarrays**

- Final answer:

  ```
  Sum of Subarray Ranges = Sum of Subarray Maximums − Sum of Subarray Minimums
  ```

- For both minimums and maximums:
  - Treat **each element as a contributor**
  - Count how many subarrays consider it as:
    - the **minimum** (using NSE + PSEE)
    - the **maximum** (using NGE + PGEE)

- Monotonic stacks help efficiently find boundaries where an element remains the minimum or maximum.
- Each element contributes independently based on:

  ```
  contribution = value × (choices on left) × (choices on right)
  ```

---

## 2. Code

```cpp
class Solution {
private:
    /* Function to find the indices of
    next smaller elements */
    vector<int> findNSE(vector<int> &arr) {
        int n = arr.size();
        vector<int> ans(n);
        stack<int> st;
        
        for(int i = n - 1; i >= 0; i--) {
            int currEle = arr[i];
            while(!st.empty() && arr[st.top()] >= currEle){
                st.pop();
            }
            ans[i] = !st.empty() ? st.top() : n;
            st.push(i);
        }
        return ans;
    }
    
    /* Function to find the indices of
    next greater elements */
    vector<int> findNGE(vector<int> &arr) {
        int n = arr.size();
        vector<int> ans(n);
        stack<int> st;
        
        for(int i = n - 1; i >= 0; i--) {
            int currEle = arr[i];
            while(!st.empty() && arr[st.top()] <= currEle){
                st.pop();
            }
            ans[i] = !st.empty() ? st.top() : n;
            st.push(i);
        }
        return ans;
    }
    
    /* Function to find the indices of
    previous smaller or equal elements */
    vector<int> findPSEE(vector<int> &arr) {
        int n = arr.size();
        vector<int> ans(n);
        stack<int> st;
        
        for(int i=0; i < n; i++) {
            int currEle = arr[i];
            while(!st.empty() && arr[st.top()] > currEle){
                st.pop();
            }
            ans[i] = !st.empty() ? st.top() : -1;
            st.push(i);
        }
        return ans;
    }
    
    /* Function to find the indices of
    previous greater or equal elements */
    vector<int> findPGEE(vector<int> &arr) {
        int n = arr.size();
        vector<int> ans(n);
        stack<int> st;
        
        for(int i=0; i < n; i++) {
            int currEle = arr[i];
            while(!st.empty() && arr[st.top()] < currEle){
                st.pop();
            }
            ans[i] = !st.empty() ? st.top() : -1;
            st.push(i);
        }
        return ans;
    }
    
    /* Function to find the sum of the
    minimum value in each subarray */
    long long sumSubarrayMins(vector<int> &arr) {
        vector<int> nse = findNSE(arr);
        vector<int> psee = findPSEE(arr);
        int n = arr.size();
        long long sum = 0;
        
        for(int i=0; i < n; i++) {
            int left = i - psee[i];
            int right = nse[i] - i;
            long long freq = left*right*1LL;
            long long val = (freq*arr[i]*1LL);
            sum += val;
        }
        return sum;
    }
    
    /* Function to find the sum of the
    maximum value in each subarray */
    long long sumSubarrayMaxs(vector<int> &arr) {
        vector<int> nge = findNGE(arr);
        vector<int> pgee = findPGEE(arr);
        int n = arr.size();
        long long sum = 0;
        
        for(int i=0; i < n; i++) {
            int left = i - pgee[i];
            int right = nge[i] - i;
            long long freq = left*right*1LL;
            long long val = (freq*arr[i]*1LL);
            sum += val;
        }
        return sum;
    }
    
public:
    /* Function to find the sum of
    subarray ranges in each subarray */
    long long subArrayRanges(vector<int> &arr) {
        return ( sumSubarrayMaxs(arr) -
                 sumSubarrayMins(arr) );
    }
};
```

---

## 3. Time & Space Complexity

### Time Complexity
- Each helper function uses a monotonic stack.
- Every element is pushed and popped at most once.

```
TC = O(N) + O(N) + O(N) + O(N)
   = O(4N) ≈ O(N)
```

### Space Complexity
- Arrays for NSE, NGE, PSEE, PGEE → O(N)
- Stack usage → O(N)

```
SC = O(N)
```

---

