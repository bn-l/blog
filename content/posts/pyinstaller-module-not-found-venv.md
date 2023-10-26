---
title: 'Pyinstaller Module Not Found Venv'
date: 2023-10-26T09:40:36+11:00
draft: false
description: Or "unable to load metadata", etc
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: pyinstaller-module-not-found-venv
# keywords:
#   -  
# lastmod: 
series:
   - python
---



It's very hard to package python desktop apps so that they can be portable. Here's a [comparison](https://docs.python-guide.org/shipping/freezing/) of the available libraries that try to do this. I have tried all of them at various points. None are ideal or without many issues and all have absolutely terrible, opaque documentation (like almost all python projects including the official docs of the language).

From the available options, only cxfreeze and pyinstaller work for me. cx-freeze will not do a single file (but still gives the user an executable to click on—and might be less likely to trigger antivirus), pyinstaller will put everything into a single binary but you will need to whitelist on basically all antivirus (including windows defender).

Recently I used pyinstaller in a project with a virtual environment (the venv module). Pyinstaller couldn't find the packages even though I specified the site-packages inside the venv folder with the CLI options I would get: "Could not find module" errors. In the end I installed it in the venv project and ran it as a module within the venv. This changed the error to "could not find metadata for package". An answer on [stackoverflow](https://stackoverflow.com/a/73825016/2083958) gave me the solution to this. The final command used that got pyinstaller working was:

```bash
python -m PyInstaller --onefile --clean --noconfirm --windowed --hidden-import=ocrmypdf --collect-data ocrmypdf --copy-metadata ocrmypdf --collect-submodules ocrmypdf .\main.py
```

<mark>Take note of the capitalization of PyInstaller</mark>. The cli command is lowercase but the module name, for reasons known **only** to the author of the package, is pascal case. The hidden import is necessary to tell pyinstaller to find and copy the module (it doesn't pick it up automatically), and the `—collect-data` and `—copy-metadata` arguments are also necessary. `—windowed` stops a console coming up for gui apps.













