---
layout: post
author: Alexander Myltsev
title: "Knuth-Morris-Prath Algorithm in Scala3 (Dotty)"
subtitle: ""
date: 2019-11-25 08:00:00 +0400
comments: true
tags: [scala, dotty, coding interview, competitive programming]
---

I wrote at [the previous post]({% post_url 2019-11-11-repeated-string-match_naive %}) how to solve ["Repeated String Match" problem from Leetcode](https://leetcode.com/problems/repeated-string-match/). I tell you at the next post how to solve it linearly to string lengths. For that, I need the implementation of [Knuth-Morris-Prath algorithm](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm).

Robert Sedgewick and Kevin Wayne [proposed the Java solution](https://algs4.cs.princeton.edu/53substring/KMPplus.java.html) at their book. Gary Struthers translated it to [Scala 2 code](https://github.com/garyaiki/Scala-Algorithms/blob/master/src/main/scala/org/gs/search/KMP.scala). Scala 2 code has straightforward translation to Dotty code:

```scala
class KMPplus private (pattern: String, next: Array[Int])
  def search(text: String): Option[Int] =
    val m = pattern.length
    val n = text.length
    var i = 0
    var j = 0
    while i < n && j < m do
      while j >= 0 && text(i) != pattern(j) do
        j = next(j)
      j += 1
      i += 1
    if j == m
      Some(i - m)
    else
      None
 
object KMPplus
  def apply(pattern: String): KMPplus =
    val next = Array.ofDim[Int](pattern.length)
    var j = -1
    for i <- 0 until pattern.length do
      if i == 0
        next(i) = -1
      else if pattern(i) != pattern(j)
        next(i) = j
      else
        next(i) = next(j)
      while j >= 0 && pattern(i) != pattern(j) do
        j = next(j)
      j += 1
    new KMPplus(pattern, next)
```

This code is extensively imperative. I wonder if there is a purely functional implementation of the algorithm. Twan van Laarhoven [implemented it in Haskell](https://www.twanvl.nl/blog/haskell/Knuth-Morris-Pratt-in-Haskell). Also, there is [the detailed explanation for the implementation at "Stack Overflow"](https://stackoverflow.com/questions/16694306/knuth-morris-pratt-algorithm-in-haskell). The implementation translated to Scala is as follows:

```scala
case class KMP[T](done: Boolean, next: T => KMP[T])

def makeTable[T](ss: List[T]): KMP[T] =
  def makeTable_(xs: List[T], failure: T => KMP[T]): KMP[T] =
    xs match
      case List()  => 
        KMP(done = true, failure)
      case x :: xs =>
        def success: KMP[T] =
          makeTable_(xs, failure(x).next)
        def test(c: T): KMP[T] =
          if c == x then success else failure(c)
        KMP(done = false, test)

  def table: KMP[T] = makeTable_(ss, _ => table)

  table

implicit class StringOps(s: String)
  def isSubstrnigOf(b: String): Boolean =
    @tailrec
    def mtch[T](table: KMP[T], xs: List[T]): Boolean =
      xs match
        case List()  => table.done
        case x :: xs => table.done || mtch(table.next(x), xs)

    mtch(makeTable(s.toList), b.toList)
```

The usage:

```scala
@ "abc".isSubstrnigOf("abc") 
_: Boolean = true
@ "abc".isSubstrnigOf("aabc") 
_: Boolean = true
@ "abc".isSubstrnigOf("aabx") 
_: Boolean = false
@ ("a" * 10).isSubstrnigOf("a" * 8 + "ba") 
_: Boolean = false
```
