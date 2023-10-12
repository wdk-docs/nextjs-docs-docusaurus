# rehype-toc

Rehype的目录插件

一个[rehype](https://github.com/rehypejs/rehype)插件，为页面添加一个目录(TOC)

## Features

- 添加`<nav>`和`<ol>`列表，列出页面上的所有标题
- 结合[rehype-slug](https://github.com/rehypejs/rehype-slug)创建每个标题的链接
- 忽略`<main>`以外的标题(如果它存在)
- 您可以自定义包含哪些标题 (默认为 `<h1>` - `<h6>`)
- 您可以自定义每个TOC元素上的CSS类
- Hooks为生成的HTML提供了完全的自定义

## 例子

**input.html**

这是原始的HTML文件。有三个级别的标题(`<h1>` - `<h3>`)，它们都没有id。

```html
<html>
  <body>
    <h1>Apple Pie Recipe</h1>
    <p>This is the world's best apple pie recipe...</p>

    <div>
      <h2>Filling</h2>
      <p>The filling is the best part...</p>

      <h3>Preparing the apples</h3>
      <p>Cut the apples into 1/4 inch slices...</p>

      <h3>Preparing the spice mix</h3>
      <p>In a mixing bowl, combine sugar, cinnamon...</p>
    </div>

    <div>
      <h2>Crust</h2>
      <p>How to make the perfect flaky crust...</p>

      <h3>Preparing the dough</h3>
      <p>Combine flour, sugar, salt...</p>

      <h3>The criss-cross top</h3>
      <p>Cut the top crust into 1/2 inch strips...</p>
    </div>
  </body>
</html>
```

**example.js**

该脚本读取上面的`input.html`文件，并将结果写入`output.html`(如下所示)。
脚本使用[unified](https://unifiedjs.com/)、[rehype-parse](https://github.com/rehypejs/rehype/tree/master/packages/rehype-parse)、[rehype-slug](https://github.com/rehypejs/rehype-slug)和[rehype-stringify](https://github.com/rehypejs/rehype/tree/master/packages/rehype-stringify)。

```javascript
const unified = require('unified')
const parse = require('rehype-parse')
const slug = require('rehype-slug')
const toc = require('@jsdevtools/rehype-toc')
const stringify = require('rehype-stringify')
const fs = require('fs')

async function example() {
  // Create a Rehype processor with the TOC plugin
  const processor = unified().use(parse).use(slug).use(toc).use(stringify)

  // Read the original HTML file
  let inputHTML = await fs.promises.readFile('input.html')

  // Process the HTML, adding heading IDs and Table of Contents
  let outputHTML = await processor.process(inputHTML)

  // Save the new HTML
  await fs.promises.writeFile('output.html', outputHTML)
}
```

**output.html**

下面是由上述脚本创建的HTML。
注意，在`<body>`的顶部添加了一个目录，其中包含指向页面上每个标题的链接。
多亏了[rehype-slug](https://github.com/rehypejs/rehype-slug)，标题现在也有了id。

```html
<html>
  <body>
    <nav class="toc">
      <ol class="toc-level toc-level-1">
        <li class="toc-item toc-item-h1">
          <a class="toc-link toc-link-h1" href="#apple-pie-recipe">
            Apple Pie Recipe
          </a>

          <ol class="toc-level toc-level-2">
            <li class="toc-item toc-item-h2">
              <a class="toc-link toc-link-h2" href="#filling"> Filling </a>

              <ol class="toc-level toc-level-3">
                <li class="toc-item toc-item-h3">
                  <a class="toc-link toc-link-h3" href="#preparing-the-apples">
                    Preparing the apples
                  </a>
                </li>
                <li class="toc-item toc-item-h3">
                  <a
                    class="toc-link toc-link-h3"
                    href="#preparing-the-spice-mix"
                  >
                    Preparing the spice mix
                  </a>
                </li>
              </ol>
            </li>

            <li class="toc-item toc-item-h2">
              <a class="toc-link toc-link-h2" href="#crust"> Crust </a>

              <ol class="toc-level toc-level-3">
                <li class="toc-item toc-item-h3">
                  <a class="toc-link toc-link-h3" href="#preparing-the-dough">
                    Preparing the dough
                  </a>
                </li>
                <li class="toc-item toc-item-h3">
                  <a class="toc-link toc-link-h3" href="#the-criss-cross-top">
                    The criss-cross top
                  </a>
                </li>
              </ol>
            </li>
          </ol>
        </li>
      </ol>
    </nav>

    <h1 id="apple-pie-recipe">Apple Pie Recipe</h1>
    <p>This is the world's best apple pie recipe...</p>

    <div>
      <h2 id="filling">Filling</h2>
      <p>The filling is the best part...</p>

      <h3 id="preparing-the-apples">Preparing the apples</h3>
      <p>Cut the apples into 1/4 inch slices...</p>

      <h3 id="preparing-the-spice-mix">Preparing the spice mix</h3>
      <p>In a mixing bowl, combine sugar, cinnamon...</p>
    </div>

    <div>
      <h2 id="crust">Crust</h2>
      <p>How to make the perfect flaky crust...</p>

      <h3 id="preparing-the-dough">Preparing the dough</h3>
      <p>Combine flour, sugar, salt...</p>

      <h3 id="the-criss-cross-top">The criss-cross top</h3>
      <p>Cut the top crust into 1/2 inch strips...</p>
    </div>
  </body>
</html>
```

## 安装

你可以通过[npm](https://docs.npmjs.com/about-npm/)安装Rehype TOC。

```bash
npm install @jsdevtools/rehype-toc
```

您可能还需要安装[unified](https://unifiedjs.com/)、[rehype-parse](https://github.com/rehypejs/rehype/tree/master/packages/rehype-parse)、[rehype-stringify](https://github.com/rehypejs/rehype/tree/master/packages/rehype-stringify)和[rehype-slug](https://github.com/rehypejs/rehype-slug)。

```bash
npm install unified rehype-parse rehype-stringify rehype-slug
```

## 使用

使用Rehype TOC插件需要了解如何使用Unified和Rehype。
[这是一个优秀的指南](https://unifiedjs.com/learn/guide/introduction-to-unified/)学习基础知识。

Rehype TOC插件就像任何其他Rehype插件一样工作。将其传递给[`.use()`方法](https://github.com/unifiedjs/unified#processoruseplugin-options)，可选地使用[options 对象](#options)。

```javascript
const unified = require('unified')
const toc = require('@jsdevtools/rehype-toc')

// Use the Rehype TOC plugin with its default options
unified().use(toc)

// Use the Rehype TOC plugin with custom options
unified().use(toc, {
  headings: ['h1', 'h2'], // Only include <h1> and <h2> headings in the TOC
  cssClasses: {
    toc: 'page-outline', // Change the CSS class for the TOC
    link: 'page-link', // Change the CSS class for links in the TOC
  },
})
```

## 选项

Rehype TOC插件支持以下选项:

| 选项                  | 类型                   | 默认值                 | 描述                                                                                                                                                                                                                                                |
| :-------------------- | :--------------------- | :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nav`                 | boolean                | true                   | 确定是否将目录包装在`<nav>`元素中。                                                                                                                                                                                                                 |
| `position`            | string                 | "afterbegin"           | 相对于`<main>`或`<body>`元素，应该插入目录的位置。可以为`beforebegin`、`afterbegin`、`beforeend`或`afterend`。参见[`insertAdjacentElement()`文档](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement)获取每个值的解释。 |
| `headings`            | array of strings       | h1, h2, h3, h4, h5, h6 | 要包含在目录中的HTML标题标签                                                                                                                                                                                                                        |
| `cssClasses.toc`      | string                 | toc                    | 包含整个目录的顶层`<nav>`或`<ol>`元素的CSS类名。                                                                                                                                                                                                    |
| `cssClasses.list`     | string                 | toc-level              | 目录表中所有`<ol>`元素的CSS类名，包括顶层元素。                                                                                                                                                                                                     |
| `cssClasses.listItem` | string                 | toc-item               | 目录表中所有`<li>`元素的CSS类名。                                                                                                                                                                                                                   |
| `cssClasses.link`     | string                 | toc-link               | 目录表中所有`<a>`元素的CSS类名。                                                                                                                                                                                                                    |
| `customizeTOC`        | function(toc)          |                        | 允许您在将目录添加到页面之前自定义目录。<br /><br />该函数接收TOC节点树，并可以以您想要的任何方式修改它。或者您可以返回一个新的节点树以供使用。或者返回`false`以阻止TOC被添加到页面中。                                                             |
| `customizeTOCItem`    | function(toc, heading) |                        | 允许您在将目录中的每个项目添加到页面之前自定义它。<br /><br />函数接收TOC项的节点树及其引用的标题节点。您可以以任何需要的方式修改节点。或者您可以返回一个新的节点树以供使用。或者返回`false`以阻止TOC被添加到页面中。                               |

## 贡献

欢迎贡献、增强和bug修复!
[打开一个问题](https://github.com/JS-DevTools/rehype-toc/issues)在GitHub和[提交拉请求](https://github.com/JS-DevTools/rehype-toc/pulls)。

#### 构建

要在计算机上本地构建项目:

1. **克隆这个仓库**<br />
   `git clone https://github.com/JS-DevTools/rehype-toc.git`

2. **安装依赖关系**<br />
   `npm install`

3. **构建代码**<br />
   `npm run build`

4. **运行测试**<br />
   `npm test`
