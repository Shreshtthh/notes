# Lesson 2

## The Call Stack IS the Tree
### What Actually Happens When You Recurse?
In the last lesson, we wrote `countNodes` and traced which calls happen and what they return. But we skipped something important: where does the computer store all those in-progress calls?

When `countNodes(1)` calls `countNodes(2)`, node 1 isn't finished yet. It's waiting. When `countNodes(2)` calls `countNodes(4)`, node 2 is also waiting. We now have three unfinished calls stacked on top of each other.

That stack of waiting calls is the **call stack**. And here's the thing that changes how you think about tree recursion:

**At any moment during DFS, the call stack contains exactly the nodes on the path from the root to the current node.**

Not some of them. Not an approximation. The call stack IS the root-to-current-node path. The computer is literally holding the path in memory for you, one frame per node.

### A 7-Node Tree
We'll trace this tree for the rest of the lesson:

```text
          1
         / \
        2   3
       / \   \
      4   5   7
     /
    6
```

Seven nodes. The deepest path is $1 \rightarrow 2 \rightarrow 4 \rightarrow 6$, which has depth 3 (three edges).

## Frame-by-Frame Trace
We'll run a simple DFS that visits every node. At each step, we show the tree on the left and the call stack on the right.

*Call stack growing and shrinking alongside the tree during DFS traversal*

Here's the full trace. The "Call Stack" column shows the frames from bottom (root) to top (current node):

| Step | Action | Call Stack (bottom → top) | Path from Root |
| :--- | :--- | :--- | :--- |
| 1 | Enter node 1 | `[1]` | $1$ |
| 2 | Enter node 2 | `[1, 2]` | $1 \rightarrow 2$ |
| 3 | Enter node 4 | `[1, 2, 4]` | $1 \rightarrow 2 \rightarrow 4$ |
| 4 | Enter node 6 | `[1, 2, 4, 6]` | $1 \rightarrow 2 \rightarrow 4 \rightarrow 6$ |
| 5 | 6 has no children — return | `[1, 2, 4]` | $1 \rightarrow 2 \rightarrow 4$ |
| 6 | 4's right is `nullptr` — return | `[1, 2]` | $1 \rightarrow 2$ |
| 7 | Enter node 5 | `[1, 2, 5]` | $1 \rightarrow 2 \rightarrow 5$ |
| 8 | 5 has no children — return | `[1, 2]` | $1 \rightarrow 2$ |
| 9 | 2 is done — return | `[1]` | $1$ |
| 10 | Enter node 3 | `[1, 3]` | $1 \rightarrow 3$ |
| 11 | 3's left is `nullptr` — skip | `[1, 3]` | $1 \rightarrow 3$ |
| 12 | Enter node 7 | `[1, 3, 7]` | $1 \rightarrow 3 \rightarrow 7$ |
| 13 | 7 has no children — return | `[1, 3]` | $1 \rightarrow 3$ |
| 14 | 3 is done — return | `[1]` | $1$ |
| 15 | 1 is done — return | `[]` | (empty) |

Read columns 3 and 4. They're identical at every step. The call stack and the root-to-current path are the same thing.

## Why This Matters
This isn't a coincidence — it's structural. Think about what a function call does:

* When you enter a child, the child's frame gets pushed onto the stack.
* When you finish a child and return to the parent, the child's frame gets popped off the stack.

Pushing and popping match the DFS traversal exactly. Going deeper in the tree = pushing. Backtracking = popping. The call stack mirrors the tree path at every moment.

**Recursion doesn't traverse the tree. It becomes the tree.**

The call tree of a DFS function is an exact copy of the input tree. Every node in the tree corresponds to exactly one stack frame. The parent-child relationships in the tree are the caller-callee relationships in the recursion.

### Stack Depth = Tree Height
Look at the trace again. The maximum call stack size was 4 frames — at step 4, when we reached node 6. The stack was `[1, 2, 4, 6]`.

The height of the tree is 3 (the longest root-to-leaf path has 3 edges: $1 \rightarrow 2 \rightarrow 4 \rightarrow 6$). Add 1 for the root frame itself: maximum stack depth = height + 1.

In terms of space complexity, the recursive DFS on a tree uses $O(h)$ space, where $h$ is the height.

This has a real consequence. If someone gives you a tree that's actually a long chain — say, $10^5$ nodes in a straight line — the height is $10^5$. The call stack will hold $10^5$ frames. On most systems, the default stack size is about $10^6$ bytes (1 MB). Each frame takes at least a few dozen bytes. You will get a stack overflow.

**Stack overflow on tree problems is not a bug in your logic. It's a depth problem.** When the tree is deep, recursion costs real memory — one frame per level.

## Two Stacks, Two Uses
The call stack during DFS serves two purposes that are worth distinguishing:

1. **The Active Path Stack (Push & Pop):** At any moment during DFS, the call stack contains exactly the path from root to the current node. When you enter a node, it's pushed. When you leave, it's popped. This gives you $O(1)$ access to all ancestors of the current node — no need for parent pointers or binary lifting.

You can make this explicit by maintaining your own stack alongside the recursion:

```cpp
vector<int> pathStack;

void dfs(Node* node) {
    if (!node) return;
    pathStack.push_back(node->value);
    // pathStack now holds the root-to-current path
    // print it, query ancestors, check conditions — all O(1) per ancestor
    dfs(node->left);
    dfs(node->right);
    pathStack.pop_back();  // backtrack
}
```

2. **The History Stack (Push Only):** If you record every node you visit in order (without popping), you get the preorder traversal — a 1D "shadow" of the tree. Add entry and exit timestamps, and you get the Euler Tour — the foundation of tree linearization in Chapter 11. The active path gives you ancestor access; the history gives you subtree ranges.

This is why balanced trees ($h = O(\log n)$) are safe to recurse on, but degenerate trees ($h = O(n)$) are dangerous. A balanced tree with $10^6$ nodes has height ~20. A degenerate chain with $10^6$ nodes has height $10^6$.

## The Code
Here's a simple DFS that prints each node and its current depth. The depth parameter is the call stack position — proof that the stack tracks the path.

```cpp
void dfs(Node* root, int currentDepth) {
    if (!root) return;
    cout << "Node " << root->value << " at depth " << currentDepth << "\n";
    dfs(root->left, currentDepth + 1);
    dfs(root->right, currentDepth + 1);
}
```

Output:

```text
Node 1 at depth 0
Node 2 at depth 1
Node 4 at depth 2
Node 6 at depth 3
Node 5 at depth 2
Node 3 at depth 1
Node 7 at depth 2
```

The `currentDepth` variable tells you how many frames are on the stack above main. Node 6 is at depth 3 — the stack holds frames for nodes 1, 2, 4, and 6. Four frames, depth 3. It lines up.

## Max Depth: Your First Height Problem
Now use the stack insight to solve a real problem.

**Problem:** Given a binary tree, return its maximum depth. Maximum depth is the number of edges on the longest path from root to any leaf.

**The recursive question:** If I know the max depth of the left subtree and the max depth of the right subtree, what's the max depth of the whole tree?

It's `1 + max(leftDepth, rightDepth)`. One edge connects this node to the deeper child, plus however deep that child goes.

**Base case:** `nullptr` returns $-1$. A single node (leaf) then computes `1 + max(-1, -1) = 0`, which is correct — a leaf has depth 0.

```cpp
int maxDepth(Node* root) {
    if (!root) return -1;
    int leftDepth = maxDepth(root->left);
    int rightDepth = maxDepth(root->right);
    return 1 + max(leftDepth, rightDepth);
}
```

Trace on our 7-node tree:

| Call | Node | Left returns | Right returns | Result |
| :--- | :--- | :--- | :--- | :--- |
| `maxDepth(6)` | 6 | -1 | -1 | 0 |
| `maxDepth(4)` | 4 | 0 (from 6) | -1 (`nullptr`) | 1 |
| `maxDepth(5)` | 5 | -1 | -1 | 0 |
| `maxDepth(2)` | 2 | 1 (from 4) | 0 (from 5) | 2 |
| `maxDepth(7)` | 7 | -1 | -1 | 0 |
| `maxDepth(3)` | 3 | -1 (`nullptr`) | 0 (from 7) | 1 |
| `maxDepth(1)` | 1 | 2 (from 2) | 1 (from 3) | 3 |

Maximum depth = 3. The path is $1 \rightarrow 2 \rightarrow 4 \rightarrow 6$. This matches the maximum stack depth we observed in the trace table above — 4 frames, 3 edges.

## Try It
The starter code has a `maxDepth` function with a `TODO`. Fill in the body.

```text
Input: (the 7-node tree built in main)
Output: 3
```

**Predict before running:** At what point during the DFS does the call stack reach its maximum size? Which node triggers it? (Answer: node 6. The stack holds `[1, 2, 4, 6]` — 4 frames.)

**Challenge:** What does `maxDepth` return on a tree that's a straight line: $1 \rightarrow 2 \rightarrow 3 \rightarrow 4 \rightarrow 5$? How many stack frames are active at the deepest point? What happens if the chain has $10^6$ nodes?

**Practice — Predict the Call Stack:** For our 7-node tree, what is the exact call stack at the moment node 5 is being processed? Write it out before checking the trace table. (Answer: `[1, 2, 5]`.)

What about when node 7 is being processed? (Answer: `[1, 3, 7]`.)

What about right after node 6 returns and we're back at node 4, about to check its right child? (Answer: `[1, 2, 4]`.)

## Edge Cases to Watch For
* **Depth definition varies by problem:** Some problems define depth as number of nodes on the longest path (returning 0 for null), others as number of edges (returning $-1$ for null). Always check whether the problem counts nodes or edges and adjust the base case accordingly.
* **Minimum depth vs maximum depth:** `minDepth` has a subtle trap — if one child is `nullptr`, you must NOT take the min of $-1$ and the other child's depth. A null child does not form a leaf path. Handle the one-child case explicitly with an `if` guard.

## Problems

| Problem | Link | Difficulty |
| :--- | :--- | :--- |
| LC 104 — Maximum Depth of Binary Tree | [https://leetcode.com/problems/maximum-depth-of-binary-tree](https://leetcode.com/problems/maximum-depth-of-binary-tree) | Easy |
| LC 111 — Minimum Depth of Binary Tree | [https://leetcode.com/problems/minimum-depth-of-binary-tree](https://leetcode.com/problems/minimum-depth-of-binary-tree) | Easy |

[Trees Are Recursive Structures](lesson1.md)

[Back to chapter](../README.md)
