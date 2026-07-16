# Lesson 3

## Top-Down vs Bottom-Up — The Only Two Directions
Two Kinds of Questions

Every tree problem asks you to move information through the tree. And information can only flow in two directions: down from parent to child, or up from child to parent.

Top-down passes information from root toward leaves; bottom-up collects answers from leaves toward root.

That's it. There is no third option. Once you learn to identify which direction a problem needs, the recursive structure writes itself.

## Top-Down: Passing Information Downward
Top-down means the parent tells the child something. The child uses that information, updates it, and passes it to its own children. The flow is root → leaves.

This is preorder processing. You do work at the current node before recursing into children, because the children need the result of that work.

**Example question:** "Does any root-to-leaf path in this tree sum to 22?"

Think about what a node needs to answer this. Node 11 can't answer on its own — it doesn't know what the nodes above it summed to. The parent has to pass that running sum down.

Here's our tree:

```text
        5
       / \
      4   8
     /
    11
   /  \
  7    2
```

The path 5 → 4 → 11 → 2 sums to 5 + 4 + 11 + 2 = 22. That's a match.

The algorithm: at each node, pass the remaining sum to the children. Start with targetSum = 22. At node 5, subtract 5 — pass 17 down. At node 4, subtract 4 — pass 13 down. At node 11, subtract 11 — pass 2 down. At node 2, subtract 2 — remaining is 0. We've hit a leaf with remaining sum 0. Found it.

| Node | Remaining Sum Received | After Subtracting | Leaf? | Match? |
|---|---|---|---|---|
| 5 | 22 | 17 | No | — |
| 4 | 17 | 13 | No | — |
| 11 | 13 | 2 | No | — |
| 7 | 2 | -5 | Yes | No |
| 2 | 2 | 0 | Yes | Yes |
| 8 | 17 | 9 | Yes | No |

The information flows downward. Each node receives something from its parent, processes it, and hands the updated version to its children.

```cpp
bool hasPathSum(Node* root, int remainingSum) {
    if (!root) return false;
    remainingSum -= root->value;
    if (!root->left && !root->right) return remainingSum == 0;
    return hasPathSum(root->left, remainingSum) || hasPathSum(root->right, remainingSum);
}
```

Notice the structure: the work happens before the recursive calls. We update remainingSum, then pass it down. That's the top-down signature.

## Bottom-Up: Collecting Information Upward
Bottom-up means the children report results to the parent. The parent combines those results with its own value to produce its answer. The flow is leaves → root.

This is postorder processing. You recurse into children first, then do work at the current node, because the current node needs the children's answers.

**Example question:** "What is the height of this tree?"

A node can't know its height without knowing the height of its subtrees. It has to wait for its children to report back.

Same tree:

```text
        5
       / \
      4   8
     /
    11
   /  \
  7    2
```

| Call | Node | Left returns | Right returns | Result |
|---|---|---|---|---|
| height(7) | 7 | -1 | -1 | 0 |
| height(2) | 2 | -1 | -1 | 0 |
| height(11) | 11 | 0 (from 7) | 0 (from 2) | 1 |
| height(4) | 4 | 1 (from 11) | -1 (nullptr) | 2 |
| height(8) | 8 | -1 | -1 | 0 |
| height(5) | 5 | 2 (from 4) | 0 (from 8) | 3 |

The information flows upward. Each node waits for its children, then combines.

```cpp
int height(Node* root) {
    if (!root) return -1;
    int leftHeight = height(root->left);
    int rightHeight = height(root->right);
    return 1 + max(leftHeight, rightHeight);
}
```

Notice the structure: the recursive calls happen before the work. We get leftHeight and rightHeight first, then compute the answer. That's the bottom-up signature.

## The Same Problem, Both Ways
To really see the difference, solve the same problem using both approaches.

**Problem:** Does a tree contain a node with value `target`?

**Top-down approach:** Pass the target down. At each node, check if the current value matches. If yes, return true. Otherwise, keep passing the target to children.

```cpp
bool containsTopDown(Node* root, int target) {
    if (!root) return false;
    if (root->value == target) return true;
    return containsTopDown(root->left, target) || containsTopDown(root->right, target);
}
```

**Bottom-up approach:** Ask each subtree "do you contain the target?" Collect the boolean results upward.

```cpp
bool containsBottomUp(Node* root, int target) {
    if (!root) return false;
    bool foundInLeft = containsBottomUp(root->left, target);
    bool foundInRight = containsBottomUp(root->right, target);
    return root->value == target || foundInLeft || foundInRight;
}
```

Both work. Both visit every node in the worst case. But notice the difference in when each node does its work:

- **Top-down** checks the current node first, then recurses. If it finds the target early (near the root), it can short-circuit.
- **Bottom-up** recurses first, then checks the current node. It always visits the entire subtree before deciding.

For this particular problem, top-down is more natural. But for problems like height, diameter, or subtree sums — where you need information from below — bottom-up is the only option.

If you need information from above (parent, ancestors, running totals), it's top-down. If you need information from below (subtree sizes, heights, existence checks), it's bottom-up.

## The Classification Skill
Recognizing which direction you need is THE skill for tree problems. It determines the shape of your recursion before you write a single line of code.

Here are six problems. For each one, decide: top-down, bottom-up, or both?

**1. "Return the maximum value in the tree."**
You need to know the max of the left subtree and the max of the right subtree to compute the max of the whole tree. Information flows up. **Bottom-up.**

**2. "Print all root-to-leaf paths."**
You need to know the path from the root to the current node, which means you need information from above. Pass the current path down as a parameter. **Top-down.**

**3. "Return the diameter of the tree (longest path between any two nodes)."**
The diameter through a node is `leftHeight + rightHeight + 2`. You need heights from below (bottom-up). But you also need to track the global maximum as you go — which you can do with a reference variable passed downward or maintained across calls. **Primarily bottom-up.**

**4. "Check if the tree is symmetric."**
You compare left and right subtrees by mirroring. The comparison results bubble up from children to parents. **Bottom-up.**

**5. "Given a path sum target, return all root-to-leaf paths that sum to it."**
You need the running sum from above (top-down to pass the remaining sum). You also need to know when you've hit a leaf (bottom-up isn't needed — you check at the leaf). **Top-down.**

**6. "Find the lowest common ancestor of two nodes."**
You need to know whether the target nodes exist in the left or right subtree. That information comes from below. **Bottom-up.**

## The Combined Pattern
Some problems need both directions. Here's the template:

```cpp
int solve(Node* root, int infoFromAbove, int& globalAnswer) {
    if (!root) return baseValue;

    int leftResult = solve(root->left, updatedInfo, globalAnswer);
    int rightResult = solve(root->right, updatedInfo, globalAnswer);

    int localAnswer = combine(leftResult, rightResult, root->value);
    globalAnswer = max(globalAnswer, localAnswer);

    return localAnswer;
}
```

The `infoFromAbove` parameter carries top-down information. The return value carries bottom-up information. The `globalAnswer` reference collects the overall result. You'll see this exact pattern in diameter, max path sum, and a dozen other problems later in the course.

## The Code: Path Sum (Top-Down)
The full solution for LC 112 — the top-down example from above:

```cpp
bool hasPathSum(Node* root, int remainingSum) {
    if (!root) return false;
    remainingSum -= root->value;
    if (!root->left && !root->right) return remainingSum == 0;
    return hasPathSum(root->left, remainingSum) || hasPathSum(root->right, remainingSum);
}
```

Complexity: O(n) time, O(h) space — where h is the height of the tree (the call stack depth).

## Try It
The starter code has `hasPathSum` with a TODO. Fill in the body.

```text
Input: tree = [5,4,8,11,null,null,null,7,2], targetSum = 22
Output: 1 (true)
```

**Predict before running:** What does `hasPathSum` return for the path 5 → 8? Node 8 is a leaf. Remaining sum after subtracting: 22 − 5 − 8 = 9. Not zero. So this path fails.

**Challenge — Bottom-Up Variant:** Write a function `allPathSums(Node* root)` that returns a vector of all root-to-leaf path sums. Is this top-down or bottom-up? *(Hint: you need the running sum from above, so you pass it down as a parameter. It's top-down, even though you collect results into a vector.)*

**Challenge 2:** Write `binaryTreePaths(Node* root)` that returns all root-to-leaf paths as strings like `"5->4->11->2"`. What direction does the information flow? What do you pass as a parameter?

## Edge Cases to Watch For
- **Target sum of zero with non-leaf nodes:** A non-leaf node with value 0 should NOT trigger a match. The path must end at a leaf. Always check `!root->left && !root->right` before declaring a match.
- **Negative node values:** The remaining sum can go negative and then positive again. Do not prune early based on the remaining sum being negative — it might recover.
- **Collecting all matching paths (LC 113):** The `||` short-circuit in `hasPathSum` stops after finding the first match. When collecting ALL matching paths, you must explore both subtrees unconditionally — use separate recursive calls instead of `||`.

## Problems

| Problem | Link | Difficulty |
|---|---|---|
| LC 112 — Path Sum | [leetcode.com/problems/path-sum](https://leetcode.com/problems/path-sum) | Easy |
| LC 257 — Binary Tree Paths | [leetcode.com/problems/binary-tree-paths](https://leetcode.com/problems/binary-tree-paths) | Easy |
