# Build Instructions

请遵循以下指南来构建Electron。

## 平台要求

各个平台所对应的构建要求如下：

  * [macOS](build-instructions-macos.md#prerequisites)
  * [Linux](build-instructions-linux.md#prerequisites)
  * [Windows](build-instructions-windows.md#prerequisites)

## 前置知识

此外，你还需要安装[`depot_tools`](http://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)，这是一个用于获取Chromium，及其相关依赖工具。

另外，如果使用Windows系统, 你需要设置环境变量`DEPOT_TOOLS_WIN_TOOLCHAIN=0`。 依次打开 `Control Panel` → `System and
Security` → `System` → `Advanced system settings` ，然后添加系统变量 `DEPOT_TOOLS_WIN_TOOLCHAIN` ，并设置默认值为 `0`.  这将促使`depot_tools` 使用本地已安装的Visual Studio(默认状态下，`depot_tools`将会下载一个只有谷歌内部员工有权限使用的内部版本)。

## Cached builds (optional step)

### GIT\_CACHE\_PATH

If you plan on building Electron more than once, adding a git cache will speed up subsequent calls to `gclient`. To do this, set a `GIT_CACHE_PATH` environment variable:

```sh
$ export GIT_CACHE_PATH="${HOME}/.git_cache"
$ mkdir -p "${GIT_CACHE_PATH}"
# This will use about 16G.
```

> **NOTE**: the git cache will set the `origin` of the `src/electron` repository to point to the local cache, instead of the upstream git repository. This is undesirable when running `git push`—you probably want to push to github, not your local cache. To fix this, from the `src/electron` directory, run:

```sh
$ git remote set-url origin https://github.com/electron/electron
```

### sccache

Thousands of files must be compiled to build Chromium and Electron. You can avoid much of the wait by reusing Electron CI's build output via [sccache](https://github.com/mozilla/sccache). This requires some optional steps (listed below) and these two environment variables:

```sh
export SCCACHE_BUCKET="electronjs-sccache-ci"
export SCCACHE_TWO_TIER=true
```

## Getting the code

```sh
$ mkdir electron-gn && cd electron-gn
$ gclient config \
    --name "src/electron" \
    --unmanaged \
    https://github.com/electron/electron
$ gclient sync --with_branch_heads --with_tags
# This will take a while, go get a coffee.
```

> Instead of `https://github.com/electron/electron`, you can use your own fork here (something like `https://github.com/<username>/electron`).

#### A note on pulling/pushing

If you intend to `git pull` or `git push` from the official `electron` repository in the future, you now need to update the respective folder's origin URLs.

```sh
$ cd src/electron
$ git remote remove origin
$ git remote add origin https://github.com/electron/electron
$ git branch --set-upstream-to=origin/master
$ cd -
```

:memo: `gclient` works by checking a file called `DEPS` inside the `src/electron` folder for dependencies (like Chromium or Node.js). Running `gclient sync -f` ensures that all dependencies required to build Electron match that file.

So, in order to pull, you'd run the following commands:
```sh
$ cd src/electron
$ git pull
$ gclient sync -f
```

## 构建

```sh
$ cd src
$ export CHROMIUM_BUILDTOOLS_PATH=`pwd`/buildtools
# this next line is needed only if building with sccache
$ export GN_EXTRA_ARGS="${GN_EXTRA_ARGS} cc_wrapper=\"${PWD}/electron/external_binaries/sccache\""
$ gn gen out/Debug --args="import(\"//electron/build/args/debug.gn\") $GN_EXTRA_ARGS"
```

Or on Windows (without the optional argument):
```sh
$ cd src
$ set CHROMIUM_BUILDTOOLS_PATH=%cd%\buildtools
$ gn gen out/Debug --args="import(\"//electron/build/args/debug.gn\")"
```

This will generate a build directory `out/Debug` under `src/` with debug build configuration. You can replace `Debug` with another name, but it should be a subdirectory of `out`. Also you shouldn't have to run `gn gen` again—if you want to change the build arguments, you can run `gn args out/Debug` to bring up an editor.

To see the list of available build configuration options, run `gn args
out/Debug --list`.

**For generating Debug (aka "component" or "shared") build config of Electron:**

```sh
$ gn gen out/Debug --args="import(\"//electron/build/args/debug.gn\") $GN_EXTRA_ARGS"
```

**For generating Release (aka "non-component" or "static") build config of Electron:**

```sh
$ gn gen out/Release --args="import(\"//electron/build/args/release.gn\") $GN_EXTRA_ARGS"
```

**To build, run `ninja` with the `electron` target:** Nota Bene: This will also take a while and probably heat up your lap.

For the debug configuration:
```sh
$ ninja -C out/Debug electron
```

For the release configuration:
```sh
$ ninja -C out/Release electron
```

这个过程会构建 'libchromiumcontent' 里的所有内容，(如` chromium`中的`content`，及其依赖（包括Webkit 和 V8）)。因此，这个构建过程会比较费时。

你可以使用[sccache](https://github.com/mozilla/sccache)命令来提高后面的构建过程。 Add the GN arg `cc_wrapper = "sccache"` by running `gn args out/Debug` to bring up an editor and adding a line to the end of the file.

The built executable will be under `./out/Debug`:

```sh
$ ./out/Debug/Electron.app/Contents/MacOS/Electron
# or, on Windows
$ ./out/Debug/electron.exe
# or, on Linux
$ ./out/Debug/electron
```

### 打包

On linux, first strip the debugging and symbol information:
```sh
electron/script/strip-binaries.py -d out/Release
```

To package the electron build as a distributable zip file:
```sh
ninja -C out/Release electron:electron_dist_zip
```

### 交叉编译

To compile for a platform that isn't the same as the one you're building on, set the `target_cpu` and `target_os` GN arguments. For example, to compile an x86 target from an x64 host, specify `target_cpu = "x86"` in `gn args`.

```sh
$ gn gen out/Debug-x86 --args='... target_cpu = "x86"'
```

Not all combinations of source and target CPU/OS are supported by Chromium.

<table>
<tr><th>Host</th><th>Target</th><th>状态</th></tr>
<tr><td>Windows x64</td><td>Windows arm64</td><td>实验功能</td>
<tr><td>Windows x64</td><td>Windows x86</td><td>Automatically tested</td></tr>
<tr><td>Linux x64</td><td>Linux x86</td><td>Automatically tested</td></tr>
</table>

If you test other combinations and find them to work, please update this document :)

See the GN reference for allowable values of [`target_os`](https://gn.googlesource.com/gn/+/master/docs/reference.md#built_in-predefined-variables-target_os_the-desired-operating-system-for-the-build-possible-values) and [`target_cpu`](https://gn.googlesource.com/gn/+/master/docs/reference.md#built_in-predefined-variables-target_cpu_the-desired-cpu-architecture-for-the-build-possible-values).

#### Windows on Arm (experimental)

To cross-compile for Windows on Arm, [follow Chromium's guide](https://chromium.googlesource.com/chromium/src/+/refs/heads/master/docs/windows_build_instructions.md#Visual-Studio) to get the necessary dependencies, SDK and libraries, then build with `ELECTRON_BUILDING_WOA=1` in your environment before running `gclient sync`.

```bat
set ELECTRON_BUILDING_WOA=1
gclient sync -f --with_branch_heads --with_tags
```

Or (if using PowerShell):
```powershell
$env:ELECTRON_BUILDING_WOA=1
gclient sync -f --with_branch_heads --with_tags
```

Next, run `gn gen` as above with `target_cpu="arm64"`.

## 测试

To run the tests, you'll first need to build the test modules against the same version of Node.js that was built as part of the build process. To generate build headers for the modules to compile against, run the following under `src/` directory.

```sh
$ ninja -C out/Debug third_party/electron_node:headers
# Install the test modules with the generated headers
$ (cd electron/spec && npm i --nodedir=../../out/Debug/gen/node_headers)
```

接着，通过`electron/spec`命令来运行Electron：

```sh
# on Mac:
$ ./out/Debug/Electron.app/Contents/MacOS/Electron electron/spec
# on Windows:
$ ./out/Debug/electron.exe electron/spec
# on Linux:
$ ./out/Debug/electron electron/spec
```

可以通过增加其它标记来调试程序，例如：

```sh
$ ./out/Debug/Electron.app/Contents/MacOS/Electron electron/spec \
  --ci --enable-logging -g 'BrowserWindow module'
```

## Sharing the git cache between multiple machines

It is possible to share the gclient git cache with other machines by exporting it as SMB share on linux, but only one process/machine can be using the cache at a time. The locks created by git-cache script will try to prevent this, but it may not work perfectly in a network.

On Windows, SMBv2 has a directory cache that will cause problems with the git cache script, so it is necessary to disable it by setting the registry key

```sh
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Lanmanworkstation\Parameters\DirectoryCacheLifetime
```

to 0. More information: https://stackoverflow.com/a/9935126

## 故障排查

### Stale locks in the git cache
If `gclient sync` is interrupted while using the git cache, it will leave the cache locked. To remove the lock, pass the `--break_repo_locks` argument to `gclient sync`.

### I'm being asked for a username/password for chromium-internal.googlesource.com
If you see a prompt for `Username for 'https://chrome-internal.googlesource.com':` when running `gclient sync` on Windows, it's probably because the `DEPOT_TOOLS_WIN_TOOLCHAIN` environment variable is not set to 0. Open `Control Panel` → `System and Security` → `System` → `Advanced system settings` and add a system variable `DEPOT_TOOLS_WIN_TOOLCHAIN` with value `0`.  这将促使`depot_tools` 使用本地已安装的Visual Studio(默认状态下，`depot_tools`将会下载一个只有谷歌内部员工有权限使用的内部版本)。
