---
title: How to add a build step to your GitHub action
---

**This is for GitHub Actions publishers creating GitHub Actions, not consumers who use GitHub Actions in their GitHub Actions Workflows.**

Most GitHub Actions repositories look something like this:

```
.
├── dist/
│   ├── main.js
│   └── post.js
├── src/
│   ├── main.ts
│   ├── post.ts
│   └── utils.ts
├── package.json
├── package-lock.json
└── action.yml
```

🤮 **`dist/` is tracked by Git!** \
That means you need to remember to run `npm run build` or similar before pushing and/or have a GitHub Actions Workflow that builds & pushes code on each push to the default branch. Ugh.

But why do we need to do this? It's because GitHub Actions **do not run `npm install`** when running your `action.yml`. It just runs `node <file>` without any accompanying `node_modules/` dependencies. This means that we need to bundle everything that our GitHub Action needs into a file/folder that can be run directly with `node <file>` without a preceding `npm install`.

Ok, that's great, but why do we need to have such a bundled artifact **in source control**? Why can't it be like `npm run build && npm publish` so that consumers of the GitHub Action get the compiled output bundle with everything, while contributors can work with the source code? Because GitHub Actions Workflows just download the repositories' source tarball/zip archive for the tag you give it. When you do `uses: actions/setup-node@v6`, the GitHub Actions Workflow controller does `curl https://github.com/actions/setup-node/archive/refs/tags/v6.zip` to get the action's code. There's no special registry that you can publish compiled artifacts to.

**Or is there?** Normally, Git tags are just pointers to normal commits that are somewhere in the Git history chain along the default branch. ...but they don't have to be. What if we could `git tag` a snapshot that had our `dist/` folder while we kept `dist/ `out of the main source control tracking workflow? This is the trick to having an ergonomic build step for your GitHub Action.

Here's a quick rundown of how this trick can be done:

#### 1\. Add a build step to your developer workflow

Usually this is something like `npm run build` which calls a bundler of some kind. You probably already have this.

```json
{
  "scripts": {
    "build": "tsdown ..."
  }
}
```

#### 2\. Add a `.github/workflows/create-release.yml` workflow that will create new releases

You can't create GitHub releases normally anymore; that would create tags that point to commits on the default branch. Instead, you should create a "Create release" GitHub Actions Workflow that you can trigger using `workflow_dispatch: { ... }` with all the required parameters needed to create a release and it will `npm run build` (or similar) your GitHub Action before `git tag`-ing it and `gh release publish`-ing it for you. All the magic✨ will happen here.

<div><code>.github/workflows/create-release.yml</code></div>

```yml
# Based on https://jcbhmr.com/2025/10/22/github-actions-build-step/
name: Create release
on:
  workflow_dispatch:
    inputs:
      draft:
        type: boolean

jobs:
  create-release:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-node@v6

      - run: npm ci
      - run: npm run test
      - run: npm run build
      - run: "version=$(npm pkg get version)" >> "$GITHUB_ENV"

      - uses: fregante/setup-git-user@v2
      - name: Create tag
        run: |
          git add --force --all
          git commit --message "v$version"
          git tag "v$version"
          git push origin "v$version"
      - name: Create release
        run: |
          gh release create "v$version" \
            --title "v$version" \
            ${{ inputs.draft && '--draft' || '' }} \
            --verify-tag
```

Instead of creating releases through the `https://github.com/octocat/awesome-project/releases` page, you create them through `https://github.com/octocat/awesome-project/actions/workflows/create-release.yml` by clicking "Run workflow". This will run the build step, tag it, and then create a release (optionally a draft) for you.

#### 3\. Perform your build step before running your GitHub Action when testing

Normally, you'd test your GitHub Action like this:

<div><i>Excerpt from <code>.github/workflows/test.yml</code></i></div>

```yml
steps:
  - uses: actions/checkout@v5
  - id: my-action
    uses ./
    with:
      my-input: testing123
      testing-value: 456
  - shell: node {0}
    env:
      my_output: ${{ steps.my-action.outputs.my-output }}
    run: |
      import assert from "node:assert/strict"
      assert(process.env.my_output === "789");
```

Since you no longer track `dist/` compiled artifacts in source control you can't expect `actions/checkout` to checkout `dist/main.js` for you; you have to build it yourself before running `uses: ./`.

```diff
 - uses: actions/checkout@v5
+- uses: actions/setup-node@v6
+- run: npm ci
+- run: npm run build
 - id: my-action
   uses ./
   with:
     my-input: testing123
     testing-value: 456
```
