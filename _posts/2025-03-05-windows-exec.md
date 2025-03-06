---
title: execve() on Windows doesn't replace the process
---

I mean, it _sorta_ does but not in the way you'd expect.

On Linux systems I know that when you `exec awesome "Hello, World!"` it **replaces** the shell with that command and will terminate with whatever exit code `awesome "Hello, World!"` returns. All signals are passed to the new replaced command as well. Effectively the new `awesome` process would subsume the `sh` instance.

This same thing applies to programs that use the `execve()` POSIX C Standard Library function.

> ```c
> int execve(char *pathname, char *argv[], char *envp[]);
> ```
>
> execve() executes the program referred to by pathname.  This causes the program that is currently being run by the calling process to be replaced with a new program, with newly initialized stack, heap, and (initialized and uninitialized) data segments.

https://man7.org/linux/man-pages/man2/execve.2.html

Windows offers a `execve()` POSIX alias to the underlying Microsoft-specific `_execve()` function. This function name _implies_ that it should do something similar to the effects of `execve()` on Linux/POSiX.

> ```c
> intptr_t _execve(
>    char *cmdname,
>    char **argv,
>    char **envp
> );
> ```

https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/execve-wexecve?view=msvc-170

...but it doesn't actually document that it's "like POSIX execve()". Instead it's documented as:

> Loads and executes new child processes.
>
> ...loads and executes a new process, passing an array of pointers to command-line arguments and an array of pointers to environment settings.

Notice how the docs make no mention of "replaces the current process". That's key. **They _don't_ replace the current process.** I mean, they kinda do. Let me show you:

```
PS C:\TempTest> ptime go run y.go
=== go run exec.go ===
execing "go version"
Execution time: 2.086 s
PS C:\TempTest> go version go1.23.4 windows/amd64

```

Notice how the "go version ..." output from `go version` was printed _after_ the `go run exec.go` **finished** and returned `0`. The `ptime` parent process that spawned `go run exec.go` **did not** capture the stdout of the `go version` command that was `execve()`-ed by `go run exec.go`.

From this the behaviour of `execve()` is:

1. Use whatever normal process-creation Windows API to create a new process
2. Set the parent of the new process to the parent of the current process
3. If all goes well, terminate the current process

From a cursory Google search, Windows does not seem to support the `execve()` replace-me functionality that Linux/POSIX does natively in any way. The `execve()` C runtime function is just doing the best it can to fit the "end the process and spawn a new one in its place" part of the `execve()` logic. In doing so, though, it misses the "exit with the exit code of the spawned process" and "forward all signals to the new process" parts.

Here's an issue from cosmopolitan that reiterates this issue: https://github.com/jart/cosmopolitan/issues/1253

If you actually want to "exit with this process' exit code and forward all signals" on Windows then you need to do the usual `exec.Command()`, `child_process.spawn()`, `std::process::Command::new()`, etc. and handle the exit code & signals for Windows correctly.

Cargo does some magic Ctrl+C interception to forward Ctrl+C to the child process.

> On Windows this isn’t technically possible. Instead we emulate it to the best of our ability. One aspect we fix here is that we specify a handler for the Ctrl-C handler. In doing so (and by effectively ignoring it) we should emulate proxying Ctrl-C handling to the application at hand, which will either terminate or handle it itself. According to Microsoft’s documentation at https://docs.microsoft.com/en-us/windows/console/ctrl-c-and-ctrl-break-signals. the Ctrl-C signal is sent to all processes attached to a terminal, which should include our child process. If the child terminates then we’ll reap them in Cargo pretty quickly, and if the child handles the signal then we won’t terminate (and we shouldn’t!) until the process itself later exits.

https://docs.rs/cargo-util/latest/cargo_util/struct.ProcessBuilder.html#method.exec_replace

```rs
unsafe extern "system" fn ctrlc_handler(_: u32) -> BOOL {
    // Do nothing; let the child process handle it.
    TRUE
}

pub fn exec_replace(process_builder: &ProcessBuilder) -> Result<()> {
    unsafe {
        if SetConsoleCtrlHandler(Some(ctrlc_handler), TRUE) == FALSE {
            return Err(ProcessError::new("Could not set Ctrl-C handler.", None, None).into());
        }
    }

    // Just execute the process as normal.
    process_builder.exec()
}
```

https://docs.rs/cargo-util/latest/src/cargo_util/process_builder.rs.html#605
