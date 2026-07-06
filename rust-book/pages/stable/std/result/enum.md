---
type: Web Page
title: Result in std::result - Rust
description: '`Result` is a type that represents either success (`Ok`) or failure
  (`Err`).'
resource: https://doc.rust-lang.org/stable/std/result/enum.Result.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

```
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
## Expand description

`Result` is a type that represents either success (`Ok`) or failure (`Err`).

See the module documentation for details.

## Variants§

## Implementations§

Source§### impl<T, E> Result<T, E>

 

### impl<T, E> Result<T, E>

1.70.0 (const: unstable) · Source#### pub fn is_ok_and<F>(self, f: F) -> bool

 

#### pub fn is_ok_and<F>(self, f: F) -> bool

Returns `true` if the result is `Ok` and the value inside of it matches a predicate.

##### §Examples

```
let x: Result<u32, &str> = Ok(2);
assert_eq!(x.is_ok_and(|x| x > 1), true);
let x: Result<u32, &str> = Ok(0);
assert_eq!(x.is_ok_and(|x| x > 1), false);
let x: Result<u32, &str> = Err("hey");
assert_eq!(x.is_ok_and(|x| x > 1), false);
let x: Result<String, &str> = Ok("ownership".to_string());
assert_eq!(x.as_ref().is_ok_and(|x| x.len() > 1), true);
println!("still alive {:?}", x);
```
1.70.0 (const: unstable) · Source#### pub fn is_err_and<F>(self, f: F) -> bool

 

#### pub fn is_err_and<F>(self, f: F) -> bool

Returns `true` if the result is `Err` and the value inside of it matches a predicate.

##### §Examples

```
use std::io::{Error, ErrorKind};
let x: Result<u32, Error> = Err(Error::new(ErrorKind::NotFound, "!"));
assert_eq!(x.is_err_and(|x| x.kind() == ErrorKind::NotFound), true);
let x: Result<u32, Error> = Err(Error::new(ErrorKind::PermissionDenied, "!"));
assert_eq!(x.is_err_and(|x| x.kind() == ErrorKind::NotFound), false);
let x: Result<u32, Error> = Ok(123);
assert_eq!(x.is_err_and(|x| x.kind() == ErrorKind::NotFound), false);
let x: Result<u32, String> = Err("ownership".to_string());
assert_eq!(x.as_ref().is_err_and(|x| x.len() > 1), true);
println!("still alive {:?}", x);
```
1.0.0 (const: 1.48.0) · Source#### pub const fn as_ref(&self) -> Result<&T, &E>

 

#### pub const fn as_ref(&self) -> Result<&T, &E>

Converts from `&Result<T, E>` to `Result<&T, &E>`.

Produces a new `Result`, containing a reference
into the original, leaving the original in place.

##### §Examples

1.0.0 (const: 1.83.0) · Source#### pub const fn as_mut(&mut self) -> Result<&mut T, &mut E>

 

#### pub const fn as_mut(&mut self) -> Result<&mut T, &mut E>

Converts from `&mut Result<T, E>` to `Result<&mut T, &mut E>`.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn map<U, F>(self, op: F) -> Result<U, E>where
    F: FnOnce(T) -> U,

 

#### pub fn map<U, F>(self, op: F) -> Result<U, E>where
    F: FnOnce(T) -> U,

1.41.0 (const: unstable) · Source#### pub fn map_or<U, F>(self, default: U, f: F) -> Uwhere
    F: FnOnce(T) -> U,

 

#### pub fn map_or<U, F>(self, default: U, f: F) -> Uwhere
    F: FnOnce(T) -> U,

Returns the provided default (if `Err`), or
applies a function to the contained value (if `Ok`).

Arguments passed to `map_or` are eagerly evaluated; if you are passing
the result of a function call, it is recommended to use `map_or_else`,
which is lazily evaluated.

##### §Examples

1.41.0 (const: unstable) · Source#### pub fn map_or_else<U, D, F>(self, default: D, f: F) -> U

 

#### pub fn map_or_else<U, D, F>(self, default: D, f: F) -> U

Source#### pub const fn map_or_default<U, F>(self, f: F) -> U

 🔬This is a nightly-only experimental API. (`result_option_map_or_default` #138099)

#### pub const fn map_or_default<U, F>(self, f: F) -> U

`result_option_map_or_default` #138099)Maps a `Result<T, E>` to a `U` by applying function `f` to the contained
value if the result is `Ok`, otherwise if `Err`, returns the
default value for the type `U`.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn map_err<F, O>(self, op: O) -> Result<T, F>where
    O: FnOnce(E) -> F,

 

#### pub fn map_err<F, O>(self, op: O) -> Result<T, F>where
    O: FnOnce(E) -> F,

1.76.0 (const: unstable) · Source#### pub fn inspect_err<F>(self, f: F) -> Result<T, E>

 

#### pub fn inspect_err<F>(self, f: F) -> Result<T, E>

1.47.0 (const: unstable) · Source#### pub fn as_deref(&self) -> Result<&<T as Deref>::Target, &E>where
    T: Deref,

 

#### pub fn as_deref(&self) -> Result<&<T as Deref>::Target, &E>where
    T: Deref,

1.47.0 (const: unstable) · Source#### pub fn as_deref_mut(&mut self) -> Result<&mut <T as Deref>::Target, &mut E>where
    T: DerefMut,

 

#### pub fn as_deref_mut(&mut self) -> Result<&mut <T as Deref>::Target, &mut E>where
    T: DerefMut,

Converts from `Result<T, E>` (or `&mut Result<T, E>`) to `Result<&mut <T as DerefMut>::Target, &mut E>`.

Coerces the `Ok` variant of the original `Result` via `DerefMut`
and returns the new `Result`.

##### §Examples

```
let mut s = "HELLO".to_string();
let mut x: Result<String, u32> = Ok("hello".to_string());
let y: Result<&mut str, &mut u32> = Ok(&mut s);
assert_eq!(x.as_deref_mut().map(|x| { x.make_ascii_uppercase(); x }), y);
let mut i = 42;
let mut x: Result<String, u32> = Err(42);
let y: Result<&mut str, &mut u32> = Err(&mut i);
assert_eq!(x.as_deref_mut().map(|x| { x.make_ascii_uppercase(); x }), y);
```
1.0.0 (const: unstable) · Source#### pub fn iter(&self) -> Iter<'_, T> ⓘ

 

#### pub fn iter(&self) -> Iter<'_, T> ⓘ

Returns an iterator over the possibly contained value.

The iterator yields one value if the result is `Result::Ok`, otherwise none.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn iter_mut(&mut self) -> IterMut<'_, T> ⓘ

 

#### pub fn iter_mut(&mut self) -> IterMut<'_, T> ⓘ

Returns a mutable iterator over the possibly contained value.

The iterator yields one value if the result is `Result::Ok`, otherwise none.

##### §Examples

1.4.0 · Source#### pub fn expect(self, msg: &str) -> Twhere
    E: Debug,

 

#### pub fn expect(self, msg: &str) -> Twhere
    E: Debug,

Returns the contained `Ok` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged.
Instead, prefer to use pattern matching and handle the `Err`
case explicitly, or call `unwrap_or`, `unwrap_or_else`, or
`unwrap_or_default`.

##### §Panics

Panics if the value is an `Err`, with a panic message including the
passed message, and the content of the `Err`.

##### §Examples

```
let x: Result<u32, &str> = Err("emergency failure");
x.expect("Testing expect"); // panics with `Testing expect: emergency failure`
```
##### §Recommended Message Style

We recommend that `expect` messages are used to describe the reason you
*expect* the `Result` should be `Ok`.

```
let path = std::env::var("IMPORTANT_PATH")
    .expect("env variable `IMPORTANT_PATH` should be set by `wrapper_script.sh`");
```
**Hint**: If you’re having trouble remembering how to phrase expect
error messages remember to focus on the word “should” as in “env
variable should be set by blah” or “the given binary should be available
and executable by the current user”.

For more detail on expect message styles and the reasoning behind our recommendation please
refer to the section on “Common Message
Styles” in the
`std::error` module docs.

1.0.0 · Source#### pub fn unwrap(self) -> Twhere
    E: Debug,

 

#### pub fn unwrap(self) -> Twhere
    E: Debug,

Returns the contained `Ok` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged. Panics are meant for unrecoverable errors, and may abort the entire program.

Instead, prefer to use the `?` (try) operator, or pattern matching
to handle the `Err` case explicitly, or call `unwrap_or`,
`unwrap_or_else`, or `unwrap_or_default`.

##### §Panics

Panics if the value is an `Err`, with a panic message provided by the
`Err`’s value.

##### §Examples

Basic usage:

```
let x: Result<u32, &str> = Err("emergency failure");
x.unwrap(); // panics with `emergency failure`
```
1.16.0 (const: unstable) · Source#### pub fn unwrap_or_default(self) -> Twhere
    T: Default,

 

#### pub fn unwrap_or_default(self) -> Twhere
    T: Default,

Returns the contained `Ok` value or a default

Consumes the `self` argument then, if `Ok`, returns the contained
value, otherwise if `Err`, returns the default value for that
type.

##### §Examples

Converts a string to an integer, turning poorly-formed strings
into 0 (the default value for integers). `parse` converts
a string to any other type that implements `FromStr`, returning an
`Err` on error.

1.17.0 · Source#### pub fn expect_err(self, msg: &str) -> Ewhere
    T: Debug,

 

#### pub fn expect_err(self, msg: &str) -> Ewhere
    T: Debug,

1.0.0 · Source#### pub fn unwrap_err(self) -> Ewhere
    T: Debug,

 

#### pub fn unwrap_err(self) -> Ewhere
    T: Debug,

Source#### pub const fn into_ok(self) -> T

 🔬This is a nightly-only experimental API. (`unwrap_infallible` #61695)

#### pub const fn into_ok(self) -> T

`unwrap_infallible` #61695)Returns the contained `Ok` value, but never panics.

Unlike `unwrap`, this method is known to never panic on the
result types it is implemented for. Therefore, it can be used
instead of `unwrap` as a maintainability safeguard that will fail
to compile if the error type of the `Result` is later changed
to an error that can actually occur.

##### §Examples

Source#### pub const fn into_err(self) -> E

 🔬This is a nightly-only experimental API. (`unwrap_infallible` #61695)

#### pub const fn into_err(self) -> E

`unwrap_infallible` #61695)Returns the contained `Err` value, but never panics.

Unlike `unwrap_err`, this method is known to never panic on the
result types it is implemented for. Therefore, it can be used
instead of `unwrap_err` as a maintainability safeguard that will fail
to compile if the ok type of the `Result` is later changed
to a type that can actually occur.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn and<U>(self, res: Result<U, E>) -> Result<U, E>

 

#### pub fn and<U>(self, res: Result<U, E>) -> Result<U, E>

Returns `res` if the result is `Ok`, otherwise returns the `Err` value of `self`.

Arguments passed to `and` are eagerly evaluated; if you are passing the
result of a function call, it is recommended to use `and_then`, which is
lazily evaluated.

##### §Examples

```
let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("late error"));
let x: Result<u32, &str> = Err("early error");
let y: Result<&str, &str> = Ok("foo");
assert_eq!(x.and(y), Err("early error"));
let x: Result<u32, &str> = Err("not a 2");
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("not a 2"));
let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Ok("different result type");
assert_eq!(x.and(y), Ok("different result type"));
```
1.0.0 (const: unstable) · Source#### pub fn and_then<U, F>(self, op: F) -> Result<U, E>

 

#### pub fn and_then<U, F>(self, op: F) -> Result<U, E>

Calls `op` if the result is `Ok`, otherwise returns the `Err` value of `self`.

This function can be used for control flow based on `Result` values.

##### §Examples

```
fn sq_then_to_string(x: u32) -> Result<String, &'static str> {
    x.checked_mul(x).map(|sq| sq.to_string()).ok_or("overflowed")
}
assert_eq!(Ok(2).and_then(sq_then_to_string), Ok(4.to_string()));
assert_eq!(Ok(1_000_000).and_then(sq_then_to_string), Err("overflowed"));
assert_eq!(Err("not a number").and_then(sq_then_to_string), Err("not a number"));
```
Often used to chain fallible operations that may return `Err`.

```
use std::{io::ErrorKind, path::Path};
// Note: on Windows "/" maps to "C:\"
let root_modified_time = Path::new("/").metadata().and_then(|md| md.modified());
assert!(root_modified_time.is_ok());
let should_fail = Path::new("/bad/path").metadata().and_then(|md| md.modified());
assert!(should_fail.is_err());
assert_eq!(should_fail.unwrap_err().kind(), ErrorKind::NotFound);
```
1.0.0 (const: unstable) · Source#### pub fn or<F>(self, res: Result<T, F>) -> Result<T, F>

 

#### pub fn or<F>(self, res: Result<T, F>) -> Result<T, F>

Returns `res` if the result is `Err`, otherwise returns the `Ok` value of `self`.

Arguments passed to `or` are eagerly evaluated; if you are passing the
result of a function call, it is recommended to use `or_else`, which is
lazily evaluated.

##### §Examples

```
let x: Result<u32, &str> = Ok(2);
let y: Result<u32, &str> = Err("late error");
assert_eq!(x.or(y), Ok(2));
let x: Result<u32, &str> = Err("early error");
let y: Result<u32, &str> = Ok(2);
assert_eq!(x.or(y), Ok(2));
let x: Result<u32, &str> = Err("not a 2");
let y: Result<u32, &str> = Err("late error");
assert_eq!(x.or(y), Err("late error"));
let x: Result<u32, &str> = Ok(2);
let y: Result<u32, &str> = Ok(100);
assert_eq!(x.or(y), Ok(2));
```
1.0.0 (const: unstable) · Source#### pub fn or_else<F, O>(self, op: O) -> Result<T, F>

 

#### pub fn or_else<F, O>(self, op: O) -> Result<T, F>

Calls `op` if the result is `Err`, otherwise returns the `Ok` value of `self`.

This function can be used for control flow based on result values.

##### §Examples

```
fn sq(x: u32) -> Result<u32, u32> { Ok(x * x) }
fn err(x: u32) -> Result<u32, u32> { Err(x) }
assert_eq!(Ok(2).or_else(sq).or_else(sq), Ok(2));
assert_eq!(Ok(2).or_else(err).or_else(sq), Ok(2));
assert_eq!(Err(3).or_else(sq).or_else(err), Ok(9));
assert_eq!(Err(3).or_else(err).or_else(err), Err(3));
```
1.0.0 (const: unstable) · Source#### pub fn unwrap_or(self, default: T) -> T

 

#### pub fn unwrap_or(self, default: T) -> T

Returns the contained `Ok` value or a provided default.

Arguments passed to `unwrap_or` are eagerly evaluated; if you are passing
the result of a function call, it is recommended to use `unwrap_or_else`,
which is lazily evaluated.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn unwrap_or_else<F>(self, op: F) -> Twhere
    F: FnOnce(E) -> T,

 

#### pub fn unwrap_or_else<F>(self, op: F) -> Twhere
    F: FnOnce(E) -> T,

1.58.0 (const: unstable) · Source#### pub unsafe fn unwrap_unchecked(self) -> T

 

#### pub unsafe fn unwrap_unchecked(self) -> T

1.58.0 · Source#### pub unsafe fn unwrap_err_unchecked(self) -> E

 

#### pub unsafe fn unwrap_err_unchecked(self) -> E

Source§### impl<T, E> Result<&T, E>

 

### impl<T, E> Result<&T, E>

Source§### impl<T, E> Result<&mut T, E>

 

### impl<T, E> Result<&mut T, E>

1.59.0 (const: 1.83.0) · Source#### pub const fn copied(self) -> Result<T, E>where
    T: Copy,

 

#### pub const fn copied(self) -> Result<T, E>where
    T: Copy,

Maps a `Result<&mut T, E>` to a `Result<T, E>` by copying the contents of the
`Ok` part.

##### §Examples

Source§### impl<T, E> Result<Option<T>, E>

 

### impl<T, E> Result<Option<T>, E>

Source§### impl<T, E> Result<Result<T, E>, E>

 

### impl<T, E> Result<Result<T, E>, E>

1.89.0 (const: 1.89.0) · Source#### pub const fn flatten(self) -> Result<T, E>

 

#### pub const fn flatten(self) -> Result<T, E>

Converts from `Result<Result<T, E>, E>` to `Result<T, E>`

##### §Examples

```
let x: Result<Result<&'static str, u32>, u32> = Ok(Ok("hello"));
assert_eq!(Ok("hello"), x.flatten());
let x: Result<Result<&'static str, u32>, u32> = Ok(Err(6));
assert_eq!(Err(6), x.flatten());
let x: Result<Result<&'static str, u32>, u32> = Err(6);
assert_eq!(Err(6), x.flatten());
```
Flattening only removes one level of nesting at a time:

## Trait Implementations§

1.0.0 · Source§### impl<A, E, V> FromIterator<Result<A, E>> for Result<V, E>where
    V: FromIterator<A>,

 

### impl<A, E, V> FromIterator<Result<A, E>> for Result<V, E>where
    V: FromIterator<A>,

Source§#### fn from_iter<I>(iter: I) -> Result<V, E>where
    I: IntoIterator<Item = Result<A, E>>,

 

#### fn from_iter<I>(iter: I) -> Result<V, E>where
    I: IntoIterator<Item = Result<A, E>>,

Takes each element in the `Iterator`: if it is an `Err`, no further
elements are taken, and the `Err` is returned. Should no `Err` occur, a
container with the values of each `Result` is returned.

Here is an example which increments every integer in a vector, checking for overflow:

```
let v = vec![1, 2];
let res: Result<Vec<u32>, &'static str> = v.iter().map(|x: &u32|
    x.checked_add(1).ok_or("Overflow!")
).collect();
assert_eq!(res, Ok(vec![2, 3]));
```
Here is another example that tries to subtract one from another list of integers, this time checking for underflow:

```
let v = vec![1, 2, 0];
let res: Result<Vec<u32>, &'static str> = v.iter().map(|x: &u32|
    x.checked_sub(1).ok_or("Underflow!")
).collect();
assert_eq!(res, Err("Underflow!"));
```
Here is a variation on the previous example, showing that no
further elements are taken from `iter` after the first `Err`.

```
let v = vec![3, 2, 1, 10];
let mut shared = 0;
let res: Result<Vec<u32>, &'static str> = v.iter().map(|x: &u32| {
    shared += x;
    x.checked_sub(2).ok_or("Underflow!")
}).collect();
assert_eq!(res, Err("Underflow!"));
assert_eq!(shared, 6);
```
Since the third element caused an underflow, no further elements were taken,
so the final value of `shared` is 6 (= `3 + 2 + 1`), not 16.

Source§### impl<T, E, F> FromResidual<Result<Infallible, E>> for Poll<Option<Result<T, F>>>where
    F: From<E>,

 

### impl<T, E, F> FromResidual<Result<Infallible, E>> for Poll<Option<Result<T, F>>>where
    F: From<E>,

Source§#### fn from_residual(x: Result<Infallible, E>) -> Poll<Option<Result<T, F>>>

 

#### fn from_residual(x: Result<Infallible, E>) -> Poll<Option<Result<T, F>>>

`try_trait_v2` #84277)`Residual` type. Read moreSource§### impl<T, E, F> FromResidual<Result<Infallible, E>> for Poll<Result<T, F>>where
    F: From<E>,

 

### impl<T, E, F> FromResidual<Result<Infallible, E>> for Poll<Result<T, F>>where
    F: From<E>,

Source§#### fn from_residual(x: Result<Infallible, E>) -> Poll<Result<T, F>>

 

#### fn from_residual(x: Result<Infallible, E>) -> Poll<Result<T, F>>

`try_trait_v2` #84277)`Residual` type. Read moreSource§### impl<T, E, F> FromResidual<Result<Infallible, E>> for Result<T, F>where
    F: From<E>,

 

### impl<T, E, F> FromResidual<Result<Infallible, E>> for Result<T, F>where
    F: From<E>,

Source§#### fn from_residual(residual: Result<Infallible, E>) -> Result<T, F>

 

#### fn from_residual(residual: Result<Infallible, E>) -> Result<T, F>

`try_trait_v2` #84277)`Residual` type. Read more1.4.0 · Source§### impl<'a, T, E> IntoIterator for &'a Result<T, E>

 

### impl<'a, T, E> IntoIterator for &'a Result<T, E>

1.4.0 · Source§### impl<'a, T, E> IntoIterator for &'a mut Result<T, E>

 

### impl<'a, T, E> IntoIterator for &'a mut Result<T, E>

1.0.0 · Source§### impl<T, E> IntoIterator for Result<T, E>

 

### impl<T, E> IntoIterator for Result<T, E>

1.0.0 (const: unstable) · Source§### impl<T, E> Ord for Result<T, E>

 

### impl<T, E> Ord for Result<T, E>

1.21.0 · Source§#### fn max(self, other: Self) -> Selfwhere
    Self: Sized,

 

#### fn max(self, other: Self) -> Selfwhere
    Self: Sized,

1.0.0 (const: unstable) · Source§### impl<T, E> PartialOrd for Result<T, E>where
    T: PartialOrd,
    E: PartialOrd,

 

### impl<T, E> PartialOrd for Result<T, E>where
    T: PartialOrd,
    E: PartialOrd,

1.16.0 · Source§### impl<T, U, E> Product<Result<U, E>> for Result<T, E>where
    T: Product<U>,

 

### impl<T, U, E> Product<Result<U, E>> for Result<T, E>where
    T: Product<U>,

Source§#### fn product<I>(iter: I) -> Result<T, E>

 

#### fn product<I>(iter: I) -> Result<T, E>

Source§### impl<T, E> Residual<T> for Result<Infallible, E>

 

### impl<T, E> Residual<T> for Result<Infallible, E>

1.16.0 · Source§### impl<T, U, E> Sum<Result<U, E>> for Result<T, E>where
    T: Sum<U>,

 

### impl<T, U, E> Sum<Result<U, E>> for Result<T, E>where
    T: Sum<U>,

Source§#### fn sum<I>(iter: I) -> Result<T, E>

 

#### fn sum<I>(iter: I) -> Result<T, E>

1.61.0 · Source§### impl<T: Termination, E: Debug> Termination for Result<T, E>

 

### impl<T: Termination, E: Debug> Termination for Result<T, E>

Source§### impl<T, E> Try for Result<T, E>

 

### impl<T, E> Try for Result<T, E>

Source§#### type Output = T

 

#### type Output = T

`try_trait_v2` #84277)`?` when *not*short-circuiting.

Source§#### type Residual = Result<Infallible, E>

 

#### type Residual = Result<Infallible, E>

`try_trait_v2` #84277)`FromResidual::from_residual`
as part of `?` when short-circuiting. Read moreSource§#### fn from_output(output: <Result<T, E> as Try>::Output) -> Result<T, E>

 

#### fn from_output(output: <Result<T, E> as Try>::Output) -> Result<T, E>

`try_trait_v2` #84277)`Output` type. Read moreSource§#### fn branch(
    self,
) -> ControlFlow<<Result<T, E> as Try>::Residual, <Result<T, E> as Try>::Output>

 

#### fn branch( self, ) -> ControlFlow<<Result<T, E> as Try>::Residual, <Result<T, E> as Try>::Output>

`try_trait_v2` #84277)`?` to decide whether the operator should produce a value
(because this returned `ControlFlow::Continue`)
or propagate a value back to the caller
(because this returned `ControlFlow::Break`). Read more

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/result/enum.Result.html
