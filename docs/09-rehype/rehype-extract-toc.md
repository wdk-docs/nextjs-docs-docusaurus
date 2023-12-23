# rehype-extract-toc

> https://github.com/stefanprobst/rehype-extract-toc

这是一个[`rehype`](https://github.com/rehypejs/rehype)插件，它将文档的目录附加到VFile。

## 如何安装

```sh
yarn add @stefanprobst/rehype-extract-toc
```

## 如何使用

```js
import { rehype } from 'rehype'
import withSlugs from 'rehype-slug'
import withToc from '@stefanprobst/rehype-extract-toc'

const processor = rehype().use(withSlugs).use(withToc)

const file = processor.processSync(doc)

console.dir(file.data.toc, { depth: null })
```

目录将被附加到生成的VFile的`data.toc`属性中。
它的形状如下:

```ts
interface TocEntry {
  value: string
  depth: number
  id?: string
  children?: Array<TocEntry>
}

type Toc = Array<TocEntry>
```

注意，这个插件*不*会生成任何id，你可能会想要将它与`rehype-slug`结合使用。

如何呈现目录由您决定。

## 如何与MDX一起使用

在转换MDX文档时，可以将目录表公开为命名导出，默认为`tableOfContents`。

```js
import { compile } from '@mdx-js/mdx'
import withSlugs from 'rehype-slug'
import withToc from '@stefanprobst/rehype-extract-toc'
import withTocExport from '@stefanprobst/rehype-extract-toc/mdx'

async function run() {
  const file = await compile(doc, {
    rehypePlugins: [
      withSlugs,
      withToc,
      withTocExport,
      /** Optionally, provide a custom name for the export. */
      // [withTocExport, { name: 'toc' }],
    ],
  })

  console.log(String(file))
}

run()
```

如果你使用的是TypeScript，你可以这样添加类型:

```js
/** mdx.d.ts (should be referenced in `tsconfig.json#include`) */
declare module '*.mdx' {
  import type { MDXProps } from 'mdx/types'
  import type { Toc } from '@stefanprobst/rehype-extract-toc'

  export const tableOfContents: Toc
  export default function MDXContent(props: MDXProps): JSX.Element
}
```
