---
title: Go make struct uncopyable
---

Well, it doesn't actually _prevent_ it; it just gives you a `go vet` error.

```go
package main

import "fmt"

type noCopy struct{}

func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}

type CannotBeCopied struct {
	noCopy noCopy
	Name   string
}

func main() {
	joeSmith := CannotBeCopied{Name: "Joe Smith"}
	joeSmithCopy := joeSmith
	fmt.Println(&joeSmith, &joeSmithCopy)
}
```

```
./main.go:17:18: assignment copies lock value to joeSmithCopy: CannotBeCopied contains noCopy

Go vet failed.

&{{} Joe Smith} &{{} Joe Smith}
```

The trick here is that `go vet` knows that `.Lock()` and `.Unlock()` mean that a value is ✨special and it probably should be passed by `*TheLockableType` instead of via copy. This was introduced to underline the fact that `sync.Mutex` and friends (which have `.Lock()` and `.Unlock()` methods) should probably be passed by pointer, not by copy.

We can use this `go vet` check to our advantage! If you have a type that you don't want anyone to copy (by accident) then use this trick!

> A current way to declare that a struct is not copyable is to add both a Lock and Unlock method, even if the method doesn't do anything. That will cause "go vet" to complain about attempts to copy the struct. If you don't want to expose the methods, you can instead add them to a zero-length unexported struct, and embed that struct in the exported one. See, e.g., https://play.golang.org/p/ovFyFQHzZFH (the playground will run "go vet" when you click on the "Run" button).

&mdash; ianlancetaylor in [golang/go#30450](https://github.com/golang/go/issues/30450#issuecomment-476848903)

The Go standard library uses this trick too. This is the current best way to make a struct pseudo-uncopyable.

https://github.com/search?q=repo%3Agolang%2Fgo%20noCopy&type=code
![image](/media/2025-01-24-001.png)
