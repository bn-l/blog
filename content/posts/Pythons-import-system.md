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

Relative imports have dots at the start of the path, absolute imports do not. Absolute: `import somepackage`. Relative: `import .sibling`. 

You might have done this before:  
In `/folder/pyfile.py` you have the line `from siblingfile import somefunc`, and then you ran: `python pyfile.py`, and it worked.

```shell
folder/
├── __init__.py
├── pyfile.py 
└── siblingfile.py
```


You did an absolute import of a module. Python knew where to look because the folder containing `pyfile.py` and `siblingfile.py` was automatically added to the module search paths when you ran `python pyfile.py`.

Python needs absolute paths behind the scenes so relative import paths need to be converted to absolute. It will use a variable called `__package__` (more detail further on) to do this. [This is the exact function that does this in the source code](https://github.com/python/cpython/blob/c1e5343928b4e52cc91251fc8680ec3acc31e7a8/Lib/importlib/_bootstrap.py#L1225) (the `package` parameter comes from the `__package__` variable). 

All this probably isn't fully clear at the moment but I invite you to retain your curiosity as I’ll flesh out the explanation further down.

### 2. Cache check

After converting the path from relative to absolute (if necessary) python will check if the module is already imported by looking at the already imported module cache[^1]. If it's there, it does nothing.

### 3. Lookup list of paths

If not in the cache it will try to find the module by prepending various paths to the import path. E.g. for `import somepackage.subpackage.somecode` it will add the path `C:\Python311\Lib\` and check if the folder `C:\Python311\Lib\somepackage\subpackage` exists and whether a folder `somecode\` or a file `somecode.py` is in it. If not it will keep searching by trying other paths. It's really that basic.

This list of paths it checks can be seen if you run this command: `import sys` then `print(sys.path)`[^2]. For me on windows the output is:

- 'C:\\Python311\\Lib', 
- 'C:\\Python311', 
- 'C:\\Python311\\Lib\\site-packages', 
- **C:\\Users\\me\\Desktop\\mypackage',**  <mark>(this is critical)</mark>

Note the last line above:

- I ran: `python.exe C:\Users\me\Desktop\mypackage\main.py` and when you do this <mark>python adds the current folder</mark> to the lookup path list.

- So: `C:\Users\me\Desktop\mypackage\` was added and all python files and subfolders in that folder are now available for import.

**However if I run a file in a subfolder:**  
`python.exe C:\Users\me\Desktop\mypackage\subfolder1\module1.py.py` 

-  Then only: `C:\Users\me\Desktop\mypackage\subfolder1\` is added to the lookup list (and only *its* files and subfolders are searched—apart from the default paths). 

**The issue:** In `module1.py` (below) if I try to import from `module2.py` there will be an error as python only knows about what's in `subfolder1`. 

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

### 4. Execute `__init__.py` files

Any `__init__.py` files along the import path and the module being imported will be executed. E.g. `from subpackage.somefile import somefunc` will execute: 

- `subpackage\__init__.py` if it exists, and:

- `subpackage.py`[^3]

Back to relative imports...
    

## Where relative imports work

**Doesn't work**: 

- `python somefile.py` (AKA "running directly"[^4])

- `python -m somefile`  (where "somefile" is a file not a folder)

**Works**: 

- `python -m somefolder`
    - As long as the argument after -m is a folder or a dot path to a file within a folder[^5] (E.g. for the folder structure `/somedirectory/somefile.py`: running `python -m somedirectory`[^6] will work and so will `python.exe -m somedirectory.somefile`).
- `import somefile` or `import somefolder` 
    - Relative imports also work in `somefile` and `somefolder` when you import them in another package (NB: they have to be findable in the lookup path).  

Behind the scenes python does a simple check to see `if __package__ == None`.  When running a file directly: `python.exe somefile.py` or: `python.exe -m somefile`, `__package__` is set by python to `None`. And that's why relative imports create an error. Check out the source code if you don't believe me. There's no real technical reason why relative imports can't work everywhere but I suppose a lot of code would have to be changed. The exact code for this check is [here](https://github.com/python/cpython/blob/e7ae43ad7dde74e731a9d258e372d17f3b2eb893/Lib/importlib/_bootstrap.py#L1225C13-L1225C13).

A relative import will work when calling `python.exe -m somefolder` because the module's `__package__` variable is set to `somefolder`. To check whether or not to allow a relative import it will look at the import string in a file and count the number of dots. Literally it will look at `from ..somepackage import blahblah` and count the dots at the beginning (two) and call this the import "level". If the "level" is > 0, and the `__package__` variable is `None` it will throw this error ([link](https://github.com/python/cpython/blob/78e4a6de48749f4f76987ca85abb0e18f586f9c4/Lib/importlib/_bootstrap.py#L1289) to exact location in code):  `attempted relative import with no known parent package`. This is because for each dot "level" you need a certain number of packages in `__package__` separated by dots. E.g. if `__package__` = `folder.somepackage ` then the above import will work because python runs  `len(__package__.split("."))` and then compares that to the number of dots. I'm not even kidding.



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

The relevance to imports is that you might have been unintentionally importing namespace packages if you have a python project folder and you didn't create `__init__.py` files (which can lead to confusing behaviour).



## References

*5. The import system*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/reference/import.html

*6. Modules*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/tutorial/modules.html

*9. Top-level components*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3.12/reference/toplevel_components.html

*Cpython importlib/_bootstrap.py source code*. (2023, August 29). Retrieved October 17, 2023, from https://github.com/python/cpython/blob/3.12/Lib/importlib/_bootstrap.py

*Cpython importlib/__init__.py source code*. (2023, May 3). Retrieved October 17, 2023, from https://github.com/python/cpython/blob/3.12/Lib/importlib/__init__.py

*Glossary*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/glossary.html

*__main__ — Top-level code environment*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/__main__.html

*Packaging namespace packages — Python Packaging User Guide*. (2023, June 16). Python.org. Retrieved October 17, 2023, from https://packaging.python.org/en/latest/guides/packaging-namespace-packages/

*PEP 328 – imports: Multi-line and absolute/relative*. (2003, December 21). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0328/

*PEP 338 – Executing modules as scripts*. (2004, October 16). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0338/

*PEP 366 – Main module explicit relative imports*. (2007, May 01). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0366/

*PEP 420 – implicit namespace packages*. (2012, April 19). Python.org. Retrieved October 17, 2023, from https://peps.python.org/pep-0420/

*runpy — Locating and executing Python modules*. (2023, August 7). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/runpy.html

*sys — System-specific parameters and functions*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/sys.html

*The initialization of the sys.path module search path*. (2023). Python Documentation. Retrieved October 17, 2023, from https://docs.python.org/3/library/sys_path_init.html

*What is the purpose of the -m switch?* (2011, September 30). Stack Overflow. Retrieved October 17, 2023, from https://stackoverflow.com/questions/7610001/what-is-the-purpose-of-the-m-switch



[^1]: Which, for whatever reason, is in a variable called `sys.modules`. You can `import sys` then `print(sys.modules)` in a python file to see the cache.
[^2]: Python stores this list of paths in the variable `sys.path`. This variable is [initialized](https://docs.python.org/3/library/sys_path_init.html) from a set of defaults (like C:\\Python311\\Lib\\site-packages, etc), and the **current folder where python is being executed**.
[^3]: Python executes files as they're imported. 
[^4]: Running a file directly like this in python sets it manually as the the "`__main__`" file. The "main" file has its `__name__` variable set to `__main__` by python interpretor (see point above also). The idea being to have an "entry point" of execution into a python program. From the [docs](https://docs.python.org/3.12/reference/toplevel_components.html#programs:~:text=The%20latter%20is%20used%20to%20provide%20the%20local%20and%20global%20namespace%20for%20execution%20of%20the%20complete%20program.): The file designated as `__main__` "provides the local and global namespace for execution of the complete program"
[^5]: Trivia: When running using the "-m switch" python will run the code / folder, etc with a program called "[runpy](https://docs.python.org/3.12/library/runpy.html)"

[^6]: Need to have a `__main__.py` [file](https://docs.python.org/3/library/__main__.html#main-py-in-python-packages) in the folder—which will be run and considered the main file automatically (see footnote 4).

