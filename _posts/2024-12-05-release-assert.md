---
layout: post
title: assert.h even in release mode
---

I like assertions. They let you establish invariants or other "it should be like this" before you do your function's logic. They're sorta like the [early return pattern](https://medium.com/swlh/return-early-pattern-3d18a41bba8) but with stronger consequences (like `abort()` instead of `return null`). Sometimes you do want a distinction between a debug hint assertion for developers and a release assertion that should always be there for safety assertions. Rust has this. It even favors release-always asserts since the on-in-release-and-debug assert macro is called `assert!()` and not `release_assert!()`. `debug_assert!()` is longer to type and is less commonly (but still commonly) used.

In C (and C++ too) there's the `assert.h` or `cassert` headers.

```c
#include <assert.h>
// OR in C++
#include <cassert>

int main() {
  assert(1 == 2); // will fail and abort()
}
```

But these can be disabled with `NDEBUG` which is common in some release mode configurations. A scenario where that might be bad is testing for an invariant that indirectly relies on some variable source (like user input or an environment variable like `$HOME` being present).

```c
int main() {
  unsigned int some_uint_from_somewhere = ...;
  non_zero_uint = uint_to_non_zero_uint(some_uint_from_somewhere);
  // ...
}

non_zero_uint uint_to_non_zero_uint(unsigned int x) {
  assert(x > 0);
  return x;
}
```

What happens if you supply `some_uint_from_somewhere` with 0? In debug mode, everything is OK. It crashes as expected since the assertion failed. That means you (who ran the program somehow) can see that some number failed some test and you should make sure your program is not providing the wrong data somewhere. You fix your logic. In release mode you think "oh it works!" and continue on **without fixing the issue**. This could cause problems or it could be completely harmless. Who knows.

The point is that sometimes you just want a release-and-debug always-there cannot-be-removed `assert()` function/macro in C/C++. It's not in the standard library so you have to make your own.

```c
#include <stdio.h>
#include <stdlib.h>

static void release_assert_fail(const char *expr, const char *file, int line) {
	fprintf(stderr, "Assertion failed: %s (%s: %d)\n", expr, file, line);
	fflush(NULL);
	abort();
}

#define release_assert(x) ((void)((x) || (release_assert_fail(#x, __FILE__, __LINE__), 0)))
```

```c
non_zero_uint uint_to_non_zero_uint(unsigned int x) {
  release_assert(x > 0);
  return x;
}
```
