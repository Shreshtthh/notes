# Nodes vs Edges, Subtrees vs Paths

## The Most Common Bug Isn't in Your Code
It's in your reading. You misidentify what the problem is actually asking about, write a clean solution for the wrong abstraction, and wonder why it fails on test case 37.

Tree problems deal with four things. Only four. Every tree problem you'll ever see is about one of these:

- **Nodes** — count them, sum their values, find the max.
- **Edges** — the connections between nodes. A tree with n nodes always has exactly n−1 edges.
- **Subtrees** — everything below a given node, including the node itself.
- **Paths** — a sequence of connected nodes from one point to another.

The difference between subtree and path is where most people get confused. And that confusion is the difference between solving a problem in 10 minutes and staring at it for an hour.

## The Four Abstractions
Here's our tree for the rest of this lesson:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7
```

**Nodes:** The tree has 7 nodes. Node 2 has value 2. That's it — individual data points.

**Edges:** There are 6 edges (always n−1). Each node except the root has exactly one edge connecting it to its parent. Node 4 has an edge to 2. Node 2 has an edge to 1. This is why edge count = n−1: the root has no parent edge, every other node has exactly one.

**Subtree of node 2:** Everything below node 2, including 2 itself. That's {2, 4, 5, 7}. The subtree of 3 is {3, 6}. The subtree of 7 is just {7}. The subtree of the root is the entire tree.

**Path from 7 to 6:** 7 → 4 → 2 → 1 → 3 → 6. A path goes UP from one node, through the lowest common ancestor, and DOWN to the other. Paths can go through any part of the tree — they're not confined to a subtree.

Subtrees grow downward from a node. Paths travel through the tree between any two nodes.

*Subtree of node 2 highlighted vs the path from node 7 to node 6 passing through the LCA*

## Why This Distinction Matters
Consider two problems:

**Problem A:** "What is the sum of all values in the subtree rooted at node 2?"
You stand at node 2. You need everything below: nodes 2, 4, 5, 7. The answer is 2 + 4 + 5 + 7 = 18. This is a subtree problem — the answer depends entirely on what's BELOW node 2.

**Problem B:** "What is the longest path between any two nodes?"
Look at the tree. The longest path is 7 → 4 → 2 → 1 → 3 → 6, which has 5 edges. This path doesn't stay inside any one subtree — it goes UP from 7 through 2 and 1, then DOWN to 6. This is a path problem.

If you try to solve the diameter with a pure subtree approach — "find the biggest subtree and measure it" — you'll fail. The diameter path often crosses through an ancestor, spanning multiple subtrees.

## The Diagnostic Test
When you read a tree problem, ask yourself one question:

**Does the answer depend on what's BELOW this node (subtree), or on a route THROUGH this node (path)?**

- If it's "below" — you need postorder. Collect information from children, combine at the current node.
- If it's "through" — you still use postorder to collect child information, but the answer update happens by combining the left path and right path at each node.

## The Classification Exercise
Here's the same tree again:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7
```

For each problem, classify it as Node, Edge, Subtree, or Path:

| # | Problem Statement | Type | Why |
|---|---|---|---|
| 1 | "How many nodes are in the tree?" | Node | You're counting individual nodes. |
| 2 | "How many edges are in the tree?" | Edge | Always n−1. No traversal needed. |
| 3 | "What is the sum of all values below node 2?" | Subtree | "Below" = everything in the subtree. |
| 4 | "What is the longest path between any two nodes?" | Path | The path travels through ancestors. |
| 5 | "What is the difference between a node's left subtree sum and right subtree sum?" | Subtree | "Left subtree sum" and "right subtree sum" are both subtree queries. |

Problems 3 and 5 are pure postorder — collect from children, combine at the node.

Problem 4 is the tricky one. You use postorder to compute heights, but the diameter is updated by combining left height + right height at each node. The answer isn't returned up the recursion — it's tracked in a global variable.

## Example 1: Diameter of a Binary Tree (LC 543)
**Problem:** Given a binary tree, return the length of the diameter — the longest path between any two nodes. The length is measured by number of edges.

This sounds like it might be about nodes. "Longest path" sounds like it might involve BFS. But it's a path problem, and the key insight is:

At every node, the longest path THROUGH that node is `leftHeight + rightHeight`. The diameter is the maximum of this value across all nodes.

The path from the deepest leaf on the left to the deepest leaf on the right passes through the current node. If you compute this at every node and take the max, you have the diameter.

Trace on our tree:

| Node | leftHeight | rightHeight | pathThroughNode | diameterSoFar |
|---|---|---|---|---|
| 7 | 0 | 0 | 0 | 0 |
| 4 | 1 (from 7) | 0 | 1 | 1 |
| 5 | 0 | 0 | 0 | 1 |
| 2 | 2 (from 4) | 1 (from 5) | 3 | 3 |
| 6 | 0 | 0 | 0 | 3 |
| 3 | 0 | 1 (from 6) | 1 | 3 |
| 1 | 3 (from 2) | 2 (from 3) | 5 | 5 |

The diameter is 5 edges: 7 → 4 → 2 → 1 → 3 → 6.

Manual verification: Count the edges in that path. 7-4, 4-2, 2-1, 1-3, 3-6. That's 5 edges. Correct.

The code:

```cpp
int diameterResult = 0;

int height(TreeNode* root) {
    if (!root) return 0;
    int leftHeight = height(root->left);
    int rightHeight = height(root->right);
    diameterResult = max(diameterResult, leftHeight + rightHeight);
    return 1 + max(leftHeight, rightHeight);
}

int diameterOfBinaryTree(TreeNode* root) {
    diameterResult = 0;
    height(root);
    return diameterResult;
}
```

Notice the structure. The function `height` is a standard postorder computation — it returns the height up to the parent. But at each node, before returning, it updates `diameterResult` with `leftHeight + rightHeight`. The diameter is a side effect of computing heights.

This is the hallmark of path problems: you compute something via postorder (subtree-style), but the actual answer is a combination at each node that you track separately.

## Example 2: Binary Tree Tilt (LC 563)
**Problem:** The tilt of a node is the absolute difference between the sum of its left subtree and the sum of its right subtree. Return the sum of all tilts across every node.

This is a subtree problem. The tilt of each node depends on its left subtree sum and right subtree sum — both are "what's below."

At each node, compute the subtree sum (postorder). The tilt is `|leftSum - rightSum|`. Accumulate all tilts.

Trace:

```text
        1
       / \
      2   3
     / \   \
    4   5   6
   /
  7
```

| Node | leftSum | rightSum | subtreeSum | tilt = \|leftSum - rightSum\| | totalTilt |
|---|---|---|---|---|---|
| 7 | 0 | 0 | 7 | 0 | 0 |
| 4 | 7 | 0 | 11 | 7 | 7 |
| 5 | 0 | 0 | 5 | 0 | 7 |
| 2 | 11 | 5 | 18 | 6 | 13 |
| 6 | 0 | 0 | 6 | 0 | 13 |
| 3 | 0 | 6 | 9 | 6 | 19 |
| 1 | 18 | 9 | 28 | 9 | 28 |

Total tilt = 28.

The code:

```cpp
int totalTilt = 0;

int subtreeSum(TreeNode* root) {
    if (!root) return 0;
    int leftSum = subtreeSum(root->left);
    int rightSum = subtreeSum(root->right);
    totalTilt += abs(leftSum - rightSum);
    return root->value + leftSum + rightSum;
}
```

Same postorder shape as diameter. The function returns the subtree sum (needed by the parent), and as a side effect, accumulates the tilt.

## Example 3: Number of Edges
**Problem:** How many edges are in a tree with n nodes?

This isn't even a traversal problem. Every node except the root has exactly one parent edge. That's n−1 edges. Always.

For our 7-node tree: 7−1=6 edges.

You don't need code for this. But if an interviewer asks you to "count the edges by traversal," you'd count nodes and subtract 1. Or equivalently, each recursive call to a non-null child represents one edge:

```cpp
int countEdges(TreeNode* root) {
    if (!root) return 0;
    int edges = 0;
    if (root->left) edges += 1 + countEdges(root->left);
    if (root->right) edges += 1 + countEdges(root->right);
    return edges;
}
```

## The Pattern
Both diameter and tilt used the same trick:

1. Write a postorder function that returns something the parent needs (height, subtree sum).
2. At each node, compute the answer-relevant quantity from the left and right returns.
3. Track the global answer as a side effect.

The difference between subtree and path problems is subtle but important:

- **Subtree problems:** The answer for a node IS the combined result of its children. The return value is the answer.
- **Path problems:** The answer for a node USES the combined result of its children, but the answer itself isn't what gets returned. You return one branch (the best path going down), but the answer considers both branches (a path going through).

In the diameter problem, height returns `1 + max(leftHeight, rightHeight)` — that's the best single branch. But the diameter update uses `leftHeight + rightHeight` — that's a path through the node.

Subtree = the return value IS the answer. Path = the return value is an ingredient; the answer is a combination tracked separately.

## Try It
The starter code has `diameterOfBinaryTree` with a TODO. Implement it using the height + side-effect pattern.

```text
Input: the tree built in main (nodes 1-5)
Output: 3
```

**Predict before running:** The tree in main is:

```text
        1
       / \
      2   3
     / \
    4   5
```

The longest path is 4 → 2 → 1 → 3 (or 5 → 2 → 1 → 3), which has 3 edges. So the diameter is 3.

**Challenge:** What if the tree is a straight line (1 → 2 → 3 → 4)? The diameter is 3 — the whole tree is the path. Does your solution handle this? Trace through it.

**Challenge 2:** Can the diameter path avoid the root? Consider a tree where the root has a short right subtree but a deep, wide left subtree. Draw one and verify.

## Edge Cases to Watch For
- **Diameter does not pass through the root:** The longest path may be entirely within one subtree. You must check `leftHeight + rightHeight` at ALL nodes via a global max, not just at the root.
- **Confusing nodes vs edges:** Diameter is measured in EDGES, not nodes. The path 4-2-1-3 has 3 edges and 4 nodes. Off-by-one errors from miscounting are the most common bug.

## Problems

| Problem | Link | Difficulty |
|---|---|---|
| LC 543 — Diameter of Binary Tree | [leetcode.com/problems/diameter-of-binary-tree](https://leetcode.com/problems/diameter-of-binary-tree) | Easy |
| LC 563 — Binary Tree Tilt | [leetcode.com/problems/binary-tree-tilt](https://leetcode.com/problems/binary-tree-tilt) | Easy |
