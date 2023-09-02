# `@mdx-js/react`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

React context for MDX.

<!-- more -->

## Contents

- [`@mdx-js/react`](#mdx-jsreact)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [Use](#use)
  - [API](#api)
    - [`MDXProvider(props?)`](#mdxproviderprops)
      - [`props`](#props)
        - [`props.components`](#propscomponents)
        - [`props.disableParentContext`](#propsdisableparentcontext)
        - [`props.children`](#propschildren)
      - [返回](#返回)
    - [`useMDXComponents(components?)`](#usemdxcomponentscomponents) - [`components`](#components) - [返回](#返回-1)
    - [`MDXContext`](#mdxcontext)
    - [`withMDXComponents(Component)`](#withmdxcomponentscomponent)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个基于上下文的组件提供程序，用于将React与MDX结合起来。

## 我应该什么时候使用这个?

这个包不需要MDX与React一起工作。
关于何时以及如何使用MDX提供程序，请参阅§Using MDX中的[¶MDX提供程序][use-provider]。

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/react
```

[yarn][]:

```sh
yarn add @mdx-js/react
```

## Use

```js
import {MDXProvider} from '@mdx-js/react'
import Post from './post.mdx'
// ^-- Assumes an integration is used to compile MDX to JS, such as
// `@mdx-js/esbuild`, `@mdx-js/loader`, `@mdx-js/node-loader`, or
// `@mdx-js/rollup`, and that it is configured with
// `options.providerImportSource: '@mdx-js/react'`.

const components = {
  em: props => <i {...props} />
}

<MDXProvider components={components}>
  <Post />
</MDXProvider>
```

注意，你不必使用`MDXProvider`，可以直接传递组件:

```diff
-<MDXProvider components={components}>
-  <Post />
-</MDXProvider>
+<Post components={components} />
```

关于如何开始使用MDX和React，请参见§开始中的[¶React][start-react]。
有关如何使用MDX提供程序，请参阅§使用MDX中的[¶MDX提供程序][use-provider]。

## API

这个包导出以下标识符:`MDXContext`, `MDXProvider`,`useMDXComponents`, 和 `withMDXComponents`.
没有默认的导出。

### `MDXProvider(props?)`

MDX上下文的提供程序。

##### `props`

配置 (`Object`, 可选).

###### `props.components`

JSX组件到React组件的名称映射(`Record<string, string|Component|Components>`, 可选).

###### `props.disableParentContext`

关闭外部组件上下文 (`boolean`, 默认的: `false`).

###### `props.children`

Children (JSX 元素,可选).

##### 返回

JSX 元素.

### `useMDXComponents(components?)`

从MDX上下文中获取当前组件。

###### `components`

附加组件 (`Components`) 使用一个函数来获取当前组件并过滤/合并/更改它们 (`(currentComponents: Components) => Components`).

###### 返回

`Components`.

### `MDXContext`

> 🪦 **弃用**: 不建议使用此导出，因为它暴露了应该隐藏的内部内容。
> 它可能会在未来的主要版本中被删除。
> 请使用`useMDXComponents`来获取基于上下文的组件，使用`MDXProvider`来设置基于上下文的组件。

MDX的React上下文 (`React.Context`).

### `withMDXComponents(Component)`

> 🪦 **弃用**: This export is not recommended for use.
> It might be removed in a future major release.
> Please use `useMDXComponents` to get context based components instead.

Create a HOC of `Components` which is given the current context based MDX
components.

## Types

This package is fully typed with [TypeScript][].

To enable types for imported `.mdx`, `.md`, etcetera files, you should make sure
the TypeScript `JSX` namespace is typed.
This is done by installing and using the types of your framework, such as
[`@types/react`](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/HEAD/types/react).
Then you can install and use
[`@types/mdx`](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/HEAD/types/mdx),
which adds types to import statements of supported files.

## 安全

详见我们网站上的[§Security][Security]。

## 贡献

请参阅我们网站上的[§Contribute][Contribute]了解开始的方法。
参见[§Support][Support]获取帮助的方法。

这个项目有[行为准则][coc]。
通过与此存储库、组织或社区进行交互，您同意遵守其条款。

## 许可证

[MIT][] © Compositor and [Vercel][]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/react.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/react
[size-badge]: https://img.shields.io/bundlephobia/minzip/@mdx-js/react.svg
[size]: https://bundlephobia.com/result?p=@mdx-js/react
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/react/license
[vercel]: https://vercel.com
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[start-react]: https://mdxjs.com/getting-started/#react
[use-provider]: https://mdxjs.com/docs/using-mdx/#mdx-provider
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
