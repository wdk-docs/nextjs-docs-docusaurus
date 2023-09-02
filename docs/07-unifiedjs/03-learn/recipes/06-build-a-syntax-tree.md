---
group: recipe
index: 8
title: 构建语法树
description: How to build content with syntax trees
tags:
  - unist
  - mdast
  - hast
  - xast
author: Christian Murphy
authorGithub: ChristianMurphy
published: 2020-06-09
modified: 2020-06-15
---

## 如何构建语法树

在添加或替换内容时，构建新的(片段)语法树通常很有用。
可以使用普通对象和数组文字(JSON)或使用小实用程序以编程方式创建树。
最后，甚至可以使用JSX来构建树。

### JSON

创建树的最基本方法是使用普通对象和数组。
为了防止类型错误，可以使用给定语法树语言的类型进行检查，在本例中为mdast:

```ts
import type { Root } from 'mdast'

// Note the `: Root` is a TypeScript annotation. Remove it (and the import) for plain JavaScript.
const mdast: Root = {
  type: 'root',
  children: [
    {
      type: 'paragraph',
      children: [
        {
          type: 'text',
          value: 'example',
        },
      ],
    },
  ],
}
```

#### `unist-builder`

也可以使用[`unist-builder`][u]来构建树。
它允许更简洁的，类似于“hyperscript”的语法(也类似于`React.createElement`):

```js
import { u } from 'unist-builder'

const mdast = u('root', [u('paragraph', [u('text', 'example')])])
```

#### `hastscript`

当使用hast (HTML)时，可以使用[`hastscript`][h]。

```js
import { h, s } from 'hastscript'

console.log(
  h('div#some-id.foo', [
    h('span', 'some text'),
    h('input', { type: 'text', value: 'foo' }),
    h('a.alpha.bravo.charlie', { download: true }, 'delta'),
  ]),
)

// SVG:
console.log(
  s('svg', { xmlns: 'http://www.w3.org/2000/svg', viewbox: '0 0 500 500' }, [
    s('title', 'SVG `<circle>` element'),
    s('circle', { cx: 120, cy: 120, r: 100 }),
  ]),
)
```

`hastscript` 也可以用作JSX配置注释:

```jsx
/** @jsx h @jsxFrag null */
import { h } from 'hastscript'

console.log(
  <form method="POST">
    <input type="text" name="foo" />
    <input type="text" name="bar" />
    <input type="submit" name="send" />
  </form>,
)
```

#### `xastscript`

使用xast (XML)时, [`xastscript`][x] 可以使用.

```js
import { x } from 'xastscript'

console.log(
  x('album', { id: 123 }, [
    x('name', 'Exile in Guyville'),
    x('artist', 'Liz Phair'),
    x('releasedate', '1993-06-22'),
  ]),
)
```

`xastscript` 也可以用作JSX配置注释:

```jsx
/** @jsx x @jsxFrag null */
import { x } from 'xastscript'

console.log(
  <album id={123}>
    <name>Born in the U.S.A.</name>
    <artist>Bruce Springsteen</artist>
    <releasedate>1984-04-06</releasedate>
  </album>,
)
```

<!-- Definitions -->

[u]: https://github.com/syntax-tree/unist-builder
[h]: https://github.com/syntax-tree/hastscript
[x]: https://github.com/syntax-tree/xastscript
