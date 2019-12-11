---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): Repeated String Match"
subtitle: ""
date: 2019-11-11 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

The solution of [Repeated String
Match](https://leetcode.com/problems/repeated-string-match/) is relatively
simple:

```scala
object Solution
  // time: complexity to find `b_len` string in `a_len + b_len` string 
  //       O(b_len * (a_len + b_len))
  // space: O(a_len + b_len)
  def repeatedStringMatch(a: String, b: String): Int =
    val repeatCount = (b.length + a.length - 1) / a.length
    val aRepeated = a * repeatCount

    if (aRepeated.contains(b))
      repeatCount
    else if ((aRepeated + a).contains(b))
      repeatCount + 1
    else
      -1
```

The next thing I'd like to know is that if the current solution with string
multiplication is slower than the one with `StringBuilder`. My first attempt
is as follows:

```scala
import scala.annotation.tailrec

object Solution

  def repeatedStringMatch(a: String, b: String): Int = 
    val repeatCount = (b.length + a.length - 1) / a.length

    @tailrec
    def loop(sb: StringBuilder = new StringBuilder): StringBuilder =
      if (sb.length >= b.length)
        sb
      else
        loop(sb.append(a))

    val sb = loop()

    if (sb.contains(b))
      repeatCount
    else if (sb.append(a).contains(b))
      repeatCount + 1
    else
      -1
```

It compiles but gives the incorrect answer. Why? You might notice that
`StringBuilder` calls `contains`. It would cause a compilation error in Java
because `StringBuilder` doesn't have this method. The explanation is that the
`contains` is not one that I think.

`StringBuilder` in Scala is
[derived](https://www.scala-lang.org/api/2.13.1/scala/collection/mutable/StringBuilder.html)
from `AbstractSeq[Char]`. `AbstractSeq[Char]` has the polymorphic method
`contains` defined as:

```scala
def contains[A1 >: A](elem: A1): Boolean = exists (_ == elem)
```

Where `A` is `Char` for a `StringBuilder`. If `StringBuilder` calls
`contains("a")`, then the compiler infers `A1` to the least common supertype
of `Char` and `String` which is `Any`. With `Any` and `AnyVal`, such a call
would never cause a compilation error with any argument given to `contains`:

```scala
@ val sb = new StringBuilder("abc")
sb: StringBuilder = IndexedSeq('a', 'b', 'c')

@ sb.contains("a")
res1: Boolean = false

@ sb.contains(1)
res2: Boolean = false
```

Moreover, the problem is well known and much broader as I found out from [the
community](https://users.scala-lang.org/t/why-stringbuilder-contains-of-string-int-argument-throws-no-compilation-error/5436/2).

The funny thing is that `Int` is a supertype of `Char`, that causes

```scala
@ sb.contains(97)
res3: Boolean = true
```

Finally, the correct code is as follows:

```scala
import scala.annotation.tailrec

object Solution

  def repeatedStringMatch(a: String, b: String): Int = 
    val repeatCount = (b.length + a.length - 1) / a.length

    @tailrec
    def loop(sb: StringBuilder = new StringBuilder): StringBuilder =
      if (sb.length >= b.length)
        sb
      else
        loop(sb.append(a))

    val sb = loop()

    if (sb.toString.contains(b))
      repeatCount
    else if (sb.append(a).toString.contains(b))
      repeatCount + 1
    else
      -1
```
