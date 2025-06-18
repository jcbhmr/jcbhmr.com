---
title: JSDoc types are not TypeScript types
---

**TL;DR:** If you write typed JS code using JSDoc, your code's types will likely not be recognized by some portion of the JS ecosystem. You need `.d.ts` type files for that.

I would like to use JavaScript as-is without a build step. [Lots of people would love it if TypeScript types could be written as part of valid JavaScript code.](https://github.com/tc39/proposal-type-annotations) I tried to test the idea of "just use JSDoc instead of TypeScript" to see how well it works.

It doesn't work the same. JSDoc types aren't recognized by [npm](https://www.npmjs.com/), [jsdocs.io](https://www.jsdocs.io/), or [tsdocs.dev](https://tsdocs.dev/).

**npm** doesn't display the "TS" badge next to the package name **unless you have a `"types": "./dist/index.d.ts"` or similar `types` key in your `package.json`**.

![image](https://i.imgur.com/ZNhvlLj.png)

[📚 Read the official GitHub/npm announcement](https://github.blog/changelog/2020-12-16-npm-displays-packages-with-bundled-typescript-declarations/)

**tsdocs.dev** doesn't work if you don't have a file with a `.ts` or `.tsx` extension (like `.d.ts`).

**jsdocs.io** doesn't work if you aren't using `.d.ts` files.

**VSCode** uses the TypeScript Language Server which _does_ recognize JSDoc types as a source for TypeScript information. It just doesn't process `node_modules/**/*.js` JSDoc types quite the same way as `.d.ts` files.

```
app.ts(1,23): error TS7016: Could not find a declaration file for module 'lib'. '.../index.js' implicitly has an 'any' type.
Try `npm install @types/lib` if it exists or add a new declaration (.d.ts) file containing `declare module 'lib';
```

[🐛 JSDoc-typed node modules require special configuration in consumers to be useful · Issue #19145 · microsoft/TypeScript](https://github.com/microsoft/TypeScript/issues/19145)

👆 You get this annoying warning when using JSDoc-typed npm packages. Clearly `.d.ts` is the preferred way here.

Ugh. The solution to all of this "it's typed, but nobody can see it" issue is to use `tsc --allowJs --checkJs --declaration` to emit a `.d.ts` file for every `.js` file **based on the existing JSDoc types**. Yes, this is a hassle.

Or you can just give up on it and do what [Sindre Sorhus](https://github.com/sindresorhus) does: manually write `.js` (without types) and then write the types in `.d.ts`.

I just wanted to rant a bit to vent my frustration that "just use JSDoc" isn't _quite_ there yet. Someday I hope to eliminate the JS compile step (for basic libraries at least; forget about frameworks 😅). That day is not today.
