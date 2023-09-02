---
group: recipe
index: 3
title: 树遍历
description: How to do tree traversal (also known as walking or visiting a tree)
tags:
  - unist
  - tree
  - traverse
  - walk
  - visit
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
published: 2019-12-23
modified: 2020-06-14
---

## 如何在树上行走

### Contents

- [如何在树上行走](#如何在树上行走)
  - [Contents](#contents)
  - [树遍历](#树遍历)
  - [设置](#设置)
  - [遍历树](#遍历树)
  - [访问某一种节点](#访问某一种节点)

### 树遍历

**树遍历**是处理语法树时的常见任务。
这里的术语树是指节点及其所有后代(其中的所有节点)。
遍历意味着在每个节点停下来做一些事情。
因此，树遍历意味着对树中的每个节点都做一些操作。

树遍历通常也被称为“遍历树”或“访问树”。

要了解更多，请继续阅读，但当使用unist(unified的树)时，您可能需要以下两种:

- [`unist-util-visit`][visit]
- [`unist-util-visit-parents`][visit-parents]

### 设置

很高兴你还在这里!
假设我们有以下HTML片段，在一个名为`example.html`的文件中:

```html
<p>
  <!-- A comment. -->
  Some <strong>strong importance</strong>, <em>emphasis</em>, and a dash of
  <code>code</code>.
</p>
```

您可以用下面的代码解析它 (使用 [`unified`][unified] 和 [`rehype-parse`][rehype-parse]):

```js
import fs from 'node:fs'
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'

const doc = fs.readFileSync('example.html')

const tree = unified().use(rehypeParse, { fragment: true }).parse(doc)

console.log(tree)
```

这将产生(为简洁起见，忽略位置信息):

```js
{
  type: 'root',
  children: [
    {
      type: 'element',
      tagName: 'p',
      properties: {},
      children: [
        { type: 'text', value: '\n  ' },
        { type: 'comment', value: ' A comment. ' },
        { type: 'text', value: '\n  Some ' },
        {
          type: 'element',
          tagName: 'strong',
          properties: {},
          children: [ { type: 'text', value: 'strong importance' } ]
        },
        { type: 'text', value: ', ' },
        {
          type: 'element',
          tagName: 'em',
          properties: {},
          children: [ { type: 'text', value: 'emphasis' } ]
        },
        { type: 'text', value: ', and a dash of\n  ' },
        {
          type: 'element',
          tagName: 'code',
          properties: {},
          children: [ { type: 'text', value: 'code' } ]
        },
        { type: 'text', value: '.\n' }
      ]
    },
    { type: 'text', value: '\n' }
  ],
  data: { quirksMode: false }
}
```

设置好之后，我们就可以遍历树了。

### 遍历树

一个有用的工具是[`unist-util-visit`][visit]，它是这样工作的:

```js
import { visit } from 'unist-util-visit'

// …

visit(tree, (node) => {
  console.log(node.type)
})
```

```txt
root
element
text
comment
text
element
text
text
element
text
text
element
text
text
text
```

我们遍历整个树，对于每个节点，我们输出它的`type`。

### 访问某一种节点

要`visit`某一`type`的节点，通过[`unist-util-visit`][visit]测试，如下所示:

```js
import { visit } from 'unist-util-visit'

// …

visit(tree, 'element', (node) => {
  console.log(node.tagName)
})
```

```txt
p
strong
em
code
```

你也可以自己做。
上述工作原理与:

```js
visit(tree, (node) => {
  if (node.type === 'element') {
    console.log(node.tagName)
  }
})
```

但是通过`visit`的测试可以更高级，例如下面访问不同类型的节点。

```js
visit(tree, ['comment', 'text'], (node) => {
  console.log([node.value])
})
```

```txt
[ '\n  ' ]
[ ' A comment. ' ]
[ '\n  Some ' ]
[ 'strong importance' ]
[ ', ' ]
[ 'emphasis' ]
[ ', and a dash of\n  ' ]
[ 'code' ]
[ '.\n' ]
[ '\n' ]
```

阅读更多关于[`unist-util-visit`][visit]的信息。

[visit]: https://github.com/syntax-tree/unist-util-visit
[visit-parents]: https://github.com/syntax-tree/unist-util-visit-parents
[unified]: https://github.com/unifiedjs/unified
[rehype-parse]: https://github.com/rehypejs/rehype/tree/HEAD/packages/rehype-parse
