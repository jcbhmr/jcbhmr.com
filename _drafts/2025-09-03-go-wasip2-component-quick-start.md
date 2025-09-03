---
title: Go wasip2 component quick start guide
---

The end goal:

```sh
wasmtime --invoke 'greet("Alan Turing")' ./.out/hello-world.wasm
```

```
Hello, Alan Turing!
```

### 1. Create your Go project

```sh
mkdir ./hello-world/
go mod init github.com/octocat/hello-world
```

<details><summary>Current file tree</summary>

```
TODO
```

</details>

### 2. Define your WIT interface definitions

<div><code>wit/package.wit</code></div>

```wit
package octocat:hello-world@0.1.0;

interface greet {
    /// Prints a greeting to the console.
    greet(name: string);
}

world hello-world {
    include wasi:cli/imports@0.2.0;
    export greet;
}
```

### 3. Run `wit-bindgen-go generate`

This will turn the WIT we created in step #2 into Go `//go:wasmimport` and `//go:wasmexport` bindings.

```sh
go get -tool go.bytecodealliance.org/cmd/wit-bindgen-go
go tool wit-bindgen-go generate --out ./internal/ ./wit/
```

You should add this "generate bindings" step to a `helloworld_test.go` file as a `//go:generate` command so that you can `go generate` instead of remembering that long `wit-bindgen-go` invocation.

<div><code>helloworld_test.go</code></div>

```go
// TODO: Find a cross-platform 'rm -rf' Go tool
//go:generate rm -rf ./internal/
//go:generate go tool wit-bindgen-go generate --out ./internal/ ./wit/

package helloworld_test
```

### 4. Centralize the generated `wasi/...` bindings types

Why? You don't want to expose a locally-defined WASI binding type like `wasi:io/streams`'s `input-stream` type and not allow the importer to refer to the type name. Such a locally-defined type is also not the same as another module's own locally-defined `input-stream` type that it might want as a parameter. It's a shared type, so it should have a well-known name. [github.com/jcbhmr/go-wasi/io/streams.InputStream](https://pkg.go.dev/github.com/jcbhmr/go-wasi/io/streams#InputStream) is that well-known name. This doesn't apply to this guide since we aren't exposing any WASI types (it's just `greet(name)`).

Install a text-replacer CLI like [github.com/NicoNex/jet](https://pkg.go.dev/github.com/NicoNex/jet) to replace all instances of `github.com/octocat/hello-world/internal/wasi/` with `github.com/jcbhmr/go-wasi/`.

```sh
go get github.com/jcbhmr/go-wasi@v0.2.0
go get -tool github.com/NicoNex/jet
```

Then add this postprocessing step to your `//go:generate` script.

```diff
 // TODO: Find a cross-platform 'rm -rf' Go tool
 //go:generate rm -rf ./internal/
 //go:generate go tool wit-bindgen-go generate --out ./internal/ ./wit/
+//go:generate go tool jet "github\\.com/octocat/hello-world/internal/wasi/" "github.com/jcbhmr/go-wasi" ./
+//go:generate rm -rf ./internal/wasi/

 package helloworld_test
```

_In the future there might be a `golang.org/x/sys/wasi` or `syscall/wasi` or something instead of `github.com/jcbhmr/go-wasi`. Who knows what will happen._

Now you can run `go generate` again to regenerate everything with this new postprocessing step.

### 4. Write your functions to the exports structure

Make sure you `//go:build wasip2`; you don't want to use WASI imports when compiling for `windows/amd64`.

```go
//go:build wasip2

package main

import (
    "fmt"

    "github.com/octocat/hello-world/internal/octocat/hello-world/greet"
)

func init() {
    greet.Exports.Greet = func(name string) {
        fmt.Printf("Hello, %s!\n", name)
    }
}

func main() {}
```

### 5. Build it using TinyGo

TinyGo natively supports WASI Preview 2. Let's use it.

```sh
GOOS=wasip2 GOARCH=wasm tinygo build \
    -wit-package ./wit/ -wit-world hello-world \
    -o ./.out/hello-world.wasm
```

💡 You can put this all into a `_scripts/build/main.go` or similar if you want to avoid retyping such a long command every time you want to build your project.

### 6. Run your WASM component using Wasmtime

```sh
wasmtime --invoke 'greet("Alan Turing")' ./.out/hello-world.wasm
```

```
Hello, Alan Turing!
```
