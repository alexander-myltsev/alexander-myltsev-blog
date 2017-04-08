---
layout: post
author: Alexander Myltsev
title: "Capturing Positions in parboiled2"
subtitle: ""
date: 2016-01-08 08:00:00 +0400
comments: true
tags: [scala, parboiled2]
---

It is a common task to track positions of recognized AST nodes. `parboiled2` has no built-in facilities to do that. The reason is that [different applications have different needs with regard to position tracking](https://github.com/sirthias/parboiled2/pull/152). Still, there are ways to capture positions.

One way is to compose capturing rules with existing DSL rules (originally proposed at [parboiled user group](https://groups.google.com/d/msg/parboiled-user/Aoxz1E9bwPM/xORI_eZNrHEJ)):

```scala
    var nodes = Map.empty[AstNode, Position]

    def someAstNode = rule {
      startAstNode ~ ... /* match AST node syntax */ ... ~> MyAstNode ~ endAstNode ~ restOfRule
    }

    def startAstNode = rule { push(cursor) }

    def endAstNode[A <: AstNode]: Rule[Int :: A :: HNil, A :: HNil] = rule {
      run { (start: Int, node: A) =>
        val pos = Position(start, cursor)
        nodes = nodes.updated(node, pos)
        node
      }
    }
```

The problem with approach is that `nodes` structure together with both rules do not have properties of `value stack`. For example, if `restOfRule` fails then something (not clear how to implement) should track all stored AST nodes in `nodes` to clean it up later.

It would be useful if `parboiled2` has facilities to add custom rules to DSL. Alas, `parboiled2` has [no easy way to do it]({% post_url 2015-11-27-parboiled2-custom-rules %}). I know only one way to make it effectively: to patch source code. Hopefully, there are few steps to do it that are as follows:

1. add DSL entity to [`RuleDSLActions.scala`](https://github.com/alexander-myltsev/parboiled2/blob/feature-capture_pos/parboiled-core/src/main/scala/org/parboiled2/RuleDSLActions.scala#L26-L32)
2. add quasiquote AST match to [`OpTreeContext.scala`](https://github.com/alexander-myltsev/parboiled2/blob/feature-capture_pos/parboiled-core/src/main/scala/org/parboiled2/support/OpTreeContext.scala#L113)
3. implement code rendering at the same [`OpTreeContext.scala`](https://github.com/alexander-myltsev/parboiled2/blob/feature-capture_pos/parboiled-core/src/main/scala/org/parboiled2/support/OpTreeContext.scala#L535-L544)

Summary of changes to capture positions is [publicly available](https://github.com/alexander-myltsev/parboiled2/compare/bf2ac7b4ee4510841c36cf9f8b143e3bd797fbcf...alexander-myltsev:feature-capture_pos).
