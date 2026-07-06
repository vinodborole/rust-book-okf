---
type: Web Page
title: Stdin in std::io - Rust
description: A handle to the standard input stream of a process.
resource: https://doc.rust-lang.org/stable/std/io/struct.Stdin.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

`pub struct Stdin { /* private fields */ }`## Expand description

A handle to the standard input stream of a process.

Each handle is a shared reference to a global buffer of input data to this
process. A handle can be `lock`’d to gain full access to `BufRead` methods
(e.g., `.lines()`). Reads to this handle are otherwise locked with respect
to other reads.

This handle implements the `Read` trait, but beware that concurrent reads
of `Stdin` must be executed with care.

Created by the `io::stdin` method.

#### §Note: Windows Portability Considerations

When operating in a console, the Windows implementation of this stream does not support non-UTF-8 byte sequences. Attempting to read bytes that are not valid UTF-8 will return an error.

In a process with a detached console, such as one using
`#![windows_subsystem = "windows"]`, or in a child process spawned from such a process,
the contained handle will be null. In such cases, the standard library’s `Read` and
`Write` will do nothing and silently succeed. All other I/O operations, via the
standard library or via raw Windows API calls, will fail.

## §Examples

## Implementations§

Source§### impl Stdin

 

### impl Stdin

1.0.0 · Source#### pub fn lock(&self) -> StdinLock<'static> ⓘ

 

#### pub fn lock(&self) -> StdinLock<'static> ⓘ

1.0.0 · Source#### pub fn read_line(&self, buf: &mut String) -> Result<usize>

 

#### pub fn read_line(&self, buf: &mut String) -> Result<usize>

Locks this handle and reads a line of input, appending it to the specified buffer.

For detailed semantics of this method, see the documentation on
`BufRead::read_line`. In particular:

- Previous content of the buffer will be preserved. To avoid appending
to the buffer, you need to `clear`it first.
- The trailing newline character, if any, is included in the buffer.

##### §Examples

```
use std::io;
let mut input = String::new();
match io::stdin().read_line(&mut input) {
    Ok(n) => {
        println!("{n} bytes read");
        println!("{input}");
    }
    Err(error) => println!("error: {error}"),
}
```
You can run the example one of two ways:

- Pipe some text to it, e.g., `printf foo | path/to/executable`
- Give it text interactively by running the executable directly, in which case it will wait for the Enter key to be pressed before continuing

## Trait Implementations§

1.63.0 · Source§### impl AsFd for Stdin

Available on **Unix or Hermit or Trusty or WASI or Motor OS** only. 

### impl AsFd for Stdin

**Unix or Hermit or Trusty or WASI or Motor OS**only.

Source§#### fn as_fd(&self) -> BorrowedFd<'_>

 

#### fn as_fd(&self) -> BorrowedFd<'_>

1.63.0 · Source§### impl AsHandle for Stdin

Available on **Windows** only. 

### impl AsHandle for Stdin

**Windows**only.

Source§#### fn as_handle(&self) -> BorrowedHandle<'_>

 

#### fn as_handle(&self) -> BorrowedHandle<'_>

1.21.0 · Source§### impl AsRawFd for Stdin

Available on **(Unix or Hermit or Trusty or WASI or Motor OS) and non-Trusty** only. 

### impl AsRawFd for Stdin

**(Unix or Hermit or Trusty or WASI or Motor OS) and non-Trusty**only.

1.21.0 · Source§### impl AsRawHandle for Stdin

Available on **Windows** only. 

### impl AsRawHandle for Stdin

**Windows**only.

Source§#### fn as_raw_handle(&self) -> RawHandle

 

#### fn as_raw_handle(&self) -> RawHandle

1.70.0 · Source§### impl IsTerminal for Stdin

 

### impl IsTerminal for Stdin

Source§#### fn is_terminal(&self) -> bool

 

#### fn is_terminal(&self) -> bool

`true` if the descriptor/handle refers to a terminal/tty. Read more1.78.0 · Source§### impl Read for &Stdin

 

### impl Read for &Stdin

Source§#### fn read(&mut self, buf: &mut [u8]) -> Result<usize>

 

#### fn read(&mut self, buf: &mut [u8]) -> Result<usize>

Source§#### fn read_buf(&mut self, buf: BorrowedCursor<'_>) -> Result<()>

 

#### fn read_buf(&mut self, buf: BorrowedCursor<'_>) -> Result<()>

`read_buf` #78485)Source§#### fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize>

 

#### fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize>

`read`, except that it reads into a slice of buffers. Read moreSource§#### fn is_read_vectored(&self) -> bool

 

#### fn is_read_vectored(&self) -> bool

`can_vector` #69941)Source§#### fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize>

 

#### fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize>

`buf`. Read moreSource§#### fn read_to_string(&mut self, buf: &mut String) -> Result<usize>

 

#### fn read_to_string(&mut self, buf: &mut String) -> Result<usize>

`buf`. Read moreSource§#### fn read_exact(&mut self, buf: &mut [u8]) -> Result<()>

 

#### fn read_exact(&mut self, buf: &mut [u8]) -> Result<()>

`buf`. Read moreSource§#### fn read_buf_exact(&mut self, cursor: BorrowedCursor<'_>) -> Result<()>

 

#### fn read_buf_exact(&mut self, cursor: BorrowedCursor<'_>) -> Result<()>

`read_buf` #78485)`cursor`. Read more1.0.0 · Source§#### fn by_ref(&mut self) -> &mut Selfwhere
    Self: Sized,

 

#### fn by_ref(&mut self) -> &mut Selfwhere
    Self: Sized,

`Read`. Read more1.0.0 · Source§#### fn chain<R: Read>(self, next: R) -> Chain<Self, R> ⓘwhere
    Self: Sized,

 

#### fn chain<R: Read>(self, next: R) -> Chain<Self, R> ⓘwhere
    Self: Sized,

1.0.0 · Source§### impl Read for Stdin

 

### impl Read for Stdin

Source§#### fn read(&mut self, buf: &mut [u8]) -> Result<usize>

 

#### fn read(&mut self, buf: &mut [u8]) -> Result<usize>

Source§#### fn read_buf(&mut self, buf: BorrowedCursor<'_>) -> Result<()>

 

#### fn read_buf(&mut self, buf: BorrowedCursor<'_>) -> Result<()>

`read_buf` #78485)Source§#### fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize>

 

#### fn read_vectored(&mut self, bufs: &mut [IoSliceMut<'_>]) -> Result<usize>

`read`, except that it reads into a slice of buffers. Read moreSource§#### fn is_read_vectored(&self) -> bool

 

#### fn is_read_vectored(&self) -> bool

`can_vector` #69941)Source§#### fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize>

 

#### fn read_to_end(&mut self, buf: &mut Vec<u8>) -> Result<usize>

`buf`. Read moreSource§#### fn read_to_string(&mut self, buf: &mut String) -> Result<usize>

 

#### fn read_to_string(&mut self, buf: &mut String) -> Result<usize>

`buf`. Read moreSource§#### fn read_exact(&mut self, buf: &mut [u8]) -> Result<()>

 

#### fn read_exact(&mut self, buf: &mut [u8]) -> Result<()>

`buf`. Read moreSource§#### fn read_buf_exact(&mut self, cursor: BorrowedCursor<'_>) -> Result<()>

 

#### fn read_buf_exact(&mut self, cursor: BorrowedCursor<'_>) -> Result<()>

`read_buf` #78485)`cursor`. Read more1.0.0 · Source§#### fn by_ref(&mut self) -> &mut Selfwhere
    Self: Sized,

 

#### fn by_ref(&mut self) -> &mut Selfwhere
    Self: Sized,

`Read`. Read more1.0.0 · Source§#### fn chain<R: Read>(self, next: R) -> Chain<Self, R> ⓘwhere
    Self: Sized,

 

#### fn chain<R: Read>(self, next: R) -> Chain<Self, R> ⓘwhere
    Self: Sized,

Source§### impl StdioExt for Stdin

Available on **Unix** only. 

### impl StdioExt for Stdin

**Unix**only.

Source§#### fn set_fd<T: Into<OwnedFd>>(&mut self, fd: T) -> Result<()>

 

#### fn set_fd<T: Into<OwnedFd>>(&mut self, fd: T) -> Result<()>

`stdio_swap` #150667)`fd`. Read more

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/io/struct.Stdin.html
