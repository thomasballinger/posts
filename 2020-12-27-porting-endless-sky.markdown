---
date: 2020-12-27T23:00:00Z
title: Porting Endless Sky
tags: endless-sky
url: /porting-endless-sky
---

<style>
img {
  width: 100%;
}
</style>

I spent a couple weekends in [2018](https://twitter.com/ballingt/status/1021620043170246656) and 2019 trying and giving up at porting [Endless Sky](http://endless-sky.github.io/), an open source 2D space exploration game in the vein of Escape Velocity, to the web. I'd start with writing Hello World in C using [Emscripten](https://emscripten.org/) to compile to WebAssembly and then try to draw the rest of the owl: porting a whole application.

This year I decided to ask for help. I was fortunate to make contact with [janisozaur](https://github.com/janisozaur), who already had experience porting [OpenRCT2](https://openrct2.org/) to the web. janisozaur did basically the entire port and I added some JavaScript integration for things like saving games.

![](/assets/endlessskyweb.png)

You can see the final proof of concept at [play-endless-sky.com](https://play-endless-sky.com/). I've outlined the process below. Thanks so much to janisozaur, who made this project possible and taught me a lot about graphics programming and linkers and compiler flags.

---

When I asked for help, I could compile the code on a Linux VM and was [starting to be able to compile the source code using the Emscripten](https://gist.github.com/thomasballinger/5bffacaf1c6d168fa265ada2000ff27d). I was initially stuck on dependencies: Endless Sky used libraries like SDL2 that were already well-supported by Emscripten, but also [libjpeg-turbo](https://libjpeg-turbo.org/) and libmad which it seemed I needed to compile myself. You can read [his commits directly on GitHub](https://github.com/janisozaur/endless-sky/commits/es-wasm?after=f9a0994bdb0125d89bbdac0352743a9d5a6bc3dd+34&branch=es-wasm) or check out [the branch I'm cleaning up](https://github.com/thomasballinger/endless-sky/pull/1). 

1. Use RGBA color encoding instead of BGRA. ([upstream PR](https://github.com/endless-sky/endless-sky/pull/5153)) [libjpeg-turbo](https://libjpeg-turbo.org/) supports BGRA but the more standard [libjpeg](http://libjpeg.sourceforge.net/) does not. libjpeg is easy to use with Emscripten, libjpeg-turbo isn't. I could have figured this one out, I sort of did when I first made progress; but I didn't actually fix the issue, I just replaced calls using BGRA to use RGBA and expected to get funny looking images.

2. Add a swizzling fallback in shader code. (~~[upstream PR](https://github.com/endless-sky/endless-sky/pull/5152)~~) Swizzling (swapping RGB channels to produce 8 different variants of an image/texture from one) previously required an OpenGL extension not available on all machines, and adding equivalent code to a shader removed this requirement.

3. Make shaders compliant with [OpenGL ES](https://en.wikipedia.org/wiki/OpenGL_ES). ([upstream PR](https://github.com/endless-sky/endless-sky/pull/5588)) OpenGL ES is OpenGL-lite spec that WebGL and embedded devices use. By using the subset of features and syntax supported by both OpenGL 3 and OpenGL ES 3, the same shader code could run on the desktop and web. This was also the motivation for steps 1 and 2.

4. Add a compiler flag for disabling music, ([upstream PR](https://github.com/endless-sky/endless-sky/pull/5591)) and therefore removing need for the libmad mp3-decoding library. I didn't realize Endless Sky had music, but it turns out that there's one mp3 that loops while the player is docked with a space station.

5. Add a compiler flag for disabling threads. ([PR preview](https://github.com/endless-sky/endless-sky/compare/master...thomasballinger:no-threads?expand=1)) This requires a few things:
    * Add `#ifdefs` around mutexes, condition variables and thread objects
    * Load sounds on the main thread instead of in a background thread
    * Run engine calculations on the main thread instead of in a background thread
    * Transform multithreaded sprite queue control flow to run on the main thread

6. Add an emscripten build ([commit](https://github.com/thomasballinger/endless-sky/pull/1/commits/a5ff98f9b9ec4219484ba21858e3d565a0226064) - todo: change commit author)
    * Transform the main game loop from a forever loop to something that could be called periodically, scheduled in the browser by `window.requestAnimationFrame`
    * Add a build mode for the web build and a bunch of compiler flags to link libraries, set memory requirements, and make data files available in the browser

* Patch emscripten to allow single quotes in preloaded data directory names. (~~[Emscripten PR](https://github.com/emscripten-core/emscripten/pull/13147)~~) Ka'het is a great name for an alien civilization but was could not be simply surrounded by single quotes to be inserted into JavaScript code. Contributing to (a fairly standalong portion of) Emscripten was pretty simple, it seems like a very welcoming project.

* Write an HTML wrapper for the magic to happen in! I helped with this part.
  * loading screen
  * make a big canvas for the game to be played on
  * store saved game in browser cache
  * allow uploading of existing saved games and downloading save games after they have been played
  * support for dynamically loading plugins from a central plugin list

* Cache data too large for automatic caches with IndexedDB, only expiring the cache when wasm.data has changed (I helped with this too)

* Add support for webp, a more efficient lossless image format that gave a ~30% total memory savings! This also decreased the download size, but the in-memory savings were the important part.
  * webp stores multiple frames of an animation in a single file
  * webp works on desktop too, but for now this branch does the png -> webp conversion during the build process because changing the image format used seems like something that would affect content creators a lot

Whew!

## Lessons

Reflecting on what I learned... I know C++ a little bit better now? I learned a lot about Emscripten, janisozaur's taught me enough about shaders and openGL to basically understand why he made the changes he did, and I saw practical solutions to dependency management / linking / compiling issues I had been stumped by before.
