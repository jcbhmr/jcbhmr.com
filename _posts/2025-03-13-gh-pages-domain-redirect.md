---
title: GitHub Pages redirect to new domain
---

Here's the code:

<div><code>404.html</code></div>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="refresh" content="0; url=DEST_ROOT_URL" />
    <meta name="robots" content="noindex" />
    <title>Moved to DEST_ROOT_URL</title>
    <script>
      location.replace("DEST_ROOT_URL" + location.toString().slice(origin.length));
    </script>
  </head>
  <body>
    <p>
      Moved to <a href="DEST_ROOT_URL">DEST_ROOT_URL</a>
    </p>
  </body>
</html>
```

Where `DEST_ROOT_URL` is the destination URL root **without the trailing slash** since `origin` doesn't include the trailing slash.

The trick is the JavaScript in the `404.html` page which is rendered when a URL is not found anywhere in your GitHub Pages site. The `<script>` tag is in `<head>` so it _should_ run before the page is even shown. It **replaces** (no back-button history entry) the current URL with the one you give it. In this case that's the current URL (`location.toString()`) minus the `https://example.com` prefix (`.slice(origin.length)`) with the destination URL prepended to it.

For example:

1. Navigate to `https://example.org/hello-world`
2. 404, serve `404.html`
3. `"https://example.org/hello-world".slice("https://example.org".length)`
4. `location.replace("https://example.com" + $3)`
