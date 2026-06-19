# Dynamic Programming & Recursion Revision Guide
*A comprehensive, deep-dive revision guide engineered to achieve true technical mastery through deep conceptual understanding. Focuses on the "why" behind state transitions and aggressively optimizing space complexity.*
## The Universal Framework for DP & Recursion
Before jumping into specific patterns, always anchor back to this 3-step thought process for defining recursive DP:
1. **Represent the State**: Express the problem in terms of indices (e.g., `ind`, `row`, `col`).
2. **Explore all paths**: Write out all possible choices at that specific state (e.g., pick/not pick, jump 1/jump 2, go down/go right).
3. **Do the computation**: Aggregate the results of those paths (Take min, max, or sum of all paths).
---
## Part 1: 1D Dynamic Programming
### 1. Frog Jump (Minimum Energy)
**Problem**: A frog can jump 1 or 2 steps. Each jump costs `abs(a[ind] - a[ind - jump])`. Find the minimum energy to reach the last index from the 0th index.
- **Recurrence Relation**: `f(ind)` represents the minimum energy required to reach index `ind` from index `0`.
- **Base Case**: `if (ind == 0) return 0;` (Cost to reach the starting point is 0).
**Optimal Approach (Tabulation + Space Optimization)**
Notice that calculating the cost for step `i` only requires the costs from step `i-1` and step `i-2`. We do not need an entire `O(N)` DP array; two variables are sufficient.
```cpp
int frogJump(int n, vector<int>& heights) {
    int prev1 = 0; // Represents dp[i-1]
    int prev2 = 0; // Represents dp[i-2]
    
    for(int i = 1; i < n; i++) {
        int jumpTwo = INT_MAX;
        int jumpOne = prev1 + abs(heights[i] - heights[i-1]);
        if(i > 1) {
            jumpTwo = prev2 + abs(heights[i] - heights[i-2]);
        }
        
        int curi = min(jumpOne, jumpTwo);
        prev2 = prev1;
        prev1 = curi;
    }
    return prev1; 
}
```
* **Time Complexity**: `O(N)`
* **Space Complexity**: `O(1)`
### 2. Frog Jump with K-Steps
**Problem**: Same as above, but the frog can jump up to `K` steps.
- **Recurrence Relation**: Loop `j` from `1` to `K`. `cost = f(ind-j) + abs(a[ind] - a[ind-j])`. Take the minimum of all valid jumps.
- **Space Optimization Note**: Space optimization is technically possible by maintaining a rolling list/queue of size `K`. However, if `K` is close to `N`, the space complexity remains `O(N)`. For interviews, standard tabulation (an `O(N)` array) is expected here.
**Tabulation Code:**
```cpp
int frogJumpK(int n, int k, vector<int>& heights) {
    vector<int> dp(n, 0);
    dp[0] = 0;
    
    for(int i = 1; i < n; i++) {
        int minSteps = INT_MAX;
        for(int j = 1; j <= k; j++) {
            if(i - j >= 0) {
                int jump = dp[i-j] + abs(heights[i] - heights[i-j]);
                minSteps = min(minSteps, jump);
            }
        }
        dp[i] = minSteps;
    }
    return dp[n-1];
}
```
* **Time Complexity**: `O(N * K)`
* **Space Complexity**: `O(N)`
### 3. Maximum Sum of Non-Adjacent Elements (House Robber I)
**Problem**: Pick elements to maximize the sum, but you cannot pick any two adjacent elements.
- **Logic**: At any index, you have two choices: 
  - **Pick**: add `a[i]` and move to `i-2`
  - **Not Pick**: add `0` and move to `i-1`
- **Base Cases**: `if (ind == 0) return a[0];` and `if (ind < 0) return 0;`
**Optimal Approach (Space Optimized)**
Just like Frog Jump, `dp[i]` only relies on `dp[i-1]` (notPick) and `dp[i-2]` (pick).
```cpp
int maximumNonAdjacentSum(vector<int> &nums) {
    int n = nums.size();
    int prev1 = nums[0];
    int prev2 = 0;
    
    for(int i = 1; i < n; i++) {
        int pick = nums[i] + prev2;     // a[i] + f(ind-2)
        int notPick = 0 + prev1;        // 0 + f(ind-1)
        
        int curi = max(pick, notPick);
        prev2 = prev1;
        prev1 = curi;
    }
    return prev1;
}
```
### 4. House Robber II (Circular Array)
**Problem**: Same as above, but the houses are in a circle (first and last elements are adjacent).
- **Core Trick**: Since the array is circular, if you pick the first house (0), you cannot pick the last house (n-1), and vice versa.
- **Solution**: Abstract out the logic from House Robber I into a helper function. Call it twice:
  1. On the array excluding the last element: `helper(nums from 0 to n-2)`
  2. On the array excluding the first element: `helper(nums from 1 to n-1)`
- **Final Answer**: Return the maximum of these two results. *(Edge case: If N=1, just return nums[0])*
---
## Part 2: 2D DP / State Machine DP
### 5. Ninja's Training
**Problem**: A ninja can do 3 tasks a day. He cannot do the same task on two consecutive days. Maximize merit points over N days.
- **State**: `f(day, last_task)`. We need to track what we did yesterday to know what is valid today.
- **Base Case (Day 0)**: `dp[0][last_task]` should store the maximum points we can get on Day 0, given the `last_task` restriction from Day 1.
**Optimal Approach (Space Optimization)**
To calculate the states for day `i`, we only need the 4 states (tasks 0, 1, 2, and "none") from day `i-1`. We can reduce the `O(N * 4)` DP table to a 1D array of size 4.
```cpp
int ninjaTraining(int n, vector<vector<int>> &points) {
    vector<int> prev(4, 0);
    
    // Base Case initialization for Day 0
    prev[0] = max(points[0][1], points[0][2]);
    prev[1] = max(points[0][0], points[0][2]);
    prev[2] = max(points[0][0], points[0][1]);
    prev[3] = max(points[0][0], max(points[0][1], points[0][2]));
    
    // Iterate from day 1 to n-1
    for(int day = 1; day < n; day++) {
        vector<int> temp(4, 0); // Current day's states
        for(int last = 0; last < 4; last++) {
            for(int task = 0; task <= 2; task++) {
                if(task != last) {
                    temp[last] = max(temp[last], points[day][task] + prev[task]);
                }
            }
        }
        prev = temp; // Update for the next iteration
    }
    return prev[3]; // We start on the last day with 'no task' performed the day after.
}
```
* **Time Complexity**: `O(N * 4 * 3)`
* **Space Complexity**: `O(4) ≈ O(1)`
---
## Part 3: DP on Grids
### 6. Grid Unique Paths
**Problem**: Count total unique paths from `(0,0)` to `(m-1, n-1)`. Allowed moves: Right or Down.
- **Thought Process**: Moving Right and Down from `(0,0)` is mathematically identical to moving Up and Left from `(m-1, n-1)`. We define `f(i, j)` as the number of ways to reach `(0,0)` from `(i, j)`.
- **Base Cases**: 
  - `if(i == 0 && j == 0) return 1;` (Found a valid path).
  - `if(i < 0 || j < 0) return 0;` (Out of bounds).
**Optimal Approach (Space Optimization)**
In grid tabulation, the current row `i` only ever needs data from the previous row (`i-1`, the up move) and the current row's previous column (`j-1`, the left move). We only need a 1D array of size `N` (the columns).
```cpp
int uniquePaths(int m, int n) {
    vector<int> prev(n, 0); // Represents the row above
    
    for(int i = 0; i < m; i++) {
        vector<int> cur(n, 0); // Current row being calculated
        for(int j = 0; j < n; j++) {
            if(i == 0 && j == 0) {
                cur[j] = 1;
                continue;
            }
            int up = 0, left = 0;
            if(i > 0) up = prev[j];     // Grab from the row above
            if(j > 0) left = cur[j-1];  // Grab from the current row, just calculated
            
            cur[j] = up + left;
        }
        prev = cur; // Current row becomes the previous row for the next iteration
    }
    return prev[n-1];
}
```
* **Time Complexity**: `O(M * N)`
* **Space Complexity**: `O(N)` *(Massive reduction from O(M * N))*
### 7. Grid Unique Paths with Obstacles
**Problem**: Same as above, but if a cell contains an obstacle (e.g., `-1`), you cannot step on it.
- **Adjustment**: The core logic remains entirely identical to Problem 6. You just add one extra base-case check inside the loops to immediately zero out paths that hit an obstacle.
**Code modifications inside the loops:**
```cpp
// ... inside the nested for loops ...
if (maze[i][j] == -1) {
    cur[j] = 0; // Dead end.
    continue;
}
if(i == 0 && j == 0) {
    cur[j] = 1;
    continue;
}
// ... calculate up and left as usual ...
```
