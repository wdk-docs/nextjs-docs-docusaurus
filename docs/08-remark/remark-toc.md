# remark-toc

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

[**remark**][remark]插件生成目录表。

## Contents

- [remark-toc](#remark-toc)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`unified().use(remarkToc[, options])`](#unifieduseremarktoc-options)
      - [`options`](#options)
      - [`options.heading`](#optionsheading)
      - [`options.skip`](#optionsskip)
        - [`options.maxDepth`](#optionsmaxdepth)
        - [`options.tight`](#optionstight)
        - [`options.ordered`](#optionsordered)
        - [`options.prefix`](#optionsprefix)
        - [`options.parents`](#optionsparents)
  - [例子](#例子)
    - [示例: 不同的标题](#示例-不同的标题)
    - [示例: 有序、紧凑的列表](#示例-有序紧凑的列表)
    - [示例: 包括和不包括标题](#示例-包括和不包括标题)
    - [例如: 添加前缀](#例如-添加前缀)
  - [Types](#types)
  - [兼容性](#兼容性)
  - [安全](#安全)
  - [相关的](#相关的)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个[unified][] ([remark][])插件，用于生成像上面那样的文档的目录。

**unified** 是一个使用抽象语法树(ASTs)转换内容的项目。
**remark** 增加对 unified markdown的支持。
**mdast** 是remark使用的 markdown AST。这是一个转换mdast的remark插件。

## 我应该什么时候使用这个?

当作者在markdown中编写文档时，这个项目非常有用，这些文档有时很长，因此可以从其中的自动概述中受益。
假设标题定义了文档的结构，并且可以链接到它们。
当使用这个插件时，作者可以在文档中添加一个特定的标题(比如， `## Table of contents`)，这个插件将用链接到所有后续部分的列表填充这些部分。

GitHub和类似的服务会自动为标题添加id(以及链接到self的锚)。
在这个插件之后，你可以通过[`remark-rehype`][remark-rehype]将remark和[rehype][]组合起来添加类似的功能。
然后可以使用rehype插件[`rehype-slug`][rehype-slug](用于标题上的id)和[`rehype-autolink-headings`][rehype-autolink-headings](用于链接到自己的锚)。

这个插件不会为*whole*文档生成目录，也不会将其暴露给其他插件。
你可以使用底层的mdast实用程序[`mdast-util-toc`][mdast-util-toc]和[创建一个插件][create-a-plugin]来完成这些工作，甚至更多。

## 安装

此包是[仅限ESM](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c).
在Node.js(12.20+， 14.14+或16.0+版本)中，使用[npm][]安装:

```sh
npm install remark-toc
```

在 Deno 中使用 ['esm.sh'][esmsh]：

```js
import remarkToc from 'https://esm.sh/remark-toc@8'
```

在浏览器中使用[`esm.sh`][esmsh]:

```html
<script type="module">
  import remarkToc from 'https://esm.sh/remark-toc@8?bundle'
</script>
```

## 使用

假设我们有以下文件: `example.md`:

```markdown
# Alpha

## Table of contents

## Bravo

### Charlie

## Delta
```

我们的模块，`example.js`，看起来如下:

```js
import { read } from 'to-vfile'
import { remark } from 'remark'
import remarkToc from 'remark-toc'

main()

async function main() {
  const file = await remark()
    .use(remarkToc)
    .process(await read('example.md'))

  console.log(String(file))
}
```

现在，运行`node example`会得到:

```markdown
# Alpha

## Table of contents

- [Bravo](#bravo)

  - [Charlie](#charlie)

- [Delta](#delta)

## Bravo

### Charlie

## Delta
```

## API

此包不导出任何标识符。
默认导出为`remarkToc`。

### `unified().use(remarkToc[, options])`

生成一个目录。
查找某个标题，删除它与相同或更高标题之间的所有内容，并用表示文档结构的列表替换它，并链接到所有进一步的标题。

##### `options`

配置(可选)。

##### `options.heading`

查找标题的模式文本 (`string`, 默认: `'toc|table[ -]of[ -]contents?'`).
包裹在`new RegExp('^(' + options.heading + ')$', 'i')`里面 , 它不区分大小写，匹配整个标题文本。

##### `options.skip`

要从生成的列表中排除的标题的模式文本 (`string`,可选).
封装在`new RegExp('^(' + options.skip + ')$', 'i')`里, 它不区分大小写，匹配整个标题文本。

###### `options.maxDepth`

要包含在生成列表中的最大标题深度 (`number?`, 默认:`6`).
这是包含的:当设置为`3`时，包含秩为3的标题(有三个哈希值的: `###`).

###### `options.tight`

是否紧密编译列表项(`boolean?`, 默认: `false`)。
默认情况是在项目周围添加空格。

###### `options.ordered`

是否将列表项编译为有序列表 (`boolean?`, 默认: `false`).
默认是使用无序列表。

###### `options.prefix`

在指向标题的链接之前附加的字符串 (`string?`, 默认: `null`, 例子: `'user-content-'`).
在这个插件之后，通过[`remark-rehype`][remark-rehype]将remark和[rehype][]结合使用，并使用[`rehype-sanitize`][rehype-sanitize]来防止用户生成的markdown被DOM破坏，这是很有用的。

###### `options.parents`

要包含在生成的列表中的标题的父(如块引号和列表)([`is`-compatible][is] test，默认:根节点)。
默认情况下，只使用顶级标题。
传递`['root', 'blockquote']`也可以链接到块引号中的标题。

## 例子

### 示例: 不同的标题

选项`heading` 可以设置为搜索不同的标题。
前面的例子可以更改为搜索不同的标题，如下所示:

```diff
@@ -6,7 +6,7 @@ main()

 async function main() {
   const file = await remark()
-    .use(remarkToc)
+    .use(remarkToc, {heading: 'contents'})
     .process(await read('example.md'))

   console.log(String(file))
```

…它将搜索`Contents`(不区分大小写)标题。

### 示例: 有序、紧凑的列表

可以打开`ordered` 和 `tight`选项来更改列表。
前面的示例可以更改为生成一个紧凑的有序列表，如下所示:

```diff
@@ -6,7 +6,10 @@ main()

 async function main() {
   const file = await remark()
-    .use(remarkToc)
+    .use(remarkToc, {tight: true, ordered: true})
     .process(await read('example.md'))

   console.log(String(file))
```

…这将生成以下列表:

```markdown
1.  [Bravo](#bravo)
    1.  [Charlie](#charlie)
2.  [Delta](#delta)
```

### 示例: 包括和不包括标题

选项`maxDepth`, `skip`, 和 `parents`可用于从列表中包括或排除某些标题。
前面的示例可以更改为生成一个紧凑的有序列表，如下所示:

```diff
@@ -6,7 +6,10 @@ main()

 async function main() {
   const file = await remark()
-    .use(remarkToc)
+    .use(remarkToc, {maxDepth: 3, skip: 'delta', parents: ['root', 'listItem']})
     .process(await read('example.md'))

   console.log(String(file))
```

…这将排除第4、5和6级标题，排除`delta`(不区分大小写，完全匹配)的标题，并直接在列表项中包含标题。

### 例如: 添加前缀

选项`prefix`可以设置为在生成的列表中为所有指向标题的链接添加一个字符串:

```diff
@@ -6,7 +6,10 @@ main()

 async function main() {
   const file = await remark()
-    .use(remarkToc)
+    .use(remarkToc, {prefix: 'user-content-'})
     .process(await read('example.md'))

   console.log(String(file))
```

…这将生成以下列表:

```markdown
- [Bravo](#user-content-bravo)

  - [Charlie](#user-content-charlie)

- [Delta](#user-content-delta)
```

## Types

这个包是完全使用[TypeScript][]类型的。
它导出一个`Options` 类型，该类型指定可接受选项的接口。

## 兼容性

unified集体维护的项目兼容所有维护的Node.js版本。
到目前为止，这是Node.js 12.20+、14.14+和16.0+。
我们的项目有时会使用旧版本，但这不能保证。

此插件适用于`unified`版本3+和`remark`版本4+。

## 安全

使用`remark-toc`涉及用户内容并更改树，因此它可以打开您的[跨站点脚本(XSS)][xss]攻击。

将现有节点复制到目录中。
下面的示例展示了如何将现有脚本复制到目录中。

以下是markdown:

```markdown
# Table of Contents

## Bravo<script>alert(1)</script>

## Charlie
```

Yields:

```markdown
# Table of Contents

- [Bravo<script>alert(1)</script>](#bravoscriptalert1script)
- [Charlie](#charlie)

## Bravo<script>alert(1)</script>

## Charlie
```

如果markdown后转换为[**rehype**][rehype] ([**hast**][hast])或在不安全的markdown查看器中打开，则可能会出现问题。

## 相关的

- [`remark-normalize-headings`](https://github.com/remarkjs/remark-normalize-headings)
  — 通过规范化标题等级，确保只有一个顶级标题
- [`remark-collapse`](https://github.com/Rokt33r/remark-collapse)
  – 让一些部分可折叠
- [`remark-contributors`](https://github.com/remarkjs/remark-contributors)
  – 生成贡献者部分
- [`remark-license`](https://github.com/remarkjs/remark-license)
  – 生成许可证部分
- [`remark-package-dependencies`](https://github.com/unlight/remark-package-dependencies)
  – 生成一个依赖项部分
- [`remark-usage`](https://github.com/remarkjs/remark-usage)
  – 生成一个用法部分

## 贡献

请参阅[`remarkjs/.github`][health]中的[`contributing.md`][contributing.md]了解如何开始。
请参阅[`support.md`][support]获取帮助的方法。

这个项目有[行为准则][coc]。
通过与此存储库、组织或社区进行交互，您同意遵守其条款。

## 许可证

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/remarkjs/remark-toc/workflows/main/badge.svg
[build]: https://github.com/remarkjs/remark-toc/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/remarkjs/remark-toc.svg
[coverage]: https://codecov.io/github/remarkjs/remark-toc
[downloads-badge]: https://img.shields.io/npm/dm/remark-toc.svg
[downloads]: https://www.npmjs.com/package/remark-toc
[size-badge]: https://img.shields.io/bundlephobia/minzip/remark-toc.svg
[size]: https://bundlephobia.com/result?p=remark-toc
[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg
[backers-badge]: https://opencollective.com/unified/backers/badge.svg
[collective]: https://opencollective.com/unified
[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg
[chat]: https://github.com/remarkjs/remark/discussions
[npm]: https://docs.npmjs.com/cli/install
[esmsh]: https://esm.sh
[health]: https://github.com/remarkjs/.github
[contributing]: https://github.com/remarkjs/.github/blob/HEAD/contributing.md
[support]: https://github.com/remarkjs/.github/blob/HEAD/support.md
[coc]: https://github.com/remarkjs/.github/blob/HEAD/code-of-conduct.md
[license]: license
[author]: https://wooorm.com
[remark]: https://github.com/remarkjs/remark
[unified]: https://github.com/unifiedjs/unified
[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
[typescript]: https://www.typescriptlang.org
[rehype]: https://github.com/rehypejs/rehype
[hast]: https://github.com/syntax-tree/hast
[remark-rehype]: https://github.com/remarkjs/remark-rehype
[rehype-slug]: https://github.com/rehypejs/rehype-slug
[rehype-autolink-headings]: https://github.com/rehypejs/rehype-autolink-headings
[rehype-sanitize]: https://github.com/rehypejs/rehype-sanitize
[mdast-util-toc]: https://github.com/syntax-tree/mdast-util-toc
[create-a-plugin]: https://unifiedjs.com/learn/guide/create-a-plugin/
[is]: https://github.com/syntax-tree/unist-util-is#api
