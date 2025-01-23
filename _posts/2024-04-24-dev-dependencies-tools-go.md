---
title: Put dev dependencies in tools.go
redirect_from: /blog/dev-dependencies-tools-go
---

**Problem:** You have a `./gen.go` script that uses `github.com/arkady-emelyanov/go-shellparse` to do some stuff. When you run `go mod tidy` it removes `github.com/arkady-emelyanov/go-shellparse` from your `go.mod` because `./gen.go` has a `//go:build ignore` in it... which means `go mod tidy` doesn't include it in its checks. 😔

<div><code>gen.go</code></div>

```go
//go:build ignore

package main

import (
	"github.com/arkady-emelyanov/go-shellparse"
    // etc.
)

func main() {
    shellparse.ParseVarsFile("file.txt")
    exec.Command("go", "run", "github.com/melbahja/got/cmd/got").Run()
}
```

**Solution:** Use `tools.go` with `//go:build tools` which `go mod tidy` _does_ scan and add any dependencies to list all the dependencies that `./gen.go` needs.

<div><code>tools.go</code></div>

```go
//go:build tools

package tools

import (
	_ "github.com/melbahja/got/cmd/got"
	_ "github.com/arkady-emelyanov/go-shellparse"
    // etc.
)
```

```sh
go run ./gen.go
```

Further reading 📚

- https://play-with-go.dev/tools-as-dependencies_go119_en/
- https://www.tiredsg.dev/blog/golang-tools-as-dependencies/
