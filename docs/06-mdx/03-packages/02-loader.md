# `@mdx-js/loader`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

MDX的webpack加载器。

<!-- more -->

## Contents

- [`@mdx-js/loader`](#mdx-jsloader)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api) - [`options`](#options) - [Note: Babel](#note-babel)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个支持MDX的webpack加载器。

## 我应该什么时候使用这个?

如果你正在使用webpack(或者其他使用webpack的工具，比如Next.js)，这个集成是很有用的。

这种集成可以与Babel加载器相结合，以支持非标准的JSX运行时(例如Vue)，或者将现代JavaScript功能编译为用户支持的功能。

如果要评估MDX代码，则可以手动使用较低级别的编译器(`@mdx-js/mdx`)。

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/loader
```

[yarn][]:

```sh
yarn add @mdx-js/loader
```

## 使用

Add something along these lines to your `webpack.config.js`:

```js
module.exports = {
  module: {
    // …
    rules: [
      // …
      {
        test: /\.mdx?$/,
        use: [
          {
            loader: '@mdx-js/loader',
            /** @type {import('@mdx-js/loader').Options} */
            options: {},
          },
        ],
      },
    ],
  },
}
```

See also [¶ Create React App (CRA)][cra], [¶ Next.js][next], and
[¶ Vue CLI][vue-cli], if you’re using webpack through them, for more info.

## API

This package exports a [webpack][] plugin as the default export.

Source maps are supported based on how you configure webpack.
You do not need to pass `options.SourceMapGenerator`.

###### `options`

`options` are the same as [`compile` from `@mdx-js/mdx`][options].

###### Note: Babel

If you use modern JavaScript features you might want to use Babel through
[`babel-loader`][babel-loader] to compile to code that works:

```js
// …
use: [
  // Note that Webpack runs right-to-left: `@mdx-js/loader` is used first, then
  // `babel-loader`.
  { loader: 'babel-loader', options: {} },
  {
    loader: '@mdx-js/loader',
    /** @type {import('@mdx-js/loader').Options} */
    options: {},
  },
]
// …
```

## Types

This package is fully typed with [TypeScript][].
See [§ Types][types] on our website for information.

An `Options` type is exported, which represents acceptable configuration.

## 安全

See [§ Security][security] on our website for information.

## 贡献

See [§ Contribute][contribute] on our website for ways to get started.
See [§ Support][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## 许可证

[MIT][] © Compositor and [Vercel][]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/loader.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/loader
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/loader/license
[vercel]: https://vercel.com
[webpack]: https://webpack.js.org
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[types]: https://mdxjs.com/getting-started/#types
[security]: https://mdxjs.com/getting-started/#security
[options]: https://mdxjs.com/packages/mdx/#compilefile-options
[typescript]: https://www.typescriptlang.org
[babel-loader]: https://webpack.js.org/loaders/babel-loader/
[cra]: https://mdxjs.com/getting-started/#create-react-app-cra
[next]: https://mdxjs.com/getting-started/#nextjs
[vue-cli]: https://mdxjs.com/getting-started/#vue-cli
