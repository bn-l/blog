---
title: 'More Python Vscode QoL'
date: 2023-10-20T16:03:04+11:00
draft: false
description: How to make sure the venv opens in the terminal on startup automatically and other tips
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: more-python-vscode-qol
# keywords:
#   -  
# lastmod: 
series:
  -  python
  -  vscode
---

<!-- <span class="summary">**Summary**: In a sentence... </span> -->

### Helpful User Settings for Python in Vscode  



Activate the the venv in the current terminal on startup:
```json
"python.terminal.activateEnvInCurrentTerminal": true,
```



Hide python related folders from vscode's file explorer (stops venv from opening and taking up a lot of space each time you install something):

```json
    "files.exclude": {
        "**/__pycache__": true,    // Exclude Python bytecode cache folder
        "**/*.pyc": true,          // Exclude Python compiled bytecode files
        "**/*.pyo": true,          // Exclude Python optimized bytecode files
        "**/*.egg-info": true,     // Exclude Python package metadata folders
        "**/.mypy_cache": true,    // Exclude mypy type checking cache folder
        "**/.pytest_cache": true,  // Exclude pytest test runner cache folder
        "**/.tox": true,           // Exclude tox virtual environment folder
        "**/.venv": true           // Exclude Python virtual environment folder
    },
```



Various color customizations. Including changes to decorators. I use a light theme and I like decorators to be more subtle (note this won't work if the decorators have arguments):

```json
    "editor.tokenColorCustomizations": {
        "textMateRules": [
            {
                "scope": "entity.name.function.decorator.python",
                "settings": {
                    "foreground": "#a1a1a1"
                }
            },
        ]
    },

    "editor.semanticTokenColorCustomizations": {
        "enabled": true,
        "rules": 
            {
            "function.decorator": {
                "foreground": "#a1a1a1"
            },
            "class.decorator": {
                "foreground": "#a1a1a1"
            },
            "variable.decorator": {
                "foreground": "#a1a1a1"
            },
            "variable.typeHint": {
                "foreground": "#267F99"
            },
            "property": {
                "foreground": "#572c2c"
            },
        }
    },
```


In `.vscode/tasks.js` this will run the current file as a module within the correct environment, show any output and then close the **panel** on key press (and not just the separate terminal the task runs in--this is extremely confusing). It also hides any annoying boilerplate messages (note: you will need the plugin mentioned [here](/posts/debugging-python-in-vscode)):

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "run python",
            "type": "shell",
            "command": "${command:python.interpreterPath}",
            "args": ["-m", "${workspaceFolderBasename}.${command:extension.commandvariable.file.relativeFileDotsNoExtension}"],
            "options": {
                "cwd": "${command:extension.commandvariable.workspace.folder1Up}"
            },
            "presentation": {
                "reveal": "always",
                "focus": true,
                "panel": "shared",
                "echo": false,
                "clear": true,
                "close": false,
                "showReuseMessage": false,
            },
            "problemMatcher": []
        },
        {
            "label": "close terminal panel",
            "command": "${command:workbench.action.closePanel}"
        },
        {
            "label": "run python then close panel",
            "dependsOrder": "sequence",
            "dependsOn": ["run python", "close terminal panel"]
        }
    ],
}
```

In `keybindings.json` assign keyboard shortcuts to a command and to a task (using the args property). Here I've assigned the above run and close task:

```json
    {
        "key": "ctrl+shift+alt+enter",
        "command": "workbench.action.debug.run",
        "when": "debuggersAvailable && debugState != 'initializing'"
    },
    {
        "key": "ctrl+alt+enter",
        "command": "workbench.action.tasks.runTask",
        "args": "run python then close panel",
        "when": "editorTextFocus && !notebookEditorFocused && editorLangId == 'python'"
    },
```









