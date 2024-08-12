---
layout: post
title: Wrapping non-C/C++ projects with Cosmopolitan jumper bundles
---

[Cosmopolitan Libc](https://github.com/jart/cosmopolitan) is cool. But it only works for C/C++ projects. What if you already have five different platform-specific binaries that you want to combine into a single [actually portable executable](https://justine.lol/ape.html)? You embed them all into a single binary of course!

```cpp
auto cache_dir = user_cache_dir();
if (!std::filesystem::exists(cache_dir)) {
  #if defined(__x86_64__) || defined(_M_X64)
    if (IsWindows()) {
      std::filesystem::copy_file("/zip/myapp-windows-x86_64.exe", cache_dir / "myapp.exe");
    } else if (IsXnu()) {
      std::filesystem::copy_file("/zip/myapp-macos-x86_64", cache_dir / "myapp");
    } else if (IsLinux()) {
      std::filesystem::copy_file("/zip/myapp-linux-x86_64", cache_dir / "myapp");
    }
  #elif defined(__aarch64__) || defined(_M_ARM64)
    if (IsXnu()) {
      std::filesystem::copy_file("/zip/myapp-macos-arm64", cache_dir / "myapp");
    } else if (IsLinux()) {
      std::filesystem::copy_file("/zip/myapp-linux-arm64", cache_dir / "myapp");
    }
  #endif
}

if (IsWindows()) {
  execv_windows((cache_dir / "myapp.exe").c_str(), argv);
} else {
  execv((cache_dir / "myapp").c_str(), argv);
}
```

```sh
cosmoc++ -std=c++23 -o myapp myapp.cpp
zip -Ar myapp \
  myapp-windows-x86_64.exe \
  myapp-macos-arm64 myapp-macos-x86_64 \
  myapp-linux-arm64 myapp-linux-x86_64
```

This trick has a few major downsides: 1) It balloons your binary size. 2) It can take a few extra seconds to unzip the first time. To me those factors are an acceptable tradeoff in certain scenarios.

Take Zig for example. The procedure to install Zig right now is to download a platform-specific tarball to an empty directory somewhere, extract it, and then add a folder to your `PATH` so that you can run `zig`. **What if instead you could directly download an all-in-one self-contained `zig` binary?**

```sh
wget -O ~/.local/bin/zig https://example.org/download/zig-ape-0.13.0.com
chmod +x ~/.local/bin/zig
```

With the way things are usually distributed in `.zip` or `.tar.gz` archives a more realistic alternative for an existing project like Zig is something like this:

```sh
wget https://example.org/download/zig-ape-0.13.0.zip
unzip -j zig-ape-*.zip -d ~/.local/bin
rm zig-ape-*.zip
```

<sup>[I made a thing.](https://github.com/jcbhmr/release-cutter) You can actually do this.</sup>

Cool, right? I think this would be pretty cool because...

1. You now have a cross-platform binary that can cross-compile to many other platforms.
2. It can easily be copied across systems since it's a single file with no relative path dependencies.
3. It's one file. You can put it right in your `<prefix>/bin` folder and that's it.
4. It works on Windows.

If you really want to you could even commit `zig`/`zig.com` to Git LFS to bundle your project's development toolchain with the project itself. Forrest Smith does this in [Using Zig to Commit Toolchains to VCS](https://www.forrestthewoods.com/blog/using-zig-to-commit-toolchains-to-vcs/). It's not for everyone but it is a thing you can do.
