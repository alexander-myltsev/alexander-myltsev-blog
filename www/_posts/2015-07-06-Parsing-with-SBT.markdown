---
layout: post
author: Alexander Myltsev
title: "Interactive Parsing"
subtitle: "Joggling parboiled2 Parsers with sbt"
date: 2015-07-06 08:00:00 +0400
comments: true
header-img: "img/nix-terminal.png"
categories:
---

Sometimes it is good to use facilities to rapid parboiled2 parsers prototyping right in the console. Let's use `sbt` of version `0.13.8`:

```bash
$ mkdir project
$ echo "sbt.version=0.13.8" > project/build.properties
```

When minimal project is set up, `sbt` can be launched: 

```bash
$ sbt
> sbt
```

Let's setup Scala compiler version compatible with `parboiled2`, and add dependency to it. And then run Scala interpreter with `console` command:

```bash
sbt> set scalaVersion := "2.11.7"
sbt> set libraryDependencies += "org.parboiled" %% "parboiled" % "2.1.0"
sbt> console
scala>
```

We need to import namespace: 

```bash
scala> import org.parboiled2._
```

Now console is ready to accept parser implementation. `parboiled2` parsers are easier to understand in infix notation. On the other hand this make hard to paste parser code to interpreter line by line. We would you `:paste` command for that:

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

class Calculator1(val input: ParserInput) extends Parser {
  def InputLine = rule { Expression ~ EOI }

  def Expression: Rule1[Int] = rule {
    Term ~ zeroOrMore(
      '+' ~ Term ~> ((_: Int) + _) | 
      '-' ~ Term ~> ((_: Int) - _))
  }

  def Term = rule {
    Factor ~ zeroOrMore(
      '*' ~ Factor ~> ((_: Int) * _) | 
      '/' ~ Factor ~> ((_: Int) / _))
  }

  def Factor = rule { Number | Parens }

  def Parens = rule { '(' ~ Expression ~ ')' }

  def Number = rule { capture(Digits) ~> (_.toInt) }

  def Digits = rule { oneOrMore(CharPredicate.Digit) }
}

// Exiting paste mode, now interpreting.

defined class Calculator1
```

Now having `Calculator1` defined in the environment we could parse a string with instance of it:

```scala
scala> new Calculator1("1+2*3").InputLine.run()
res0: scala.util.Try[Int] = Success(7)

scala> new Calculator1("a").InputLine.run()
res1: scala.util.Try[Int] = Failure(ParseError(Position(0,1,1), Position(0,1,1), <2 traces>))
```

`parboiled2` utilities could be used for `res1` pretty-formatting as follows: 

```scala
scala> import scala.util.Failure
import scala.util.Failure

scala> val calc = new Calculator1("a")
calc: Calculator1 = Calculator1@5ed32ee0

scala> val Failure(e: ParseError) = calc.InputLine.run()
e: org.parboiled2.ParseError = ParseError(Position(0,1,1), Position(0,1,1), <2 traces>)

scala> calc.formatError(e)
res2: String =
Invalid input 'a', expected Number or Parens (line 1, column 1):
a
^
```

Happy parsing with `sbt`!
