---
title: "Setting up ~~this~~ _a_ blog using metalsmith - Part 2"
description: "Changing the markdown parser to add extensions: emoji, containers and abbreviations"
date: 2022-12-08
lastmod: 2022-12-14
draft: true
slug: setting-up-this-blog-using-metalsmith-part-2
---

[Link to part 1](/posts/setting-up-this-blog-using-metalsmith-part-1)

I added support for a few rarer markdown extensions by changing the markdown renderer used from "[Marked](https://github.com/markedjs/marked)" to [markdown-it](https://github.com/markdown-it)--which has a plugin system and good plugins. To do this I'm using the [metalsmith-markdownit](https://github.com/mayo/metalsmith-markdownit) [package](https://www.npmjs.com/package/metalsmith-markdownit) (which unfortunately was last updated Dec 2018) with the following extensions:

- [markdown-it-abbr](https://github.com/markdown-it/markdown-it-abbr): Allows for abbreviations <abbr title="Like This One">LTO</abbr>
- [markdown-it-container](https://github.com/markdown-it/markdown-it-container): For ["custom containers"](https://github.com/markdown-it/markdown-it-container) (a nice syntax to wrap a paragraph in a custom class div. E.g. for warnings)
- [markdown-it-emoji](https://github.com/markdown-it/markdown-it-emoji): We all know emojis are **100% extremely critical** :fearful:
- [markdown-it-footnote](https://github.com/markdown-it/markdown-it-footnote)

And I made these changes to my Metalsmith.js to make use of the above:

``` js
const markdown = require('metalsmith-markdownit')
var md = markdown({html: true});
md.parser
  .use(require('markdown-it-emoji'))
  .use(require('markdown-it-abbr'))
  .use(require('markdown-it-footnote'))
  .use(require('markdown-it-container'), "update")
  .use(require('markdown-it-container'), "info")
  .use(require('markdown-it-container'), "summary")
Metalsmith
// ... the other metalsmith middleware use calls
    .use(md)
```

## Dates

The [barebones / startbootstrap template](https://github.com/metalsmith/startbootstrap-clean-blog/) I am using as a base uses [static dates](https://github.com/metalsmith/startbootstrap-clean-blog/blob/master/metalsmith.js#L31), created by JS at build time. In order to get dynamic dates based on the client's locale, I (very lazily) adapted the terrible code on this template with Luxon's (formerly moment.js) DateTime parser / formatter:

```js
const DateTime = require("luxon").DateTime
```

And passed that to the metadata object "sitedata":

```js
const sitedata = {
    DateTime: DateTime
    // the rest of the metadata
}
```

So that from my pug template I can do this:

```pug
.date= "Posted: " + DateTime.fromMillis(parseInt(timestamp)).toFormat("dd-LL-yy")
```

Pretty nice.


<span class="update">**Update 14/12/22:** I added [mermaidjs](https://mermaid-js.github.io/) graph parsing to markdown-it. Does flowcharts, UML class diagrams and more. Incredibly cool. There are a few packages on NPM and a few repos. The one to use is: ["markdown-it-textual-uml"](https://github.com/manastalukdar/markdown-it-textual-uml#readme). But take care to include the mermaid script with the defer and type="module" setting.</span>

