---
title: 'Windows Console Popup in Python Subprocess'
date: 2023-10-26T10:14:17+11:00
draft: false
description: When running the python subprocess module a windows console will flash / popup. 
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: Windows-console-popup-python-exe-binary
# keywords:
#   -  
# lastmod: 
series:
  - python 
# tags: 
#   -
---



Another esoteric python issue on windows for which there is little documentation. When running a subprocess a console window will popup. This can be especially bad UX if you are running multiple fast subprocesses. The user will see a series of flashing terminal windows. For me, this only happened when the app was bundled into a exe by PyInstaller (see my other [post](/posts/pyinstaller-module-not-found-venv) on the nightmare that is), and not when running as a module with `python -m modulename` (i.e.: [runpy](/posts/relative-imports-pythons-terrible-system)).

I found the solution in a [series](https://stackoverflow.com/questions/7006238/how-do-i-hide-the-console-when-i-use-os-system-or-subprocess-call?noredirect=1&lq=1) of [stackoverflow](https://stackoverflow.com/questions/15899798/subprocess-popen-in-different-console) [posts](https://stackoverflow.com/questions/62221051/subprocess-run-not-suppressing-all-console-output?rq=3):

```python
startupinfo = subprocess.STARTUPINFO()
startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW

proc = subprocess.run(args, creationflags=subprocess.CREATE_NO_WINDOW, startupinfo=startupinfo)
```



 
