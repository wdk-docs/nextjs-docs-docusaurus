---
group: recipe
index: 7
title: 移除节点
description: 如何在任意unist树中移除节点
tags:
  - node
  - tree
  - remove
  - delete
  - traverse
  - walk
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
published: 2020-06-15
modified: 2020-06-15
---

## 如何移除节点

一旦找到了要删除的节点(参见[树遍历][tree-traversal])，就可以删除它们了。

### Contents

- [如何移除节点](#如何移除节点)
  - [Contents](#contents)
  - [先决条件](#先决条件)
  - [移除节点](#移除节点)
  - [用子节点替换节点](#用子节点替换节点)

### 先决条件

- [树遍历][tree-traversal]
  — 介绍如何使用`unist-util-visit`遍历树并查找特定节点

### 移除节点

在大多数情况下，删除节点必须首先找到它们(参见[树遍历][tree-traversal])，所以假设我们已经有一些代码来查找所有`emphasis`节点。

首先，我们的`example.md`文件:

```markdown
Some text with _emphasis_.

Another paragraph with **importance** (and _more emphasis_).
```

还有一个脚本, `example.js`:

```js
import fs from 'node:fs'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import { visit } from 'unist-util-visit'

const doc = fs.readFileSync('example.md')

const tree = unified().use(remarkParse).parse(doc)

visit(tree, 'emphasis', function (node) {
  console.log(node)
})
```

现在，运行`node example`会得到(为了简洁，忽略了位置):

```js
{
  type: 'emphasis',
  children: [ { type: 'text', value: 'emphasis', position: [Object] } ]
}
{
  type: 'emphasis',
  children: [ { type: 'text', value: 'more emphasis', position: [Object] } ]
}
```

如上面的日志所示，节点是对象。
每个节点都在另一个节点的`children`属性的数组中。
换句话说，要删除一个节点，它必须从它的父节点的`children`中删除。

接下来的问题是从数组中删除一个值。
可以使用标准的JavaScript [Array函数][Array]:即[`splice`][splice]。

我们有强调节点，但没有它们的父节点，也没有它们在父节点的`children`字段中的位置。
幸运的是，给`visit`的函数不仅得到`node`，还得到`index`和`parent`:

```diff
+++ b/example.js
@@ -7,6 +7,6 @@ const doc = fs.readFileSync('example.md')

 const tree = unified().use(remarkParse).parse(doc)

-visit(tree, 'emphasis', function (node) {
-  console.log(node)
+visit(tree, 'emphasis', function (node, index, parent) {
+  console.log(node.type, index, parent.type)
 })
```

Yields:

```txt
emphasis 1 paragraph
emphasis 3 paragraph
```

`parent`是对`node`的父节点的引用，`index`是`node`在`parent`的`children` `node`中的位置。
有了这些信息和“splice”，我们现在可以删除强调节点了:

```diff
--- a/example.js
+++ b/example.js
@@ -8,5 +8,8 @@ const doc = fs.readFileSync('example.md')
 const tree = unified().use(remarkParse).parse(doc)

 visit(tree, 'emphasis', function (node, index, parent) {
-  console.log(node.type, index, parent.type)
+  parent.children.splice(index, 1)
+  // (Note: this is buggy, see next section)
 })
+
+console.log(tree)
```

Yields:

```js
{
  type: 'root',
  children: [
    {
      type: 'paragraph',
      children: [
        {type: 'text', value: 'Some text with '},
        {type: 'text', value: '.'}
      ]
    },
    {
      type: 'paragraph',
      children: [
        {type: 'text', value: 'Another paragraph with '},
        {type: 'strong', children: [Array]},
        {type: 'text', value: ' (and '},
        {type: 'text', value: ').'}
      ]
    }
  ]
}
```

这看起来很棒，但是要小心bug。
我们现在正在改变树，同时遍历它。
这可能会导致bug和性能问题。

当改变树时，在大多数情况下，你应该发出`visit`它应该如何继续的信号。
关于如何发出下一步操作的信号的更多信息，请参见[`unist-util-visit-parents`][visit-parents]。

在这种情况下，我们不希望遍历被删除的节点(我们希望跳过它)。
我们想继续处理这个节点它现在在被移走的节点所在的位置。
要做到这一点:从`visitor`返回该信息:

```diff
--- a/example.js
+++ b/example.js
@@ -1,15 +1,15 @@
 import fs from 'node:fs'
 import {unified} from 'unified'
 import remarkParse from 'remark-parse'
-import {visit} from 'unist-util-visit'
+import {visit, SKIP} from 'unist-util-visit'

 const doc = fs.readFileSync('example.md')

 const tree = unified().use(remarkParse).parse(doc)

 visit(tree, 'emphasis', function (node, index, parent) {
   parent.children.splice(index, 1)
-  // (Note: this is buggy, see next section)
+  // Do not traverse `node`, continue at the node *now* at `index`.
+  return [SKIP, index]
 })

 console.log(tree)
```

这将产生与之前相同的输出，但不再有错误。
很好，我们现在可以删除节点了!

### 用子节点替换节点

还有一件事让这个例子更有用:与其去掉`emphasis`*和*它的子元素，不如用它的子元素*代替*`emphasis`更有意义。

要做到这一点，我们可以这样做:

```diff
--- a/example.js
+++ b/example.js
@@ -8,7 +8,7 @@ const doc = fs.readFileSync('example.md')
 const tree = unified().use(remarkParse).parse(doc)

 visit(tree, 'emphasis', function (node, index, parent) {
-  parent.children.splice(index, 1)
+  parent.children.splice(index, 1, ...node.children)
   // Do not traverse `node`, continue at the node *now* at `index`.
   return [SKIP, index]
 })
```

Yields:

```js
{
  type: 'root',
  children: [
    {
      type: 'paragraph',
      children: [
        {type: 'text', value: 'Some text with '},
        {type: 'text', value: 'emphasis'},
        {type: 'text', value: '.'}
      ]
    },
    {
      type: 'paragraph',
      children: [
        {type: 'text', value: 'Another paragraph with '},
        {type: 'strong', children: [Array]},
        {type: 'text', value: ' (and '},
        {type: 'text', value: 'more emphasis'},
        {type: 'text', value: ').'}
      ]
    }
  ]
}
```

[tree-traversal]: /tree-traversal/
[array]: https://developer.mozilla.org/docs/JavaScript/Reference/Global_Objects/Array
[splice]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice
[visit-parents]: https://github.com/syntax-tree/unist-util-visit-parents#visittree-test-visitor-reverse
