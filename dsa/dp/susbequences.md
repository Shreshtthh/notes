# DP on Subsequences
### 1. Subset Sum Equal to Target (DP-14)
**Problem**: Given an array `ARR` with `N` positive integers, find if there is a subset (subsequence) with a sum equal to `target`. 
**The "Why" (Intuition)**:
We cannot use a greedy approach because we are not optimizing anything; we need to explore all valid subsequences to find an exact match.
The core logic relies on the "Pick / Not Pick" technique.
- **State**: `f(ind, target)` means: Is there a subset in the array from index `0` to `ind` whose sum equals `target`?
- **Transitions**: 
  - **Not Pick**: Exclude the current element. Does the target exist in the rest of the array? `f(ind-1, target)`.
  - **Pick**: Include the current element (only if `arr[ind] <= target`). Does `target - arr[ind]` exist in the rest of the array? `f(ind-1, target - arr[ind])`.
- **Base Cases**: 
  - `if (target == 0) return true;` (We successfully found a subset).
  - `if (ind == 0) return arr[0] == target;` (At the last element, it must exactly equal the remaining target).
**Optimal Approach (Space Optimization)**
To calculate a value of a cell in the DP array, we only need values from the previous row. Therefore, we don't need to store an entire `O(N * target)` 2D array. A single 1D array of size `target + 1` representing the previous row is sufficient.
```cpp
bool subsetSumToK(int n, int k, vector<int> &arr) {
    // Initialize a vector 'prev' to represent the previous row of the DP table
    vector<bool> prev(k + 1, false);
    // Base case: sum 0 can always be formed by an empty subset
    prev[0] = true;
    // Base case: if the first element <= k, mark true
    if (arr[0] <= k) {
        prev[arr[0]] = true;
    }
    // Iterate over all elements from second to last
    for (int ind = 1; ind < n; ind++) {
        vector<bool> cur(k + 1, false);
        cur[0] = true; // sum 0 is always possible
        for (int target = 1; target <= k; target++) {
            bool notTaken = prev[target]; // skip current element
            bool taken = false;
            if (arr[ind] <= target) {
                taken = prev[target - arr[ind]]; // take current element
            }
            cur[target] = notTaken || taken; // store result for current target
        }
        prev = cur; // move to next iteration
    }
    // Final answer: can we form sum k using all elements?
    return prev[k];
}
```
* **Time Complexity**: `O(N * K)`
* **Space Complexity**: `O(K)`
### 2. Partition Equal Subset Sum (DP-15)
**Problem**: Given an array `arr` of `n` integers, return true if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal, else return false.
**The "Why" (Intuition)**:
This is a direct application of the "Subset Sum Equal to Target" (DP-14). 
If the array can be partitioned into two subsets of equal sum, then each subset must have a sum exactly equal to `Total Sum / 2`. 
- If the total sum of the array is odd, we can never divide it equally. Return `false`.
- If it's even, the problem simply reduces to: *Find if there is a subset with target sum = `Total Sum / 2`*.
**Optimal Approach (Space Optimization)**
We completely reuse the space-optimized logic from DP-14, just replacing the target `K` with `Total Sum / 2`.
```cpp
bool canPartition(int n, vector<int>& arr) {
    int totSum = 0;
    // Calculate the total sum of the array
    for (int i = 0; i < n; i++) {
        totSum += arr[i];
    }
    // If the total sum is odd, it cannot be partitioned into two equal subsets
    if (totSum % 2 == 1)
        return false;
        
    int k = totSum / 2;
    // Create a vector to represent the previous row of the DP table
    vector<bool> prev(k + 1, false);
    // Base case: target sum 0 is achievable
    prev[0] = true;
    // Initialize the first row based on the first element
    if (arr[0] <= k)
        prev[arr[0]] = true;
    // Fill in the DP table using bottom-up approach
    for (int ind = 1; ind < n; ind++) {
        vector<bool> cur(k + 1, false);
        cur[0] = true;
        for (int target = 1; target <= k; target++) {
            bool notTaken = prev[target];
            bool taken = false;
            if (arr[ind] <= target)
                taken = prev[target - arr[ind]];
            cur[target] = notTaken || taken;
        }
        prev = cur;
    }
    return prev[k];
}
```
* **Time Complexity**: `O(N * K)` (where `K` is `Total Sum / 2`)
* **Space Complexity**: `O(K)`
### 3. Partition Set Into 2 Subsets With Min Absolute Sum Diff (DP-16)
**Problem**: Given an array of `n` integers, partition the array into two subsets such that the absolute difference between their sums is minimized.
**The "Why" (Intuition)**:
In DP-14 and DP-15, our DP table told us whether a specific subset sum was possible or not.
If we run the subset sum DP logic for a target equal to the `Total Sum`, the very last row of our DP table (`prev` array in space-optimized version) acts as a truth table!
`prev[i] == true` means that a subset with sum `i` is possible.
If we know one subset has sum `S1`, the other subset automatically has sum `S2 = Total Sum - S1`.
The absolute difference is `|S1 - S2|` which simplifies to `|S1 - (Total Sum - S1)|`.
**Algorithm**:
1. Calculate `Total Sum`.
2. Run the standard space-optimized subset sum DP (DP-14) up to a target of `Total Sum`.
3. Loop through all possible sums `S1` from `0` to `Total Sum / 2` (to avoid double checking).
4. If `prev[S1]` is true, calculate the difference `abs(S1 - (Total Sum - S1))` and update the minimum difference.
**Optimal Approach (Space Optimization)**
```cpp
int minSubsetSumDifference(vector<int>& arr, int n) {
    int totSum = 0;
    for (int i = 0; i < n; i++) {
        totSum += arr[i];
    }
    // Initialize a boolean vector 'prev' to represent the previous row
    vector<bool> prev(totSum + 1, false);
    // Base case: sum 0 is always achievable
    prev[0] = true;
    if (arr[0] <= totSum)
        prev[arr[0]] = true;
    // Fill DP table
    for (int ind = 1; ind < n; ind++) {
        vector<bool> cur(totSum + 1, false);
        cur[0] = true;
        for (int target = 1; target <= totSum; target++) {
            bool notTaken = prev[target];
            bool taken = false;
            if (arr[ind] <= target)
                taken = prev[target - arr[ind]];
            cur[target] = notTaken || taken;
        }
        prev = cur;
    }
    // Find the minimum absolute difference
    int mini = INT_MAX;
    for (int i = 0; i <= totSum; i++) {
        if (prev[i]) {
            // Calculate absolute difference between two subset sums
            int diff = abs(i - (totSum - i));
            mini = min(mini, diff);
        }
    }
    return mini;
}
```
* **Time Complexity**: `O(N * Total Sum)`
* **Space Complexity**: `O(Total Sum)`

### 4. Count Subsets with Sum K (DP - 17)

**Problem Statement**: Given an array `arr` of `n` integers and an integer `K`, count the number of subsets of the given array that have a sum equal to `K`.

**Examples**
**Input**: arr = [1, 2, 2, 3], K = 3
**Output**: 3
**Explanation**: The subarrays [1,2], [1,2] and [3] have a sum of 3. 

**Input**: arr = [1, 2, 3, 4, 5], K = 5
**Output**: 3
**Explanation**: The subsets are [5], [2, 3], and [1, 4]. 

**Memoization**
**Algorithm**
The problem of counting subsets with a given sum K can be thought of as a decision process where, for each element in the array, we either include it in the subset or exclude it. If we include it, the target reduces by that element’s value and if we exclude it, the target remains the same. This naturally forms a recursion. However, many subproblems repeat, such as computing the number of ways to reach the same target with the same index again and again.

To avoid redundant work, we use memoization, where each state defined by the pair (index, target) is stored after being computed once. This way, instead of exploring an exponential number of recursive paths, we reuse previously computed results, bringing the complexity down to polynomial time.
1. Define a recursive function that takes the current index and the remaining target sum as parameters
2. If the target becomes zero, return 1 because one valid subset is found
3. If the index is zero, check if the first element equals the target, and return 1 if true, else 0
4. Before solving, check if the result for this state is already stored in a dp table, and if yes, return it
5. Recursively call the function by excluding the current element, keeping the target unchanged
6. If the current element is less than or equal to the target, also call the function by including it and reducing the target
7. Add the results of including and excluding to get the total number of subsets for this state
8. Store the result in the dp table and return it

**Code**
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    // Function to count subsets that sum up to target
    int countSubsets(vector<int>& nums, int target) {
        // Initialize dp table with -1 (uncomputed states)
        vector<vector<int>> dp(nums.size(), vector<int>(target + 1, -1));
        return solve(nums.size() - 1, target, nums, dp);
    }

private:
    // Recursive helper with memoization
    int solve(int index, int target, vector<int>& nums, vector<vector<int>>& dp) {
        // Base case: if target is 0, we found a valid subset
        if (target == 0) return 1;

        // Base case: if we are at index 0, check if nums[0] equals target
        if (index == 0) return (nums[0] == target ? 1 : 0);

        // If already computed, return from dp
        if (dp[index][target] != -1) return dp[index][target];

        // Case 1: Exclude current element
        int notTake = solve(index - 1, target, nums, dp);

        // Case 2: Include current element (if it is not greater than target)
        int take = 0;
        if (nums[index] <= target) {
            take = solve(index - 1, target - nums[index], nums, dp);
        }

        // Store result in dp and return
        return dp[index][target] = take + notTake;
    }
};

int main() {
    vector<int> nums = {1, 2, 3, 3};
    int target = 6;
    Solution obj;
    cout << obj.countSubsets(nums, target) << endl;
    return 0;
}
```
**Complexity Analysis**
* **Time Complexity**: O(N * K), each state defined by index and target is computed once.
* **Space Complexity**: O(N * K), extra space is used for the dp table and recursion stack.

**Tabulation**
**Algorithm**
The problem of counting subsets with a given sum K can be solved using tabulation by building the solution bottom-up. Instead of starting with recursion, we construct a dp table where each entry dp[i][t] represents the number of subsets that can be formed using the first i elements to achieve the sum t. We iteratively fill this table, considering for each element whether we include it or exclude it.

This way, we systematically compute the answer without recursion and avoid stack overhead, while still ensuring that overlapping subproblems are reused effectively.
1. Create a 2D dp table where dp[i][t] means number of subsets using first i elements to make sum t
2. Initialize dp[0][0] as 1 because with zero elements, only one subset (empty set) makes sum 0
3. If the first element is less than or equal to K, set dp[0][arr[0]] as 1 because a single element can form that sum
4. Iterate over all elements from index 1 to n-1
5. For each element, iterate over all target sums from 0 to K
6. For each dp[i][t], first take the value from dp[i-1][t] (excluding current element)
7. If current element is less than or equal to t, also add dp[i-1][t - arr[i]] (including current element)
8. At the end, dp[n-1][K] will hold the total number of subsets with sum equal to K

**Code**
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int countSubsets(vector<int>& arr, int K) {
        // Get number of elements
        int n = arr.size();

        // Create dp table with dimensions n x (K+1)
        vector<vector<int>> dp(n, vector<int>(K + 1, 0));

        // Base case: one subset (empty set) makes sum 0
        dp[0][0] = 1;

        // If first element is <= K, mark dp[0][arr[0]] as 1
        if (arr[0] <= K) dp[0][arr[0]] = 1;

        // Fill the table
        for (int i = 1; i < n; i++) {
            for (int target = 0; target <= K; target++) {
                // Exclude current element
                int notTake = dp[i - 1][target];

                // Include current element if possible
                int take = 0;
                if (arr[i] <= target) take = dp[i - 1][target - arr[i]];

                // Total ways
                dp[i][target] = notTake + take;
            }
        }

        // Final answer
        return dp[n - 1][K];
    }
};

int main() {
    Solution obj;
    vector<int> arr = {1, 2, 3, 3};
    int K = 6;
    cout << obj.countSubsets(arr, K) << endl;
    return 0;
}
```
**Complexity Analysis**
* **Time Complexity**: O(N * K), each state defined by index and target is computed once.
* **Space Complexity**: O(N * K), extra space is used for the dp table.

**Space Optimization**
**Algorithm**
The problem of counting subsets with a given sum K can also be solved with space optimization. Instead of storing results for all n rows, we notice that dp[i][t] only depends on the previous row. Thus instead of storing entire table, we can only store the previous row.
1. Create a 1D dp array of size (K + 1), where dp[t] represents number of subsets that sum to t
2. Initialize dp[0] = 1 because there’s always one subset (empty set) with sum 0
3. If the first element is less than or equal to K, set dp[arr[0]] = 1
4. Iterate through the array from index 1 to n-1
5. For each element, traverse target sums backwards from K down to arr[i]
6. Update dp[t] by adding dp[t - arr[i]], meaning we include the current element
7. Backward traversal ensures we don’t overwrite values that are yet to be used
8. At the end, dp[K] will hold the total number of subsets that sum to K

**Code**
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int countSubsets(vector<int>& arr, int K) {
        // Create a dp array of size K+1 initialized to 0
        vector<int> dp(K + 1, 0);

        // Base case: One subset (empty set) makes sum 0
        dp[0] = 1;

        // If first element <= K, mark it as a subset
        if (arr[0] <= K) dp[arr[0]] += 1;

        // Loop through elements starting from index 1
        for (int i = 1; i < arr.size(); i++) {
            // Create a temporary dp array for current element
            vector<int> curr(K + 1, 0);

            // Empty set always makes sum 0
            curr[0] = 1;

            // Iterate over all possible targets
            for (int t = 0; t <= K; t++) {
                // Exclude current element
                int notTake = dp[t];

                // Include current element if possible
                int take = 0;
                if (arr[i] <= t) {
                    take = dp[t - arr[i]];
                }

                // Total ways = include + exclude
                curr[t] = take + notTake;
            }

            // Update dp for next iteration
            dp = curr;
        }

        // Return answer for sum K
        return dp[K];
    }
};

int main() {
    Solution sol;
    vector<int> arr = {1, 2, 3};
    int K = 4;
    cout << sol.countSubsets(arr, K) << endl; 
    return 0;
}
```
