---
title: Custom Opencv WASM With Any Frontend Libray (ESM Imports)
date: 2024-08-31T20:19:33+10:00
draft: false
images:
  - ./images/default-cover.png
slug: Custom-opencv-with-any-frontend-libray
description: Import opencv like any other module. Very easy.
weight: 4
---
<span class="summary">If you especially had issues with TechStark's <a href="https://github.com/TechStark/opencv-js">opencv-js repo</a> read on. Also, this works great with vite.</span>

For convenience here's a list of useful links on the subject:
- [Official section](https://docs.opencv.org/4.x/df/df7/tutorial_js_table_of_contents_setup.html) on the opencv docs website
	- The docs in general are out of date and wrong in many places. Also there is **zero** documentation for the javascript bindings or even official types to guide you. It's left up to you to figure out what works and what doesn't with trial and error.
- [Official build docs](https://docs.opencv.org/4.x/d4/da1/tutorial_js_setup.html)
	- May be useful in some places but I recommend following my instructions below.
- [Lambda labs build instructions](https://lambda-it.ch/blog/build-opencv-js): 
	- This is out of date but was still useful to fill in some gaps.

## 1. Build process (works on every OS)
[Install docker](https://docs.docker.com/engine/install/) then run the following commands:

### Option A: One line docker build (quick):
Setup (**All OSs**). Requires NPM be installed. Uses [ghex](https://bn-l.github.io/GithubExtractor/cli.html) to make this fast as it's a big repo:
```bash
npx ghex opencv/opencv -d opencv
cd opencv
```

Build (**MacOS / Linux**):
```bash
docker run --rm -v $(pwd):/src -u $(id -u):$(id -g) emscripten/emsdk emcmake python3 ./platforms/js/build_js.py --clean_build_dir --disable_single_file --build_wasm --build_flags=" -s MODULARIZE=1 -s EXPORT_ES6=1 -s IGNORE_MISSING_MAIN=1 -s MAXIMUM_MEMORY=2147483648 -s ENVIRONMENT=web " cust_build
```

Build (**Windows, powershell)**:
```powershell
docker run --rm --workdir /src -v "$(get-location):/src" "emscripten/emsdk" emcmake python3 ./platforms/js/build_js.py --clean_build_dir --disable_single_file --build_wasm --build_flags=" -s MODULARIZE=1 -s EXPORT_ES6=1 -s IGNORE_MISSING_MAIN=1 -s MAXIMUM_MEMORY=2147483648 -s ENVIRONMENT=web " cust_build
```

In the folder `build_js/bin/opencv.js`  you will see the following files (the rest are unimportant). Save just these. These are the files we want:

```bash
opencv_js.js
opencv_js.wasm
```

### Option B: Minimal 3 MB build (advanced):
This option allows more control over the size of the bundle. To start:
1. mount a local folder into the docker container (or use a vscode devcontainer--I used the basic ubuntu image) to save the build to.
2. Run the following commands inside the container:

Prerequisites:
```bash
sudo apt install git cmake python
```

Set up emscripten's sdk:
```bash
mkdir customopencv
cd customopencv/
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk/
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
```

Opencv (using [nano](https://linuxize.com/post/how-to-use-nano-text-editor/) to edit the file):
```bash
git clone https://github.com/opencv/opencv.git
nano opencv/platforms/js/build_js.py
```

In this file (here's a [link to its source](https://github.com/opencv/opencv/blob/4d665419992dda6e40364f741ae4765176b64bb0/platforms/js/build_js.py)) find the `def get_cmake_cmd(self):` function. The first part of it looks like this (comments removed):

```python
def get_cmake_cmd(self):
        cmd = [
            "cmake",
            "-DPYTHON_DEFAULT_EXECUTABLE=%s" % sys.executable,
               "-DENABLE_PIC=FALSE",
               "-DCMAKE_BUILD_TYPE=Release",
               "-DCPU_BASELINE=''",
               
	   # .......... continues (too big to paste it all here)
```

To get to 3 MB I changed the following to `=OFF`: 

```python
               "-DBUILD_opencv_calib3d=OFF",
               "-DBUILD_opencv_dnn=OFF",
               "-DBUILD_opencv_features2d=OFF",
               "-DBUILD_opencv_flann=OFF", 
               "-DBUILD_opencv_photo=OFF",
```
I found the [java based docs](https://docs.opencv.org/4.x/javadoc/index-all.html) to have the best UI and descriptions for what each of these do. E.g. "[flan](https://docs.opencv.org/4.x/javadoc/org/opencv/features2d/FlannBasedMatcher.html)" (whatever that is).

Then I ran this command in the `customopencv` created before:

```bash
emcmake python ./opencv/platforms/js/build_js.py --clean_build_dir --disable_single_file --build_wasm --build_flags=" -s MODULARIZE=1 -s EXPORT_ES6=1 -s IGNORE_MISSING_MAIN=1 -s MAXIMUM_MEMORY=2147483648 -s ENVIRONMENT=web " ${PWD}/cust_build
```

In the folder `build_js/bin/opencv.js` you will see the following files (the rest are unimportant). Save just these. These are the files we want:

```bash
opencv_js.js
opencv_js.wasm
```

## 2. Use in javascript / typescript
TechStark has a [repo](https://github.com/TechStark/opencv-js) that provides opencv in a way that I believe can be imported using ES import syntax (e.g.: `import cv from ./opencv.js`). I haven't tested it because it requires applying an unnecessary [patch](https://github.com/TechStark/opencv-js/blob/6a539c08d2ca78dc544dc6fee670af38740c4237/dist/opencv.js.patch) to the js output from the build and setting up some manual way of waiting for the wasm to be ready to run. Emscripten has all of this built in and **if you followed the steps above using opencv in javascript / typescript is as simple as**: 

1. Adding the files `opencv_js.js` and `opencv_js.wasm` to your project (I added these to the $lib folder in sveltekit but it should work with any frontend framework):
2. Importing from the javascript directly:
```typescript
import cvLoader from "./opencv_js.js";
import type CV from "@techstark/opencv-js";
const cv: CV = await cvLoader();
```

And that's it. `cvLoader` resolves with an instance of opencv when the wasm has been downloaded and initialised so opencv is ready to use. 

I am using just the types from the [@techstark/opencv-js](https://www.npmjs.com/package/@techstark/opencv-js) package and with the `import type` statement nothing from that module will end up in my final bundle (I'm using vite via sveltekit).

