---
type: Web Page
title: String in std::string - Rust
description: A UTF-8–encoded, growable string.
resource: https://doc.rust-lang.org/stable/std/string/struct.String.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

`pub struct String { /* private fields */ }`## Expand description

A UTF-8–encoded, growable string.

`String` is the most common string type. It has ownership over the contents
of the string, stored in a heap-allocated buffer (see Representation).
It is closely related to its borrowed counterpart, the primitive `str`.

## §Examples

You can create a `String` from a literal string with `String::from`:

You can append a `char` to a `String` with the `push` method, and
append a `&str` with the `push_str` method:

If you have a vector of UTF-8 bytes, you can create a `String` from it with
the `from_utf8` method:

```
// some bytes, in a vector
let sparkle_heart = vec![240, 159, 146, 150];
// We know these bytes are valid, so we'll use `unwrap()`.
let sparkle_heart = String::from_utf8(sparkle_heart).unwrap();
assert_eq!("💖", sparkle_heart);
```
## §UTF-8

`String`s are always valid UTF-8. If you need a non-UTF-8 string, consider
`OsString`. It is similar, but without the UTF-8 constraint. Because UTF-8
is a variable width encoding, `String`s are typically smaller than an array of
the same `char`s:

```
// `s` is ASCII which represents each `char` as one byte
let s = "hello";
assert_eq!(s.len(), 5);
// A `char` array with the same contents would be longer because
// every `char` is four bytes
let s = ['h', 'e', 'l', 'l', 'o'];
let size: usize = s.into_iter().map(|c| size_of_val(&c)).sum();
assert_eq!(size, 20);
// However, for non-ASCII strings, the difference will be smaller
// and sometimes they are the same
let s = "💖💖💖💖💖";
assert_eq!(s.len(), 20);
let s = ['💖', '💖', '💖', '💖', '💖'];
let size: usize = s.into_iter().map(|c| size_of_val(&c)).sum();
assert_eq!(size, 20);
```
This raises interesting questions as to how `s[i]` should work.
What should `i` be here? Several options include byte indices and
`char` indices but, because of UTF-8 encoding, only byte indices
would provide constant time indexing. Getting the `i`th `char`, for
example, is available using `chars`:

```
let s = "hello";
let third_character = s.chars().nth(2);
assert_eq!(third_character, Some('l'));
let s = "💖💖💖💖💖";
let third_character = s.chars().nth(2);
assert_eq!(third_character, Some('💖'));
```
Next, what should `s[i]` return? Because indexing returns a reference
to underlying data it could be `&u8`, `&[u8]`, or something similar.
Since we’re only providing one index, `&u8` makes the most sense but that
might not be what the user expects and can be explicitly achieved with
`as_bytes()`:

```
// The first byte is 104 - the byte value of `'h'`
let s = "hello";
assert_eq!(s.as_bytes()[0], 104);
// or
assert_eq!(s.as_bytes()[0], b'h');
// The first byte is 240 which isn't obviously useful
let s = "💖💖💖💖💖";
assert_eq!(s.as_bytes()[0], 240);
```
Due to these ambiguities/restrictions, indexing with a `usize` is simply
forbidden:

It is more clear, however, how `&s[i..j]` should work (that is,
indexing with a range). It should accept byte indices (to be constant-time)
and return a `&str` which is UTF-8 encoded. This is also called “string slicing”.
Note this will panic if the byte indices provided are not character
boundaries - see `is_char_boundary` for more details. See the implementations
for `SliceIndex<str>` for more details on string slicing. For a non-panicking
version of string slicing, see `get`.

The `bytes` and `chars` methods return iterators over the bytes and
codepoints of the string, respectively. To iterate over codepoints along
with byte indices, use `char_indices`.

## §Deref

`String` implements `Deref<Target = str>`, and so inherits all of `str`’s
methods. In addition, this means that you can pass a `String` to a
function which takes a `&str` by using an ampersand (`&`):

This will create a `&str` from the `String` and pass it in. This
conversion is very inexpensive, and so generally, functions will accept
`&str`s as arguments unless they need a `String` for some specific
reason.

In certain cases Rust doesn’t have enough information to make this
conversion, known as `Deref` coercion. In the following example a string
slice `&'a str` implements the trait `TraitExample`, and the function
`example_func` takes anything that implements the trait. In this case Rust
would need to make two implicit conversions, which Rust doesn’t have the
means to do. For that reason, the following example will not compile.

```
trait TraitExample {}
impl<'a> TraitExample for &'a str {}
fn example_func<A: TraitExample>(example_arg: A) {}
let example_string = String::from("example_string");
example_func(&example_string);
```
There are two options that would work instead. The first would be to
change the line `example_func(&example_string);` to
`example_func(example_string.as_str());`, using the method `as_str()`
to explicitly extract the string slice containing the string. The second
way changes `example_func(&example_string);` to
`example_func(&*example_string);`. In this case we are dereferencing a
`String` to a `str`, then referencing the `str` back to
`&str`. The second way is more idiomatic, however both work to do the
conversion explicitly rather than relying on the implicit conversion.

## §Representation

A `String` is made up of three components: a pointer to some bytes, a
length, and a capacity. The pointer points to the internal buffer which `String`
uses to store its data. The length is the number of bytes currently stored
in the buffer, and the capacity is the size of the buffer in bytes. As such,
the length will always be less than or equal to the capacity.

This buffer is always stored on the heap.

You can look at these with the `as_ptr`, `len`, and `capacity`
methods:

```
let story = String::from("Once upon a time...");
// Deconstruct the String into parts.
let (ptr, len, capacity) = story.into_raw_parts();
// story has nineteen bytes
assert_eq!(19, len);
// We can re-build a String out of ptr, len, and capacity. This is all
// unsafe because we are responsible for making sure the components are
// valid:
let s = unsafe { String::from_raw_parts(ptr, len, capacity) } ;
assert_eq!(String::from("Once upon a time..."), s);
```
If a `String` has enough capacity, adding elements to it will not
re-allocate. For example, consider this program:

```
let mut s = String::new();
println!("{}", s.capacity());
for _ in 0..5 {
    s.push_str("hello");
    println!("{}", s.capacity());
}
```
This will output the following:

```
0
8
16
16
32
32
```
At first, we have no memory allocated at all, but as we append to the
string, it increases its capacity appropriately. If we instead use the
`with_capacity` method to allocate the correct capacity initially:

```
let mut s = String::with_capacity(25);
println!("{}", s.capacity());
for _ in 0..5 {
    s.push_str("hello");
    println!("{}", s.capacity());
}
```
We end up with a different output:

```
25
25
25
25
25
25
```
Here, there’s no need to allocate more memory inside the loop.

## Implementations§

Source§### impl String

 

### impl String

1.0.0 (const: 1.39.0) · Source#### pub const fn new() -> String

 

#### pub const fn new() -> String

Creates a new empty `String`.

Given that the `String` is empty, this will not allocate any initial
buffer. While that means that this initial operation is very
inexpensive, it may cause excessive allocation later when you add
data. If you have an idea of how much data the `String` will hold,
consider the `with_capacity` method to prevent excessive
re-allocation.

##### §Examples

1.0.0 · Source#### pub fn with_capacity(capacity: usize) -> String

 

#### pub fn with_capacity(capacity: usize) -> String

Creates a new empty `String` with at least the specified capacity.

`String`s have an internal buffer to hold their data. The capacity is
the length of that buffer, and can be queried with the `capacity`
method. This method creates an empty `String`, but one with an initial
buffer that can hold at least `capacity` bytes. This is useful when you
may be appending a bunch of data to the `String`, reducing the number of
reallocations it needs to do.

If the given capacity is `0`, no allocation will occur, and this method
is identical to the `new` method.

##### §Panics

Panics if the capacity exceeds `isize::MAX` *bytes*.

##### §Examples

```
let mut s = String::with_capacity(10);
// The String contains no chars, even though it has capacity for more
assert_eq!(s.len(), 0);
// These are all done without reallocating...
let cap = s.capacity();
for _ in 0..10 {
    s.push('a');
}
assert_eq!(s.capacity(), cap);
// ...but this may make the string reallocate
s.push('a');
```
Source#### pub fn try_with_capacity(capacity: usize) -> Result<String, TryReserveError>

 🔬This is a nightly-only experimental API. (`try_with_capacity` #91913)

#### pub fn try_with_capacity(capacity: usize) -> Result<String, TryReserveError>

`try_with_capacity` #91913)1.0.0 · Source#### pub fn from_utf8(vec: Vec<u8>) -> Result<String, FromUtf8Error>

 

#### pub fn from_utf8(vec: Vec<u8>) -> Result<String, FromUtf8Error>

Converts a vector of bytes to a `String`.

A string (`String`) is made of bytes (`u8`), and a vector of bytes
(`Vec<u8>`) is made of bytes, so this function converts between the
two. Not all byte slices are valid `String`s, however: `String`
requires that it is valid UTF-8. `from_utf8()` checks to ensure that
the bytes are valid UTF-8, and then does the conversion.

If you are sure that the byte slice is valid UTF-8, and you don’t want
to incur the overhead of the validity check, there is an unsafe version
of this function, `from_utf8_unchecked`, which has the same behavior
but skips the check.

This method will take care to not copy the vector, for efficiency’s sake.

If you need a `&str` instead of a `String`, consider
`str::from_utf8`.

The inverse of this method is `into_bytes`.

##### §Errors

Returns `Err` if the slice is not UTF-8 with a description as to why the
provided bytes are not UTF-8. The vector you moved in is also included.

##### §Examples

Basic usage:

```
// some bytes, in a vector
let sparkle_heart = vec![240, 159, 146, 150];
// We know these bytes are valid, so we'll use `unwrap()`.
let sparkle_heart = String::from_utf8(sparkle_heart).unwrap();
assert_eq!("💖", sparkle_heart);
```
Incorrect bytes:

```
// some invalid bytes, in a vector
let sparkle_heart = vec![0, 159, 146, 150];
assert!(String::from_utf8(sparkle_heart).is_err());
```
See the docs for `FromUtf8Error` for more details on what you can do
with this error.

1.0.0 · Source#### pub fn from_utf8_lossy(v: &[u8]) -> Cow<'_, str>

 

#### pub fn from_utf8_lossy(v: &[u8]) -> Cow<'_, str>

Converts a slice of bytes to a string, including invalid characters.

Strings are made of bytes (`u8`), and a slice of bytes
(`&[u8]`) is made of bytes, so this function converts
between the two. Not all byte slices are valid strings, however: strings
are required to be valid UTF-8. During this conversion,
`from_utf8_lossy()` will replace any invalid UTF-8 sequences with
`U+FFFD REPLACEMENT CHARACTER`, which looks like this: �

If you are sure that the byte slice is valid UTF-8, and you don’t want
to incur the overhead of the conversion, there is an unsafe version
of this function, `from_utf8_unchecked`, which has the same behavior
but skips the checks.

This function returns a `Cow<'a, str>`. If our byte slice is invalid
UTF-8, then we need to insert the replacement characters, which will
change the size of the string, and hence, require a `String`. But if
it’s already valid UTF-8, we don’t need a new allocation. This return
type allows us to handle both cases.

##### §Examples

Basic usage:

```
// some bytes, in a vector
let sparkle_heart = vec![240, 159, 146, 150];
let sparkle_heart = String::from_utf8_lossy(&sparkle_heart);
assert_eq!("💖", sparkle_heart);
```
Incorrect bytes:

Source#### pub fn from_utf8_lossy_owned(v: Vec<u8>) -> String

 🔬This is a nightly-only experimental API. (`string_from_utf8_lossy_owned` #129436)

#### pub fn from_utf8_lossy_owned(v: Vec<u8>) -> String

`string_from_utf8_lossy_owned` #129436)Converts a `Vec<u8>` to a `String`, substituting invalid UTF-8
sequences with replacement characters.

See `from_utf8_lossy` for more details.

Note that this function does not guarantee reuse of the original `Vec`
allocation.

##### §Examples

Basic usage:

```
#![feature(string_from_utf8_lossy_owned)]
// some bytes, in a vector
let sparkle_heart = vec![240, 159, 146, 150];
let sparkle_heart = String::from_utf8_lossy_owned(sparkle_heart);
assert_eq!(String::from("💖"), sparkle_heart);
```
Incorrect bytes:

1.0.0 · Source#### pub fn from_utf16(v: &[u16]) -> Result<String, FromUtf16Error>

 

#### pub fn from_utf16(v: &[u16]) -> Result<String, FromUtf16Error>

1.0.0 · Source#### pub fn from_utf16_lossy(v: &[u16]) -> String

 

#### pub fn from_utf16_lossy(v: &[u16]) -> String

Decode a native endian UTF-16–encoded slice `v` into a `String`,
replacing invalid data with the replacement character (`U+FFFD`).

Unlike `from_utf8_lossy` which returns a `Cow<'a, str>`,
`from_utf16_lossy` returns a `String` since the UTF-16 to UTF-8
conversion requires a memory allocation.

##### §Examples

Source#### pub fn from_utf16le(v: &[u8]) -> Result<String, FromUtf16Error>

 🔬This is a nightly-only experimental API. (`str_from_utf16_endian` #116258)

#### pub fn from_utf16le(v: &[u8]) -> Result<String, FromUtf16Error>

`str_from_utf16_endian` #116258)Decode a UTF-16LE–encoded vector `v` into a `String`,
returning `Err` if `v` contains any invalid data.

##### §Examples

Basic usage:

```
#![feature(str_from_utf16_endian)]
// 𝄞music
let v = &[0x34, 0xD8, 0x1E, 0xDD, 0x6d, 0x00, 0x75, 0x00,
          0x73, 0x00, 0x69, 0x00, 0x63, 0x00];
assert_eq!(String::from("𝄞music"),
           String::from_utf16le(v).unwrap());
// 𝄞mu<invalid>ic
let v = &[0x34, 0xD8, 0x1E, 0xDD, 0x6d, 0x00, 0x75, 0x00,
          0x00, 0xD8, 0x69, 0x00, 0x63, 0x00];
assert!(String::from_utf16le(v).is_err());
```
Source#### pub fn from_utf16le_lossy(v: &[u8]) -> String

 🔬This is a nightly-only experimental API. (`str_from_utf16_endian` #116258)

#### pub fn from_utf16le_lossy(v: &[u8]) -> String

`str_from_utf16_endian` #116258)Decode a UTF-16LE–encoded slice `v` into a `String`, replacing
invalid data with the replacement character (`U+FFFD`).

Unlike `from_utf8_lossy` which returns a `Cow<'a, str>`,
`from_utf16le_lossy` returns a `String` since the UTF-16 to UTF-8
conversion requires a memory allocation.

##### §Examples

Basic usage:

Source#### pub fn from_utf16be(v: &[u8]) -> Result<String, FromUtf16Error>

 🔬This is a nightly-only experimental API. (`str_from_utf16_endian` #116258)

#### pub fn from_utf16be(v: &[u8]) -> Result<String, FromUtf16Error>

`str_from_utf16_endian` #116258)Decode a UTF-16BE–encoded vector `v` into a `String`,
returning `Err` if `v` contains any invalid data.

##### §Examples

Basic usage:

```
#![feature(str_from_utf16_endian)]
// 𝄞music
let v = &[0xD8, 0x34, 0xDD, 0x1E, 0x00, 0x6d, 0x00, 0x75,
          0x00, 0x73, 0x00, 0x69, 0x00, 0x63];
assert_eq!(String::from("𝄞music"),
           String::from_utf16be(v).unwrap());
// 𝄞mu<invalid>ic
let v = &[0xD8, 0x34, 0xDD, 0x1E, 0x00, 0x6d, 0x00, 0x75,
          0xD8, 0x00, 0x00, 0x69, 0x00, 0x63];
assert!(String::from_utf16be(v).is_err());
```
Source#### pub fn from_utf16be_lossy(v: &[u8]) -> String

 🔬This is a nightly-only experimental API. (`str_from_utf16_endian` #116258)

#### pub fn from_utf16be_lossy(v: &[u8]) -> String

`str_from_utf16_endian` #116258)Decode a UTF-16BE–encoded slice `v` into a `String`, replacing
invalid data with the replacement character (`U+FFFD`).

Unlike `from_utf8_lossy` which returns a `Cow<'a, str>`,
`from_utf16le_lossy` returns a `String` since the UTF-16 to UTF-8
conversion requires a memory allocation.

##### §Examples

Basic usage:

1.93.0 · Source#### pub fn into_raw_parts(self) -> (*mut u8, usize, usize)

 

#### pub fn into_raw_parts(self) -> (*mut u8, usize, usize)

Decomposes a `String` into its raw components: `(pointer, length, capacity)`.

Returns the raw pointer to the underlying data, the length of
the string (in bytes), and the allocated capacity of the data
(in bytes). These are the same arguments in the same order as
the arguments to `from_raw_parts`.

After calling this function, the caller is responsible for the
memory previously managed by the `String`. The only way to do
this is to convert the raw pointer, length, and capacity back
into a `String` with the `from_raw_parts` function, allowing
the destructor to perform the cleanup.

##### §Examples

1.0.0 · Source#### pub unsafe fn from_raw_parts(
    buf: *mut u8,
    length: usize,
    capacity: usize,
) -> String

 

#### pub unsafe fn from_raw_parts( buf: *mut u8, length: usize, capacity: usize, ) -> String

Creates a new `String` from a pointer, a length and a capacity.

##### §Safety

This is highly unsafe, due to the number of invariants that aren’t checked:

- all safety requirements for `Vec::<u8>::from_raw_parts`.
- all safety requirements for `String::from_utf8_unchecked`.

Violating these may cause problems like corrupting the allocator’s
internal data structures. For example, it is normally **not** safe to
build a `String` from a pointer to a C `char` array containing UTF-8
*unless* you are certain that array was originally allocated by the
Rust standard library’s allocator.

The ownership of `buf` is effectively transferred to the
`String` which may then deallocate, reallocate or change the
contents of memory pointed to by the pointer at will. Ensure
that nothing else uses the pointer after calling this
function.

##### §Examples

1.0.0 · Source#### pub unsafe fn from_utf8_unchecked(bytes: Vec<u8>) -> String

 

#### pub unsafe fn from_utf8_unchecked(bytes: Vec<u8>) -> String

Converts a vector of bytes to a `String` without checking that the
string contains valid UTF-8.

See the safe version, `from_utf8`, for more details.

##### §Safety

This function is unsafe because it does not check that the bytes passed
to it are valid UTF-8. If this constraint is violated, it may cause
memory unsafety issues with future users of the `String`, as the rest of
the standard library assumes that `String`s are valid UTF-8.

##### §Examples

1.0.0 (const: 1.87.0) · Source#### pub const fn into_bytes(self) -> Vec<u8> ⓘ

 

#### pub const fn into_bytes(self) -> Vec<u8> ⓘ

Converts a `String` into a byte vector.

This consumes the `String`, so we do not need to copy its contents.

##### §Examples

1.7.0 (const: 1.87.0) · Source#### pub const fn as_str(&self) -> &str

 

#### pub const fn as_str(&self) -> &str

Extracts a string slice containing the entire `String`.

##### §Examples

1.7.0 (const: 1.87.0) · Source#### pub const fn as_mut_str(&mut self) -> &mut str

 

#### pub const fn as_mut_str(&mut self) -> &mut str

Converts a `String` into a mutable string slice.

##### §Examples

1.87.0 · Source#### pub fn extend_from_within<R>(&mut self, src: R)where
    R: RangeBounds<usize>,

 

#### pub fn extend_from_within<R>(&mut self, src: R)where
    R: RangeBounds<usize>,

1.0.0 (const: 1.87.0) · Source#### pub const fn capacity(&self) -> usize

 

#### pub const fn capacity(&self) -> usize

Returns this `String`’s capacity, in bytes.

##### §Examples

1.0.0 · Source#### pub fn reserve(&mut self, additional: usize)

 

#### pub fn reserve(&mut self, additional: usize)

Reserves capacity for at least `additional` bytes more than the
current length. The allocator may reserve more space to speculatively
avoid frequent allocations. After calling `reserve`,
capacity will be greater than or equal to `self.len() + additional`.
Does nothing if capacity is already sufficient.

##### §Panics

Panics if the new capacity exceeds `isize::MAX` *bytes*.

##### §Examples

Basic usage:

This might not actually increase the capacity:

```
let mut s = String::with_capacity(10);
s.push('a');
s.push('b');
// s now has a length of 2 and a capacity of at least 10
let capacity = s.capacity();
assert_eq!(2, s.len());
assert!(capacity >= 10);
// Since we already have at least an extra 8 capacity, calling this...
s.reserve(8);
// ... doesn't actually increase.
assert_eq!(capacity, s.capacity());
```
1.0.0 · Source#### pub fn reserve_exact(&mut self, additional: usize)

 

#### pub fn reserve_exact(&mut self, additional: usize)

Reserves the minimum capacity for at least `additional` bytes more than
the current length. Unlike `reserve`, this will not
deliberately over-allocate to speculatively avoid frequent allocations.
After calling `reserve_exact`, capacity will be greater than or equal to
`self.len() + additional`. Does nothing if the capacity is already
sufficient.

##### §Panics

Panics if the new capacity exceeds `isize::MAX` *bytes*.

##### §Examples

Basic usage:

This might not actually increase the capacity:

```
let mut s = String::with_capacity(10);
s.push('a');
s.push('b');
// s now has a length of 2 and a capacity of at least 10
let capacity = s.capacity();
assert_eq!(2, s.len());
assert!(capacity >= 10);
// Since we already have at least an extra 8 capacity, calling this...
s.reserve_exact(8);
// ... doesn't actually increase.
assert_eq!(capacity, s.capacity());
```
1.57.0 · Source#### pub fn try_reserve(&mut self, additional: usize) -> Result<(), TryReserveError>

 

#### pub fn try_reserve(&mut self, additional: usize) -> Result<(), TryReserveError>

Tries to reserve capacity for at least `additional` bytes more than the
current length. The allocator may reserve more space to speculatively
avoid frequent allocations. After calling `try_reserve`, capacity will be
greater than or equal to `self.len() + additional` if it returns
`Ok(())`. Does nothing if capacity is already sufficient. This method
preserves the contents even if an error occurs.

##### §Errors

If the capacity overflows, or the allocator reports a failure, then an error is returned.

##### §Examples

```
use std::collections::TryReserveError;
fn process_data(data: &str) -> Result<String, TryReserveError> {
    let mut output = String::new();
    // Pre-reserve the memory, exiting if we can't
    output.try_reserve(data.len())?;
    // Now we know this can't OOM in the middle of our complex work
    output.push_str(data);
    Ok(output)
}
```
1.57.0 · Source#### pub fn try_reserve_exact(
    &mut self,
    additional: usize,
) -> Result<(), TryReserveError>

 

#### pub fn try_reserve_exact( &mut self, additional: usize, ) -> Result<(), TryReserveError>

Tries to reserve the minimum capacity for at least `additional` bytes
more than the current length. Unlike `try_reserve`, this will not
deliberately over-allocate to speculatively avoid frequent allocations.
After calling `try_reserve_exact`, capacity will be greater than or
equal to `self.len() + additional` if it returns `Ok(())`.
Does nothing if the capacity is already sufficient.

Note that the allocator may give the collection more space than it
requests. Therefore, capacity can not be relied upon to be precisely
minimal. Prefer `try_reserve` if future insertions are expected.

##### §Errors

If the capacity overflows, or the allocator reports a failure, then an error is returned.

##### §Examples

```
use std::collections::TryReserveError;
fn process_data(data: &str) -> Result<String, TryReserveError> {
    let mut output = String::new();
    // Pre-reserve the memory, exiting if we can't
    output.try_reserve_exact(data.len())?;
    // Now we know this can't OOM in the middle of our complex work
    output.push_str(data);
    Ok(output)
}
```
1.0.0 · Source#### pub fn shrink_to_fit(&mut self)

 

#### pub fn shrink_to_fit(&mut self)

Shrinks the capacity of this `String` to match its length.

##### §Examples

1.56.0 · Source#### pub fn shrink_to(&mut self, min_capacity: usize)

 

#### pub fn shrink_to(&mut self, min_capacity: usize)

Shrinks the capacity of this `String` with a lower bound.

The capacity will remain at least as large as both the length and the supplied value.

If the current capacity is less than the lower limit, this is a no-op.

##### §Examples

1.0.0 · Source#### pub fn truncate(&mut self, new_len: usize)

 

#### pub fn truncate(&mut self, new_len: usize)

1.0.0 · Source#### pub fn remove(&mut self, idx: usize) -> char

 

#### pub fn remove(&mut self, idx: usize) -> char

Removes a `char` from this `String` at byte position `idx` and returns it.

Copies all bytes after the removed char to new positions.

Note that calling this in a loop can result in quadratic behavior.

##### §Panics

Panics if `idx` is larger than or equal to the `String`’s length,
or if it does not lie on a `char` boundary.

##### §Examples

Source#### pub fn remove_matches<P>(&mut self, pat: P)where
    P: Pattern,

 🔬This is a nightly-only experimental API. (`string_remove_matches` #72826)

#### pub fn remove_matches<P>(&mut self, pat: P)where
    P: Pattern,

`string_remove_matches` #72826)Remove all matches of pattern `pat` in the `String`.

##### §Examples

```
#![feature(string_remove_matches)]
let mut s = String::from("Trees are not green, the sky is not blue.");
s.remove_matches("not ");
assert_eq!("Trees are green, the sky is blue.", s);
```
Matches will be detected and removed iteratively, so in cases where patterns overlap, only the first pattern will be removed:

1.26.0 · Source#### pub fn retain<F>(&mut self, f: F)

 

#### pub fn retain<F>(&mut self, f: F)

Retains only the characters specified by the predicate.

In other words, remove all characters `c` such that `f(c)` returns `false`.
This method operates in place, visiting each character exactly once in the
original order, and preserves the order of the retained characters.

##### §Examples

Because the elements are visited exactly once in the original order, external state may be used to decide which elements to keep.

1.0.0 · Source#### pub fn insert(&mut self, idx: usize, ch: char)

 

#### pub fn insert(&mut self, idx: usize, ch: char)

Inserts a character into this `String` at byte position `idx`.

Reallocates if `self.capacity()` is insufficient, which may involve copying all
`self.capacity()` bytes. Makes space for the insertion by copying all bytes of
`&self[idx..]` to new positions.

Note that calling this in a loop can result in quadratic behavior.

##### §Panics

Panics if `idx` is larger than the `String`’s length, or if it does not
lie on a `char` boundary.

##### §Examples

1.16.0 · Source#### pub fn insert_str(&mut self, idx: usize, string: &str)

 

#### pub fn insert_str(&mut self, idx: usize, string: &str)

Inserts a string slice into this `String` at byte position `idx`.

Reallocates if `self.capacity()` is insufficient, which may involve copying all
`self.capacity()` bytes. Makes space for the insertion by copying all bytes of
`&self[idx..]` to new positions.

Note that calling this in a loop can result in quadratic behavior.

##### §Panics

Panics if `idx` is larger than the `String`’s length, or if it does not
lie on a `char` boundary.

##### §Examples

1.0.0 (const: 1.87.0) · Source#### pub const unsafe fn as_mut_vec(&mut self) -> &mut Vec<u8> ⓘ

 

#### pub const unsafe fn as_mut_vec(&mut self) -> &mut Vec<u8> ⓘ

Returns a mutable reference to the contents of this `String`.

##### §Safety

This function is unsafe because the returned `&mut Vec` allows writing
bytes which are not valid UTF-8. If this constraint is violated, using
the original `String` after dropping the `&mut Vec` may violate memory
safety, as the rest of the standard library assumes that `String`s are
valid UTF-8.

##### §Examples

1.0.0 (const: 1.87.0) · Source#### pub const fn len(&self) -> usize

 

#### pub const fn len(&self) -> usize

1.0.0 (const: 1.87.0) · Source#### pub const fn is_empty(&self) -> bool

 

#### pub const fn is_empty(&self) -> bool

Returns `true` if this `String` has a length of zero, and `false` otherwise.

##### §Examples

1.16.0 · Source#### pub fn split_off(&mut self, at: usize) -> String

 

#### pub fn split_off(&mut self, at: usize) -> String

Splits the string into two at the given byte index.

Returns a newly allocated `String`. `self` contains bytes `[0, at)`, and
the returned `String` contains bytes `[at, len)`. `at` must be on the
boundary of a UTF-8 code point.

Note that the capacity of `self` does not change.

##### §Panics

Panics if `at` is not on a `UTF-8` code point boundary, or if it is beyond the last
code point of the string.

##### §Examples

1.0.0 · Source#### pub fn clear(&mut self)

 

#### pub fn clear(&mut self)

Truncates this `String`, removing all contents.

While this means the `String` will have a length of zero, it does not
touch its capacity.

##### §Examples

1.6.0 · Source#### pub fn drain<R>(&mut self, range: R) -> Drain<'_> ⓘwhere
    R: RangeBounds<usize>,

 

#### pub fn drain<R>(&mut self, range: R) -> Drain<'_> ⓘwhere
    R: RangeBounds<usize>,

Removes the specified range from the string in bulk, returning all removed characters as an iterator.

The returned iterator keeps a mutable borrow on the string to optimize its implementation.

##### §Panics

Panics if the range has `start_bound > end_bound`, or, if the range is
bounded on either end and does not lie on a `char` boundary.

##### §Leaking

If the returned iterator goes out of scope without being dropped (due to
`core::mem::forget`, for example), the string may still contain a copy
of any drained characters, or may have lost characters arbitrarily,
including characters outside the range.

##### §Examples

```
let mut s = String::from("α is alpha, β is beta");
let beta_offset = s.find('β').unwrap_or(s.len());
// Remove the range up until the β from the string
let t: String = s.drain(..beta_offset).collect();
assert_eq!(t, "α is alpha, ");
assert_eq!(s, "β is beta");
// A full range clears the string, like `clear()` does
s.drain(..);
assert_eq!(s, "");
```
Source#### pub fn into_chars(self) -> IntoChars ⓘ

 🔬This is a nightly-only experimental API. (`string_into_chars` #133125)

#### pub fn into_chars(self) -> IntoChars ⓘ

`string_into_chars` #133125)Converts a `String` into an iterator over the `char`s of the string.

As a string consists of valid UTF-8, we can iterate through a string
by `char`. This method returns such an iterator.

It’s important to remember that `char` represents a Unicode Scalar
Value, and might not match your idea of what a ‘character’ is. Iteration
over grapheme clusters may be what you actually want. That functionality
is not provided by Rust’s standard library, check crates.io instead.

##### §Examples

Basic usage:

```
#![feature(string_into_chars)]
let word = String::from("goodbye");
let mut chars = word.into_chars();
assert_eq!(Some('g'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('d'), chars.next());
assert_eq!(Some('b'), chars.next());
assert_eq!(Some('y'), chars.next());
assert_eq!(Some('e'), chars.next());
assert_eq!(None, chars.next());
```
Remember, `char`s might not match your intuition about characters:

1.27.0 · Source#### pub fn replace_range<R>(&mut self, range: R, replace_with: &str)where
    R: RangeBounds<usize>,

 

#### pub fn replace_range<R>(&mut self, range: R, replace_with: &str)where
    R: RangeBounds<usize>,

Source#### pub fn replace_first<P>(&mut self, from: P, to: &str)where
    P: Pattern,

 🔬This is a nightly-only experimental API. (`string_replace_in_place` #147949)

#### pub fn replace_first<P>(&mut self, from: P, to: &str)where
    P: Pattern,

`string_replace_in_place` #147949)Replaces the leftmost occurrence of a pattern with another string, in-place.

This method can be preferred over `string = string.replacen(..., 1);`,
as it can use the `String`’s existing capacity to prevent a reallocation if
sufficient space is available.

##### §Examples

Basic usage:

Source#### pub fn replace_last<P>(&mut self, from: P, to: &str)

 🔬This is a nightly-only experimental API. (`string_replace_in_place` #147949)

#### pub fn replace_last<P>(&mut self, from: P, to: &str)

`string_replace_in_place` #147949)Replaces the rightmost occurrence of a pattern with another string, in-place.

##### §Examples

Basic usage:

1.4.0 · Source#### pub fn into_boxed_str(self) -> Box<str>

 

#### pub fn into_boxed_str(self) -> Box<str>

Converts this `String` into a `Box<str>`.

Before doing the conversion, this method discards excess capacity like `shrink_to_fit`.
Note that this call may reallocate and copy the bytes of the string.

##### §Examples

1.72.0 · Source#### pub fn leak<'a>(self) -> &'a mut str

 

#### pub fn leak<'a>(self) -> &'a mut str

Consumes and leaks the `String`, returning a mutable reference to the contents,
`&'a mut str`.

The caller has free choice over the returned lifetime, including `'static`. Indeed,
this function is ideally used for data that lives for the remainder of the program’s life,
as dropping the returned reference will cause a memory leak.

It does not reallocate or shrink the `String`, so the leaked allocation may include unused
capacity that is not part of the returned slice. If you want to discard excess capacity,
call `into_boxed_str`, and then `Box::leak` instead. However, keep in mind that
trimming the capacity may result in a reallocation and copy.

##### §Examples

## Methods from Deref<Target = str>§

1.0.0 · Source#### pub fn is_empty(&self) -> bool

 

#### pub fn is_empty(&self) -> bool

Returns `true` if `self` has a length of zero bytes.

##### §Examples

1.9.0 · Source#### pub fn is_char_boundary(&self, index: usize) -> bool

 

#### pub fn is_char_boundary(&self, index: usize) -> bool

Checks that `index`-th byte is the first byte in a UTF-8 code point
sequence or the end of the string.

The start and end of the string (when `index == self.len()`) are
considered to be boundaries.

Returns `false` if `index` is greater than `self.len()`.

##### §Examples

1.91.0 · Source#### pub fn floor_char_boundary(&self, index: usize) -> usize

 

#### pub fn floor_char_boundary(&self, index: usize) -> usize

Finds the closest `x` not exceeding `index` where `is_char_boundary(x)` is `true`.

This method can help you truncate a string so that it’s still valid UTF-8, but doesn’t exceed a given number of bytes. Note that this is done purely at the character level and can still visually split graphemes, even though the underlying characters aren’t split. For example, the emoji 🧑🔬 (scientist) could be split so that the string only includes 🧑 (person) instead.

##### §Examples

1.91.0 · Source#### pub fn ceil_char_boundary(&self, index: usize) -> usize

 

#### pub fn ceil_char_boundary(&self, index: usize) -> usize

Finds the closest `x` not below `index` where `is_char_boundary(x)` is `true`.

If `index` is greater than the length of the string, this returns the length of the string.

This method is the natural complement to `floor_char_boundary`. See that method
for more details.

##### §Examples

1.20.0 · Source#### pub unsafe fn as_bytes_mut(&mut self) -> &mut [u8] ⓘ

 

#### pub unsafe fn as_bytes_mut(&mut self) -> &mut [u8] ⓘ

Converts a mutable string slice to a mutable byte slice.

##### §Safety

The caller must ensure that the content of the slice is valid UTF-8
before the borrow ends and the underlying `str` is used.

Use of a `str` whose contents are not valid UTF-8 is undefined behavior.

##### §Examples

Basic usage:

```
let mut s = String::from("Hello");
let bytes = unsafe { s.as_bytes_mut() };
assert_eq!(b"Hello", bytes);
```
Mutability:

1.0.0 · Source#### pub fn as_ptr(&self) -> *const u8

 

#### pub fn as_ptr(&self) -> *const u8

Converts a string slice to a raw pointer.

As string slices are a slice of bytes, the raw pointer points to a
`u8`. This pointer will be pointing to the first byte of the string
slice.

The caller must ensure that the returned pointer is never written to.
If you need to mutate the contents of the string slice, use `as_mut_ptr`.

##### §Examples

1.36.0 · Source#### pub fn as_mut_ptr(&mut self) -> *mut u8

 

#### pub fn as_mut_ptr(&mut self) -> *mut u8

Converts a mutable string slice to a raw pointer.

As string slices are a slice of bytes, the raw pointer points to a
`u8`. This pointer will be pointing to the first byte of the string
slice.

It is your responsibility to make sure that the string slice only gets modified in a way that it remains valid UTF-8.

1.20.0 · Source#### pub fn get<I>(&self, i: I) -> Option<&<I as SliceIndex<str>>::Output>where
    I: SliceIndex<str>,

 

#### pub fn get<I>(&self, i: I) -> Option<&<I as SliceIndex<str>>::Output>where
    I: SliceIndex<str>,

1.20.0 · Source#### pub fn get_mut<I>(
    &mut self,
    i: I,
) -> Option<&mut <I as SliceIndex<str>>::Output>where
    I: SliceIndex<str>,

 

#### pub fn get_mut<I>(
    &mut self,
    i: I,
) -> Option<&mut <I as SliceIndex<str>>::Output>where
    I: SliceIndex<str>,

Returns a mutable subslice of `str`.

This is the non-panicking alternative to indexing the `str`. Returns
`None` whenever equivalent indexing operation would panic.

##### §Examples

```
let mut v = String::from("hello");
// correct length
assert!(v.get_mut(0..5).is_some());
// out of bounds
assert!(v.get_mut(..42).is_none());
assert_eq!(Some("he"), v.get_mut(0..2).map(|v| &*v));
assert_eq!("hello", v);
{
    let s = v.get_mut(0..2);
    let s = s.map(|s| {
        s.make_ascii_uppercase();
        &*s
    });
    assert_eq!(Some("HE"), s);
}
assert_eq!("HEllo", v);
```
1.20.0 · Source#### pub unsafe fn get_unchecked<I>(&self, i: I) -> &<I as SliceIndex<str>>::Outputwhere
    I: SliceIndex<str>,

 

#### pub unsafe fn get_unchecked<I>(&self, i: I) -> &<I as SliceIndex<str>>::Outputwhere
    I: SliceIndex<str>,

Returns an unchecked subslice of `str`.

This is the unchecked alternative to indexing the `str`.

##### §Safety

Callers of this function are responsible that these preconditions are satisfied:

- The starting index must not exceed the ending index;
- Indexes must be within bounds of the original slice;
- Indexes must lie on UTF-8 sequence boundaries.

Failing that, the returned string slice may reference invalid memory or
violate the invariants communicated by the `str` type.

##### §Examples

1.20.0 · Source#### pub unsafe fn get_unchecked_mut<I>(
    &mut self,
    i: I,
) -> &mut <I as SliceIndex<str>>::Outputwhere
    I: SliceIndex<str>,

 

#### pub unsafe fn get_unchecked_mut<I>(
    &mut self,
    i: I,
) -> &mut <I as SliceIndex<str>>::Outputwhere
    I: SliceIndex<str>,

Returns a mutable, unchecked subslice of `str`.

This is the unchecked alternative to indexing the `str`.

##### §Safety

Callers of this function are responsible that these preconditions are satisfied:

- The starting index must not exceed the ending index;
- Indexes must be within bounds of the original slice;
- Indexes must lie on UTF-8 sequence boundaries.

Failing that, the returned string slice may reference invalid memory or
violate the invariants communicated by the `str` type.

##### §Examples

1.0.0 · Source#### pub unsafe fn slice_unchecked(&self, begin: usize, end: usize) -> &str

 👎Deprecated since 1.29.0: use `get_unchecked(begin..end)` instead

#### pub unsafe fn slice_unchecked(&self, begin: usize, end: usize) -> &str

use `get_unchecked(begin..end)` instead

Creates a string slice from another string slice, bypassing safety checks.

This is generally not recommended, use with caution! For a safe
alternative see `str` and `Index`.

This new slice goes from `begin` to `end`, including `begin` but
excluding `end`.

To get a mutable string slice instead, see the
`slice_mut_unchecked` method.

##### §Safety

Callers of this function are responsible that three preconditions are satisfied:

- `begin`must not exceed- `end`.
- `begin`and- `end`must be byte positions within the string slice.
- `begin`and- `end`must lie on UTF-8 sequence boundaries.

##### §Examples

1.5.0 · Source#### pub unsafe fn slice_mut_unchecked(
    &mut self,
    begin: usize,
    end: usize,
) -> &mut str

 👎Deprecated since 1.29.0: use `get_unchecked_mut(begin..end)` instead

#### pub unsafe fn slice_mut_unchecked( &mut self, begin: usize, end: usize, ) -> &mut str

use `get_unchecked_mut(begin..end)` instead

Creates a string slice from another string slice, bypassing safety checks.

This is generally not recommended, use with caution! For a safe
alternative see `str` and `IndexMut`.

This new slice goes from `begin` to `end`, including `begin` but
excluding `end`.

To get an immutable string slice instead, see the
`slice_unchecked` method.

##### §Safety

Callers of this function are responsible that three preconditions are satisfied:

- `begin`must not exceed- `end`.
- `begin`and- `end`must be byte positions within the string slice.
- `begin`and- `end`must lie on UTF-8 sequence boundaries.

1.4.0 · Source#### pub fn split_at(&self, mid: usize) -> (&str, &str)

 

#### pub fn split_at(&self, mid: usize) -> (&str, &str)

Divides one string slice into two at an index.

The argument, `mid`, should be a byte offset from the start of the
string. It must also be on the boundary of a UTF-8 code point.

The two slices returned go from the start of the string slice to `mid`,
and from `mid` to the end of the string slice.

To get mutable string slices instead, see the `split_at_mut`
method.

##### §Panics

Panics if `mid` is not on a UTF-8 code point boundary, or if it is past
the end of the last code point of the string slice.  For a non-panicking
alternative see `split_at_checked`.

##### §Examples

1.4.0 · Source#### pub fn split_at_mut(&mut self, mid: usize) -> (&mut str, &mut str)

 

#### pub fn split_at_mut(&mut self, mid: usize) -> (&mut str, &mut str)

Divides one mutable string slice into two at an index.

The argument, `mid`, should be a byte offset from the start of the
string. It must also be on the boundary of a UTF-8 code point.

The two slices returned go from the start of the string slice to `mid`,
and from `mid` to the end of the string slice.

To get immutable string slices instead, see the `split_at` method.

##### §Panics

Panics if `mid` is not on a UTF-8 code point boundary, or if it is past
the end of the last code point of the string slice.  For a non-panicking
alternative see `split_at_mut_checked`.

##### §Examples

1.80.0 · Source#### pub fn split_at_checked(&self, mid: usize) -> Option<(&str, &str)>

 

#### pub fn split_at_checked(&self, mid: usize) -> Option<(&str, &str)>

Divides one string slice into two at an index.

The argument, `mid`, should be a valid byte offset from the start of the
string. It must also be on the boundary of a UTF-8 code point. The
method returns `None` if that’s not the case.

The two slices returned go from the start of the string slice to `mid`,
and from `mid` to the end of the string slice.

To get mutable string slices instead, see the `split_at_mut_checked`
method.

##### §Examples

1.80.0 · Source#### pub fn split_at_mut_checked(
    &mut self,
    mid: usize,
) -> Option<(&mut str, &mut str)>

 

#### pub fn split_at_mut_checked( &mut self, mid: usize, ) -> Option<(&mut str, &mut str)>

Divides one mutable string slice into two at an index.

The argument, `mid`, should be a valid byte offset from the start of the
string. It must also be on the boundary of a UTF-8 code point. The
method returns `None` if that’s not the case.

The two slices returned go from the start of the string slice to `mid`,
and from `mid` to the end of the string slice.

To get immutable string slices instead, see the `split_at_checked` method.

##### §Examples

```
let mut s = "Per Martin-Löf".to_string();
if let Some((first, last)) = s.split_at_mut_checked(3) {
    first.make_ascii_uppercase();
    assert_eq!("PER", first);
    assert_eq!(" Martin-Löf", last);
}
assert_eq!("PER Martin-Löf", s);
assert_eq!(None, s.split_at_mut_checked(13));  // Inside “ö”
assert_eq!(None, s.split_at_mut_checked(16));  // Beyond the string length
```
1.0.0 · Source#### pub fn chars(&self) -> Chars<'_> ⓘ

 

#### pub fn chars(&self) -> Chars<'_> ⓘ

Returns an iterator over the `char`s of a string slice.

As a string slice consists of valid UTF-8, we can iterate through a
string slice by `char`. This method returns such an iterator.

It’s important to remember that `char` represents a Unicode Scalar
Value, and might not match your idea of what a ‘character’ is. Iteration
over grapheme clusters may be what you actually want. This functionality
is not provided by Rust’s standard library, check crates.io instead.

##### §Examples

Basic usage:

```
let word = "goodbye";
let count = word.chars().count();
assert_eq!(7, count);
let mut chars = word.chars();
assert_eq!(Some('g'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('o'), chars.next());
assert_eq!(Some('d'), chars.next());
assert_eq!(Some('b'), chars.next());
assert_eq!(Some('y'), chars.next());
assert_eq!(Some('e'), chars.next());
assert_eq!(None, chars.next());
```
Remember, `char`s might not match your intuition about characters:

1.0.0 · Source#### pub fn char_indices(&self) -> CharIndices<'_> ⓘ

 

#### pub fn char_indices(&self) -> CharIndices<'_> ⓘ

Returns an iterator over the `char`s of a string slice, and their
positions.

As a string slice consists of valid UTF-8, we can iterate through a
string slice by `char`. This method returns an iterator of both
these `char`s, as well as their byte positions.

The iterator yields tuples. The position is first, the `char` is
second.

##### §Examples

Basic usage:

```
let word = "goodbye";
let count = word.char_indices().count();
assert_eq!(7, count);
let mut char_indices = word.char_indices();
assert_eq!(Some((0, 'g')), char_indices.next());
assert_eq!(Some((1, 'o')), char_indices.next());
assert_eq!(Some((2, 'o')), char_indices.next());
assert_eq!(Some((3, 'd')), char_indices.next());
assert_eq!(Some((4, 'b')), char_indices.next());
assert_eq!(Some((5, 'y')), char_indices.next());
assert_eq!(Some((6, 'e')), char_indices.next());
assert_eq!(None, char_indices.next());
```
Remember, `char`s might not match your intuition about characters:

```
let yes = "y̆es";
let mut char_indices = yes.char_indices();
assert_eq!(Some((0, 'y')), char_indices.next()); // not (0, 'y̆')
assert_eq!(Some((1, '\u{0306}')), char_indices.next());
// note the 3 here - the previous character took up two bytes
assert_eq!(Some((3, 'e')), char_indices.next());
assert_eq!(Some((4, 's')), char_indices.next());
assert_eq!(None, char_indices.next());
```
1.0.0 · Source#### pub fn bytes(&self) -> Bytes<'_> ⓘ

 

#### pub fn bytes(&self) -> Bytes<'_> ⓘ

Returns an iterator over the bytes of a string slice.

As a string slice consists of a sequence of bytes, we can iterate through a string slice by byte. This method returns such an iterator.

##### §Examples

1.1.0 · Source#### pub fn split_whitespace(&self) -> SplitWhitespace<'_> ⓘ

 

#### pub fn split_whitespace(&self) -> SplitWhitespace<'_> ⓘ

Splits a string slice by whitespace.

The iterator returned will return string slices that are sub-slices of the original string slice, separated by any amount of whitespace.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`. If you only want to split on ASCII whitespace
instead, use `split_ascii_whitespace`.

##### §Examples

Basic usage:

```
let mut iter = "A few words".split_whitespace();
assert_eq!(Some("A"), iter.next());
assert_eq!(Some("few"), iter.next());
assert_eq!(Some("words"), iter.next());
assert_eq!(None, iter.next());
```
All kinds of whitespace are considered:

```
let mut iter = " Mary   had\ta\u{2009}little  \n\t lamb".split_whitespace();
assert_eq!(Some("Mary"), iter.next());
assert_eq!(Some("had"), iter.next());
assert_eq!(Some("a"), iter.next());
assert_eq!(Some("little"), iter.next());
assert_eq!(Some("lamb"), iter.next());
assert_eq!(None, iter.next());
```
If the string is empty or all whitespace, the iterator yields no string slices:

1.34.0 · Source#### pub fn split_ascii_whitespace(&self) -> SplitAsciiWhitespace<'_> ⓘ

 

#### pub fn split_ascii_whitespace(&self) -> SplitAsciiWhitespace<'_> ⓘ

Splits a string slice by ASCII whitespace.

The iterator returned will return string slices that are sub-slices of the original string slice, separated by any amount of ASCII whitespace.

This uses the same definition as `char::is_ascii_whitespace`.
To split by Unicode `Whitespace` instead, use `split_whitespace`.

##### §Examples

Basic usage:

```
let mut iter = "A few words".split_ascii_whitespace();
assert_eq!(Some("A"), iter.next());
assert_eq!(Some("few"), iter.next());
assert_eq!(Some("words"), iter.next());
assert_eq!(None, iter.next());
```
Various kinds of ASCII whitespace are considered
(see `char::is_ascii_whitespace`):

```
let mut iter = " Mary   had\ta little  \n\t lamb".split_ascii_whitespace();
assert_eq!(Some("Mary"), iter.next());
assert_eq!(Some("had"), iter.next());
assert_eq!(Some("a"), iter.next());
assert_eq!(Some("little"), iter.next());
assert_eq!(Some("lamb"), iter.next());
assert_eq!(None, iter.next());
```
If the string is empty or all ASCII whitespace, the iterator yields no string slices:

1.0.0 · Source#### pub fn lines(&self) -> Lines<'_> ⓘ

 

#### pub fn lines(&self) -> Lines<'_> ⓘ

Returns an iterator over the lines of a string, as string slices.

Lines are split at line endings that are either newlines (`\n`) or
sequences of a carriage return followed by a line feed (`\r\n`).

Line terminators are not included in the lines returned by the iterator.

Note that any carriage return (`\r`) not immediately followed by a
line feed (`\n`) does not split a line. These carriage returns are
thereby included in the produced lines.

The final line ending is optional. A string that ends with a final line ending will return the same lines as an otherwise identical string without a final line ending.

An empty string returns an empty iterator.

##### §Examples

Basic usage:

```
let text = "foo\r\nbar\n\nbaz\r";
let mut lines = text.lines();
assert_eq!(Some("foo"), lines.next());
assert_eq!(Some("bar"), lines.next());
assert_eq!(Some(""), lines.next());
// Trailing carriage return is included in the last line
assert_eq!(Some("baz\r"), lines.next());
assert_eq!(None, lines.next());
```
The final line does not require any ending:

```
let text = "foo\nbar\n\r\nbaz";
let mut lines = text.lines();
assert_eq!(Some("foo"), lines.next());
assert_eq!(Some("bar"), lines.next());
assert_eq!(Some(""), lines.next());
assert_eq!(Some("baz"), lines.next());
assert_eq!(None, lines.next());
```
An empty string returns an empty iterator:

1.0.0 · Source#### pub fn lines_any(&self) -> LinesAny<'_> ⓘ

 👎Deprecated since 1.4.0: use lines() instead now

#### pub fn lines_any(&self) -> LinesAny<'_> ⓘ

use lines() instead now

Returns an iterator over the lines of a string.

1.8.0 · Source#### pub fn encode_utf16(&self) -> EncodeUtf16<'_> ⓘ

 

#### pub fn encode_utf16(&self) -> EncodeUtf16<'_> ⓘ

Returns an iterator of `u16` over the string encoded
as native endian UTF-16 (without byte-order mark).

##### §Examples

1.0.0 · Source#### pub fn contains<P>(&self, pat: P) -> boolwhere
    P: Pattern,

 

#### pub fn contains<P>(&self, pat: P) -> boolwhere
    P: Pattern,

1.0.0 · Source#### pub fn starts_with<P>(&self, pat: P) -> boolwhere
    P: Pattern,

 

#### pub fn starts_with<P>(&self, pat: P) -> boolwhere
    P: Pattern,

Returns `true` if the given pattern matches a prefix of this
string slice.

Returns `false` if it does not.

The pattern can be a `&str`, in which case this function will return true if
the `&str` is a prefix of this string slice.

The pattern can also be a `char`, a slice of `char`s, or a
function or closure that determines if a character matches.
These will only be checked against the first character of this string slice.
Look at the second example below regarding behavior for slices of `char`s.

##### §Examples

1.0.0 · Source#### pub fn ends_with<P>(&self, pat: P) -> bool

 

#### pub fn ends_with<P>(&self, pat: P) -> bool

1.0.0 · Source#### pub fn find<P>(&self, pat: P) -> Option<usize>where
    P: Pattern,

 

#### pub fn find<P>(&self, pat: P) -> Option<usize>where
    P: Pattern,

Returns the byte index of the first character of this string slice that matches the pattern.

Returns `None` if the pattern doesn’t match.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

Simple patterns:

```
let s = "Löwe 老虎 Léopard Gepardi";
assert_eq!(s.find('L'), Some(0));
assert_eq!(s.find('é'), Some(14));
assert_eq!(s.find("pard"), Some(17));
```
More complex patterns using point-free style and closures:

```
let s = "Löwe 老虎 Léopard";
assert_eq!(s.find(char::is_whitespace), Some(5));
assert_eq!(s.find(char::is_lowercase), Some(1));
assert_eq!(s.find(|c: char| c.is_whitespace() || c.is_lowercase()), Some(1));
assert_eq!(s.find(|c: char| (c < 'o') && (c > 'a')), Some(4));
```
Not finding the pattern:

1.0.0 · Source#### pub fn rfind<P>(&self, pat: P) -> Option<usize>

 

#### pub fn rfind<P>(&self, pat: P) -> Option<usize>

Returns the byte index for the first character of the last match of the pattern in this string slice.

Returns `None` if the pattern doesn’t match.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

Simple patterns:

```
let s = "Löwe 老虎 Léopard Gepardi";
assert_eq!(s.rfind('L'), Some(13));
assert_eq!(s.rfind('é'), Some(14));
assert_eq!(s.rfind("pard"), Some(24));
```
More complex patterns with closures:

```
let s = "Löwe 老虎 Léopard";
assert_eq!(s.rfind(char::is_whitespace), Some(12));
assert_eq!(s.rfind(char::is_lowercase), Some(20));
```
Not finding the pattern:

1.0.0 · Source#### pub fn split<P>(&self, pat: P) -> Split<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn split<P>(&self, pat: P) -> Split<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over substrings of this string slice, separated by characters matched by a pattern.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

If there are no matches the full string slice is returned as the only item in the iterator.

##### §Iterator behavior

The returned iterator will be a `DoubleEndedIterator` if the pattern
allows a reverse search and forward/reverse search yields the same
elements. This is true for, e.g., `char`, but not for `&str`.

If the pattern allows a reverse search but its results might differ
from a forward search, the `rsplit` method can be used.

##### §Examples

Simple patterns:

```
let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
assert_eq!(v, ["Mary", "had", "a", "little", "lamb"]);
let v: Vec<&str> = "".split('X').collect();
assert_eq!(v, [""]);
let v: Vec<&str> = "lionXXtigerXleopard".split('X').collect();
assert_eq!(v, ["lion", "", "tiger", "leopard"]);
let v: Vec<&str> = "lion::tiger::leopard".split("::").collect();
assert_eq!(v, ["lion", "tiger", "leopard"]);
let v: Vec<&str> = "AABBCC".split("DD").collect();
assert_eq!(v, ["AABBCC"]);
let v: Vec<&str> = "abc1def2ghi".split(char::is_numeric).collect();
assert_eq!(v, ["abc", "def", "ghi"]);
let v: Vec<&str> = "lionXtigerXleopard".split(char::is_uppercase).collect();
assert_eq!(v, ["lion", "tiger", "leopard"]);
```
If the pattern is a slice of chars, split on each occurrence of any of the characters:

```
let v: Vec<&str> = "2020-11-03 23:59".split(&['-', ' ', ':', '@'][..]).collect();
assert_eq!(v, ["2020", "11", "03", "23", "59"]);
```
A more complex pattern, using a closure:

```
let v: Vec<&str> = "abc1defXghi".split(|c| c == '1' || c == 'X').collect();
assert_eq!(v, ["abc", "def", "ghi"]);
```
If a string contains multiple contiguous separators, you will end up with empty strings in the output:

```
let x = "||||a||b|c".to_string();
let d: Vec<_> = x.split('|').collect();
assert_eq!(d, &["", "", "", "", "a", "", "b", "c"]);
```
Contiguous separators are separated by the empty string.

```
let x = "(///)".to_string();
let d: Vec<_> = x.split('/').collect();
assert_eq!(d, &["(", "", "", ")"]);
```
Separators at the start or end of a string are neighbored by empty strings.

When the empty string is used as a separator, it separates every character in the string, along with the beginning and end of the string.

Contiguous separators can lead to possibly surprising behavior when whitespace is used as the separator. This code is correct:

```
let x = "    a  b c".to_string();
let d: Vec<_> = x.split(' ').collect();
assert_eq!(d, &["", "", "", "", "a", "", "b", "c"]);
```
It does *not* give you:

Use `split_whitespace` for this behavior.

1.51.0 · Source#### pub fn split_inclusive<P>(&self, pat: P) -> SplitInclusive<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn split_inclusive<P>(&self, pat: P) -> SplitInclusive<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over substrings of this string slice, separated by characters matched by a pattern.

Differs from the iterator produced by `split` in that `split_inclusive`
leaves the matched part as the terminator of the substring.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

```
let v: Vec<&str> = "Mary had a little lamb\nlittle lamb\nlittle lamb."
    .split_inclusive('\n').collect();
assert_eq!(v, ["Mary had a little lamb\n", "little lamb\n", "little lamb."]);
```
If the last element of the string is matched, that element will be considered the terminator of the preceding substring. That substring will be the last item returned by the iterator.

1.0.0 · Source#### pub fn rsplit<P>(&self, pat: P) -> RSplit<'_, P> ⓘ

 

#### pub fn rsplit<P>(&self, pat: P) -> RSplit<'_, P> ⓘ

Returns an iterator over substrings of the given string slice, separated by characters matched by a pattern and yielded in reverse order.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator requires that the pattern supports a reverse
search, and it will be a `DoubleEndedIterator` if a forward/reverse
search yields the same elements.

For iterating from the front, the `split` method can be used.

##### §Examples

Simple patterns:

```
let v: Vec<&str> = "Mary had a little lamb".rsplit(' ').collect();
assert_eq!(v, ["lamb", "little", "a", "had", "Mary"]);
let v: Vec<&str> = "".rsplit('X').collect();
assert_eq!(v, [""]);
let v: Vec<&str> = "lionXXtigerXleopard".rsplit('X').collect();
assert_eq!(v, ["leopard", "tiger", "", "lion"]);
let v: Vec<&str> = "lion::tiger::leopard".rsplit("::").collect();
assert_eq!(v, ["leopard", "tiger", "lion"]);
```
A more complex pattern, using a closure:

1.0.0 · Source#### pub fn split_terminator<P>(&self, pat: P) -> SplitTerminator<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn split_terminator<P>(&self, pat: P) -> SplitTerminator<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over substrings of the given string slice, separated by characters matched by a pattern.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

Equivalent to `split`, except that the trailing substring
is skipped if empty.

This method can be used for string data that is *terminated*,
rather than *separated* by a pattern.

##### §Iterator behavior

The returned iterator will be a `DoubleEndedIterator` if the pattern
allows a reverse search and forward/reverse search yields the same
elements. This is true for, e.g., `char`, but not for `&str`.

If the pattern allows a reverse search but its results might differ
from a forward search, the `rsplit_terminator` method can be used.

##### §Examples

1.0.0 · Source#### pub fn rsplit_terminator<P>(&self, pat: P) -> RSplitTerminator<'_, P> ⓘ

 

#### pub fn rsplit_terminator<P>(&self, pat: P) -> RSplitTerminator<'_, P> ⓘ

Returns an iterator over substrings of `self`, separated by characters
matched by a pattern and yielded in reverse order.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

Equivalent to `split`, except that the trailing substring is
skipped if empty.

This method can be used for string data that is *terminated*,
rather than *separated* by a pattern.

##### §Iterator behavior

The returned iterator requires that the pattern supports a reverse search, and it will be double ended if a forward/reverse search yields the same elements.

For iterating from the front, the `split_terminator` method can be
used.

##### §Examples

1.0.0 · Source#### pub fn splitn<P>(&self, n: usize, pat: P) -> SplitN<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn splitn<P>(&self, n: usize, pat: P) -> SplitN<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over substrings of the given string slice, separated
by a pattern, restricted to returning at most `n` items.

If `n` substrings are returned, the last substring (the `n`th substring)
will contain the remainder of the string.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator will not be double ended, because it is not efficient to support.

If the pattern allows a reverse search, the `rsplitn` method can be
used.

##### §Examples

Simple patterns:

```
let v: Vec<&str> = "Mary had a little lambda".splitn(3, ' ').collect();
assert_eq!(v, ["Mary", "had", "a little lambda"]);
let v: Vec<&str> = "lionXXtigerXleopard".splitn(3, "X").collect();
assert_eq!(v, ["lion", "", "tigerXleopard"]);
let v: Vec<&str> = "abcXdef".splitn(1, 'X').collect();
assert_eq!(v, ["abcXdef"]);
let v: Vec<&str> = "".splitn(1, 'X').collect();
assert_eq!(v, [""]);
```
A more complex pattern, using a closure:

1.0.0 · Source#### pub fn rsplitn<P>(&self, n: usize, pat: P) -> RSplitN<'_, P> ⓘ

 

#### pub fn rsplitn<P>(&self, n: usize, pat: P) -> RSplitN<'_, P> ⓘ

Returns an iterator over substrings of this string slice, separated by a
pattern, starting from the end of the string, restricted to returning at
most `n` items.

If `n` substrings are returned, the last substring (the `n`th substring)
will contain the remainder of the string.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator will not be double ended, because it is not efficient to support.

For splitting from the front, the `splitn` method can be used.

##### §Examples

Simple patterns:

```
let v: Vec<&str> = "Mary had a little lamb".rsplitn(3, ' ').collect();
assert_eq!(v, ["lamb", "little", "Mary had a"]);
let v: Vec<&str> = "lionXXtigerXleopard".rsplitn(3, 'X').collect();
assert_eq!(v, ["leopard", "tiger", "lionX"]);
let v: Vec<&str> = "lion::tiger::leopard".rsplitn(2, "::").collect();
assert_eq!(v, ["leopard", "lion::tiger"]);
```
A more complex pattern, using a closure:

1.52.0 · Source#### pub fn split_once<P>(&self, delimiter: P) -> Option<(&str, &str)>where
    P: Pattern,

 

#### pub fn split_once<P>(&self, delimiter: P) -> Option<(&str, &str)>where
    P: Pattern,

Splits the string on the first occurrence of the specified delimiter and returns prefix before delimiter and suffix after delimiter.

##### §Examples

1.52.0 · Source#### pub fn rsplit_once<P>(&self, delimiter: P) -> Option<(&str, &str)>

 

#### pub fn rsplit_once<P>(&self, delimiter: P) -> Option<(&str, &str)>

Splits the string on the last occurrence of the specified delimiter and returns prefix before delimiter and suffix after delimiter.

##### §Examples

1.2.0 · Source#### pub fn matches<P>(&self, pat: P) -> Matches<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn matches<P>(&self, pat: P) -> Matches<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over the disjoint matches of a pattern within the given string slice.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator will be a `DoubleEndedIterator` if the pattern
allows a reverse search and forward/reverse search yields the same
elements. This is true for, e.g., `char`, but not for `&str`.

If the pattern allows a reverse search but its results might differ
from a forward search, the `rmatches` method can be used.

##### §Examples

1.2.0 · Source#### pub fn rmatches<P>(&self, pat: P) -> RMatches<'_, P> ⓘ

 

#### pub fn rmatches<P>(&self, pat: P) -> RMatches<'_, P> ⓘ

Returns an iterator over the disjoint matches of a pattern within this string slice, yielded in reverse order.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator requires that the pattern supports a reverse
search, and it will be a `DoubleEndedIterator` if a forward/reverse
search yields the same elements.

For iterating from the front, the `matches` method can be used.

##### §Examples

1.5.0 · Source#### pub fn match_indices<P>(&self, pat: P) -> MatchIndices<'_, P> ⓘwhere
    P: Pattern,

 

#### pub fn match_indices<P>(&self, pat: P) -> MatchIndices<'_, P> ⓘwhere
    P: Pattern,

Returns an iterator over the disjoint matches of a pattern within this string slice as well as the index that the match starts at.

For matches of `pat` within `self` that overlap, only the indices
corresponding to the first match are returned.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator will be a `DoubleEndedIterator` if the pattern
allows a reverse search and forward/reverse search yields the same
elements. This is true for, e.g., `char`, but not for `&str`.

If the pattern allows a reverse search but its results might differ
from a forward search, the `rmatch_indices` method can be used.

##### §Examples

```
let v: Vec<_> = "abcXXXabcYYYabc".match_indices("abc").collect();
assert_eq!(v, [(0, "abc"), (6, "abc"), (12, "abc")]);
let v: Vec<_> = "1abcabc2".match_indices("abc").collect();
assert_eq!(v, [(1, "abc"), (4, "abc")]);
let v: Vec<_> = "ababa".match_indices("aba").collect();
assert_eq!(v, [(0, "aba")]); // only the first `aba`
```
1.5.0 · Source#### pub fn rmatch_indices<P>(&self, pat: P) -> RMatchIndices<'_, P> ⓘ

 

#### pub fn rmatch_indices<P>(&self, pat: P) -> RMatchIndices<'_, P> ⓘ

Returns an iterator over the disjoint matches of a pattern within `self`,
yielded in reverse order along with the index of the match.

For matches of `pat` within `self` that overlap, only the indices
corresponding to the last match are returned.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Iterator behavior

The returned iterator requires that the pattern supports a reverse
search, and it will be a `DoubleEndedIterator` if a forward/reverse
search yields the same elements.

For iterating from the front, the `match_indices` method can be used.

##### §Examples

```
let v: Vec<_> = "abcXXXabcYYYabc".rmatch_indices("abc").collect();
assert_eq!(v, [(12, "abc"), (6, "abc"), (0, "abc")]);
let v: Vec<_> = "1abcabc2".rmatch_indices("abc").collect();
assert_eq!(v, [(4, "abc"), (1, "abc")]);
let v: Vec<_> = "ababa".rmatch_indices("aba").collect();
assert_eq!(v, [(2, "aba")]); // only the last `aba`
```
1.0.0 · Source#### pub fn trim(&self) -> &str

 

#### pub fn trim(&self) -> &str

Returns a string slice with leading and trailing whitespace removed.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`, which includes newlines.

##### §Examples

1.30.0 · Source#### pub fn trim_start(&self) -> &str

 

#### pub fn trim_start(&self) -> &str

Returns a string slice with leading whitespace removed.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`, which includes newlines.

##### §Text directionality

A string is a sequence of bytes. `start` in this context means the first
position of that byte string; for a left-to-right language like English or
Russian, this will be left side, and for right-to-left languages like
Arabic or Hebrew, this will be the right side.

##### §Examples

Basic usage:

Directionality:

1.30.0 · Source#### pub fn trim_end(&self) -> &str

 

#### pub fn trim_end(&self) -> &str

Returns a string slice with trailing whitespace removed.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`, which includes newlines.

##### §Text directionality

A string is a sequence of bytes. `end` in this context means the last
position of that byte string; for a left-to-right language like English or
Russian, this will be right side, and for right-to-left languages like
Arabic or Hebrew, this will be the left side.

##### §Examples

Basic usage:

Directionality:

1.0.0 · Source#### pub fn trim_left(&self) -> &str

 👎Deprecated since 1.33.0: superseded by `trim_start`

#### pub fn trim_left(&self) -> &str

superseded by `trim_start`

Returns a string slice with leading whitespace removed.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`.

##### §Text directionality

A string is a sequence of bytes. ‘Left’ in this context means the first
position of that byte string; for a language like Arabic or Hebrew
which are ‘right to left’ rather than ‘left to right’, this will be
the *right* side, not the left.

##### §Examples

Basic usage:

Directionality:

1.0.0 · Source#### pub fn trim_right(&self) -> &str

 👎Deprecated since 1.33.0: superseded by `trim_end`

#### pub fn trim_right(&self) -> &str

superseded by `trim_end`

Returns a string slice with trailing whitespace removed.

‘Whitespace’ is defined according to the terms of the Unicode Derived
Core Property `White_Space`.

##### §Text directionality

A string is a sequence of bytes. ‘Right’ in this context means the last
position of that byte string; for a language like Arabic or Hebrew
which are ‘right to left’ rather than ‘left to right’, this will be
the *left* side, not the right.

##### §Examples

Basic usage:

Directionality:

1.0.0 · Source#### pub fn trim_matches<P>(&self, pat: P) -> &str

 

#### pub fn trim_matches<P>(&self, pat: P) -> &str

Returns a string slice with all prefixes and suffixes that match a pattern repeatedly removed.

The pattern can be a `char`, a slice of `char`s, or a function
or closure that determines if a character matches.

##### §Examples

Simple patterns:

```
assert_eq!("11foo1bar11".trim_matches('1'), "foo1bar");
assert_eq!("123foo1bar123".trim_matches(char::is_numeric), "foo1bar");
let x: &[_] = &['1', '2'];
assert_eq!("12foo1bar12".trim_matches(x), "foo1bar");
```
A more complex pattern, using a closure:

1.30.0 · Source#### pub fn trim_start_matches<P>(&self, pat: P) -> &strwhere
    P: Pattern,

 

#### pub fn trim_start_matches<P>(&self, pat: P) -> &strwhere
    P: Pattern,

Returns a string slice with all prefixes that match a pattern repeatedly removed.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Text directionality

A string is a sequence of bytes. `start` in this context means the first
position of that byte string; for a left-to-right language like English or
Russian, this will be left side, and for right-to-left languages like
Arabic or Hebrew, this will be the right side.

##### §Examples

1.45.0 · Source#### pub fn strip_prefix<P>(&self, prefix: P) -> Option<&str>where
    P: Pattern,

 

#### pub fn strip_prefix<P>(&self, prefix: P) -> Option<&str>where
    P: Pattern,

Returns a string slice with the prefix removed.

If the string starts with the pattern `prefix`, returns the substring after the prefix,
wrapped in `Some`. Unlike `trim_start_matches`, this method removes the prefix exactly once.

If the string does not start with `prefix`, returns `None`.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

1.45.0 · Source#### pub fn strip_suffix<P>(&self, suffix: P) -> Option<&str>

 

#### pub fn strip_suffix<P>(&self, suffix: P) -> Option<&str>

Returns a string slice with the suffix removed.

If the string ends with the pattern `suffix`, returns the substring before the suffix,
wrapped in `Some`.  Unlike `trim_end_matches`, this method removes the suffix exactly once.

If the string does not end with `suffix`, returns `None`.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

Source#### pub fn strip_circumfix<P, S>(&self, prefix: P, suffix: S) -> Option<&str>

 🔬This is a nightly-only experimental API. (`strip_circumfix` #147946)

#### pub fn strip_circumfix<P, S>(&self, prefix: P, suffix: S) -> Option<&str>

`strip_circumfix` #147946)Returns a string slice with the prefix and suffix removed.

If the string starts with the pattern `prefix` and ends with the pattern `suffix`, returns
the substring after the prefix and before the suffix, wrapped in `Some`.
Unlike `trim_start_matches` and `trim_end_matches`, this method removes both the prefix
and suffix exactly once.

If the string does not start with `prefix` or does not end with `suffix`, returns `None`.

Each pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

Source#### pub fn trim_prefix<P>(&self, prefix: P) -> &strwhere
    P: Pattern,

 🔬This is a nightly-only experimental API. (`trim_prefix_suffix` #142312)

#### pub fn trim_prefix<P>(&self, prefix: P) -> &strwhere
    P: Pattern,

`trim_prefix_suffix` #142312)Returns a string slice with the optional prefix removed.

If the string starts with the pattern `prefix`, returns the substring after the prefix.
Unlike `strip_prefix`, this method always returns `&str` for easy method chaining,
instead of returning `Option<&str>`.

If the string does not start with `prefix`, returns the original string unchanged.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

```
#![feature(trim_prefix_suffix)]
// Prefix present - removes it
assert_eq!("foo:bar".trim_prefix("foo:"), "bar");
assert_eq!("foofoo".trim_prefix("foo"), "foo");
// Prefix absent - returns original string
assert_eq!("foo:bar".trim_prefix("bar"), "foo:bar");
// Method chaining example
assert_eq!("<https://example.com/>".trim_prefix('<').trim_suffix('>'), "https://example.com/");
```
Source#### pub fn trim_suffix<P>(&self, suffix: P) -> &str

 🔬This is a nightly-only experimental API. (`trim_prefix_suffix` #142312)

#### pub fn trim_suffix<P>(&self, suffix: P) -> &str

`trim_prefix_suffix` #142312)Returns a string slice with the optional suffix removed.

If the string ends with the pattern `suffix`, returns the substring before the suffix.
Unlike `strip_suffix`, this method always returns `&str` for easy method chaining,
instead of returning `Option<&str>`.

If the string does not end with `suffix`, returns the original string unchanged.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Examples

```
#![feature(trim_prefix_suffix)]
// Suffix present - removes it
assert_eq!("bar:foo".trim_suffix(":foo"), "bar");
assert_eq!("foofoo".trim_suffix("foo"), "foo");
// Suffix absent - returns original string
assert_eq!("bar:foo".trim_suffix("bar"), "bar:foo");
// Method chaining example
assert_eq!("<https://example.com/>".trim_prefix('<').trim_suffix('>'), "https://example.com/");
```
1.30.0 · Source#### pub fn trim_end_matches<P>(&self, pat: P) -> &str

 

#### pub fn trim_end_matches<P>(&self, pat: P) -> &str

Returns a string slice with all suffixes that match a pattern repeatedly removed.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Text directionality

A string is a sequence of bytes. `end` in this context means the last
position of that byte string; for a left-to-right language like English or
Russian, this will be right side, and for right-to-left languages like
Arabic or Hebrew, this will be the left side.

##### §Examples

Simple patterns:

```
assert_eq!("11foo1bar11".trim_end_matches('1'), "11foo1bar");
assert_eq!("123foo1bar123".trim_end_matches(char::is_numeric), "123foo1bar");
let x: &[_] = &['1', '2'];
assert_eq!("12foo1bar12".trim_end_matches(x), "12foo1bar");
```
A more complex pattern, using a closure:

1.0.0 · Source#### pub fn trim_left_matches<P>(&self, pat: P) -> &strwhere
    P: Pattern,

 👎Deprecated since 1.33.0: superseded by `trim_start_matches`

#### pub fn trim_left_matches<P>(&self, pat: P) -> &strwhere
    P: Pattern,

superseded by `trim_start_matches`

Returns a string slice with all prefixes that match a pattern repeatedly removed.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Text directionality

A string is a sequence of bytes. ‘Left’ in this context means the first
position of that byte string; for a language like Arabic or Hebrew
which are ‘right to left’ rather than ‘left to right’, this will be
the *right* side, not the left.

##### §Examples

1.0.0 · Source#### pub fn trim_right_matches<P>(&self, pat: P) -> &str

 👎Deprecated since 1.33.0: superseded by `trim_end_matches`

#### pub fn trim_right_matches<P>(&self, pat: P) -> &str

superseded by `trim_end_matches`

Returns a string slice with all suffixes that match a pattern repeatedly removed.

The pattern can be a `&str`, `char`, a slice of `char`s, or a
function or closure that determines if a character matches.

##### §Text directionality

A string is a sequence of bytes. ‘Right’ in this context means the last
position of that byte string; for a language like Arabic or Hebrew
which are ‘right to left’ rather than ‘left to right’, this will be
the *left* side, not the right.

##### §Examples

Simple patterns:

```
assert_eq!("11foo1bar11".trim_right_matches('1'), "11foo1bar");
assert_eq!("123foo1bar123".trim_right_matches(char::is_numeric), "123foo1bar");
let x: &[_] = &['1', '2'];
assert_eq!("12foo1bar12".trim_right_matches(x), "12foo1bar");
```
A more complex pattern, using a closure:

1.0.0 · Source#### pub fn parse<F>(&self) -> Result<F, <F as FromStr>::Err>where
    F: FromStr,

 

#### pub fn parse<F>(&self) -> Result<F, <F as FromStr>::Err>where
    F: FromStr,

Parses this string slice into another type.

Because `parse` is so general, it can cause problems with type
inference. As such, `parse` is one of the few times you’ll see
the syntax affectionately known as the ‘turbofish’: `::<>`. This
helps the inference algorithm understand specifically which type
you’re trying to parse into.

`parse` can parse into any type that implements the `FromStr` trait.

##### §Errors

Will return `Err` if it’s not possible to parse this string slice into
the desired type.

##### §Examples

Basic usage:

Using the ‘turbofish’ instead of annotating `four`:

Failing to parse:

1.23.0 · Source#### pub fn is_ascii(&self) -> bool

 

#### pub fn is_ascii(&self) -> bool

Checks if all characters in this string are within the ASCII range.

An empty string returns `true`.

##### §Examples

Source#### pub fn as_ascii(&self) -> Option<&[AsciiChar]>

 🔬This is a nightly-only experimental API. (`ascii_char` #110998)

#### pub fn as_ascii(&self) -> Option<&[AsciiChar]>

`ascii_char` #110998)If this string slice `is_ascii`, returns it as a slice
of ASCII characters, otherwise returns `None`.

Source#### pub unsafe fn as_ascii_unchecked(&self) -> &[AsciiChar]

 🔬This is a nightly-only experimental API. (`ascii_char` #110998)

#### pub unsafe fn as_ascii_unchecked(&self) -> &[AsciiChar]

`ascii_char` #110998)Converts this string slice into a slice of ASCII characters, without checking whether they are valid.

##### §Safety

Every character in this string must be ASCII, or else this is UB.

1.23.0 · Source#### pub fn eq_ignore_ascii_case(&self, other: &str) -> bool

 

#### pub fn eq_ignore_ascii_case(&self, other: &str) -> bool

Checks that two strings are an ASCII case-insensitive match.

Same as `to_ascii_lowercase(a) == to_ascii_lowercase(b)`,
but without allocating and copying temporaries.

##### §Examples

1.23.0 · Source#### pub fn make_ascii_uppercase(&mut self)

 

#### pub fn make_ascii_uppercase(&mut self)

Converts this string to its ASCII upper case equivalent in-place.

ASCII letters ‘a’ to ‘z’ are mapped to ‘A’ to ‘Z’, but non-ASCII letters are unchanged.

To return a new uppercased value without modifying the existing one, use
`to_ascii_uppercase()`.

##### §Examples

1.23.0 · Source#### pub fn make_ascii_lowercase(&mut self)

 

#### pub fn make_ascii_lowercase(&mut self)

Converts this string to its ASCII lower case equivalent in-place.

ASCII letters ‘A’ to ‘Z’ are mapped to ‘a’ to ‘z’, but non-ASCII letters are unchanged.

To return a new lowercased value without modifying the existing one, use
`to_ascii_lowercase()`.

##### §Examples

1.80.0 · Source#### pub fn trim_ascii_start(&self) -> &str

 

#### pub fn trim_ascii_start(&self) -> &str

Returns a string slice with leading ASCII whitespace removed.

‘Whitespace’ refers to the definition used by
`u8::is_ascii_whitespace`.

##### §Examples

1.80.0 · Source#### pub fn trim_ascii_end(&self) -> &str

 

#### pub fn trim_ascii_end(&self) -> &str

Returns a string slice with trailing ASCII whitespace removed.

‘Whitespace’ refers to the definition used by
`u8::is_ascii_whitespace`.

##### §Examples

1.80.0 · Source#### pub fn trim_ascii(&self) -> &str

 

#### pub fn trim_ascii(&self) -> &str

Returns a string slice with leading and trailing ASCII whitespace removed.

‘Whitespace’ refers to the definition used by
`u8::is_ascii_whitespace`.

##### §Examples

1.34.0 · Source#### pub fn escape_debug(&self) -> EscapeDebug<'_> ⓘ

 

#### pub fn escape_debug(&self) -> EscapeDebug<'_> ⓘ

Returns an iterator that escapes each char in `self` with `char::escape_debug`.

Note: only extended grapheme codepoints that begin the string will be escaped.

##### §Examples

As an iterator:

Using `println!` directly:

Both are equivalent to:

Using `to_string`:

1.34.0 · Source#### pub fn escape_default(&self) -> EscapeDefault<'_> ⓘ

 

#### pub fn escape_default(&self) -> EscapeDefault<'_> ⓘ

Returns an iterator that escapes each char in `self` with `char::escape_default`.

##### §Examples

As an iterator:

Using `println!` directly:

Both are equivalent to:

Using `to_string`:

1.34.0 · Source#### pub fn escape_unicode(&self) -> EscapeUnicode<'_> ⓘ

 

#### pub fn escape_unicode(&self) -> EscapeUnicode<'_> ⓘ

Returns an iterator that escapes each char in `self` with `char::escape_unicode`.

##### §Examples

As an iterator:

Using `println!` directly:

Both are equivalent to:

Using `to_string`:

Source#### pub fn substr_range(&self, substr: &str) -> Option<Range<usize>>

 🔬This is a nightly-only experimental API. (`substr_range` #126769)

#### pub fn substr_range(&self, substr: &str) -> Option<Range<usize>>

`substr_range` #126769)Returns the range that a substring points to.

Returns `None` if `substr` does not point within `self`.

Unlike `str::find`, **this does not search through the string**.
Instead, it uses pointer arithmetic to find where in the string
`substr` is derived from.

This is useful for extending `str::split` and similar methods.

Note that this method may return false positives (typically either
`Some(0..0)` or `Some(self.len()..self.len())`) if `substr` is a
zero-length `str` that points at the beginning or end of another,
independent, `str`.

##### §Examples

```
#![feature(substr_range)]
use core::range::Range;
let data = "a, b, b, a";
let mut iter = data.split(", ").map(|s| data.substr_range(s).unwrap());
assert_eq!(iter.next(), Some(Range { start: 0, end: 1 }));
assert_eq!(iter.next(), Some(Range { start: 3, end: 4 }));
assert_eq!(iter.next(), Some(Range { start: 6, end: 7 }));
assert_eq!(iter.next(), Some(Range { start: 9, end: 10 }));
```
Source#### pub fn as_str(&self) -> &str

 🔬This is a nightly-only experimental API. (`str_as_str` #130366)

#### pub fn as_str(&self) -> &str

`str_as_str` #130366)Returns the same string as a string slice `&str`.

This method is redundant when used directly on `&str`, but
it helps dereferencing other string-like types to string slices,
for example references to `Box<str>` or `Arc<str>`.

1.0.0 · Source#### pub fn replace<P>(&self, from: P, to: &str) -> Stringwhere
    P: Pattern,

 

#### pub fn replace<P>(&self, from: P, to: &str) -> Stringwhere
    P: Pattern,

Replaces all matches of a pattern with another string.

`replace` creates a new `String`, and copies the data from this string slice into it.
While doing so, it attempts to find matches of a pattern. If it finds any, it
replaces them with the replacement string slice.

##### §Examples

```
let s = "this is old";
assert_eq!("this is new", s.replace("old", "new"));
assert_eq!("than an old", s.replace("is", "an"));
```
When the pattern doesn’t match, it returns this string slice as `String`:

1.16.0 · Source#### pub fn replacen<P>(&self, pat: P, to: &str, count: usize) -> Stringwhere
    P: Pattern,

 

#### pub fn replacen<P>(&self, pat: P, to: &str, count: usize) -> Stringwhere
    P: Pattern,

Replaces first N matches of a pattern with another string.

`replacen` creates a new `String`, and copies the data from this string slice into it.
While doing so, it attempts to find matches of a pattern. If it finds any, it
replaces them with the replacement string slice at most `count` times.

##### §Examples

```
let s = "foo foo 123 foo";
assert_eq!("new new 123 foo", s.replacen("foo", "new", 2));
assert_eq!("faa fao 123 foo", s.replacen('o', "a", 3));
assert_eq!("foo foo new23 foo", s.replacen(char::is_numeric, "new", 1));
```
When the pattern doesn’t match, it returns this string slice as `String`:

1.2.0 · Source#### pub fn to_lowercase(&self) -> String

 

#### pub fn to_lowercase(&self) -> String

Returns the lowercase equivalent of this string slice, as a new `String`.

‘Lowercase’ is defined according to the terms of the Unicode Derived Core Property
`Lowercase`.

Since some characters can expand into multiple characters when changing
the case, this function returns a `String` instead of modifying the
parameter in-place.

##### §Examples

Basic usage:

A tricky example, with sigma:

```
let sigma = "Σ";
assert_eq!("σ", sigma.to_lowercase());
// but at the end of a word, it's ς, not σ:
let odysseus = "ὈΔΥΣΣΕΎΣ";
assert_eq!("ὀδυσσεύς", odysseus.to_lowercase());
```
Languages without case are not changed:

1.2.0 · Source#### pub fn to_uppercase(&self) -> String

 

#### pub fn to_uppercase(&self) -> String

Returns the uppercase equivalent of this string slice, as a new `String`.

‘Uppercase’ is defined according to the terms of the Unicode Derived Core Property
`Uppercase`.

Since some characters can expand into multiple characters when changing
the case, this function returns a `String` instead of modifying the
parameter in-place.

##### §Examples

Basic usage:

Scripts without case are not changed:

One character can become multiple:

1.16.0 · Source#### pub fn repeat(&self, n: usize) -> String

 

#### pub fn repeat(&self, n: usize) -> String

1.23.0 · Source#### pub fn to_ascii_uppercase(&self) -> String

 

#### pub fn to_ascii_uppercase(&self) -> String

Returns a copy of this string where each character is mapped to its ASCII upper case equivalent.

ASCII letters ‘a’ to ‘z’ are mapped to ‘A’ to ‘Z’, but non-ASCII letters are unchanged.

To uppercase the value in-place, use `make_ascii_uppercase`.

To uppercase ASCII characters in addition to non-ASCII characters, use
`to_uppercase`.

##### §Examples

1.23.0 · Source#### pub fn to_ascii_lowercase(&self) -> String

 

#### pub fn to_ascii_lowercase(&self) -> String

Returns a copy of this string where each character is mapped to its ASCII lower case equivalent.

ASCII letters ‘A’ to ‘Z’ are mapped to ‘a’ to ‘z’, but non-ASCII letters are unchanged.

To lowercase the value in-place, use `make_ascii_lowercase`.

To lowercase ASCII characters in addition to non-ASCII characters, use
`to_lowercase`.

##### §Examples

## Trait Implementations§

1.0.0 · Source§### impl Add<&str> for String

Implements the `+` operator for concatenating two strings.

 

### impl Add<&str> for String

Implements the `+` operator for concatenating two strings.

This consumes the `String` on the left-hand side and re-uses its buffer (growing it if
necessary). This is done to avoid allocating a new `String` and copying the entire contents on
every operation, which would lead to *O*(*n*^2) running time when building an *n*-byte string by
repeated concatenation.

The string on the right-hand side is only borrowed; its contents are copied into the returned
`String`.

#### §Examples

Concatenating two `String`s takes the first by value and borrows the second:

```
let a = String::from("hello");
let b = String::from(" world");
let c = a + &b;
// `a` is moved and can no longer be used here.
```
If you want to keep using the first `String`, you can clone it and append to the clone instead:

```
let a = String::from("hello");
let b = String::from(" world");
let c = a.clone() + &b;
// `a` is still valid here.
```
Concatenating `&str` slices can be done by converting the first to a `String`:

1.12.0 · Source§### impl AddAssign<&str> for String

Implements the `+=` operator for appending to a `String`.

 

### impl AddAssign<&str> for String

Implements the `+=` operator for appending to a `String`.

This has the same behavior as the `push_str` method.

Source§#### fn add_assign(&mut self, other: &str)

 

#### fn add_assign(&mut self, other: &str)

`+=` operation. Read more1.36.0 · Source§### impl BorrowMut<str> for String

 

### impl BorrowMut<str> for String

Source§#### fn borrow_mut(&mut self) -> &mut str

 

#### fn borrow_mut(&mut self) -> &mut str

Source§### impl<'a> Extend<&'a AsciiChar> for String

 

### impl<'a> Extend<&'a AsciiChar> for String

Source§#### fn extend<I>(&mut self, iter: I)where
    I: IntoIterator<Item = &'a AsciiChar>,

 

#### fn extend<I>(&mut self, iter: I)where
    I: IntoIterator<Item = &'a AsciiChar>,

Source§#### fn extend_one(&mut self, c: &'a AsciiChar)

 

#### fn extend_one(&mut self, c: &'a AsciiChar)

`extend_one` #72631)1.2.0 · Source§### impl<'a> Extend<&'a char> for String

 

### impl<'a> Extend<&'a char> for String

1.0.0 · Source§### impl<'a> Extend<&'a str> for String

 

### impl<'a> Extend<&'a str> for String

1.45.0 · Source§### impl<A> Extend<Box<str, A>> for Stringwhere
    A: Allocator,

 

### impl<A> Extend<Box<str, A>> for Stringwhere
    A: Allocator,

Source§### impl Extend<AsciiChar> for String

 

### impl Extend<AsciiChar> for String

1.19.0 · Source§### impl<'a> Extend<Cow<'a, str>> for String

 

### impl<'a> Extend<Cow<'a, str>> for String

1.4.0 · Source§### impl Extend<String> for String

 

### impl Extend<String> for String

1.0.0 · Source§### impl Extend<char> for String

 

### impl Extend<char> for String

1.14.0 · Source§### impl<'a> From<Cow<'a, str>> for String

 

### impl<'a> From<Cow<'a, str>> for String

1.6.0 · Source§### impl<'a> From<String> for Box<dyn Error + 'a>

 

### impl<'a> From<String> for Box<dyn Error + 'a>

1.0.0 · Source§### impl<'a> From<String> for Box<dyn Error + Sync + Send + 'a>

 

### impl<'a> From<String> for Box<dyn Error + Sync + Send + 'a>

Source§### impl<'a> FromIterator<&'a AsciiChar> for String

 

### impl<'a> FromIterator<&'a AsciiChar> for String

1.17.0 · Source§### impl<'a> FromIterator<&'a char> for String

 

### impl<'a> FromIterator<&'a char> for String

1.0.0 · Source§### impl<'a> FromIterator<&'a str> for String

 

### impl<'a> FromIterator<&'a str> for String

Source§### impl FromIterator<AsciiChar> for String

 

### impl FromIterator<AsciiChar> for String

1.4.0 · Source§### impl FromIterator<String> for String

 

### impl FromIterator<String> for String

1.0.0 · Source§### impl FromIterator<char> for String

 

### impl FromIterator<char> for String

1.0.0 · Source§### impl Ord for String

 

### impl Ord for String

Source§### impl PartialEq<ByteString> for String

 

### impl PartialEq<ByteString> for String

Source§### impl PartialEq<String> for ByteString

 

### impl PartialEq<String> for ByteString

1.0.0 · Source§### impl PartialOrd for String

 

### impl PartialOrd for String

Source§### impl<'b> Pattern for &'b String

A convenience impl that delegates to the impl for `&str`.

 

### impl<'b> Pattern for &'b String

A convenience impl that delegates to the impl for `&str`.

#### §Examples

Source§#### type Searcher<'a> = <&'b str as Pattern>::Searcher<'a>

 

#### type Searcher<'a> = <&'b str as Pattern>::Searcher<'a>

`pattern` #27721)Source§#### fn into_searcher(self, haystack: &str) -> <&'b str as Pattern>::Searcher<'_>

 

#### fn into_searcher(self, haystack: &str) -> <&'b str as Pattern>::Searcher<'_>

`pattern` #27721)`self` and the `haystack` to search in.Source§#### fn is_contained_in(self, haystack: &str) -> bool

 

#### fn is_contained_in(self, haystack: &str) -> bool

`pattern` #27721)Source§#### fn is_prefix_of(self, haystack: &str) -> bool

 

#### fn is_prefix_of(self, haystack: &str) -> bool

`pattern` #27721)Source§#### fn strip_prefix_of(self, haystack: &str) -> Option<&str>

 

#### fn strip_prefix_of(self, haystack: &str) -> Option<&str>

`pattern` #27721)Source§#### fn is_suffix_of<'a>(self, haystack: &'a str) -> bool

 

#### fn is_suffix_of<'a>(self, haystack: &'a str) -> bool

`pattern` #27721)Source§#### fn strip_suffix_of<'a>(self, haystack: &'a str) -> Option<&'a str>

 

#### fn strip_suffix_of<'a>(self, haystack: &'a str) -> Option<&'a str>

`pattern` #27721)Source§#### fn as_utf8_pattern(&self) -> Option<Utf8Pattern<'_>>

 

#### fn as_utf8_pattern(&self) -> Option<Utf8Pattern<'_>>

`pattern` #27721)1.16.0 · Source§### impl ToSocketAddrs for String

 

### impl ToSocketAddrs for String

Source§#### type Iter = IntoIter<SocketAddr>

 

#### type Iter = IntoIter<SocketAddr>

Source§#### fn to_socket_addrs(&self) -> Result<IntoIter<SocketAddr>>

 

#### fn to_socket_addrs(&self) -> Result<IntoIter<SocketAddr>>

`SocketAddr`s. Read more

# Citations

1. Source page: https://doc.rust-lang.org/stable/std/string/struct.String.html
