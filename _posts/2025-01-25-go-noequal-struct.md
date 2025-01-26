---
title: Make a Go struct un-equal-able
---

The trick is to use the fact that any struct that has a `func()` of any kind **cannot be used in `==` expressions**. This means that if you include a zero-length array (remember Go has arrays in addition to slices) of `func()` it will still disable `==` comparisons without taking up any additional space in your struct.

```go
package main

import "fmt"

type Equalable struct {
	Name string
}

type NotEqualable struct {
	_    [0]func()
	Name string
}

func main() {
	eA := Equalable{Name: "Alan Turing"}
	eB := Equalable{Name: "Alan Turing"}
	fmt.Println(eA == eB)

	neA := NotEqualable{Name: "Ada Lovelace"}
	neB := NotEqualable{Name: "Ada Lovelace"}
	// fmt.Println(neA == neB)
	// error: invalid operation: neA == neB (struct containing [0]func() cannot be compared)
	_ = neA
	_ = neB
}
```

https://go.dev/play/p/pwao1PGSlP-

This same trick also works with Go `map` types. `func(...)` and `map[...]` types cannot be compared using `==` and `!=` value-based comparisons.
