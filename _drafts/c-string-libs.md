---
layout: post
title: Best C string libraries review & comparison
---

## The contenders

This isn't an exhaustive list.

- **[github.com/msteinert/bstring](https://github.com/msteinert/bstring):** A fork of bstring
- **[github.com/websnarf/bstrlib](https://github.com/websnarf/bstrlib):** The original bstring
- **[and.org/ustr](http://www.and.org/ustr/):**
- **[and.org/vstr](http://www.and.org/vstr/):**
- **[mibsoftware.com/libmib/astring](http://www.mibsoftware.com/libmib/astring/):**
- **[github.com/antirez/sds](https://github.com/antirez/sds):** The original sds
- **[github.com/jcorporation/sds](https://github.com/jcorporation/sds):** Updated fork of sds
- **[github.com/maxim2266/str](https://github.com/maxim2266/str):**
- **[github.com/mickjc750/str](https://github.com/mickjc750/str):**
- **[github.com/aheck/clib](https://github.com/aheck/clib):** Header-only tiny version of GLib

## Layouts

Length-first

```c
typedef struct my_string {
  size_t length;   // AKA "len" or "slen"
  char*  data;     // AKA "buf" or "str"
  size_t capacity; // AKA "cap", "alloc", or "mlen"
} my_string;
```

Pointer-last

```c
typedef struct my_string {
  size_t capacity; // AKA "cap", "alloc", or "mlen"
  size_t length;   // AKA "len" or "slen"
  char*  data;     // AKA "buf" or "str"
} my_string;
```

Pointer-first

```c
typedef struct my_string {
  char*  data;     // AKA "buf" or "str"
  size_t length;   // AKA "len" or "slen"
  size_t capacity; // AKA "cap", "alloc", or "mlen"
} my_string;
```

## Interoperability

The best C string libraries are those that are **minimal**. Some libraries add additional string information to the struct such as destructors, reference counting, reference vs. owned flags, etc. When passing strings across FFI boundaries you don't want to deal with that.

```rs
#[no_mangle]
pub extern "C" fn greet(name: ???) -> ??? {
  
}
```
