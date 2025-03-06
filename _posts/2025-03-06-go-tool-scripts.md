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

Because `go build ./...` will recursively crawl into `scripts/` but _it will ignore `_`/`.`-prefixed folders_ like `_scripts/`. `go install <module_path>/...@latest` will also ignore `_scripts/` too. You can still explicitly specify the package path which may include `_`/`.`-prefixed names but they aren't included in the default `./...` pattern.

_You could also use `_tools/` or whatever you want._

💡 This pattern works great for creating examples in a `_examples/` subdirectory similar to Rust's [`examples/` folder in the standard project layout](https://doc.rust-lang.org/cargo/guide/project-layout.html). You can still explicitly `go run ./_examples/<name>` but `go build ./...` and/or `go install <module_path>/...` won't install them.

