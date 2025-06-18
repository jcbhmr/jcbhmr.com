---
title: It's easy to dev blog
redirect_from: /blog/its-easy-to-dev-blog
---

...the hard part is writing the words.

Here's a basic step-by-step to creating the _most basic_ dev blog that looks _reasonably_ OK. No, we aren't using JavaScript. No, we aren't using a template. Yes, signing up for a WordPress instance is probably easier. 😅

## 1. Create your .github.io repository

You get a ready-made GitHub Pages `<username>.github.io` domain that's perfect for your basic website needs. Just create a new repository like `jcbhmr.github.io` but replace `jcbhmr` with your GitHub username.

![](https://i.imgur.com/6sB2ARp.png)

## 2. Enable GitHub pages

In your repository settings you need to turn on GitHub Pages to make it pull [Jekyll](https://jekyllrb.com/) content (that's the magic✨ default GitHub Pages build tool) from your GitHub repository.

You can find the menu to turn on GitHub Pages in the Settings tab of your repository under the Pages sidebar menu.

![](https://i.imgur.com/p2kbz19.png)

Make sure you set the source branch!

After you hit Save you should be able to visit `<username>.github.io` as a website. (It may take a minute to deploy things so be patient.)

## 3. Setup a post index page theme

The default Primer GitHub Pages theme is nice... but it lacks a homepage index of your most recent blog posts. 😢 IMO the best path forward is to switch to another default theme: the builtin default [Jekyll Minima theme](https://github.com/jekyll/minima).

![](https://i.imgur.com/4hpy8TZ.png)

To do that, create a `_config.yml` file in your GitHub repository that you created in step 1 and add the following line:

<div><code>_config.yml</code></div>

```yml
remote_theme: jekyll/minima
```

[📚⚙️ More Minima settings!](https://github.com/jekyll/minima/blob/master/_config.yml)

Why again do you need to do this? Because we need to set the theme to _something_ and Minima provides a good default blog style & setup. If you prefer GitHub-Flavored Markdown to the default Kramdown you'll also want to add `markdown: GFM` to `_config.yml`.

## 4. Write your content!

Make a `_posts/` folder and then create a `_posts/2020-01-01-hello-world.md` document. _(Change the date to reflect your date)_ Add some content to it! Just make sure it has some YAML frontmatter specifying it's `layout: post` and `title: Your text here`.

<div><code>_posts/2020-01-01-hello-world.md</code></div>

```md
---
title: Hello world!
---

<mark>Hello world!</mark> This is some [blog](#) _post_ **content**.

## I'm another section header
```

Now you have your new fancy dev blog post on _your own website_! 🥳 And it only took a few files!

<sub>It should look something like this</sub>

![](https://i.imgur.com/NZ1xSA1.png)

## Or just use something else

You can always use WordPress, Google Docs Publishing, a GitHub Gist, DEV.to, or any other platform that you feel comfortable with. Once you get **something** on a (digital) paper somewhere so that you can show someone or post a link somewhere, that's **OUTSTANDING** already! 🎉
