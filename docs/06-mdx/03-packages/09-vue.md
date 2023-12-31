# `@mdx-js/vue`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

Vue context for MDX.

<!-- more -->

## Contents

- [`@mdx-js/vue`](#mdx-jsvue)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`MDXProvider(props?)`](#mdxproviderprops)
      - [`props`](#props)
        - [`props.components`](#propscomponents)
      - [Returns](#returns)
    - [`useMDXComponents()`](#usemdxcomponents) - [Returns](#returns-1)
  - [Types](#types)
  - [安全](#安全)
  - [贡献](#贡献)
  - [License](#license)

## 这是什么?

这个包是一个基于上下文的组件提供程序，用于将Vue与MDX结合起来。

## 我应该什么时候使用这个?

This package is not needed for MDX to work with Vue.
See [¶ MDX provider in § Using MDX][use-provider] for when and how to use an MDX
provider.

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/vue
```

[yarn][]:

```sh
yarn add @mdx-js/vue
```

## 使用

```js
import { MDXProvider } from '@mdx-js/vue'
import { createApp } from 'vue'
import Post from './post.mdx'
// ^-- Assumes an integration is used to compile MDX to JS, such as
// `@mdx-js/esbuild`, `@mdx-js/loader`, `@mdx-js/node-loader`, or
// `@mdx-js/rollup`, and that it is configured with
// `options.providerImportSource: '@mdx-js/vue'`.

createApp({
  data() {
    return { components: { h1: 'h2' } }
  },
  template:
    '<MDXProvider v-bind:components="components"><Post /></MDXProvider>',
  components: { MDXProvider, Post },
})
```

Note that you don’t have to use `MDXProvider` and can pass components
directly:

```diff
-createApp({
-  data() {
-    return {components: {h1: 'h2'}}
-  },
-  template: '<MDXProvider v-bind:components="components"><Post /></MDXProvider>',
-  components: {MDXProvider, Post}
-})
+createApp(Post, {components})
```

See [¶ Vue in § Getting started][start-vue] for how to get started with MDX and
Vue.
See [¶ MDX provider in § Using MDX][use-provider] for how to use an MDX
provider.

## API

This package exports the following identifiers: `MDXProvider` and
`useMDXComponents`.
There is no default export.

### `MDXProvider(props?)`

Provider for MDX context.

##### `props`

Configuration (`Object`, optional).

###### `props.components`

Mapping of names for JSX components to Vue components
(`Record<string, string|Component|Components>`, optional).

##### 返回

Fragment (with the default slot if given).

### `useMDXComponents()`

Get current components from the MDX Context.

###### 返回

`Components`.

## Types

This package is fully typed with [TypeScript][].

## 安全

See [§ Security][security] on our website for information.

## 贡献

See [§ Contribute][contribute] on our website for ways to get started.
See [§ Support][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## License

[MIT][] © Compositor and [Vercel][]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/vue.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/vue
[size-badge]: https://img.shields.io/bundlephobia/minzip/@mdx-js/vue.svg
[size]: https://bundlephobia.com/result?p=@mdx-js/vue
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/vue/license
[vercel]: https://vercel.com
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[start-vue]: https://mdxjs.com/getting-started/#vue
[use-provider]: https://mdxjs.com/docs/using-mdx/#mdx-provider
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
