---
title: How I did Go abstract classes one time
---

The trick is an `X__this` pointer to the top of your inheritance chain stored at the bottom of the inheritance hierarchy.

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

type path string
type PlatformDirs interface {
	UserCacheDir() string
	UserCachePath() path
	isPlatformDirs()
}
type PlatformDirsImpl struct {
	X__this   PlatformDirs // topmost impl to dyn dispatch to
	appname   string
	appauthor string
}

func NewPlatformDirs(appname string, appauthor string) *PlatformDirsImpl {
	p := &PlatformDirsImpl{
		appname:   appname,
		appauthor: appauthor,
	}
	p.X__this = p
	return p
}
func (p *PlatformDirsImpl) UserCacheDir() string {
	panic("abstract")
}
func (p *PlatformDirsImpl) UserCachePath() path {
	return path(p.X__this.(PlatformDirs).UserCacheDir())
}
func (p *PlatformDirsImpl) isPlatformDirs() {}

type Windows interface {
	PlatformDirs
	isWindows()
}
type WindowsImpl struct {
	PlatformDirsImpl
}

func NewWindows(appname string, appauthor string) *WindowsImpl {
	w := &WindowsImpl{*NewPlatformDirs(appname, appauthor)}
	w.X__this = w
	return w
}
func (p *WindowsImpl) UserCacheDir() string {
	d, err := os.UserHomeDir()
	if err != nil {
		panic(err)
	}
	return filepath.Join(d, "the-cache-folder", p.appauthor, p.appname)
}
func (p *WindowsImpl) isWindows() {}

func main() {
	w := NewWindows("MyApp", "MyCompany")
	fmt.Println(w.UserCachePath())
	// Output: /root/the-cache-folder/MyCompany/MyApp
}
```

https://go.dev/play/p/a3ijyB9h77-

`X__this` can be any implementation of the root interface (which every child class will be) and then the parent class can call all child methods (which **can be overridden**) _on that `X__this` pointer_. I think that's cool. There are a few caveats though:

1. You have to store an interface pointer which has a size and performance cost
2. This requires parallel interface and struct hierarchies that intertwine
4. This requires the user to make sure they don't do unexpected things with the returned instance's pointer
5. You have to manually set the `X__this` pointer to the topmost implementation when you construct or compose it
6. `X__this` must be public (hence the `X__` prefix) if you want to extend the class outside the current package
7. The `ThingImpl` struct must be public so it is embedded publicly in child classes

But you finally get dynamic dispatch on child-overridden methods!
