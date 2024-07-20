---
layout: post
title: Put dev dependencies in tools.go
---

**TL;DR:** Use a `tools.go` file with `//go:build tools` to list `import _ "github.com/octocat/somedevtool"` to stop `go mod tidy` from removing them.

<div><code>task.go</code></div>

```go
//go:build tools

package tools

import (
	_ "github.com/melbahja/got/cmd/got"
	_ "github.com/arkady-emelyanov/go-shellparse"
)
```

<sup>💡 `go mod tidy` doesn't care if the package is cmd or lib</sup>

Then you can use whatever you list in `tools.go` in `//go:generate` and `task.go`!

```go
package mylib

//go:generate -command got go run github.com/melbahja/got/cmd/got

import (
  "fmt"
  _ "embed"
)
//go:generate got https://example.org/image.png -o image.png
//go:embed image.png
var imagePNG []byte
```

<div><code>task.go</code></div>

```go
//go:build ignore

package main

import "github.com/arkady-emelyanov/go-shellparse"

func main() {
  // ...
}
```

Further reading 🤓

- https://play-with-go.dev/tools-as-dependencies_go119_en/
- https://www.tiredsg.dev/blog/golang-tools-as-dependencies/