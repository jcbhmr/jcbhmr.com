---
title: Node.js platform/arch possible values
---

The official Node.js docs (v25.2.1) say that `process.platform` can be `'aix' | 'darwin' | 'freebsd' | 'linux' | 'openbsd' | 'sunos' | 'win32'` and `process.arch` can be `'arm' | 'arm64' | 'ia32' | 'loong64' | 'mips' | 'mipsel' | 'ppc64' | 'riscv64' | 's390' | 's390x' | 'x64'`. That's not quite true. This is a dive into where these values come from and what their possible values could actually be.

- `node:process` `platform` & `node:os` `platform()`
- `node:process` `arch` & `node:os` `arch()`
- `node:os` `machine()`

## `node:process` `platform` & `node:os` `platform()`

The Node.js process object is defined across some C++ files as a global object. The `node:process` module just reexports it with all the appropriate named properties. This means there's nothing to see in `lib/process.js`

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/lib/process.js"><code>lib/process.js</code></a></div>

```js
'use strict';

// Re-export process as a built-in module
module.exports = process;
```

Instead, we have to jump to the `src/node_*.cc` files to see anything that defines `process.*` properties & methods. Don't worry about where and how this v8 `process` object gets set as a global variable; that's not important right now.

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_process_object.cc#L132"><code>src/node_process_object.cc</code></a></div>

```cpp
// process.platform
READONLY_STRING_PROPERTY(process, "platform", per_process::metadata.platform);
```

What's this `per_process::metadata` struct? Where is it defined? It must be some kind of singleton/constant since it's being used to access `.platform` as a property. Sure enough, it is!

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_metadata.h#L152"><code>src/node_metadata.h</code></a></div>

```cpp
// Per-process global
namespace per_process {
extern Metadata metadata;
}
```

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_metadata.cc#L54"><code>src/node_metadata.cc</code></a></div>

```cpp
namespace per_process {
Metadata metadata;
}
```

`node::Metadata` is a class that has a single constructor that sets `.arch` and `.platform` (`std::string`-s) to some `#define`-ed macro string literals.

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_metadata.h#L147"><code>src/node_metadata.h</code></a></div>

```cpp
class Metadata {
 public:
  Metadata();
  Metadata(Metadata&) = delete;
  Metadata(Metadata&&) = delete;
  Metadata operator=(Metadata&) = delete;
  Metadata operator=(Metadata&&) = delete;

  struct Versions {
    // -- ✂️ SNIP --
  }

  Versions versions;
  const Release release;
  const std::string arch;
  const std::string platform;
};
```

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_metadata.cc#L205"><code>src/node_metadata.cc</code></a></div>

```cpp
Metadata::Metadata() : arch(NODE_ARCH), platform(NODE_PLATFORM) {}
```

So far we know that `node.platform` is set to whatever value `NODE_PLATFORM` is set to. Where is that set? It's set by the [GYP build system](https://gyp.gsrc.io/) configuration.

```py
{
  'target_name': '<(node_core_target_name)',
  'type': 'executable',

  'defines': [
    'NODE_ARCH="<(target_arch)"',
    'NODE_PLATFORM="<(OS)"', # 👈
    'NODE_WANT_INTERNALS=1',
  ],

  # -- ✂️ SNIP --
}
```

What's this `OS` variable? Where does it come from? It is a [GYP predefined variable](https://gyp.gsrc.io/docs/InputFormatReference.md#predefined-variables). I don't really know if you're supposed to manually set it or if it's inferred from your host platform.

> - `OS`: The name of the operating system that the generator produces output for. Common values for values for `OS` are:
>
>   - `'linux'`
>   - `'mac'`
>   - `'win'`
>
>   But other values may be encountered and this list should not be considered exhaustive.

&mdash; [GYP input format reference docs](https://gyp.gsrc.io/docs/InputFormatReference.md#predefined-variables)

There are other files that appear to overwrite the value of `NODE_PLATFORM` to other values that are more specialized than just `<(OS)`.

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L69"><code>node.gypi</code></a></div>

```py
[ 'OS=="win"', {
  'defines!': [
    'NODE_PLATFORM="win"',
  ],
  'defines': [
    'FD_SETSIZE=1024',
    # we need to use node's preferred "win32" rather than gyp's preferred "win"
    'NODE_PLATFORM="win32"', # 👈
    '_UNICODE=1',
  ],
  # -- ✂️ SNIP --
]
```

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L257"><code>node.gypi</code></a></div>

```py
[ 'OS=="mac"', {
  # linking Corefoundation is needed since certain macOS debugging tools
  # like Instruments require it for some features. Security is needed for
  # --use-system-ca.
  'libraries': [ '-framework CoreFoundation -framework Security' ],
  'defines!': [
    'NODE_PLATFORM="mac"',
  ],
  'defines': [
    # we need to use node's preferred "darwin" rather than gyp's preferred "mac"
    'NODE_PLATFORM="darwin"', # 👈
  ],
}],
```

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L310"><code>node.gypi</code></a></div>

```py
[ 'OS=="solaris"', {
  'libraries': [
    '-lkstat',
    '-lumem',
  ],
  'defines!': [
    'NODE_PLATFORM="solaris"',
  ],
  'defines': [
    # we need to use node's preferred "sunos"
    # rather than gyp's preferred "solaris"
    'NODE_PLATFORM="sunos"', # 👈
  ],
}],
```

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/common.gypi#L730"><code>common.gypi</code></a></div>

```py
['OS == "zos"', {
  'defines': [
    '_XOPEN_SOURCE_EXTENDED',
    '_XOPEN_SOURCE=600',
    '_UNIX03_THREADS',
    '_UNIX03_WITHDRAWN',
    '_UNIX03_SOURCE',
    '_OPEN_SYS_SOCK_IPV6',
    '_OPEN_SYS_FILE_EXT=1',
    '_POSIX_SOURCE',
    '_OPEN_SYS',
    '_OPEN_SYS_IF_EXT',
    '_OPEN_SYS_SOCK_IPV6',
    '_OPEN_MSGQ_EXT',
    '_LARGE_TIME_API',
    '_ALL_SOURCE',
    '_AE_BIMODAL=1',
    '__IBMCPP_TR1__',
    'NODE_PLATFORM="os390"', # 👈
    'PATH_MAX=1024',
    '_ENHANCED_ASCII_EXT=0xFFFFFFFF',
    '_Export=extern',
    '__static_assert=static_assert',
  ],
  # -- ✂️ SNIP --
]
```

But what are the other possible values for this `OS` variable? We have to take a look at how GYP initializes that variable with a default value. It seems like all code paths call `gyp.common.GetFlavor(params)` somehow to get an initial OS name and then sometimes do custom configuration and initialization based on the return value (if win, if linux, if mac, etc.).

<div><a href="https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/generator/compile_commands_json.py#L42">nodejs/gyp-next: <code>pylib/gyp/generator/compile_commands_json.py</code></a></div>

```py
default_variables.setdefault("OS", gyp.common.GetFlavor(params))
```

The `GetFlavor()` function delegates to `GetFlavorByPlatform()`...

<div><a href="https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/common.py#L513">nodejs/gyp-next: <code>pylib/gyp/common.py</code></a></div>

```py
def GetFlavor(params):
    if "flavor" in params:
        return params["flavor"]

    defines = GetCompilerPredefines()
    if "__EMSCRIPTEN__" in defines:
        return "emscripten"
    if "__wasm__" in defines:
        return "wasi" if "__wasi__" in defines else "wasm"

    return GetFlavorByPlatform()
```

...which is where the real `if sys.platform == "..."` magic happens.

<div><a href="https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/common.py#L485">nodejs/gyp-next: <code>pylib/gyp/common.py</code></a></div>

```py
def GetFlavorByPlatform():
    """Returns |params.flavor| if it's set, the system's default flavor else."""
    flavors = {
        "cygwin": "win",
        "win32": "win",
        "darwin": "mac",
    }

    if sys.platform in flavors:
        return flavors[sys.platform]
    if sys.platform.startswith("sunos"):
        return "solaris"
    if sys.platform.startswith(("dragonfly", "freebsd")):
        return "freebsd"
    if sys.platform.startswith("openbsd"):
        return "openbsd"
    if sys.platform.startswith("netbsd"):
        return "netbsd"
    if sys.platform.startswith("aix"):
        return "aix"
    if sys.platform.startswith(("os390", "zos")):
        return "zos"
    if sys.platform == "os400":
        return "os400"

    return "linux"
```

All told, the returned flavor string can be `"emscripten"`, `"wasi"`, `"wasm"`, `"win"`, `"mac"`, `"solaris"`, `"freebsd"`, `"openbsd"`, `"netbsd"`, `"aix"`, `"zos"`, `"os400"`, `"linux"`, or any other custom name set by `params["flavor"]`.

So where does the Node.js build system set `-DOS=<something>`? Node.js doesn't use GYP directly, instead the official build procedure is to run `./configure` and `make -j4` (on Unix & macOS). The `./configure` script is a `/bin/sh` script that reexecutes itself as a Python script and then imports `./configure.py` if the Python version is satisfactory. That `configure.py` file is where the magic happens.

<div><a href="https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L47"><code>configure.py</code></a></div>

```py
valid_os = ('win', 'mac', 'solaris', 'freebsd', 'openbsd', 'linux',
            'android', 'aix', 'cloudabi', 'os400', 'ios', 'openharmony')
```

<div><a href="https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L139"><code>configure.py</code></a></div>

```py
parser.add_argument('--dest-os',
    action='store',
    dest='dest_os',
    choices=valid_os,
    help=f"operating system to build for ({', '.join(valid_os)})")
```

<div><a href="https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L2388"><code>configure.py</code></a></div>

```py
# determine the "flavor" (operating system) we're building for,
# leveraging gyp's GetFlavor function
flavor_params = {}
if options.dest_os:
  flavor_params['flavor'] = options.dest_os
flavor = GetFlavor(flavor_params)
```

<sup>Yes, it's the same `gyp.common.GetFlavor()` function</sup>

This means that the OS **when set manually** (for cross compiling or whatever) must be one of `'win'`, `'mac'`, `'solaris'`, `'freebsd'`, `'openbsd'`, `'linux'`, `'android'`, `'aix'`, `'cloudabi'`, `'os400'`, `'ios'`, or `'openharmony'`. But if it's **not set at all** (if `options.dest_os` is falsey) then it can be any of the possible return values from `GetFlavor()` with the knowledge that `params["flavor"]` is **not set**. `GetCompilerPredefines()` may still return `__EMSCRIPTEN__`, `__wasm__`, or `__wasi__` though so we can't rule those out as impossible values. That means the total combined list is:

- `"emscripten"`
- `"wasi"`
- `"wasm"`
- `"netbsd"`
- `"zos"`
- `"win"`
- `"mac"`
- `"solaris"`
- `"freebsd"`
- `"openbsd"`
- `"linux"`
- `"android"`
- `"aix"`
- `"cloudabi"`
- `"os400"`
- `"ios"`
- `"openharmony"`

But some of these values, remember, have special mappings to Node.js `NODE_PLATFORM` values.

- `zos` becomes `os390`
- `solaris` becomes `sunos`
- `mac` becomes `darwin`
- `win` becomes `win32`

All together here's a list of **all** the possible Node.js `NODE_PLATFORM` (and by extension `process.platform` & friends) values with some usage numbers from a quick GitHub search:

- [0 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29emscripten%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"emscripten"`
- [1 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29wasi%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"wasi"`
- [0 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29wasm%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"wasm"`
- [114 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29netbsd%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"netbsd"`
- [284 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29os390%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"os390"` (`"zos"` in GYP)
- [196 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29win32%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"win32"` (`"win"` in GYP)
- [198 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29darwin%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"darwin"` (`"mac"` in GYP)
- [215 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29sunos%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"sunos"` (`"solaris"` in GYP)
- [211 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29freebsd%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"freebsd"`
- [223 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29openbsd%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"openbsd"`
- [148 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29linux%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"linux"`
- [198 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29android%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"android"`
- [240 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29aix%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"aix"`
- [0 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29cloudabi%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"cloudabi"` <sup>[Yes, really.](https://github.com/search?q=%2F%28%3F-i%29%28%22%7C%27%29cloudabi%28%22%7C%27%29%2F+%28path%3A%2F%5C.%5Bcm%5D%3F%5Bjt%5Dsx%3F%24%2F+OR+path%3A%2F%28%5E%7C%5C%2F%29package%5C.json%24%2F%29&type=code)</sup>
- [1.1k matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29os400%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"os400"`
- [200 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29ios%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"ios"`
- [229 matches](https://github.com/search?q=%2F%28process%5C.platform%7Cos%5C.platform%5C%28%5C%29%29%5Cs*%3D%3D%3D%3F%5Cs*%28%22%7C%27%29openharmony%28%22%7C%27%29%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code) `"openharmony"`

Note that a few of these values are extremely rare and can be considered impossible to happen. `emscripten`, `wasi`, `wasm`, and `cloudabi` can all be considered _not gonna happen_. With those omissions the final `process.platform` exhaustive list is: `"netbsd"`, `"os390"`, `"win32"`, `"darwin"`, `"sunos"`, `"freebsd"`, `"openbsd"`, `"linux"`, `"android"`, `"aix"`, `"os400"`, `"ios"`, `"openharmony"`. That's different from the [official documentation](https://nodejs.org/api/process.html#processplatform):

> Currently possible values are:
>
> - `'aix'`
> - `'darwin'`
> - `'freebsd'`
> - `'linux'`
> - `'openbsd'`
> - `'sunos'`
> - `'win32'`
>
> The value 'android' may also be returned if the Node.js is built on the Android operating system. However, Android support in Node.js is experimental.

&mdash; [Node.js `process.platform` docs](https://nodejs.org/api/process.html#processplatform)

## `node:process` `arch` & `node:os` `arch()`

The first few layers of `platform.arch` are quite similar to `node.platform`; it's just sourced from a different macro `NODE_ARCH`, which is set in a different way.

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_process_object.cc#L129"><code>src/node_process_object.cc</code></a></div>

```cpp
// process.arch
READONLY_STRING_PROPERTY(process, "arch", per_process::metadata.arch);
```

And we know from the above `process.platform` investigation that `per_process::metadata.arch` is set to `NODE_ARCH`.

<div><a href="https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/src/node_metadata.cc#L205"><code>src/node_metadata.cc</code></a></div>

```cpp
Metadata::Metadata() : arch(NODE_ARCH), platform(NODE_PLATFORM) {}
```

So where is this `NODE_ARCH` macro defined?

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/node.gyp#L555"><code>node.gyp</code></div>

```py
{
 'target_name': '<(node_core_target_name)',
 'type': 'executable',

 'defines': [
   'NODE_ARCH="<(target_arch)"', # 👈
   'NODE_PLATFORM="<(OS)"',
   'NODE_WANT_INTERNALS=1',
 ],
 # -- ✂️ SNIP --
}
```

There's a `target_arch` variable defined somewhere. There's no monkey business with custom GYP-to-Node.js arch name conversions like there was with `NODE_PLATFORM`.

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/configure.py#L1581"><code>configure.py</code></a></div>

```py
host_arch = host_arch_win() if os.name == 'nt' else host_arch_cc()
target_arch = options.dest_cpu or host_arch
# ia32 is preferred by the build tools (GYP) over x86 even if we prefer the latter
# the Makefile resets this to x86 afterward
if target_arch == 'x86':
  target_arch = 'ia32'
# x86_64 is common across linuxes, allow it as an alias for x64
if target_arch == 'x86_64':
  target_arch = 'x64'
o['variables']['host_arch'] = host_arch
o['variables']['target_arch'] = target_arch # 👈
```

First, let's look at `host_arch` to see what possible values it could be.

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/configure.py#L1465"><code>configure.py</code></a></div>

```py
def host_arch_win():
  """Host architecture check using environ vars (better way to do this?)"""

  observed_arch = os.environ.get('PROCESSOR_ARCHITECTURE', 'AMD64')
  arch = os.environ.get('PROCESSOR_ARCHITEW6432', observed_arch)

  matchup = {
    'AMD64'  : 'x64',
    'arm'    : 'arm',
    'mips'   : 'mips',
    'ARM64'  : 'arm64'
  }

  return matchup.get(arch, 'x64')
```

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/configure.py#L1425"><code>configure.py</code></a></div>

```py
def host_arch_cc():
  """Host architecture check using the CC command."""

  if sys.platform.startswith('zos'):
    return 's390x'
  k = cc_macros(os.environ.get('CC_host'))

  matchup = {
    '__aarch64__' : 'arm64',
    '__arm__'     : 'arm',
    '__i386__'    : 'ia32',
    '__MIPSEL__'  : 'mipsel',
    '__mips__'    : 'mips',
    '__PPC64__'   : 'ppc64',
    '__PPC__'     : 'ppc64',
    '__x86_64__'  : 'x64',
    '__s390x__'   : 's390x',
    '__riscv'     : 'riscv',
    '__loongarch64': 'loong64',
  }

  rtn = 'ia32' # default

  for key, value in matchup.items():
    if k.get(key, 0) and k[key] != '0':
      rtn = value
      break

  if rtn == 'mipsel' and '_LP64' in k:
    rtn = 'mips64el'

  if rtn == 'riscv':
    if k['__riscv_xlen'] == '64':
      rtn = 'riscv64'
    else:
      rtn = 'riscv32'

  return rtn
```

<sup>`cc_macros()` returns all predefined C compiler macros as a `dict`</sup>

`host_arch_win()` possible values: `x64`, `arm`, `mips`, `arm64` \
`host_arch_cc()` possible values: `s390x`, `arm64`, `arm`, `ia32`, `mipsel`, `mips`, `mips64el`, `ppc64`, `x64`, `riscv64`, `riscv32`, `loong64`

Now let's look at what options are valid `options.dest_cpu` `--dest-cpu` values.

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/configure.py#L49"><code>configure.py</code></a></div>

```py
valid_arch = ('arm', 'arm64', 'ia32', 'mips', 'mipsel', 'mips64el',
              'ppc64', 'x64', 'x86', 'x86_64', 's390x', 'riscv64', 'loong64')
```

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/configure.py#L110"><code>configure.py</code></a></div>

```py
parser.add_argument('--dest-cpu',
    action='store',
    dest='dest_cpu',
    choices=valid_arch,
    help=f"CPU architecture to build for ({', '.join(valid_arch)})")
```

**But** `x86` gets mapped to `ia32` and `x86_64` gets mapped to `x64`. Here's the logic flow. Note that some values that aren't allowed in `--dest-cpu` can sneak in if `--dest-cpu` isn't set and the arch is inferred from the environment.

```py
host_arch = host_arch_win() if os.name == 'nt' else host_arch_cc()
# TYPE: "s390x" | "arm64" | "arm" | "ia32" | "mipsel" | "mips" | "mips64el" | "ppc64" | "x64" | "riscv64" | "riscv32" | "loong64"

target_arch = options.dest_cpu or host_arch
# TYPE: "x86" | "x86_64" | "s390x" | "arm64" | "arm" | "ia32" | "mipsel" | "mips" | "mips64el" | "ppc64" | "x64" | "riscv64" | "riscv32" | "loong64"

# ia32 is preferred by the build tools (GYP) over x86 even if we prefer the latter
# the Makefile resets this to x86 afterward
if target_arch == 'x86':
  target_arch = 'ia32'
# target_arch TYPE: "x86_64" | "s390x" | "arm64" | "arm" | "ia32" | "mipsel" | "mips" | "mips64el" | "ppc64" | "x64" | "riscv64" | "riscv32" | "loong64"

# x86_64 is common across linuxes, allow it as an alias for x64
if target_arch == 'x86_64':
  target_arch = 'x64'
# target_arch TYPE: "s390x" | "arm64" | "arm" | "ia32" | "mipsel" | "mips" | "mips64el" | "ppc64" | "x64" | "riscv64" | "riscv32" | "loong64"

o['variables']['host_arch'] = host_arch

# FINAL TYPE: "s390x" | "arm64" | "arm" | "ia32" | "mipsel" | "mips" | "mips64el" | "ppc64" | "x64" | "riscv64" | "riscv32" | "loong64"
o['variables']['target_arch'] = target_arch
```

So the final possible values for `process.arch` are the same as those possible for `NODE_ARCH` which are: `"s390x"`, `"arm64"`, `"arm"`, `"ia32"`, `"mipsel"`, `"mips"`, `"mips64el"`, `"ppc64"`, `"x64"`, `"riscv64"`, `"riscv32"`, `"loong64"`.

## `node:os` `machine()`

`os.machine()` is a wrapper that returns the `...[3]` property of the return value of `internalBinding("os").getOSInformation()`.

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/lib/os.js#L330"><code>lib/os.js</code></a></div>

```js
module.exports = {
  // -- ✂️ SNIP --
  machine: getMachine,
};
```

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/lib/os.js#L100"><code>lib/os.js</code></a></div>

```js
const getMachine = () => machine;
```

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/lib/os.js#L77"><code>lib/os.js</code></a></div>

```js
const {
  0: type,
  1: version,
  2: release,
  3: machine,
} = _getOSInformation();
```

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/lib/os.js#L77"><code>lib/os.js</code></a></div>

```js
const {
  getAvailableParallelism,
  getCPUs,
  getFreeMem,
  getHomeDirectory: _getHomeDirectory,
  getHostname: _getHostname,
  getInterfaceAddresses: _getInterfaceAddresses,
  getLoadAvg,
  getPriority: _getPriority,
  getOSInformation: _getOSInformation, // 👈
  getTotalMem,
  getUserInfo,
  getUptime: _getUptime,
  isBigEndian,
  setPriority: _setPriority,
} = internalBinding('os');
```

That `getOSInformation()` C++ function sets `...[3]` to `info.machine` which is set by `uv_os_name()`.

<div><a href="https://github.com/nodejs/node/blob/4e1f39b678b37017ac9baa0971e3aeecd3b67b51/src/node_os.cc#L82"><code>src/node_os.cc</code></a></div>

```cpp
static void GetOSInformation(const FunctionCallbackInfo<Value>& args) {
  Environment* env = Environment::GetCurrent(args);
  uv_utsname_t info;
  int err = uv_os_uname(&info);

  if (err != 0) {
    CHECK_GE(args.Length(), 1);
    USE(env->CollectUVExceptionInfo(
        args[args.Length() - 1], err, "uv_os_uname"));
    return;
  }

  // [sysname, version, release, machine]
  Local<Value> osInformation[4];
  if (String::NewFromUtf8(env->isolate(), info.sysname)
          .ToLocal(&osInformation[0]) &&
      String::NewFromUtf8(env->isolate(), info.version)
          .ToLocal(&osInformation[1]) &&
      String::NewFromUtf8(env->isolate(), info.release)
          .ToLocal(&osInformation[2]) &&
      String::NewFromUtf8(env->isolate(), info.machine) // 👈
          .ToLocal(&osInformation[3])) {
    args.GetReturnValue().Set(
        Array::New(env->isolate(), osInformation, arraysize(osInformation)));
  }
}
```

`uv_os_name()` has two different implementations: one for Unix and one for Windows. The Unix implementation uses `uname()` `.machine`.

```cpp
int uv_os_uname(uv_utsname_t* buffer) {
  struct utsname buf;
  int r;

  if (buffer == NULL)
    return UV_EINVAL;

  if (uname(&buf) == -1) {
    r = UV__ERR(errno);
    goto error;
  }

  // -- ✂️ SNIP --

#if defined(_AIX) || defined(__PASE__)
  r = uv__strscpy(buffer->machine, "ppc64", sizeof(buffer->machine));
#else
  r = uv__strscpy(buffer->machine, buf.machine, sizeof(buffer->machine));
#endif

  if (r == UV_E2BIG)
    goto error;

  return 0;

error:
  buffer->sysname[0] = '\0';
  buffer->release[0] = '\0';
  buffer->version[0] = '\0';
  buffer->machine[0] = '\0';
  return r;
}
```

So the possible values are _whatever `uname()` `.machine` can be_ plus `ppc64` if it's AIX. But what are the possible values of `uname()` `.machine`?

_TODO_ https://stackoverflow.com/questions/45125516/possible-values-for-uname-m

**Back to `uv_os_name()`.** The Windows implementation sets the `.machine` property to a variety of static strings depending on some conditions.

```cpp
int uv_os_uname(uv_utsname_t* buffer) {
  /* Implementation loosely based on
     https://github.com/gagern/gnulib/blob/master/lib/uname.c */
  OSVERSIONINFOW os_info;
  SYSTEM_INFO system_info;
  HKEY registry_key;
  WCHAR product_name_w[256];
  DWORD product_name_w_size;
  size_t version_size;
  int processor_level;
  int r;

  // -- ✂️ SNIP --

  /* Populate the machine field. */
  GetSystemInfo(&system_info);

  switch (system_info.wProcessorArchitecture) {
    case PROCESSOR_ARCHITECTURE_AMD64:
      uv__strscpy(buffer->machine, "x86_64", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_IA64:
      uv__strscpy(buffer->machine, "ia64", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_INTEL:
      uv__strscpy(buffer->machine, "i386", sizeof(buffer->machine));

      if (system_info.wProcessorLevel > 3) {
        processor_level = system_info.wProcessorLevel < 6 ?
                          system_info.wProcessorLevel : 6;
        buffer->machine[1] = '0' + processor_level;
      }

      break;
    case PROCESSOR_ARCHITECTURE_IA32_ON_WIN64:
      uv__strscpy(buffer->machine, "i686", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_MIPS:
      uv__strscpy(buffer->machine, "mips", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_ALPHA:
    case PROCESSOR_ARCHITECTURE_ALPHA64:
      uv__strscpy(buffer->machine, "alpha", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_PPC:
      uv__strscpy(buffer->machine, "powerpc", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_SHX:
      uv__strscpy(buffer->machine, "sh", sizeof(buffer->machine));
      break;
    case PROCESSOR_ARCHITECTURE_ARM:
      uv__strscpy(buffer->machine, "arm", sizeof(buffer->machine));
      break;
    default:
      uv__strscpy(buffer->machine, "unknown", sizeof(buffer->machine));
      break;
  }

  return 0;

error:
  buffer->sysname[0] = '\0';
  buffer->release[0] = '\0';
  buffer->version[0] = '\0';
  buffer->machine[0] = '\0';
  return r;
}
```

All possible values are: `x86_64`, `ia64`, `i386`, `i486`, `i586`, `i686`, `mips`, `alpha`, `powerpc`, `sh`, `arm`, `unknown`.
