# `@mdx-js/mdx`

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

MDX 编译器.

<!-- more -->

## Contents

- [What is this?](#what-is-this)
- [When should I use this?](#when-should-i-use-this)
- [Install](#install)
- [Use](#use)
- [API](#api)
  - [`compile(file, options?)`](#compilefile-options)
  - [`compileSync(file, options?)`](#compilesyncfile-options)
  - [`evaluate(file, options)`](#evaluatefile-options)
  - [`evaluateSync(file, options)`](#evaluatesyncfile-options)
  - [`run(functionBody, options)`](#runfunctionbody-options)
  - [`runSync(functionBody, options)`](#runsyncfunctionbody-options)
  - [`createProcessor(options)`](#createprocessoroptions)
- [Types](#types)
- [Architecture](#architecture)
- [Security](#security)
- [Contribute](#contribute)
- [License](#license)

## 这是什么?

这个包是一个编译器，可以将MDX转换为JavaScript。
它还可以计算MDX代码。

## 我应该什么时候使用这个?

这是将MDX转换为JavaScript的核心编译器，它为您提供了最大的控制权。
如果你使用的是一个捆绑器(webpack, Rollup, esbuild)，
或者一个站点构建器(Gatsby, Next.js)，
或者一个带有捆绑器的构建系统(Vite, WMR)，
你最好使用一个集成:参见[§集成][integrations]。

## 安装

这个包是[仅限ESM][ESM]: 使用它需要Node 12+，它必须被`import`而不是`require`。

[npm][]:

```sh
npm install @mdx-js/mdx
```

[yarn][]:

```sh
yarn add @mdx-js/mdx
```

## 使用

假设我们有一个MDX文档， `example.mdx`:

```mdx
export const Thing = () => <>World!</>

# Hello, <Thing />
```

在 `example.js`中添加一些代码，将`example.mdx`编译为JavaScript:

```js
import fs from 'node:fs/promises'
import { compile } from '@mdx-js/mdx'

const compiled = await compile(await fs.readFile('example.mdx'))

console.log(String(compiled))
```

大致收益率:

```js
/* @jsxRuntime automatic @jsxImportSource react */
import {
  Fragment as _Fragment,
  jsx as _jsx,
  jsxs as _jsxs,
} from 'react/jsx-runtime'

export const Thing = () => _jsx(_Fragment, { children: 'World' })

function _createMdxContent(props) {
  const _components = {
    h1: 'h1',
    ...props.components,
  }
  return _jsxs(_components.h1, {
    children: ['Hello ', _jsx(Thing, {})],
  })
}

export default function MDXContent(props = {}) {
  const { wrapper: MDXLayout } = props.components || {}
  return MDXLayout
    ? _jsx(
        MDXLayout,
        Object.assign({}, props, { children: _jsx(_createMdxContent, props) }),
      )
    : _createMdxContent(props)
}
```

有关MDX如何工作以及如何使用结果的更多信息，请参见[§使用MDX][using-mdx]。

## API

这个包导出以下标识符:
[`compile`][compile],
[`compileSync`][compile-sync],
[`evaluate`][eval],
[`evaluateSync`](#evaluatesyncfile-options),
[`run`][run],
[`runSync`](#runsyncfunctionbody-options), and
[`createProcessor`][create-processor].
没有默认的导出。

### `compile(file, options?)`

将MDX编译为JS。

###### `file`

MDX文档解析(`string`， 在UTF-8中[`Buffer`][buffer]， [`vfile`][vfile]，或任何可以给`vfile`)。

<details>
<summary>展开例子</summary>

```js
import { VFile } from 'vfile'
import { compile } from '@mdx-js/mdx'

await compile(':)')
await compile(Buffer.from(':-)'))
await compile({ path: 'path/to/file.mdx', value: '🥳' })
await compile(new VFile({ path: 'path/to/file.mdx', value: '🤭' }))
```

</details>

###### `options.remarkPlugins`

[remark插件][remark-plugins]，预设和配对列表。

<details>
<summary>展开例子</summary>

```js
import remarkFrontmatter from 'remark-frontmatter' // YAML and such.
import remarkGfm from 'remark-gfm' // Tables, footnotes, strikethrough, task lists, literal URLs.

await compile(file, { remarkPlugins: [remarkGfm] }) // One plugin.
await compile(file, { remarkPlugins: [[remarkFrontmatter, 'toml']] }) // A plugin with options.
await compile(file, { remarkPlugins: [remarkGfm, remarkFrontmatter] }) // Two plugins.
await compile(file, {
  remarkPlugins: [[remarkGfm, { singleTilde: false }], remarkFrontmatter],
}) // Two plugins, first w/ options.
```

</details>

###### `options.rehypePlugins`

List of [rehype plugins][rehype-plugins], presets, and pairs.

<details>
<summary>展开例子</summary>

```js
import rehypeKatex from 'rehype-katex' // Render math with KaTeX.
import remarkMath from 'remark-math' // Support math like `$so$`.

await compile(file, {
  remarkPlugins: [remarkMath],
  rehypePlugins: [rehypeKatex],
})

await compile(file, {
  remarkPlugins: [remarkMath],
  // A plugin with options:
  rehypePlugins: [[rehypeKatex, { throwOnError: true, strict: true }]],
})
```

</details>

###### `options.recmaPlugins`

List of recma plugins.
This is a new ecosystem, currently in beta, to transform [esast][] trees
(JavaScript).

###### `options.remarkRehypeOptions`

传递给[`remark-rehype`][remark-rehype]的选项。
The option `allowDangerousHtml` will always be set to `true` and the MDX nodes
are passed through.
In particular, you might want to pass `clobberPrefix`, `footnoteLabel`, and
`footnoteBackLabel`.

<details>
<summary>展开例子</summary>

```js
compile({ value: '…' }, { remarkRehypeOptions: { clobberPrefix: 'comment-1' } })
```

</details>

###### `options.mdExtensions`

List of markdown extensions, with dot (`Array<string>`, default: `['.md',
'.markdown', '.mdown', '.mkdn', '.mkd', '.mdwn', '.mkdown', '.ron']`).

###### `options.mdxExtensions`

List of MDX extensions, with dot (`Array<string>`, default: `['.mdx']`).
Has no effect in `compile` or `evaluate` but does affect
[§ Integrations][integrations].

###### `options.format`

Format the file is in (`'detect' | 'mdx' | 'md'`, default: `'detect'`).

- `'detect'` — use `'markdown'` for files with an extension in `mdExtensions`
  and `'mdx'` otherwise
- `'mdx'` — treat file as [MDX][mdx-syntax]
- `'md'` — treat file as plain vanilla markdown

The format cannot be detected if a file is passed without a path or extension:
`mdx` will be assumed.
So pass a full vfile (with `path`) or an object with a path.

<details>
<summary>展开例子</summary>

```js
compile({ value: '…' }) // Seen as MDX
compile({ value: '…' }, { format: 'md' }) // Seen as markdown
compile({ value: '…', path: 'readme.md' }) // Seen as markdown

// Please do not use `.md` for MDX as other tools won’t know how to handle it.
compile({ value: '…', path: 'readme.md' }, { format: 'mdx' }) // Seen as MDX
compile({ value: '…', path: 'readme.md' }, { mdExtensions: [] }) // Seen as MDX
```

</details>

This option mostly affects [§ Integrations][integrations]
because in those it affects _which_ files are “registered”:

- `format: 'mdx'` registers the extensions in `options.mdxExtensions`
- `format: 'md'` registers the extensions in `options.mdExtensions`
- `format: 'detect'` registers both lists of extensions

###### `options.outputFormat`

Output format to generate (`'function-body' | 'program'`, default: `'program'`).
In most cases `'program'` should be used, as it results in a whole program.
Internally, [`evaluate`][eval] uses `outputFormat: 'function-body'` to compile
to code that can be `eval`ed with [`run`][run].
In some cases, you might want to do what `evaluate` does in separate steps
yourself, such as when compiling on the server and running on the client.

The `'program'` format will use import statements to import the runtime (and
optionally provider) and use an export statement to yield the `MDXContent`
component.

The `'function-body'` format will get the runtime (and optionally provider) from
`arguments[0]`, rewrite export statements, and use a return statement to yield
what was exported.
Normally, this output format will throw on `import` (and `export … from`)
statements, but you can support them by setting
[`options.useDynamicImport`][usedynamicimport].

<details>
<summary>展开例子</summary>

A module `example.js`:

```js
import { compile } from '@mdx-js/mdx'

const code = 'export const no = 3.14\n\n# hi {no}'

console.log(String(await compile(code, { outputFormat: 'program' }))) // Default
console.log(String(await compile(code, { outputFormat: 'function-body' })))
```

…yields:

```js
/* @jsxRuntime automatic @jsxImportSource react */
import { jsx as _jsx, jsxs as _jsxs } from 'react/jsx-runtime'
export const no = 3.14
function _createMdxContent(props) {
  /* … */
}
export default function MDXContent(props = {}) {
  /* … */
}
```

```js
const { Fragment: _Fragment, jsx: _jsx } = arguments[0]
const no = 3.14
function _createMdxContent(props) {
  /* … */
}
function MDXContent(props = {}) {
  /* … */
}
return { no, default: MDXContent }
```

</details>

###### `options.useDynamicImport`

Whether to compile to dynamic import expressions (`boolean`, default: `false`).
This option applies when [`options.outputFormat`][outputformat] is
`'function-body'`.

`@mdx-js/mdx` can turn import statements (`import x from 'y'`) into dynamic
imports (`const {x} = await import('y')`).
This is useful because import statements only work at the top level of
JavaScript modules, whereas `import()` is available inside function bodies.

When you turn `useDynamicImport` on, you should probably set [`options.baseUrl`][baseurl] too.

<details>
<summary>展开例子</summary>

Say we have a couple modules:

```js
// meta.js:
export const title = 'World'

// numbers.js:
export const no = 3.14

// example.js:
import { compileSync } from '@mdx-js/mdx'

const code = `import {name} from './meta.js'
export {no} from './numbers.js'

# hi {name}!`

console.log(
  String(
    compileSync(code, {
      outputFormat: 'function-body',
      useDynamicImport: true,
    }),
  ),
)
```

…now running `node example.js` yields:

```js
const { Fragment: _Fragment, jsx: _jsx, jsxs: _jsxs } = arguments[0]
const { name } = await import('./meta.js')
const { no } = await import('./numbers.js')
function _createMdxContent(props) {
  /* … */
}
function MDXContent(props = {}) {
  /* … */
}
return { no, default: MDXContent }
```

</details>

###### `options.baseUrl`

Resolve `import`s (and `export … from`, and `import.meta.url`) from this URL
(`string?`, example: `import.meta.url`).

Relative specifiers are non-absolute URLs that start with `/`, `./`, or `../`.
For example: `/index.js`, `./folder/file.js`, or `../main.js`.

This option is useful when code will run in a different place.
One example is when `.mdx` files are in path _a_ but compiled to path _b_ and
imports should run relative the path _b_.
Another example is when evaluating code, whether in Node or a browser.

<details>
<summary>展开例子</summary>

Say we have a module `example.js`:

```js
import { compile } from '@mdx-js/mdx'

const code = 'export {number} from "./data.js"\n\n# hi'
const baseUrl = 'https://a.full/url' // Typically `import.meta.url`

console.log(String(await compile(code, { baseUrl })))
```

…now running `node example.js` yields:

```js
import { Fragment as _Fragment, jsx as _jsx } from 'react/jsx-runtime'
export { number } from 'https://a.full/data.js'
function _createMdxContent(props) {
  /* … */
}
function MDXContent(props = {}) {
  /* … */
}
export default MDXContent
```

</details>

###### `options.development`

Whether to add extra info to error messages in generated code
(`boolean?`, default: `false`).
This also results in the development automatic JSX runtime (`/jsx-dev-runtime`,
`jsxDEV`) being used, which passes positional info to nodes.
The default can be set to `true` in Node.js through environment variables: set
`NODE_ENV=development`.

<details>
<summary>展开例子</summary>

Say we had some MDX that references a component that can be passed or provided
at runtime:

```mdx
**Note**<NoteIcon />: some stuff.
```

And a module to evaluate that:

```js
import fs from 'node:fs/promises'
import * as runtime from 'react/jsx-runtime'
import { evaluate } from '@mdx-js/mdx'

const path = 'example.mdx'
const value = await fs.readFile(path)
const MDXContent = (await evaluate({ path, value }, runtime)).default

console.log(MDXContent())
```

Running that would normally (production) yield:

```txt
Error: Expected component `NoteIcon` to be defined: you likely forgot to import, pass, or provide it.
    at _missingMdxReference (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:27:9)
    at _createMdxContent (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:15:20)
    at MDXContent (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:9:9)
    at main (…/example.js:11:15)
```

But if we change add `development: true` to our example:

```diff
@@ -7,6 +7,6 @@
 import fs from 'node:fs/promises'
-import * as runtime from 'react/jsx-runtime'
+import * as runtime from 'react/jsx-dev-runtime'
 import {evaluate} from '@mdx-js/mdx'

 const path = 'example.mdx'
 const value = await fs.readFile(path)
-const MDXContent = (await evaluate({path, value}, runtime)).default
+const MDXContent = (await evaluate({path, value}, {development: true, ...runtime})).default
 console.log(MDXContent({}))
```

And we’d run it again, we’d get:

```txt
Error: Expected component `NoteIcon` to be defined: you likely forgot to import, pass, or provide it.
It’s referenced in your code at `1:9-1:21` in `example.mdx`
provide it.
    at _missingMdxReference (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:27:9)
    at _createMdxContent (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:15:20)
    at MDXContent (eval at run (…/@mdx-js/mdx/lib/run.js:18:10), <anonymous>:9:9)
    at main (…/example.js:11:15)
```

</details>

###### `options.SourceMapGenerator`

The `SourceMapGenerator` class from [`source-map`][source-map] (optional).
When given, the resulting file will have a `map` field set to a source map (in
object form).

<details>
<summary>展开例子</summary>

Assuming `example.mdx` from [§ Use][use] exists, then:

```js
import fs from 'node:fs/promises'
import { SourceMapGenerator } from 'source-map'
import { compile } from '@mdx-js/mdx'

const file = await compile(
  { path: 'example.mdx', value: await fs.readFile('example.mdx') },
  { SourceMapGenerator },
)

console.log(file.map)
```

…yields:

```js
{
  version: 3,
  sources: ['example.mdx'],
  names: ['Thing'],
  mappings: ';;aAAaA,QAAQ;YAAQ;;;;;;;;iBAE3B',
  file: 'example.mdx'
}
```

</details>

###### `options.providerImportSource`

Place to import a provider from (`string?`, example: `'@mdx-js/react'`).
Useful for runtimes that support context (React, Preact).
The provider must export a `useMDXComponents`, which is called to access an
object of components.

<details>
<summary>展开例子</summary>

If `file` is the contents of `example.mdx` from [§ Use][use], then:

```js
compile(file, { providerImportSource: '@mdx-js/react' })
```

…yields this difference:

```diff
 /* @jsxRuntime automatic @jsxImportSource react */
 import {Fragment as _Fragment, jsx as _jsx, jsxs as _jsxs} from 'react/jsx-runtime'
+import {useMDXComponents as _provideComponents} from '@mdx-js/react'

 export const Thing = () => _jsx(_Fragment, {children: 'World!'})

 function _createMdxContent(props) {
   const _components = {
     h1: 'h1',
+    ..._provideComponents(),
     ...props.components
   }
   return _jsxs(_components.h1, {children: ['Hello ', _jsx(Thing, {})]})
 }

 export default function MDXContent(props = {}) {
-  const {wrapper: MDXLayout} = props.components || {}
+  const {wrapper: MDXLayout} = {
+    ..._provideComponents(),
+    ...props.components
+  }

   return MDXLayout
     ? _jsx(MDXLayout, Object.assign({}, props, {children: _jsx(_createMdxContent, {})}))
     : _createMdxContent()
```

</details>

###### `options.jsx`

Whether to keep JSX (`boolean?`, default: `false`).
The default is to compile JSX away so that the resulting file is immediately
runnable.

<details>
<summary>展开例子</summary>

If `file` is the contents of `example.mdx` from [§ Use][use], then:

```js
compile(file, { jsx: true })
```

…yields this difference:

```diff
 /* @jsxRuntime automatic @jsxImportSource react */
-import {Fragment as _Fragment, jsx as _jsx, jsxs as _jsxs} from 'react/jsx-runtime'

-export const Thing = () => _jsx(_Fragment, {children: 'World!'})
+export const Thing = () => <>World!</>

 function _createMdxContent(props) {
   const _components = {
     h1: 'h1',
     ...props.components
   }
-  return _jsxs(_components.h1, {children: ['Hello ', _jsx(Thing, {})]})
+  return <_components.h1>{"Hello "}<Thing /></_components.h1>
 }

 export default function MDXContent(props = {}) {
   const {wrapper: MDXLayout} = props.components || {}
   return MDXLayout
-    ? _jsx(MDXLayout, Object.assign({}, props, {children: _jsx(_createMdxContent, props)}))
+    ? <MDXLayout {...props}><_createMdxContent {...props} /></MDXLayout>
     : _createMdxContent(props)
 }
 }
```

</details>

###### `options.jsxRuntime`

JSX runtime to use (`'automatic' | 'classic'`, default: `'automatic'`).
The classic runtime compiles to calls such as `h('p')`, the automatic runtime
compiles to `import _jsx from '$importSource/jsx-runtime'\n_jsx('p')`.

<details>
<summary>展开例子</summary>

If `file` is the contents of `example.mdx` from [§ Use][use], then:

```js
compile(file, { jsxRuntime: 'classic' })
```

…yields this difference:

```diff
-/* @jsxRuntime automatic @jsxImportSource react */
-import {Fragment as _Fragment, jsx as _jsx, jsxs as _jsxs} from 'react/jsx-runtime'
+/* @jsxRuntime classic @jsx React.createElement @jsxFrag React.Fragment */
+import React from 'react'

-export const Thing = () => _jsx(_Fragment, {children: 'World!'})
+export const Thing = () => React.createElement(React.Fragment, null, 'World!')
…
```

</details>

###### `options.jsxImportSource`

Place to import automatic JSX runtimes from (`string?`, default: `'react'`).
When in the `automatic` runtime, this is used to define an import for
`Fragment`, `jsx`, `jsxs`, and `jsxDEV`.

<details>
<summary>展开例子</summary>

If `file` is the contents of `example.mdx` from [§ Use][use], then:

```js
compile(file, { jsxImportSource: 'preact' })
```

…yields this difference:

```diff
-/* @jsxRuntime automatic @jsxImportSource react */
-import {Fragment as _Fragment, jsx as _jsx, jsxs as _jsxs} from 'react/jsx-runtime'
+/* @jsxRuntime automatic @jsxImportSource preact */
+import {Fragment as _Fragment, jsx as _jsx, jsxs as _jsxs } from 'preact/jsx-runtime'
```

</details>

###### `options.pragma`

Pragma for JSX (`string?`, default: `'React.createElement'`).
When in the `classic` runtime, this is used as an identifier for function calls:
`<x />` to `React.createElement('x')`.

You should most probably define `pragmaFrag` and `pragmaImportSource` too when
changing this.

<details>
<summary>展开例子</summary>

If `file` is the contents of `example.mdx` from [§ Use][use], then:

```js
compile(file, {
  jsxRuntime: 'classic',
  pragma: 'preact.createElement',
  pragmaFrag: 'preact.Fragment',
  pragmaImportSource: 'preact/compat',
})
```

…yields this difference:

```diff
-/* @jsxRuntime classic @jsx React.createElement @jsxFrag React.Fragment */
-import React from 'react'
+/* @jsxRuntime classic @jsx preact.createElement @jsxFrag preact.Fragment */
+import preact from 'preact/compat'

-export const Thing = () => React.createElement(React.Fragment, null, 'World!')
+export const Thing = () => preact.createElement(preact.Fragment, null, 'World!')
…
```

</details>

###### `options.pragmaFrag`

Pragma for JSX fragments (`string?`, default: `'React.Fragment'`).
When in the `classic` runtime, this is used as an identifier for fragments: `<>`
to `React.createElement(React.Fragment)`.

See `options.pragma` for an example.

###### `options.pragmaImportSource`

Where to import the identifier of `pragma` from (`string?`, default: `'react'`).
When in the `classic` runtime, this is used to import the `pragma` function.
To illustrate with an example: when `pragma` is `'a.b'` and `pragmaImportSource`
is `'c'` this following will be generated: `import a from 'c'`.

See `options.pragma` for an example.

###### `options.elementAttributeNameCase`

Specify casing to use for attribute names (`'html' | 'react`, default:
`'react'`).

This casing is used for hast elements, not for embedded MDX JSX nodes
(components that someone authored manually).

###### `options.stylePropertyNameCase`

Specify casing to use for property names in `style` objects (`'css' | 'dom`,
default: `'dom'`).

This casing is used for hast elements, not for embedded MDX JSX nodes
(components that someone authored manually).

###### Returns

`Promise<VFile>` — Promise that resolves to the compiled JS as a [vfile][].

<details>
<summary>展开例子</summary>

```js
import remarkPresetLintConsistent from 'remark-preset-lint-consistent' // Lint rules to check for consistent markdown.
import { reporter } from 'vfile-reporter'
import { compile } from '@mdx-js/mdx'

const file = await compile('*like this* or _like this_?', {
  remarkPlugins: [remarkPresetLintConsistent],
})

console.error(reporter(file))
```

Yields:

```txt
  1:16-1:27  warning  Emphasis should use `*` as a marker  emphasis-marker  remark-lint

⚠ 1 warning
```

</details>

### `compileSync(file, options?)`

Compile MDX to JS.
Synchronous version of `compile`.
When possible please use the async `compile`.

### `evaluate(file, options)`

> ☢️ **Danger**: It’s called **evaluate** because it `eval`s JavaScript.

[Compile][] and [run][] MDX.
When possible, please use `compile`, write to a file, and then run with Node,
or use one of the
[§ Integrations][integrations].
But if you trust your content, `evaluate` can work.

Typically, `import` (or `export … from`) do not work here.
They can be compiled to dynamic `import()` by passing
[`options.useDynamicImport`][usedynamicimport].

> ☢️ **Danger**: you likely must set `development: boolean`.

###### `file`

See [`compile`][compile].

###### `options`

Most options are the same as [`compile`][compile], with the following
exceptions:

- `providerImportSource` is replaced by `useMDXComponents`
- `jsx*` and `pragma*` options are replaced by `Fragment`, `jsx`, `jsxs`, and
  `jsxDEV`
- `outputFormat` is set to `function-body`

###### `options.jsx`

###### `options.jsxs`

###### `options.jsxDEV`

###### `options.Fragment`

These options are required: `Fragment` always, when `development: true`
then `jsx` and `jsxs`, when `development: false` then `jsxDEV`.
They come from an automatic JSX runtime that you must import yourself.

<details>
<summary>展开例子</summary>

```js
import * as runtime from 'react/jsx-runtime'

const { default: Content } = await evaluate('# hi', {
  ...runtime,
  ...otherOptions,
  development: false,
})
```

</details>

###### `options.useMDXComponents`

Needed if you want to support a provider.

<details>
<summary>展开例子</summary>

```js
import * as provider from '@mdx-js/react'
import * as runtime from 'react/jsx-runtime'

const { default: Content } = await evaluate('# hi', {
  ...provider,
  ...runtime,
  ...otherOptions,
  development: false,
})
```

</details>

###### Returns

`Promise<MDXModule>` — Promise that resolves to something that looks a bit like
a module: an object with a `default` field set to the component and anything
else that was exported from the MDX file available too.

<details>
<summary>展开例子</summary>

Assuming the contents of `example.mdx` from [§ Use][use] was in `file`, then:

```js
import * as runtime from 'react/jsx-runtime'
import { evaluate } from '@mdx-js/mdx'

console.log(await evaluate(file, { ...runtime, development: false }))
```

…yields:

```js
{Thing: [Function: Thing], default: [Function: MDXContent]}
```

</details>

###### Note: Performance

Compiling (and running) MDX takes time.
If you’re live-rendering a string of MDX that often changes using a virtual DOM
based framework (such as React), one performance improvement is to call the
`MDXContent` component yourself.
The reason is that the `evaluate` creates a new function each time, which cannot
be diffed:

```diff
 const {default: MDXContent} = await evaluate('…')

-<MDXContent {...props} />
+MDXContent(props)
```

### `evaluateSync(file, options)`

> ☢️ **Danger**: It’s called **evaluate** because it `eval`s JavaScript.

Compile and run MDX.
Synchronous version of [`evaluate`][eval].
When possible please use the async `evaluate`.

### `run(functionBody, options)`

> ☢️ **Danger**: This `eval`s JavaScript.

Run MDX compiled as [`options.outputFormat: 'function-body'`][outputformat].

###### `options`

You can pass `Fragment`, `jsx`, `jsxs`, and `jsxDEV`, from an automatic JSX
runtime as `options`.
You can also pass `useMDXComponents` from a provider in options if the MDX is
compiled with `options.providerImportSource: '#'` (the exact value of this
compile option doesn’t matter).
All other options have to be passed to `compile` instead.

###### Returns

`Promise<MDXModule>` — See `evaluate`

<details>
<summary>展开例子</summary>

On the server:

```js
import { compile } from '@mdx-js/mdx'

const code = String(
  await compile('# hi', {
    outputFormat: 'function-body',
    development: false,
  }),
)
// To do: send `code` to the client somehow.
```

On the client:

```js
import * as runtime from 'react/jsx-runtime'
import { run } from '@mdx-js/mdx'

const code = '' // To do: get `code` from server somehow.

const { default: Content } = await run(code, runtime)
```

…yields:

```js
[Function: MDXContent]
```

</details>

### `runSync(functionBody, options)`

> ☢️ **Danger**: This `eval`s JavaScript.

Run MDX.
Synchronous version of [`run`][run].
When possible please use the async `run`.

### `createProcessor(options)`

Create a unified processor to compile MDX to JS.
Has the same options as [`compile`][compile], but returns a configured
[`processor`][processor].

Note that `format: 'detect'` does not work here: only `'md'` or `'mdx'` are
allowed (and `'mdx'` is the default).

## Types

这个包是完全使用[TypeScript][]类型的。
请参阅我们网站上的[§Types][Types]获取信息。

额外的`CompileOptions`, `EvaluateOptions`, 和 `ProcessorOptions`类型被导出，它们代表了各自方法的可接受配置。

## 体系结构

要了解这个项目做什么，首先了解unified做什么是非常重要的:
请阅读[`unifiedjs/unified`][unified]自述文件(部分直到你击中API部分是必须阅读的)。

`@mdx-js/mdx`是一个unified的管道包装，所以大多数人不需要知道unified: [`core.js#L65`][core]。
处理器执行以下步骤:

1.  将MDX(带有嵌入式JSX、ESM和表达式的序列化markdown)解析为mdast(markdown语法树)
2.  通过remark进行转型(markdown生态系统)
3.  转换mdast到hast(HTML语法树)
4.  通过rehype进行转换(HTML生态系统)
5.  转换hast到esast (JS语法树)
6.  获得组件需要做哪些工作
7.  通过recma (JS生态系统)进行转型
8.  将esast序列化为JavaScript

*输入*是MDX(带有嵌入式JSX、ESM和表达式的序列化标记)。
The markdown is parsed with [`micromark/micromark`][micromark] and the embedded
JS with one of its extensions
[`micromark/micromark-extension-mdxjs`][micromark-extension-mdxjs] (which in
turn uses [acorn][]).
Then [`syntax-tree/mdast-util-from-markdown`][mdast-util-from-markdown] and its
extension [`syntax-tree/mdast-util-mdx`][mdast-util-mdx] are used to turn the
results from the parser into a syntax tree: [mdast][].

Markdown is closest to the source format.
This is where [remark plugins][remark-plugins] come in.
Typically, there shouldn’t be much going on here.
But perhaps you want to support GFM (tables and such) or frontmatter?
Then you can add a plugin here: `remark-gfm` or `remark-frontmatter`,
respectively.

After markdown, we go to [hast][] (HTML).
This transformation is done by
[`syntax-tree/mdast-util-to-hast`][mdast-util-to-hast].
Wait, why, what does HTML have to do with it?
Part of the reason is that we care about HTML semantics: we want to know that
something is an `<a>`, not whether it’s a link with a resource (`[text](url)`)
or a reference to a defined link definition (`[text][id]\n\n[id]: url`).
So an HTML AST is _closer_ to where we want to go.
Another reason is that there are many things folks need when they go MDX -> JS,
markdown -> HTML, or even folks who only process their HTML -> HTML: use cases
other than MDX.
By having a single AST in these cases and writing a plugin that works on that
AST, that plugin can supports _all_ these use cases (for example,
[`rehypejs/rehype-highlight`][rehype-highlight] for syntax highlighting or
[`rehypejs/rehype-katex`][rehype-katex] for math).
So, this is where [rehype plugins][rehype-plugins] come in: most of the plugins,
probably.

Then we go to JavaScript: [esast][] (JS; an
AST which is compatible with estree but looks a bit more like other unist ASTs).
This transformation is done by
[`syntax-tree/hast-util-to-estree`][hast-util-to-estree].
This is a new ecosystem that does not have utilities or plugins yet.
But it’s where `@mdx-js/mdx` does its thing: where it adds imports/exports,
where it compiles JSX away into `_jsx()` calls, and where it does the other cool
things that it provides.

Finally, The output is serialized JavaScript.
That final step is done by [astring][], a
small and fast JS generator.

## 安全

详见我们网站上的[§Security][Security]。

## 贡献

请参阅我们网站上的[§Contribute][Contribute]了解开始的方法。
参见[§Support][Support]获取帮助的方法。

这个项目有[行为准则][准则]。
通过与此存储库、组织或社区进行交互，您同意遵守其条款。

## 许可证

[MIT][] © Compositor and [Vercel][]

[build-badge]: https://github.com/mdx-js/mdx/workflows/main/badge.svg
[build]: https://github.com/mdx-js/mdx/actions
[coverage-badge]: https://img.shields.io/codecov/c/github/mdx-js/mdx/main.svg
[coverage]: https://codecov.io/github/mdx-js/mdx
[downloads-badge]: https://img.shields.io/npm/dm/@mdx-js/mdx.svg
[downloads]: https://www.npmjs.com/package/@mdx-js/mdx
[size-badge]: https://img.shields.io/bundlephobia/minzip/@mdx-js/mdx.svg
[size]: https://bundlephobia.com/result?p=@mdx-js/mdx
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
[mit]: https://github.com/mdx-js/mdx/blob/main/packages/mdx/license
[vercel]: https://vercel.com
[compile]: #compilefile-options
[compile-sync]: #compilesyncfile-options
[eval]: #evaluatefile-options
[run]: #runfunctionbody-options
[create-processor]: #createprocessoroptions
[buffer]: https://nodejs.org/api/buffer.html
[source-map]: https://github.com/mozilla/source-map
[vfile]: https://github.com/vfile/vfile
[remark-plugins]: https://github.com/remarkjs/remark/blob/main/doc/plugins.md#list-of-plugins
[rehype-plugins]: https://github.com/rehypejs/rehype/blob/main/doc/plugins.md#list-of-plugins
[remark-rehype]: https://github.com/remarkjs/remark-rehype
[mdx-syntax]: https://mdxjs.com/docs/what-is-mdx/#mdx-syntax
[use]: #use
[outputformat]: #optionsoutputformat
[baseurl]: #optionsbaseurl
[usedynamicimport]: #optionsusedynamicimport
[unified]: https://github.com/unifiedjs/unified
[processor]: https://github.com/unifiedjs/unified#processor
[core]: https://github.com/mdx-js/mdx/blob/main/packages/mdx/lib/core.js#L65
[micromark]: https://github.com/micromark/micromark
[acorn]: https://github.com/acornjs/acorn
[micromark-extension-mdxjs]: https://github.com/micromark/micromark-extension-mdxjs
[mdast-util-from-markdown]: https://github.com/syntax-tree/mdast-util-from-markdown
[mdast-util-mdx]: https://github.com/syntax-tree/mdast-util-mdx
[mdast]: https://github.com/syntax-tree/mdast
[mdast-util-to-hast]: https://github.com/syntax-tree/mdast-util-to-hast
[hast]: https://github.com/syntax-tree/hast
[hast-util-to-estree]: https://github.com/syntax-tree/hast-util-to-estree
[rehype-highlight]: https://github.com/rehypejs/rehype-highlight
[rehype-katex]: https://github.com/remarkjs/remark-math/tree/main/packages/rehype-katex
[esast]: https://github.com/syntax-tree/esast
[astring]: https://github.com/davidbonnet/astring
[integrations]: https://mdxjs.com/getting-started/#integrations
[using-mdx]: https://mdxjs.com/docs/using-mdx/
[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[types]: https://mdxjs.com/getting-started/#types
[security]: https://mdxjs.com/getting-started/#security
[typescript]: https://www.typescriptlang.org
