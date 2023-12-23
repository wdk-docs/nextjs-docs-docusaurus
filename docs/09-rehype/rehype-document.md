# rehype-document

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

**[rehype][]**插件在文档中包装一个片段。

## Contents

- [rehype-document](#rehype-document)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`unified().use(rehypeDocument[, options])`](#unifieduserehypedocument-options)
      - [`options`](#options)
        - [`options.title`](#optionstitle)
        - [`options.language`](#optionslanguage)
        - [`options.dir`](#optionsdir)
        - [`options.responsive`](#optionsresponsive)
        - [`options.style`](#optionsstyle)
        - [`options.css`](#optionscss)
        - [`options.meta`](#optionsmeta)
        - [`options.link`](#optionslink)
        - [`options.script`](#optionsscript)
        - [`options.js`](#optionsjs)
  - [Example](#example)
    - [例子: language](#例子-language)
    - [例子: CSS](#例子-css)
    - [例子: JS](#例子-js)
    - [例子: metadata 和 links](#例子-metadata-和-links)
  - [Types](#types)
  - [兼容性](#兼容性)
  - [安全](#安全)
  - [相关的](#相关的)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个[unified][] ([rehype][])插件，用于在文档中包装片段。
当从表示文章的markdown文件转换为完整的HTML文档时，它特别有用。

**unified** 项目是否使用抽象语法树转换内容 (ASTs).
**rehype** 添加了对HTML的统一支持。
**hast** 是rehype使用的HTML AST。
这是一个在文档中包装片段的rehype插件。

## 我应该什么时候使用这个?

当您想要将片段(特别是可以存在于`<body>`元素中的一些节点)转换为整个文档(`<html>`, `<head>`, 和 `<body>`，后者将包含片段)时，此项目非常有用。

这个插件可以使碎片有效的整个文档。
它不是一个(社交)元数据管理器。
这是通过[`rehype-meta`][rehype-meta]完成的。
你可以同时使用两者。

## 安装

此包是[仅限ESM](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c).
在Node.js(12.20+， 14.14+， 16.0+， 18.0+版本)中，使用[npm][]安装:

```sh
npm install rehype-document
```

在 Deno 中使用 ['esm.sh'][esmsh]：

```js
import rehypeDocument from 'https://esm.sh/rehype-document@6'
```

在浏览器中使用[' esm.sh '][esmsh]:

```html
<script type="module">
  import rehypeDocument from 'https://esm.sh/rehype-document@6?bundle'
</script>
```

## 使用

假设我们有以下文件 `example.md`:

```markdown
## Hello world!

This is **my** document.
```

我们的模块`example.js`看起来如下:

```js
import { read } from 'to-vfile'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import rehypeDocument from 'rehype-document'
import rehypeStringify from 'rehype-stringify'

const file = await unified()
  .use(remarkParse)
  .use(remarkRehype)
  .use(rehypeDocument, { title: 'Hi!' })
  .use(rehypeStringify)
  .process(await read('example.md'))

console.log(String(file))
```

现在运行`node example.js`会得到:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Hi!</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <h2>Hello world!</h2>
    <p>This is <strong>my</strong> document.</p>
  </body>
</html>
```

## API

此包不导出任何标识符。
默认导出为 `rehypeDocument`.

### `unified().use(rehypeDocument[, options])`

在文档中包装一个片段。

##### `options`

配置(可选).

###### `options.title`

用作标题的文本 (`string`, 默认的: 文件的`stem`).

###### `options.language`

文件的自然语言 (`string`, 默认: `'en'`).
应该是[bcp47] [bcp47]语言标签。

> 👉 **Note**: 如果内容不是英文的，你应该设置这个。

###### `options.dir`

文档中文本的方向 (`'ltr'`, `'rtl'`, `'auto'`, 可选).

###### `options.responsive`

是否插入 `meta[viewport]` (`boolean`, 默认: `true`).

###### `options.style`

在`<style>`元素的`head`中包含的CSS (`string`或`Array<string>`,默认: `[]`).

###### `options.css`

指向要包含在`head`中的样式表的链接 (`string`或`Array<string>`,默认: `[]`).

###### `options.meta`

要包含在`head`中的元数据 (`Object`或`Array<Object>`, 默认: `[]`).
每个对象都以[`properties`][props]的形式传递给[`hastscript`][h]，并带有一个`meta`元素。

###### `options.link`

要包含在`head`中的链接标签 (`Object`或`Array<Object>`, 默认: `[]`).
每个对象都以[`properties`][props]的形式通过一个`link`元素传递给[`hastscript`][h]。

###### `options.script`

在`body`结尾包含内联脚本 (`string`或`Array<string>`,默认: `[]`).

###### `options.js`

在`body`结尾包含外部脚本 (`string`或`Array<string>`,默认: `[]`).

## 例子

### 例子: language

这个例子展示了如何设置语言:

```js
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'
import rehypeDocument from 'rehype-document'
import rehypeStringify from 'rehype-stringify'

const file = await unified()
  .use(rehypeParse, { fragment: true })
  .use(rehypeDocument, { title: 'Плутон', language: 'ru' })
  .use(rehypeStringify)
  .process('<h1>Привет, Плутон!</h1>')

console.log(String(file))
```

Yields:

```html
<!doctype html>
<html lang="ru">
  <head>
    <meta charset="utf-8" />
    <title>Плутон</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <h1>Привет, Плутон!</h1>
  </body>
</html>
```

### 例子: CSS

这个例子展示了如何引用CSS文件和包含样式表:

```js
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'
import rehypeDocument from 'rehype-document'
import rehypeStringify from 'rehype-stringify'

const file = await unified()
  .use(rehypeParse, { fragment: true })
  .use(rehypeDocument, {
    css: 'https://example.com/index.css',
    style: 'body { color: red }',
  })
  .use(rehypeStringify)
  .process('')

console.log(String(file))
```

Yields:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      body {
        color: red;
      }
    </style>
    <link rel="stylesheet" href="https://example.com/index.css" />
  </head>
  <body></body>
</html>
```

### 例子: JS

这个例子展示了如何引用JS文件和包含脚本:

```js
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'
import rehypeDocument from 'rehype-document'
import rehypeStringify from 'rehype-stringify'

const file = await unified()
  .use(rehypeParse, { fragment: true })
  .use(rehypeDocument, {
    js: 'https://example.com/index.js',
    script: 'console.log(1)',
  })
  .use(rehypeStringify)
  .process('')

console.log(String(file))
```

Yields:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <script>
      console.log(1)
    </script>
    <script src="https://example.com/index.js"></script>
  </body>
</html>
```

### 例子: metadata 和 links

这个例子展示了如何定义元数据和包含链接(而不是样式):

```js
import { unified } from 'unified'
import rehypeParse from 'rehype-parse'
import rehypeDocument from 'rehype-document'
import rehypeStringify from 'rehype-stringify'

const file = await unified()
  .use(rehypeParse, { fragment: true })
  .use(rehypeDocument, {
    link: [
      { rel: 'icon', href: '/favicon.ico', sizes: 'any' },
      { rel: 'icon', href: '/icon.svg', type: 'image/svg+xml' },
    ],
    meta: [{ name: 'generator', content: 'rehype-document' }],
  })
  .use(rehypeStringify)
  .process('')

console.log(String(file))
```

Yields:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="generator" content="rehype-document" />
    <link rel="icon" href="/favicon.ico" sizes="any" />
    <link rel="icon" href="/icon.svg" type="image/svg+xml" />
  </head>
  <body></body>
</html>
```

> 💡 **Tip**: [`rehype-meta`][rehype-meta] 是(社区)元数据管理器吗.

## Types

This package is fully typed with [TypeScript][].
It exports an `Options` type, which specifies the interface of the accepted
options.

## 兼容性

Projects maintained by the unified collective are compatible with all maintained
versions of Node.js.
As of now, that is Node.js 12.20+, 14.14+, 16.0+, and 18.0+.
Our projects sometimes work with older versions, but this is not guaranteed.

This plugin works with `rehype-parse` version 3+, `rehype-stringify` version 3+,
`rehype` version 5+, and `unified` version 6+.

## 安全

Use of `rehype-document` can open you up to a [cross-site scripting (XSS)][xss]
attack if you pass user provided content in options.
Always be wary of user input and use [`rehype-sanitize`][rehype-sanitize].

## 相关的

- [`rehype-meta`](https://github.com/rehypejs/rehype-meta)
  — add metadata to the head of a document
- [`rehype-format`](https://github.com/rehypejs/rehype-format)
  — format HTML
- [`rehype-minify`](https://github.com/rehypejs/rehype-minify)
  — minify HTML

## 贡献

See [`contributing.md`][contributing] in [`rehypejs/.github`][health] for ways
to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization,或community you agree to
abide by its terms.

## 许可证

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/rehypejs/rehype-document/workflows/main/badge.svg
[build]: https://github.com/rehypejs/rehype-document/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/rehypejs/rehype-document.svg
[coverage]: https://codecov.io/github/rehypejs/rehype-document
[downloads-badge]: https://img.shields.io/npm/dm/rehype-document.svg
[downloads]: https://www.npmjs.com/package/rehype-document
[size-badge]: https://img.shields.io/bundlephobia/minzip/rehype-document.svg
[size]: https://bundlephobia.com/result?p=rehype-document
[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg
[backers-badge]: https://opencollective.com/unified/backers/badge.svg
[collective]: https://opencollective.com/unified
[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg
[chat]: https://github.com/rehypejs/rehype/discussions
[esmsh]: https://esm.sh
[npm]: https://docs.npmjs.com/cli/install
[health]: https://github.com/rehypejs/.github
[contributing]: https://github.com/rehypejs/.github/blob/HEAD/contributing.md
[support]: https://github.com/rehypejs/.github/blob/HEAD/support.md
[coc]: https://github.com/rehypejs/.github/blob/HEAD/code-of-conduct.md
[license]: license
[author]: https://wooorm.com
[unified]: https://github.com/unifiedjs/unified
[rehype]: https://github.com/rehypejs/rehype
[bcp47]: https://tools.ietf.org/html/bcp47
[props]: https://github.com/syntax-tree/hastscript#hselector-properties-children
[h]: https://github.com/syntax-tree/hastscript
[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
[typescript]: https://www.typescriptlang.org
[rehype-sanitize]: https://github.com/rehypejs/rehype-sanitize
[rehype-meta]: https://github.com/rehypejs/rehype-meta
