---
group: guide
index: 1
title: 介绍 unified
description: 指南总结了unified的是什么和为什么
author: Merlijn Vos
authorTwitter: Murderlon
authorGithub: Murderlon
tags:
  - welcome
  - introduction
published: 2019-12-12
modified: 2020-06-14
---

## 介绍 unified

读完这篇文章，你就会明白了:

- 了解unified是做什么的
- 体验一下生态系统
- 知道如何使用它
- 了解(未来)用例需要哪些部件(处理器)
- 有继续学习或开始学习的资源列表吗

![][unified-overview]

### Contents

- [介绍 unified](#介绍-unified)
  - [Contents](#contents)
  - [介绍](#介绍)
  - [集体](#集体)
  - [它们是如何结合在一起的](#它们是如何结合在一起的)
  - [用例](#用例)
  - [Summary](#summary)
  - [下一个步骤](#下一个步骤)

### 介绍

**unity是一个友好的界面，由一个为创建和操作内容而构建的插件生态系统支持**.
它通过将Markdown、HTML或纯文本散文转化为结构化数据，并将其提供给100多个插件来实现这一目标。
例如，插件可以执行拼写检查、检查或缩小等任务。

使用unified，您不需要手动处理语法或解析。
相反，您通常编写一行代码将插件链接到unified的进程中。

Unified本身是一个相当小的模块，充当unified不同内容格式处理的接口。
围绕某种格式，存在一个生态系统，例如Markdown的remark。
几个生态系统存在unified。
它们与其他工具和规范一起形成unified的集合。

### 集体

这个unified的集体涵盖了志同道合的组织。
这些组织的共同目标是创新内容处理。
无缝、可互换和可插拔的工具是实现这一目标的方式。

根据您想要做的事情，您将引用不同的组织。
让我们从一轮介绍开始。

生态系统:

- remark — Markdown
- rehype — HTML
- retext — 自然语言
- redot — Graphviz

语法树的规范:

- unist — 通用语法树
- mdast — Markdown 抽象语法树格式
- hast — HTML 抽象语法树格式
- xast — XML 抽象语法树格式
- esast — ECMAScript 抽象语法树格式
- nlcst — 自然语言具体语法树格式

其他构建模块:

- syntax-tree — 用于构建插件的低级实用程序
- vfile — 用于文本处理的虚拟文件格式
- MDX — Markdown 和 JSX

我们将在下一节中了解它们是如何组合在一起的。
如果你已经开始冒险了, 你可以直接去 “[使用unified][using-unified]” 或 “[如何开始使用插件][using-plugins]”.

### 它们是如何结合在一起的

这些处理器、规范和工具由三部分组成。
处理机的运行过程:

1.  **Parse**:
    无论您的输入是Markdown、HTML还是散文，都需要将其解析为可操作的格式。
    这种格式称为语法树。
    规范(例如mdast)定义了这种语法树的外观。
    处理器(比如mast的remark)负责创建它们。
2.  **Transform**:
    这就是奇迹发生的地方。
    用户编写插件和他们运行的顺序。
    插件插入到这个阶段，并转换和检查它们得到的格式。
3.  **Stringify**:
    最后一步是采用(调整后的)格式，并将其字符串化为Markdown、HTML或散文(可能与输入格式不同!)

unity可以在Node.js中以编程方式使用。
通过构建步骤，它也可以在浏览器中使用。
处理器的CLI版本、Grunt插件和Gulp插件也存在。

unified的独特之处在于它可以在相同的过程中在格式之间切换，例如Markdown到HTML。
这允许更强大的组合。

以下插件桥接格式:

- [`remark-rehype`][remark-rehype] — Markdown to HTML
- [`rehype-remark`][rehype-remark] — HTML to Markdown
- [`remark-retext`][remark-retext] — Markdown to prose
- [`rehype-retext`][rehype-retext] — HTML to prose

### 用例

**每当你想到处理内容——你就会想到unified**.
它是一个强大的工具，所以对于一些任务，比如将Markdown转换为HTML，你也可以使用更简单的工具，比如[`marked`][marked]。
当你想要超越单一任务时，unified才是真正的亮点。
例如，当您想要执行格式规则、检查拼写、生成目录，以及(潜在的)更多其他功能时，就应该选择unified。

> MDX成功的很大一部分原因是利用了unified和remark的生态系统。
> 我能够在几个小时内制作出一个原型，因为我不必担心Markdown解析:remark免费提供给了我。
> 它提供了用于构建的原语。
>
> — [John Otander][john], [`mdx-js/mdx`][mdx]作者

为了进一步激发人们的想象力，以下是在unified管道中使用的更常见的插件，用于做有趣的事情:

- [`remark-toc`][remark-toc] — 生成一个目录
- [`rehype-prism`][rehype-prism] — 用Prism突出显示HTML中的代码
- [`retext-spell`][retext-spell] — 检查拼写
- [`remark-lint`][remark-lint] — 检查Markdown代码样式
- [`retext-equality`][retext-equality] — 检查可能不敏感的语言
- [`remark-math`][remark-math] — 支持数学Markdown / HTML
- [`retext-repeated-words`][retext-repeated-words]
  — 检查 `for for` 重复的单词
- [`rehype-minify`][rehype-minify] — 使变小 HTML
- …探索所有 [remark][all-remark-plugins], [rehype][all-rehype-plugins] 或 [retext][all-retext-plugins] plugins

### Summary

- unity是一个友好的界面，由一个为创建和操作内容而构建的插件生态系统支持。
  您不必担心解析，因为您有可以构建的原语
- 有数百个插件可用
- remark用于Markdown, rehype用于HTML, retext用于自然语言
- unity的插件管道通常允许您编写一行代码来将一个特性链接到流程中，例如桥接格式(例如Markdown到HTML)。

### 下一个步骤

- [使用unified的][using-unified]
- [开始使用插件][using-plugins]
- [语法树介绍][intro-to-syntax-trees]

<!--Definitions-->

[mdx]: https://github.com/mdx-js/mdx
[john]: https://github.com/johno/
[remark-rehype]: https://github.com/remarkjs/remark-rehype/
[rehype-remark]: https://github.com/rehypejs/rehype-remark
[remark-retext]: https://github.com/remarkjs/remark-retext/
[rehype-retext]: https://github.com/rehypejs/rehype-retext
[remark-toc]: https://github.com/remarkjs/remark-toc
[rehype-prism]: https://github.com/mapbox/rehype-prism
[retext-spell]: https://github.com/retextjs/retext-spell
[remark-lint]: https://github.com/remarkjs/remark-lint
[retext-equality]: https://github.com/retextjs/retext-equality
[remark-math]: https://github.com/remarkjs/remark-math
[retext-repeated-words]: https://github.com/retextjs/retext-repeated-words
[rehype-minify]: https://github.com/rehypejs/rehype-minify
[all-remark-plugins]: https://github.com/topics/remark-plugin
[all-rehype-plugins]: https://github.com/topics/rehype-plugin
[all-retext-plugins]: https://github.com/topics/retext-plugin
[marked]: https://github.com/markedjs/marked
[unified-overview]: /image/unified-overview.png
[using-unified]: /learn/guide/using-unified/
[using-plugins]: /learn/guide/using-plugins/
[intro-to-syntax-trees]: /learn/guide/introduction-to-syntax-trees/
