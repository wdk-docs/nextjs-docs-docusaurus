---
group: recipe
index: 6
title: HTML 和 remark
description: How to use remark to turn markdown into HTML, and to allow embedded HTML inside markdown
tags:
  - remark
  - html
  - plugin
  - markdown
  - html
  - parse
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
published: 2021-03-09
modified: 2021-03-09
---

## HTML 和 remark

remark 是一个markdown编译器。
它在两个方面与HTML有关:

1.  markdown 经常被转换成HTML
2.  markdown 有时会嵌入HTML

在处理HTML和markdown时，我们将同时使用remark和rehype。
本文展示了如何做到这一点的一些示例。

### Contents

- [HTML 和 remark](#html-和-remark)
  - [Contents](#contents)
  - [如何将markdown转换为HTML](#如何将markdown转换为html)
  - [如何把HTML变成markdown](#如何把html变成markdown)
  - [如何允许HTML嵌入在markdown](#如何允许html嵌入在markdown)
  - [如何正确地支持HTML内markdown](#如何正确地支持html内markdown)

### 如何将markdown转换为HTML

Remark处理markdown:它可以解析和序列化markdown。
但它*不*适合HTML。
这就是rehype所做的，它的存在是为了解析和序列化HTML。

要将markdown转换为HTML，我们需要 [`remark-parse`][remark-parse],[`remark-rehype`][remark-rehype], and [`rehype-stringify`][rehype-stringify]:

```javascript
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

unified()
  .use(remarkParse) // 将标记内容解析为语法树
  .use(remarkRehype) // 将标记下调语法树转换为HTML语法树，忽略嵌入的HTML
  .use(rehypeStringify) // 序列化HTML语法树
  .process('*emphasis* and **strong**')
  .then((file) => console.log(String(file)))
  .catch((error) => {
    throw error
  })
```

这将把`*emphasis* and **strong**`变成`<em>emphasis</em>`和`<strong>strong</strong>`，但它不支持嵌入在markdown中的HTML(例如`*emphasis* and <strong>strong</strong>`)。

此解决方案**是安全的**:您不信任的内容不会导致XSS漏洞。

### 如何把HTML变成markdown

我们也可以做逆运算。
要将HTML转换为markdown，我们需要[`rehype-parse`][rehype-parse],[`rehype-remark`][rehype-remark], 和 [`remark-stringify`][remark-stringify]:

```javascript
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'
import rehypeRemark from 'rehype-remark'
import remarkStringify from 'remark-stringify'

unified()
  .use(rehypeParse) // Parse HTML to a syntax tree
  .use(rehypeRemark) // Turn HTML syntax tree to markdown syntax tree
  .use(remarkStringify) // Serialize HTML syntax tree
  .process('<em>emphasis</em> and <strong>strong</strong>')
  .then((file) => console.log(String(file)))
  .catch((error) => {
    throw error
  })
```

这将`<em>emphasis</em> and <strong>strong</strong>`变成了`*emphasis* and **strong**`。

### 如何允许HTML嵌入在markdown

Markdown是一种非常适合基本内容的内容格式:写`*emphasis*`比写`<em>emphasis</em>`更好。
但是，它是有限的:它简洁的语法只支持一些事情。
幸运的是，对于更复杂的东西，markdown允许在其中使用HTML。
一个常见的例子是包含一个`<details>`元素。

通过配置[`remark-rehype`][remark-rehype] 和 [`rehype-stringify`][rehype-stringify]，可以允许markdown中嵌入的HTML从markdown转换为HTML:

```javascript
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

unified()
  .use(remarkParse)
  .use(remarkRehype, { allowDangerousHtml: true }) // Pass raw HTML strings through.
  .use(rehypeStringify, { allowDangerousHtml: true }) // Serialize the raw HTML strings
  .process('*emphasis* and <strong>strong</strong>')
  .then((file) => console.log(String(file)))
  .catch((error) => {
    throw error
  })
```

此解决方案**不安全**:您不信任的内容可能导致XSS漏洞。

### 如何正确地支持HTML内markdown

为了正确支持嵌入在markdown中的HTML，我们需要另一个插件: [`rehype-raw`][rehype-raw].
这个插件将采用嵌入在markdown中的HTML字符串，并使用实际的HTML解析器对其进行解析。

```javascript
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeRaw from 'rehype-raw'
import rehypeStringify from 'rehype-stringify'

unified()
  .use(remarkParse)
  .use(remarkRehype, { allowDangerousHtml: true })
  .use(rehypeRaw) // *Parse* the raw HTML strings embedded in the tree
  .use(rehypeStringify)
  .process('*emphasis* and <strong>strong</strong>')
  .then((file) => console.log(String(file)))
  .catch((error) => {
    throw error
  })
```

此解决方案**不安全**:您不信任的内容可能导致XSS漏洞。

但是因为我们现在有了一个完整的HTML语法树，所以我们可以对该树进行清理。
为了安全起见，在`rehype-stringify`之前加上[`rehype-sanitize`][rehype-sanitize]

[remark-parse]: https://github.com/remarkjs/remark/tree/main/packages/remark-parse
[remark-stringify]: https://github.com/remarkjs/remark/tree/main/packages/remark-stringify
[remark-rehype]: https://github.com/remarkjs/remark-rehype
[rehype-parse]: https://github.com/rehypejs/rehype/tree/main/packages/rehype-parse
[rehype-stringify]: https://github.com/rehypejs/rehype/tree/main/packages/rehype-stringify
[rehype-remark]: https://github.com/rehypejs/rehype-remark
[rehype-raw]: https://github.com/rehypejs/rehype-raw
[rehype-sanitize]: https://github.com/rehypejs/rehype-sanitize
