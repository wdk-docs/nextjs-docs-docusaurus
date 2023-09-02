# remark-mdx

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

remark 插件来支持MDX语法 (JSX, 表达式, import/exports).

<!-- more -->

## Contents

- [remark-mdx](#remark-mdx)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [Use](#use)
  - [API](#api)
    - [`unified().use(remarkMdx)`](#unifieduseremarkmdx)
  - [语法](#语法)
  - [语法树](#语法树)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个支持MDX语法的注释插件。

## 我应该什么时候使用这个?

这个插件是有用的，如果你正在处理MDX语法和集成的remark, rehype，和其他unified。
一些示例用例是当您想要检测语法或将其编译为JavaScript以外的东西时。

**remark** is an AST (abstract syntax tree) based transform project.
The layer under remark is called mdast, which is just the syntax tree without
the convention on how to transform.
mdast is useful when transforming to other formats.
Another layer underneath is micromark, which is just the parser and has support
for concrete tokens.
micromark is useful for linting and formatting.
`remark-mdx` is a small wrapper to integrate all of these.
Its parts can be used separately.

Typically though, you’d want to move a layer up: `@mdx-js/mdx`.
That package is the core compiler for turning MDX into JavaScript which
gives you the most control.
Or even higher: if you’re using a bundler (webpack, Rollup, esbuild), or a site
builder (Gatsby, Next.js) or build system (Vite, WMR) which comes with a
bundler, you’re better off using an integration: see
[§ Integrations][integrations].

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install remark-mdx
```

[yarn][]:

```sh
yarn add remark-mdx
```

## Use

```js
import { remark } from 'remark'
import remarkMdx from 'remark-mdx'

const file = remark()
  .use(remarkMdx)
  .processSync('import a from "b"\n\na <b /> c {1 + 1} d')

console.log(String(file))
```

Yields:

```mdx
import a from 'b'

a <b/> c {1 + 1} d
```

## API

This package exports no identifiers.
The default export is `remarkMdx`.

### `unified().use(remarkMdx)`

Configures remark so that it can parse and serialize MDX (JSX, expressions,
import/exports).
It doesn’t do anything with the syntax: you can
[create your own plugin][create-plugin] to transform them.

## 语法

This plugin applies several micromark extensions to parse the syntax.
See their readmes for parse details:

- [`micromark-extension-mdx-expression`](https://github.com/micromark/micromark-extension-mdx-expression#syntax)
  — expressions (`{1 + 1}`)
- [`micromark-extension-mdx-jsx`](https://github.com/micromark/micromark-extension-mdx-jsx#syntax)
  — JSX (`<div />`)
- [`micromark-extension-mdxjs-esm`](https://github.com/micromark/micromark-extension-mdxjs-esm#syntax)
  — ESM (`export x from 'y'`)
- [`micromark-extension-mdx-md`](https://github.com/micromark/micromark-extension-mdx-md#mdxmd)
  — Turn off HTML, autolinks, and indented code

## 语法树

This plugin applies several mdast utilities to build and serialize the AST.
See their readmes for the node types supported in the tree:

- [`mdast-util-mdx-expression`](https://github.com/syntax-tree/mdast-util-mdx-expression#syntax-tree)
  — expressions (`{1 + 1}`)
- [`mdast-util-mdx-jsx`](https://github.com/syntax-tree/mdast-util-mdx-jsx#syntax-tree)
  — JSX (`<div />`)
- [`mdast-util-mdxjs-esm`](https://github.com/syntax-tree/mdast-util-mdxjs-esm#syntax-tree)
  — ESM (`export x from 'y'`)

## Types

This package is fully typed with [TypeScript][].

If you’re working with the syntax tree, make sure to import this plugin
somewhere in your types, as that registers the new node types in the tree.

```js
/**
 * @typedef {import('remark-mdx')}
 */

import {visit} from 'unist-util-visit'

export default function myRemarkPlugin() => {
  /** @param {import('@types/mdast').Root} tree */
  return (tree) => {
    visit(tree, (node) => {
      // `node` can now be one of the nodes for JSX, expressions, or ESM.
    })
  }
}
```

Alternatively, in TypeScript, do:

```ts
/// <reference types="remark-mdx" />

import type {Root} from '@types/mdast'
import {visit} from 'unist-util-visit'

export default function myRemarkPlugin() => {
  return (tree: Root) => {
    visit(tree, (node) => {
      // `node` can now be one of the nodes for JSX, expressions, or ESM.
    })
  }
}
```

## 安全

See [§ Security][security] on our website for information.

## 贡献

See [§ Contribute][contribute] on our website for ways to get started.
See [§ Support][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## 许可证

[MIT][] © [Titus Wormer][author]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/remark-mdx.svg
[downloads]: https://www.npmjs.com/package/remark-mdx
[size-badge]: https://img.shields.io/bundlephobia/minzip/remark-mdx.svg
[size]: https://bundlephobia.com/result?p=remark-mdx
[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg
[backers-badge]: https://opencollective.com/unified/backers/badge.svg
[collective]: https://opencollective.com/unified
[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg
[chat]: https://github.com/mdx-js/mdx/discussions
[npm]: https://docs.npmjs.com/cli/install
[yarn]: https://classic.yarnpkg.com/docs/cli/add/
[contribute]: https://mdxjs.com/community/contribute/
[support]: https://mdxjs.com/community/support/
[coc]: https://github.com/mdx-js/.github/blob/main/code-of-conduct.md
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/remark-mdx/license
[author]: https://wooorm.com
[create-plugin]: https://unifiedjs.com/learn/guide/create-a-plugin/
[integrations]: https://mdxjs.com/getting-started/#integrations
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
