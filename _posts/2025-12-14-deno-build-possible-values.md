---
title: "Possible platform/arch names in Deno.build & node:process"
---

I did some diving into what the possible values for these string enums are:

- [`Deno.build.os`](https://docs.deno.com/api/deno/~/Deno.build.os)
- [`Deno.build.arch`](https://docs.deno.com/api/deno/~/Deno.build.arch)
- [`Deno.build.target`](https://docs.deno.com/api/deno/~/Deno.build.target)
- [`node:process` `platform`](https://docs.deno.com/api/node/process/~/Process#property_platform) & [`node:os` `platform()`](https://docs.deno.com/api/node/os/~/platform)
- [`node:process` `arch`](https://docs.deno.com/api/node/process/~/Process#property_arch) & [`node:os` `arch()`](https://docs.deno.com/api/node/os/~/arch)
- [`node:os` `machine()`](https://docs.deno.com/api/node/os/~/machine)

Note that the `node:`-related values that I'm discussing are **for Deno's Node.js API polyfills**, not the Node.js' implementation of them. That's another deep dive that I might explore later. **This is just Deno's platform/arch values** -- where and how they are exposed to JS code.

**TL;DR:**

- `Deno.build.os` is [any possible OS name across all valid Rust targets](https://docs.rs/platforms/latest/platforms/target/enum.OS.html)
- `Deno.build.arch` is [any possible arch name across all valid Rust targets](https://docs.rs/platforms/latest/platforms/target/enum.Arch.html)
- `Deno.build.target` is [any valid Rust target tuple](https://doc.rust-lang.org/stable/rustc/targets/built-in.html)
- `process.platform` is [any possible OS name across all valid Rust targets](https://docs.rs/platforms/latest/platforms/target/enum.OS.html) **but replace `windows` with `win32`**
- `process.arch` is `"x64" | "arm64" | "riscv64"` **only**; it throws an `Error` when run on any other arch
- `os.machine()` is [any possible arch name across all valid Rust targets](https://docs.rs/platforms/latest/platforms/target/enum.Arch.html) **but replace `aarch64` with `arm64`**

⚠️ **Remember!** These are the possible values _for Deno's Node.js `node:` polyfills_, not for Node.js' own implementation.

## `Deno.build`

There's this `Deno.build` object. It's configured with default values.

<div><a href="https://github.com/denoland/deno_core/blob/2898ad965ab1b5c616c9b6738b0b7ae046653320/core/00_infra.js#L58">denoland/deno: <code>core/00_infra.js</code></a></div>

```js
const build = {
  target: "unknown",
  arch: "unknown",
  os: "unknown",
  vendor: "unknown",
  env: undefined,
};
```

There's a `setBuildInfo()` function...

<div><a href="https://github.com/denoland/deno_core/blob/2898ad965ab1b5c616c9b6738b0b7ae046653320/core/00_infra.js#L66">denoland/deno_core: <code>core/00_infra.js</code></a></div>

```js
function setBuildInfo(target) {
  const { 0: arch, 1: vendor, 2: os, 3: env } = StringPrototypeSplit(
    target,
    "-",
    4,
  );
  build.target = target;
  build.arch = arch;
  build.vendor = vendor;
  build.os = os;
  build.env = env;
  ObjectFreeze(build);
}
```

...which is exposed as `Deno.core.setBuildInfo()`.

<div><a href="https://github.com/denoland/deno_core/blob/2898ad965ab1b5c616c9b6738b0b7ae046653320/core/00_infra.js#L432">denoland/deno_core: <code>core/00_infra.js</code></a></div>

```js
// Extra Deno.core.* exports
const core = ObjectAssign(globalThis.Deno.core, {
  build,
  setBuildInfo, // 👈
  registerErrorBuilder,
  buildCustomError,
  registerErrorClass,
  setUpAsyncStub,
  hasPromise,
  promiseIdSymbol,
});
```

Since this function is now exposed, it can be called from elsewhere.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/js/99_main.js#L408">denoland/deno: <code>runtime/js/99_main.js</code></a></div>

```js
function runtimeStart(
  denoVersion,
  v8Version,
  tsVersion,
  target,
) {
  core.setWasmStreamingCallback(fetch.handleWasmStreaming);
  core.setReportExceptionCallback(event.reportException);
  op_set_format_exception_callback(formatException);
  version.setVersions(
    denoVersion,
    v8Version,
    tsVersion,
  );
  core.setBuildInfo(target); // 👈
}
```

Where does this `runtimeStart()` function get called? In the same file.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/js/99_main.js#L765">denoland/deno: <code>runtime/js/99_main.js</code></a></div>

```js
runtimeStart(
  denoVersion,
  v8Version,
  tsVersion,
  target,
);
```

But where does that `target` variable come from? `op_snapshot_options()`.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/js/99_main.js#L604">denoland/deno: <code>runtime/js/99_main.js</code></a></div>

```js
const {
  tsVersion,
  v8Version,
  target,
} = op_snapshot_options();
```

But where does _that_ come from? It's imported from `ext:core/ops`...

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/js/99_main.js#L21">denoland/deno: <code>runtime/js/99_main.js</code></a></div>

```js
import {
  op_bootstrap_args,
  op_bootstrap_is_from_unconfigured_runtime,
  op_bootstrap_no_color,
  op_bootstrap_pid,
  op_bootstrap_stderr_no_color,
  op_bootstrap_stdout_no_color,
  op_get_ext_import_meta_proto,
  op_internal_log,
  op_main_module,
  op_ppid,
  op_set_format_exception_callback,
  op_snapshot_options, // 👈
  op_worker_close,
  op_worker_get_type,
  op_worker_post_message,
  op_worker_recv_message,
  op_worker_sync_fetch,
} from "ext:core/ops";
```

...which is defined in `deno_core`.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/ops/bootstrap.rs#L71">denoland/deno: <code>runtime/ops/bootstrap.rs</code></a></div>

```rs
// Note: Called at snapshot time, op perf is not a concern.
#[op2]
#[serde]
pub fn op_snapshot_options(state: &mut OpState) -> SnapshotOptions {
  #[cfg(feature = "hmr")]
  {
    state.try_take::<SnapshotOptions>().unwrap_or_default()
  }
  #[cfg(not(feature = "hmr"))]
  {
    state.take::<SnapshotOptions>()
  }
}
```

But what's that `SnapshotOptions`?

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/ops/bootstrap.rs#L41">denoland/deno: <code>runtime/ops/bootstrap.rs</code></a></div>

```rs
#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct SnapshotOptions {
  pub ts_version: String,
  pub v8_version: &'static str,
  pub target: String, // 👈
}
```

Where does the actual `target` string come from? Before we get to the `Default` implementation for `SnapshotOptions`, let's look at where else it's constructed. There's a manifest-like extension definition macro that defines a `deno_bootstrap` extension. It takes in a `SnapshotOptions` struct.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/ops/bootstrap.rs#L33">denoland/deno: <code>runtime/ops/bootstrap.rs</code></a></div>

```rs
deno_core::extension!(
  deno_bootstrap,
  ops = [
    op_bootstrap_args,
    op_bootstrap_pid,
    op_bootstrap_numcpus,
    op_bootstrap_user_agent,
    op_bootstrap_language,
    op_bootstrap_log_level,
    op_bootstrap_color_depth,
    op_bootstrap_no_color,
    op_bootstrap_stdout_no_color,
    op_bootstrap_stderr_no_color,
    op_bootstrap_unstable_args,
    op_bootstrap_is_from_unconfigured_runtime,
    op_snapshot_options,
  ],
  options = {
    snapshot_options: Option<SnapshotOptions>, // 👈
    is_from_unconfigured_runtime: bool,
  },
  state = |state, options| {
    if let Some(snapshot_options) = options.snapshot_options {
      state.put::<SnapshotOptions>(snapshot_options); // 👈
    }
    state.put(IsFromUnconfiguredRuntime(options.is_from_unconfigured_runtime));
  },
);
```

I'll be honest, I don't know what the type of `state` is or what its lifecycle is. Somehow, though, `state` must be populated with a `SnapshotOptions` since it gets `.take()`-en in the `op_snapshot_options()` function above (sometimes with a default value via `.unwrap_or_default()`). But where does this extension get initialized with a `SnapshotOptions` struct? In the `common_extensions()` convenience function that initializes and returns a bunch of -- you guessed it -- common extensions.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/worker.rs#L1076">denoland/deno: <code>runtime/worker.rs</code></a></div>

```rs
fn common_extensions<
  TInNpmPackageChecker: InNpmPackageChecker + 'static,
  TNpmPackageFolderResolver: NpmPackageFolderResolver + 'static,
  TExtNodeSys: ExtNodeSys + 'static,
>(
  has_snapshot: bool,
  unconfigured_runtime: bool,
) -> Vec<Extension> {
  // NOTE(bartlomieju): ordering is important here, keep it in sync with
  // `runtime/worker.rs`, `runtime/web_worker.rs`, `runtime/snapshot_info.rs`
  // and `runtime/snapshot.rs`!
  vec![
    // -- ✂️ SNIP --
    ops::bootstrap::deno_bootstrap::init(
      has_snapshot.then(Default::default), // 👈
      unconfigured_runtime,
    ),
    // -- ✂️ SNIP --
  ]
}
```

There's also this snapshotting thing that seems to happen at build time, which probably magically bakes in the snapshot options somehow? I'm not really sure. All I know is that these options are somehow stashed and retrieved by the Deno runtime. 🤷

<div><a href="https://github.com/denoland/deno/blob/main/cli/snapshot/build.rs#L22">denoland/deno: <code>cli/snapshot/build.rs</code></a></div>

```rs
let snapshot_options = SnapshotOptions {
  ts_version: shared::TS_VERSION.to_string(),
  v8_version: deno_runtime::deno_core::v8::VERSION_STRING,
  target: std::env::var("TARGET").unwrap(),
};
```

There's also the default implementation, which uses the Rust-provided compile-time OS/arch constants to reconstruct the likely target triple string.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/runtime/ops/bootstrap.rs#L53">denoland/deno: <code>runtime/ops/bootstrap.rs</code></a></div>

```rs
impl Default for SnapshotOptions {
  fn default() -> Self {
    let arch = std::env::consts::ARCH;
    let platform = std::env::consts::OS;
    let target = match platform {
      "macos" => format!("{}-apple-darwin", arch),
      "linux" => format!("{}-unknown-linux-gnu", arch),
      "windows" => format!("{}-pc-windows-msvc", arch),
      rest => format!("{}-{}", arch, rest),
    };

    Self {
      ts_version: "n/a".to_owned(),
      v8_version: deno_core::v8::VERSION_STRING,
      target,
    }
  }
}
```

So that `Deno.build.target` value (which is obtained from Rust via `op_snapshot_options()`) is what the rest of the `Deno.build` object's properties are based on. It's is either passed in at build time via snapshotting or reconstructed from the Rust-provided information at runtime.

```rs
let target = std::env::var("TARGET").unwrap();
```

```rs
let target = match std::env::consts::OS {
  "macos" => format!("{}-apple-darwin", std::env::consts::ARCH),
  "linux" => format!("{}-unknown-linux-gnu", std::env::consts::ARCH),
  "windows" => format!("{}-pc-windows-msvc", std::env::consts::ARCH),
  rest => format!("{}-{}", std::env::consts::ARCH, rest),
};
```

So that's [every Rust target tuple](https://doc.rust-lang.org/stable/rustc/targets/built-in.html) that Deno happens to successfully compile on. I don't know how long a list that is.

The [targets that Deno provides official releases for (as of Deno v2.6.0)](https://github.com/denoland/deno/releases/tag/v2.6.0) are:

- `aarch64-apple-darwin`
- `aarch64-unknown-linux-gnu`
- `x86_64-apple-darwin`
- `x86_64-pc-windows-msvc`
- `x86_64-unknown-linux-gnu`

### `Deno.build.os`

So the final verdict is that `Deno.build.os` is derived from the third component of a `<arch>-<vendor>-<os>-<env>` target tuple which means that **`Deno.build.os` would be `darwin`, `linux`, or `windows`** unless **Deno is compiled for another target** (which is possible). [The official Deno docs for `Deno.build.os`](https://docs.deno.com/api/deno/~/Deno.build.os) state that its type is `"darwin" | "linux" | "android" | "windows" | "freebsd" | "netbsd" | "aix" | "solaris" | "illumos"`.

### `Deno.build.arch`

The `Deno.build.arch` value is a similar story. Given the official targets, the first `arch` component can be **either `aarch64` or `x86_64`**. Other architectures like `riscv64gc` might successfully compile for all I know. The docs say that `Deno.build.arch`'s type is `"aarch64" | "x86_64"`.

### `node:process` `platform` & `node:os` `platform()`

Deno's `node:os` `platform()` function is just a wrapper around `process.platform`.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/os.ts#L241">denoland/deno: <code>ext/node/polyfills/os.ts</code></a></div>

```ts
/** Returns the a string identifying the operating system platform. The value is set at compile time. Possible values are 'darwin', 'linux', and 'win32'. */
export function platform(): string {
  return process.platform;
}
```

Deno's `node:process` `platform` (or `process.platform`) is a variable...

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L82">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
export let platform = isWindows ? "win32" : ""; // initialized during bootstrap
```

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L748">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
/** https://nodejs.org/api/process.html#process_process_platform */
Object.defineProperty(process, "platform", {
  get() {
    return platform;
  },
});
```

...that's initialized at bootstrapping time to the value of `Deno.build.os`. Unless `isWindows` is true; then it's set to `win32` (**not** `windows`).

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L1077">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
// Should be called only once, in `runtime/js/99_main.js` when the runtime is
// bootstrapped.
internals.__bootstrapNodeProcess = function (
  argv0Val: string | undefined,
  args: string[],
  denoVersions: Record<string, string>,
  nodeDebug: string,
  warmup = false,
) {
    // -- ✂️ SNIP --
    arch = arch_();
    platform = isWindows ? "win32" : Deno.build.os; // 👈
    pid = Deno.pid;
    ppid = Deno.ppid;
    initializeDebugEnv(nodeDebug);
    // -- ✂️ SNIP --
};
```

So what's this `isWindows` thing? It's pretty self-explanatory, but let's investigate anyway.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L74">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
import { isAndroid, isWindows } from "ext:deno_node/_util/os.ts";
```

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/_util/os.ts">denoland/deno: <code>ext/node/polyfills/_util/os.ts</code></a></div>

```js
// -- Some parts ✂️snipped for brevity --
import { op_node_build_os } from "ext:core/ops";
export const osType: OSType = op_node_build_os();
export const isWindows = osType === "windows";
```

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/lib.rs#L93">denoland/deno: <code>ext/node/lib.rs</code></a></div>

```rs
#[op2]
#[string]
fn op_node_build_os() -> String {
  env!("TARGET").split('-').nth(2).unwrap().to_string()
}
```

A Rust `TARGET` string is `<arch>-<vendor>-<os>-<env>`. `op_node_build_os()` just returns the `os` part of that tuple. It seems like it's the same value as `Deno.build.os`, just retrieved a bit differently. I'm not really sure why `Deno.build.os` wasn't used. Maybe there's a good reason. 🤷

So that means that `process.platform`/`os.platform()` is the **same value as `Deno.build.os` _except_ it's `win32` instead of `windows`**. That's a string value of `darwin`, `linux`, `win32`, or any [other valid Rust target OS name](https://docs.rs/platforms/latest/platforms/target/enum.OS.html).

### `node:process` `arch` & `node:os` `arch()`

Deno's `node:os` `arch()` polyfill is just a wrapper around `node:process` `arch`.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/os.ts#L115">denoland/deno: <code>ext/node/polyfills/os.ts</code></a></div>

```js
export function arch(): string {
  return process.arch;
}
```

Deno's `node:process` `arch` (or `process.arch`) is a variable...

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L80">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
export let arch = "";
```

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L613">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
/** https://nodejs.org/api/process.html#process_process_arch */
Object.defineProperty(process, "arch", {
  get() {
    return arch;
  },
});
```

...that's initialized at bootstrapping time to the result of an internal `arch()` getter.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L1077">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
// Should be called only once, in `runtime/js/99_main.js` when the runtime is
// bootstrapped.
internals.__bootstrapNodeProcess = function (
  argv0Val: string | undefined,
  args: string[],
  denoVersions: Record<string, string>,
  nodeDebug: string,
  warmup = false,
) {
    // -- ✂️ SNIP --
    arch = arch_(); // 👈
    platform = isWindows ? "win32" : Deno.build.os;
    pid = Deno.pid;
    ppid = Deno.ppid;
    initializeDebugEnv(nodeDebug);
    // -- ✂️ SNIP --
};
```

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/process.ts#L46">denoland/deno: <code>ext/node/polyfills/process.ts</code></a></div>

```js
import {
  arch as arch_, // 👈
  chdir,
  cwd,
  env,
  nextTick as _nextTick,
  version,
  versions,
} from "ext:deno_node/_process/process.ts";
```

That `arch()` getter is a very well-defined and limited function that **throws an error on all but three architectures**.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/_process/process.ts#L33">denoland/deno: <code>ext/node/polyfills/_process/process.ts</code></a></div>

```ts
/** Returns the operating system CPU architecture for which the Deno binary was compiled */
export function arch(): string {
  if (build.arch == "x86_64") {
    return "x64";
  } else if (build.arch == "aarch64") {
    return "arm64";
  } else if (build.arch == "riscv64gc") {
    return "riscv64";
  } else {
    throw new Error("unreachable");
  }
}
```

Therefore, **the exact guaranteed type for `node:process` `arch` or `node:os` `arch()` in Deno is `"x64" | "arm64" | "riscv64"`**. This is far more limited than Node.js' implementation of these APIs.

> Possible values are `'arm'`, `'arm64'`, `'ia32'`, `'loong64'`, `'mips'`, `'mipsel'`, `'ppc64'`, `'riscv64'`, `'s390x'`, and `'x64'`. The return value is equivalent to `process.arch`.

&mdash; [Node.js `node:os` `arch()` docs](https://nodejs.org/api/os.html#osarch)

### `node:os` `machine()`

`machine()` is just `Deno.build.arch` with `aarch64` mapped to `arm64` instead.

<div><a href="https://github.com/denoland/deno/blob/d17ae8ca9b0777c64934ae332b83ff0996a9fae0/ext/node/polyfills/os.ts#L258">denoland/deno: <code>ext/node/polyfills/os.ts</code></a></div>

```ts
/** Returns the machine type as a string */
export function machine(): string {
  if (Deno.build.arch == "aarch64") {
    return "arm64";
  }

  return Deno.build.arch;
}
```

The full type is _probably_ `"arm64" | "x86_64" | "riscv64gc"` but it could have more possible values.
