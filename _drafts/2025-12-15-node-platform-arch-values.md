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

```gyp
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

https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L69

```gyp
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

https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L257

```gyp
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

https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/node.gypi#L310

```gyp
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

https://github.com/nodejs/node/blob/05f8772096f974190b11eabce0ea657bc5c8c1b1/common.gypi#L730

```gyp
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

https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/generator/compile_commands_json.py#L42

```py
default_variables.setdefault("OS", gyp.common.GetFlavor(params))
```

The `GetFlavor()` function delegates to `GetFlavorByPlatform()`...

https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/common.py#L513

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

https://github.com/nodejs/gyp-next/blob/732b09edc711ce3bad2df6db02ba8d5f315ca375/pylib/gyp/common.py#L485

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

All told, the returned flavor string can be `"emscripten"`, `"wasi"`, `"wasm"`, `"win"`, `"mac"`, `"solaris"`, `"freebsd"`, `"openbsd"`, `"netbsd"`, `"aix"`, `"zos"`, `"os400"`, `"linux"`, or any other custom `-DOS=<custom>` name.

So where does the Node.js build system set `-DOS=<something>`? Node.js doesn't use GYP directly, instead the official build procedure is to run `./configure` and `make -j4` (on Unix & macOS). The `./configure` script is a `/bin/sh` script that reexecutes itself as a Python script and then imports `./configure.py` if the Python version is satisfactory. That `configure.py` file is where the magic happens.

https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L47

```py
valid_os = ('win', 'mac', 'solaris', 'freebsd', 'openbsd', 'linux',
            'android', 'aix', 'cloudabi', 'os400', 'ios', 'openharmony')
```

https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L139

```py
parser.add_argument('--dest-os',
    action='store',
    dest='dest_os',
    choices=valid_os,
    help=f"operating system to build for ({', '.join(valid_os)})")
```

https://github.com/nodejs/node/blob/dcb9573d0f90ee0d66001d26514bbdcfd8e0610e/configure.py#L2388

```py
# determine the "flavor" (operating system) we're building for,
# leveraging gyp's GetFlavor function
flavor_params = {}
if options.dest_os:
  flavor_params['flavor'] = options.dest_os
flavor = GetFlavor(flavor_params)
```

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
