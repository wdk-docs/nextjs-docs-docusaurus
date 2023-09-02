# remark-mdx-frontmatter

[![github actions](https://github.com/remcohaszing/remark-mdx-frontmatter/actions/workflows/ci.yaml/badge.svg)](https://github.com/remcohaszing/remark-mdx-frontmatter/actions/workflows/ci.yaml)
[![npm](https://img.shields.io/npm/v/remark-mdx-frontmatter)](https://www.npmjs.com/package/remark-mdx-frontmatter)
[![prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg)](https://prettier.io)
[![codecov](https://codecov.io/gh/remcohaszing/remark-mdx-frontmatter/branch/main/graph/badge.svg)](https://codecov.io/gh/remcohaszing/remark-mdx-frontmatter)

一个[remark](https://remark.js.org)插件，用于将前文元数据转换为MDX导出

## Table of Contents

- [remark-mdx-frontmatter](#remark-mdx-frontmatter)
  - [Table of Contents](#table-of-contents)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [选项](#选项)
  - [许可证](#许可证)

## 安装

这个包依赖于[remark-frontmatter](https://github.com/remarkjs/remark-frontmatter)的AST输出。

```sh
npm install remark-frontmatter remark-mdx-frontmatter
```

## 使用

这个remark插件接受frontmatter内容，并将其作为JavaScript导出输出。
支持YAML和TOML前页数据。

例如，给定一个名为`example.mdx`的文件，其中包含以下内容:

```mdx
---
hello: frontmatter
---

Rest of document
```

下面的脚本:

```js
import { readFile } from 'node:fs/promises'

import { compile } from '@mdx-js/mdx'
import remarkFrontmatter from 'remark-frontmatter'
import remarkMdxFrontmatter from 'remark-mdx-frontmatter'

const { contents } = await compile(await readFile('example.mdx'), {
  jsx: true,
  remarkPlugins: [remarkFrontmatter, remarkMdxFrontmatter],
})
console.log(contents)
```

约收益率:

```jsx
export const frontmatter = {
  hello: 'frontmatter',
}

export default function MDXContent() {
  return <p>Rest of document</p>
}
```

## API

默认导出是一个[remark](https://remark.js.org)插件。

### 选项

- `name`: 前端内容数据分配给的变量的标识符名称。 (默认: `frontmatter`).
- `parsers`: 节点类型到解析器的映射。每个键代表一个前端节点类型。
  该值是一个函数，它接受前端数据作为字符串，并返回解析后的数据。
  默认情况下，`yaml`节点将使用[`yaml`](https://github.com/eemeli/yaml)和`toml`节点使用[`toml`](https://github.com/BinaryMuse/toml-node)进行解析。

## 许可证

[MIT](LICENSE.md) © [Remco Haszing](https://github.com/remcohaszing)
