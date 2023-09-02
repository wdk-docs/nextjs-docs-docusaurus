---
group: guide
title: 创建编辑器
description: Guide that shows how to create a fancy app ✨
author: Titus Wormer
authorTwitter: wooorm
authorGithub: wooorm
tags:
  - editor
  - dingus
published: 2017-05-03
modified: 2020-06-14
---

## 创建编辑器

本指南展示了如何使用unity创建一个交互式在线编辑器。
在其中，我们将通过“语法高亮”来可视化文本的语法属性。
编辑器将在浏览器中运行。
它会很快，因为我们使用[`virtual-dom`][vdom](但你也可以使用[React][]之类的)。

对于这个例子，我们将创建一个可视化句子长度的应用程序。
它是基于[Gary Provost][gary]的提示，而可视化是基于[@gregoryciotti][tweet]。

你也可以在网上[查看这个项目][write-music]。

> 卡住了吗?
> 有其他指南的想法吗?
> 看[`support.md`][support].

### Contents

- [创建编辑器](#创建编辑器)
  - [Contents](#contents)
  - [案例](#案例)
  - [项目结构](#项目结构)
  - [设置JavaScript](#设置javascript)
  - [自然语言语法树](#自然语言语法树)
  - [虚拟DOM](#虚拟dom)
  - [突出](#突出)
  - [颜色](#颜色)
  - [挤压缺陷](#挤压缺陷)
  - [进一步练习](#进一步练习)

### 案例

在我们开始之前，让我们先概述一下我们想做什么。
我们希望根据文本中有多少单词来突出显示句子。
用户应该能够改变文本，它应该高亮显示。

我们将使用[xo][]作为筛选器，[browserify][]作为打包器，用`require`调用在浏览器中工作的JavaScript来编译我们的JavaScript(你可以把它们换成你喜欢的筛选器和打包器)。

### 项目结构

让我们先勾勒一下项目结构:

```txt
demo/
├─ index.js
├─ build.js
├─ index.html
├─ index.css
└─ package.json
```

…其中`demo/`是我们的目录，`build.js`是编译`index.js`生成的JavaScript。

保持`index.js`, `index.html`, 和 `index.css`暂时为空，并填充`package.json`包含以下内容。

```json
{
  "name": "demo",
  "private": true,
  "type": "module",
  "devDependencies": {
    "esbuild": "^0.13.0",
    "prettier": "^2.0.0",
    "xo": "^0.44.0"
  },
  "scripts": {
    "build": "esbuild index.js --bundle --minify --target=es2020 --format=esm --outfile=build.js",
    "lint": "prettier . -w && xo",
    "test": "npm run build && npm run lint"
  },
  "prettier": {
    "tabWidth": 2,
    "useTabs": false,
    "singleQuote": true,
    "bracketSpacing": false,
    "semi": false,
    "trailingComma": "none"
  },
  "xo": {
    "envs": ["browser"],
    "prettier": true,
    "ignore": ["build.js"]
  }
}
```

> [`private: true`][private] 意味着你不会意外地将包发布到npm。

现在，运行`npm install` 和 `npm test`之后，你会看到 `build.js` 也出现了。
上面的包将[xo][]设置为筛选器，将[browserify][]设置为打包器。

现在，用以下内容填充`index.html`:

```html
<!doctype html>
<meta charset="utf8" />
<title>demo</title>
<link rel="stylesheet" href="index.css" />
<div id="root"></div>
<script type="module" src="build.js"></script>
```

它链接了`index.css` 和 `build.js`，并添加了一个元素(`#root`)，我们将在后面添加编辑器。
哦，你知道`<html>`， `<head>`和`<body>`是可选的吗?
对于本例，我们将尽量减少HTML，但如果您喜欢，可以随意添加。

还要加上`.pretierignore`文件不格式化我们的构建:

```ignore
build.js
```

### 设置JavaScript

好吧!
现在，让我们设置JavaScript。
首先在 `index.js` 中添加以下代码:

```js
import h fromvirtual-dom/h.js'
import createElement fromvirtual-dom/create-element.js'
import diff fromvirtual-dom/diff.js'
import patch fromvirtual-dom/patch.js'

const root = document.querySelector('#root')
let tree = render('The initial text.')
let dom = root.append(createElement(tree))

function onchange(ev) {
  const next = render(ev.target.value)
  dom = patch(dom, diff(tree, next))
  tree = next
}

function render(text) {
  const node = parse(text)

  return h('div', { className:editor}, [
    h('div', { key:draw', className:draw}, highlight(node)),
    h('textarea', {
      key:area',
      value: text,
      oninput: onchange,
    }),
  ])

  function parse() {}

  function highlight() {}
}
```

> 别忘了 `npm install virtual-dom`.

我可以想象，如果你以前从未见过[`virtual-dom`][vdom]的使用，那么这有点太快了。
如果是这样的话，请在空闲的时候花点时间阅读一下`virtual-dom`文档。
本指南将等待!

总结一下代码中所有这些东西的含义:

- `h` creates “virtual” nodes
- `createElement` turns them into DOM nodes
- `diff` finds the difference between two virtual nodes
- `patch` applies `diff` to a DOM node
- `root` is our anchor into the document
- `tree` is the current virtual tree
- `dom` is the current DOM tree
- `onchange` handles any state change (the text in our case)
- `render` creates a new virtual tree based on that state
- `parse` transforms the state into a natural language syntax tree
- `highlight` transforms that syntax tree into a virtual tree

In `render`, we’re creating two elements: a `<div>` that we’ll draw our syntax
highlighting in, and a `<textarea>` that the user can edit.
Both are wrapped in a parent `<div>`.
We’ll style the text area and the drawing area exactly the same, and position
the text above the drawing area, with the following styles.

```css
html {
  font-size: 16px;
  line-height: 1.5;
}

.editor {
  position: relative;
  max-width: 37em;
  margin: auto;
  overflow: hidden;
}

textarea,
.draw {
  margin: 0;
  padding: 0;
  width: 100%;
  border: none;
  outline: none;
  resize: none;
  overflow: hidden;
  /* Can’t use a nice font: kerning renders differently in textareas. */
  font-family: monospace;
  line-height: inherit;
  font-size: inherit;
  background: transparent;
  white-space: pre-wrap;
  word-wrap: break-word;
  font-size: inherit;
  line-height: inherit;
}

textarea {
  color: inherit;
  position: absolute;
  top: 0;
}

.draw {
  min-height: 100vh;
}
```

这是相当多的代码:主要是为了在我们的文本和绘图区域上执行相同的样式。

### 自然语言语法树

现在，让我们设置自然语言语法树解析。
We’ll use unified (d’oh), and [`retext-english`][english] to parse English
natural language.

Change `index.js` like so:

```diff
--- a/index.js
+++ b/index.js
@@ -2,7 +2,10 @@ import h fromvirtual-dom/h.js'
 import createElement fromvirtual-dom/create-element.js'
 import diff fromvirtual-dom/diff.js'
 import patch fromvirtual-dom/patch.js'
+import {unified} fromunified'
+import retextEnglish fromretext-english'

+const processor = unified().use(retextEnglish)
 const root = document.querySelector('#root')
 let tree = render('The initial text.')
 let dom = root.append(createElement(tree))
@@ -25,7 +28,9 @@ function render(text) {
     })
   ])

-  function parse() {}
+  function parse(value) {
+    return processor.runSync(processor.parse(value))
+  }

   function highlight() {}
 }
```

> Don’t forget to `npm install unified retext-english`.

Sweet, now we have access to a lot of info on the text.
It still doesn’t do anything yet though.
Let’s add some usefulness.

### 虚拟DOM

Our next task is to go from a natural language syntax tree to a virtual DOM.
We already have `highlight` for that, but it’s empty, so let’s add code to fill
it:

```diff
--- a/index.js
+++ b/index.js
@@ -32,5 +32,19 @@ function render(text) {
     return processor.runSync(processor.parse(value))
   }

-  function highlight() {}
+  function highlight(node) {
+    const results = []
+    let index = -1
+
+    while (++index < node.children.length) {
+      results.push(...one(node.children[index]))
+    }
+
+    return results
+  }
+
+  function one(node) {
+    const result =valuein node ? [node.value] : highlight(node)
+    return result
+  }
 }
```

`highlight` searches all children in the given `node`, and `one` returns either
the “text content” of a node, or the result of searching its children for text
content.

If you’d now run `npm test` again, and open `index.html` in your browser, you’ll
see that the drawing area already has our text (it’s hidden with styles, but you
should be able to see it in your web inspector).

We need one more thing before we can start highlighting: we need to detect
sentences, and apply styles to them.
Change `index.js` like so:

```diff
--- a/index.js
+++ b/index.js
@@ -18,6 +18,7 @@ function onchange(ev) {

 function render(text) {
   const node = parse(text)
+  let key = 0

   return h('div', {className:editor'}, [
     h('div', {key:draw', className:draw'}, highlight(node)),
@@ -45,6 +46,22 @@ function render(text) {

   function one(node) {
     const result =valuein node ? [node.value] : highlight(node)
+
+    if (node.type ===SentenceNode') {
+      key++
+      return [
+        h(
+         span',
+          {key:s-+ key, style: {backgroundColor: color(count(node))}},
+          result
+        )
+      ]
+    }
+
     return result
   }
+
+  function count() {}
+
+  function color() {}
 }
```

> `key` is needed for `virtual-dom` [to be performant][vdom-key].

We don’t color sentences yet, but there’s `<span>` elements wrapping them now.
You can see that in action by running `npm test` again and using your web
inspector to inspect the drawing area.

We’ve also set up two functions to highlight sentences.
`count` will count the number of words of a given sentence, and `color` will
pick a corresponding color.

### 突出

Now, let’s add colors.
Update `index.js` like so:

```diff
--- a/index.js
+++ b/index.js
@@ -4,6 +4,9 @@ import diff fromvirtual-dom/diff.js'
 import patch fromvirtual-dom/patch.js'
 import {unified} fromunified'
 import retextEnglish fromretext-english'
+import {visit} fromunist-util-visit'
+
+const hues = [0]

 const processor = unified().use(retextEnglish)
 const root = document.querySelector('#root')
@@ -61,7 +64,18 @@ function render(text) {
     return result
   }

-  function count() {}
+  function count(node) {
+    let value = 0
+
+    visit(node,WordNode', () => {
+      value++
+    })
+
+    return value
+  }

-  function color() {}
+  function color(count) {
+    const value = count < hues.length ? hues[count] : hues[hues.length - 1]
+    returnhsl(+ [value,93%',85%'].join(',) +)'
+  }
 }
```

> Don’t forget to `npm install unist-util-visit`.

The `count` function searches `node` for all occurrences of words, through
[`unist-util-visit`][visit], and returns that count.

`color` takes a number, and returns a nice color in HSL for it.
It does so based on if there’s a corresponding hue for it in `hues` (now only
one value).
If there’s no corresponding hue, it uses the last specified hue.

It’s not much, but it’s something.
Try it out by running `npm test` again, and viewing `index.html` in your
browser.
If everything went okay, you should see each sentence highlighted in red.

### 颜色

One color isn’t that cool, and we’re trying to recreate that [visual by
@gregoryciotti][tweet].
We need some more colors.
From that image, I deducted the following hues.
But you could use any hues you like!

To match that image, change `hues` like so:

```diff
--- a/index.js
+++ b/index.js
@@ -6,7 +6,7 @@ import {unified} fromunified'
 import retextEnglish fromretext-english'
 import {visit} fromunist-util-visit'

-const hues = [0]
+const hues = [60, 60, 60, 300, 300, 0, 0, 120, 120, 120, 120, 120, 120, 180]

 const processor = unified().use(retextEnglish)
 const root = document.querySelector('#root')
```

### 挤压缺陷

💃 After running `npm test` again, and reopening `index.html` in your browser,
you should now see `The initial text` in purple!
If you add more sentences, they each should receive colors based on how many
words they have.

If you add more text, you’ll notice that our drawing area grows nicely, but our
text area does not.
That’s because this example positions the `<textarea>` absolutely on top of the
drawing area.
The easiest way to get both areas the same height, is with the following
slightly hacky code:

```diff
--- a/index.js
+++ b/index.js
@@ -13,10 +13,13 @@ const root = document.querySelector('#root')
 let tree = render('The initial text.')
 let dom = root.append(createElement(tree))

+setTimeout(resize, 4)
+
 function onchange(ev) {
   const next = render(ev.target.value)
   dom = patch(dom, diff(tree, next))
   tree = next
+  setTimeout(resize, 4)
 }

 function render(text) {
@@ -79,3 +82,11 @@ function render(text) {
     returnhsl(+ [value,93%',85%'].join(',) +)'
   }
 }
+
+function resize() {
+  dom.lastChild.rows =
+    Math.ceil(
+      dom.firstChild.getBoundingClientRect().height /
+        Number.parseInt(window.getComputedStyle(dom.firstChild).lineHeight, 10)
+    ) + 1
+}
```

This updates the `rows` attribute on the text area to correspondent with the
size of the drawing area.

### 进一步练习

The above code has a few issues:

- [ ] `onchange` is not [debounced][debounce], which leads to performance
      issues
- [ ] `input` events are not supported in some older browsers
- [ ] The styles aren’t perfect
- [ ] and probably some other things!

…maybe you could solve some?
Other than those issues, it’s a pretty cool little demo.

If you haven’t already, check out the other articles in the
[learn section][learn]!

<!--Definitions-->

[support]: https://github.com/unifiedjs/.github/blob/main/support.md
[vdom]: https://github.com/Matt-Esch/virtual-dom
[react]: https://reactjs.org
[gary]: http://www.garyprovost.com
[tweet]: https://twitter.com/gregoryciotti/status/639837682844090369
[write-music]: https://wooorm.com/write-music/
[xo]: https://github.com/sindresorhus/xo
[browserify]: https://github.com/browserify/browserify
[english]: https://github.com/retextjs/retext/tree/HEAD/packages/retext-english
[vdom-key]: https://github.com/Matt-Esch/virtual-dom/tree/HEAD/virtual-hyperscript#key
[visit]: https://github.com/syntax-tree/unist-util-visit
[private]: https://docs.npmjs.com/files/package.json#private
[debounce]: https://www.npmjs.com/package/debounce
[learn]: /learn/
