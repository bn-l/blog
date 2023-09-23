---
title: 'Moving Blog to Hugo'
date: 2023-09-22T08:21:45+10:00
draft: false
images: 
    - ./images/default-cover.png
slug: moving-blog-hugo
tags: 
    - meta
---

### Moving to Hugo

The site previously used metalsmith.js in a clunky DIY way. Metalsmith allows a lot of markdown extensions quite easily. With hugo you have [markdown render hooks](https://gohugo.io/templates/render-hooks/) that can provide some of the functionality.

#### Documentation

It's weirdly long (verbose) and yet shallow at the same time. The API is documented in a non-standard way. I spent way, way more time than I should have just trying to piece together bits of information spread out on various pages. For that reason, I don't recommend starting with Hugo. And it's a reminder to myself to always check the quality of the documentation (or to see if the code is self documenting) before using some library.

#### Misc

Apart from the documentation the experience is pretty good. There are a lot of good functions (although good luck with the docs) and the templating language isn't too bad. No go specific knowledge is required.

#### QoL Fixes

I also made many customisations, improvements and corrections (view on small screens fixed) to an existing hugo template: https://github.com/bn-l/hugo-paper



A nice wrapper around the hugo command that creates a new post from a yaml template:

```bash
new-hugo() {

  FILENAME=$(echo "content/posts/$*" | sed 's/ /-/g').md
  URL=$(echo "http://localhost:1313/posts/$*" | sed 's/ /-/g')

  hugo new "$FILENAME" && "/c/Users/me/AppData/Local/Programs/Typora/Typora.exe" "$FILENAME" && sleep 1 && hugo server --navigateToChanged -D --disableFastRender --templateMetrics --templateMetricsHints &
  python -m webbrowser "$URL"
}
```

Running `new-hugo some cool new post` will create a new post with the title Some Cool New Post, start the hugo dev server (with the -D flag to show drafts), open the url in a browser, and then open the file in my markdown editor (Typora). 

