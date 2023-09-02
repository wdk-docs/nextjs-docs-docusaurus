# `@mdx-js/node-loader`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

Node loader for MDX.

<!-- more -->

> 💡 **实验**: 这是一个实验包，可能不能很好地工作，并且可能在次要版本中进行更改。

## Contents

- [What is this?](#what-is-this)
- [When should I use this?](#when-should-i-use-this)
- [Install](#install)
- [Use](#use)
- [API](#api)
  - [`createLoader(options?)`](#createloaderoptions)
- [Types](#types)
- [Security](#security)
- [Contribute](#contribute)
- [License](#license)

## 这是什么?

这个包是一个支持MDX的Node ESM加载器。
[ESM加载器][loader]是Node中的一个实验性特性，预计会更改。
他们让项目“劫持”导入来做各种各样奇特的事情，在这种情况下，它让你`import`MD(X)文件。

## 我应该什么时候使用这个?

如果您正在使用Node并希望从文件系统导入MDX文件，那么这种集成非常有用。

如果你使用的是一个捆绑器(webpack, Rollup, esbuild)，或者一个站点构建器(Gatsby, Next.js)，或者一个自带捆绑器的构建系统(Vite, WMR)，你最好使用另一个集成:参见[§Integrations][Integrations]。

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/node-loader
```

[yarn][]:

```sh
yarn add @mdx-js/node-loader
```

## 使用

假设我们有一个MDX文档， `example.mdx`:

```mdx
export const Thing = () => <>World!</>

# Hello, <Thing />
```

…我们的模块`example.js`看起来如下:

```js
import { renderToStaticMarkup } from 'react-dom/server'
import React from 'react'
import Content from './example.mdx'

console.log(renderToStaticMarkup(React.createElement(Content)))
```

…然后用:

```sh
node --experimental-loader=@mdx-js/node-loader example.js
```

…yields:

```html
<h1>Hello, World!</h1>
```

## API

> 💡 **实验**: 这是一个实验包，可能不能很好地工作，并且可能在次要版本中进行更改。

这个包导出一个Node [ESM加载程序][loader]。
它还导出以下标识符:`createLoader`。

### `createLoader(options?)`

创建一个Node ESM加载器，将MDX编译为JS。

##### `options`

`options` are the same as [`compile` from `@mdx-js/mdx`][options].
支持一个额外字段:

###### `options.fixRuntimeWithoutExportMap`

Fix broken export maps (`boolean`, default: `true`).

Several JSX runtimes, notably React below 18 and Emotion below 11.10.0, don’t
have a proper export map set up.
Export maps are needed to map `xxx/jsx-runtime` to an actual file in ESM.
This option fixes React et al by turning those into `xxx/jsx-runtime.js`.

> 👉 **Note**: If you are using recent React, or other proper packages, you
> have to turn this field off.
> See the example below on how to configure your loader.
> Pass `fixRuntimeWithoutExportMap: false` in options to it.

###### 例子

`my-loader.js`:

```js
import { createLoader } from '@mdx-js/node-loader'

// Load is for Node 17+, the rest for 12-16.
const { load, getFormat, transformSource } = createLoader(/* Options… */)

export { load, getFormat, transformSource }
```

This example can then be used with `node --experimental-loader=./my-loader.js`.

Node itself does not yet support multiple loaders but it is possible to combine
multiple loaders with [`@node-loader/core`][node-loader-core].

## Types

这个包是完全使用[TypeScript][]类型的。
请参阅我们网站上的[§Types][Types]获取信息。

导出`Options`类型，它表示可接受的配置。

## 安全

详见我们网站上的[§Security][Security]。

## 贡献

请参阅我们网站上的[§Contribute][Contribute]了解开始的方法。
参见[§Support][Support]获取帮助的方法。

这个项目有[行为准则][coc]。
通过与此存储库、组织或社区进行交互，您同意遵守其条款。

## 许可证

[MIT][] © [Titus Wormer][author]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/node-loader.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/node-loader
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/node-loader/license
[author]: https://wooorm.com
[loader]: https://nodejs.org/api/esm.html#esm_loaders
[integrations]: https://mdxjs.com/getting-started/#integrations
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[types]: https://mdxjs.com/getting-started/#types
[security]: https://mdxjs.com/getting-started/#security
[options]: https://mdxjs.com/packages/mdx/#compilefile-options
[typescript]: https://www.typescriptlang.org
[node-loader-core]: https://github.com/node-loader/node-loader-core
