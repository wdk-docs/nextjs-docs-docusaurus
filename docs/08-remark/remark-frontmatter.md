# remark-frontmatter

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

[**remark**][remark] 插件支持frontmatter (YAML, TOML, 和更多的).

## Contents

- [remark-frontmatter](#remark-frontmatter)
  - [Contents](#contents)
  - [这是什么?](#这是什么)
  - [我应该什么时候使用这个?](#我应该什么时候使用这个)
  - [安装](#安装)
  - [使用](#使用)
  - [API](#api)
    - [`unified().use(remarkFrontmatter[, options])`](#unifieduseremarkfrontmatter-options)
      - [`options`](#options)
      - [`preset`](#preset)
      - [`Matter`](#matter)
  - [例子](#例子)
    - [示例:自定义标记](#示例自定义标记)
    - [示例:自定义围栏](#示例自定义围栏)
    - [例如:frontmatter作为元数据](#例如frontmatter作为元数据)
    - [例如:MDX中的前页](#例如mdx中的前页)
  - [语法](#语法)
  - [语法树](#语法树)
  - [Types](#types)
  - [兼容性](#兼容性)
  - [安全](#安全)
  - [相关的](#相关的)
  - [贡献](#贡献)
  - [许可证](#许可证)

## 这是什么?

这个包是一个[unified][] ([remark][])插件，用于添加对YAML, TOML和其他前端的支持。
您可以使用它来添加对解析和序列化此语法扩展的支持。

**unified** 项目是否使用抽象语法树转换内容(ASTs).
**remark** 增加对unified markdown的支持。
**mdast** 是remark使用的markdownAST。
**micromark** 是我们使用的markdown解析器。
这是一个remark插件，增加了对frontmatter语法和AST的支持。

## 我应该什么时候使用这个?

Frontmatter是内容前面的元数据格式。
它通常是用YAML编写的，并且经常与markdown一起使用。
当您希望具有一定标记经验的作者配置内容的显示位置或方式，或提供有关内容的元数据时，这种机制非常有效。
Frontmatter并不是在任何地方都有效，所以它使得markdown不那么便携。
一个很好的用例是由(静态)站点生成器呈现的markdown。

## 安装

此包是[仅限ESM](https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c).
在Node.js(12.20+， 14.14+或16.0+版本)中，使用[npm][]安装:

```sh
npm install remark-frontmatter
```

在 Deno 中使用 [`esm.sh`][esmsh]：

```js
import remarkFrontmatter from 'https://esm.sh/remark-frontmatter@4'
```

在浏览器中使用[`esm.sh`][esmsh]:

```html
<script type="module">
  import remarkFrontmatter from 'https://esm.sh/remark-frontmatter@4?bundle'
</script>
```

## 使用

假设我们有以下文件, `example.md`:

```markdown
+++
title = "New Website"
+++

# Other markdown
```

我们的模块，`example.js`，看起来如下:

```js
import { read } from 'to-vfile'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkFrontmatter from 'remark-frontmatter'
import remarkStringify from 'remark-stringify'

const file = await unified()
  .use(remarkParse)
  .use(remarkStringify)
  .use(remarkFrontmatter, ['yaml', 'toml'])
  .use(() => (tree) => {
    console.dir(tree)
  })
  .process(await read('example.md'))

console.log(String(file))
```

现在，运行`node example`会得到:

```js
{
  type: 'root',
  children: [
    {type: 'toml', value: 'title = "New Website"', position: [Object]},
    {type: 'heading', depth: 1, children: [Array], position: [Object]}
  ],
  position: {
    start: {line: 1, column: 1, offset: 0},
    end: {line: 6, column: 1, offset: 48}
  }
}
```

```markdown
+++
title = "New Website"
+++

# Other markdown
```

## API

此包不导出任何标识符。
默认导出为`remarkFrontmatter`。

### `unified().use(remarkFrontmatter[, options])`

配置remark，使其能够解析和序列化前端内容(YAML、TOML等)。
不解析其中的数据:[创建自己的插件][create-plugin]来做。

##### `options`

一个`preset` 或 `Matter`，或它们的数组，定义所有支持的前端事项 (默认: `'yaml'`).

##### `preset`

要么`'yaml'` 要么 `'toml'`:

- `'yaml'` — `Matter` 定义为 `{type: 'yaml', marker: '-'}`
- `'toml'` — `Matter` 定义为 `{type: 'toml', marker: '+'}`

##### `Matter`

带有`type`和`marker`或`fence`的对象:

- `type` (`string`)
  — 键入要标记为
- `marker` (`string` 或 `{open: string, close: string}`)
  — 用来建造栅栏的人物。
  通过为一个对象提供`open` 和 `close`，可以使用不同的字符来打开和关闭栅栏。
  例如，字符`'-'`将导致`'---'`被用作围栏
- `fence` (`string` 或 `{open: string, close: string}`)
  — 用作完整围栏的绳子。
  通过为对象提供`open` 和 `close`，可以使用不同的值来打开和关闭围栏。
  如果栅栏包含不同的字符或长度，也可以使用这个方法
- `anywhere` (`boolean`, 默认: `false`)
  – 如果 `true`, 在文件的任何地方都可以找到问题。
  如果 `false` (默认), 只识别文档开头的内容

## 例子

### 示例:自定义标记

一个带有不同打开和关闭标记的自定义首页，重复3次，看起来像这样:

```text
<<<
data
>>>

# hi
```

…可支持:

```js
// …
.use(remarkFrontmatter, {type: 'custom', marker: {open: '<', close: '>'}})
// …
```

### 示例:自定义围栏

一个带有自定义栅栏的自定义门面，不像这样重复:

```text
{
  "key": "value"
}

# hi
```

…可以支持:

```js
// …
.use(remarkFrontmatter, {type: 'json', fence: {open: '{', close: '}'}})
// …
```

### 例如:frontmatter作为元数据

这个插件处理markdown中frontmatter的语法。
它不会像YAML或TOML那样解析前端内容并将其公开到某个地方。

在unified中，有关于文件的元数据的位置: [`file.data`][file-data].
对于frontmatter，通常将其暴露在`file.data.matter`处。

我们可以做一个插件来做这个。
这个例子使用了实用程序[`vfile-matter`][vfile-matter]，它是特定于YAML的。
要支持其他数据语言，请查看此实用程序以获得灵感。

```js
import { matter } from 'vfile-matter'

/**
 * Plugin to parse YAML frontmatter and expose it at `file.data.matter`.
 *
 * @type {import('unified').Plugin<Array<void>>}
 */
export default function myUnifiedPluginHandlingYamlMatter() {
  return function (_, file) {
    matter(file)
  }
}
```

这个插件可以这样使用:

```js
import { read } from 'to-vfile'
import { unified } from 'unified'
import remarkParse from 'remark-parse'
import remarkFrontmatter from 'remark-frontmatter'
import remarkStringify from 'remark-stringify'
import myUnifiedPluginHandlingYamlMatter from './my-unified-plugin-handling-yaml-matter.js'

const file = await unified()
  .use(remarkParse)
  .use(remarkStringify)
  .use(remarkFrontmatter)
  .use(myUnifiedPluginHandlingYamlMatter)
  .process(await read('example.md'))

console.log(file.data.matter) // Logs an object.
```

### 例如:MDX中的前页

MDX能够从中导出数据，markdown则不能。
在编写MDX时，可以编写`export` 语句并通过它们公开任意数据。
也可以编写frontmatter，然后让插件将其转换为导出语句。

要自动将frontmatter转换为导出语句，使用[`remark-mdx-frontmatter`][remark-mdx-frontmatter]
这个插件可以这样使用:

```js
import { read, write } from 'to-vfile'
import { compile } from '@mdx-js/mdx'
import remarkFrontmatter from 'remark-frontmatter'
import remarkMdxFrontmatter from 'remark-mdx-frontmatter'

const file = compile(await read('input.mdx'), {
  remarkPlugins: [
    remarkFrontmatter,
    [remarkMdxFrontmatter, { name: 'matter' }],
  ],
})
file.path = 'output.js'
await write(file)

const mod = await import('./output.js')
console.log(mod.matter) // Logs an object.
```

## 语法

这个插件应用一个micromark扩展来解析语法。
请参阅其自述以了解其工作原理:

- [`micromark-extension-frontmatter`](https://github.com/micromark/micromark-extension-frontmatter)

支持的语法取决于给定的配置。

## 语法树

这个插件应用一个mdast实用程序来构建和序列化AST。
请参阅其自述以了解其工作原理:

- [`mdast-util-frontmatter`](https://github.com/syntax-tree/mdast-util-frontmatter)

树中支持的节点类型取决于给定的配置。

## Types

这个包是完全使用[TypeScript][]类型的。
默认情况下，`@types/mdast`中支持YAML节点类型。
要添加其他节点类型，通过将它们添加到`FrontmatterContentMap`来注册它们:

```ts
import type { Literal } from 'mdast'

interface TOML extends Literal {
  type: 'toml'
}

declare module 'mdast' {
  interface FrontmatterContentMap {
    // Allow using toml nodes defined by `remark-frontmatter`.
    toml: TOML
  }
}
```

## 兼容性

统一集体维护的项目兼容所有维护的Node.js版本。
到目前为止，这是Node.js 12.20+、14.14+和16.0+。
我们的项目有时会使用旧版本，但这不能保证。

此插件适用于统一版本6+和remark版本13+。

## 安全

使用`remark-frontmatter`不涉及[**rehype**][rehype] ([**hast**][hast])或用户内容，因此没有[跨站点脚本(XSS)][XSS]攻击的机会。

## 相关的

- [`remark-yaml-config`](https://github.com/remarkjs/remark-yaml-config)
  — 从YAML配置中配置注释
- [`remark-gfm`](https://github.com/remarkjs/remark-gfm)
  — 支持 GFM (autolink literals, footnotes, strikethrough, tables, tasklists)
- [`remark-mdx`](https://github.com/mdx-js/mdx/tree/main/packages/remark-mdx)
  — 支持MDX (JSX, expressions, ESM)
- [`remark-directive`](https://github.com/remarkjs/remark-directive)
  — 支持指令
- [`remark-math`](https://github.com/remarkjs/remark-math)
  — 支持数学

## 贡献

See [`contributing.md`][contributing] in [`remarkjs/.github`][health] for ways
to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## 许可证

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/remarkjs/remark-frontmatter/workflows/main/badge.svg
[build]: https://github.com/remarkjs/remark-frontmatter/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/remarkjs/remark-frontmatter.svg
[coverage]: https://codecov.io/github/remarkjs/remark-frontmatter
[downloads-badge]: https://img.shields.io/npm/dm/remark-frontmatter.svg
[downloads]: https://www.npmjs.com/package/remark-frontmatter
[size-badge]: https://img.shields.io/bundlephobia/minzip/remark-frontmatter.svg
[size]: https://bundlephobia.com/result?p=remark-frontmatter
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
[unified]: https://github.com/unifiedjs/unified
[remark]: https://github.com/remarkjs/remark
[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
[typescript]: https://www.typescriptlang.org
[rehype]: https://github.com/rehypejs/rehype
[hast]: https://github.com/syntax-tree/hast
[create-plugin]: https://unifiedjs.com/learn/guide/create-a-plugin/
[file-data]: https://github.com/vfile/vfile#filedata
[vfile-matter]: https://github.com/vfile/vfile-matter
[remark-mdx-frontmatter]: https://github.com/remcohaszing/remark-mdx-frontmatter
