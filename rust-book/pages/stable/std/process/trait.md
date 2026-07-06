---
type: Web Page
title: Termination in std::process - Rust
description: A trait for implementing arbitrary return types in the `main` function.
resource: https://doc.rust-lang.org/stable/std/process/trait.Termination.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

```
pub trait Termination {
    // Required method
    fn report(self) -> ExitCode;
}
```
## Expand description

A trait for implementing arbitrary return types in the `main` function.

The C-main function only supports returning integers.
So, every type implementing the `Termination` trait has to be converted
to an integer.

The default implementations are returning `libc::EXIT_SUCCESS` to indicate
a successful execution. In case of a failure, `libc::EXIT_FAILURE` is returned.

Because different runtimes have different specifications on the return value
of the `main` function, this trait is likely to be available only on
standard library’s runtime for convenience. Other runtimes are not required
to provide similar functionality.

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/process/trait.Termination.html
