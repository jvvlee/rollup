---
title: Command Line Interface
---

Rollup should typically be used from the command line. You can provide an  optional Rollup configuration file to simplify command line usage and enable advanced Rollup functionality.

### Configuration Files

Rollup configuration files are optional, but they are powerful and convenient and thus **recommended**.

A config file is an ES module that exports a default object with the desired options. Typically, it is called `rollup.config.js` and sits in the root directory of your project.

Also you can use CJS modules syntax for the config file.

```javascript
module.exports = {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

It may be pertinent if you want to use the config file not only from the command line, but also from your custom scripts programmatically.

Consult the [big list of options](guide/en#big-list-of-options) for details on each option you can include in your config file.

```javascript
// rollup.config.js

export default { // can be an array (for multiple inputs)
  // core input options
  external,
  input, // required
  plugins,

  // advanced input options
  cache,
  inlineDynamicImports,
  manualChunks,
  onwarn,
  preserveModules,

  // danger zone
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // experimental
  chunkGroupingSize,
  experimentalCacheExpiry,
  experimentalOptimizeChunks,
  experimentalTopLevelAwait,
  perf,

  output: { // required (can be an array, for multiple outputs)
    // core output options
    dir,
    file,
    format, // required
    globals,
    name,

    // advanced output options
    assetFileNames,
    banner,
    chunkFileNames,
    compact,
    entryFileNames,
    extend,
    footer,
    interop,
    intro,
    outro,
    paths,
    sourcemap,
    sourcemapExcludeSources,
    sourcemapFile,
    sourcemapPathTransform,

    // danger zone
    amd,
    esModule,
    exports,
    freeze,
    indent,
    namespaceToStringTag,
    noConflict,
    preferConst,
    strict
  },

  watch: {
    chokidar,
    clearScreen,
    exclude,
    include
  }
};
```

You can export an **array** from your config file to build bundles from several different unrelated inputs at once, even in watch mode. To build different bundles with the same input, you supply an array of output options for each input:

```javascript
// rollup.config.js (building more than one bundle)

export default [{
  input: 'main-a.js',
  output: {
    file: 'dist/bundle-a.js',
    format: 'cjs'
  }
}, {
  input: 'main-b.js',
  output: [
    {
      file: 'dist/bundle-b1.js',
      format: 'cjs'
    },
    {
      file: 'dist/bundle-b2.js',
      format: 'esm'
    }
  ]
}];
```

If you want to create your config asynchronously, Rollup can also handle a `Promise` which resolves to an object or an array.

```javascript
// rollup.config.js
import fetch from 'node-fetch';
export default fetch('/some-remote-service-or-file-which-returns-actual-config');
```

Similarly, you can do this as well:

```javascript
// rollup.config.js (Promise resolving an array)
export default Promise.all([
  fetch('get-config-1'),
  fetch('get-config-2')
])
```

You *must* use a configuration file in order to do any of the following:

- bundle one project into multiple output files
- use Rollup plugins, such as [rollup-plugin-node-resolve](https://github.com/rollup/rollup-plugin-node-resolve) and [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs) which let you load CommonJS modules from the Node.js ecosystem

To use Rollup with a configuration file, pass the `--config` or `-c` flags.

```console
# use Rollup with a rollup.config.js file
$ rollup --config

# alternatively, specify a custom config file location
$ rollup --config my.config.js
```

You can also export a function that returns any of the above configuration formats. This function will be passed the current command line arguments so that you can dynamically adapt your configuration to respect e.g. `--silent`. You can even define your own command line options if you prefix them with `config`:

```javascript
// rollup.config.js
import defaultConfig from './rollup.default.config.js';
import debugConfig from './rollup.debug.config.js';

export default commandLineArgs => {
  if (commandLineArgs.configDebug === true) {
    return debugConfig;
  }
  return defaultConfig;
}
```

If you now run `rollup --config --configDebug`, the debug configuration will be used.


### Command line flags

Many options have command line equivalents. Any arguments passed here will override the config file, if you're using one. See the [big list of options](guide/en#big-list-of-options) for details.

```text
-c, --config                Use this config file (if argument is used but value
                              is unspecified, defaults to rollup.config.js)
-d, --dir                   Directory for chunks (if absent, prints to stdout)
-e, --external              Comma-separate list of module IDs to exclude
-f, --format <format>       Type of output (amd, cjs, esm, iife, umd)
-g, --globals               Comma-separate list of `moduleID:Global` pairs
-i, --input                 Input (alternative to <entry file>)
-m, --sourcemap             Generate sourcemap (`-m inline` for inline map)
-n, --name                  Name for UMD export
-o, --file <output>         Single output file (if absent, prints to stdout)
--amd.id <id>               ID for AMD module (default is anonymous)
--amd.define <name>         Function to use in place of `define`
--assetFileNames <pattern>  Name pattern for emitted assets
--banner <text>             Code to insert at top of bundle (outside wrapper)
--chunkFileNames <pattern>  Name pattern for emitted secondary chunks
--compact                   Minify wrapper code
--context <variable>        Specify top-level `this` value
--entryFileNames <pattern>  Name pattern for emitted entry chunks
--no-esModule               Do not add __esModule property
--exports <mode>            Specify export mode (auto, default, named, none)
--extend                    Extend global variable defined by --name
--footer <text>             Code to insert at end of bundle (outside wrapper)
--no-freeze                 Do not freeze namespace objects
--no-indent                 Don't indent result
--no-interop                Do not include interop block
--inlineDynamicImports      Create single bundle when using dynamic imports
--intro <text>              Code to insert at top of bundle (inside wrapper)
--namespaceToStringTag      Create proper `.toString` methods for namespaces
--noConflict                Generate a noConflict method for UMD globals
--no-strict                 Don't emit `"use strict";` in the generated modules
--outro <text>              Code to insert at end of bundle (inside wrapper)
--preferConst               Use `const` instead of `var` for exports
--preserveModules           Preserve module structure
--preserveSymlinks          Do not follow symlinks when resolving files
--shimMissingExports        Create shim variables for missing exports
--sourcemapExcludeSources   Do not include source code in source maps
--sourcemapFile <file>      Specify bundle position for source maps
--no-treeshake              Disable tree-shaking optimisations
--no-treeshake.propertyReadSideEffects Ignore property access side-effects
--treeshake.pureExternalModules        Assume side-effect free externals
```

In addition, the following arguments can be used:

#### `-h`/`--help`

Print the help document.

#### `-v`/`--version`

Print the installed version number.

#### `-w`/`--watch`

Rebuild the bundle when its source files change on disk.

#### `--silent`

Don't print warnings to the console.

#### `--environment <values>`

Pass additional settings to the config file via `process.ENV`.

```sh
rollup -c --environment INCLUDE_DEPS,BUILD:production
```

will set `process.env.INCLUDE_DEPS === 'true'` and `process.env.BUILD === 'production'`. You can use this option several times. In that case, subsequently set variables will overwrite previous definitions. This enables you for instance to overwrite environment variables in `package.json` scripts:

```json
// in package.json
{
  "scripts": {
    "build": "rollup -c --environment INCLUDE_DEPS,BUILD:production"
  }
}
```

If you call this script via:

```console
npm run build -- --environment BUILD:development
```

then the config file will receive `process.env.INCLUDE_DEPS === 'true'` and `process.env.BUILD === 'development'`.