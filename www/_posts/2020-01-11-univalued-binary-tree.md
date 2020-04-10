---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): Univalued Binary Tree"
subtitle: ""
date: 2020-01-11 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

A solution for [Leetcode's "univalued-binary-tree"](https://leetcode.com/problems/univalued-binary-tree/) using original data structures in Scala as follows:

```scala
class TreeNode(var _value: Int):
  var value: Int = _value
  var left: TreeNode = null
  var right: TreeNode = null

object Solution:
  def isUnival(t: TreeNode, v: Int): Boolean =
      Option(t)
          .map(x => x.value == v && isUnival(x.left, v) && isUnival(x.right, v))
          .getOrElse(true)
  
  def isUnivalTree(root: TreeNode): Boolean =
      isUnival(root, root.value)
```

A solution wrapping `null` values to `Option[Tree]` is as follows:

```scala
class TreeNode(var _value: Int):
  var value: Int = _value
  var left: TreeNode = null
  var right: TreeNode = null

case class Tree(value: Int, left: Option[Tree], right: Option[Tree]):
  def isLeaf: Boolean = left.isEmpty && right.isEmpty

def (n: TreeNode).convert: Option[Tree] =
  Option(n).map(n => Tree(value = n.value, left = n.left.convert, right = n.right.convert))

def (n: Option[Tree]).convert: TreeNode =
  n.map { n =>
    val tn = new TreeNode(n.value)
    tn.left = convert(n.left)
    tn.right = convert(n.right)
    tn
  }.getOrElse(null)

object Solution:
  def isUnival(t: Option[Tree], v: Int): Boolean =
    t.map { case Tree(x, l, r) => v == x && isUnival(l, v) && isUnival(r, v) }
     .getOrElse(true)

  def isUnivalTree(tn: TreeNode): Boolean =
    tn.convert match
      case None => throw new IllegalArgumentException("")
      case s @ Some(t) => isUnival(s, t.value)
```
