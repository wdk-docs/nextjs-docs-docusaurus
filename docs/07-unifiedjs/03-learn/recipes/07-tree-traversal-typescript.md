---
group: recipe
index: 9
title: 用TypeScript遍历树
description: How to do tree traversal in TypeScript
tags:
  - typescript
  - unist
  - tree
  - traverse
author: Christian Murphy
authorGithub: ChristianMurphy
published: 2020-06-09
modified: 2020-06-11
---

## 用TypeScript遍历树

📓 在阅读本节之前，请先阅读[JavaScript中的树遍历介绍](/learn/recipe/tree-traversal/)。

使用unified时的一个常见任务是遍历树以找到某些节点，然后对它们进行处理(通常是验证或转换它们)。
可以使用几个类型安全的统一实用程序来帮助实现这一点。

### `unist-util-visit`

[`unist-util-visit`](https://github.com/syntax-tree/unist-util-visit#readme)接受一个语法树、一个`Test`和一个回调。
对于树中通过`Test`的每个节点调用回调。

例如，如果我们想要增加markdown文档中所有标题的标题级别:

```ts
import { remark } from 'remark'
import type { Root } from 'mdast'
import { visit } from 'unist-util-visit'

const markdownFile = await remark()
  .use(() => (mdast: Root) => {
    visit(
      mdast,
      // Check that the Node is a heading:
      'heading',
      (node) => {
        // The types know `node` is a heading.
        node.depth += 1
      },
    )
  })
  .process('## Hello, *World*!')

console.log(markdownFile.toString())
```

或者如果我们想让markdown文档中的所有有序列表无序:

```ts
import { remark } from 'remark'
import type { Root } from 'mdast'
import { visit } from 'unist-util-visit'

const markdownFile = await remark()
  .use(() => (mdast: Root) => {
    visit(
      mdast,
      // Check that the Node is a list:
      'list',
      (node) => {
        if (node.ordered) {
          // The types know `node` is an ordered list.
          node.ordered = false
        }
      },
    )
  })
  .process('1. list item')

console.log(markdownFile.toString())
```

### `unist-util-visit-parents`

有时需要知道节点的祖先(它的所有父节点)。
[`unist-util-visit-parents`](https://github.com/syntax-tree/unist-util-visit-parents)类似于`unist-util-visit`，但包含所有父节点的列表。

例如，如果我们想检查是否所有markdown `ListItem`都在`List`中，我们可以:

```ts
import remark from 'remark'
import type { Root, ListItem } from 'mdast'
import { visitParents } from 'unist-util-visit-parents'

remark()
  .use(() => (mdast: Root) => {
    visitParents(mdast, 'listItem', (listItem, parents) => {
      // The types know `listItem` is a list item, and that `parents` are mdast
      // parents.
      if (!parents.some((parent) => parent.type === 'list')) {
        console.warn('listItem is outside a list')
      }
    })
  })
  .process('1. list item')
```

### `unist-util-select`

有时CSS选择器比几个(嵌套的)if/else语句更容易阅读。
[`unist-util-select`](https://github.com/syntax-tree/unist-util-select)允许您这样做。
例如，如果我们想找到所有的`Paragraph`，在某个地方的`Blockquote`，我们可以:

```ts
import remark from 'remark'
import type { Root } from 'mdast'
import { selectAll } from 'unist-util-select'

remark()
  .use(() => (mdast: Root) => {
    const matches = selectAll('blockquote paragraph', mdast)
    console.log(matches)
  })
  .process('1. list item')
```
