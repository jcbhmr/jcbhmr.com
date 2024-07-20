---
layout: post
title: Write your Rust project scripts in task.rs
---

🏃‍♂️ Write your scripts in a single `task.rs` file
💡 Inspired by [matklad/cargo-xtask](https://github.com/matklad/cargo-xtask)

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
uuid = { version = "1.8.0", features = ["v4"] }
---
fn generate() -> Result<(), Box<dyn std::error::Error>> {
    use uuid::Uuid;
    use std::fs;
    let id = Uuid::new_v4();
    fs::write("id.txt", id)?;
    Ok(())
}
fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "generate" => generate(),
        _ => Err("no such task".into()),
    }
}
```

```sh
cargo +nightly -Zscript task.rs generate
# In the future "+nightly -Zscript" won't be required.
```

🤩 Plain Rust code; use the same language for code and scripting.
☝ Just one file! 🆕 Uses [the new Cargo script feature](https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#script).
🚀 Modify it to suit your needs. This is a template/idea not a library.
😎 Runs wherever Rust does; no more Linux-only `Makefile`.

---

There's nothing to install! Just create your own `task.rs` file in the root of your project and write your tasks. You will need the nightly version of Cargo so that you can use `cargo +nightly -Zscript` to run `task.rs` with the script feature enabled. You can install the nightly version of the Rust toolchain (which includes Cargo nightly) using Rustup like this:

```sh
rustup toolchain install nightly
```

[📚 Read more about the `-Zscript` nightly Cargo feature](https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#script)

The basic template for a `task.rs` file is this:

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
# Your dependencies here!
---
fn generate() -> Result<(), Box<dyn std::error::Error>> {
  // Your code here!
  Ok(())
}
fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "generate" => generate(),
        // Add more tasks as match arms here.
        _ => Err("no such task".into()),
    }
}
```

You can see more in depth examples below. 👇

Then you can run `cargo +nightly -Zscript task.rs <task_name>` to run your user-defined task!

```sh
cargo +nightly -Zscript task.rs generate
# In the future "+nightly -Zscript" won't be required.
```

💡 If you're smart you can also `chmod +x task.rs` so that you can do `./task.rs <task_name>` instead of `cargo +nightly -Zscript task.rs`. 😉

## How is `cargo task.rs` different from `cargo xtask`?

- `cargo xtask` is a complete subproject. `cargo task.rs` is a single file.
- `cargo xtask` uses a `.cargo/config.toml` file to define the `cargo xtask` alias. `cargo task.rs` does not.

Other than the difference of being single-file vs multi-file there's not much difference. The idea is the same: use Rust to write your task scripts.

## Custom build script for releases

Maybe you want to run some kind of post-processing operations on the resulting binary output that `cargo build` gives you by default. Maybe you want to add some extra assets like DLLs or images to the resulting target folder. Or maybe you just want to customize how the binary is archived and compressed. 🤷‍♀️

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
xshell = "0.2.6"
---

fn build_release() -> Result<(), Box<dyn std::error::Error>> {
    use xshell::{Shell, cmd};
    use std::fs::copy;
    let sh = Shell::new()?;
    cmd!(sh, "cargo build --release").run()?;
    if cfg!(unix) {
        let exe = "target/release/hello-world";
        let sh = Shell::new()?;
        cmd!(sh, "strip {exe}").run()?;
    }
    copy("assets/icon.png", "target/release/icon.png")?;
    Ok(())
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "build-release" => build_release(),
        _ => Err("no such task".into()),
    }
}
```

```sh
cargo +nightly -Zscript task.rs build-release
```

## Publish all crates in a workspace

You can use a local `publish-all` script to avoid foisting a global dependency on your contributors like [cargo-publish-all](https://crates.io/crates/cargo-publish-all). Just use `cargo task.rs` to do that!

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
xshell = "0.2.6"
---

fn publish_all() -> Result<(), Box<dyn std::error::Error>> {
    use std::env;
    use xshell::{cmd, Shell};
    let sh = Shell::new()?;
    cmd!(sh, "cargo publish --package thing-a").run()?;
    cmd!(sh, "cargo publish --package thing-b").run()?;
    cmd!(sh, "cargo publish --package thing-c").run()?;
    cmd!(sh, "cargo publish --package thing-d").run()?;
    Ok(())
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "publish-all" => publish_all(),
        _ => Err("no such task".into()),
    }
}
```

<details><summary>OR a dynamic approach</summary>

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
xshell = "0.2.6"
---

fn publish_all() -> Result<(), Box<dyn std::error::Error>> {
    use std::env;
    use xshell::{cmd, Shell};
    let sh = Shell::new()?;
    let stdout = cmd!(sh, "cargo tree --depth 0").read()?;
    let packages = stdout
        .split_terminator("\n\n")
        .filter_map(|line| line.split_whitespace().next());
    let args_rest: Vec<String> = env::args().collect();
    let args_rest = args_rest.split_off(2);
    for package in packages {
        let sh = Shell::new()?;
        let args_rest_slice = args_rest.as_slice();
        cmd!(sh, "cargo publish --package {package} {args_rest_slice...}").run()?;
    }
    Ok(())
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "publish-all" => publish_all(),
        _ => Err("no such task".into()),
    }
}
```

</details>

```sh
cargo +nightly -Zscript task.rs publish-all
```

## Use feature flags to conditionally compile heavy dependencies

We can use feature flags to avoid compiling heavy dependencies for tasks that don't actually use said heavy dependencies. The hack is that we rerun the script with the task-specific `--features <task_feature>` flag set and then enable the heavy dependencies when said feature flag is provided.

<div><code>task.rs</code></div>

```rs
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[features]
generate = ["quick-xml", "wasmtime"]
[dependencies]
quick-xml = { version = "0.3.1", optional = true }
wasmtime = { version = "19.0.1", optional = true }
---

#[cfg(feature = "generate")]
fn generate() -> Result<(), Box<dyn std::error::Error>> {
    use quick_xml::*;
    use wasmtime::*;
    // Do something with quick_xml and wasmtime...
    Ok(())
}

fn build_release() -> Result<(), Box<dyn std::error::Error>> {
    // Do the quick & easy build copy stuff.
    Ok(())
}

fn test_e2e() -> Result<(), Box<dyn std::error::Error>> {
    // Another easy one that requires only quick deps.
    Ok(())
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    match std::env::args().nth(1).ok_or("no task")?.as_str() {
        "generate" => {
            #[cfg(feature = "generate")]
            return generate();
            #[cfg(not(feature = "generate"))]
            return std::process::Command::new("cargo")
              .args(["+nightly", "-Zscript", "run", "--manifest-path", file!()])
              .args(["--features", "generate", "--", "generate"])
              .status()?
              .success()
              .then_some(())
              .ok_or("cmd failed".into());
        },
        "build-release" => build_release(),
        "test-e2e" => test_e2e(),
        _ => Err("no such task".into()),
    }
}
```

```sh
cargo +nightly -Zscript task.rs generate
```

---

<sup>Do you have a cool example use of `task.rs` you'd like to share? Post it online and show me! ❤️🤩</sup>