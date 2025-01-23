---
title: Get GOAMD64/GOARM/GO386 info like runtime.GOOS
---

You know how you can do `//go:build windows` but then also `if runtime.GOOS == "windows"`? What about the `runtime.GOOS` equivalent for `//go:build wasm.signext`?

```go
if runtime.GOARCH == "wasm" && ??? {
  fmt.Println("You're using WASM with signext")
} else if runtime.GOARCH == "wasm" {
  fmt.Println("You're using WASM but without signext)
} else {
  fmt.Println("You're not using WASM at all")
}
```

_Or any other `GO${GOARCH}` arch-specific Go settings like `GOARM=7` or `GO386=sse2` or `GOAMD64=v1` or whatever._

You'd think that you can't and that the only way to get that information would be to stash some `//go:build` information into a `const` like this:

<table><td>

```go
//go:build wasm.signext
package main
const TheMessage = "You're using WASM with signext"
```

<tr><td>

```go
//go:build wasm && !wasm.signext
package main
const TheMessage = "You're using WASM without signext"
```

<tr><td>

```go
//go:build !wasm
packag main
const TheMessage = "You're not using WASM at all"
```

</table>

...but that's tedious. There's a more runtime-y way to get that information: `runtime/debug.ReadBuildInfo()`

```go
bi, _ := debug.ReadBuildInfo()
var gowasm string
for _, bs := range bi.Settings {
  if bs.Key == "GOWASM" {
    gowasm = bs.Value
    break
  }
}

if runtime.GOARCH == "GOWASM" && slices.Contains(strings.Split(gowasm, ","), "signext") {
  fmt.Println("You're using WASM with signext")
} else if runtime.GOARCH == "wasm" {
  fmt.Println("You're using WASM but without signext)
} else {
  fmt.Println("You're not using WASM at all")
}
```

The key is the `BuildSetting` structs that are attached to the `BuildInfo` module info struct that we get from `ReadBuildInfo()`. https://pkg.go.dev/runtime/debug#BuildSetting
