---
layout: post
title: Best C string libraries review & comparison
---

**"C strings" aren't that good.** As soon as you start dealing with strings that may contain interior nuls things get messy.

<div><code>print_test_wasm.c</code></div>

```c
FILE *file = fopen("test.wasm", "rb");
fseek(file, 0, SEEK_END);
long file_len = ftell(file);
fseek(file, 0, SEEK_SET);
char *contents = (char *)malloc(file_len + 1);
fread(contents, 1, file_len, file);
buffer[contents] = '\0';
// Every WASM file starts with "\0asm". This will print nothing.
puts(contents);
```

The obvious solution is to store the length of the string with the string `char*` in some kind of struct. Then you can use that length instead of the `\0` nul terminator to tell the computer how long the string is.

```c
typedef struct my_string {
  size_t length;
  char* data;
} my_string;

int my_string_puts(my_string str) {
  fwrite(str.data, 1, str.length, stdout);
}
```

But now you've started down the rabbit hole of writing your own string library. Should you include a larger-than-needed self-reallocating buffer for resizing the string if it's mutated? Should you put the length first or the pointer first? Should it be reference counted? What about an immutable view into an existing string? Maybe you want to go through all of that and create your own C string library. Maybe you don't. 🤷‍♀️

Lucky for you, there are numerous C string libraries out there. But which one should you use? 🤔

**The contenders**

This isn't an exhaustive list. This is just what I could come up with in 15 minutes of surfing the web.

1. **[github.com/msteinert/bstring](https://github.com/msteinert/bstring):** A fork of bstring
2. **[github.com/websnarf/bstrlib](https://github.com/websnarf/bstrlib):** The original bstring
3. **[and.org/ustr](http://www.and.org/ustr/):** Comprehensive basic string library
4. **[github.com/antirez/sds](https://github.com/antirez/sds):** The original sds
5. **[github.com/jcorporation/sds](https://github.com/jcorporation/sds):** Updated fork of sds
6. **[github.com/maxim2266/str](https://github.com/maxim2266/str):** Another C string library
7. **[github.com/mickjc750/str](https://github.com/mickjc750/str):** C string library
8. **[github.com/aheck/clib](https://github.com/aheck/clib):** Header-only tiny version of GLib

Honestly they all kinda blend together name-wise. Naming things really is one of the hardest problems in computer science. 😆

**TL;DR table**

|                                                                      | Struct style                 | Literal? | Ownership extras? | Ease of installation | Ease of use | Popularity | License      |
| -------------------------------------------------------------------- | ---------------------------- | -------- | ----------------- | -------------------- | ----------- | ---------- | ------------ |
| [github.com/msteinert/bstring](https://github.com/msteinert/bstring) | Pointer-last                 | ✅       | ❌                | Fair                 | Good        | ⭐134      | BSD-3-Clause |
| [github.com/websnarf/bstrlib](https://github.com/websnarf/bstrlib)   | Pointer-last                 | ✅       | ❌                | Fair                 | Good        | ⭐497      | BSD-3-Clause |
| [and.org/ustr](http://www.and.org/ustr/)                             | Witchcraft bit magic         | ✅       | Everything        | Poor                 | Fair        | 🔎213      | MIT          |
| [github.com/antirez/sds](https://github.com/antirez/sds)             | Before-pointer bit magic     | ❌       | ❌                | Great                | Great       | ⭐4.9k     | BSD-2-Clause |
| [github.com/jcorporation/sds](https://github.com/jcorporation/sds)   | Before-pointer bit magic     | ❌       | ❌                | Great                | Great       | ⭐40       | BSD-2-Clause |
| [github.com/maxim2266/str](https://github.com/maxim2266/str)         | Pointer-first & bit magic    | ✅       | `is_ref` flag     | Best                 | Great       | ⭐289      | BSD-3-Clause |
| [github.com/mickjc750/str](https://github.com/mickjc750/str)         | Pointer-last & pointer-first | ✅       | `strview`         | Great                | Great       | ⭐240      | Unlicense    |
| [github.com/aheck/clib](https://github.com/aheck/clib)               | Pointer-first                | ❌       | ❌                | Great                | Great       | ⭐89       | MIT          |



## How do the structs compare?

<table><td valign=top>

1\. [github.com/msteinert/bstring](https://github.com/msteinert/bstring)

```c
struct tagbstring {
  int mlen;
  int slen;
  unsigned char *data;
};
typedef struct tagbstring *bstring;
```

<td valign=top>

2\. [github.com/websnarf/bstrlib](https://github.com/websnarf/bstrlib)

```c
struct tagbstring {
  int mlen;
  int slen;
  unsigned char *data;
};
typedef struct tagbstring *bstring;
```

<tr><td valign=top>

3\. [and.org/ustr](http://www.and.org/ustr/)

```c
struct Ustr {
  // Some weird bit packing magic 😵
};
```

<td valign=top>

4\. [github.com/antirez/sds](https://github.com/antirez/sds)

```c
// sdshdr5, sdshdr8, sdshdr16, sdshdr32, or sdshdr64
// [sdshdrN] [char...] [\0]
//           ^ pointer returned from sdsnewlen()
struct __attribute__ ((__packed__)) sdshdr32 {
  uint32_t len;
  uint32_t alloc;
  unsigned char flags;
  char buf[];
};
typedef char *sds;
```

<tr><td valign=top>

5\. [github.com/jcorporation/sds](https://github.com/jcorporation/sds)

```c
// sdshdr5, sdshdr8, sdshdr16, sdshdr32, or sdshdr64
// [sdshdrN] [char...] [\0]
//           ^ pointer returned from sdsnewlen()
struct __attribute__ ((__packed__)) sdshdr32 {
  uint32_t len;
  uint32_t alloc;
  unsigned char flags;
  char buf[];
};
typedef char *sds;
```

<td valign=top>

6\. [github.com/maxim2266/str](https://github.com/maxim2266/str)

```c
typedef struct {
  const char *ptr;
  size_t info;
} str;
```

<tr><td valign=top>

7\. [https://github.com/mickjc750/str](https://github.com/mickjc750/str)

```c
typedef struct strbuf_t {
  int size;
  int capacity;
  strbuf_allocator_t allocator;
  char cstr[];
} strbuf_t;
```

<td valign=top>

8\. [github.com/aheck/clib](https://github.com/aheck/clib)

```c
typedef struct GString {
  char *str;
  size_t len;
  size_t allocated_len;
} GString;
```

</table>

| | Struct style | Literal? | Reference counted? | Read-only struct? | How to get? | Popularity |


## Interoperability

The best C string libraries are those that are **minimal**. Some libraries add additional string information to the struct such as destructors, reference counting, reference vs. owned flags, etc. When passing strings across FFI boundaries you don't want to deal with that.

```rs
#[no_mangle]
pub extern "C" fn greet(name: ???) -> ??? {
  
}
```
