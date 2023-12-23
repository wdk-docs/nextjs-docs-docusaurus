# rehype-slug

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

**[rehype][]** plugin to add `id`s to headings.

## Contents

- [rehype-slug](#rehype-slug)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`unified().use(rehypeSlug[, options])`](#unifieduserehypeslug-options)
      - [`options`](#options)
        - [`options.prefix`](#optionsprefix)
  - [Types](#types)
  - [兼容性](#兼容性)
  - [安全](#安全)
  - [相关的](#相关的)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个[unified][] ([rehype][])插件，用于在标题中添加`id`。
它查找标题(所以`<h1>`到`<h6>`)还没有`id`，并根据它们包含的文本为它们添加`id`属性。
这样做的算法是[`github-slugger`][GitHub -slugger]，这与GitHub的工作原理相匹配。

**unified** 项目是否使用抽象语法树转换内容(ASTs).
**rehype** 添加了对HTML的统一支持.
**hast** i这是rehype使用的HTML AST.这是一个rehype插件，在AST的标题中添加`id`。

## 我应该什么时候使用这个?

This plugin is useful when you have relatively long documents and you want to be
able to link to particular sections.

A different plugin, [`rehype-autolink-headings`][rehype-autolink-headings], adds
links to these headings back to themselves, which is useful as it lets users
more easily link to particular sections.

## 安装

This package is [ESM only](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c).
In Node.js (version 12.20+, 14.14+, or 16.0+), install with [npm][]:

```sh
npm install rehype-slug
```

In Deno with [`esm.sh`][esmsh]:

```js
import rehypeSlug from 'https://esm.sh/rehype-slug@5'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import rehypeSlug from 'https://esm.sh/rehype-slug@5?bundle'
</script>
```

## 使用

Say we have the following file `example.html`:

```html
<h1 id="some-id">Lorem ipsum</h1>
<h2>Dolor sit amet 😪</h2>
<h3>consectetur &amp; adipisicing</h3>
<h4>elit</h4>
<h5>elit</h5>
```

And our module `example.js` looks as follows:

```js
import { read } from 'to-vfile'
import { rehype } from 'rehype'
import rehypeSlug from 'rehype-slug'

const file = await rehype()
  .data('settings', { fragment: true })
  .use(rehypeSlug)
  .process(await read('example.html'))

console.log(String(file))
```

Now, running `node example.js` yields:

```html
<h1 id="some-id">Lorem ipsum</h1>
<h2 id="dolor-sit-amet-">Dolor sit amet 😪</h2>
<h3 id="consectetur--adipisicing">consectetur &#x26; adipisicing</h3>
<h4 id="elit">elit</h4>
<h5 id="elit-1">elit</h5>
```

## API

This package exports no identifiers.
The default export is `rehypeSlug`.

### `unified().use(rehypeSlug[, options])`

Add `id`s to headings.

##### `options`

Configuration (可选).

###### `options.prefix`

Prefix to add in front of `id`s (`string`, default: `''`).

## Types

这个包是完全使用[TypeScript][]类型的。
它导出附加类型`Options`。

## 兼容性

Projects maintained by the unified collective are compatible with all maintained
versions of Node.js.
As of now, that is Node.js 12.20+, 14.14+, and 16.0+.
Our projects sometimes work with older versions, but this is not guaranteed.

This plugin works with `rehype-parse` version 1+, `rehype-stringify` version 1+,
`rehype` version 1+, and `unified` version 4+.

## 安全

Use of `rehype-slug` can open you up to a [cross-site scripting (XSS)][xss]
attack as it sets `id` attributes on headings, which causes what is known
as “DOM clobbering”.
Please use [`rehype-sanitize`][rehype-sanitize] and see its
[Example: headings (DOM clobbering)][rehype-sanitize-example] for information on
how to properly solve it.

## 相关的

- [`rehype-autolink-headings`][rehype-autolink-headings]
  — add links to headings with IDs back to themselves

## 贡献

See [`contributing.md`][contributing] in [`rehypejs/.github`][health] for ways
to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## 许可证

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/rehypejs/rehype-slug/workflows/main/badge.svg
[build]: https://github.com/rehypejs/rehype-slug/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/rehypejs/rehype-slug.svg
[coverage]: https://codecov.io/github/rehypejs/rehype-slug
[downloads-badge]: https://img.shields.io/npm/dm/rehype-slug.svg
[downloads]: https://www.npmjs.com/package/rehype-slug
[size-badge]: https://img.shields.io/bundlephobia/minzip/rehype-slug.svg
[size]: https://bundlephobia.com/result?p=rehype-slug
[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg
[backers-badge]: https://opencollective.com/unified/backers/badge.svg
[collective]: https://opencollective.com/unified
[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg
[chat]: https://github.com/rehypejs/rehype/discussions
[npm]: https://docs.npmjs.com/cli/install
[esmsh]: https://esm.sh
[health]: https://github.com/rehypejs/.github
[contributing]: https://github.com/rehypejs/.github/blob/HEAD/contributing.md
[support]: https://github.com/rehypejs/.github/blob/HEAD/support.md
[coc]: https://github.com/rehypejs/.github/blob/HEAD/code-of-conduct.md
[license]: license
[author]: https://wooorm.com
[typescript]: https://www.typescriptlang.org
[unified]: https://github.com/unifiedjs/unified
[rehype]: https://github.com/rehypejs/rehype
[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
[github-slugger]: https://github.com/Flet/github-slugger
[rehype-autolink-headings]: https://github.com/rehypejs/rehype-autolink-headings
[rehype-sanitize]: https://github.com/rehypejs/rehype-sanitize
[rehype-sanitize-example]: https://github.com/rehypejs/rehype-sanitize#example-headings-dom-clobbering
