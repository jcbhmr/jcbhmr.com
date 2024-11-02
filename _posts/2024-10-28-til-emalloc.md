---
layout: post
title: TIL emalloc() auto-exits on out-of-memory errors
redirect_from: /blog/til-emalloc
---

I was tired of writing this over and over:

```c
double* data = (double*)malloc(20 * sizeof(double));
if (data == NULL) {
  fputs("out of memory", stderr);
  abort();
}
```

And today I learned that there's a family of `e___()` functions like `emalloc()` and `ecalloc()` which will exit the process if the `malloc()`-returned pointer is `NULL`. These functions are only present in the BSD family of operating system's `util.h` system header. They aren't standard C functions.

```c
#include <util.h>

void (*)(int, const char *, ...) esetfunc(void (*)(int, const char *, ...));

int easprintf(char ** restrict str, const char * restrict fmt, ...);

FILE * efopen(const char *p, const char *m);

void * ecalloc(size_t n, size_t s);

void * emalloc(size_t n);

void * erealloc(void *p, size_t n);

void ereallocarr(void *p, size_t n, size_t s);

char * estrdup(const char *s);

char * estrndup(const char *s, size_t len);

size_t estrlcat(char *dst, const char *src, size_t len);

size_t estrlcpy(char *dst, const char *src, size_t len);

intmax_t estrtoi(const char * nptr, int base, intmax_t lo, intmax_t hi);

uintmax_t estrtou(const char * nptr, int base, uintmax_t lo, uintmax_t hi);

int evasprintf(char ** restrict str, const char * restrict fmt, ...);
```

> The `easprintf()`, `efopen()`, `ecalloc()`, `emalloc()`, `erealloc()`, `ereallocarr()`, `estrdup()`, `estrndup()`, `estrlcat()`, `estrlcpy()`, `estrtoi()`, `estrtou()`, and `evasprintf()` functions operate exactly as the corresponding functions that do not start with an `e` except that in case of an error, they call the installed error handler that can be configured with `esetfunc()`.
>
> For the string handling functions, it is an error when the destination buffer is not large enough to hold the complete string. For functions that allocate memory or open a file, it is an error when they would return a null pointer.  The default error handler is `err()`. The function `esetfunc()` returns the previous error handler function. A `NULL` error handler will just call `exit()`.

&mdash; [emalloc(3) - NetBSD Manual Pages](https://man.netbsd.org/emalloc.3)

These functions are relatively simple. You can easily implement them yourself.

```c
// I don't know if static or inline is better.
inline void * emalloc(size_t n) {
  void *p = malloc(n);
  if (p == NULL) {
    fputs("out of memory", stderr);
    abort();
  }
  return p;
}

// Do the same wrapper thing for all the other functions.
```

I just thought it was cool that this was a thing that is _built into an operating system's C standard library_. That's batteries included for sure.

## What do other languages do on out-of-memory errors?

- Java: Throw an exception
- C++: Throw an exception
- Rust: Abort or panic
- JavaScript: Crash the engine
- Python: Throws a special exception
- Go: Terminate

And then there's C: Return a null pointer. 🤷‍♀️ Great if you need that control, a hassle if you want the happy path.
