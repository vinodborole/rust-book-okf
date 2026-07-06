---
type: Web Page
title: Option in std::option - Rust
description: The `Option` type. See the module level documentation for more.
resource: https://doc.rust-lang.org/stable/std/option/enum.Option.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

```
pub enum Option<T> {
    None,
    Some(T),
}
```
## Expand description

The `Option` type. See the module level documentation for more.

## Variants§

## Implementations§

Source§### impl<T> Option<T>

 

### impl<T> Option<T>

1.70.0 (const: unstable) · Source#### pub fn is_some_and(self, f: impl FnOnce(T) -> bool) -> bool

 

#### pub fn is_some_and(self, f: impl FnOnce(T) -> bool) -> bool

Returns `true` if the option is a `Some` and the value inside of it matches a predicate.

##### §Examples

```
let x: Option<u32> = Some(2);
assert_eq!(x.is_some_and(|x| x > 1), true);
let x: Option<u32> = Some(0);
assert_eq!(x.is_some_and(|x| x > 1), false);
let x: Option<u32> = None;
assert_eq!(x.is_some_and(|x| x > 1), false);
let x: Option<String> = Some("ownership".to_string());
assert_eq!(x.as_ref().is_some_and(|x| x.len() > 1), true);
println!("still alive {:?}", x);
```
1.82.0 (const: unstable) · Source#### pub fn is_none_or(self, f: impl FnOnce(T) -> bool) -> bool

 

#### pub fn is_none_or(self, f: impl FnOnce(T) -> bool) -> bool

Returns `true` if the option is a `None` or the value inside of it matches a predicate.

##### §Examples

```
let x: Option<u32> = Some(2);
assert_eq!(x.is_none_or(|x| x > 1), true);
let x: Option<u32> = Some(0);
assert_eq!(x.is_none_or(|x| x > 1), false);
let x: Option<u32> = None;
assert_eq!(x.is_none_or(|x| x > 1), true);
let x: Option<String> = Some("ownership".to_string());
assert_eq!(x.as_ref().is_none_or(|x| x.len() > 1), true);
println!("still alive {:?}", x);
```
1.0.0 (const: 1.48.0) · Source#### pub const fn as_ref(&self) -> Option<&T>

 

#### pub const fn as_ref(&self) -> Option<&T>

Converts from `&Option<T>` to `Option<&T>`.

##### §Examples

Calculates the length of an `Option<String>` as an `Option<usize>`
without moving the `String`. The `map` method takes the `self` argument by value,
consuming the original, so this technique uses `as_ref` to first take an `Option` to a
reference to the value inside the original.

```
let text: Option<String> = Some("Hello, world!".to_string());
// First, cast `Option<String>` to `Option<&String>` with `as_ref`,
// then consume *that* with `map`, leaving `text` on the stack.
let text_length: Option<usize> = text.as_ref().map(|s| s.len());
println!("still can print text: {text:?}");
```
1.0.0 (const: 1.83.0) · Source#### pub const fn as_mut(&mut self) -> Option<&mut T>

 

#### pub const fn as_mut(&mut self) -> Option<&mut T>

Converts from `&mut Option<T>` to `Option<&mut T>`.

##### §Examples

1.33.0 (const: 1.84.0) · Source#### pub const fn as_pin_mut(self: Pin<&mut Option<T>>) -> Option<Pin<&mut T>>

 

#### pub const fn as_pin_mut(self: Pin<&mut Option<T>>) -> Option<Pin<&mut T>>

1.75.0 (const: 1.84.0) · Source#### pub const fn as_slice(&self) -> &[T]

 

#### pub const fn as_slice(&self) -> &[T]

Returns a slice of the contained value, if any. If this is `None`, an
empty slice is returned. This can be useful to have a single type of
iterator over an `Option` or slice.

Note: Should you have an `Option<&T>` and wish to get a slice of `T`,
you can unpack it via `opt.map_or(&[], std::slice::from_ref)`.

##### §Examples

The inverse of this function is (discounting
borrowing) `[_]::first`:

1.75.0 (const: 1.84.0) · Source#### pub const fn as_mut_slice(&mut self) -> &mut [T]

 

#### pub const fn as_mut_slice(&mut self) -> &mut [T]

Returns a mutable slice of the contained value, if any. If this is
`None`, an empty slice is returned. This can be useful to have a
single type of iterator over an `Option` or slice.

Note: Should you have an `Option<&mut T>` instead of a
`&mut Option<T>`, which this method takes, you can obtain a mutable
slice via `opt.map_or(&mut [], std::slice::from_mut)`.

##### §Examples

The result is a mutable slice of zero or one items that points into
our original `Option`:

The inverse of this method (discounting borrowing)
is `[_]::first_mut`:

1.0.0 (const: 1.83.0) · Source#### pub const fn expect(self, msg: &str) -> T

 

#### pub const fn expect(self, msg: &str) -> T

Returns the contained `Some` value, consuming the `self` value.

##### §Panics

Panics if the value is a `None` with a custom panic message provided by
`msg`.

##### §Examples

##### §Recommended Message Style

We recommend that `expect` messages are used to describe the reason you
*expect* the `Option` should be `Some`.

**Hint**: If you’re having trouble remembering how to phrase expect
error messages remember to focus on the word “should” as in “env
variable should be set by blah” or “the given binary should be available
and executable by the current user”.

For more detail on expect message styles and the reasoning behind our
recommendation please refer to the section on “Common Message
Styles” in the `std::error` module docs.

1.0.0 (const: 1.83.0) · Source#### pub const fn unwrap(self) -> T

 

#### pub const fn unwrap(self) -> T

Returns the contained `Some` value, consuming the `self` value.

Because this function may panic, its use is generally discouraged. Panics are meant for unrecoverable errors, and may abort the entire program.

Instead, prefer to use pattern matching and handle the `None`
case explicitly, or call `unwrap_or`, `unwrap_or_else`, or
`unwrap_or_default`. In functions returning `Option`, you can use
the `?` (try) operator.

##### §Panics

Panics if the self value equals `None`.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn unwrap_or(self, default: T) -> T

 

#### pub fn unwrap_or(self, default: T) -> T

Returns the contained `Some` value or a provided default.

Arguments passed to `unwrap_or` are eagerly evaluated; if you are passing
the result of a function call, it is recommended to use `unwrap_or_else`,
which is lazily evaluated.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn unwrap_or_else<F>(self, f: F) -> Twhere
    F: FnOnce() -> T,

 

#### pub fn unwrap_or_else<F>(self, f: F) -> Twhere
    F: FnOnce() -> T,

1.0.0 (const: unstable) · Source#### pub fn unwrap_or_default(self) -> Twhere
    T: Default,

 

#### pub fn unwrap_or_default(self) -> Twhere
    T: Default,

Returns the contained `Some` value or a default.

Consumes the `self` argument then, if `Some`, returns the contained
value, otherwise if `None`, returns the default value for that
type.

##### §Examples

1.58.0 (const: 1.83.0) · Source#### pub const unsafe fn unwrap_unchecked(self) -> T

 

#### pub const unsafe fn unwrap_unchecked(self) -> T

1.0.0 (const: unstable) · Source#### pub fn map<U, F>(self, f: F) -> Option<U>where
    F: FnOnce(T) -> U,

 

#### pub fn map<U, F>(self, f: F) -> Option<U>where
    F: FnOnce(T) -> U,

1.76.0 (const: unstable) · Source#### pub fn inspect<F>(self, f: F) -> Option<T>

 

#### pub fn inspect<F>(self, f: F) -> Option<T>

1.0.0 (const: unstable) · Source#### pub fn map_or<U, F>(self, default: U, f: F) -> Uwhere
    F: FnOnce(T) -> U,

 

#### pub fn map_or<U, F>(self, default: U, f: F) -> Uwhere
    F: FnOnce(T) -> U,

Returns the provided default result (if none), or applies a function to the contained value (if any).

Arguments passed to `map_or` are eagerly evaluated; if you are passing
the result of a function call, it is recommended to use `map_or_else`,
which is lazily evaluated.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn map_or_else<U, D, F>(self, default: D, f: F) -> U

 

#### pub fn map_or_else<U, D, F>(self, default: D, f: F) -> U

Computes a default function result (if none), or applies a different function to the contained value (if any).

##### §Basic examples

```
let k = 21;
let x = Some("foo");
assert_eq!(x.map_or_else(|| 2 * k, |v| v.len()), 3);
let x: Option<&str> = None;
assert_eq!(x.map_or_else(|| 2 * k, |v| v.len()), 42);
```
##### §Handling a Result-based fallback

A somewhat common occurrence when dealing with optional values
in combination with `Result<T, E>` is the case where one wants to invoke
a fallible fallback if the option is not present.  This example
parses a command line argument (if present), or the contents of a file to
an integer.  However, unlike accessing the command line argument, reading
the file is fallible, so it must be wrapped with `Ok`.

Source#### pub const fn map_or_default<U, F>(self, f: F) -> U

 🔬This is a nightly-only experimental API. (`result_option_map_or_default` #138099)

#### pub const fn map_or_default<U, F>(self, f: F) -> U

`result_option_map_or_default` #138099)Maps an `Option<T>` to a `U` by applying function `f` to the contained
value if the option is `Some`, otherwise if `None`, returns the
default value for the type `U`.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn ok_or<E>(self, err: E) -> Result<T, E>

 

#### pub fn ok_or<E>(self, err: E) -> Result<T, E>

Transforms the `Option<T>` into a `Result<T, E>`, mapping `Some(v)` to
`Ok(v)` and `None` to `Err(err)`.

Arguments passed to `ok_or` are eagerly evaluated; if you are passing the
result of a function call, it is recommended to use `ok_or_else`, which is
lazily evaluated.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn ok_or_else<E, F>(self, err: F) -> Result<T, E>where
    F: FnOnce() -> E,

 

#### pub fn ok_or_else<E, F>(self, err: F) -> Result<T, E>where
    F: FnOnce() -> E,

Transforms the `Option<T>` into a `Result<T, E>`, mapping `Some(v)` to
`Ok(v)` and `None` to `Err(err())`.

##### §Examples

1.40.0 (const: unstable) · Source#### pub fn as_deref(&self) -> Option<&<T as Deref>::Target>where
    T: Deref,

 

#### pub fn as_deref(&self) -> Option<&<T as Deref>::Target>where
    T: Deref,

1.40.0 (const: unstable) · Source#### pub fn as_deref_mut(&mut self) -> Option<&mut <T as Deref>::Target>where
    T: DerefMut,

 

#### pub fn as_deref_mut(&mut self) -> Option<&mut <T as Deref>::Target>where
    T: DerefMut,

Converts from `Option<T>` (or `&mut Option<T>`) to `Option<&mut T::Target>`.

Leaves the original `Option` in-place, creating a new one containing a mutable reference to
the inner type’s `Deref::Target` type.

##### §Examples

1.0.0 · Source#### pub fn iter(&self) -> Iter<'_, T> ⓘ

 

#### pub fn iter(&self) -> Iter<'_, T> ⓘ

Returns an iterator over the possibly contained value.

##### §Examples

1.0.0 · Source#### pub fn iter_mut(&mut self) -> IterMut<'_, T> ⓘ

 

#### pub fn iter_mut(&mut self) -> IterMut<'_, T> ⓘ

Returns a mutable iterator over the possibly contained value.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn and<U>(self, optb: Option<U>) -> Option<U>

 

#### pub fn and<U>(self, optb: Option<U>) -> Option<U>

Returns `None` if the option is `None`, otherwise returns `optb`.

Arguments passed to `and` are eagerly evaluated; if you are passing the
result of a function call, it is recommended to use `and_then`, which is
lazily evaluated.

##### §Examples

```
let x = Some(2);
let y: Option<&str> = None;
assert_eq!(x.and(y), None);
let x: Option<u32> = None;
let y = Some("foo");
assert_eq!(x.and(y), None);
let x = Some(2);
let y = Some("foo");
assert_eq!(x.and(y), Some("foo"));
let x: Option<u32> = None;
let y: Option<&str> = None;
assert_eq!(x.and(y), None);
```
1.0.0 (const: unstable) · Source#### pub fn and_then<U, F>(self, f: F) -> Option<U>

 

#### pub fn and_then<U, F>(self, f: F) -> Option<U>

Returns `None` if the option is `None`, otherwise calls `f` with the
wrapped value and returns the result.

Some languages call this operation flatmap.

##### §Examples

```
fn sq_then_to_string(x: u32) -> Option<String> {
    x.checked_mul(x).map(|sq| sq.to_string())
}
assert_eq!(Some(2).and_then(sq_then_to_string), Some(4.to_string()));
assert_eq!(Some(1_000_000).and_then(sq_then_to_string), None); // overflowed!
assert_eq!(None.and_then(sq_then_to_string), None);
```
Often used to chain fallible operations that may return `None`.

1.27.0 (const: unstable) · Source#### pub fn filter<P>(self, predicate: P) -> Option<T>

 

#### pub fn filter<P>(self, predicate: P) -> Option<T>

Returns `None` if the option is `None`, otherwise calls `predicate`
with the wrapped value and returns:

- `Some(t)`if- `predicate`returns- `true`(where- `t`is the wrapped value), and
- `None`if- `predicate`returns- `false`.

This function works similar to `Iterator::filter()`. You can imagine
the `Option<T>` being an iterator over one or zero elements. `filter()`
lets you decide which elements to keep.

##### §Examples

1.0.0 (const: unstable) · Source#### pub fn or(self, optb: Option<T>) -> Option<T>

 

#### pub fn or(self, optb: Option<T>) -> Option<T>

1.0.0 (const: unstable) · Source#### pub fn or_else<F>(self, f: F) -> Option<T>

 

#### pub fn or_else<F>(self, f: F) -> Option<T>

Returns the option if it contains a value, otherwise calls `f` and
returns the result.

##### §Examples

1.37.0 (const: unstable) · Source#### pub fn xor(self, optb: Option<T>) -> Option<T>

 

#### pub fn xor(self, optb: Option<T>) -> Option<T>

1.53.0 (const: unstable) · Source#### pub fn insert(&mut self, value: T) -> &mut T

 

#### pub fn insert(&mut self, value: T) -> &mut T

Inserts `value` into the option, then returns a mutable reference to it.

If the option already contains a value, the old value is dropped.

See also `Option::get_or_insert`, which doesn’t update the value if
the option already contains `Some`.

##### §Example

1.20.0 · Source#### pub fn get_or_insert(&mut self, value: T) -> &mut T

 

#### pub fn get_or_insert(&mut self, value: T) -> &mut T

Inserts `value` into the option if it is `None`, then
returns a mutable reference to the contained value.

See also `Option::insert`, which updates the value even if
the option already contains `Some`.

##### §Examples

1.83.0 (const: unstable) · Source#### pub fn get_or_insert_default(&mut self) -> &mut Twhere
    T: Default,

 

#### pub fn get_or_insert_default(&mut self) -> &mut Twhere
    T: Default,

1.20.0 (const: unstable) · Source#### pub fn get_or_insert_with<F>(&mut self, f: F) -> &mut Twhere
    F: FnOnce() -> T,

 

#### pub fn get_or_insert_with<F>(&mut self, f: F) -> &mut Twhere
    F: FnOnce() -> T,

Source#### pub fn get_or_try_insert_with<'a, R, F>(
    &'a mut self,
    f: F,
) -> <<R as Try>::Residual as Residual<&'a mut T>>::TryType

 🔬This is a nightly-only experimental API. (`option_get_or_try_insert_with` #143648)

#### pub fn get_or_try_insert_with<'a, R, F>( &'a mut self, f: F, ) -> <<R as Try>::Residual as Residual<&'a mut T>>::TryType

`option_get_or_try_insert_with` #143648)If the option is `None`, calls the closure and inserts its output if successful.

If the closure returns a residual value such as `Err` or `None`,
that residual value is returned and nothing is inserted.

If the option is `Some`, nothing is inserted.

Unless a residual is returned, a mutable reference to the value of the option will be output.

##### §Examples

```
#![feature(option_get_or_try_insert_with)]
let mut o1: Option<u32> = None;
let mut o2: Option<u8> = None;
let number = "12345";
assert_eq!(o1.get_or_try_insert_with(|| number.parse()).copied(), Ok(12345));
assert!(o2.get_or_try_insert_with(|| number.parse()).is_err());
assert_eq!(o1, Some(12345));
assert_eq!(o2, None);
```
1.80.0 (const: unstable) · Source#### pub fn take_if<P>(&mut self, predicate: P) -> Option<T>

 

#### pub fn take_if<P>(&mut self, predicate: P) -> Option<T>

Takes the value out of the option, but only if the predicate evaluates to
`true` on a mutable reference to the value.

In other words, replaces `self` with `None` if the predicate returns `true`.
This method operates similar to `Option::take` but conditional.

##### §Examples

1.31.0 (const: 1.83.0) · Source#### pub const fn replace(&mut self, value: T) -> Option<T>

 

#### pub const fn replace(&mut self, value: T) -> Option<T>

1.46.0 (const: unstable) · Source#### pub fn zip<U>(self, other: Option<U>) -> Option<(T, U)>

 

#### pub fn zip<U>(self, other: Option<U>) -> Option<(T, U)>

Zips `self` with another `Option`.

If `self` is `Some(s)` and `other` is `Some(o)`, this method returns `Some((s, o))`.
Otherwise, `None` is returned.

##### §Examples

Source#### pub const fn zip_with<U, F, R>(self, other: Option<U>, f: F) -> Option<R>where
    F: FnOnce(T, U) -> R,

 🔬This is a nightly-only experimental API. (`option_zip` #70086)

#### pub const fn zip_with<U, F, R>(self, other: Option<U>, f: F) -> Option<R>where
    F: FnOnce(T, U) -> R,

`option_zip` #70086)Zips `self` and another `Option` with function `f`.

If `self` is `Some(s)` and `other` is `Some(o)`, this method returns `Some(f(s, o))`.
Otherwise, `None` is returned.

##### §Examples

```
#![feature(option_zip)]
#[derive(Debug, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}
impl Point {
    fn new(x: f64, y: f64) -> Self {
        Self { x, y }
    }
}
let x = Some(17.5);
let y = Some(42.7);
assert_eq!(x.zip_with(y, Point::new), Some(Point { x: 17.5, y: 42.7 }));
assert_eq!(x.zip_with(None, Point::new), None);
```
Source#### pub fn reduce<U, R, F>(self, other: Option<U>, f: F) -> Option<R>

 🔬This is a nightly-only experimental API. (`option_reduce` #144273)

#### pub fn reduce<U, R, F>(self, other: Option<U>, f: F) -> Option<R>

`option_reduce` #144273)Reduces two options into one, using the provided function if both are `Some`.

If `self` is `Some(s)` and `other` is `Some(o)`, this method returns `Some(f(s, o))`.
Otherwise, if only one of `self` and `other` is `Some`, that one is returned.
If both `self` and `other` are `None`, `None` is returned.

##### §Examples

Source§### impl<T> Option<T>where
    T: IntoIterator,

 

### impl<T> Option<T>where
    T: IntoIterator,

Source#### pub fn into_flat_iter<A>(self) -> OptionFlatten<A> ⓘwhere
    T: IntoIterator<IntoIter = A>,

 🔬This is a nightly-only experimental API. (`option_into_flat_iter` #148441)

#### pub fn into_flat_iter<A>(self) -> OptionFlatten<A> ⓘwhere
    T: IntoIterator<IntoIter = A>,

`option_into_flat_iter` #148441)Transforms an optional iterator into an iterator.

If `self` is `None`, the resulting iterator is empty.
Otherwise, an iterator is made from the `Some` value and returned.

##### §Examples

Source§### impl<T, U> Option<(T, U)>

 

### impl<T, U> Option<(T, U)>

Source§### impl<T> Option<&T>

 

### impl<T> Option<&T>

Source§### impl<T> Option<&mut T>

 

### impl<T> Option<&mut T>

Source§### impl<T, E> Option<Result<T, E>>

 

### impl<T, E> Option<Result<T, E>>

Source§### impl<T> Option<Option<T>>

 

### impl<T> Option<Option<T>>

1.40.0 (const: 1.83.0) · Source#### pub const fn flatten(self) -> Option<T>

 

#### pub const fn flatten(self) -> Option<T>

Converts from `Option<Option<T>>` to `Option<T>`.

##### §Examples

Basic usage:

```
let x: Option<Option<u32>> = Some(Some(6));
assert_eq!(Some(6), x.flatten());
let x: Option<Option<u32>> = Some(None);
assert_eq!(None, x.flatten());
let x: Option<Option<u32>> = None;
assert_eq!(None, x.flatten());
```
Flattening only removes one level of nesting at a time:

Source§### impl<'a, T> Option<&'a Option<T>>

 

### impl<'a, T> Option<&'a Option<T>>

Source§### impl<'a, T> Option<&'a mut Option<T>>

 

### impl<'a, T> Option<&'a mut Option<T>>

Source#### pub const fn flatten_ref(self) -> Option<&'a T>

 🔬This is a nightly-only experimental API. (`option_reference_flattening` #149221)

#### pub const fn flatten_ref(self) -> Option<&'a T>

`option_reference_flattening` #149221)Converts from `Option<&mut Option<T>>` to `&Option<T>`.

##### §Examples

Basic usage:

```
#![feature(option_reference_flattening)]
let y = &mut Some(6);
let x: Option<&mut Option<u32>> = Some(y);
assert_eq!(Some(&6), x.flatten_ref());
let y: &mut Option<u32> = &mut None;
let x: Option<&mut Option<u32>> = Some(y);
assert_eq!(None, x.flatten_ref());
let x: Option<&mut Option<u32>> = None;
assert_eq!(None, x.flatten_ref());
```
Source#### pub const fn flatten_mut(self) -> Option<&'a mut T>

 🔬This is a nightly-only experimental API. (`option_reference_flattening` #149221)

#### pub const fn flatten_mut(self) -> Option<&'a mut T>

`option_reference_flattening` #149221)Converts from `Option<&mut Option<T>>` to `Option<&mut T>`.

##### §Examples

Basic usage:

```
#![feature(option_reference_flattening)]
let y: &mut Option<u32> = &mut Some(6);
let x: Option<&mut Option<u32>> = Some(y);
assert_eq!(Some(&mut 6), x.flatten_mut());
let y: &mut Option<u32> = &mut None;
let x: Option<&mut Option<u32>> = Some(y);
assert_eq!(None, x.flatten_mut());
let x: Option<&mut Option<u32>> = None;
assert_eq!(None, x.flatten_mut());
```
## Trait Implementations§

1.30.0 (const: unstable) · Source§### impl<'a, T> From<&'a Option<T>> for Option<&'a T>

 

### impl<'a, T> From<&'a Option<T>> for Option<&'a T>

Source§#### fn from(o: &'a Option<T>) -> Option<&'a T>

 

#### fn from(o: &'a Option<T>) -> Option<&'a T>

1.30.0 (const: unstable) · Source§### impl<'a, T> From<&'a mut Option<T>> for Option<&'a mut T>

 

### impl<'a, T> From<&'a mut Option<T>> for Option<&'a mut T>

1.0.0 · Source§### impl<A, V> FromIterator<Option<A>> for Option<V>where
    V: FromIterator<A>,

 

### impl<A, V> FromIterator<Option<A>> for Option<V>where
    V: FromIterator<A>,

Source§#### fn from_iter<I>(iter: I) -> Option<V>where
    I: IntoIterator<Item = Option<A>>,

 

#### fn from_iter<I>(iter: I) -> Option<V>where
    I: IntoIterator<Item = Option<A>>,

Takes each element in the `Iterator`: if it is `None`,
no further elements are taken, and the `None` is
returned. Should no `None` occur, a container of type
`V` containing the values of each `Option` is returned.

##### §Examples

Here is an example which increments every integer in a vector.
We use the checked variant of `add` that returns `None` when the
calculation would result in an overflow.

```
let items = vec![0_u16, 1, 2];
let res: Option<Vec<u16>> = items
    .iter()
    .map(|x| x.checked_add(1))
    .collect();
assert_eq!(res, Some(vec![1, 2, 3]));
```
As you can see, this will return the expected, valid items.

Here is another example that tries to subtract one from another list of integers, this time checking for underflow:

```
let items = vec![2_u16, 1, 0];
let res: Option<Vec<u16>> = items
    .iter()
    .map(|x| x.checked_sub(1))
    .collect();
assert_eq!(res, None);
```
Since the last element is zero, it would underflow. Thus, the resulting
value is `None`.

Here is a variation on the previous example, showing that no
further elements are taken from `iter` after the first `None`.

```
let items = vec![3_u16, 2, 1, 10];
let mut shared = 0;
let res: Option<Vec<u16>> = items
    .iter()
    .map(|x| { shared += x; x.checked_sub(2) })
    .collect();
assert_eq!(res, None);
assert_eq!(shared, 6);
```
Since the third element caused an underflow, no further elements were taken,
so the final value of `shared` is 6 (= `3 + 2 + 1`), not 16.

Source§### impl<T> FromResidual<Option<Infallible>> for Option<T>

 

### impl<T> FromResidual<Option<Infallible>> for Option<T>

Source§#### fn from_residual(residual: Option<Infallible>) -> Option<T>

 

#### fn from_residual(residual: Option<Infallible>) -> Option<T>

`try_trait_v2` #84277)`Residual` type. Read more1.4.0 · Source§### impl<'a, T> IntoIterator for &'a Option<T>

 

### impl<'a, T> IntoIterator for &'a Option<T>

1.4.0 · Source§### impl<'a, T> IntoIterator for &'a mut Option<T>

 

### impl<'a, T> IntoIterator for &'a mut Option<T>

1.0.0 (const: unstable) · Source§### impl<T> IntoIterator for Option<T>

 

### impl<T> IntoIterator for Option<T>

1.0.0 (const: unstable) · Source§### impl<T> Ord for Option<T>where
    T: Ord,

 

### impl<T> Ord for Option<T>where
    T: Ord,

1.0.0 (const: unstable) · Source§### impl<T> PartialOrd for Option<T>where
    T: PartialOrd,

 

### impl<T> PartialOrd for Option<T>where
    T: PartialOrd,

1.37.0 · Source§### impl<T, U> Product<Option<U>> for Option<T>where
    T: Product<U>,

 

### impl<T, U> Product<Option<U>> for Option<T>where
    T: Product<U>,

Source§#### fn product<I>(iter: I) -> Option<T>

 

#### fn product<I>(iter: I) -> Option<T>

Source§### impl<T> Residual<T> for Option<Infallible>

 

### impl<T> Residual<T> for Option<Infallible>

1.37.0 · Source§### impl<T, U> Sum<Option<U>> for Option<T>where
    T: Sum<U>,

 

### impl<T, U> Sum<Option<U>> for Option<T>where
    T: Sum<U>,

Source§#### fn sum<I>(iter: I) -> Option<T>

 

#### fn sum<I>(iter: I) -> Option<T>

Takes each element in the `Iterator`: if it is a `None`, no further
elements are taken, and the `None` is returned. Should no `None`
occur, the sum of all elements is returned.

##### §Examples

This sums up the position of the character ‘a’ in a vector of strings,
if a word did not have the character ‘a’ the operation returns `None`:

Source§### impl<T> Try for Option<T>

 

### impl<T> Try for Option<T>

Source§#### type Output = T

 

#### type Output = T

`try_trait_v2` #84277)`?` when *not*short-circuiting.

Source§#### type Residual = Option<Infallible>

 

#### type Residual = Option<Infallible>

`try_trait_v2` #84277)`FromResidual::from_residual`
as part of `?` when short-circuiting. Read moreSource§#### fn from_output(output: <Option<T> as Try>::Output) -> Option<T>

 

#### fn from_output(output: <Option<T> as Try>::Output) -> Option<T>

`try_trait_v2` #84277)`Output` type. Read moreSource§#### fn branch(
    self,
) -> ControlFlow<<Option<T> as Try>::Residual, <Option<T> as Try>::Output>

 

#### fn branch( self, ) -> ControlFlow<<Option<T> as Try>::Residual, <Option<T> as Try>::Output>

`try_trait_v2` #84277)`?` to decide whether the operator should produce a value
(because this returned `ControlFlow::Continue`)
or propagate a value back to the caller
(because this returned `ControlFlow::Break`). Read more

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/option/enum.Option.html
