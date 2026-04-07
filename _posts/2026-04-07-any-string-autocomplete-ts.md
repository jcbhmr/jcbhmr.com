---
title: "'\"one\" | \"two\" | string' autocomplete TypeScript trick"
---

Have you ever wanted to suggest some strings but still allow any string? Like `"GET" | "POST" | string`? But then it just _doesn't work_ like you want it to?

<div><a href="https://www.typescriptlang.org/play/?&q=191#code/PTAEDkHsBcFNQBaQO6gOQCIDiBRAKhqAD6gYAKA8gMoHGgDO0ATgJYB2A5mqB7NPaCawAJgFcAxiNDRIoAFajGpRq06F67SaBbQ0A6AlgAoEKABGTSAENhsJdACeAB3iQAZtMPa2cJvVji0CyQbAD8oHgIVroCsAAeLoFSZrBRAG7BokwAdEZGji6gABJ4eGQAsnxIwqAAvKS4tCTk1E0MzOwcJmCgoAB6od29wyOjYyN9EQgsAvRIogA2NSnKHWp5bqJsgcFsoMKQJWUAFAC2VZDCAFzFpRUXwgCUN2mQLDUA3kYj4iH0kAtYNkFpAOGcHo9vsNTONegMob0YbDhpNIjMGPMluZ4Bhjk4rEwrOdfI9QMTqjcVJ0MLkAL55A5HMjHDAYSFIuGgMhCegCADCeAASgAZADUVDIAEE+ThELAhNJZMwHKArGwarxoKrRDJfqcnIC4NlQABNSCiDHmrGa0BsGDTTjZIA">[TypeScript playground link]</a></div>

```ts
// Note how '"GET" | "POST" | string' gets reduced to just "string" since it's the
// broadest type of the intersection? That's expected behaviour.

type HTTPMethod = "GET" | "POST" | string
//   ^?
//                        ^ This should be "string"

function doHTTP(method: HTTPMethod): void {
    console.log(method)
    //          ^?
    //             ^ This should be "(parameter) method: string".
}

doHTTP("")
//     ^ Press CTRL+SPACE here to try and get autocomplete. You should get nothing.
```

> From the compiler's point of view, this is just a very fancy way of writing `string`. By the time we're looking for suggestions at \[HTTPMethod\], all the literals are lost.

&mdash; [microsoft/TypeScript#29729](https://github.com/Microsoft/TypeScript/issues/29729)

The trick is to fool the type checker into thinking that the `| string` part of the union type is magically (somehow?) different than the type of the literals -- that way it doesn't get reduced to just `type T = string`. The magic is `string & {}`.

<div><a href="https://www.typescriptlang.org/play/?&q=191#code/PTAEEkBdQdwewE4GsDOBCAUByBPADgKagASAKqQAoCyBkAFnACagC8oARAOICip7oAHw4UA8gGU+g0AAoUkBAEsAdgHNQAMlABvAL4BKDCFDGAegH5DYY9Zu271k6FJ0FKUCgYBXADbMARkQA5LLyymqaunpSXLz8QuyiEuyBWABmnkoAxpAKcEqgjHBklNIAtrQMjABcJOTUFUx6NQBucArMWhg2mXkocN4EAHTecCplDYwGNkb2phbTVrMOTi5uHnA+-kTs0ngAhgh75ZAECFHHlTXF9fRM7IMYOliF19Ls7AYzyxQIBChuAGFSAAlAAyAGoxBQAIIA7igOinIiQOCgeQ4UB7JTMFS0TGeFE9Up4AYnQagACaG3cXl8oFx0DkilUoG8ChOh286CwRkGfL8BNAOGpmSxmK5qIAVp45Gj8EQsRimWEhdSYFjIOTnK5QDrmns2YwHi86m8lHBIABaLGWuiQSB4S0XO56IA">[TypeScript playground link]</a></div>

 ```ts
// It works!

type HTTPMethod = "GET" | "POST" | (string & {})
//   ^?
//                        ^ This should be '(string & {}) | "GET" | "POST"'

function doHTTP(method: HTTPMethod): void {
    console.log(method)
    //          ^?
    //             ^ This should be "(parameter) method: HTTPMethod".
}

doHTTP("")
//     ^ Press CTRL+SPACE here to try and get autocomplete. You should get string literals!

// ...but you can also just type any string you want. This is valid.
doHTTP("not-an-http-method")
```

The [type-fest](https://www.npmjs.com/package/type-fest) npm package has a type for this exact use-case: [`LiteralUnion`](https://github.com/sindresorhus/type-fest/blob/main/source/literal-union.d.ts).

```ts
import type {LiteralUnion} from 'type-fest';

// Before

type Pet = 'dog' | 'cat' | string;

const petWithoutAutocomplete: Pet = '';
// Start typing in your TypeScript-enabled IDE.
// You **will not** get auto-completion for `dog` and `cat` literals.

// After

type Pet2 = LiteralUnion<'dog' | 'cat', string>;

const petWithAutoComplete: Pet2 = '';
// You **will** get auto-completion for `dog` and `cat` literals.
```

There was a TypeScript GitHub issue opened about having this `"one" | "two" | string` literal stuff work out of the box, but it didn't go anywhere. [Microsoft/TypeScript#29729](https://github.com/Microsoft/TypeScript/issues/29729)
