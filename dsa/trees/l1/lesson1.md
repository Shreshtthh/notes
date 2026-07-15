# Lesson 1

## Trees Are Recursive Structures
If you're coming from our Array and DP courses: you're used to dependency chaining (value A can't be computed until B is resolved) and parameter fixing (lock in one dimension, solve the rest). Trees aren't a departure from that worldview — they're the purest expression of it. Bottom-up traversal IS dependency chaining on a branching structure. Top-down traversal IS parameter fixing where the "fixed parameter" narrows at each level. By the end of this course, you'll see that trees were hiding inside every recursive algorithm you've ever written.

## Forget Everything You've Heard About Trees
Before we write a single tree algorithm, we need to settle one thing: what is a tree, really?

Not the textbook answer. Not "a connected acyclic graph." That's technically correct but useless for problem-solving. It tells you what a tree isn't (no cycles, no disconnection) rather than what a tree is.

Here's the definition that actually helps you write code:

**A tree is a node connected to smaller trees.**

That's it. A tree is a value, plus zero or more children, each of which is itself a tree. The children are independent — what happens in the left subtree has no effect on the right subtree.

This is a recursive definition, and it's not an accident. Trees are the only data structure whose definition IS its algorithm. When someone says "process a tree," they mean: process this node, then process each child the same way.

Recursion isn't about "calling a function that calls itself." It's about state transition: you're at state $S_1$, and you define how to jump to $S_2$, and when to stop. On arrays, the state is an index $i$ transitioning to $i+1$. On trees, the state is a node transitioning to its children. The structure changes, the principle doesn't.

**The Leap of Faith:** When you write `solve(root->left)`, don't simulate what happens inside. Trust that it returns the correct answer for the left subtree. Your only job is to combine it with the right subtree's answer and the current node. This is the single most important skill for tree problems — and the hardest habit to build.

**Prerequisite:** This course assumes you can convert any iterative pattern (for-loops) to its recursive form, and vice versa. If that feels unfamiliar, practice converting a few simple loops to recursion before starting.

*A tree node connected to two smaller trees, each of which is itself a tree of smaller trees*

## Types of Binary Trees
Before we work with a tree, know the vocabulary. These four types come up in interviews and complexity analysis:

| Type | Rule | Height | Node count |
| :--- | :--- | :--- | :--- |
| **Full** | Every node has 0 or 2 children (never 1) | Varies | Odd number |
| **Complete** | All levels filled except possibly the last, which fills left-to-right | $O(\log n)$ | $[2^h, 2^{h+1} - 1]$ at height $h$ |
| **Perfect** | All levels completely filled | $h = \log_2(n+1) - 1$ | Exactly $2^{h+1} - 1$ |
| **Balanced** | Left and right subtree heights differ by at most 1 | $O(\log n)$ | Varies |

A perfect tree is also complete. A complete tree is also balanced. A full tree may not be balanced (could be deep on one side).

The key insight: balanced = $O(\log n)$ height. This is why balanced BSTs guarantee $O(\log n)$ search — and why skewed trees (essentially linked lists) degrade to $O(n)$.

## Your First Tree
Here's a tree with 5 nodes:

```text
        1
       / \
      2   3
     / \
    4   5
```

The root is 1. It has two children: 2 and 3. Node 2 has two children: 4 and 5. Nodes 3, 4, 5 have no children — they're called leaves.

Now look at it differently. Node 1 is connected to two smaller trees:

* **Left subtree of 1:** a tree rooted at 2, which itself has two subtrees (nodes 4 and 5).
* **Right subtree of 1:** a tree rooted at 3, which is a single node — a tree of size 1.

And node 2 is connected to two even smaller trees:

* **Left subtree of 2:** a single node 4.
* **Right subtree of 2:** a single node 5.

At the bottom, each leaf is a tree of size 1. And `nullptr` (an empty child) is a tree of size 0. This bottoming out is what stops the recursion.

## The One Rule: Decompose, Don't Enumerate
When you see a tree problem, your first instinct might be to iterate through all the nodes somehow — maybe put them in an array, or use a queue. Resist that instinct.

Instead, ask: *"If I knew the answer for the left subtree and the right subtree, could I combine them to get the answer for the whole tree?"*

If yes — and it almost always is — the algorithm writes itself.

## Example: Counting Nodes
**Problem:** Given a binary tree, return the total number of nodes.

How many nodes are in our tree?

```text
        1
       / \
      2   3
     / \
    4   5
```

You could count them visually: 5 nodes. But how would you write code for a tree of a million nodes?

**The recursive question:** If I knew the count of the left subtree and the count of the right subtree, could I compute the total?

Yes: `total = 1 + leftCount + rightCount`.

* Count of left subtree of 1 = count of tree rooted at 2 = 3 (nodes 2, 4, 5)
* Count of right subtree of 1 = count of tree rooted at 3 = 1
* Total = 1 + 3 + 1 = 5

And how did we get "count of tree rooted at 2 = 3"? Same logic:

* Count of left subtree of 2 = count of tree rooted at 4 = 1
* Count of right subtree of 2 = count of tree rooted at 5 = 1
* Total at 2 = 1 + 1 + 1 = 3

And how did we get "count of tree rooted at 4 = 1"? Node 4 has no children:

* Left subtree = `nullptr` → count = 0
* Right subtree = `nullptr` → count = 0
* Total at 4 = 1 + 0 + 0 = 1

This is recursion doing what recursion does. We never managed a global counter. We never kept a list. We asked each node the same question, and each node answered by asking its children.

## The Full Trace
Let's trace exactly how the recursion unfolds. The calls happen depth-first — we go all the way down the left side before touching the right.

| Call | Node | Left returns | Right returns | Result |
| :--- | :--- | :--- | :--- | :--- |
| `countNodes(4)` | 4 | 0 (`nullptr`) | 0 (`nullptr`) | 1 |
| `countNodes(5)` | 5 | 0 (`nullptr`) | 0 (`nullptr`) | 1 |
| `countNodes(2)` | 2 | 1 (from node 4) | 1 (from node 5) | 3 |
| `countNodes(3)` | 3 | 0 (`nullptr`) | 0 (`nullptr`) | 1 |
| `countNodes(1)` | 1 | 3 (from node 2) | 1 (from node 3) | 5 |

Read the table bottom to top: the answer "bubbles up" from the leaves to the root. Node 4 reports 1. Node 5 reports 1. Node 2 collects both and reports 3. Node 3 reports 1. Node 1 collects both and reports 5.

This bottom-up bubbling is the most important pattern in tree algorithms. It has a name: **postorder processing**. You'll see it in every chapter of this course.

## The Code

```cpp
int countNodes(Node* root) {
    if (!root) return 0;
    int leftCount = countNodes(root->left);
    int rightCount = countNodes(root->right);
    return 1 + leftCount + rightCount;
}
```

Three things to notice:

1. **The base case is `nullptr`, not "leaf."** A leaf node isn't special — it's just a node whose children are both `nullptr`. The recursion handles it naturally.
2. **No global variables.** Each call returns its own answer. The parent combines them. Nothing is shared.
3. **The structure mirrors the tree.** The call tree of `countNodes` IS the original tree. Every node gets visited exactly once. Time: $O(n)$.

## The Mental Shift: Trees Are Not Arrays
Arrays are flat. You can index into position 7. You can iterate from start to end. You know the size.

Trees are hierarchical. You can't index into "the 7th node" without defining what "7th" means. You can't iterate in a flat loop. You don't know the size without computing it (which itself requires a tree algorithm).

This discomfort is normal. The shift that makes trees click:

**Stop thinking about "all the nodes" and start thinking about "this node and its children."**

You never need to hold the whole tree in your head. At any point in the recursion, you're standing at one node, and the only things that exist are:

1. This node's value
2. The answer from the left subtree (someone else computed it)
3. The answer from the right subtree (someone else computed it)

"Someone else" is the recursion. You don't care how the left subtree got its answer — only that it did.

## A Second Problem: Sum of All Values
Given a binary tree where each node holds an integer, return the sum of all values.

**The recursive question:** If I knew the sum of the left subtree and the sum of the right subtree, could I compute the total sum?

Yes: `totalSum = thisNode.value + leftSum + rightSum`.

This is literally the same structure as `countNodes`, with 1 replaced by `root->value`:

```cpp
int sumValues(Node* root) {
    if (!root) return 0;
    int leftSum = sumValues(root->left);
    int rightSum = sumValues(root->right);
    return root->value + leftSum + rightSum;
}
```

Trace on our tree (values: 1, 2, 3, 4, 5):

| Call | Node | Value | Left returns | Right returns | Result |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `sumValues(4)` | 4 | 4 | 0 | 0 | 4 |
| `sumValues(5)` | 5 | 5 | 0 | 0 | 5 |
| `sumValues(2)` | 2 | 2 | 4 | 5 | 11 |
| `sumValues(3)` | 3 | 3 | 0 | 0 | 3 |
| `sumValues(1)` | 1 | 1 | 11 | 3 | 15 |

Sum = 1 + 2 + 3 + 4 + 5 = 15. Matches.

## The Pattern You Just Learned
Both problems — `countNodes` and `sumValues` — follow the exact same shape:

1. **Base case:** if the node is `nullptr`, return some identity value (0 for both count and sum).
2. **Recurse:** get the answer from the left child. Get the answer from the right child.
3. **Combine:** merge the two child answers with the current node's contribution.

This is the postorder template, and it is the single most important pattern in tree algorithms. We'll formalize it in Chapter 3 and use it to solve diameter, max path sum, LCA, and a dozen other problems — all with this exact structure.

For now, the takeaway is simpler:

**A tree is a node plus smaller trees. An algorithm on a tree is a function on this node plus recursive calls on smaller trees.**

Once you internalize this, trees stop being scary. They become the most predictable data structure in all of computer science.

## Try It
The starter code has a `countNodes` function with a `TODO`. Fill in the body.

```text
Input: (the tree built in main)
Output: 5
```

**Predict before running:** What does `countNodes` return on a single-node tree (root with no children)? Trace through the code mentally: `leftCount = countNodes(nullptr) = 0`, `rightCount = countNodes(nullptr) = 0`, return `1 + 0 + 0 = 1`. Correct.

**Challenge:** Write a function `maxValue(Node* root)` that returns the maximum value in the tree. What's the base case? (Hint: what should `maxValue(nullptr)` return? Think carefully — what identity value works for max?)

**Challenge 2:** Write `height(Node* root)` that returns the height of the tree (longest root-to-leaf path). Height of a single node is 0. Height of `nullptr` is -1 (so a single node returns `max(-1, -1) + 1 = 0`).

## Problems

| Problem | Link | Difficulty |
| :--- | :--- | :--- |
| LC 104 — Maximum Depth of Binary Tree | [https://leetcode.com/problems/maximum-depth-of-binary-tree](https://leetcode.com/problems/maximum-depth-of-binary-tree) | Easy |
| LC 226 — Invert Binary Tree | [https://leetcode.com/problems/invert-binary-tree](https://leetcode.com/problems/invert-binary-tree) | Easy |
| LC 222 — Count Complete Tree Nodes | [https://leetcode.com/problems/count-complete-tree-nodes](https://leetcode.com/problems/count-complete-tree-nodes) | Medium |
