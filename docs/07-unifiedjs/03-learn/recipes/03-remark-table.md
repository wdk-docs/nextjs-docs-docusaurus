---
group: recipe
index: 5
title: remark支持表格
description: 如何在remark(或react-markdown)中支持github样式的表
tags:
  - remark
  - plugin
  - gfm
  - github
  - table
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
published: 2021-02-24
modified: 2021-02-24
---

## 如何支持表格 in remark

表是markdown中的一个非标准特性:它们**不**是在[CommonMark][]中定义的，并且不会在任何地方工作。

表是GitHub在其[GFM][]中支持的扩展。
他们在`github.com`的大多数地方工作:自述，问题，公关，讨论，评论等。

remark和unified可以通过插件来支持它们: [`remark-gfm`][remark-gfm].

### Contents

- [如何支持表格 in remark](#如何支持表格-in-remark)
  - [Contents](#contents)
  - [表格是什么?](#表格是什么)
  - [如何编写表格](#如何编写表格)
  - [如何支持表格](#如何支持表格)
  - [如何在`react-markdown`中支持表格](#如何在react-markdown中支持表格)

### 表格是什么?

markdown中的表用于表格数据，看起来像这样:

```markdown
| Beep |  No.   |   Boop |
| :--- | :----: | -----: |
| beep |  1024  |    xyz |
| boop | 338845 |    tuv |
| foo  | 10106  | qrstuv |
| bar  |   45   |   lmno |
```

网站上的结果是这样的:

| Beep |  No.   |   Boop |
| :--- | :----: | -----: |
| beep |  1024  |    xyz |
| boop | 338845 |    tuv |
| foo  | 10106  | qrstuv |
| bar  |   45   |   lmno |

### 如何编写表格

在一行中的单元格之间使用管道字符(`|`)。
新的一行开始新的一行。
您不必对齐管道(`|`)来形成一个漂亮的网格，但它确实使
源代码更具可读性。

第一行是*表标题*，其中的单元格是各自列的标签。

第二行是*对齐行*，每个“单元格”必须包含一个破折号(`-`)。
单元格可以在开始处使用冒号(`:--`)向左对齐，在结束处使用冒号(`--:`)向右对齐，或者在开始和结束处使用冒号居中对齐(`:-:`)。

其他行是*表主体*，是可选的。
它们的单元格是表数据。

### 如何支持表格

由于表是非标准的，remark默认不支持表。
但它可以支持他们的插件:[`remark-gfm`][remark-gfm]。
假设我们有一个GFM表的markdown，在一个`example.md`文件中:

```markdown
# Table

| Branch  | Commit           |
| ------- | ---------------- |
| main    | 0123456789abcdef |
| staging | fedcba9876543210 |
```

还有一个脚本可以将表格转换为HTML， `example.js`:

```javascript
import fs from 'node:fs'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkGfm from 'remark-gfm'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

const doc = fs.readFileSync('example.md')

const file = unified()
  .use(remarkParse) // Parse markdown.
  .use(remarkGfm) // Support GFM (tables, autolinks, tasklists, strikethrough).
  .use(remarkRehype) // Turn it into HTML.
  .use(rehypeStringify) // Serialize HTML.
  .processSync(doc)

console.log(String(file))
```

现在，运行`node example` 会得到:

```html
<h1>Table</h1>
<table>
  <thead>
    <tr>
      <th>Branch</th>
      <th>Commit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>main</td>
      <td>0123456789abcdef</td>
    </tr>
    <tr>
      <td>staging</td>
      <td>fedcba9876543210</td>
    </tr>
  </tbody>
</table>
```

### 如何在`react-markdown`中支持表格

由于表是非标准的，`react-markdown`默认情况下不支持它们。
但它可以支持他们的插件:[`remark-gfm`][remark-gfm]。

假设我们有一个GFM表的markdown，在一个`example.md`文件中:

```js
import React from 'react'
import ReactMarkdown from 'react-markdown'
import remarkGfm from 'remark-gfm'

const markdown = `# Table

| Branch  | Commit           |
| ------- | ---------------- |
| main    | 0123456789abcdef |
| staging | fedcba9876543210 |`

console.log(<ReactMarkdown plugins={[remarkGfm]} children={markdown} />)
```

JSX的收益率:

```javascript
<>
  <h1>Table</h1>
  <table>
    <thead>
      <tr>
        <th>Branch</th>
        <th>Commit</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>main</td>
        <td>0123456789abcdef</td>
      </tr>
      <tr>
        <td>staging</td>
        <td>fedcba9876543210</td>
      </tr>
    </tbody>
  </table>
</>
```

[commonmark]: https://commonmark.org
[gfm]: https://github.github.com/gfm/
[remark-gfm]: https://github.com/remarkjs/remark-gfm
