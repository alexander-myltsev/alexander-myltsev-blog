---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): N-Repeated Element in Size 2N Array"
subtitle: ""
date: 2019-12-29 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

A baseline __imperative solution__ for [Leetcode's "N-Repeated Element in Size 2N Array"](https://leetcode.com/problems/n-repeated-element-in-size-2n-array) is as follows:

```scala
object Solution
  def repeatedNTimes(a: Array[Int]): Int = 
    var i = 0
    while (i < a.size)
      var j = 1
      while (j < 4 && j + i < a.size)
        if (a(i) == a(i + j))
          return a(i)
        j += 1
      i += 1
    throw new IllegalArgumentException("a repeated element must exist")
```

It takes **560 ms** to pass all tests. A __functional solution__ as follows takes
**588 ms** to go through the same tests:

```scala
object Solution
  def repeatedNTimes(a: Array[Int]): Int = 
    val v = a.toVector
    val slides = v.sliding(4) ++ (2 to 3).map(v.takeRight(_))
    val resOpt = slides.find(s => s.tail.contains(s.head))
    resOpt match
      case Some(r +: _) => r
      case Some(_) => ??? // never happens
      case None => 
        throw IllegalArgumentException("a repeated element must exist")
```
