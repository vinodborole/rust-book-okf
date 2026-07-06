---
type: Web Page
title: Attributes - The Rust Reference
resource: https://doc.rust-lang.org/stable/reference/attributes.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

# Attributes

Syntax

InnerAttribute → # ! [ Attr ]

OuterAttribute → # [ Attr ]

Attr → 

      SimplePath AttrInput? 

    | unsafe ( SimplePath AttrInput? )

An *attribute* is a general, free-form metadatum that is interpreted according to name, convention, language, and compiler version. Attributes are modeled on Attributes in ECMA-335, with the syntax coming from ECMA-334 (C#).

*Inner attributes*, written with a bang (`!`) after the hash (`#`), apply to the form that the attribute is declared within.

Example

`#![allow(unused)] fn main() { // General metadata applied to the enclosing module or crate. #![crate_type = "lib"] // Inner attribute applies to the entire function. fn some_unused_variables() { #![allow(unused_variables)] let x = (); let y = (); let z = (); } }`

*Outer attributes*, written without the bang after the hash, apply to the form that follows the attribute.

Example

`#![allow(unused)] fn main() { // A function marked as a unit test #[test] fn test_foo() { /* ... */ } // A conditionally-compiled module #[cfg(target_os = "linux")] mod bar { /* ... */ } // A lint attribute used to suppress a warning/error #[allow(non_camel_case_types)] type int8_t = i8; }`

The attribute consists of a path to the attribute, followed by an optional delimited token tree whose interpretation is defined by the attribute. Attributes other than macro attributes also allow the input to be an equals sign (`=`) followed by an expression. See the meta item syntax below for more details.

An attribute may be unsafe to apply. To avoid undefined behavior when using these attributes, certain obligations that cannot be checked by the compiler must be met. To assert these have been, the attribute is wrapped in `unsafe(..)`, e.g. `#[unsafe(no_mangle)]`.

The following attributes are unsafe:

Attributes can be classified into the following kinds:

Attributes may be applied to many forms in the language:

- All item declarations accept outer attributes while external blocks, functions, implementations, and modules accept inner attributes.
- Most statements accept outer attributes (see Expression Attributes for limitations on expression statements).
- Block expressions accept outer and inner attributes, but only when they are the outer expression of an expression statement or the final expression of another block expression.
- Enum variants and struct and union fields accept outer attributes.
- Match expression arms accept outer attributes.
- Generic lifetime or type parameter accept outer attributes.
- Expressions accept outer attributes in limited situations, see Expression Attributes for details.
- Function, closure and function pointer parameters accept outer attributes. This includes attributes on variadic parameters denoted with `...`in function pointers and external blocks.
- Inline assembly template strings and operands accept outer attributes. Only certain attributes are accepted semantically; for details, see asm.attributes.supported-attributes.

## Meta item attribute syntax

A “meta item” is the syntax used for the Attr rule by most built-in attributes. It has the following grammar:

Syntax

MetaItem → 

      SimplePath 

    | SimplePath = Expression 

    | SimplePath ( MetaSeq? )

MetaSeq → 

    MetaItemInner ( , MetaItemInner )* ,?

Expressions in meta items must macro-expand to literal expressions, which must not include integer or float type suffixes. Expressions which are not literal expressions will be syntactically accepted (and can be passed to proc-macros), but will be rejected after parsing.

Note that if the attribute appears within another macro, it will be expanded after that outer macro. For example, the following code will expand the `Serialize` proc-macro first, which must preserve the `include_str!` call in order for it to be expanded:

```
#[derive(Serialize)]
struct Foo {
    #[doc = include_str!("x.md")]
    x: u32
}
```
Additionally, macros in attributes will be expanded only after all other attributes applied to the item:

```
#[macro_attr1] // expanded first
#[doc = mac!()] // `mac!` is expanded fourth.
#[macro_attr2] // expanded second
#[derive(MacroDerive1, MacroDerive2)] // expanded third
fn foo() {}
```
Various built-in attributes use different subsets of the meta item syntax to specify their inputs. The following grammar rules show some commonly used forms:

Syntax

MetaWord → 

    IDENTIFIER

MetaNameValueStr → 

    IDENTIFIER = ( STRING_LITERAL | RAW_STRING_LITERAL )

MetaListPaths → 

    IDENTIFIER ( ( SimplePath ( , SimplePath )* ,? )? )

MetaListIdents → 

    IDENTIFIER ( ( IDENTIFIER ( , IDENTIFIER )* ,? )? )

MetaListNameValueStr → 

    IDENTIFIER ( ( MetaNameValueStr ( , MetaNameValueStr )* ,? )? )

Some examples of meta items are:

| Style | Example | 
|---|---|
| MetaWord | `no_std` | 
| MetaNameValueStr | `doc = "example"` | 
| MetaListPaths | `allow(unused, clippy::inline_always)` | 
| MetaListIdents | `macro_use(foo, bar)` | 
| MetaListNameValueStr | `link(name = "CoreFoundation", kind = "framework")` | 

## Active and inert attributes

An attribute is either active or inert. During attribute processing, *active attributes* remove themselves from the form they are on while *inert attributes* stay on.

The `cfg` and `cfg_attr` attributes are active. Attribute macros are active. All other attributes are inert.

## Tool attributes

The compiler may allow attributes for external tools where each tool resides in its own module in the tool prelude. The first segment of the attribute path is the name of the tool, with one or more additional segments whose interpretation is up to the tool.

When a tool is not in use, the tool’s attributes are accepted without a warning. When the tool is in use, the tool is responsible for processing and interpretation of its attributes.

Tool attributes are not available if the `no_implicit_prelude` attribute is used.

```
#![allow(unused)]
fn main() {
// Tells the rustfmt tool to not format the following element.
#[rustfmt::skip]
struct S {
}
// Controls the "cyclomatic complexity" threshold for the clippy tool.
#[clippy::cyclomatic_complexity = "100"]
pub fn f() {}
}
```
Note

`rustc`currently recognizes the tools “clippy”, “rustfmt”, “diagnostic”, “miri”, and “rust_analyzer”.

## Built-in attributes index

The following is an index of all built-in attributes.

- 
Conditional compilation 
- 
Testing - `test`— Marks a function as a test.
- `ignore`— Disables a test function.
- `should_panic`— Indicates a test should generate a panic.
 
- 
Derive - `derive`— Automatic trait implementations.
- `automatically_derived`— Marker for implementations created by- `derive`.
 
- 
Macros - `macro_export`— Exports a- `macro_rules`macro for cross-crate usage.
- `macro_use`— Expands macro visibility, or imports macros from other crates.
- `proc_macro`— Defines a function-like macro.
- `proc_macro_derive`— Defines a derive macro.
- `proc_macro_attribute`— Defines an attribute macro.
 
- 
Diagnostics - `allow`,- `expect`,- `warn`,- `deny`,- `forbid`— Alters the default lint level.
- `deprecated`— Generates deprecation notices.
- `must_use`— Generates a lint for unused values.
- `diagnostic::on_unimplemented`— Hints the compiler to emit a certain error message if a trait is not implemented.
- `diagnostic::do_not_recommend`— Hints the compiler to not show a certain trait impl in error messages.
 
- 
ABI, linking, symbols, and FFI - `link`— Specifies a native library to link with an- `extern`block.
- `link_name`— Specifies the name of the symbol for functions or statics in an- `extern`block.
- `link_ordinal`— Specifies the ordinal of the symbol for functions or statics in an- `extern`block.
- `no_link`— Prevents linking an extern crate.
- `repr`— Controls type layout.
- `crate_type`— Specifies the type of crate (library, executable, etc.).
- `no_main`— Disables emitting the- `main`symbol.
- `export_name`— Specifies the exported symbol name for a function or static.
- `link_section`— Specifies the section of an object file to use for a function or static.
- `no_mangle`— Disables symbol name encoding.
- `used`— Forces the compiler to keep a static item in the output object file.
- `crate_name`— Specifies the crate name.
 
- 
Code generation - `inline`— Hint to inline code.
- `cold`— Hint that a function is unlikely to be called.
- `naked`— Prevent the compiler from emitting a function prologue and epilogue.
- `no_builtins`— Disables use of certain built-in functions.
- `target_feature`— Configure platform-specific code generation.
- `track_caller`— Pass the parent call location to- `std::panic::Location::caller()`.
- `instruction_set`— Specify the instruction set used to generate a function’s code.
 
- 
Documentation - `doc`— Specifies documentation. See The Rustdoc Book for more information. Doc comments are transformed into- `doc`attributes.
 
- 
Preludes - `no_std`— Removes std from the prelude.
- `no_implicit_prelude`— Disables prelude lookups within a module.
 
- 
Modules - `path`— Specifies the filename for a module.
 
- 
Limits - `recursion_limit`— Sets the maximum recursion limit for certain compile-time operations.
- `type_length_limit`— Sets the maximum size of a polymorphic type.
 
- 
Runtime - `panic_handler`— Sets the function to handle panics.
- `global_allocator`— Sets the global memory allocator.
- `windows_subsystem`— Specifies the windows subsystem to link with.
 
- 
Features - `feature`— Used to enable unstable or experimental compiler features. See The Unstable Book for features implemented in- `rustc`.
 
- 
Type System - `non_exhaustive`— Indicate that a type will have more fields/variants added in future.
 
- 
Debugger - `debugger_visualizer`— Embeds a file that specifies debugger output for a type.
- `collapse_debuginfo`— Controls how macro invocations are encoded in debuginfo.

# Citations

1. Source page: https://doc.rust-lang.org/stable/reference/attributes.html
