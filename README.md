# file-loader

Load file path values within an object and cache requests.

[![Travis](https://img.shields.io/travis/panosoft/file-loader.svg)](https://travis-ci.org/panosoft/file-loader)

A file-loader scans an objects values, identifies local and remote paths, loads the corresponding files, evaluates them if they are JavaScript files, and replaces the path values with the file contents or `module.exports` as applicable.

Each file loader has it's own cache that stores the most recently requested files and references that cache when a server replies with a `304` HTTP status code for a requested file.

## Installation

```sh
npm install @panosoft/file-loader
```

### create ( options )

Creates a new File Loader.

__Arguments__

- `options` - An object.

  - `max` -A number determining the maximum cache size. The cache size is calculated by summing the return values of the `length` function applied to each item in the cache. Defaults to `500`.
  - `length` - A function used to determine the length of each item in cache. It is iteratively called with each item in the cache. Defaults to giving each item a static length of `1`.

__Example__

```js
const FileLoader = require('@panosoft/file-loader');

const fileLoader = FileLoader.create({
  max: 500,
  length: (item) => 1
});
```

### load ( manifest , options )

Loads file contents from local or remote path values in a manifest object. It performs a shallow search for paths (i.e. only top level properties of an object are scanned).

JS files are read/requested, evaluated, and finally the path to each file is replaced with their module.exports. All other files are read/requested and have their paths replaced with their `utf8` encoded contents.

Returns a `Promise` that is fulfilled with one of two values:
- If `manifest` is a string path, the file contents or module.exports value is returned.
- If `manifest` is an object containing paths, a clone of the `manifest` object is returned with path values replaced with their respective file contents or module.exports.

__Arguments__

- `manifest` - A string path or an object containing string path values. Paths can be any one of the following:

   - fully qualified uri (i.e. `http://test.com/file.txt`)
   - absolute path (i.e. starting with `/`)
   - prefixed relative path (i.e. starting with `./` or `../`)


- `options` - An object.

  - `basePath` - A string used to determine the base path used to resolve relative paths. Defaults to the process' current working directory.
  - `dirname` - A string determining the directory used to set `__dirname` for loaded modules. For remote files: defaults to the current working directory of the process. For local files: defaults to the dirname of the file being loaded.

__Example__

```js
const co = require('co');

co(function * () {
  var obj = {
    file: '/path/to/file',
    module: 'http://path/to/module',
    string: 'a',
    fn: () => {}
  };
  const loadedObj = yield fileLoader.load(obj);
  console.log(loadedObj);
  /**
      {
        file: 'utf8 encoded file contents ...',
        module: module.exports,
        string: 'a',
        fn: () => {}
      }
   */
});
```
