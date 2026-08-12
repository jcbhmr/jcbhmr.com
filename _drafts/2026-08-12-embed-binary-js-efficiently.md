---
title: Embed binary data in JavaScript source efficiently
---

**Situation:** You have a 5 MiB WebAssembly binary that you want to embed directly into a `.js` file so that everything is bundled in that one file. The alternative of `fetch()`-ing a `./code.wasm` file won't cut it. So what do you do?

```js
const data = ...; // Something here. But what?
const { module, instance } = await WebAssembly.instantiate(data, imports);
// Do something with the WebAssembly.Instance...
```
