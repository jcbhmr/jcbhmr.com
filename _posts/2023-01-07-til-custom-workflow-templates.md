---
title: TIL you can create your own GitHub Actions workflow templates
redirect_from: /blog/til-custom-workflow-templates
---

So you know how there's this wizard that you can use to create new GitHub Actions right? It turns out that there's a way to add your own GitHub Actions templates there!

![](/uploads/2023-01-07-001.png)

All you need to do is add a `workflow-templates/*.yml` and `workflow-templates/*.properties.json` pair in your `user/.github` repository. This works for both organizations (like the official GitHub docs say) **and regular users**.

<div><code>workflow-templates/say-hi.yml</code></div>

```yml
name: Say hi!
on:
  push:
    branches: [$default-branch]
jobs:
  say-hi:
    runs-on: ubuntu-latest
    steps:
      - name: Echo "Hi!" to the user
        run: echo "Hi!"
```

<div><code>workflow-templates/say-hi.properties.json</code></div>

```json
{
    "name": "Say hi!",
    "description": "Says hi to the user",
    "iconName": "emoji-wave",
    "categories": ["Automation"],
    "filePatterns": ["README.md"]
}
```

<div><code>workflow-templates/emoji-wave.svg</code></div>

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <text y="75px" font-size="75px">👋</text>
</svg>
```

![](/uploads/2023-01-07-002.png)

📚 Further reading: [Creating starter workflows for your organization](https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization) (also works for users too)

## How you might want to use this

A great way to take advantage of this template feature is to make workflow templates for things you do often or in every repository. Think like...

1. Repetitive Node.js `test.yml` workflow
2. Workflow to deploy to GitHub Pages from `npm run build`
3. Mirroring `wiki/` in your Git reposutory to a GitHub wiki page
4. Enabling `/help` in any issue

Anytime you think "I wish I could just copy-paste this workflow from my previous project" then it might be time to make that a template.
