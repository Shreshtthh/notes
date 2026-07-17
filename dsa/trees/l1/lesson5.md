# Lesson 5

## The DFS Template — One Function, Three Traversals
### There Aren't Three Algorithms
Most resources teach preorder, inorder, and postorder as three separate traversal algorithms. They're not. They're three hooks in the same skeleton. One function. Three places to put your work.

Once you see this, you'll never memorize traversal orders again. You'll just know where to place your logic.

## The Skeleton
Here's the DFS template. Every tree traversal you'll ever write is a variation of this:

```cpp
void dfs(TreeNode* root) {
    if (!root) return;

    /* [PRE] — work here happens BEFORE visiting children */

    dfs(root->left);

    /* [IN] — work here happens BETWEEN left and right children */

    dfs(root->right);

    /* [POST] — work here happens AFTER visiting both children */
}
```

*The DFS skeleton with three hook points labeled PRE, IN, and POST between the recursive calls*

That's the whole thing. The recursion visits every node exactly once. The only question is: where do you place your logic?

- Put it at `[PRE]` → preorder traversal.
- Put it at `[IN]` → inorder traversal.
- Put it at `[POST]` → postorder traversal.

The recursive calls don't change. The base case doesn't change. Only the placement of your work changes.

## All Three From the Same Template
Here's a 7-node complete binary tree:

```text
         1
       /   \
      2     3
     / \   / \
    4   5 6   7
```

Place `print(node->value)` at each of the three positions, one at a time, and trace the output.

### Preorder: Work BEFORE Children
```cpp
void preorder(TreeNode* root) {
    if (!root) return;
    cout << root->value << " ";
    preorder(root->left);
    preorder(root->right);
}
```
You process the node first, then recurse into children. You see each node on the way down.

**Trace:** Start at 1. Print 1. Go left to 2. Print 2. Go left to 4. Print 4. Left is null, right is null, return. Back at 2, go right to 5. Print 5. Null, null, return. Back at 1, go right to 3. Print 3. Go left to 6. Print 6. Null, null, return. Back at 3, go right to 7. Print 7.

**Output:** `1 2 4 5 3 6 7`

### Inorder: Work BETWEEN Children
```cpp
void inorder(TreeNode* root) {
    if (!root) return;
    inorder(root->left);
    cout << root->value << " ";
    inorder(root->right);
}
```
You go all the way left, then process the node, then go right. You see each node in between its subtrees.

**Trace:** Start at 1. Go left to 2. Go left to 4. Go left — null, return. Print 4. Go right — null, return. Back at 2. Print 2. Go right to 5. Left null. Print 5. Right null. Back at 1. Print 1. Go right to 3. Go left to 6. Left null. Print 6. Right null. Back at 3. Print 3. Go right to 7. Left null. Print 7. Right null.

**Output:** `4 2 5 1 6 3 7`

### Postorder: Work AFTER Children
```cpp
void postorder(TreeNode* root) {
    if (!root) return;
    postorder(root->left);
    postorder(root->right);
    cout << root->value << " ";
}
```
You process both children first, then the current node. You see each node on the way up.

**Trace:** Start at 1. Go left to 2. Go left to 4. Left null, right null. Print 4. Back at 2, go right to 5. Left null, right null. Print 5. Print 2. Back at 1, go right to 3. Go left to 6. Left null, right null. Print 6. Go right to 7. Left null, right null. Print 7. Print 3. Print 1.

**Output:** `4 5 2 6 7 3 1`

## The Full Trace Table
Same tree, all three orders side by side. The "Step" column shows the order in which each node gets processed (printed):

```text
         1
       /   \
      2     3
     / \   / \
    4   5 6   7
```

| Node | Preorder Step | Inorder Step | Postorder Step |
|---|---|---|---|
| 1 | 1st | 4th | 7th |
| 2 | 2nd | 2nd | 3rd |
| 3 | 5th | 6th | 6th |
| 4 | 3rd | 1st | 1st |
| 5 | 4th | 3rd | 2nd |
| 6 | 6th | 5th | 4th |
| 7 | 7th | 7th | 5th |

Read each column top to bottom and you see a different permutation of the same 7 nodes. Same tree. Same DFS. Three different processing orders.

Notice: in preorder, the root is always first. In postorder, the root is always last. In inorder, the root sits between the left and right halves.

## What Each Order Means
This is the connection back to Lesson 3 (Top-Down vs Bottom-Up):

- **Preorder = top-down.** You do work BEFORE your children have reported back. You're passing information DOWN.
- **Postorder = bottom-up.** You do work AFTER your children have reported back. You're collecting information UP.
- **Inorder = between children.** This only matters for BSTs, where it gives you sorted order.

### Preorder is Top-Down
When you place your work at `[PRE]`, you act on the current node before its children are visited. This means you can pass information downward — a running sum, a path from the root, a depth counter.

**Example:** "Print the depth of every node." You know your depth before visiting children, so you pass `depth + 1` to each child. That's preorder / top-down.

### Postorder is Bottom-Up
When you place your work at `[POST]`, you wait for both children to finish and return their answers. Then you combine them. This is how `countNodes`, `sumValues`, `height`, and `diameter` all worked in the previous lessons.

**Example:** "Find the height of the tree." You can't know your height until your children report theirs. That's postorder / bottom-up.

### Inorder is BST-Specific
For a general binary tree, inorder traversal gives you... nothing useful. The order `4 2 5 1 6 3 7` has no inherent meaning for our example tree.

But if this were a Binary Search Tree — where `left < root < right` at every node — inorder would give you the values in sorted order. That's the entire reason inorder exists. For non-BST problems, you'll rarely use it.

## The Template is the Foundation
Every tree problem in the rest of this course is one of these:

1. **Preorder (top-down):** Carry information from parent to child. Process at `[PRE]`.
2. **Postorder (bottom-up):** Collect information from children. Process at `[POST]`.
3. **Both:** Carry information down AND collect it up. Use both `[PRE]` and `[POST]` hooks.

You will never write a tree function that doesn't fit this template. The function signature might change — you might return an `int`, or take extra parameters, or track a global variable — but the skeleton is always the same:

`base case → [PRE] → recurse left → [IN] → recurse right → [POST]`

Memorize this. Write it on a whiteboard from memory. It's the single most important piece of code in this entire course.

## The Code: All Three Traversals
```cpp
void preorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    result.push_back(root->value);
    preorder(root->left, result);
    preorder(root->right, result);
}

void inorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    inorder(root->left, result);
    result.push_back(root->value);
    inorder(root->right, result);
}

void postorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    postorder(root->left, result);
    postorder(root->right, result);
    result.push_back(root->value);
}
```

Output:
```text
1 2 4 5 3 6 7
4 2 5 1 6 3 7
4 5 2 6 7 3 1
```

Three traversals. One skeleton. Zero tricks.

## A Quick Proof That Inorder = Sorted for BSTs
Consider this BST:

```text
         4
       /   \
      2     6
     / \   / \
    1   3 5   7
```

Every node's left child is smaller, right child is larger. Inorder visits: left subtree, then root, then right subtree. So for the root 4: everything in the left subtree (1, 2, 3) comes before 4, and everything in the right subtree (5, 6, 7) comes after.

Inorder output: `1 2 3 4 5 6 7`. Sorted.

This is why BST problems almost always use inorder. And why, for general binary trees, inorder has no special role.

## Try It
The starter code has the DFS template with three TODO markers. Pick ONE position, place `cout << root->value << " "` there, and predict the output before running.

```text
Input: the tree built in main (complete binary tree, nodes 1-7)
Preorder output:  1 2 4 5 3 6 7
Inorder output:   4 2 5 1 6 3 7
Postorder output: 4 5 2 6 7 3 1
```

**Predict before running:** If you place the print at `[IN]`, what's the first value printed? It's NOT 1. The recursion goes left, left, left until it hits null, then prints the first node it backtracks to. That's node 4.

**Challenge:** Write a function that prints every node's depth using preorder. The function signature should be `void printDepths(TreeNode* root, int currentDepth)`. Why does this HAVE to be preorder? *(Hint: you need to know the depth before you visit the children, not after.)*

**Challenge 2:** Write a function that returns the number of leaf nodes using postorder. A leaf is a node where both children are null. Why is this naturally postorder? *(Hint: can you know if a node is a leaf without checking its children first? Actually, you CAN — this works as preorder too. Think about why both work here.)*

## Problems

| Problem | Link | Difficulty |
|---|---|---|
| LC 144 — Binary Tree Preorder Traversal | [leetcode.com/problems/binary-tree-preorder-traversal](https://leetcode.com/problems/binary-tree-preorder-traversal) | Easy |
| LC 94 — Binary Tree Inorder Traversal | [leetcode.com/problems/binary-tree-inorder-traversal](https://leetcode.com/problems/binary-tree-inorder-traversal) | Easy |
| LC 145 — Binary Tree Postorder Traversal | [leetcode.com/problems/binary-tree-postorder-traversal](https://leetcode.com/problems/binary-tree-postorder-traversal) | Easy |

## What's Next?
You now have the DFS template with three hooks. But which hook should you use for which problem? Chapter 2 makes you fluent in all three — recursive, iterative, BFS, and the O(1)-space Morris traversal.