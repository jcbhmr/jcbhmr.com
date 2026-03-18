---
layout: page
title: Conventions
---

I like conventions. I like having pointers for all the minor decisions, like "what do I name my test folder?" so that I don't have to think about it each time. I also want to match what others would expect the test folder to be named. Conventions are a great way to satisfy these two criteria. But where are those conventions written down? They often _aren't_. This is my personal log of all the times I've asked "what do I name _something_?" or "what's the preferred way to _do something_?" and found an answer that I want to remember.

#### Is it `rust-toolchain` or `rust-toolchain.toml`?

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

#### What name for my test code folder in a JavaScript (not TypeScript) project?

- ⭐ `test/` [3M](https://github.com/search?q=path%3A%2F%5Etest%5C%2F%2F+language%3AJavaScript&type=code)
- `tests/` [1.6M](https://github.com/search?q=path%3A%2F%5Etests%5C%2F%2F+language%3AJavaScript&type=code)
- `__tests__/` [161k](https://github.com/search?q=path%3A%2F%5E__tests__%5C%2F%2F+language%3AJavaScript&type=code)
- `__test__/` [15.9k](https://github.com/search?q=path%3A%2F%5E__test__%5C%2F%2F+language%3AJavaScript&type=code)

Note that this is a different convention (`test/`) from TypeScript projects (`tests/`).

#### What name for my test code folder in a TypeScript (not JavaScript) project?

- ⭐ `tests/` [1.5M](https://github.com/search?q=path%3A%2F%5Etests%5C%2F%2F+language%3ATypeScript&type=code)
- `test/` [1.3M](https://github.com/search?q=path%3A%2F%5Etest%5C%2F%2F+language%3ATypeScript&type=code)
- `__tests__/` [186k](https://github.com/search?q=path%3A%2F%5E__tests__%5C%2F%2F+language%3ATypeScript&type=code)
- `__test__/` [11.1k](https://github.com/search?q=path%3A%2F%5E__test__%5C%2F%2F+language%3ATypeScript&type=code)

Note that this is a different convention (`tests/`) from non-TypeScript JavaScript projects (`test/`).

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

- ⭐ `.equals()` [43.5k](https://github.com/search?q=%2F%5E%5Cs%2Bequals%5C%28.*%5C%7B%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)
- `.equal()` [13.8k](https://github.com/search?q=%2F%5E%5Cs%2Bequal%5C%28.*%5C%7B%2F+%28language%3AJavaScript+OR+language%3ATypeScript%29&type=code)

#### `.equal()` or `.equals()` in general?

- ⭐ `.equals()` [1.9M](https://github.com/search?q=%2Fequals%5C%28other%2F&type=code)
- `.equal()` [134k](https://github.com/search?q=%2Fequal%5C%28other%2F&type=code)

#### What file extension for C++?

- ⭐ `*.cpp` [46.4M](https://github.com/search?q=path%3A*.cpp&type=code)
- `*.cc` [6.8M](https://github.com/search?q=path%3A*.cc&type=code)
- `*.cxx` [881k](https://github.com/search?q=path%3A*.cxx&type=code)

#### What file extension for C++ modules?

- ⭐ `*.ixx` [24.4k](https://github.com/search?q=path%3A*.ixx&type=code)
- `*.cppm` [15.2k](https://github.com/search?q=path%3A*.cppm&type=code)

#### What name for a "smaller" or "slimmed-down" version of a popular library?

- ⭐ `mini*` [1.8k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/mini%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `micro*` [1.5k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/micro%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `tiny*` [1.2k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/tiny%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)

#### What name for the Go method that turns a Go object into a `js.Value`?

- ⭐ `.JSValue()` [538](https://github.com/search?q=language%3AGo+%2F%5C%29+JSValue%5C%28%5C%29+%28js%5C.%29%3FValue%2F&type=code)
- `.ToJS()` [47](https://github.com/search?q=language%3AGo+%2F%5C%29+ToJS%5C%28%5C%29+%28js%5C.%29%3FValue%2F&type=code)

#### Is it camelCase, kebab-case, snake_case, or alloneword for GitHub repository names?

*This is all hearsay. I haven't done the data digging to back this up with stats.*

Prefer ⭐ kebab-case since `-` is treated as a word separator by search engines like Google. The repository name appears in the GitHub.com URL `github.com/octocat/my-awesome-project` and should thus be URL-friendly if you have no other constraints.

#### Is it camelCase, kebab-case, snake_case, alloneword, or "with spaces" for file names?

*There isn't really any data for this that isn't skewed towards repository-tracked data files on GitHub or published PDF files on WordPress sites from a Google search. This is all anecdotal from me.*

- ⭐ If it's a **data file** name, prefer kebab-case or snake_case depending on the conventions of the consuming software. If it's tracked in Git, make it match your project's file name style.
- Avoid case-sensitive naming. Prefer all lowercase. (Remember Windows is case-insensitive.)
- Prefer avoiding spaces in programming contexts. \*nix-related scripting in shells is made more difficult by spaces in file names.
- Prefer regular "with spaces" title-style naming for `.docx`, `.pdf`, `.jpg`, etc. files that are meant to be shared or viewed by a human. Like Google Drive: think of the file name as the document title.

#### What's the preferred npm registry subdomain?

*Couldn't find an easy way to get stats on this*

`npm.example.org`. Example: https://npm.jsr.io/.

_npmjs.com gets away with https://registry.npmjs.com because it's unambiguous._

#### What's the preferred OCI registry subdomain?

- ⭐ `docker.example.org` [718](https://sourcegraph.com/search?q=context%3Aglobal+%2F%28%5E%7C%5Cs%29docker%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29%2F+count%3Aall&patternType=keyword&sm=0&__cc=1)
- `cr.example.org` [438](https://sourcegraph.com/search?q=context%3Aglobal+%2F%28%5E%7C%5Cs%29cr%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29%2F+count%3Aall&patternType=keyword&sm=0&__cc=1) _note that `cr` is an ISO 639-1 language code and an ISO 3166 country code_
- `containers.example.org` [142](https://sourcegraph.com/search?q=context:global+/%28%5E%7C%5Cs%29containers%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29/+count:all&patternType=keyword&sm=0)
- `oci.example.org` [59](https://sourcegraph.com/search?q=context:global+/%28%5E%7C%5Cs%29oci%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29/+count:all&patternType=keyword&sm=0)

Sometimes container registries use a `*cr.io` TLD. Ex: `gcr.io` and `ghcr.io`. *I couldn't think of a good way to filter for just `*cr.io` domains that happen to serve an OCI registry. Let me know if you have one.*

#### What's the preferred Go module subdomain?

If your domain is related to Go, then none; use Go module `<meta>` tags in your main website's HTML to point to your module source. [https://rsc.io does this](https://pkg.go.dev/search?q=rsc.io&m=). Otherwise, `go.example.org`. Example: https://go.bytecodealliance.org/, https://go.uber.org/.

- ⭐ `go.example.org` [910](https://sourcegraph.com/search?q=context:global+file:go%5C.mod+/%5Emodule+go%5C.%28%5Cw%2B%29%5C.%28com%7Corg%7Cnet%7Cdev%7Cio%29%28%5C/%7C%24%29/+count:all&patternType=keyword&sm=0)
- `git.example.org` [28](https://sourcegraph.com/search?q=context:global+file:go%5C.mod+/%5Emodule+git%5C.%28%5Cw%2B%29%5C.%28com%7Corg%7Cnet%7Cdev%7Cio%29%28%5C/%7C%24%29/+count:all&patternType=keyword&sm=0)
- `(gitlab|forgejo|gitea|gogs|cgit).example.org` [14](https://sourcegraph.com/search?q=context:global+file:go%5C.mod+/%5Emodule+%28gitlab%7Cforgejo%7Cgitea%7Cgogs%7Ccgit%29%5C.%28%5Cw%2B%29%5C.%28com%7Corg%7Cnet%7Cdev%7Cio%29%28%5C/%7C%24%29/+count:all&patternType=keyword&sm=0)

#### What's the preferred Git code host subdomain?

- ⭐ `git.example.org` [2.8k](https://sourcegraph.com/search?q=context:global+/%28%5E%7C%5Cs%29git%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29/+count:all&patternType=keyword&sm=0)
- `code.example.org` [520](https://sourcegraph.com/search?q=context:global+/%28%5E%7C%5Cs%29code%5C.%5Ba-z0-9%5C-%5D%2B%5C.%28com%7Corg%7Cnet%7Cio%7Cdev%7Cus%7Cuk%7Ccc%7Ctv%7Cai%7Cme%7Cblog%7Csite%29%28%5Cs%7C%24%29/+count:all&patternType=keyword&sm=0)
- `(github|gitlab|forgejo|gogs|gitea|cgit).example.org` [295](https://sourcegraph.com/search?q=context:global+/%28%5E%7C%5Cs%29https:%5C/%5C/%28github%7Cgitlab%7Cforgejo%7Cgogs%7Cgitea%7Ccgit%29%5C.%28%5Ba-z0-9%5C-%5D%2B%5C.%29%2B%28%5Ba-z0-9%5C-%5D%2B%29%28%5Cs%7C%24%29/+count:all&patternType=keyword&sm=0)
