---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): Repeated String Match with Knuth-Morris-Prath Algorithm"
subtitle: ""
date: 2019-12-10 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

I wrote [how to solve]({% post_url 2019-11-11-repeated-string-match_naive %}) how to solve ["Repeated String Match" problem from Leetcode](https://leetcode.com/problems/repeated-string-match/) for `O(a_len * (a_len + b_len)`.

I told [how to imlement  Knuth-Morris-Prath algorithm in Scala]({% post_url 2019-11-25-Knuth-Morris-Prath_implementation %}). Having a faster search, the "Repeated String Problem" can be solved for `O(a_len + b_len)` as follows:

```scala
class KMP private(pattern: String, /* ... */)
  def search(text: String): Option[Int] = // ...

object KMP
  def apply(pattern: String): KMP =
    // ...
    new KMP(pattern, /* ... */)

given intOps: (v: Int) extended with
  def divRoundUp(d: Int): Int = (v + d - 1) / d

object Solution
  // time: O(a_len + b_len)
  // space: O(a_len + b_len)
  def repeatedStringMatch(a: String, b: String): Int =
    val repeatCount = b.length.divRoundUp(a.length)
    val aRepeated = a * repeatCount
    val kmp = KMP(pattern = b)

    if kmp.search(text = aRepeated).isDefined
      repeatCount
    else if kmp.search(text = aRepeated + a).isDefined
      repeatCount + 1
    else
      -1
  
  def main(args: Array[String]): Unit =
    assert(repeatedStringMatch("abcd", "cdabcdab") == 3)
```

I used the `KMP` instance twice to search the pattern in two different texts. But there is a way to use a single run for both texts. The solution is to build prefix function and use it to search for a pattern in the string: `pattern + '#' + text`. The `text` is input string repeated `ceil(b / a)` times. The solution in total is as follows:

```scala
opaque type PrefixFunction = Array[Int]

object PrefixFunction
  def apply(pattern: String): PrefixFunction =
    val pi = Array.ofDim[Int](pattern.length)
    var i = 1
    while (i < pattern.size)
      var j = pi(i - 1)
      while (j > 0 && pattern(i) != pattern(j))
        j = pi(j - 1)
      if pattern(i) == pattern(j)
        j += 1
      pi(i) = j
      i += 1
    pi

given prefixFunctionOps: (x: PrefixFunction) extended with 
  def apply(idx: Int) = x(idx)

given intOps: (v: Int) extended with
  def divRoundUp(d: Int): Int = (v + d - 1) / d

object Solution
  // time: O(a_len + b_len)
  // space: O(a_len + b_len)
  def repeatedStringMatch(a: String, b: String): Int =
    val repeatCount = b.length.divRoundUp(a.length)
    val str = b + "#" + (a * (repeatCount + 1))
    val pf = PrefixFunction(pattern = str)

    var idx = b.size + 1
    var found = false
    while (!found && idx < str.size)
      if pf(idx) == b.size
        found = true
      else
        idx += 1

    if !found
      -1
    else
      (idx - b.size).divRoundUp(a.length)
  
  def main(args: Array[String]): Unit =
    assert(repeatedStringMatch("abcd", "cdabcdab") == 3)
```
