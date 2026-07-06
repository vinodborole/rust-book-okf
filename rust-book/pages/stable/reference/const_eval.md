---
type: Web Page
title: Constant evaluation - The Rust Reference
resource: https://doc.rust-lang.org/stable/reference/const_eval.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

# Constant evaluation

Constant evaluation is the process of computing the result of expressions during compilation. Only a subset of all expressions can be evaluated at compile-time.

## Constant expressions

Certain forms of expressions, called constant expressions, can be evaluated at compile time.

Expressions in a const context must be constant expressions.

Expressions in const contexts are always evaluated at compile time.

Outside of const contexts, constant expressions *may* be, but are not guaranteed to be, evaluated at compile time.

Behaviors such as out of bounds array indexing or overflow are compiler errors if the value must be evaluated at compile time (i.e. in const contexts). Otherwise, these behaviors are warnings, but will likely panic at run-time.

The following expressions are constant expressions, so long as any operands are also constant expressions and do not cause any `Drop::drop` calls to be run.

- 
Paths to statics with these restrictions: - Writes to `static`items are not allowed in any constant evaluation context.
- Reads from `extern`statics are not allowed in any constant evaluation context.
- If the evaluation is *not*carried out in an initializer of a`static`item, then reads from any mutable`static`are not allowed. A mutable`static`is a`static mut`item, or a`static`item with an interior-mutable type.
 These requirements are checked only when the constant is evaluated. In other words, having such accesses syntactically occur in const contexts is allowed as long as they never get executed. 
- Writes to 

- Block expressions, including `unsafe`and`const`blocks.- let statements and thus irrefutable patterns, including mutable bindings
- assignment expressions
- compound assignment expressions
- expression statements
 

- Array and slice indexing expressions, where the index is a `usize`.

- Closure expressions which don’t capture variables from the environment.

- Built-in negation, arithmetic, logical, comparison or lazy boolean operators used on integer and floating point types, `bool`, and`char`.

- 
All forms of borrows, including raw borrows, except borrows of expressions whose temporary scopes would be extended (see temporary lifetime extension) to the end of the program and which are either: - Mutable borrows.
- Shared borrows of expressions that result in values with interior mutability.
 `#![allow(unused)] fn main() { // Due to being in tail position, this borrow extends the scope of the // temporary to the end of the program. Since the borrow is mutable, // this is not allowed in a const expression. const C: &u8 = &mut 0; // ERROR not allowed }``#![allow(unused)] fn main() { // Const blocks are similar to initializers of `const` items. let _: &u8 = const { &mut 0 }; // ERROR not allowed }``#![allow(unused)] fn main() { use core::sync::atomic::AtomicU8; // This is not allowed as 1) the temporary scope is extended to the // end of the program and 2) the temporary has interior mutability. const C: &AtomicU8 = &AtomicU8::new(0); // ERROR not allowed }``#![allow(unused)] fn main() { use core::sync::atomic::AtomicU8; // As above. let _: &_ = const { &AtomicU8::new(0) }; // ERROR not allowed }``#![allow(unused)] fn main() { #![allow(static_mut_refs)] // Even though this borrow is mutable, it's not of a temporary, so // this is allowed. const C: &u8 = unsafe { static mut S: u8 = 0; &mut S }; // OK }``#![allow(unused)] fn main() { use core::sync::atomic::AtomicU8; // Even though this borrow is of a value with interior mutability, // it's not of a temporary, so this is allowed. const C: &AtomicU8 = { static S: AtomicU8 = AtomicU8::new(0); &S // OK }; }``#![allow(unused)] fn main() { use core::sync::atomic::AtomicU8; // This shared borrow of an interior mutable temporary is allowed // because its scope is not extended. const C: () = { _ = &AtomicU8::new(0); }; // OK }``#![allow(unused)] fn main() { // Even though the borrow is mutable and the temporary lives to the // end of the program due to promotion, this is allowed because the // borrow is not in tail position and so the scope of the temporary // is not extended via temporary lifetime extension. const C: () = { let _: &'static mut [u8] = &mut []; }; // OK // ~~ // Promoted temporary. }`Note In other words — to focus on what’s allowed rather than what’s not allowed — shared borrows of interior mutable data and mutable borrows are only allowed in a const context when the borrowed place expression is *transient*,*indirect*, or*static*.A place expression is *transient*if it is a variable local to the current const context or an expression whose temporary scope is contained inside the current const context.`#![allow(unused)] fn main() { // The borrow is of a variable local to the initializer, therefore // this place expression is transient. const C: () = { let mut x = 0; _ = &mut x; }; }``#![allow(unused)] fn main() { // The borrow is of a temporary whose scope has not been extended, // therefore this place expression is transient. const C: () = { _ = &mut 0u8; }; }``#![allow(unused)] fn main() { // When a temporary is promoted but not lifetime extended, its // place expression is still treated as transient. const C: () = { let _: &'static mut [u8] = &mut []; }; }`A place expression is *indirect*if it is a dereference expression.`#![allow(unused)] fn main() { const C: () = { _ = &mut *(&mut 0); }; }`A place expression is *static*if it is a`static`item.`#![allow(unused)] fn main() { #![allow(static_mut_refs)] const C: &u8 = unsafe { static mut S: u8 = 0; &mut S }; }`Note One surprising consequence of these rules is that we allow this, `#![allow(unused)] fn main() { const C: &[u8] = { let x: &mut [u8] = &mut []; x }; // OK // ~~~~~~~ // Empty arrays are promoted even behind mutable borrows. }`but we disallow this similar code: `#![allow(unused)] fn main() { const C: &[u8] = &mut []; // ERROR // ~~~~~~~ // Tail expression. }`The difference between these is that, in the first, the empty array is promoted but its scope does not undergo temporary lifetime extension, so we consider the place expression to be transient (even though after promotion the place indeed lives to the end of the program). In the second, the scope of the empty array temporary does undergo lifetime extension, and so it is rejected due to being a mutable borrow of a lifetime-extended temporary (and therefore borrowing a non-transient place expression). The effect is surprising because temporary lifetime extension, in this case, causes less code to compile than would without it. See issue #143129 for more details. 

- 
`#![allow(unused)] fn main() { use core::cell::UnsafeCell; const _: u8 = unsafe { let x: *mut u8 = &raw mut *&mut 0; // ^^^^^^^ // Dereference of mutable reference. *x = 1; // Dereference of mutable pointer. *(x as *const u8) // Dereference of constant pointer. }; const _: u8 = unsafe { let x = &UnsafeCell::new(0); *x.get() = 1; // Mutation of interior mutable value. *x.get() }; }`

- Grouped expressions.

- Cast expressions, except
- pointer to address casts and
- function pointer to address casts.
 

- Calls of const functions and const methods.

## Const context

A *const context* is one of the following:

- The initializer of

Array type length expressions, array repeat length expressions, and const generic arguments are restricted in their use of outer generic parameters: such an expression must either be a single const generic parameter, or an expression that does not reference any generic parameters.

## Const functions

A *const function* is a function that can be called from a const context. It is defined with the `const` qualifier, and also includes tuple struct and tuple enum variant constructors.

Example

`#![allow(unused)] fn main() { const fn square(x: i32) -> i32 { x * x } const VALUE: i32 = square(12); }`

When called from a const context, a const function is interpreted by the compiler at compile time. The interpretation happens in the environment of the compilation target and not the host. So `usize` is `32` bits if you are compiling against a `32` bit system, irrelevant of whether you are building on a `64` bit or a `32` bit system.

When a const function is called from outside a const context, it behaves the same as if it did not have the `const` qualifier.

The body of a const function may only use constant expressions.

Const functions are not allowed to be async.

The types of a const function’s parameters and return type are restricted to those that are compatible with a const context.

# Citations

1. Source page: https://doc.rust-lang.org/stable/reference/const_eval.html
