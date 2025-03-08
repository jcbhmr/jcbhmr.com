---
title: "//go:linkname list for Go 1.24"
---

## What is `//go:linkname`?

TLDR it's a way to expose & link to specific symbols from other packages in an abnormal way.

In the case of this list, all of the `//go:linkname <private_thing>` directives are to expose select internal Go standard library/runtime functions to non-local Go code (other packages or other modules even outside the standard library). In https://go.dev/issue/67401 it was determined that only a subset of Go internal symbols would be exposed to non-local Go code. That's what this list is; a list of all those "this is something we'd like to be private but we can't make it so without breaking popular Go packages" `//go:linkname` exposure directives.

This is mostly a list for myself so that I don't have to mess with search tools. Instead, I have a complete list in one place (until that list becomes outdated).

## The list

💡 Tip: Use Ctrl+F to search through this list.

```go
import (
  _ "runtime"
  _ "unsafe"
)

//go:linkname iscgo runtime.iscgo
var iscgo bool

//go:linkname set_crosscall2 runtime.set_crosscall2
var set_crosscall2 func()

//go:linkname makemap_small runtime.makemap_small
func makemap_small() *maps.Map

//go:linkname makemap runtime.makemap
func makemap(t *abi.SwissMapType, hint int, m *maps.Map) *maps.Map

//go:linkname mapaccess2 runtime.mapaccess2
func mapaccess2(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer) (unsafe.Pointer, bool)

//go:linkname mapassign runtime.mapassign
func mapassign(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer) unsafe.Pointer

//go:linkname mapdelete runtime.mapdelete
func mapdelete(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer)

//go:linkname getitab runtime.getitab
func getitab(inter *interfacetype, typ *_type, canfail bool) *itab

//go:linkname convT64 runtime.convT64
func convT64(val uint64) (x unsafe.Pointer)

//go:linkname convTstring runtime.convTstring
func convTstring(val string) (x unsafe.Pointer)

//go:linkname convTslice runtime.convTslice
func convTslice(val []byte) (x unsafe.Pointer)

//go:linkname mapiternext runtime.mapiternext
func mapiternext(it *linknameIter)

//go:linkname reflect_makemap reflect.makemap
func reflect_makemap(t *abi.SwissMapType, cap int) *maps.Map

//go:linkname reflect_mapaccess reflect.mapaccess
func reflect_mapaccess(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer) unsafe.Pointer

//go:linkname reflect_mapassign0 reflect.mapassign0
func reflect_mapassign0(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer, elem unsafe.Pointer)

//go:linkname reflect_maplen reflect.maplen
func reflect_maplen(m *maps.Map) int

//go:linkname reflect_mapiterinit reflect.mapiterinit
func reflect_mapiterinit(t *abi.SwissMapType, m *maps.Map, it *linknameIter)

//go:linkname reflect_mapiternext reflect.mapiternext
func reflect_mapiternext(it *linknameIter)

//go:linkname reflect_mapiterkey reflect.mapiterkey
func reflect_mapiterkey(it *linknameIter) unsafe.Pointer

//go:linkname reflect_mapiterelem reflect.mapiterelem
func reflect_mapiterelem(it *linknameIter) unsafe.Pointer

//go:linkname cgocall runtime.cgocall
func cgocall(fn, arg unsafe.Pointer) int32

//go:linkname mapaccess2_fast32 runtime.mapaccess2_fast32
func mapaccess2_fast32(t *abi.SwissMapType, m *maps.Map, key uint32) (unsafe.Pointer, bool)

//go:linkname mapassign_fast32 runtime.mapassign_fast32
func mapassign_fast32(t *abi.SwissMapType, m *maps.Map, key uint32) unsafe.Pointer

//go:linkname mapassign_fast32ptr runtime.mapassign_fast32ptr
func mapassign_fast32ptr(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer) unsafe.Pointer

//go:linkname mapaccess2_fast64 runtime.mapaccess2_fast64
func mapaccess2_fast64(t *abi.SwissMapType, m *maps.Map, key uint64) (unsafe.Pointer, bool)

//go:linkname mapassign_fast64 runtime.mapassign_fast64
func mapassign_fast64(t *abi.SwissMapType, m *maps.Map, key uint64) unsafe.Pointer

//go:linkname mapassign_fast64ptr runtime.mapassign_fast64ptr
func mapassign_fast64ptr(t *abi.SwissMapType, m *maps.Map, key unsafe.Pointer) unsafe.Pointer
```

```go
import (
  _ "math"
  _ "unsafe"
)

//go:linkname Float32bits
func Float32bits(f float32) uint32
```

```go
```
