---
layout: post
title: Bundling Zig into an actually portable executable
---

## The problem

There are many ways to install Zig but only one way to install multiple versions of Zig: the direct binary download.

> This is the most straight-forward way of obtaining Zig: grab a Zig bundle for your platform from the Downloads page, extract it in a directory and add it to your PATH to be able to call zig from any location.

```sh
wget https://ziglang.org/builds/zig-linux-x86_64-0.13.0.tar.xz
tar xf zig-linux-x86_64-0.13.0.tar.xz
ln -s "$PWD/zig-linux-x86_64-0.13.0/zig" ~/.local/bin/zig-0.13

wget https://ziglang.org/builds/zig-linux-x86_64-0.14.0-dev.1021+fc2924080.tar.xz
tar xf zig-linux-x86_64-0.14.0-dev.1021+fc2924080.tar.xz
ln -s "$PWD/zig-linux-x86_64-0.14.0-dev.1021+fc2924080/zig" ~/.local/bin/zig-0.14
```

There's a bit of hassle here though; you need to choose a spot to house the actual `zig-linux-x86_64-0.13.0` installation root and then symlink it somewhere onto the `PATH` with a unique name. Ugh. 30 extra seconds of typing. 🙄

Besides the hassle of dealing with an installation root that you have to store or add to `PATH`, there's also the trick of programmatically deducing which Zig tarball you need if you're installing Zig locally as a project dependency. For example, a C/C++ project might fetch Zig to use the `zig cc`/`zig c++` C/C++ compiler toolchain across Windows, macOS, and Linux to avoid relying on the (potentially outdated) system C/C++ toolchain. Wouldn't it be nice if there was a way to create multiplatform binaries that worked on Windows, macOS, and Linux?

## The solution

The solution to the second problem (multiplatform binaries) is clear: Create an [actually portable executable](https://justine.lol/ape.html). The only problem is that there's only a `cosmocc`/`cosmoc++` C/C++ toolchain. The Zig project _is written in the Zig language_. So... we wrap it in a C/C++ jumper binary. 🤷‍♀️ Actually portable executables also just so happen to also have a `zipos` feature that means they are also `.zip` archives! This means you can `zip -A ./myapp.com ./assets/*` to bundle those assets into the binary as `/zip/assets/*` pseudo-files. This means we can do something like this in C/C++ to bundle all the various platform-specific `zig`/`zig.exe` binaries into a single actually portable executable.

```cpp
auto cache_dir = platformdirs::user_cache_dir("zig", "ziglang", "0.13.0");
if (!std::filesystem::exists(cache_dir)) {
  #if defined(__x86_64__)
    if (IsWindows()) {
      std::filesystem::copy_file("/zip/zig-windows-x86_64.exe", cache_dir / "zig.exe");
    } else if (IsXnu()) {
      std::filesystem::copy_file("/zip/zig-macos-x86_64", cache_dir / "zig");
    } else if (IsLinux()) {
      std::filesystem::copy_file("/zip/zig-linux-x86_64", cache_dir / "zig");
    }
  #elif defined(__aarch64__)
    if (IsXnu()) {
      std::filesystem::copy_file("/zip/zig-macos-aarch64", cache_dir / "zig");
    } else if (IsLinux()) {
      std::filesystem::copy_file("/zip/zig-linux-aarch64", cache_dir / "zig");
    }
  #endif
}
execv((cache_dir / (IsWindows() ? "zig.exe" : "zig")).c_str(), argv);
```

This leads perfectly to the solution for the first gripe: you need a dedicated "zig root" to hold the `lib/*` files. What if we bundled `lib/` into the binary's `/zip/` pseudo-filesystem too? Then we could extract `lib/` when we extract the `zig`/`zig.exe` binaries and have a completely self-contained self-extracting multiplatform single executable! 🤩

```sh
cosmoc++ -std=c++23 -o myapp myapp.cpp
zip -Ar myapp \
  myapp-windows-x86_64.exe \
  myapp-macos-arm64 myapp-macos-x86_64 \
  myapp-linux-arm64 myapp-linux-x86_64
```

This trick has a few major downsides:

1. It balloons your binary size. By quite a bit.
2. It can take a few extra seconds to unzip the first time. More files = more time.

To me these factors are an acceptable tradeoff in certain scenarios.

If you really want to you could even commit `zig`/`zig.com` to Git LFS to bundle your project's development toolchain with the project itself. Forrest Smith does this in [Using Zig to Commit Toolchains to VCS](https://www.forrestthewoods.com/blog/using-zig-to-commit-toolchains-to-vcs/). It's not for everyone but it is a _thing you can do_ if you want to.
