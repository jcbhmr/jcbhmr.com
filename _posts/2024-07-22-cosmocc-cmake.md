---
layout: post
title: Use cosmocc to cross‐compile a CMake project
---

We know [how to compile an Autotools project using `cosmocc`](https://github.com/jart/cosmopolitan/tree/master/tool/cosmocc#building-open-source-software) but how do you compile a CMake project with `cosmocc`?

Here's an example project:

<div><code>main.c</code></div>

```c
#include <stdio.h>

int main() {
    puts("Hello world!");
    return 0;
}
```

<div><code>CMakeLists.txt</code></div>

```cmake
cmake_minimum_required(VERSION 3.29)
project(hello-world LANGUAGES C)
add_executable(hello-world main.c)
```

You can _almost_ just do `CC=cosmocc cmake ...` and call it day. Almost. But not quite! 😉 There are some other CMake settings you need to set to get things to work well with `cosmocc`.

First, you'll need to add a `CMAKE_USER_MAKE_RULES_OVERRIDE` file to define the object suffix as `.o` instead of `.obj`. `cosmocc` currently doesn't work if the object extension is not `.o`. 🤷‍♀️

<div><code>cosmocc-override.cmake</code></div>

```cmake
set(CMAKE_ASM_OUTPUT_EXTENSION .o)
set(CMAKE_C_OUTPUT_EXTENSION .o)
set(CMAKE_CXX_OUTPUT_EXTENSION .o)
```

**Why can't we just specify `-DCMAKE_C_OUTPUT_EXTENSION=".o"` on the command line?** It needs to be defined after the `Generic` configuration is loaded so it doesn't get reset to the default `.obj`. `CMAKE_USER_MAKE_RULES_OVERRIDE` is the recommended way to do it. https://gitlab.kitware.com/cmake/cmake/-/issues/18713

<table><td>

```sh
ASM="cosmocc" \
CC="cosmocc" \
CXX="cosmoc++" \
cmake \
  -DCMAKE_SYSTEM_NAME="Generic" \
  -UCMAKE_SYSTEM_PROCESSOR \
  -DCMAKE_USER_MAKE_RULES_OVERRIDE="$PWD/cosmocc-override.cmake" \
  -DCMAKE_AR="$(command -v cosmoar)" \
  -DCMAKE_RANLIB="$(command -v cosmoranlib)" \
  -B build
```

<tr><td>

```
-- The C compiler identification is GNU 12.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /somewhere/cosmocc/bin/cosmocc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Configuring done (2.9s)
-- Generating done (0.0s)
-- Build files have been written to: /somewhere/hello-world/build
```

</table>

**Do I have to specify `CMAKE_SYSTEM_NAME` and `CMAKE_SYSTEM_PROCESSOR`?** Yes. They configure some CMake magic internally and configure what your `CMakeLists.txt` thinks it's compilation target is.

**Why not `CMAKE_SYSTEM_NAME="Linux"`?** So that `if(LINUX)` and other Linux-specific stuff in `CMakeLists.txt` doesn't activate for <code>unknown-<u><i>unknown</i></u>-cosmo</code>.

**Why not `CMAKE_SYSTEM_PROCESSOR="unknown"`?** _`unset`_ is what means <code><u><i>unknown</i></u>-unknown-cosmo</code> to CMake. CMake internals use `if(CMAKE_SYSTEM_PROCESSOR)`, not `if(CMAKE_SYSTEM_PROCESSOR STREQUAL unknown)`.

**Why do `CMAKE_AR` and `CMAKE_RANLIB` need to be an absolute path?** Unfixed issue. https://gitlab.kitware.com/cmake/cmake/-/issues/18087

**Why can't we do `AR="cosmoar"` like `CC` and `CXX`?** Unfixed issue. https://gitlab.kitware.com/cmake/cmake/-/issues/18712

Now that the project is configured with all the compiler settings and other magic✨ we can run the build step:

<table><td>

```sh
cmake --build build
```

<tr><td>

```
[ 50%] Building C object CMakeFiles/hello-world.dir/main.c.o
[100%] Linking C executable hello-world
[100%] Built target hello-world
```

</table>

You just made a magic✨ binary! 🥳

<sup>On Windows you need to rename it or symlink it to have an `.exe` or `.com` suffix to execute it.</sup>

<table><td>

```sh
./build/hello-world     
```

<tr><td>

```
Hello world!
```

</table>
