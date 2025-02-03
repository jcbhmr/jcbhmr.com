---
title: Some popular conventions that I've observed
---

#### What name for the `rust-toolchain.toml` file?

- ⭐ `rust-toolchain.toml` [8.6k](https://github.com/search?q=path%3A%2F%5Erust-toolchain%5C.toml%24%2F&type=code)
- `rust-toolchain` [2.6k](https://github.com/search?q=path%3A%2F%5Erust-toolchain%24%2F&type=code)

#### What name for my WASM repository that needs disambiguation?

- ⭐ `*-wasm` [214](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-wasm%24+count:all&patternType=keyword&sm=0)
- `wasm-*` [183](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/wasm-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*.wasm` [19](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B%5C.wasm%24+count:all&patternType=keyword&sm=0)

#### What name for my Go repository that needs disambiguation?

- ⭐ `go-*` [3.5k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/go-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*-go` [1.3k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-go%24+count:all&patternType=keyword&sm=0)

#### What name for my C++ repository that needs disambiguation?

- ⭐ `*-cpp` [508](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-cpp%24+count:all&patternType=keyword&sm=0)
- `cpp-*` [293](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/cpp-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*.cpp` [77](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B%5C.cpp%24+count:all&patternType=keyword&sm=0)

#### What name for my Rust repository that needs disambigation?

- ⭐ `*-rs` [1.6k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-rs%24+count:all&patternType=keyword&sm=0)
- `rust-*` [1.2k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/rust-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*-rust` [524](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-rust%24+count:all&patternType=keyword&sm=0)
- `rs-*` [69](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/rs-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)


#### What name for my JavaScript repository that needs disambiguation?

- ⭐ `*.js` [3.4k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B%5C.js%24+count:all&patternType=keyword&sm=0)
- `node-*` [3.3k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/node-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*-js` [2.0k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-js%24+count:all&patternType=keyword&sm=0)
- `js-*` [793](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/js-%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `*-node` [766](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/%5B%5E%5C/%5D%2B-node%24+count:all&patternType=keyword&sm=0)

#### What name for my test code folder in a JavaScript project?

- ⭐ `test/` [3.8M](https://github.com/search?q=path%3A%2F%5Etest%5C%2F%2F%20(language%3AJavaScript%20OR%20language%3ATypeScript)&type=code)
- `tests/` [1.6M](https://github.com/search?q=path%3A%2F%5Etests%5C%2F%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)
- `__tests__/` [164k](https://github.com/search?q=path%3A%2F%5E__tests__%5C%2F%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)
- `__test__/` [22.7k](https://github.com/search?q=path%3A%2F%5E__test__%5C%2F%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)

#### What suffix for my test code files in a JavaScript project?

- ⭐ `*.test.[jt]{sx,x}` [4.6M](https://github.com/search?q=path%3A%2F%5Cw%2B%5C.test%5C.%5Cw%2B%24%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)
- `*_test.[jt]{sx,x}` [293k](https://github.com/search?q=path%3A%2F%5Cw%2B_test%5C.%5Cw%2B%24%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)

#### What name for my test code folder in a C++ project?

- ⭐ `test/` [1.1M](https://github.com/search?q=path%3A%2F%5Etest%5C%2F%2F+language%3AC%2B%2B&type=code)
- `tests/` [958k](https://github.com/search?q=path%3A%2F%5Etests%5C%2F%2F+language%3AC%2B%2B&type=code)

#### `.yaml` or `.yml` for GitHub actions workflows?

- ⭐ `.yml` [4.5M](https://github.com/search?q=path%3A%2F%5E%5C.github%5C%2Fworkflows%5C%2F.%2B%5C.yml%24%2F&type=code)
- `.yaml` [807k](https://github.com/search?q=path%3A%2F%5E%5C.github%5C%2Fworkflows%5C%2F.%2B%5C.yaml%24%2F&type=code)

#### `.yaml` or `.yml` in general?

- ⭐ `.yml` [29M](https://github.com/search?q=path%3A%2F%5C.yml%24%2F&type=code)
- `.yaml` [26M](https://github.com/search?q=path%3A%2F%5C.yaml%24%2F&type=code)

#### `.Equal()` or `.Equals()` in Go?

- ⭐ `.Equal()` [53.5k](https://github.com/search?q=%2F%5C%29+Equal%5C%28%2F+language%3AGo&type=code)
- `.Equals()` [40.6k](https://github.com/search?q=%2F%5C%29+Equals%5C%28%2F+language%3AGo&type=code)

#### `.equal()` or `.equals()` in JavaScript?

- `.equals()` [43.5k](https://github.com/search?q=%2F%5E%5Cs%2Bequals%5C%28.*%5C%7B%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)
- `.equal()` [13.8k](https://github.com/search?q=%2F%5E%5Cs%2Bequal%5C%28.*%5C%7B%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)

