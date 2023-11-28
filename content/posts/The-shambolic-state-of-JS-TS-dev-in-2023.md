---
title: 'The Shambolic State of Typescript Dev in 2023'
date: 2023-11-28T00:21:30+11:00
draft: false
# description: 
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: The-shambolic-state-of-JS-TS-dev-in-2023
keywords:
    - vscode
    - typescript
    - debugging
    - ts-node
    - tsx 
# lastmod: 
series:
    - typescript
    - vscode
    - node
---



I recently started working on a typescript project after not using JS or TS for couple of years. Just getting my dev environment setup (vscode) has been an absolute nightmare and all the problems that annoyed me a few years ago still plague the ecosystem. The biggest being the different import system that's meant to run in a browser vs node. CommonJS (whatever that is) and the interop between the other style: ESM MODULES / ES MODULES / esm / es style modules or ecma script style modules (does anyone know or care what [ecma](https://ecma-international.org/mission/) is? Why do we keep saying ecma all the time?) 

Trying to get typescript working for the first time adds another layer of frustration to an already annoying ecosystem. You can waste an amazing amount of time just trying to use *typescript*.

## Executing

Getting a typescript script running was a real pain coming back after a few years. Node 20, now LTS, breaks ([issue](https://github.com/TypeStrong/ts-node/issues/1997) opened 7 months ago) [ts-node](https://typestrong.org/ts-node/docs/), the executable that allows node to run typescript files without compiling them to js. It's strange to me. Typescript is not just some niche in the wider JS ecosystem—where it might be understandable to be in this state—it's pretty much the front end of the whole web now and the stats back that up. 38% out 87k total respondents in stackoverflow's dev survey rated it as their most used language. To just run the programs these people make in a dev environment, of the available options, ts-node is the default way ([18.4 million](https://www.npmjs.com/package/ts-node) weekly npm downloads). And yet it's been broken for seven months on the current node LTS.

A commonly recommended solution is [tsx](https://github.com/privatenumber/tsx#readme) which has 6k stars on github and 860k weekly downloads on [npm](https://www.npmjs.com/package/tsx). It's fast and "just works" (sort of) **BUT** the error messages it produces are garbled because it uses the compiled ("minified" etc) javascript. So [sourcemaps](https://www.typescriptlang.org/tsconfig#sourceMap) are broken for some reason. I can live with this but it's highly annoying. 

In the end, for running code in a dev environment, I went down to node 18 so I could use ts-node.

This is the vscode task:

```json
		{
			"label": "run",
			"type": "npm",
			"script": "start:dev -- ${file}",
			"problemMatcher": [
				"$tsc"
			],
		}
```

And this is the package.json script:


```json
// scripts: {
	"start:dev": "ts-node"
//...
```

It passes the current file as an argument to `ts-node`.

## Debugging

I couldn't get debugging to work properly in vscode. It does not pause the debugger on uncaught exceptions. 

There are various ways to debug a node app using vscode. This [post](https://sevic.dev/notes/debugging-nodejs-vscode/) ([a](https://archive.md/Vo30R)) is the best I found. But it doesn't mention uncaught exceptions not being caught.

After some really deep and tediously long searching I found out that this has something to do with the way node imports """""es""""" modules when it runs. It wraps the import statement in a try catch catch block so even if the module throws, it will won't show up when debugging (no uncaught exception to break on). I found this out by scouring the web and finding this **single** stackoverflow [question](https://stackoverflow.com/questions/77353715/why-doesnt-the-vs-code-js-debugger-pause-on-uncaught-exceptions-in-es-modules-i) ([a](https://archive.md/HvYko)) which lead to this [issue](https://github.com/microsoft/vscode-js-debug/issues/1861) ([a](https://archive.md/69iJK)) on the js debugger for vs code and this [issue on node itself](https://github.com/nodejs/node/issues/50430) ([a](https://archive.md/k0CW7)).

The vscode js debugger plugin, even though it's under the microsoft github org, has been [maintained by one person](https://github.com/microsoft/vscode-js-debug/graphs/contributors) for the last three years (they work for ms on vscode).

So is this how people are debugging node apps with es modules? In this terrible state? Why is there not more talk on this? It's very strange (to me). 

## Modules

All these issues can be traced back to the way modules[^1] "work" in JS. The bifurcation in the system between the web and node has created a tower of babel type of chaos and legacy that towards the end of 2023 is its biggest weakness. 

For example, in this typescript project I started recently, I got the errors:

```shell
Node.js: SyntaxError: Cannot use import statement outside a module
```

And:

```shell
[ERR_UNKNOWN_FILE_EXTENSION]: Unknown file extension ".ts"...
```

Both of these errors are extremely nondescript and don't tell you how to fix them, or where in the docs you can look. If you are not already aware that they are using a special terminology[^2] it can be even more opaque. This is another huge frustration. Any issues you have you will spend hours scouring the web, hoping that some expert in the idiosyncrasies of this domain has commented on the problem. (On modules in general there's the not very good [node docs](https://nodejs.org/docs/latest-v18.x/api/esm.html#esm_modules_ecmascript_modules) v18.)

To fix these you will be advised on stackoverflow to add this to your "package.json":

```json
"type": "module"
```

The problem with this though is that node will treat all files in the project as if they are es modules. It is crucial to know that is a package-wide setting and it applies to [all the folders in node_modules.](https://nodejs.org/docs/latest-v18.x/api/packages.html#packagejson-and-file-extensions)[^3] So any dependencies you have that use commonJS imports (i.e. `require`) will now break. 

An alternative proposal is to leave this out and just put "m" in front of your js / ts file extensions ([I'm not kidding](https://nodejs.org/docs/latest-v18.x/api/packages.html#packagejson-and-file-extensions)).



Coming back to node / ts / js after a while away has made me remember why I stopped using the ecosystem. The time you can waste on nailing down issues is enormous. I came back because like it or not it's necessary for the web but just getting setup has been really frictional. There is hope though. [Bun](https://bun.sh/docs/runtime/modules) promises to be faster and have "a consistent and predictable module resolution system that just works." It's still early days and it's a beta stage library using zig, an insanely cool [but also beta stage language](https://ziglang.org/), which it uses to interface with a separate project in another language for its javascript engine (webkit). But even with those caveats, it has 65k stars on [github](https://github.com/oven-sh/bun) which to me says that people are absolutely dying to get away from node.


## Final Setup
My setup is this (went the [`.mts` route](https://nodejs.org/docs/latest-v18.x/api/packages.html#packagejson-and-file-extensions)):

`./tsconfig.json`

```json
{
    "compilerOptions": {
        "esModuleInterop": true,
        "target": "ES2022",
        "lib": [
            "DOM",
            "ES2022",
        ],
        "baseUrl": ".",
        "allowJs": true,
        "skipLibCheck": true,
        "resolveJsonModule": false,
        "emitDecoratorMetadata": false,
        "module": "Node16",
        "moduleResolution": "Node16",
        "outDir": "./dist",
        "sourceMap": true,
        "noEmitOnError": true,
        "strict": true,
        "allowUnreachableCode": false,
        "alwaysStrict": true,
        "exactOptionalPropertyTypes": true,
        "noFallthroughCasesInSwitch": true,
        "noImplicitOverride": true,
        "noImplicitReturns": true,
        "noImplicitThis": true,
        "noPropertyAccessFromIndexSignature": true,
        "noUncheckedIndexedAccess": true,
        "noImplicitAny": true,
        // "noUnusedLocals": true,
        // "noUnusedParameters": true,

    },
    "exclude": [
        "**/node_modules",
        "**/dist",
        "**/build",
        "*.js",
    ],
    "plugins": [

    ],
    "env": {
    },
    "ts-node": {
        "esm": true,
        "swc": true,
    }
}
```



`./.vscode/launch.json` (debug configuration):

```json
        {
            "name": "ts-node",
            "type": "node",
            "request": "launch",
            "runtimeExecutable": "node",
            "runtimeArgs": [ 
                "--loader",
                "ts-node/esm",
                "--unhandled-rejections=strict",
                "--nolazy"
            ],
            "program": "${file}",
            "cwd": "${workspaceFolder}",
            "console": "integratedTerminal",
            "internalConsoleOptions": "neverOpen",
            "skipFiles": ["<node_internals>/**", "node_modules/**"]
        }
```

And my `./package.json`:

```json
{
    "name": "crawlee-test",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "build": "tsc --build",
        "clean": "tsc --build --clean",
        "start:dev": "ts-node",
        "start:watch": "nodemon --exec ts-node",
        "start:prod": "node dist/index.js",
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
    },
    "devDependencies": {
        "@swc/core": "^1.3.99",
        "@swc/helpers": "^0.5.3",
        "@types/node": "^20.10.0",
        "bookmarks-to-json": "^1.0.5",
        "nodemon": "^3.0.1",
        "ts-node": "^10.9.1",
        "typescript": "^5.3.2"
    }
}
```



[^1]: By modules I mean "modules" as in the commonly understood meaning of the word everywhere else: code that you pull in (or make available to) other code. In the JS ecosystem is very important to note that the word module *usually* means """""""es"""""" "modules", but depending on the context it can also refer to modules.

[^2]: See note 1.

[^3]: Here the word is used in the normal sense.