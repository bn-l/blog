---
title: 'Relative imports in python'
description: "Extremely convoluted and a perfect trap for the beginner or otherwise"
date: 2023-10-17T09:16:34+11:00
draft: false
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: relative-imports-pythons-terrible-system
keywords: 
    - python
    - relative
    - imports
# lastmod: 
series: 
    - python
    - python-internals
# tags: 
weight: 2
---



If you have spent hours trying to fully understand the way python does imports after getting (unhelpful) errors like `ModuleNotFoundError: No module named <your module>` or `ImportError: attempted relative import with no known parent package` and have been banging your head against the truly awful python docs, then look no further.

I'll explain why relative imports don't work when you run a file directly (`python somefile.py`) and how the import system works in general. It's less convoluted than the docs make it out to be. This will be just the critical facts without the blabbing. Estimated reading time: 15 minutes.

## Module vs Package:

### Module

- Usually means a `.py` file but, confusingly, can also refer to a package.

### Package
- A module that contains other modules. E.g.: A folder with an `__init__.py` and `.py` files.

## Import procedure:

When you `import`, python will search for the module in the following way:

### 1. Convert relative imports to absolute 

Relative imports have dots at the start; Absolute imports do not. That's it.  Absolute: `import somepackage`. Relative: `import .sibling`. 

You might have done this before:  
In `/folder/pyfile.py` you have the line `from siblingfile import somefunc`, and then you ran: `python pyfile.py`, and it worked.

```shell
folder/
├── __init__.py
├── pyfile.py 
└── siblingfile.py
```


This is an absolute import of a module. Python knew where to look because the folder containing `pyfile.py` and `siblingfile.py` is automatically added to the module search paths when you ran `python pyfile.py`.

Python only does absolute imports behind the scenes so relative paths need to be converted to absolute. It will use a variable called `__package__` (more detail further on) to do this. This is less complicated than it sounds: [this is the exact function that does this in python](https://github.com/python/cpython/blob/c1e5343928b4e52cc91251fc8680ec3acc31e7a8/Lib/importlib/_bootstrap.py#L1225) (the `package` parameter comes from the `__package__` variable). 

This is probably isn't fully clear at the moment but I invite you to retain your curiosity as I’ll flesh out the explanation further down.

### 2. Cache check

Python will first look if the module is already imported by checking the already imported module cache[^1]. If it's there, it does nothing.

### 3. Lookup list of paths

If not in the cache it will use a list of paths added to the start to the absolute import path to find the module. E.g. for `import somepackage.subpackage.somecode` it will check if the folder `C:\Python311\Lib\somepackage\subpackage` exists and whether a folder `somecode\` or `somecode.py` is in it. If not it will keep searching by prepending the paths. It's really that basic.

This list of paths can be seen if you run this command: `import sys` then `print(sys.path)`[^2]. For me on windows the output is:

- 'C:\\Python311\\Lib', 
- 'C:\\Python311', 
- 'C:\\Python311\\Lib\\site-packages', 
- **C:\\Users\\me\\Desktop\\mypackage',**  <mark>(this is critical)</mark>

Note the last line above:

- I ran: `python.exe C:\Users\me\Desktop\mypackage\main.py` and when you do this <mark>python adds the current folder</mark> to the lookup path list.

- So: `C:\Users\me\Desktop\mypackage\` is also a location where python will search for absolute imports I specify by prepending that path. All python files and subfolders of this path are now available for import.

**However if I run a file in a subfolder:**  
`python.exe C:\Users\me\Desktop\mypackage\subfolder1\module1.py.py` 

-  Then only: `C:\Users\me\Desktop\mypackage\sub1\` is added to the lookup list (and only *its* files and subfolders are searched—apart from the default paths). 

-  **The issue:** In `module1.py` below if I try to import from `module2.py` there will be an error as python only knows about what's in `subfolder1`. 

```shell
mypackage/
├── __init__.py
├── main.py
├── subfolder1/
│   ├── __init__.py
│   └── module1.py
└── subfolder2/
    ├── __init__.py
    └── module2.py
```

### 5. Execute `__init__.py` files

Any `__init__.py` files along the import path and the module being imported will be executed. E.g. `from subpackage.somefile import somefunc` will execute: 

- `subpackage\__init__.py` if it exists, and:

- `subpackage.py`[^3]

    

## Where relative imports work

**Doesn't work**: 

- `python somefile.py` (AKA "running directly"[^4])

- `python -m somefile`  (note that "some<u>**file**</u>"—is a file not a directory)

**Works**: 

- `python -m somefolder`
    - As long as the argument after -m[^5] is a folder or a dot path to a file within a folder. E.g. for the folder structure `/somedirectory/somefile.py`: running `python -m somedirectory`[^6] and `python.exe -m somedirectory.somefile` will make relative imports work.
- `import somefile` or `import somefolder` 
    - Relative imports also work in `somefile` and `somefolder` when you import them in another package (NB: they have to be findable in the lookup path).  


In more detail, a relative import will work if the module's `__package__` variable is not `None`. Python uses this variable to to construct an absolute path from a relative one. There's no real technical reason why relative imports can't work everywhere but I suppose a lot of code would have to be changed. It's a pretty basic check. The exact code for it is [here](https://github.com/python/cpython/blob/78e4a6de48749f4f76987ca85abb0e18f586f9c4/Lib/importlib/_bootstrap.py#L1288C26-L1288C26). Python looks at the import string in the file and counts the number of dots. E.g. It literally looks at `from ..somepackage import blahblah` and counts the dots at the beginning (two) and calls this the import "level". If the "level" is > 0, and the `__package__` variable is `None` it will throw this error ([link](https://github.com/python/cpython/blob/78e4a6de48749f4f76987ca85abb0e18f586f9c4/Lib/importlib/_bootstrap.py#L1289) to exact location in code):  `attempted relative import with no known parent package`. For each dot "level" you need a certain number of packages in `__package__` separated by dots. E.g. if `__package__` = `folder.somepackage ` then the above import will work because python literally runs  `len(__package__.split("."))` and then compares that to the number of dots. I'm not even kidding.

When running a file directly: `python.exe somefile.py` or: `python.exe -m somefile`, `__package__` is set to `None`. And that's it. That's why relative imports create an error.



## Namespace vs "regular" packages

Before python 3.3 there were only "regular" packages: Some folder with an `__init__.py` file in it. 3.3 added "namespace" packages. The documentation on these are sparse / very badly written and the [pep](https://peps.python.org/pep-0420/) is absolutely awful.

**Namespace packages**: <mark>Can be safely ignored</mark> as a concept if you don't need to spread a package out over multiple folders (i.e. most use cases). See below if you're *really* interested in the topic otherwise just skip this and be sure to use regular packages:

**Regular packages**: A folder with an `__init__.py` file in it.

### What a namespace package is:

```shell
container1/
└── spreadoutpackage/
    └── lowercase.py
container2/
└── spreadoutpackage/
    └── niceprint.py
```

Note that `spreadoutpackage` has the **same name** in two different folders and that there are **no `__init__.py`** files. Succintly: **if** `container1/` and `container2/` are in the [lookup path list](#4-lookup-list), then these two calls from a python file: 

- `from spreadoutpackage.lowercase import CaseLowerer`, and:
-  `from spreadoutpackage.niceprint import print_this`, 

will work (whereas before python 3.3 it wouldn't). 

This means you can spread a single package out over many folders. It's like a "virtual module".

You might have been unintentionally using a namespace package if you have a python project folder and you didn't create `__init__.py` files. 



## References

*5. The import system*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/reference/import.html

*6. Modules*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/tutorial/modules.html

*9. Top-level components*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3.12/reference/toplevel_components.html

*Cpython importlib/_bootstrap.py source code*. (n.d.). Retrieved October 17, 2023, from https://github.com/python/cpython/blob/3.12/Lib/importlib/_bootstrap.py

*Cpython importlib/__init__.py source code*. (n.d.). Retrieved October 17, 2023, from https://github.com/python/cpython/blob/3.12/Lib/importlib/__init__.py

*Glossary*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/glossary.html

*__main__ — Top-level code environment*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/__main__.html

*Packaging namespace packages — Python Packaging User Guide*. (n.d.). Python.org. Retrieved October 17, 2023, from https://packaging.python.org/en/latest/guides/packaging-namespace-packages/

*PEP 328 – imports: Multi-line and absolute/relative*. (n.d.). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0328/

*PEP 338 – Executing modules as scripts*. (n.d.). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0338/

*PEP 366 – Main module explicit relative imports*. (n.d.). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0366/

*PEP 420 – implicit namespace packages*. (n.d.). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0420/

*runpy — Locating and executing Python modules*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/runpy.html

*sys — System-specific parameters and functions*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/sys.html

*The initialization of the sys.path module search path*. (n.d.). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/sys_path_init.html

*What is the purpose of the -m switch?* (n.d.). Stack Overflow. Retrieved October 17, 2023, from https://stackoverflow.com/questions/7610001/what-is-the-purpose-of-the-m-switch



[^1]: Which, for whatever reason, is in a variable called `sys.modules`. You can `import sys` then `print(sys.modules)` in a python file to see the cache.
[^2]: Python stores this list of paths in the variable`sys.path`. This variable is [initialized](https://docs.python.org/3/library/sys_path_init.html) from a set of defaults (like C:\\Python311\\Lib\\site-packages, etc), and the **current folder where python is being executed**.
[^3]: Python executes files as they're imported. 
[^4]: Running a file directly like this in python sets it manually as the the "`__main__`" file. The "main" file has its `__name__` variable set to `__main__` by python interpretor (see point above also). The idea being to have an "entry point" of execution into a python program. From the [docs](https://docs.python.org/3.12/reference/toplevel_components.html#programs:~:text=The%20latter%20is%20used%20to%20provide%20the%20local%20and%20global%20namespace%20for%20execution%20of%20the%20complete%20program.): The file designated as `__main__` "provides the local and global namespace for execution of the complete program"
[^5]: When running using the "-m switch" python will run the code / folder, etc with a program called "[runpy](https://docs.python.org/3.12/library/runpy.html)"

[^6]: Need to have a `__main__.py` [file](https://docs.python.org/3/library/__main__.html#main-py-in-python-packages) in the folder—which will be run and considered the main file automatically (see footnote 7).

