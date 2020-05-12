---
layout: post
author: Alexander Myltsev
title: "Leetcode in Scala3 (Dotty): N-Repeated Element in Size 2N Array"
subtitle: ""
date: 2019-12-29 08:00:00 +0400
comments: true
tags: [scala, dotty, leetcode, coding interview, competitive programming]
---

## Imperative Solution in Java

I start with a baseline __imperative solution__ for [Leetcode's "N-Repeated Element in Size 2N Array"](https://leetcode.com/problems/n-repeated-element-in-size-2n-array) in Java as follows:

```java
package main;

public class Solution0_java {
  public static int repeatedNTimes(int[] a) {
    for (int i = 0; i < a.length; ++i) {
      for (int j = 1; j < 4 && i + j < a.length; ++j) {
        if (a[i] == a[i + j]) {
            return a[i];
        }
      }
    }
    throw new IllegalArgumentException();
  }
}
```

## JMH Benchmark

I measure the performance with
[JMH](https://openjdk.java.net/projects/code-tools/jmh/) on the `2,4 GHz
8-Core Intel Core i9` CPU:

```scala
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@BenchmarkMode(Array(Mode.Throughput))
class SolutionBenchmark:

  def randomArray: Array[Int] =
    val n = 100
    var arr1 = Array.range(1, n + 2)
    val v = arr1(Random.nextInt(arr1.size))
    val arr2 = Array.fill(n - 1)(v)
    val arr = arr1 ++ arr2
    arr

  def generate =
    val arr = randomArray
    val vec = arr.toVector
    (arr, vec)

  @Benchmark
  def solution0Java(blackhole: Blackhole): Unit =
    val (arr, _) = generate
    val r = main.Solution0_java.repeatedNTimes(arr)
    blackhole.consume(r)

  @Benchmark
  def solution2Scala(blackhole: Blackhole): Unit =
    val (_, vec) = generate
    val r = main.Solution2.repeatedNTimes(vec)
    blackhole.consume(r)
```

I pass the instance of `Vector[Int]` to the next solutions in Scala. To avoid
conversion overhead between those data structures I generate both
`Array[Int]` and `Vector[Int]` simultaneously. Also, `generate` produces
worst-case array when the algorithm requires to pass half of an array.

The JMH result:

```
884.907 ±(99.9%) 15.321 ops/ms [Average]
(min, avg, max) = (880.368, 884.907, 891.349), stdev = 3.979
CI (99.9%): [869.587, 900.228] (assumes normal distribution)
```

## Imperative Solution in Scala

An equivalent __imperative solution__ in Scala is as follows:

```scala
object Solution1:
  def repeatedNTimes(a: Array[Int]): Int =
    var i = 0
    while (i < a.size)
      var j = 1
      while (j < 4 && j + i < a.size)
        if (a(i) == a(i + j))
          return a(i)
        j += 1
      i += 1
    throw IllegalArgumentException("a repeated element must exist")
```

The JMH result:

```
861.698 ±(99.9%) 28.190 ops/ms [Average]
(min, avg, max) = (851.490, 861.698, 871.806), stdev = 7.321
CI (99.9%): [833.508, 889.887] (assumes normal distribution)
```

Scala code is `~30 ops/ms` slower than Java code.

## Functional Solutions in Scala

I decompose `while` loops to `tailrec`-optimised recursive calls:

```scala
object Solution2:
  def isValid(v: Vector[Int], i: Int): Int =
    @tailrec
    def check(j: Int = 1): Int =
      if (j >= 4)
        -1
      else if (i + j < v.size && v(i) == v(i + j))
        v(i)
      else
        check(j + 1)
    check()

  @tailrec
  def traverse(v: Vector[Int], i: Int = 0): Int =
    if (i >= v.size)
      throw IllegalArgumentException("a repeated element must exist")
    isValid(v, i) match
      case -1 => traverse(v, i + 1)
      case x  => x

  def repeatedNTimes(v: Vector[Int]): Int =
    traverse(v)
```

The result JMH:

```
373.917 ±(99.9%) 15.016 ops/ms [Average]
(min, avg, max) = (370.203, 373.917, 379.355), stdev = 3.900
CI (99.9%): [358.900, 388.933] (assumes normal distribution)
```

Functions calls drop performance for `~40 ops/ms`.

I return `-1` from `check` because task description implies `A[i] >= 0`.
In general `check` should return `Option[Int]`. Let's measure the cost of
wrapping result to `Option[Int]`:

```scala
object Solution3:
  def isValid(v: Vector[Int], i: Int): Option[Int] =
    @tailrec
    def check(j: Int = 1): Option[Int] =
      if (j >= 4)
        None
      else if (i + j < v.size && v(i + j) == v(i))
        Some(v(i))
      else
        check(j + 1)
    check()

  @tailrec
  def traverse(v: Vector[Int], i: Int = 0): Int =
    if (i >= v.size)
      throw IllegalArgumentException("a repeated element must exist")
    isValid(v, i) match
      case None    => traverse(v, i + 1)
      case Some(x) => x

  def repeatedNTimes(v: Vector[Int]): Int =
    traverse(v)
```

The JMH result:

```
358.069 ±(99.9%) 5.131 ops/ms [Average]
(min, avg, max) = (356.584, 358.069, 360.197), stdev = 1.333
CI (99.9%): [352.938, 363.200] (assumes normal distribution)
```

Return `Option` as a result type costs `~5 ops/ms`.

Let's go further and avoid range check with `Vector.lift`:

```scala
object Solution4:
  def isValid(v: Vector[Int], i: Int): Option[Int] =
    @tailrec
    def check(j: Int = 1): Option[Int] =
      if (j >= 4)
        None
      else if (v.lift(i + j) == Some(v(i)))
        Some(v(i))
      else
        check(j + 1)
    check()

  @tailrec
  def traverse(v: Vector[Int], i: Int = 0): Int =
    if (i >= v.size)
      throw IllegalArgumentException("a repeated element must exist")
    isValid(v, i) match
      case None    => traverse(v, i + 1)
      case Some(x) => x

  def repeatedNTimes(v: Vector[Int]): Int =
    traverse(v)
```

The JMH result:

```
280.777 ±(99.9%) 33.193 ops/ms [Average]
(min, avg, max) = (267.076, 280.777, 288.710), stdev = 8.620
CI (99.9%): [247.584, 313.970] (assumes normal distribution)
```

Introducing `Vector.lift` as a result type costs `~20 ops/ms`.

Let's complete the samples by the most functional and, alas, most ineffective
solution that generates sliding windows:

```scala
object Solution5:
  def repeatedNTimes(v: Vector[Int]): Int =
    val slides = v.sliding(4) ++ (2 to 3).map(v.takeRight(_))
    val resOpt = slides.find(s => s.tail.contains(s.head))
    resOpt match
      case Some(r +: _)   => r
      case Some(_) | None => throw IllegalArgumentException("a repeated element must exist")
```

The overall JMH result is poor:

```
232.184 ±(99.9%) 27.322 ops/ms [Average]
(min, avg, max) = (221.541, 232.184, 241.193), stdev = 7.095
CI (99.9%): [204.862, 259.505] (assumes normal distribution)
```

## Bonus: Solution in Kotlin

```kotlin
package main

class Solution0_kotlin {
  companion object {
    @JvmStatic
    fun repeatedNTimes(a: IntArray): Int {
      for (i in 0..a.size - 1) {
        for (j in 1..3) {
            if (i + j >= a.size)
              continue
            if (a[i] == a[i + j])
              return a[i]
        }
      }
      throw IllegalArgumentException()
    }
  }
}
```

The JMH result:

```
897.820 ±(99.9%) 112.651 ops/ms [Average]
(min, avg, max) = (857.173, 897.820, 938.625), stdev = 29.255
CI (99.9%): [785.169, 1010.472] (assumes normal distribution)
```

## Conclusion

Let's compare the performance according to JMH normal distribution estimations
using Python code as follows:

```python
import numpy as np

def foo(m1: float, s1: float, m2: float, s2: float, q: float=0.1, N: int=1_000_000):
    arr1 = np.random.normal(m1, s1, N)
    arr2 = np.random.normal(m2, s2, N)
    arr1 = np.sort(arr1)[(int)(N*q/100.):-(int)(N*q/100.)]
    arr2 = np.sort(arr2)[(int)(N*q/100.):-(int)(N*q/100.)]
    indexes = zip(
        np.random.randint(low=0, high=arr1.size, size=arr1.size),
        np.random.randint(low=0, high=arr2.size, size=arr2.size),
    )
    arr = [(arr1[a2]-arr2[a1])/arr1[a1] for a1, a2 in indexes]
    arr = np.sort(arr)[(int)(N*q/100.):-(int)(N*q/100.)]
    return arr[0], arr[-1]

# Imperative Scala
foo(m1=884.907, s1=3.979, m2=861.698, s2=7.321, q=0.20)
> (0.0006373412546876295, 0.052451737914490114)
# Scala, tailrec
foo(m1=884.907, s1=3.979, m2=373.917, s2=3.900)
> (0.5530158708621706, 0.6024221862131598)
# Scala, Option[Int]
foo(m1=884.907, s1=3.979, m2=358.069, s2=1.333)
> (0.5771592930614815, 0.613940360993139)
# Scala, vector.lift
foo(m1=884.907, s1=3.979, m2=280.777, s2=8.620)
> (0.6432971995266645, 0.7227139948686705)
# Scala, sliding window
foo(m1=884.907, s1=3.979, m2=232.184, s2=7.095)
> (0.7022175816054018, 0.7736842716482819)
# Kotlin
foo(m1=884.907, s1=3.979, m2=897.820, s2=29.255)
> (-0.10892560898996115, 0.08285290004447453)
```

|                       | Performance, ops/ms            | Diff to Java solution |
|-----------------------|--------------------------------|-----------------------|
| Java solution         | mean = 884.907, stdev = 3.979  | 0%                    |
| Kotlin  solution      | mean = 897.820, stdev = 29.255 | ~0%                   |
| Imperative Scala      | mean = 861.698, stdev = 7.321  | ~0%                   |
| Scala, tailrec        | mean = 373.917, stdev = 3.900  | -55..61%              |
| Scala, Option[Int]    | mean = 358.069, stdev = 1.333  | -57..61%              |
| Scala, vector.lift    | mean = 280.777, stdev = 8.620  | -64..72%              |
| Scala, sliding window | mean = 232.184, stdev = 7.095  | -70..77%              |

Summarizing the results:
- Scala `while`-loops implementation has the same performance as the Kotlin
implementation, but still slower than Java implementation
- tailrec recursive calls cost ~20% of performance
- `Vector.lift` and comparing with `Some` costs ~10% of performance
- returning `Option` as a result costs ~3% of performance
- generating helper vectors with sliding window is a bad idea and costs ~80% of performance
