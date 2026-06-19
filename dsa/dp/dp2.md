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

### 8. Minimum Path Sum in a Grid (DP-10)
**Problem**: Find the path from `(0,0)` to `(m-1, n-1)` that minimizes the sum of all numbers along its path. Allowed moves: Down or Right.

**The "Why" (Intuition)**:
This is a direct evolution of the "Unique Paths" problem. Instead of counting ways, we are accumulating cost.

- **State**: `f(i, j)` represents the minimum cost to reach `(i, j)` from `(0, 0)`.
- **Base Cases**: 
  - `if (i == 0 && j == 0) return grid[0][0];` (Cost of the starting cell).
  - `if (i < 0 || j < 0) return INT_MAX;` (Out of bounds? Make the cost infinitely high so the `min()` function ignores this path).

**Optimal Approach (Space Optimization)**
Just like Unique Paths, to calculate the current row `cur`, you only need the previous row `prev` (for the "Up" move) and the element just computed in the current row `cur[j-1]` (for the "Left" move).

```cpp
int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size();
    int n = grid[0].size();
    vector<int> prev(n, 0); // Represents the row above
    
    for(int i = 0; i < m; i++) {
        vector<int> cur(n, 0); 
        for(int j = 0; j < n; j++) {
            if(i == 0 && j == 0) {
                cur[j] = grid[i][j];
                continue;
            }
            int up = grid[i][j], left = grid[i][j];
            
            // Add previous path costs if valid, else add massive value to disqualify
            if(i > 0) up += prev[j];
            else up += 1e9; // 1e9 prevents integer overflow vs INT_MAX
            
            if(j > 0) left += cur[j-1];
            else left += 1e9;
            
            cur[j] = min(up, left);
        }
        prev = cur;
    }
    return prev[n-1];
}
```
* **Time Complexity**: `O(M * N)`
* **Space Complexity**: `O(N)`

### 9. Minimum Path Sum in a Triangle (Variable Start, Fixed/Variable End - DP-11)
**Problem**: Given a triangle array, find the minimum path sum from top to bottom. From index `j` on row `i`, you can move to `j` or `j+1` on row `i+1`.

**The "Why" (Intuition)**:
Your notes highlight a crucial detail: Fixed starting point, variable ending points. If we start at `(0,0)` and move down, we have to check every single element in the last row to find the minimum.
**The Trick**: Calculate it from the bottom up! If `f(i, j)` represents the min cost from `(i, j)` to the bottom, the answer will simply rest at `f(0,0)`.

- **State Transitions**: From `(i, j)`, you can go straight down to `(i+1, j)` or diagonally down to `(i+1, j+1)`.
- **Base Case**: The cost to go from the last row to the bottom is just the value of the elements in the last row.

**Optimal Approach (Space Optimization)**
Since row `i` only needs data from row `i+1`, we can optimize the `O(N^2)` table into a single 1D array representing the row "below" us (`front`).

```cpp
int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    // Base Case: The bottom row itself is the starting 'front' array
    vector<int> front = triangle[n-1]; 
    
    // Start from the second to last row and move UP to row 0
    for(int i = n - 2; i >= 0; i--) {
        vector<int> cur(i + 1, 0); // Row i has i+1 elements
        for(int j = i; j >= 0; j--) {
            int down = triangle[i][j] + front[j];
            int diagonal = triangle[i][j] + front[j+1];
            
            cur[j] = min(down, diagonal);
        }
        front = cur; // Current row becomes the 'front' for the row above it
    }
    return front[0]; // The answer bubbles up to the absolute top
}
```
* **Time Complexity**: `O(N^2)` (where `N` is the number of rows)
* **Space Complexity**: `O(N)`

### 10. Maximum Path Sum in a Matrix (Variable Start, Variable End)
**Problem**: Start anywhere in the first row, end anywhere in the last row. Moves: Down-Left, Down, Down-Right. Maximize the sum.

**The "Why" (Intuition)**:
Since the start and end are variable, we must run our DP logic from every single cell in the first row and find the global maximum.

- **State**: `f(i, j)` is the max sum to reach the bottom starting from `(i, j)`.
- **Transitions**: `a[i][j] + max(f(i+1, j-1), f(i+1, j), f(i+1, j+1))`

**Optimal Approach (Space Optimization)**
We compute from the bottom row up to the first row. We only need the previous row to calculate the current row.

```cpp
int getMaxPathSum(vector<vector<int>> &matrix) {
    int n = matrix.size();
    int m = matrix[0].size();
    
    // Base Case: Bottom row
    vector<int> prev(m, 0);
    for(int j = 0; j < m; j++) prev[j] = matrix[n-1][j];
    
    for(int i = n - 2; i >= 0; i--) {
        vector<int> cur(m, 0);
        for(int j = 0; j < m; j++) {
            int up = matrix[i][j] + prev[j];
            int leftDiagonal = matrix[i][j] + (j - 1 >= 0 ? prev[j-1] : -1e9);
            int rightDiagonal = matrix[i][j] + (j + 1 < m ? prev[j+1] : -1e9);
            
            cur[j] = max(up, max(leftDiagonal, rightDiagonal));
        }
        prev = cur;
    }
    
    // The answer is the maximum value in the top row (which is now stored in prev)
    int maxi = -1e9;
    for(int j = 0; j < m; j++) {
        maxi = max(maxi, prev[j]);
    }
    return maxi;
}
```
* **Time Complexity**: `O(N * M)`
* **Space Complexity**: `O(M)`

### 11. Cherry Pickup II (3D DP - The Alice & Bob Problem)
**Problem**: Two agents start at top-left and top-right. They move down simultaneously (Down, Down-Left, Down-Right). Maximize grid items collected. If they land on the same cell, the item is collected only once.

**The "Why" (Intuition)**:
This is a masterpiece of state reduction. Your notes nail it: "We can omit `i_2` since they start at some row & allowed movements ensure same row". Because they move down simultaneously, `i_1` always equals `i_2`.

- **State**: `f(i, j_1, j_2)`. Both are on row `i`. Alice is at column `j_1`, Bob is at `j_2`.
- **Exploration**: For every 1 move Alice makes (3 options), Bob can make 3 moves. This results in `3 * 3 = 9` combinations of paths at every single step.

**Optimal Approach (Space Optimization)**
A standard tabulation requires a 3D table `dp[N][M][M]`. However, to compute row `i`, we only need the 2D grid of states from row `i+1`. We can reduce this to two 2D arrays: `front[M][M]` and `cur[M][M]`.

```cpp
int cherryPickup(vector<vector<int>>& grid) {
    int n = grid.size();
    int m = grid[0].size();
    
    // front[j1][j2] represents the max cherries picked from row i+1 to the bottom
    vector<vector<int>> front(m, vector<int>(m, 0));
    vector<vector<int>> cur(m, vector<int>(m, 0));
    
    // Base Case: Last Row (i == n-1)
    for(int j1 = 0; j1 < m; j1++) {
        for(int j2 = 0; j2 < m; j2++) {
            if(j1 == j2) front[j1][j2] = grid[n-1][j1];
            else front[j1][j2] = grid[n-1][j1] + grid[n-1][j2];
        }
    }
    
    // Tabulation from n-2 up to 0
    for(int i = n - 2; i >= 0; i--) {
        for(int j1 = 0; j1 < m; j1++) {
            for(int j2 = 0; j2 < m; j2++) {
                int maxi = -1e9;
                // Explore all 9 path combinations
                for(int d1 = -1; d1 <= 1; d1++) {
                    for(int d2 = -1; d2 <= 1; d2++) {
                        int val = 0;
                        if(j1 == j2) val = grid[i][j1]; // Landed on same cell
                        else val = grid[i][j1] + grid[i][j2];
                        
                        // Check bounds for the NEXT move
                        if(j1 + d1 >= 0 && j1 + d1 < m && j2 + d2 >= 0 && j2 + d2 < m) {
                            val += front[j1 + d1][j2 + d2];
                        } else {
                            val += -1e9; // Invalid path
                        }
                        maxi = max(maxi, val);
                    }
                }
                cur[j1][j2] = maxi;
            }
        }
        front = cur;
    }
    return front[0][m-1]; // Alice started at (0,0), Bob started at (0, m-1)
}
```
* **Time Complexity**: `O(N * M * M * 9)`
* **Space Complexity**: `O(M^2)` *(Reduced from `O(N * M^2)`)*

---

## Part 4: DP on Subsequences / Subsets

### 12. Subset Sum Equals K
**Problem**: Given an array and a target `K`, determine if there exists a subsequence (subset) that sums exactly to `K`.

**The "Why" (Intuition)**:
"Subsequence" triggers the "Pick / Not Pick" instinct.

- **State**: `f(ind, target)`. Can we form `target` using elements from index `0` to `ind`?
- **Transitions**: 
  - **Not Pick**: We don't take `arr[ind]`. Does the target exist in the rest of the array? `f(ind-1, target)`.
  - **Pick**: We take `arr[ind]` (only if `target >= arr[ind]`). Does `target - arr[ind]` exist in the rest of the array? `f(ind-1, target - arr[ind])`.

**Optimal Approach (Space Optimization)**
If you look at the recurrence relation, `dp[ind][target]` only requires values from the previous row `dp[ind-1]`. Therefore, we only need a 1D boolean array of size `target + 1`.

```cpp
bool isSubsetSum(vector<int>& arr, int target) {
    int n = arr.size();
    vector<bool> prev(target + 1, false);
    
    // Base Case 1: If target is 0, empty subset is always possible
    prev[0] = true;
    
    // Base Case 2: At index 0, we can only form the target if arr[0] == target
    if(arr[0] <= target) {
        prev[arr[0]] = true;
    }
    
    for(int ind = 1; ind < n; ind++) {
        vector<bool> cur(target + 1, false);
        cur[0] = true; // Base case for the current row
        
        for(int t = 1; t <= target; t++) {
            bool notTake = prev[t];
            bool take = false;
            if(arr[ind] <= t) {
                take = prev[t - arr[ind]];
            }
            cur[t] = take | notTake; // If either is true, it's possible
        }
        prev = cur;
    }
    return prev[target];
}
```
* **Time Complexity**: `O(N * target)`
* **Space Complexity**: `O(target)` *(Massive optimization over `O(N * target)` table)*
