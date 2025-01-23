---
title: Use task.go for your Go project scripts
redirect_from: /blog/task-go-project-scripts
---

**TL;DR:** `go run task.go <task_name>` makes your scripts cross-platform.

```go
//go:build ignore

package main

import (
	"log"
	"os"
	"os/exec"
)

func BuildEverything() error {
	// Just write normal Go code!
    
    // Run some commands
    // Copy some assets
    // Create an installer maybe
    // Upload something to GitHub
    // Send an email
    // Do whatever!

    return nil
}

func ConfigureEnvironment() error {
	// Prefer Go over Bash in Go projects.
    
    // Load some env values from a secret server
    // Save them to ~/.bashrc
    // Create a .env file
    // Install some global CLI dependencies
    // Make sure Python is available...
    // More magic.

    return nil
}

func main() {
	log.SetFlags(0)
	var taskName string
	if len(os.Args) >= 2 {
		taskName = os.Args[1]
	} else {
		log.Fatal("no task")
	}
	tasks := map[string]func() error{
		"build-everything": BuildEverything,
        "configure-environment": ConfigureEnvironment,
	}
	task, ok := tasks[taskName]
	if !ok {
		log.Fatal("no such task")
	}
	err := task()
	if err != nil {
		log.Fatal(err)
	}
}
```

```sh
go run task.go setup
```

🤩 It's all just Go code! There's no confusing Bash-isms. \
🚀 It's just a template! Make `task.go` fit your needs. \
☝ It's all in a **single file**; there's no `task/main.go` sub-package stuff. \
✅ `//go:build ignore` still works with Gopls and intellisense. \
[📦 Use `tools.go` if you need `task.go`-only dependencies](https://dev.to/jcbhmr/put-dev-dependencies-in-toolsgo-9e6) \
😎 Runs wherever Go does; no more GNU `bash`-specific Makefile. \
💡 Inspired by [matklad/cargo-xtask](https://github.com/matklad/cargo-xtask)

Start by creating a `task.go` file in the root of your project. This is where you will define all the tasks that you want to run with `go run task.go`. The basic template for `task.go` is this:

<div><code>tools.go</code></div>

```go
//go:build ignore

package main

import (
	"log"
	"os"
	// Your imports here!
)

func Setup() error {
	// Your code here!
	return nil
}

func main() {
	log.SetFlags(0)
	var taskName string
	if len(os.Args) >= 2 {
		taskName = os.Args[1]
	} else {
		log.Fatal("no task")
	}
	tasks := map[string]func() error{
		"setup": Setup,
		// Add more tasks here!
	}
	task, ok := tasks[taskName]
	if !ok {
		log.Fatal("no such task")
	}
	err := task()
	if err != nil {
		log.Fatal(err)
	}
}
```

There's some more in-depth examples below 👇

Then you can run your `task.go` tasks like this:

```sh
go run task.go <task_name>
```

## How does this work with other `.go` files?

That's where the special `//go:build ignore` comes in! When you use `go run` Go will completely disregard all `//go:build` conditions in that file even if it requires a different operating system. We can use this fact to conditionally include the `task.go` file in normal Go operations only when the `-tags ignore` tag is set (which is should never be). Then we can bypass that `-tags ignore` requirement using `go run` to discard the `//go:build ignore` directive and run the file anyway! Tada! 🎉 Now we have a `task.go` file which can only be run **directly** and isn't included in your normal Go library or binary.

## Use `tools.go` for task dependencies

`task.go` can use all your local `go.mod` dependencies. The problem is that `go mod tidy` doesn't see `task.go` (since it's `//go:build ignore`-ed) and thus will remove any `task.go`-only dependencies. To combat this, you are encouraged to adopt the `tools.go` pattern:

<div><code>tools.go</code></div>

```go
//go:build tools

package tools

import (
	_ "github.com/vektra/mockery/v2"
	_ "github.com/example/shellhelper"
	_ "github.com/octocat/iamawesome"
)
```

📚 https://play-with-go.dev/tools-as-dependencies_go119_en/
📚 https://www.tiredsg.dev/blog/golang-tools-as-dependencies/

## `./task.go <task_name>` with a shebang

💡 If you're smart you can add a shebang-like line to the top of your `task.go` file to allow you to do `./task.go <task_name>` instead of `go run task.go <task_name>`.

<div><code>task.go</code></div>

```go
//usr/bin/true; exec go run "$0" "$@"

// ...
```

```sh
chmod +x task.go
./task.go <task_name>
```

[📚 What's the appropriate Go shebang line?](https://stackoverflow.com/questions/7707178/whats-the-appropriate-go-shebang-line)

Go doesn't support the `#!` shebang comment so we have to use the fact that when a file is `chmod +x`-ed and doesn't have a `#!` at the top it just runs with the default system shell. The `//` line doubles as a comment for Go and a command for the shell. 👩‍💻

[_⚠️ This is officially discouraged by the Go team._](https://github.com/golang/go/issues/24118)

## Dev HTTP server

Sometimes you just need a `go run task.go serve` command to spin up an HTTP server.

<div><code>task.go</code></div>

```go
//go:build ignore

package main

import (
	"log"
	"net/http"
	"os"
)

func Serve() error {
	dir := "."
	port := "8000"
	log.Printf("Serving %#v at http://localhost:%s\n", dir, port)
	return http.ListenAndServe(":"+port, http.FileServer(http.Dir(dir)))
}

func main() {
	log.SetFlags(0)
	var taskName string
	if len(os.Args) >= 2 {
		taskName = os.Args[1]
	} else {
		log.Fatal("no task")
	}
	tasks := map[string]func() error{
		"serve": Serve,
		// Add more tasks here!
	}
	task, ok := tasks[taskName]
	if !ok {
		log.Fatal("no such task")
	}
	err := task()
	if err != nil {
		log.Fatal(err)
	}
}
```

```sh
go run task.go serve
```

## Using `task.go` with `//go:generate`

You may use `task.go` as a hub for ad-hoc `//go:generate` needs that go beyond one or two commands. It centralizes all your logic in one spot which can be good or bad. 🤷‍♀️

```go
//go:generate go run ../task.go generate:download-all-files
```

```go
//go:generate go run ./task.go fetch-and-extract-latest-release
```

```go
//go:generate go run ../../task.go build-assets
```

You can use `generate:<task_name>` or `generate-<task_name>` as a prefix if you want; it's all up to you and your project's needs.

💡 You might prefer to put your generate script in a `./gen.go` **right next to the caller file** using the same `//go:build ignore` trick. That way your niche `go generate` script doesn't bubble all the way up to the root `task.go`. It's completely your choice.

## Custom build script

When you have a lot of binaries to build and a lot of flags to provide to the `go build` command it might be nice to abstract those behind a `go run task.go build` script.

<div><code>task.go</code></div>

```go
//go:build ignore

package main

import (
	"log"
	"os"
	"os/exec"
)

func cmdRun(name string, arg ...string) error {
	cmd := exec.Command(name, arg...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	log.Printf("$ %s\n", cmd.String())
	return cmd.Run()
}

func Build() error {
	err := cmdRun("go", "build", "-o", ".out/", "-tags", "embed,nonet,purego", "./cmd/tool-one")
	if err != nil {
		return err
	}
	err = cmdRun("go", "build", "-o", ".out/", "-tags", "octokit,sqlite", "./cmd/tool-two")
	if err != nil {
		return err
	}
	// ...
	return nil
}

func main() {
	log.SetFlags(0)
	var taskName string
	if len(os.Args) >= 2 {
		taskName = os.Args[1]
	} else {
		log.Fatal("no task")
	}
	tasks := map[string]func() error{
		"build": Build,
		// Add more tasks here!
	}
	task, ok := tasks[taskName]
	if !ok {
		log.Fatal("no such task")
	}
	err := task()
	if err != nil {
		log.Fatal(err)
	}
}
```

```sh
go run task.go build
```

## Setup script to install global dependencies

Sometimes you want your contributors to have global dependencies installed. Yes, it's not ideal but it's often unavoidable. Providing collaborators with one single `go run task.go setup` command that automagically ✨ installs all required `zlib`, `libgit2`, `golangci-lint`, etc. is an amazing onboarder.

<div><code>task.go</code></div>

```go
//go:build ignore

package main

import (
	"log"
	"os"
	"os/exec"
)

func cmdRun(name string, arg ...string) error {
	cmd := exec.Command(name, arg...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	log.Printf("$ %s\n", cmd.String())
	return cmd.Run()
}

func Setup() error {
	return cmdRun("go", "install", "github.com/golangci/golangci-lint/cmd/golangci-lint")
}

func main() {
	log.SetFlags(0)
	var taskName string
	if len(os.Args) >= 2 {
		taskName = os.Args[1]
	} else {
		log.Fatal("no task")
	}
	tasks := map[string]func() error{
		"setup": Setup,
		// Add more tasks here!
	}
	task, ok := tasks[taskName]
	if !ok {
		log.Fatal("no such task")
	}
	err := task()
	if err != nil {
		log.Fatal(err)
	}
}
```

```sh
go run task.go setup
```

💡 You can even use `if runtime.GOOS == "windows"` or similar to do things for specific GOOS/GOARCH configurations!

## Still not convinced?

At least try to write your scripts in Go instead of Bash or Makefiles. This makes it more portable to more places (such as Windows 🙄) without confusion. It also means that your Go devs don't need to learn Bash-isms to change the scripts! 😉

Lots of existing Go projects make use of the `go run file.go` technique already; they just haven't taken the leap to combine all their scripts into a single `Makefile`-like `task.go` yet.

- https://github.com/mmcgrana/gobyexample/blob/master/tools/serve.go
- https://github.com/google/wuffs/blob/main/script/crawl.go
- https://github.com/syncthing/syncthing/blob/main/script/weblatedl.go
- https://github.com/gcc-mirror/gcc/blob/master/libgo/go/net/http/triv.go
- https://github.com/google/syzkaller/blob/master/pkg/csource/gen.go
- https://github.com/photoprism/photoprism/blob/develop/internal/maps/gen.go
- https://github.com/go-swagger/go-swagger/blob/master/hack/print_ast/main.go

You may prefer scripts folder so that you run each script individually like `go run scripts/build-all.go` instead of a `go run task.go build-all` AiO `task.go` file and **that's OK!** It's still better than a Linux-only Makefile. 😉

Also check out [Scripts should be written using the project main language](https://joaomagfreitas.link/scripts-should-be-written-using-the-project-main-language/) by [João Freitas](https://joaomagfreitas.link/) who hits on these points for more languages besides Go.

---

<sup>Do you have a cool use of `task.go`? Send it to me! ❤️🤩</sup>
