---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): Squares of a Sorted Array"
subtitle: ""
date: 2019-12-20 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

## Imperative Solution

I start an imperative solution for 
["Squares of a Sorted Array" problem from Leetcode](https://leetcode.com/problems/squares-of-a-sorted-array/). 
I take two pointers -- `idxL` to the begin and `idxR` to the end
of the input array -- and move them to the center until they meet. Yet another element of the result
would be `max(sqr(a(idxL)), sqr(a(idxR)))`, which equals to `sqr(max(-a(idxL),a(idxR))` since an alement at
`idxL` is always non-positive. An imperative solution is as follows:

```scala
object Solution
  def sqr(x: Int): Int = x * x

  def sortedSquares(a: Array[Int]): Array[Int] =
    var idxL = 0
    var idxR = a.length - 1
    var idx = idxR
    val res = Array.ofDim[Int](a.length)

    while (idxL <= idxR)
      if (-a(idxL) < a(idxR))
        res(idx) = sqr(a(idxR))
        idxR -= 1
      else
        res(idx) = sqr(a(idxL))
        idxL += 1
      idx -= 1

    res
```

Let's check it:

```scala
@ Solution.sortedSquares(Array(-4,-1,0,3,10))
val res0: Array[Int] = Array(0, 1, 9, 16, 100)
```

The time complexity is `O(a.size)`. And space complexity is `O(1)` for computation
and `O(a.size)` for the final result.

## Functional Solution

I start with a naive imperative-like solution:

```scala
import scala.annotation.tailrec

object Solution
  def sqr(x: Int): Int = x * x

  @tailrec
  def process(a: Vector[Int], idxL: Int, idxR: Int, 
              res: Vector[Int], idx: Int): Vector[Int] =
    if (idxL <= idxR)
      if (-a(idxL) < a(idxR))
        process(a, idxL, idxR - 1, res.updated(idx, sqr(a(idxR))), idx - 1)
      else
        process(a, idxL + 1, idxR, res.updated(idx, sqr(a(idxL))), idx - 1)
    else
      res

  def sortedSquares(a: Array[Int]): Array[Int] =
    val res = process(a=a.toVector, idxL=0, idxR=a.length - 1, 
                      Vector.fill(a.length)(0), a.length - 1)
    res.toArray
```

A more functional solution would be as follows:

```scala
object Solution
  def sqr(x: Int): Int = x * x

  def merge(neg: List[Int], pos: List[Int]): List[Int] = (neg, pos) match
    case (Nil, xs) => xs.map(sqr)
    case (xs, Nil) => xs.map(sqr)
    case (n :: ns, p :: ps) => 
      if -n > p then
        sqr(n) :: merge(ns, pos)
      else
        sqr(p) :: merge(neg, ps)

  def sortedSquares(a: Array[Int]): Array[Int] =
    val l = a.toList
    val pos = l.filter(_ >= 0)
    val neg = l.filter(_ < 0)
    val res = merge(neg, pos.reverse)
    res.reverse.toArray
```

Nevertheless, it passes all the tests, it's ineffective in a few parts:
- it creates a copy of the list while filtering the original one
- it reverses a list twice
- it doesn't make tail recursion optimization that might blow up the call stack at runtime
- it uses `List` which [is much slower](https://www.infoq.com/presentations/Functional-Data-Structures-in-Scala/) than `Vector`

A better solution is as follows:

```scala
import scala.annotation.tailrec

object Solution
  def sqr(x: Int): Int = x * x

  @tailrec
  def merge(l: Vector[Int], res: Vector[Int]): Vector[Int] = l match
    case Vector() => res
    case Vector(x) => sqr(x) +: res
    case v => 
      val neg +: back = v
      val front :+ pos = v
      if -neg > pos then
        merge(back, sqr(neg) +: res)
      else
        merge(front, sqr(pos) +: res)

  def sortedSquares(a: Array[Int]): Array[Int] =
    val res = merge(a.toVector, Vector.empty)
    res.toArray
```

The complexity is the same: `O(a.size)` for time and `O(a.size)` for space.
