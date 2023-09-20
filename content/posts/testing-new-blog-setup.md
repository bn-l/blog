---
title: 'Testing New Blog Setup'
date: 2023-09-20T11:37:39+10:00
draft: true
---

### Moving to Hugo

The site previously used metalsmith.js in a clunky DIY way. Hopefully can get the markdown extensions to work here though too.



Made a nice wrapper around the hugo command that creates a new post from a yaml template:

```bash
new-hugo() {
  FILENAME=$(echo "content/posts/$*" | sed 's/ /-/g').md
  URL=$(echo "http://localhost:1313/posts/$*" | sed 's/ /-/g')
  hugo new "$FILENAME" && "/c/Users/x230/AppData/Local/Programs/Typora/Typora.exe" "$FILENAME" && hugo server -D &
  python -m webbrowser "$URL"
}
```

Running new-hugo some cool new post will create a new post with the title Some Cool New Post, start the hugo dev server (with the -D flag to show drafts), open the url in a browser, and then open the file in my markdown editor (Typora). 

