# remark-gfm

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

支持[GFM][]**[remark][]**插件 (autolink literals, footnotes, strikethrough, tables, tasklists).

## Contents

- [remark-gfm](#remark-gfm)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`unified().use(remarkGfm[, options])`](#unifieduseremarkgfm-options)
      - [`options`](#options)
        - [`options.singleTilde`](#optionssingletilde)
        - [`options.tableCellPadding`](#optionstablecellpadding)
        - [`options.tablePipeAlign`](#optionstablepipealign)
        - [`options.stringLength`](#optionsstringlength)
  - [例子](#例子)
    - [例子: `singleTilde`](#例子-singletilde)
    - [例子: `stringLength`](#例子-stringlength)
  - [Types](#types)
  - [兼容性](#兼容性)
  - [安全](#安全)
  - [相关的](#相关的)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

This package is a [unified][] ([remark][]) plugin to enable the extensions to
markdown that GitHub adds: autolink literals (`www.x.com`), footnotes (`[^1]`),
strikethrough (`~~stuff~~`), tables (`| cell |…`), and tasklists (`* [x]`).
You can use this plugin to add support for parsing and serializing them.
These extensions by GitHub to CommonMark are called [GFM][] (GitHub Flavored
Markdown).

**unified** is a project that transforms content with abstract syntax trees
(ASTs).
**remark** adds support for markdown to unified.
**mdast** is the markdown AST that remark uses.
This is a remark plugin that transforms mdast.

## 我应该什么时候使用这个?

This project is useful when you want to support the same features that GitHub
does in files in a repo, Gists, and several other places.
Users frequently believe that some of these extensions, specifically autolink
literals and tables, are part of normal markdown, so using `remark-gfm` will
help match your implementation to their understanding of markdown.
There are several edge cases where GitHub’s implementation works in unexpected
ways or even different than described in their spec, so _writing_ in GFM is not
always the best choice.

This plugin does not handle how markdown is turned to HTML.
That’s done by [`remark-rehype`][remark-rehype].
If your content is not in English and uses footnotes, you should configure that
plugin.
When generating HTML, you might also want to enable [`rehype-slug`][rehype-slug]
to add `id`s on headings.

A different plugin, [`remark-frontmatter`][remark-frontmatter], adds support for
frontmatter.
GitHub supports YAML frontmatter for files in repos and Gists but they don’t
treat it as part of GFM.

Another plugin, [`remark-github`][remark-github], adds support for how markdown
works in relation to a certain GitHub repo in comments, issues, PRs, and
releases, by linking references to commits, issues, and users.

Yet another plugin, [`remark-breaks`][remark-breaks], turns soft line endings
(enters) into hard breaks (`<br>`s).
GitHub does this in a few places (comments, issues, PRs, and releases).

## 安装

This package is [ESM only](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c).
In Node.js (version 12.20+, 14.14+, or 16.0+), install with [npm][]:

```sh
npm install remark-gfm
```

In Deno with [`esm.sh`][esmsh]:

```js
import remarkGfm from 'https://esm.sh/remark-gfm@3'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import remarkGfm from 'https://esm.sh/remark-gfm@3?bundle'
</script>
```

## 使用

Say we have the following file `example.md`:

```markdown
# GFM

## Autolink literals

www.example.com, https://example.com, and contact@example.com.

## Footnote

A note[^1]

[^1]: Big note.

## Strikethrough

~one~ or ~~two~~ tildes.

## Table

| a   | b   |   c |  d  |
| --- | :-- | --: | :-: |

## Tasklist

- [ ] to do
- [x] done
```

And our module `example.js` looks as follows:

```js
import { read } from 'to-vfile'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkGfm from 'remark-gfm'
import remarkRehype from 'remark-rehype'
import rehypeStringify from 'rehype-stringify'

main()

async function main() {
  const file = await unified()
    .use(remarkParse)
    .use(remarkGfm)
    .use(remarkRehype)
    .use(rehypeStringify)
    .process(await read('example.md'))

  console.log(String(file))
}
```

Now running `node example` yields:

```html
<h1>GFM</h1>
<h2>Autolink literals</h2>
<p>
  <a href="http://www.example.com">www.example.com</a>,
  <a href="https://example.com">https://example.com</a>, and
  <a href="mailto:contact@example.com">contact@example.com</a>.
</p>
<h2>Footnote</h2>
<p>
  A note<sup
    ><a
      href="#user-content-fn-1"
      id="user-content-fnref-1"
      data-footnote-ref
      aria-describedby="footnote-label"
      >1</a
    ></sup
  >
</p>
<h2>Strikethrough</h2>
<p><del>one</del> or <del>two</del> tildes.</p>
<h2>Table</h2>
<table>
  <thead>
    <tr>
      <th>a</th>
      <th align="left">b</th>
      <th align="right">c</th>
      <th align="center">d</th>
    </tr>
  </thead>
</table>
<h2>Tasklist</h2>
<ul class="contains-task-list">
  <li class="task-list-item"><input type="checkbox" disabled /> to do</li>
  <li class="task-list-item">
    <input type="checkbox" checked disabled /> done
  </li>
</ul>
<section data-footnotes class="footnotes">
  <h2 id="footnote-label" class="sr-only">Footnotes</h2>
  <ol>
    <li id="user-content-fn-1">
      <p>
        Big note.
        <a
          href="#user-content-fnref-1"
          data-footnote-backref
          class="data-footnote-backref"
          aria-label="Back to content"
          >↩</a
        >
      </p>
    </li>
  </ol>
</section>
```

## API

此包不导出任何标识符。
默认导出为`remarkGfm`。

### `unified().use(remarkGfm[, options])`

支持[GFM][] (autolink literals, footnotes, strikethrough, tables,tasklists).

##### `options`

配置(可选)。

###### `options.singleTilde`

是否用单个波浪线解析划线 (`boolean`, 默认:`true`).
单个波浪可以在github.com上工作，但在技术上被GFM规范禁止。

###### `options.tableCellPadding`

用分隔符(`|`)和单元格内容之间的空格序列化表 (`boolean`, 默认: `true`).

###### `options.tablePipeAlign`

通过对齐表单元格之间的分隔符(`|`)来序列化，以便它们都很好地对齐并形成一个网格(`boolean`, 默认: `true`).

###### `options.stringLength`

函数检测表单元格内容的长度 (`Function`, 默认:`s => s.length`).
这在对齐表单元格之间的分隔符(`|`)时使用。
当查看markdown源代码时，全宽字符和表情符号会弄乱分隔符对齐。
要解决这个问题，可以传递这个函数，它接收单元格内容并返回其“visible”大小。
注意，什么是可见的，什么是不可见的取决于文本显示的位置。

## 例子

### 例子: `singleTilde`

要关闭对使用单个波浪线解析划线的支持，请传递 `singleTilde: false`:

```js
// …

const file = await unified()
  .use(remarkParse)
  .use(remarkGfm, { singleTilde: false })
  .use(remarkRehype)
  .use(rehypeStringify)
  .process('~one~ and ~~two~~')

console.log(String(file))
```

Yields:

```html
<p>~one~ and <del>two</del></p>
```

### 例子: `stringLength`

可以根据单元格的视觉宽度对表进行对齐。
首先，让我们来看看这个问题:

```js
import { remark } from 'remark'
import remarkGfm from 'remark-gfm'

main()

async function main() {
  const input = `| Alpha | Bravo |
| - | - |
| 中文 | Charlie |
| 👩‍❤️‍👩 | Delta |`

  const file = await remark().use(remarkGfm).process(input)

  console.log(String(file))
}
```

上面的代码显示了如何使用remark来格式化标记。
输出结果如下:

```markdown
| Alpha | Bravo   |
| ----- | ------- |
| 中文  | Charlie |
| 👩‍❤️‍👩    | Delta   |
```

为了改善这些全宽字符和表情符号的对齐，传递一个`stringLength`函数来计算单元格的视觉宽度。
其中一个算法是[`string-width`][string-width]。
它可以这样使用:

```diff
@@ -1,5 +1,6 @@
 import {remark} from 'remark'
 import remarkGfm from 'remark-gfm'
+import stringWidth from 'string-width'

 main()

@@ -10,7 +11,7 @@ async function main() {
 | 👩‍❤️‍👩 | Delta |`

   const file = await remark()
-    .use(remarkGfm)
+    .use(remarkGfm, {stringLength: stringWidth})
     .process(input)

   console.log(String(file))
```

修改后的代码输出如下:

```markdown
| Alpha | Bravo   |
| ----- | ------- |
| 中文  | Charlie |
| 👩‍❤️‍👩    | Delta   |
```

## Types

这个包是完全使用[TypeScript][]类型的。
它导出一个`Options`类型，该类型指定可接受选项的接口。

## 兼容性

unified集体维护的项目兼容所有维护的Node.js版本。
到目前为止，这是Node.js 12.20+、14.14+和16.0+。
我们的项目有时会使用旧版本，但这不能保证。

此插件适用于`remark-parse`版本10+ (`remark`版本 14+)。
以前的版本(v2)与`remark-parse`版本9 (`remark`版本13)一起使用。
早期版本的`remark-parse`和`remark`有一个`gfm`选项来启用此功能，默认为true。

## 安全

使用`remark-gfm`不涉及**[rehype][]** (**[hast][]**)，因此没有[跨站点脚本(XSS)][xss]攻击的机会。

## 相关的

- [`remark-github`][remark-github]
  — 链接提交、问题、pr和用户的引用
- [`remark-breaks`][remark-breaks]
  — 支持断行而不需要空格或转义 (输入到 `<br>`)
- [`remark-frontmatter`][remark-frontmatter]
  — 支持frontmatter (YAML, TOML, 和更多的)
- [`remark-directive`](https://github.com/remarkjs/remark-directive)
  — 支持指令
- [`remark-math`](https://github.com/remarkjs/remark-math)
  — 支持数学
- [`remark-mdx`](https://github.com/mdx-js/mdx/tree/main/packages/remark-mdx)
  — 支持MDX (JSX, 表达式, ESM)

## 贡献

See [`contributing.md`][contributing] in [`remarkjs/.github`][health] for ways to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to abide by its terms.

## 许可证

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/remarkjs/remark-gfm/workflows/main/badge.svg
[build]: https://github.com/remarkjs/remark-gfm/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/remarkjs/remark-gfm.svg
[coverage]: https://codecov.io/github/remarkjs/remark-gfm
[downloads-badge]: https://img.shields.io/npm/dm/remark-gfm.svg
[downloads]: https://www.npmjs.com/package/remark-gfm
[size-badge]: https://img.shields.io/bundlephobia/minzip/remark-gfm.svg
[size]: https://bundlephobia.com/result?p=remark-gfm
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
[gfm]: https://github.github.com/gfm/
[remark-frontmatter]: https://github.com/remarkjs/remark-frontmatter
[remark-github]: https://github.com/remarkjs/remark-github
[remark-breaks]: https://github.com/remarkjs/remark-breaks
[remark-rehype]: https://github.com/remarkjs/remark-rehype
[rehype-slug]: https://github.com/rehypejs/rehype-slug
[string-width]: https://github.com/sindresorhus/string-width
