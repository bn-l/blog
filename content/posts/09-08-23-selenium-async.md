---
title: Explanation of execute_async_script in selenium - Non spam post
description: The selenium docs are absolutely terrible. And especially on this method on its WebDriver implementation. What is WebDrive? I'll explain it in one sentence.
draft: false
date: 2023-08-09
lastmod: 2023-09-21
slug: selenium-async-explanation
---


<span class="summary">**Summary**: A global is given to the running js script called "arguments". It's an array and its last item is a function that when called returns its first argument to python and ends the async script</span>


WebDriver: A standard way of controlling browsers remotely. Created by the w3c a web standards non-profit. See: https://www.w3.org/TR/webdriver2/

Various browsers implement these standards to create an API for selenium to operate its "driver" class.

The problem is that selenium's documentation expects you have read the webdriver [standard](https://www.w3.org/TR/webdriver2/). And, wow, you may not have. 



**So when you see this:**

![image-20230809095711571](images/image-20230809095711571.png)



**And wonder what is going on,** you are not alone. To expland this documentation:

- the `*args` parameter is a comma separated set of values you can pass to the js script. These are wrapped up into a tuple and then converted into a list. In js the script will access them through a global called `arguments` as an array. 
- **THIS IS IMPORTANT:** The last argument added to the array is the resolve function that let's selenium know when the async function has ended. And it will pass the returned value back to selenium.

