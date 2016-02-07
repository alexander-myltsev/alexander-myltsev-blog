---
layout: post
author: Alexander Myltsev
title: "opTreePF Function"
subtitle: "Crossroad Between RuleDSL and Code Generation"
date: 2015-06-18 08:00:00 +0400
comments: true
header-img: "img/grid-pacman-1680-1050.jpg"
categories:
---

In [previous post]({% post_url 2015-06-02-Meet-parboiled2-A-Macro-Based-PEG-Parsers-Generator %}) was told how [`Parser`][pb-parser] works under the hood. Summarizing, [`rule`][pb-parser-rule] is a macro that replaces AST of rule body with another code.

Let us have parser as follows:

```scala
class SimpleParser(val input: ParserInput) extends Parser {
  def abR = rule { "ab" }
}
```

Before expanding `rule` macro, [`str`][pb-str] implicit would be applied to a nested string `"ab"` as everything inside rule body should be of type `Rule`:

```scala
  def abR = rule { SimpleParser.this.str("ab") }
```

And then it would be parsed by [`opTreePF`][pb-optree-pf]:

```scala
  val opTreePF: PartialFunction[Tree, Tree] = {
    case q"$a.this.str($s)" => ???
      // `a` would be `SimpleParser` and `s` would be `"ab"` in pattern-matched body
  }
```

### Some Options to Implement `str` Rule

`opTreePF` returns piece of code that would work at runtime. `str` might be implemented in several ways. Naive implementation is as follows:

```scala
case q"$a.this.str($s)" => q“input.sliceString(cursor, cursor + $s.lengith) == $s”
```

A more effective imperative implementation is that one:

```scala
case q"$a.this.str($s)" => q“““
  var ix = 0
  while (ix < $s.length && cursorChar == $s.charAt(ix)) {
    ix += 1
    __advance()
  }
  ix == $s.length ”””
```

Optimization when `s` is empty string could be applied as follows:

```scala
case q"$a.this.str($s)" => q“““
  if ($s.isEmpty) true
  else {
    var ix = 0
    while (ix < $s.length && cursorChar == $s.charAt(ix)) {
      ix += 1
      __advance()
    }
    ix == $s.length
  } ”””
```

As well as specialization to `s` if it is a literal constant:

```scala
def unroll(s: String, ix: Int = 0): Tree =
  if (ix < s.length) { q“““
    if (cursorChar == ${s charAt ix}) {
      __advance()
      ${unroll(s, ix + 1)}
    } else false
  ””” } else q“true”

case q"$a.this.str($s)" => s match {
  case Literal(Constant(sc)) => unroll(sc)
  case _ => q“““
    if ($s.isEmpty) true
    else {
      var ix = 0
      while (ix < $s.length && cursorChar == $s.charAt(ix)) {
        ix += 1
        __advance()
      }
      ix == $s.length
    } ”””
```

### Real-World Implementation

`parboiled2` uses last implementation. Moreover it defines [`opTreePF`][pb-optree-pf] signature differently:

```scala
val opTreePF: PartialFunction[Tree, OpTree]
```

where [`OpTree`][pb-optree] and its descendants implements code generation logic:

```scala
sealed abstract class OpTree {
  def ruleFrame: Tree

  def renderRule(ruleName: Tree): Tree = q“““
    // split out into separate method so as to not double the rule method size
    // which would effectively decrease method inlining by about 50%
    def wrapped: Boolean = ${render(wrapped = true, ruleName)}
    val matched =
      if (__collectingErrors) wrapped
      else ${render(wrapped = false)}
    // "matched" boolean is encoded as 'ruleResult ne null'
    if (matched) org.parboiled2.Rule else null”””

  // renders a Boolean Tree
  def render(wrapped: Boolean, ruleName: Tree = Literal(Constant(""))): Tree =
    if (wrapped) q“““
      try ${renderInner(wrapped)}
      catch {
        case e: org.parboiled2.Parser.CollectingRuleStackException ⇒
          e.save(org.parboiled2.RuleFrame($ruleFrame, $ruleName))
      }”””
    else renderInner(wrapped)

  // renders a Boolean Tree
  protected def renderInner(wrapped: Boolean): Tree
}
```

Important thing to notice is that `OpTree` generates same method both for error-less execution and error collection. If `__collectingErrors` flag is set then `renderInner` code is wrapped in `try/catch` block that saves context information where parsing failed.

### Summary

In this blog article we learned what `opTreePF` is, and its role on path from RuleDSL to code generation.

[pb-parser]: https://github.com/sirthias/parboiled2/blob/v2.0.1/parboiled-core/src/main/scala/org/parboiled2/Parser.scala#L26-L27
[pb-parser-rule]: https://github.com/sirthias/parboiled2/blob/v2.0.1/parboiled-core/src/main/scala/org/parboiled2/Parser.scala#L40
[pb-str]: https://github.com/sirthias/parboiled2/blob/v2.0.1/parboiled-core/src/main/scala/org/parboiled2/RuleDSLBasics.scala#L34-L35
[pb-optree-pf]: https://github.com/sirthias/parboiled2/blob/v2.0.1/parboiled-core/src/main/scala/org/parboiled2/support/OpTreeContext.scala#L65
[pb-optree]: https://github.com/sirthias/parboiled2/blob/v2.0.1/parboiled-core/src/main/scala/org/parboiled2/support/OpTreeContext.scala#L26-51