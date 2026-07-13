---
type: Web Page
title: A - Keywords - The Rust Programming Language
resource: https://doc.rust-lang.org/stable/book/appendix-01-keywords.html
timestamp: '2026-07-13T09:33:08.854356+00:00'
---

[Appendix A: Keywords](#appendix-a-keywords)

The following lists contain keywords that are reserved for current or future
use by the Rust language. As such, they cannot be used as identifiers (except
as raw identifiers, as we discuss in the [“Raw
Identifiers”](#raw-identifiers) section). *Identifiers* are names
of functions, variables, parameters, struct fields, modules, crates, constants,
macros, static values, attributes, types, traits, or lifetimes.

[Keywords Currently in Use](#keywords-currently-in-use)

The following is a list of keywords currently in use, with their functionality described.

- `as`- `use`statements.
- `async`- `Future`instead of blocking the current thread.
- `await`- `Future`is ready.
- `break`
- `const`
- `continue`
- `crate`
- `dyn`
- `else`- `if`and- `if let`control flow constructs.
- `enum`
- `extern`
- `false`
- `fn`
- `for`
- `if`
- `impl`
- `in`- `for`loop syntax.
- `let`
- `loop`
- `match`
- `mod`
- `move`
- `mut`
- `pub`- `impl`blocks, or modules.
- `ref`
- `return`
- `Self`
- `self`
- `static`
- `struct`
- `super`
- `trait`
- `true`
- `type`
- `union`- [union](../reference/items/unions.html); is a keyword only when used in a union declaration.
- `unsafe`
- `use`
- `where`
- `while`

[Keywords Reserved for Future Use](#keywords-reserved-for-future-use)

The following keywords do not yet have any functionality but are reserved by Rust for potential future use:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

[Raw Identifiers](#raw-identifiers)

*Raw identifiers* are the syntax that lets you use keywords where they wouldn’t
normally be allowed. You use a raw identifier by prefixing a keyword with `r#`.

For example, `match` is a keyword. If you try to compile the following function
that uses `match` as its name:

Filename: src/main.rs

```
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```
you’ll get this error:

```
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```
The error shows that you can’t use the keyword `match` as the function
identifier. To use `match` as a function name, you need to use the raw
identifier syntax, like this:

Filename: src/main.rs

```
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
fn main() {
    assert!(r#match("foo", "foobar"));
}
```
This code will compile without any errors. Note the `r#` prefix on the function
name in its definition as well as where the function is called in `main`.

Raw identifiers allow you to use any word you choose as an identifier, even if
that word happens to be a reserved keyword. This gives us more freedom to choose
identifier names, as well as lets us integrate with programs written in a
language where these words aren’t keywords. In addition, raw identifiers allow
you to use libraries written in a different Rust edition than your crate uses.
For example, `try` isn’t a keyword in the 2015 edition but is in the 2018, 2021,
and 2024 editions. If you depend on a library that is written using the 2015
edition and has a `try` function, you’ll need to use the raw identifier syntax,
`r#try` in this case, to call that function from your code on later editions.
See [Appendix E](appendix-05-editions.html) for more information on editions.

# Citations

1. Source page: https://doc.rust-lang.org/stable/book/appendix-01-keywords.html
