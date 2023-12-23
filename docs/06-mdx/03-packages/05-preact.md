# `@mdx-js/preact`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

Preact context for MDX.

<!-- more -->

## Contents

- [`@mdx-js/preact`](#mdx-jspreact)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`MDXProvider(props?)`](#mdxproviderprops)
      - [`props`](#props)
        - [`props.components`](#propscomponents)
        - [`props.disableParentContext`](#propsdisableparentcontext)
        - [`props.children`](#propschildren)
      - [返回](#返回)
    - [`useMDXComponents(components?)`](#usemdxcomponentscomponents) - [`components`](#components) - [Returns](#returns)
    - [`MDXContext`](#mdxcontext)
    - [`withMDXComponents(Component)`](#withmdxcomponentscomponent)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个基于上下文的组件提供程序，用于组合Preact和MDX。

## 我应该什么时候使用这个?

MDX与Preact一起工作不需要这个包。
关于何时以及如何使用MDX提供程序，请参阅§Using MDX中的[¶MDX提供程序][use-provider]。

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/preact
```

[yarn][]:

```sh
yarn add @mdx-js/preact
```

## 使用

```js
import {MDXProvider} from '@mdx-js/preact'
import Post from './post.mdx'
// ^-- Assumes an integration is used to compile MDX to JS, such as
// `@mdx-js/esbuild`, `@mdx-js/loader`, `@mdx-js/node-loader`, or
// `@mdx-js/rollup`, and that it is configured with
// `options.providerImportSource: '@mdx-js/preact'`.

const components = {
  em: props => <i {...props} />
}

<MDXProvider components={components}>
  <Post />
</MDXProvider>
```

Note that you don’t have to use `MDXProvider` and can pass components
directly:

```diff
-<MDXProvider components={components}>
-  <Post />
-</MDXProvider>
+<Post components={components} />
```

See [¶ Preact in § Getting started][start-preact] for how to get started with
MDX and Preact.
See [¶ MDX provider in § Using MDX][use-provider] for how to use an MDX
provider.

## API

This package exports the following identifiers: `MDXContext`, `MDXProvider`,
`useMDXComponents`, and `withMDXComponents`.
There is no default export.

### `MDXProvider(props?)`

Provider for MDX context.

##### `props`

Configuration (`Object`, optional).

###### `props.components`

Mapping of names for JSX components to Preact components
(`Record<string, string|Component|Components>`, optional).

###### `props.disableParentContext`

Turn off outer component context (`boolean`, default: `false`).

###### `props.children`

Children (JSX elements, optional).

##### 返回

JSX element.

### `useMDXComponents(components?)`

Get current components from the MDX Context.

###### `components`

Additional components (`Components`) to use or a function that takes the current
components and filters/merges/changes them (`(currentComponents: Components) =>
Components`).

###### 返回

`Components`.

### `MDXContext`

> 🪦 **Deprecated**: This export is not recommended for use as it exposes
> internals which should be hidden.
> It might be removed in a future major release.
> Please use `useMDXComponents` to get context based components and
> `MDXProvider` to set context based components instead.

The Preact Context for MDX (`Preact.Context`).

### `withMDXComponents(Component)`

> 🪦 **Deprecated**: This export is not recommended for use.
> It might be removed in a future major release.
> Please use `useMDXComponents` to get context based components instead.

Create a HOC of `Components` which is given the current context based MDX
components.

## Types

This package is fully typed with [TypeScript][].

To enable types for imported `.mdx`, `.md`, etcetera files, you should make sure
the TypeScript `JSX` namespace is typed.
This is done by installing and using the types of your framework, as in
[`preact`](https://github.com/preactjs/preact).
Then you can install and use
[`@types/mdx`](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/HEAD/types/mdx),
which adds types to import statements of supported files.

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
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/preact.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/preact
[size-badge]: https://img.shields.io/bundlephobia/minzip/@mdx-js/preact.svg
[size]: https://bundlephobia.com/result?p=@mdx-js/preact
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/preact/license
[vercel]: https://vercel.com
[start-preact]: https://mdxjs.com/getting-started/#preact
[use-provider]: https://mdxjs.com/docs/using-mdx/#mdx-provider
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
