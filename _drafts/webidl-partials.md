---
layout: post
title: I dislike Web IDL partial interfaces and mixins
---

Why? Because they are hard to compartmentalize.

**Scenario:** You're implementing the Navigator API for Node.js.

It's defined on [a single page](https://html.spec.whatwg.org/multipage/system-state.html#the-navigator-object), right? Sort of. The root `interface Navigator` is defined as such:

```webidl
[Exposed=Window]
interface Navigator {
  // objects implementing this interface also implement the interfaces given below
};
```

Quite literally an empty definition.

The trick is that `Navigator` is specified heavily through the use of `mixin`.

```webidl
Navigator includes NavigatorID;
Navigator includes NavigatorLanguage;
Navigator includes NavigatorOnLine;
Navigator includes NavigatorContentUtils;
Navigator includes NavigatorCookies;
Navigator includes NavigatorPlugins;
Navigator includes NavigatorConcurrentHardware;
```

```webidl
interface mixin NavigatorID { ... };
interface mixin NavigatorLanguage { ... };
```

To accurately represent this in JavaScript, it's almost like you need to continuously extend the root `class ActualRootNavigator {}` over and over to apply each mixin on top of the root class. Except, wait! You can't do that; that would break the precious inheritance hierarchy.

```js
// interface Navigator
class Navigator {}

// interface mixin NavigatorID
function NavigatorID(C) {
  return class extends C { ... };
}

// interface mixin NavigatorLanguage
function NavigatorLanguage(C) {
  return class extends C { ... };
}

export const ActualNavigatorThatWeExpose = NavigatorLanguage(NavigatorID(Navigator));

// This SHOULD be true but it's FALSE.
console.assert(Object.getPrototypeOf(ActualNavigatorThatWeExpose.prototype) === Object.prototype)
```

Clearly somehow we need to create just a single class with none of this `extends` nonsense. You may have already thought "well why not just dynamically apply the methods?" and that's a good point. That works great! For a single package. Within the realm of `my-awesome-navigator-polyfill` I can implement all seven of those mixins directly in the `class Navigator {}` body if I want.

The problem is that the `Navigator` interface is extended by other specifications outside the scope of `my-awesome-navigator-polyfill`. I want those packages such as [Clipboard API and events](https://w3c.github.io/clipboard-apis/) via `awesome-clipboard-polyfill` to be able to somehow extend or inject functionality such that `awesome-clipboard-polyfill` can just _depend on the root `my-awesome-navigator-polyfill` without reimplementing the entire `Navigator` object_. This avoids reimplementing the wheel for complex shared web specifications.

Granted this is a very niche and specific problem to have, however it is something that I don't quite know how to solve (conceptually; in practice you just monkeypatch until it works _well enough_ and call it good).

One idea that I've been toying around with lately to avoid the `extends` problem whilst also pseudo-extending the definition is to _delegate to a private property_ but still satisfy the interface.

```js
import { Navigator as SomeNavigatorPolyfillNavigator } from "some-navigator-polyfill"

export class Navigator {
  #inner;
  #clipboard;
  constructor(...args) {
    this.#inner = new SomeNavigatorPolyfill(...args);
    this.#clipboard = ...;
  }

  get clipboard() {
    return this.#clipboard;
  }

  get language() {
    return this.#inner.language;
  }
  get languages() {
    return this.#inner.languages;
  }
  get onLine() {
    return this.#inner.onLine;
  }
}
```

This does have a problem though: how do you stack the *Clipboard API and events* standard on top of the *Web Bluetooth API* on top of the *Web Serial API* when all of them fiddle with the `Navigator` object with mixins? Do you use three separate `ClipboardNavigator`, `BluetoothNavigator`, `SerialNavigator` classes? What about the root `Navigator` class?

You almost just want each splintering of the `Navigator` interface like the mixin defined in *Clipboard API* and events to only define its own to-be-mixed-in methods. Which is weird. Then you end up with something like this:

```js
import { Navigator as SomeNavigatorPolyfillNavigator } from "some-navigator-polyfill"

export class Navigator {
  #inner;
  #clipboard;
  constructor(inner) {
    this.#inner = inner;
    this.#clipboard = ...;
  }

  get clipboard() {
    return this.#clipboard;
  }
}
```

```js
import { Navigator as AwesomeClipboardPolyfillNavigator } from "awesome-clipboard-polyfill";
import { Navigator as SomeNavigatorPolyfillNavigator } from "some-navigator-polyfill";

const navigator = new SomeNavigatorPolyfillNavigator()
const navigator2 = new AwesomeClipboardPolyfillNavigator(navigator)
```
