---
title: 'Debugging Python in Vscode: The ideal setup'
date: 2023-10-20T15:38:32+11:00
draft: false
description: This is a follow on from the post about imports
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: debugging-python-in-vscode
# keywords:
#   -  
# lastmod: 
series:
    - python-imports
    - python
    - vscode
---

<!-- <span class="summary">**Summary**: In a sentence... </span> -->

Because of the way python's import system "works" (see previous [post](/posts/relative-imports-pythons-terrible-system)) I have created a custom debugging [launch.json](https://code.visualstudio.com/docs/editor/debugging) configuration in vscode for when I want to run a single file in a subpackage (also works with a file in the root folder).

To use this configuration you will need the [Command Variable](https://marketplace.visualstudio.com/items?itemName=rioj7.command-variable) ([repo](https://github.com/rioj7/command-variable)) extension.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Module",
            "type": "python",
            "request": "launch",
            "cwd": "${command:extension.commandvariable.workspace.folder1Up}",
            "module": "${workspaceFolderBasename}.${command:extension.commandvariable.file.relativeFileDotsNoExtension}",
            "justMyCode": true, // not strictly required
        },
    ]
}
```



Now when you debug this will set the working directory to the parent of the workspace folder and then run the current file as a module (without the .py extension) with the dot path relative to the workspace folder. It's complicated but it will mean that relative imports work and that you are debugging similar to how the file will be run when it's part of a larger imported module. See previous [post](/posts/relative-imports-pythons-terrible-system) for a detailed explanation on why this is necessary.



