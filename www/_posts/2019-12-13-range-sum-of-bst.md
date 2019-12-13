---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): Range Sum of BST"
subtitle: ""
date: 2019-12-13 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

I solve ["Range Sum of BST" problem from Leetcode](https://leetcode.com/problems/range-sum-of-bst/) recursively. 

Let's define data structures first:

```scala
case class TreeNode(value: Int, left: TreeNode, right: TreeNode)

def empty: TreeNode = null
def leaf(value: Int): TreeNode = TreeNode(value, null, null)

given treeNodeOps: (tn: TreeNode) extended with
  def isEmpty: Boolean = tn == null
```

Then the recursive solution is as follows:
 
```scala
object Solution
  // time: O(nodes_count)
  // space: O(tree_depth)
  def rangeSumBST(root: TreeNode, l: Int, r: Int): Int =
    if (root.isEmpty)
      0
    else
      val r1 =
        if (l <= root.value && root.value <= r) root.value
        else 0

      val r2 =
        if (root.value > l) rangeSumBST(root.left, l, r)
        else 0

      val r3 =
        if (root.value < r) rangeSumBST(root.right, l, r)
        else 0

      r1 + r2 + r3
```

The simple test:

```scala
@main def main(): Unit =
  val tree = 
    TreeNode(
      10,
      TreeNode(
        5,
        leaf(3),
        leaf(7)),
      TreeNode(
        15,
        empty,
        leaf(18)),
    )
  assert(Solution.rangeSumBST(tree, 7, 15) == 32)
```
