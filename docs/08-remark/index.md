# remark

[![remark][logo]][monorepo]

[**remark**][monorepo]是一个流行的免费开源工具，可以通过插件转换markdown。
Markdown是一种通常感觉比HTML更自然的标记语言。
插件可以检查和更改markdown。

- 要学习markdown，请参阅[备忘单和教程][cheat]
- 我们默认遵循[CommonMark][] (GFM使用[`remark-gfm`][remark-gfm]， MDX使用[`remark-mdx`][remark-mdx])
- 我们使用[`micromark`][micromark]进行解析，并使用[mdast][]作为语法树
- 要查找插件，请参见[`awesome-remark`][awesome]，[插件列表][plugins]和[`remark-plugin` 主题][topic]。

## 核心

Remark是一个由[插件](https://github.com/remarkjs/remark/blob/HEAD/doc/plugins.md#list-of-plugins)驱动的markdown处理器，是[unified](https://unifiedjs.com/)集合的一部分。
该项目[解析](https://github.com/remarkjs/remark/tree/HEAD/packages/remark-parse#readme)和[编译](https://github.com/remarkjs/remark/tree/HEAD/packages/remark-stringify#readme)markdown，并允许程序处理markdown，而无需编译为HTML([尽管它可以](https://github.com/remarkjs/remark-html))。
由[插件](https://github.com/remarkjs/remark/blob/HEAD/doc/plugins.md#list-of-plugins)提供支持，可以做各种各样的事情:检查markdown代码[风格](https://github.com/remarkjs/remark-lint)，添加[目录](https://github.com/remarkjs/remark-toc)或编译为[手册页](https://github.com/remarkjs/remark-man)。

在GitHub上查看项目或在AST Explorer上检查语法树。

## CLI

命令行接口功能强大，它支持API提供的所有设置，并内置支持根据给定的样式指南在项目中重新生成标记文件，并以漂亮的方式发出由插件触发的警告。

示例如下所示:

```md title='example.md'
- Hello

[World][]
```

现在，通过在终端中运行remark，我们将看到以下内容:

```sh
$ remark example.md --use remark-preset-lint-recommended
example.md
   1:3  warning  Incorrect list-item indent: add 2 spaces  list-item-indent
3:1-3:10  warning  Found reference to undefined definition   no-undefined-references

⚠ 2 warnings
```

在其[自述中](https://github.com/remarkjs/remark/tree/HEAD/packages/remark-cli)阅读有关CLI的更多信息。

## API

应用程序编程接口是注释和CLI的要点。
它是由`unified`提供的，因此请前往其[API文档](https://github.com/unifiedjs/unified#api)获取更多信息。

假设我们使用`remark`, [`remark-preset-lint-markdown-style-guide`](https://github.com/remarkjs/remark-lint/tree/HEAD/packages/remark-preset-lint-markdown-style-guide), [`remark-html`](https://github.com/remarkjs/remark-html), 和 [`vfile-reporter`](https://github.com/vfile/vfile-reporter)有以下`index.js`文件:

```js
import { reporter } from 'vfile-reporter'
import { remark } from 'remark'
import remarkPresetLintMarkdownStyleGuide from 'remark-preset-lint-markdown-style-guide'
import remarkHtml from 'remark-html'

remark()
  .use(remarkPresetLintMarkdownStyleGuide)
  .use(remarkHtml)
  .process('*emphasis* and _importance_')
  .then((file) => {
    console.log(String(file))
    console.error(reporter(file))
  })
```

现在，在安装依赖项并使用Node运行index.js之后，你会看到以下内容:

```sh
$ npm install remark vfile-reporter remark-html remark-preset-lint-markdown-style-guide
/Users/tilde/example
├── remark@14.0.1
├── remark-html@14.0.0
├── remark-preset-lint-markdown-style-guide@5.0.0
└── vfile-reporter@7.0.1

node index.js
<p><em>emphasis</em> and <em>importance</em></p>
example.md
1:16-1:28  warning  Emphasis should use `*` as a marker  emphasis-marker

⚠ 1 warning
```

## unified

Unified是一个用语法树处理文本并在它们之间进行转换的接口。
有三种语法连接到unified，每种语法都带有一个语法树定义，以及一个解析器和字符串分析器:
[mdast](https://github.com/syntax-tree/mdast)带有remark用于markdown，
[nlcst](https://github.com/syntax-tree/nlcst)带有[retext](https://github.com/retextjs/retext)用于散文，
而[hast](https://github.com/syntax-tree/hast)带有[rehype](https://github.com/rehypejs/rehype)用于HTML。

unified也在语法之间架起桥梁，例如markdown和HTML之间，如下所示:

```js title='index.js'
import { unified } from 'unified'
import stream from 'unified-stream'
import remarkParse from 'remark-parse'
import remarkToc from 'remark-toc'
import remarkRehype from 'remark-rehype'
import rehypeDocument from 'rehype-document'
import rehypeFormat from 'rehype-format'
import rehypeStringify from 'rehype-stringify'

const input = '## Markdown'

unified()
  .use(remarkParse)
  .use(remarkToc)
  .use(remarkRehype)
  .use(rehypeDocument, { title: 'Contents' })
  .use(rehypeFormat)
  .use(rehypeStringify)
  .process(input)
  .then((file) => {
    console.log(file)
  })
```

在[GitHub](https://github.com/unifiedjs/unified)上查看统一项目或阅读[网站](https://unifiedjs.com/)。

Remark是unified集合的一部分，它将处理内容作为结构化数据的组织聚集在一起。

- 有关我们的更多信息，请参见 [`unifiedjs.com`][site]
- 至于集体是如何被治理的，请看 [`unifiedjs/collective`][collective]
- 有关更新，请参阅在推特上 [@unifiedjs][twitter]

## 参与

:::note 👉

通过与本组织或社区互动，您同意遵守我们的[行为准则][coc].

:::

- 有关问题，请参见[支持][]
- 帮助，参见[贡献][]
- 赞助，请参见[OpenCollective][oc]或[GitHub赞助商][ghs]

[logo]: https://raw.githubusercontent.com/remarkjs/remark/1f338e72/logo.svg?sanitize=true
[monorepo]: https://github.com/remarkjs/remark
[commonmark]: https://commonmark.org
[remark-gfm]: https://github.com/remarkjs/remark-gfm
[remark-mdx]: https://github.com/mdx-js/mdx/tree/main/packages/remark-mdx
[cheat]: https://commonmark.org/help/
[micromark]: https://github.com/micromark/micromark
[mdast]: https://github.com/syntax-tree/mdast
[awesome]: https://github.com/remarkjs/awesome
[plugins]: https://github.com/remarkjs/remark/blob/main/doc/plugins.md#list-of-plugins
[topic]: https://github.com/topics/remark-plugin
[site]: https://unifiedjs.com
[twitter]: https://twitter.com/unifiedjs
[collective]: https://github.com/unifiedjs/collective
[coc]: https://github.com/remarkjs/.github/blob/main/code-of-conduct.md
[support]: https://github.com/remarkjs/.github/blob/main/support.md
[contribute]: https://github.com/remarkjs/.github/blob/main/contributing.md
[oc]: https://opencollective.com/unified
[ghs]: https://github.com/sponsors/unifiedjs
