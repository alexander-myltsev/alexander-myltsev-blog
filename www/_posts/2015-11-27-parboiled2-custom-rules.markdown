---
layout: post
author: Alexander Myltsev
title: "parboiled2 Custom Rules"
subtitle: ""
date: 2015-12-15 08:00:00 +0400
comments: true
tags: [scala, parboiled2]
---

[There are use cases]({% post_url 2016-01-08-Capturing-Positions %}) when developing custom low-level DSL rules might be very effective. It is possible without changing source code of `parboiled2`. And requirements to achive that are as follows:

First, we need a `RuleDSLActionsCustom` trait that contains rules declarations:

```scala
trait RuleDSLActionsCustom {
  @compileTimeOnly("Calls to `debug` must be inside `rule` macro")
  def debug(msg: String): Rule0 = `n/a`
}
```

Second, we need to mix it in a `class ParserCustom`:

```scala
abstract class ParserCustom extends RuleTypes
      with RuleDSLBasics with RuleDSLCombinators
      with RuleDSLActions with RuleDSLActionsCustom
```

Next we need to deliver `debug` rule to a `OpTreeContextCustom` class: it should extend [`OpTreeContext.opTreePF` function]({% post_url 2015-06-18-opTreePF-Function %}) with proper implementation of code generation for `debug` rule:

```scala
class OpTreeContextCustom[OpTreeCtx <: reflect.macros.blackbox.Context](val c1: OpTreeCtx) extends OpTreeContext[OpTreeCtx](c1) {
  import c.universe._

  override def opTreePF: PartialFunction[Tree, OpTree] = {
    case q"$a.this.debug($msg)" => Debug(OpTree(arg))
    case t => super.opTreePF(t)
  }

  case class Debug(msg: Tree) extends OpTree {
    def renderInner(wrapped: Boolean): Tree = q"println($msg)"
  }
```

And here is the place where code starts to smell. The complication arises from the implementation currently not designed for such extensions. `Parser` [tighgly depends on](https://github.com/sirthias/parboiled2/blob/release-2.1/parboiled-core/src/main/scala/org/parboiled2/Parser.scala#L37-L46) `ParserMacros`, that in its turn [tightly depends on](https://github.com/sirthias/parboiled2/blob/release-2.1/parboiled-core/src/main/scala/org/parboiled2/Parser.scala#L661) `OpTreeContext`. It is still possible to extend those things ([like it experimentally done for `CapturePos`](https://github.com/GlobalNamesArchitecture/gnparser/commit/3db444c3aec0e30de25a980941446d204c13f1c8) for custom `debug` rule. Overall, it all implies that custom extension should copy-paste almost entire inner implementation of `ParserMacros` and `OpTreeContext`, and even worse keep it updated with original upstream to properly merge updates.

The approach for custom rule implementation is useless. It is possible though to redesign `parboiled2` inner API to make it easier to extend `Parser` with custom rules. But, [it would make much harder to evolve that API](https://groups.google.com/forum/#!searchin/parboiled-user/custom/parboiled-user/RlnEDa5DuuY/YCjQCkw1pOYJ). Practically, it is easier [to keep a simple patch to a fork]({% post_url 2016-01-08-Capturing-Positions %}) then to maintain classes derived from `parboiled2` inner API.
