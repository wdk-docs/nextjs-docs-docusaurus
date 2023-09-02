---
group: recipe
index: 10
title: TypeScript中的窄节点
description: How to narrow generic `Node` to specific syntax types
tags:
  - typescript
  - unist
  - node
author: Christian Murphy
authorGithub: ChristianMurphy
published: 2020-06-09
modified: 2020-06-15
---

## 在TypeScript里如何缩小“节点”的范围

要使用特定的节点类型或一组节点类型，我们需要[缩小][ts-narrow]它们的类型。
例如，我们可以取一个`Node`，并做一个类型安全检查来获得一个更具体的类型，比如`Link`。
Unified提供了一个实用程序来帮助实现这一点。
TypeScript还提供了一些语言特性。
让我们首先看一下`unist-util-is`。

[`unist-util-is`][unist-is]接受一个`Node`和一个`Test`，并返回测试是否通过。
它可以用作[TypeScript类型谓词][ts-predicate]，当用作条件时(比如在if语句中)，它会告诉TypeScript缩小节点。

以下是一些缩小节点的方法:

```ts
import type { Node, Literal } from 'unist'
import type { List, Blockquote, Strong, Emphasis, Heading } from 'mdast'
import { is, convert } from 'unist-util-is'

// `Node` could come from a plugin, a utility, or be passed into a function
// here we hard code a Node for testing purposes
const node: Node = { type: 'example' }

if (is<List>(node, 'list')) {
  // If we’re here, node is List.
  //
  // `'list'` is compared to `node.type` to make sure they match.
  // `true` means a match, `false` means no match.
  //
  // `<List>` tells TypeScript to ensure `'list'` matches `List.type` and that
  // if `'list'` matches both `node.type` and `List.type`, we know that node is
  // `List` within this if condition.
}

if (is<Strong | Emphasis>(node, ['strong', 'emphasis'])) {
  // If we get here, node is `Strong` or `Emphasis`.

  // If we want even more specific type, we can use a discriminated union
  // <https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions>
  if (node.type === 'emphasis') {
    // If we get here, node is `Emphasis`.
  }
}

if (is<Heading>(node, { type: 'heading', depth: 1 })) {
  // If we get here, node is `Heading`.
  //
  // TypeScript checks that the properties used in the `Test` are valid
  // attributes of `<Heading>`.
  //
  // It does not narrow `node.depth` only be 1, which can be done with
  // `<Heading & {depth: 1}>`.
}

// For advanced use cases, another predicate can be passed to `is`
if (is<Literal>(node, (node: Node): node is Literal => 'value' in node)) {
  // If we get here, node is one of the `Literal` types.
  //
  // Here any comparison function can be used, as long as it is a predicate
  // <https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates>
  // and as long as the predicate and generic match.
  // For example here, `<Literal>` and `is Literal` match.
}

// Reusable predicates can also be created using any `Test`
const isBlockquote = convert<Blockquote>('blockquote')
if (isBlockquote(node)) {
  // If we get here, node is `Blockquote`.
}
```

[unist-is]: https://github.com/syntax-tree/unist-util-is
[ts-narrow]: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
[ts-predicate]: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
