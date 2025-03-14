---
title: DIY server routing with URLPattern
---

I think `URLPattern` is underrated in server-side JavaScript. Everyone ends up reinventing their own subtlety different route definition syntax.

[Hono](https://hono.dev/docs/api/routing):

```js
app.get('/user/:name', ...)
app.get('/post/:date{[0-9]+}/:title{[a-z]+}', ...)
app.get('/posts/:filename{.+\\.png}', ...)
```

[Express](https://expressjs.com/en/guide/routing.html):

```js
router.get('/user/:id(\\d+)', ...)
router.get('/plantae/:genus.:species', ...)
```

They're all fine and great and all that, I just like the builtin-ness of `URLPattern`.

Demo of `URLPattern` in action:

```js
const pattern = new URLPattern({ pathname: "/user/:name" })
const match = pattern.exec("https://example.org/user/Alan%20Turing")
//=> {
    ...
    "pathname": {
        "groups": { "name": "Alan%20Turing" },
        "input": "/user/Alan%20Turing"
    },
    ...
}
```

[📚 URL Pattern API | MDN](https://developer.mozilla.org/en-US/docs/Web/API/URL_Pattern_API)

Here's how I've started using it to DIY my own rudimentary router without any dependencies:

```js
// ⭐ Define your routes here. Order matters.
const routes = [
  [new URLPattern({ pathname: "/" }), {
    HEAD() { return this.GET(); },
    GET() { return new Response("Visit /api/<thing> to see it happen"); },
  }],
  [new URLPattern({ pathname: "/api/:thing" }), {
    HEAD(request, match) { return this.GET(request, match); },
    GET(request, match) {
      console.dir(request.url); //=> "https://localhost:8000/api/small-thing"
      console.dir(match.pathname.groups.thing); //=> "small-thing"
      // ... do the stuff with the thing
      const { thing } = match.pathname.groups;
      return new Response(`The thing is: ${thing}`);
    },
  }],
  // ... more routes here
];

export default {
  // 👨‍💻 Do the routing here.
  async fetch(request) {
    const route = routes
      .values()
      .map(([pattern, handlers]) => [pattern.exec(request.url), handlers])
      .find(([match, handlers]) => !!match);
    if (!route) return new Response(null, { status: 404 });
    const [match, handlers] = route;

    const handler = handlers[request.method];
    if (!handler) return new Response(null, { status: 405 });

    // Monkey around with pre/post middlewares here.
    return await handler.call(handlers, request, match);
  },
};
```

This works great for making easy copy-paste no-dependency demos in the [Deno Deploy playground](https://docs.deno.com/deploy/manual/playgrounds/). I'm sure it also works great with [the Cloudflare Workers playground](https://developers.cloudflare.com/workers/playground/) too.

Sometimes you just want the most basic of basics. But often the basics aren't enough; don't reinvent the wheel if you don't have a reason to.
