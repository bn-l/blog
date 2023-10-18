---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: {{ .Date }}
draft: true
# description: 
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: {{ .File.ContentBaseName }}
# keywords:
#   -  
# lastmod: 
# series:
#   -  
# tags: 
#   -
---

