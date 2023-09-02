---
group: guide
title: 创建一个插件
description: Guide that shows how to create a (retext) plugin
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
tags:
  - plugin
  - retext
published: 2017-05-03
modified: 2020-05-18
---

## 使用unified创建插件

本指南展示了如何创建一个用于检查句子之间空格量的retext插件。
这里的概念也适用于统一的其他语法。

> 卡住了吗?
> 有其他指南的想法吗?
> 看[`support.md`][support].

### Contents

- [使用unified创建插件](#使用unified创建插件)
  - [Contents](#contents)
  - [插件的基础](#插件的基础)
  - [案例](#案例)
  - [设置](#设置)
  - [插件](#插件)
  - [进一步练习](#进一步练习)

### 插件的基础

统一插件从几个方面改变了应用处理器的工作方式。
在本指南中，我们将回顾如何检查语法树。

插件可以包含两个部分:一个**attacher**，这是一个当有人调用`.use`时调用的函数;一个**transformer**，这是一个可选的函数，每次用语法树和虚拟文件处理文件时调用。

在本例中，我们希望检查每个已处理文件的语法树，因此我们指定了一个转换器。

现在您了解了统一插件的基础知识。
来看看我们的案子!

### 案例

在我们开始之前，让我们先概述一下我们想做什么。
假设我们有以下文本文件:

```markdown
One sentence. Two sentences.

One sentence. Two sentences.
```

我们希望在第二段得到一个警告，说明应该使用一个空格而不是两个空格。

在下一步中，我们将编写代码来使用我们的插件。

### 设置

让我们建立一个项目。
创建一个名为`example`的文件夹，输入它，并初始化一个新项目:

```sh
mkdir example
cd example
npm init -y
```

然后确保项目是一个模块, 通过改变`package。json`， `import` 和 `export`可以工作:

```diff
--- a/package.json
+++ b/package.json
@@ -2,6 +2,7 @@
   "name": "example",
   "version": "1.0.0",
   "description": "",
+  "type": "module",
   "main": "index.js",
   "scripts": {
     "test": "echo \"Error: no test specified\" && exit 1"
```

确保`example.md`存在:

```markdown
One sentence. Two sentences.

One sentence. Two sentences.
```

现在，让我们创建一个`example.js`文件，它将处理文本文件并报告发现的任何问题。

```javascript
import fs from 'fs'
import { retext } from 'retext'
import { reporter } from 'vfile-reporter'
import retextSentenceSpacing from './index.js'

const buffer = fs.readFileSync('example.md')

retext()
  .use(retextSentenceSpacing)
  .process(buffer)
  .then((file) => {
    console.error(reporter(file))
  })
```

> 不要忘记`npm install`依赖项 (`retext`, `vfile-reporter`)!

如果您阅读了关于[使用unified][use]的指南，您将看到一些熟悉的语句。
首先，我们加载依赖项，然后读入文件。
我们用即将创建的插件处理该文件，最后报告一个致命错误或任何发现的提示信息。

注意，我们直接依赖于retext。
这是一个公开统一处理器的包，附带解析器和编译器。

当运行我们的示例时(虽然它还不能工作)，我们希望看到第二段的消息，说应该使用一个空格而不是两个空格。

现在我们已经设置好了一切，除了插件本身。
我们将在下一节中这样做。

### 插件

正如我们在插件的基础中所读到的，我们需要一个插件，在我们的例子中还需要一个变压器。
让我们在插件文件`index.js`中创建它们:

```javascript
export default function retextSentenceSpacing() {
  return (tree, file) => {}
}
```

首先，我们需要检查`tree` 是否有模式。
我们可以使用一个实用程序来帮助我们递归地遍历树，即[`unist-util-visit`][visit]。
我们把它加上。

```diff
--- a/index.js
+++ b/index.js
@@ -1,4 +1,9 @@
+import {visit} from 'unist-util-visit'
+
 export default function retextSentenceSpacing() {
   return (tree, file) => {
+    visit(tree, 'ParagraphNode', (node) => {
+      console.log(node)
+    })
   }
 }
```

> Don’t forget to `npm install` the utility!

如果我们现在用Node.js运行我们的例子，如下所示，我们会看到在我们的例子中，访问者被调用了两个段落:

```sh
node example.js
```

```txt
{
  type: 'ParagraphNode',
  children: [
    { type: 'SentenceNode', children: [Array], position: [Object] },
    { type: 'WhiteSpaceNode', value: ' ', position: [Position] },
    { type: 'SentenceNode', children: [Array], position: [Object] }
  ],
  position: {
    start: { line: 1, column: 1, offset: 0 },
    end: { line: 1, column: 29, offset: 28 }
  }
}
{
  type: 'ParagraphNode',
  children: [
    { type: 'SentenceNode', children: [Array], position: [Object] },
    { type: 'WhiteSpaceNode', value: '  ', position: [Position] },
    { type: 'SentenceNode', children: [Array], position: [Object] }
  ],
  position: {
    start: { line: 3, column: 1, offset: 30 },
    end: { line: 3, column: 30, offset: 59 }
  }
}
no issues found
```

这个输出已经表明段落包含两种类型的节点:`SentenceNode` and `WhiteSpaceNode`.
后者是我们想要检查的，但前者很重要，因为我们只警告这个插件(可能是另一个插件)中句子之间的空白。

现在让我们循环遍历每个段落的子段。
只检查句子之间的空白。
我们使用一个小实用程序来检查节点类型: [`unist-util-is`][is].

```diff
--- a/index.js
+++ b/index.js
@@ -1,9 +1,20 @@
 import {visit} from 'unist-util-visit'
+import {is} from 'unist-util-is'

 export default function retextSentenceSpacing() {
   return (tree, file) => {
     visit(tree, 'ParagraphNode', (node) => {
-      console.log(node)
+      const children = node.children
+
+      children.forEach((child, index) => {
+        if (
+          is(children[index - 1], 'SentenceNode') &&
+          is(child, 'WhiteSpaceNode') &&
+          is(children[index + 1], 'SentenceNode')
+        ) {
+          console.log(child)
+        }
+      })
     })
   }
 }
```

> 别忘了`npm install`这个工具!

如果我们现在用Node运行我们的示例，如下所示，我们将看到只有句子之间的空白被记录。

```sh
node example.js
```

```txt
{
  type: 'WhiteSpaceNode',
  value: ' ',
  position: Position {
    start: { line: 1, column: 14, offset: 13 },
    end: { line: 1, column: 15, offset: 14 }
  }
}
{
  type: 'WhiteSpaceNode',
  value: '  ',
  position: Position {
    start: { line: 3, column: 14, offset: 43 },
    end: { line: 3, column: 16, offset: 45 }
  }
}
no issues found
```

最后，让我们为第二个空白添加一个警告，因为它的字符比需要的多。
我们可以使用[`file.message()`][message]将消息与文件关联起来。

```diff
--- a/index.js
+++ b/index.js
@@ -12,7 +12,12 @@ export default function retextSentenceSpacing() {
           is(child, 'WhiteSpaceNode') &&
           is(children[index + 1], 'SentenceNode')
         ) {
-          console.log(child)
+          if (child.value.length !== 1) {
+            file.message(
+              'Expected 1 space between sentences, not ' + child.value.length,
+              child
+            )
+          }
         }
       })
     })
```

如果我们现在最后一次运行我们的示例，我们将看到一条关于问题的消息!

```sh
$ node example.js
3:14-3:16  warning  Expected 1 space between sentences, not 2

⚠ 1 warning
```

### 进一步练习

句与句之间的空格并不适合所有人。
这个插件可以接收首选的空格量，而不是硬编码的`1`。

如果你想警告制表符或句子之间的换行符，也许创建一个插件?

如果你还没有，请查看[学习部分][learn]中的其他文章!

<!--Definitions-->

[support]: https://github.com/unifiedjs/.github/blob/main/support.md
[visit]: https://github.com/syntax-tree/unist-util-visit
[is]: https://github.com/syntax-tree/unist-util-is
[message]: https://github.com/vfile/vfile#vfilemessagereason-position-origin
[learn]: /learn/
[use]: /learn/guide/using-unified/
