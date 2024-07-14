---
date: 2024-07-13T23:00:00Z
title: JavaScript Packaging
url: /js-packaging
---

Publishing a JavaScript library requires creating an artifact (a .tar.gz file, aka tarball) that will be downloaded when your users install the package, say with `npm install your-package`. Call the directory in a git repo the "source package" and the tarball artifact the "distribution package." After creating this artifact you upload it to a package registry, typically npm. This artifact has requirements like there being a file called package.json at the root with metadata including the name of the package and its version number.

Why wouldn't the distribution package be identical to the source package? JavaScript is nominally not a compiled language!

1. Some information isn't needed in the distribution package, e.g. config files.
1. The code may be written in a language not consumable by targeted platforms. Typically TypeScript files needs to be compiled to JavaScript files (and source map files, and types files, and types declaration files).
1. For the code to be consumable by every tool you target (older versions of Node.js, Bun, Deno; browser script tags; bundlers like Webpack, Vite, Rollup; and other tools like TypeScript, tsx, vitest, and jest --- and for each of these, two formats: ESM and CJS) you often need multiple copies of the code.

   These target platforms may have different module resolution algorithms, different ways of finding the right code when you `import { something } from "your-package";` so you need to make sure the code is where these tools will look for it. Naturally there are tools for producing these various layouts of code. My new favorite is [tshy](https://github.com/isaacs/tshy).[^tshy]

Additionaly some filesystem contents not present in the repo like the node_modules directory are also typically also not included in the distribution package.

Many [simple](https://github.com/thlorenz/find-parent-dir) JavaScript packages either write their code in a way that requires no compilation or commit the build artifacts to source control so there are no differences between the source and distribution packages, or so their distribution packages are strict subsets of their source packages.

Technically you can produce this distribution artifact any way you like: your source code could be a Python script that generates a tarball with the proper file layout!

But there is a strong ecosystem convention that a source package contain a package.json file at its root with metadata used for all kinds of things: what dependencies to install, the package name, and configuration for tools. Most of this metadata is for npm or other packages managers, but other tools rely on this metadata or store their own configuration here.
There are some weaker ecosystem conventions like a `dist/` directory which contains git-ignored build artifacts and a `src/` directory which contains the source code that will be built.

Since tools like test runners and linters and npm commands like `npm link` have their own requirements for file layout that overlap with the distribution package requirements, source and distribution packages partially overlap.

<style>
svg {
  width: 100%;
}
</style>
<svg xmlns="http://www.w3.org/2000/svg" viewBox="10 65 380 310">
  <style>
    .circle { fill-opacity: 0.3; stroke-width: 2; }
    text { font-family: Arial, sans-serif; font-size: 14px; text-anchor: middle; }
    .filename { font-size: 12px; fill: #333; }
  </style>
  <!-- Filesystem Circle (largest) -->
  <ellipse cx="200" cy="220" rx="180" ry="150" class="circle" fill="#4ECDC4" stroke="#4ECDC4" />
  
  <!-- Source Control Circle -->
  <circle cx="140" cy="235" r="110" class="circle" fill="#FF6B6B" stroke="#FF6B6B" />
  <!-- Distribution Package Circle -->
  <circle cx="260" cy="235" r="110" class="circle" fill="#FFA500" stroke="#FFA500" />
  <!-- Labels -->
  <text x="125" y="170">Source control</text>
  <text x="200" y="90">Filesystem</text>
  <text x="265" y="170">Distribution package</text>
  <!-- File names -->
  <!-- In all three circles -->
  <text x="200" y="200" class="filename">package.json</text>
  <text x="200" y="220" class="filename">index.ts</text>
  <text x="200" y="240" class="filename">LICENSE</text>
  <text x="200" y="260" class="filename">tsconfig.json</text>
  <!-- In filesystem only (moved up more) -->
  <text x="200" y="110" class="filename">node_modules/</text>
  <text x="200" y="125" class="filename">.git/</text>
  <!-- In distribution package and filesystem (moved right) -->
  <text x="310" y="200" class="filename">dist/index.js</text>
  <text x="310" y="220" class="filename">dist/index.js.map</text>
  <text x="310" y="240" class="filename">dist/index.d.ts</text>
  <text x="310" y="260" class="filename">dist/index.d.ts.map</text>
  <!-- In source control and filesystem -->
  <text x="100" y="200" class="filename">index.test.ts</text>
  <text x="100" y="220" class="filename">.gitignore</text>
  <text x="100" y="240" class="filename">.prettierrc</text>
</svg>

# Multiple entry points and old resolution algorithms

One persnickety group of platforms you may choose to target use old module resolution algorithms that don't understand the `"exports"` field of a package.json. `"exports"` specifies the location of code based on the path imported from your library.
When this mapping is trivial (when there's only one code file to load) this isn't necessary but it is when you have multiple entry points.

Multiple entry points are a neat idea: install a single package like `convex` to be able to import multiple pieces of code. `import { query } from "convex/server";` comes from a completely separate codebase (importing different dependencies and with different platform requirements) as `import { ConvexClient } from "convex/browser";` but both imports are enabled by `npm install convex`. Without relying on bundler code extraction you can be certain that the portion of the codebase not imported from will not end up in the bundled code. For targets that don't tree shake like Node.js, you still know code from a different entry point than the one you imported will absolutely not run.

# Are you going to get to the story?

I didn't know much about this until about two years ago when I started to maintain npm packages for [Convex](https://www.convex.dev/), primarily the [convex](https://www.npmjs.com/package/convex) package. ([source](https://github.com/get-convex/convex-js))

We decided to support multiple entry points for platforms like Metro and older versions of Jest and Vitest that use something like Node.js v10 resolution, ignoring the entry point mapping `"export"` field of the package.json. To imitate support for multiple entry points it's necessary for the built JavaScript files to be located relative to the package root according to their entry point name: `convex/server` will refer to

- a JavaScript file in the package root called server.js,
- a file called server/index.js,
- or a file called server/package.json that points to the files.

I took the last option, which I discovered thanks to Andrew Branch's tweets and [repo](https://github.com/andrewbranch/example-subpath-exports-ts-compat) demonstrating this strategy.

There are three things we could do with these ugly stub directories like server/package.json:

1. commit them to the git repo, everyone has to deal with them
2. generate them during a build process and adding them to the .gitignore file
3. just stick them in the tarball artifact so no one ever sees them

For the convex npm package I went with 1), so the convex npm package has directories called `browser`, `values`, `server` etc. [in version control](https://github.com/get-convex/convex-js) as well [in the distribution package](https://unpkg.com/browse/convex@1.13.0/). These files being present during step 2) means tools like `npm link` or pnpm monorepos that install dependencies by linking directly to packages instead of installing the tarball also benefit from the presence of these files. In our monorepo we have tools that use these legacy module resolution strategies we have to appease. It's possible to force these package to install the tarball instead in a pnpm monorepo by installing `file:../path/to/package?pack` instead of the normal path, so option 3) is actually viable for us.

And it's not much more work given that we already have a script that
unpacks the tarball to a temporary directory, modifies it, and repacks it. ([script](https://github.com/get-convex/convex-js/blob/main/scripts/postpack.mjs))
This is a bit more hidden but as long as the tarball itself is sufficiently tested option 3 is viable.

# Venn diagrams shmem shmiagrams

There's another option: what if the source package and the distribution package, either before or after being packed into a tarball, were in completely different directories? We could create a completely synthetic package! This isn't far from the convention of using a dist/ directory today which contains compiled copies of the code in the src/ directory: we just need to create copy a modified package.json in there too. But now there are two different packages for tools to reason about.

This seems like the most direct solution to the problem: given two sets of requirements (source distribution for code authors and package distribution for npm), solve the two problems directly instead of punning between them, instead of forcing them to overlap.

If the original package.json points to files in the dist folder as usual then the package could function normally, only requiring persnickety `"exports"`-ignoring tools to either install this synthetic package directory or the packed tarball built from it.

I've never traced through to figure out quite how the [Next.js project](https://github.com/vercel/next.js) produces its [distribution package](https://www.npmjs.com/package/next) but since I couldn't follow assumed something like this was happening, and I've envied this setup ever since.

# What's the big idea?

The investigation into this technique was all done by a colleague who asked for feedback on it, but while giving it I found I was lacking a good description of the problem. Now you too can read it!

I worry about breaking from ecosystem conventions because I don't want an unusual build process or directory structure to break tools I use but don't fully understand or make it difficult to use new tools in the future.

I'm the type to wait a full year before upgrading my laptop to the next MacOS release because I want to lean on others' efforts to update Homebrew.
I don't want my setup to stick out; I want to be able to Google for solutions that early adopters posted 6 months ago. Another valid approach is to fuck around and find out; I've just gotten tired of finding out.

The trick to hacking underspecified tooling processes is accounting for all the third party tools that might use metadata in package.json for anything. For instance I worry about communicating the correspondence of the source package and the distribution package directories to a build tool trying to walk dependencies to build a minimal build tree.

So I'm more comfortable following conventions here, but I like the synthetic package idea.

[^tshy]:
    I recently [requested support](https://github.com/isaacs/tshy/issues/85) for the stub directory technique for supporting Node 10 module resolution of multiple entry points in [tshy](https://github.com/isaacs/tshy/)
    and the maintainer had reasons not to implement it: 1. tshy is a tool for TypeScript HYbridization, not a general purpose solves-everything package publishing. Use it for generating multiple versions of a package. And 2. in addition to the above, do you really need to support these platforms? Our customers say we do, so for now we'll take on this pain. I'm sympathetic to the point of view that this just delays adoption of modern ways of doing things but I also hear from developers that are sad when something doesn't work and then happy when it does; it's hard for me to prioritize the ecosystem over the pain of the developer in front of me.
