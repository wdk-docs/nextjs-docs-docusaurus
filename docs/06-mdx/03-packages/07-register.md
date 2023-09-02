# `@mdx-js/register`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

需要MDX的Node挂钩。

<!-- more -->

> 🪦 **遗产**: 不建议使用此包，因为它依赖于已弃用的Node特性。

## Contents

- [`@mdx-js/register`](#mdx-jsregister)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个支持MDX的Node CommonJS钩子。
[`require.extensions`][require-extensions]是Node中一个被弃用的特性，它允许项目“hijack” `require`调用来做一些奇特的事情，在这种情况下，它允许你`require` MD(X)文件。

## 我应该什么时候使用这个?

如果您正在使用Node，由于某种原因不得不使用CJS，并且希望从文件系统中获取MDX文件，那么这种集成非常有用。

此时，您最好使用`@mdx-js/node-loader`，即使它使用实验性的Node API。

## 安装

此包仅为[ESM][ESM]:需要Node 12+才能使用。

[npm][]:

```sh
npm install @mdx-js/register
```

[yarn][]:

```sh
yarn add @mdx-js/register
```

## 使用

假设我们有一个MDX文档， `example.mdx`:

```mdx
export const Thing = () => <>World!</>

# Hello, <Thing />
```

…我们的模块`example.cjs`看起来如下:

```js
'use strict'

const React = require('react')
const { renderToStaticMarkup } = require('react-dom/server')
const Content = require('./example.mdx')

console.log(renderToStaticMarkup(React.createElement(Content)))
```

…然后运行它:

```sh
node -r @mdx-js/register example.cjs
```

…收益率:

```html
<h1>Hello, World!</h1>
```

## API

> 🪦 **遗产**: 不建议使用此包，因为它依赖于已弃用的Node特性。

This package does not export anything.
It changes Node’s internals.

To pass options, you can make your own hook, such as this `my-hook.cjs`:

```js
'use strict'

const register = require('@mdx-js/register/lib/index.cjs')

register({
  /* Options… */
})
```

Which can then be used with `node -r ./my-hook.cjs`.

The register hook uses
[`evaluateSync`][eval-sync].
That means `import` (and `export … from`) are not supported when requiring
`.mdx` files.

## Types

This package is not typed as [TypeScript][] seems to not support `.cjs` files
yet.

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
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/register.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/register
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/register/license
[author]: https://wooorm.com
[require-extensions]: https://nodejs.org/api/modules.html#modules_require_extensions
[eval-sync]: https://mdxjs.com/packages/mdx/#evaluatesyncfile-options
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
