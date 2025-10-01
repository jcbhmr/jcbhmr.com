---
title: "Some popular conventions that I've observed [Part 2]"
---

This is a continuation of https://jcbhmr.com/2025/02/03/popular-conventions/

#### What name for a "smaller" or "slimmed-down" version of a popular library?

- ⭐ `mini*` [1.8k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/mini%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `micro*` [1.5k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/micro%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)
- `tiny*` [1.2k](https://sourcegraph.com/search?q=context:global+repo:github.com/%5B%5E%5C/%5D%2B/tiny%5B%5E%5C/%5D%2B%24+count:all&patternType=keyword&sm=0)

#### What name for the Go method that turns a Go object into a `js.Value`?

- ⭐ `.JSValue()` [538](https://github.com/search?q=language%3AGo+%2F%5C%29+JSValue%5C%28%5C%29+%28js%5C.%29%3FValue%2F&type=code)
- `.ToJS()` [47](https://github.com/search?q=language%3AGo+%2F%5C%29+ToJS%5C%28%5C%29+%28js%5C.%29%3FValue%2F&type=code)

#### Is it camelCase, kebab-case, snake_case, or alloneword for GitHub repository names?

*This is all hearsay. I haven't done the data digging to back this up with stats.*

Prefer kebab-case since `-` is treated as a word separator by search engines like Google. The repository name appears in the GitHub.com URL `github.com/octocat/my-awesome-project` and should thus be URL-friendly if you have no other constraints.

#### Is it camelCase, kebab-case, snake_case, alloneword, or "with spaces" for file names?

*There isn't really any data for this that isn't skewed towards repository-tracked data files on GitHub or published PDF files on WordPress sites from a Google search. This is all anecdotal from me.*

If it's a data file name, prefer kebab-case or snake_case depending on the conventions of the consuming software. Avoid case-sensitive naming. Prefer all lowercase. Windows is case-insensitive. Prefer avoiding spaces in programming contexts. \*nix-related scripting in shells is made more difficult by spaces in file names. Prefer regular "with spaces" title-style naming for `.docx`, `.pdf`, `.jpg`, etc. files that are meant to be shared or viewed by a human.

#### What's the preferred npm registry subdomain?

`npm.example.org`. Example: https://npm.jsr.io/.

_npmjs.com gets away with https://registry.npmjs.com because it's unambiguous._

#### What's the preferred OCI registry subdomain?

_I don't really know at this time. This is just to write these thoughts down for now._

- docker.example.org
- oci.example.org
- registry.example.org
- cr.example.org
- r.example.org

#### What's the preferred Go module subdomain?

If your domain is related to Go, then none; use Go module `<meta>` tags in your main website's HTML to point to your module source. Otherwise, `go.example.org`. Example: https://go.bytecodealliance.org/, https://go.uber.org/.
