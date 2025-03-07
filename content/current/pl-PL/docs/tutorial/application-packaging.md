# Pakowanie Aplikacji

To mitigate [issues](https://github.com/joyent/node/issues/6960) around long path names on Windows, slightly speed up `require` and conceal your source code from cursory inspection, you can choose to package your app into an [asar](https://github.com/electron/asar) archive with little changes to your source code.

Most users will get this feature for free, since it's supported out of the box by [`electron-packager`](https://github.com/electron/electron-packager), [`electron-forge`](https://github.com/electron-userland/electron-forge), and [`electron-builder`](https://github.com/electron-userland/electron-builder). If you are not using any of these tools, read on.

## Generating `asar` Archives

An [asar](https://github.com/electron/asar) archive is a simple tar-like format that concatenates files into a single file. Electron can read arbitrary files from it without unpacking the whole file.

Steps to package your app into an `asar` archive:

### 1. Install the asar Utility

```sh
$ npm install -g asar
```

### 2. Package with `asar pack`

```sh
$ asar pack your-app app.asar
```

## Using `asar` Archives

In Electron there are two sets of APIs: Node APIs provided by Node.js and Web APIs provided by Chromium. Both APIs support reading files from `asar` archives.

### API Node

With special patches in Electron, Node APIs like `fs.readFile` and `require` treat `asar` archives as virtual directories, and the files in it as normal files in the filesystem.

For example, suppose we have an `example.asar` archive under `/path/to`:

```sh
$ asar list /path/to/example.asar
/app.js
/file.txt
/dir/module.js
/static/index.html
/static/main.css
/static/jquery.min.js
```

Read a file in the `asar` archive:

```javascript
const fs = require('fs')
fs.readFileSync('/path/to/example.asar/file.txt')
```

List all files under the root of the archive:

```javascript
const fs = require('fs')
fs.readdirSync('/path/to/example.asar')
```

Use a module from the archive:

```javascript
require('/path/to/example.asar/dir/module.js')
```

You can also display a web page in an `asar` archive with `BrowserWindow`:

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow()

win.loadURL('file:///path/to/example.asar/static/index.html')
```

### Web API

In a web page, files in an archive can be requested with the `file:` protocol. Like the Node API, `asar` archives are treated as directories.

For example, to get a file with `$.get`:

```html
<script>
let $ = require('./jquery.min.js')
$.get('file:///path/to/example.asar/file.txt', (data) => {
  console.log(data)
})
</script>
```

### Treating an `asar` Archive as a Normal File

For some cases like verifying the `asar` archive's checksum, we need to read the content of an `asar` archive as a file. For this purpose you can use the built-in `original-fs` module which provides original `fs` APIs without `asar` support:

```javascript
const originalFs = require('original-fs')
originalFs.readFileSync('/path/to/example.asar')
```

You can also set `process.noAsar` to `true` to disable the support for `asar` in the `fs` module:

```javascript
const fs = require('fs')
process.noAsar = true
fs.readFileSync('/path/to/example.asar')
```

## Limitations of the Node API

Even though we tried hard to make `asar` archives in the Node API work like directories as much as possible, there are still limitations due to the low-level nature of the Node API.

### Archives Are Read-only

The archives can not be modified so all Node APIs that can modify files will not work with `asar` archives.

### Working Directory Can Not Be Set to Directories in Archive

Though `asar` archives are treated as directories, there are no actual directories in the filesystem, so you can never set the working directory to directories in `asar` archives. Passing them as the `cwd` option of some APIs will also cause errors.

### Extra Unpacking on Some APIs

Most `fs` APIs can read a file or get a file's information from `asar` archives without unpacking, but for some APIs that rely on passing the real file path to underlying system calls, Electron will extract the needed file into a temporary file and pass the path of the temporary file to the APIs to make them work. This adds a little overhead for those APIs.

API, które wymagają dodatkowego rozpakowywania to:

* `child_process.execFile`
* `child_process.execFileSync`
* `fs.open`
* `fs.openSync`
* `process.dlopen` - Used by `require` on native modules

### Fake Stat Information of `fs.stat`

The `Stats` object returned by `fs.stat` and its friends on files in `asar` archives is generated by guessing, because those files do not exist on the filesystem. So you should not trust the `Stats` object except for getting file size and checking file type.

### Executing Binaries Inside `asar` Archive

There are Node APIs that can execute binaries like `child_process.exec`, `child_process.spawn` and `child_process.execFile`, but only `execFile` is supported to execute binaries inside `asar` archive.

This is because `exec` and `spawn` accept `command` instead of `file` as input, and `command`s are executed under shell. There is no reliable way to determine whether a command uses a file in asar archive, and even if we do, we can not be sure whether we can replace the path in command without side effects.

## Adding Unpacked Files to `asar` Archives

As stated above, some Node APIs will unpack the file to the filesystem when called. Apart from the performance issues, various anti-virus scanners might be triggered by this behavior.

As a workaround, you can leave various files unpacked using the `--unpack` option. In the following example, shared libraries of native Node.js modules will not be packed:

```sh
$ asar pack app app.asar --unpack *.node
```

After running the command, you will notice that a folder named `app.asar.unpacked` was created together with the `app.asar` file. It contains the unpacked files and should be shipped together with the `app.asar` archive.

