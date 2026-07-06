---
type: Web Page
title: Implementing Vec - The Rustonomicon
description: The Dark Arts of Advanced and Unsafe Rust Programming
resource: https://doc.rust-lang.org/stable/nomicon/vec/vec.html
timestamp: '2026-07-06T08:05:31.028609+00:00'
---

# Example: Implementing Vec

To bring everything together, we’re going to write `std::Vec` from scratch.
We will limit ourselves to stable Rust. In particular we won’t use any
intrinsics that could make our code a little bit nicer or efficient because
intrinsics are permanently unstable. Although many intrinsics *do* become
stabilized elsewhere (`std::ptr` and `std::mem` consist of many intrinsics).

Ultimately this means our implementation may not take advantage of all
possible optimizations, though it will be by no means *naive*. We will
definitely get into the weeds over nitty-gritty details, even
when the problem doesn’t *really* merit it.

You wanted advanced. We’re gonna go advanced.

# Citations

1. Source page: https://doc.rust-lang.org/stable/nomicon/vec/vec.html
