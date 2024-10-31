---
layout: post
title: TIL C Standard Annex K exists but you shouldn't use it
---

Annex K is the technical name. Other common keywords are `__STDC_LIB_EXT1__ ` and `__STDC_WANT_LIB_EXT1__`. Annex K defines the "secure" `_s` suffix stuff like `sprintf_s()` and `scanf_s()`.

Also check out [Field experience with Annex K (2015)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1967.htm) and the [Bounds checking - cppreference.com](https://en.cppreference.com/w/c/error#Bounds_checking) technical documentation.

## The goal

What's the point of the `_s()` functions? They check their arguments for more invariants like "will call the constraint handler if the stream is null, the string is null, the `bufsz` is zero, or the buffer would write out-of-bounds beyond the specified length". That seems like a good idea, right? Yeah! It does!

<dl>
<dt><b>Happy path</b>
<dd>

```c
FILE *file = fopen("hello.txt", "r");
// file is OK.
```

<dd>

```c
FILE *file;
errno_t err = fopen_s(&file, "hello.txt", "r");
// file is OK
```

<dt><b>Sad path</b>
<dd>

```c
FILE *file = fopen("notexist.txt", "r");
// file is NULL, errno is set.
```

<dd>

```c
FILE *file;
errno_t err = fopen_s(&file, "notexist.txt", "r");
// file is NULL, err is set.
```

<dt><b>Bad path</b>
<dd>

```c
FILE *file = fopen(NULL, NULL);
// file is NULL, errno is set. Same as sad path.
```

<dd>

```c
FILE *file;
errno_t err = fopen_s(&file, NULL, NULL);
// Constraint violated. Abort with message.
```

</dl>

<details><summary><sup>Yes, you can customize the constraint handler to just log to a file and continue on as though nothing happened.</sup></summary>

```c
set_constraint_handler_s(ignore_handler_s);
set_constraint_handler_s(abort_handler_s);
set_constraint_handler_s(my_awesome_handler);
```

</details>

Notice how the normal `fopen()` has the same return value (possibly different `errno`) to indicate different levels of bad-ness of errors? That's kinda what this `fopen_s()` was trying to improve. At least, that's my reading of it. I think of it like Rust's `panic!()` vs a returned `Result<String, std::io::Error>`. It also probably helps stop some buffer overflow attacks by providing `size_of_dest` arguments to avoid overflowing any `dest` buffers like `strcpy_s()` and `gets_s()`.

> ```c
> char* gets( char* str ); // (removed in C11)
> char* gets_s( char* str, rsize_t n ); // (since C11, annex K)
> ```
>
> Reads stdin into the character array pointed to by str until a newline character is found or end-of-file occurs. A null character is written immediately after the last character read into the array. The newline character is discarded but not stored in the buffer.
>
> The gets() function does not perform bounds checking, therefore this function is extremely vulnerable to buffer-overflow attacks. It cannot be used safely (unless the program runs in an environment which restricts what can appear on stdin). For this reason, the function has been deprecated in the third corrigendum to the C99 standard and removed altogether in the C11 standard. fgets() and gets_s() are the recommended replacements.
>
> WARNING: Never use gets().

```c
// BAD
char buffer[1000];
gets(buffer);
// ⚠️ Could write >1000 chars to `buffer`!
```


```c
// GOOD
char buffer[1000];
gets_s(buffer, sizeof(buffer));
// This will stop at 1000 chars.
```

## The problem

**They aren't implemented everywhere.** The `_s()` functions are an _extension_ that isn't available in libc implementations like glibc from GNU for lots of Linux stuff. There's other minor issues like it not being ergonomic for multithreading and the common mistake of doing `sizeof(src)` instead of `sizeof(dest)` for things like `strcpy_s()`, but that all pales in comparison to the availablity problem.

Most online information I can find seems to indicate that MSVC is the only major compiler/libc that has implemented Annex K.

Given that these fancy `_s()` functions aren't everywhere that your code needs to compile, why would you bother writing code like this:

```c
#ifdef __STDC_LIB_EXT1__
  printf_s("data results: %s, %d, %lu\n", a, b, c);
#else
  printf("data results: %s, %d, %lu\n", a, b, c);
#endif
```

...for _every instance_ that you want to do `strlen_s()` or `fopen_s()` or `strcpy_s()`. That's a good way to go insane.

So obviously you're not going to write platform-dependent code _just to do basic `printf()` and `strcpy()`_ but what about wrapping all that `#ifdef __STDC_LIB_EXT1__` `#else` stuff in a polyfill library?

There were two promising-looking libraries that I found via a quick Google search:

- [safec: Safe C Library website](https://rurban.github.io/safeclib/doc/safec-3.8/index.html) [GitHub page](https://github.com/rurban/safeclib) ⭐335
- [sbaresearch/slibc: Implementation of C11 Annex K "Bounds-checking interfaces" ISO/IEC 9899:2011](https://github.com/sbaresearch/slibc) ⭐14

So... if you want to (or are required to by security stuff) to use `_s()` functions but also don't want to limit yourself to just MSVC then you can use one of those ☝ libraries.

📚 For more reading check out [Field experience with Annex K (2015)](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1967.htm) and the [Bounds checking - cppreference.com](https://en.cppreference.com/w/c/error#Bounds_checking) technical documentation.
