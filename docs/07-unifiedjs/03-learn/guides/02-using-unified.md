---
group: guide
index: 2
title: 使用 unified
description: Guide that delves into transforming Markdown to HTML
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
tags:
  - use
  - transform
  - remark
  - rehype
published: 2017-05-03
modified: 2020-06-14
---

## 使用 unified

本指南深入探讨了如何使用unified将Markdown文件转换为HTML。
它还将展示如何生成目录，以及如何检查散文。

> 卡住了吗?
> 有其他指南的想法吗?
> 看到 [`support.md`][support].

### Contents

- [使用 unified](#使用-unified)
  - [Contents](#contents)
  - [树的转换](#树的转换)
  - [插件](#插件)
  - [报告](#报告)
  - [检查散文](#检查散文)
  - [进一步练习](#进一步练习)

### 树的转换

对于本例，我们将从Markdown内容开始，然后转换为HTML。
我们需要一个Markdown解析器和一个HTML stringifier。
相关的项目分别是[`remark-parse`][parse]和[`rehype-stringify`][stringify]。
要在两种语法之间进行转换，我们将使用[`remark-rehype`][remark-rehype].
最后，我们将使用unified本身将这些粘合在一起，然后为流[`unified-stream`][unified-stream].

首先建立一个项目。
创建一个名为`example`的文件夹，输入它，并初始化一个新项目:

```sh
mkdir example
cd example
npm init -y
```

然后确保项目是一个模块，这样`import`和`export`就可以工作了，通过修改`package.json`:

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

现在让我们使用[npm][]安装所需的依赖项，它是与[Node][]捆绑在一起的。

```sh
npm install unified unified-stream remark-parse remark-rehype rehype-stringify
```

现在创建一个Markdown文件，`example.md`，我们要对它进行转换。

```markdown
# Hello World

## Table of Content

## Install

A **example**.

## Use

More `text`.

## License

MIT
```

然后创建`index.js`。
它将把Markdown转换为HTML。
它连接起来从标准输入读取并向标准输出写入。

```javascript
import { stream } from 'unified-stream'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

const processor = unified()
  .use(remarkParse)
  .use(remarkRehype)
  .use(rehypeStringify)

process.stdin.pipe(stream(processor)).pipe(process.stdout)
```

现在使用[Node][]运行我们的脚本(这使用您的Shell读取`example.md`并写入`example.html`):

```sh
node index.js < example.md > example.html
```

…给我们一个`example.html`文件，如下所示:

```html
<h1>Hello World</h1>
<h2>Table of Content</h2>
<h2>Install</h2>
<p>A <strong>example</strong>.</p>
<h2>Use</h2>
<p>More <code>text</code>.</p>
<h2>License</h2>
<p>MIT</p>
```

> 注意[`remark-rehype`][remark-rehype]不处理Markdown中的HTML。
> 如果你打算这么做，你需要[`rehype-raw`][rehype-raw]。

🎉
漂亮的!
它现在还做不了什么，但我们会做到的。
在下一节中，我们将通过介绍插件使其更有用。

### 插件

我们仍然缺少一些东西，特别是目录和适当的HTML文档结构。

对于前者，我们可以使用[`remark-slug`][slug]和[`remark-toc`][toc]，而对于后者，我们可以使用[`rehype-document`][document]。

```sh
npm install remark-slug remark-toc rehype-document
```

现在让我们通过修改`index.js`文件来使用这两个:

```diff
--- a/index.js
+++ b/index.js
@@ -1,12 +1,18 @@
 import {stream} from 'unified-stream'
 import {unified} from 'unified'
 import remarkParse from 'remark-parse'
+import remarkSlug from 'remark-slug'
+import remarkToc from 'remark-toc'
 import remarkRehype from 'remark-rehype'
+import rehypeDocument from 'rehype-document'
 import rehypeStringify from 'rehype-stringify'

 const processor = unified()
   .use(remarkParse)
+  .use(remarkSlug)
+  .use(remarkToc)
   .use(remarkRehype)
+  .use(rehypeDocument, {title: 'Contents'})
   .use(rehypeStringify)

 process.stdin.pipe(stream(processor)).pipe(process.stdout)
```

我们将选项传递给 `rehype-document`.
在这种情况下，我们使用它来确保在我们的`<head>`中得到一个正确的`<title>`元素，这是HTML规范所要求的。
`rehype-document`可以接受更多选项，例如使用哪个语言标签。
这些在它的[`readme.md`][document]中有详细描述。
许多其他插件也接受选项，因此请务必通读它们的文档以了解更多信息。

> 注意，remark插件在Markdown树中工作，rehype插件在HTML树中工作。
> 将`.use`调用放在正确的位置是很重要的。

现在，当像之前一样运行脚本时，我们会得到下面的`example.html`文件:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Contents</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <h1 id="hello-world">Hello World</h1>
    <h2 id="table-of-content">Table of Content</h2>
    <ul>
      <li><a href="#install">Install</a></li>
      <li><a href="#use">Use</a></li>
      <li><a href="#license">License</a></li>
    </ul>
    <h2 id="install">Install</h2>
    <p>A <strong>example</strong>.</p>
    <h2 id="use">Use</h2>
    <p>More <code>text</code>.</p>
    <h2 id="license">License</h2>
    <p>MIT</p>
  </body>
</html>
```

> 您可能注意到文档的格式不太好。
> 不过有一个插件!
> 请随意添加[`rehype-format`][rehype-format]到插件，在`doc`下面!

💯
你太棒了!
这越来越有用了，对吧?

在下一节中，我们将为创建报告奠定基础。

### 报告

在我们检查一些散文之前(是的，我们就要到了)，我们将首先切换我们的`index.js`文件来打印一个漂亮的报告(我们将在下一节中填充它)。

我们可以使用[`to-vfile`][to-vfile]从文件系统读写虚拟文件，我们可以使用[`vfile-reporter`][reporter]报告与这些文件相关的消息。
让我们来安装它们。

```sh
npm install to-vfile vfile-reporter
```

…现在将stdin/stdout从我们的示例中解绑定，并使用文件系统，如下所示:

```diff
--- a/index.js
+++ b/index.js
@@ -1,4 +1,5 @@
-import {stream} from 'unified-stream'
+import {readSync, writeSync} from 'to-vfile'
+import {reporter} from 'vfile-reporter'
 import {unified} from 'unified'
 import remarkParse from 'remark-parse'
 import remarkSlug from 'remark-slug'
@@ -15,4 +16,15 @@ const processor = unified()
   .use(rehypeDocument, {title: 'Contents'})
   .use(rehypeStringify)

-process.stdin.pipe(stream(processor)).pipe(process.stdout)
+processor
+  .process(readSync('example.md'))
+  .then(
+    (file) => {
+      console.error(reporter(file))
+      file.extname = '.html'
+      writeSync(file)
+    },
+    (error) => {
+      throw error
+    }
+  )
```

如果我们现在自己运行脚本，没有shell重定向，我们会得到一个报告，显示一切正常:

```sh
$ node index.js
example.md: no issues found
```

但一切都不好，Markdown有个错别字!
下一节将展示如何通过添加retext来检测散文错误。

### 检查散文

我确实注意到这里有一个错别字，所以让我们检查一些散文，以防止将来发生这种情况。
我们可以使用retext及其生态系统来进行自然语言解析。
当我们用英语写作时，我们使用[`retext-english`][English]来专门解析英语的自然语言。
在我们的`example.md`文件中的问题是，它有`a example`而不是`an example`，这是方便地检查[`retext-indefinite-article`][indefinite-article]。
为了从标记过渡到散文，我们将使用[`remark-retext`][remark-retext]。
首先，让我们安装这些依赖项。

```sh
npm install remark-retext retext-english retext-indefinite-article
```

…然后像这样改变我们的`index.js`:

```diff
--- a/index.js
+++ b/index.js
@@ -4,12 +4,16 @@ import {unified} from 'unified'
 import remarkParse from 'remark-parse'
 import remarkSlug from 'remark-slug'
 import remarkToc from 'remark-toc'
+import remarkRetext from 'remark-retext'
+import retextEnglish from 'retext-english'
+import retextIndefiniteArticle from 'retext-indefinite-article'
 import remarkRehype from 'remark-rehype'
 import rehypeDocument from 'rehype-document'
 import rehypeStringify from 'rehype-stringify'

 const processor = unified()
   .use(remarkParse)
+  .use(remarkRetext, unified().use(retextEnglish).use(retextIndefiniteArticle))
   .use(remarkSlug)
   .use(remarkToc)
   .use(remarkRehype)
```

如代码所示，`remark-retext`接收另一个`unified`中间件管道。
自然语言管道。
该插件将使用给定管道的解析器转换原始语法(Markdown)。
然后，它将在自然语言语法树上运行附加的插件。

现在，当最后一次运行脚本时:

```sh
$ node index.js
example.md
  7:1-7:2  warning  Use `An` before `example`, not `A`  retext-indefinite-article  retext-indefinite-article

⚠ 1 warning
```

…我们会得到有用的信息。

💃
你已经建立了一个非常酷的系统，做得很好!
这是一个总结，请查看下一节以获得更多的练习和资源。

### 进一步练习

最后，请查看[retext][retext-plugins]，[remark][remark-plugins]和[rehype][rehype-plugins]的可用插件列表，并尝试其中的一些。

如果您还没有看过，请查看[学习部分][learn]中的其他文章!

<!--Definitions-->

[support]: https://github.com/unifiedjs/.github/blob/main/support.md
[parse]: https://github.com/remarkjs/remark/tree/HEAD/packages/remark-parse
[stringify]: https://github.com/rehypejs/rehype/tree/HEAD/packages/rehype-stringify
[remark-rehype]: https://github.com/remarkjs/remark-rehype
[npm]: https://www.npmjs.com
[node]: https://nodejs.org
[slug]: https://github.com/remarkjs/remark-slug
[toc]: https://github.com/remarkjs/remark-toc
[document]: https://github.com/rehypejs/rehype-document
[to-vfile]: https://github.com/vfile/to-vfile
[reporter]: https://github.com/vfile/vfile-reporter
[unified-stream]: https://github.com/unifiedjs/unified-stream
[english]: https://github.com/retextjs/retext/tree/HEAD/packages/retext-english
[indefinite-article]: https://github.com/retextjs/retext-indefinite-article
[remark-retext]: https://github.com/remarkjs/remark-retext
[retext-plugins]: https://github.com/retextjs/retext/blob/HEAD/doc/plugins.md
[remark-plugins]: https://github.com/remarkjs/remark/blob/HEAD/doc/plugins.md
[rehype-plugins]: https://github.com/rehypejs/rehype/blob/HEAD/doc/plugins.md
[rehype-raw]: https://github.com/rehypejs/rehype-raw
[rehype-format]: https://github.com/rehypejs/rehype-format
[learn]: /learn/
