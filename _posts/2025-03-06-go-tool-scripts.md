---
title: Go project scripts with "go tool"
---

`go tool` is neat. It lets you do things like this:

```sh
go get -tool golang.org/x/tools/cmd/stringer
go tool stringer # Run it! 🚀
```

But did you know that the `tool <main_package>` directive in `go.mod` also works with _local_ packages too?

<div><code>go.mod</code></div>

```sh
module github.com/octocat/awesome

go 1.24

tool github.com/octocat/awesome/_scripts/build-release
```

<div><code>_scripts/build-release/main.go</code></div>

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World!")
  // 👩‍💻 exec.Command("go", "build", ...) etc. goes here
}
```

And then you can run it with `go tool build-release`. Pretty cool, right?

This makes it easier than ever to follow [Scripts should be written using the project main language](https://joaomagfreitas.link/scripts-should-be-written-using-the-project-main-language/).

### Why use `_scripts/` instead of `scripts/`?

Because `go build ./...` will recursively crawl into `scripts/` but it seems to ignore `_`/`.`-prefixed folders like `_scripts/`. You can still explicitly specify the package path which may include `_`/`.`-prefixed names but they aren't included in the default `./...` pattern.
