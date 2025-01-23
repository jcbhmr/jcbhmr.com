---
title: Fix "cannot take address of {value}" in Go
redirect_from: /blog/cannot-take-address-of-value-go
---

<sup>This is a summary of my Googling for future me or anyone else.</sup>

```go
fmt.Println(&"Hello")
fmt.Println(&true)
fmt.Println(&15)
fmt.Println(&fmt.Sprint(123))
```

```
./prog.go:8:15: invalid operation: cannot take address of "Hello" (untyped string constant)
./prog.go:9:15: invalid operation: cannot take address of true (untyped bool constant)
./prog.go:10:15: invalid operation: cannot take address of 15 (untyped int constant)
./prog.go:11:15: invalid operation: cannot take address of fmt.Sprint(123) (value of type string)
```

You currently can't get the address of literals or function return values in Go. You _can_, however, get the address of function parameters which means you can create a helper `ptr()` function to turn any value into a pointer. 😉

```go
func ptr[T any](value T) *T {
  return &value
}
```

```go
func ptr[T any](value T) *T {
  return &value
}
func main() {
  fmt.Println(ptr("Hello"))
  fmt.Println(ptr(true))
  fmt.Println(ptr(15))
  fmt.Println(ptr(fmt.Sprint(123)))
}
```

Of course, you could always stuff these values into a variable and `&theVariable`. 🙄 But who wants to clutter their code with extra one-time intermediary variables?

```go
a := "Hello"
fmt.Println(&a)
b := true
fmt.Println(&b)
c := 15
fmt.Println(&c)
d := fmt.Sprint(123)
fmt.Println(&d)
```

Further reading 📚

- [proposal: Go 2: make constants and literals addressable · Issue #28681 · golang/go](https://github.com/golang/go/issues/28681)
- [proposal: spec: add &T(v) to allocate variable of type T, set to v, and return address · Issue #9097 · golang/go](https://github.com/golang/go/issues/9097)
- [proposal: expression to create pointer to simple types · Issue #45624 · golang/go](https://github.com/golang/go/issues/45624)
- [go - Golang - Cannot take address of variable in struct error, untyped string constant - Stack Overflow](https://stackoverflow.com/questions/76238601/golang-cannot-take-address-of-variable-in-struct-error-untyped-string-constan)
- [pointers - Find address of constant in go - Stack Overflow](https://stackoverflow.com/questions/35146286/find-address-of-constant-in-go)
- [pointers - Reference to string literals in Go - Stack Overflow](https://stackoverflow.com/questions/11088967/reference-to-string-literals-in-go)
- [Assigning values to pointer types in a go struct - Stack Overflow](https://stackoverflow.com/questions/71012974/assigning-values-to-pointer-types-in-a-go-struct)

> Judging from the past 1.5 years, I appear to be writing this function about once every second month, when I need it in a new package. The need especially arise with pointers to strings in unit test files, I've noticed.
> 
> Admittedly, I work a lot with code generated from API specifications. That code tend to use *string a lot, since it can't be certain that nil and empty string are equivalent (and rightly so, they aren't).
> 
> It's not very annoying, but does feel a bit like I'm littering my packages with this function, so not having to write it would be welcome. I do realise I can put it in a package I import, but that also seems overkill for a one-liner.

&mdash; [@perj](https://github.com/perj) in [golang/go#45624 (comment)](https://github.com/golang/go/issues/45624#issuecomment-1886680316)
