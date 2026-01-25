+++
title = "Using Electron Node to run Drizzle Kit"
date = 2025-11-11
updated = 2025-11-11
type = "post"
description = "Notes on integrating Drizzle Kit in Electron"
in_search_index = true
[taxonomies]
TIL-Tags = ["electron", "drizzle-kit"]
+++

I was trying to create a desktop app using Electron with sqlite as the database and use Drizzle Kit to manage my database schema. I was using pnpm as my package manager and had to switch between running the electron app and running the drizzle scripts.


```
"scripts": {
    "start": "pnpm run rebuild:electron && electron-forge start",
    "package": "pnpm run build:python && electron-forge package",
    "make": "pnpm run build:python && electron-forge make",
    "publish": "pnpm run build:python && electron-forge publish",
    "lint": "eslint --ext .ts,.tsx .",
    "build:python": "python3 python/build-python-executable.py",
    "build:python:install-deps": "python3 -m pip install -r python/build-requirements.txt",
    "clean:python": "rm -rf python/build/* python/dist/*",
    "rebuild:electron": "npx @electron/rebuild",
    "rebuild:node": "npm rebuild better-sqlite3", 
    "about:rebuild:node": "echo 'npm is used instead of pnpm as pnpm 10+ has breaking changes for rebuild command'",
    "db:studio": "pnpm run rebuild:node && npx drizzle-kit studio",
    "db:push": "pnpm run rebuild:node && npx drizzle-kit push",
    "db:generate": "pnpm run rebuild:node && npx drizzle-kit generate",
    "db:migrate": "pnpm run rebuild:node && npx drizzle-kit migrate"
  },
```

I had all this nonsense to switch between running the electron app and running the drizzle scripts. This was to solve the node version mismatch issue.

Running `npx drizzle-kit studio` would throw this error

```
No config path provided, using default 'drizzle.config.ts'
Reading config file '.../drizzle.config.ts'
Error: The module '.../node_modules/better-sqlite3/build/Release/better_sqlite3.node'
was compiled against a different Node.js version using
NODE_MODULE_VERSION 136. This version of Node.js requires
NODE_MODULE_VERSION 127. Please try re-compiling or re-installing
the module (for instance, using `npm rebuild` or `npm install`).
    at Object..node (node:internal/modules/cjs/loader:1921:18)
    at Module.load (node:internal/modules/cjs/loader:1465:32)
    at Function._load (node:internal/modules/cjs/loader:1282:12)
    at TracingChannel.traceSync (node:diagnostics_channel:322:14)
    at wrapModuleLoad (node:internal/modules/cjs/loader:235:24)
    at Module.require (node:internal/modules/cjs/loader:1487:12)
    at require (node:internal/modules/helpers:135:16)
    at bindings (.../node_modules/bindings/bindings.js:112:48)
    at new Database (.../node_modules/better-sqlite3/lib/database.js:48:64)
    at connectToSQLite (.../node_modules/drizzle-kit/bin.cjs:81281:24) {
  code: 'ERR_DLOPEN_FAILED'
}
```

Turns out electron comes with its own node. You can set env variables to use the right node version for specific commands. 


```
"scripts": {
    "start": "pnpm run rebuild && electron-forge start",
    "package": "pnpm run rebuild && electron-forge package",
    "make": "pnpm run rebuild && electron-forge make",
    "publish": "pnpm run rebuild && electron-forge publish",
    "lint": "eslint --ext .ts,.tsx .",
    "build:python": "python3 python/build-python-executable.py",
    "build:python:install-deps": "python3 -m pip install -r python/build-requirements.txt",
    "clean:python": "rm -rf python/build/* python/dist/*",
    "rebuild": "npx @electron/rebuild",
    "db:studio": "ELECTRON_RUN_AS_NODE=1 pnpm exec electron ./node_modules/drizzle-kit/bin.cjs studio",
    "db:push": "ELECTRON_RUN_AS_NODE=1 pnpm exec electron ./node_modules/drizzle-kit/bin.cjs push",
    "db:generate": "ELECTRON_RUN_AS_NODE=1 pnpm exec electron ./node_modules/drizzle-kit/bin.cjs generate",
    "db:migrate": "ELECTRON_RUN_AS_NODE=1 pnpm exec electron ./node_modules/drizzle-kit/bin.cjs migrate"
  },
```

reference: [https://github.com/WiseLibs/better-sqlite3/issues/1171#issuecomment-3145948144](https://github.com/WiseLibs/better-sqlite3/issues/1171#issuecomment-3145948144)