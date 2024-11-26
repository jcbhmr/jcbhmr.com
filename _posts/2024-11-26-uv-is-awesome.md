---
layout: post
title: uv is awesome
---

[astral-sh/uv](https://github.com/astral-sh/uv) is the best Python package manager that I've used. Granted, I haven't used a lot; but still. It's pretty good.

- It **manages your Python version** so you don't have to wrangle with that _at all_.
- It **has `--dev` dependencies** unlike plain pip
- You can still use `uv pip <pip_command_args>` just as you would normal `pip`
- It's **⚡fast⚡**
- It follows PEP standards instead of doing its own thing (looking at you Poetry)
- A single binary
- Doesn't require Python to install uv

It does have some downsides:

- Doesn't bundle a formatter or linter (still have to `uv add ruff`)
- Doesn't have a task runner (no `uv task <task_name>`)
- Nobody knows about it
- People confuse it with `rye`
- It doesn't have its own build backend (yet)
- It doesn't bundle a type hint checker

The easiest way to install uv is through one of their installer scripts:

<dl>
<dt>macOS & Linux
<dd>

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

<dt>Windows
<dd>

```pwsh
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

</dl>

📚 You can find more installation instructions on [the uv website](https://docs.astral.sh/uv/).

Now to create a Python package and publish it to PyPI is as simple as this:

```sh
uv init --lib
uv add --dev ruff pyright poethepoet pyinstaller
$EDITOR src/*.py
uv publish
```

Cool, right? It even manages the Python installation for you **automatically**.

You can even write standalone Python scripts like this:

```py
#!/usr/bin/env uv run
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json")
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])
```

[📚 Read more about running scripts with uv](https://docs.astral.sh/uv/guides/scripts/)

If you're looking to install a `pip`-provided CLI tool globally like `pip install cmakelang` provides `cmake-format` you can use `uv tool install cmakelang`. There's also `uvx` if you want the `npx` equivalent.

I think uv is cool. It's fast, easy to install, and it gets out of your way. You just write code and use `uv run <my_package>` or `uv run ./entry_point.py` to run your project. The virtual environment is not hidden like Poetry; it's right there in `.venv` if you want to go poking around.
