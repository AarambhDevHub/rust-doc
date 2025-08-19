---
title: "Unsafe Rust Basics - Unsafe Functions"
description: "Explore how to define and call unsafe functions in Rust."
date: 2025-10-13
weight: 3
---

# Unsafe Rust Basics: Understanding Unsafe Functions

To understand `unsafe` Rust, you must first embrace Rust's core promise: **safety**. The compiler acts as a strict, ever-watchful guardian, preventing entire classes of bugs like null pointer dereferences, data races, and buffer overflows. However, sometimes you need to perform tasks that the compiler cannot possibly verify as safe. This is where `unsafe` comes in.

Using `unsafe` is like telling your guardian, "I know you can't verify this next action, but I am an expert, I have read the manual, and I take full responsibility for ensuring this is correct." It's a tool that gives you "superpowers" but requires you to uphold the safety guarantees manually.[^1]

## The Nuclear Reactor Analogy ☢️

### Imagine You're an Engineer in a Nuclear Power Plant

- **Safe Rust**: The **control room**. It's filled with automated checks, safety interlocks, and guardrails. It's designed to be impossible to cause a meltdown from here. The system prevents you from making catastrophic mistakes.
- **The `unsafe` Keyword**: The key that unlocks the door to the **reactor core**.
- **An `unsafe` Block/Function**: You, wearing a special suit, physically inside the reactor core. The automated safeties are off. You have the power to manually override systems, but you are also responsible for following the safety protocols precisely. One wrong move could lead to a meltdown (Undefined Behavior).

The key takeaway is that `unsafe` does not turn off all of Rust's rules. The borrow checker still works, for example. It only gives you access to a few "superpowers" that are otherwise forbidden.[^2][^1]

## The Five Unsafe Superpowers

The `unsafe` keyword allows you to perform five actions that are normally disallowed :[^1]

1. Dereference a raw pointer.
2. **Call an unsafe function or method.**
3. Access or modify a mutable static variable.
4. Implement an unsafe trait.
5. Access fields of a `union`.

This guide focuses on the second superpower: **defining and calling unsafe functions**.

## What is an Unsafe Function?

An `unsafe fn` is a function whose safety cannot be guaranteed by the Rust compiler. By marking a function as `unsafe`, you are placing a requirement on the *caller* of the function. You are saying: "To call this function safely, you must uphold certain conditions (invariants) that I will describe in my documentation. The compiler can't check these for you."

### How to Define an Unsafe Function

Defining an `unsafe` function is simple: you just add the `unsafe` keyword before `fn`.

```rust
/// This function is unsafe because it performs a "dangerous" operation
/// that the compiler cannot reason about.
unsafe fn dangerous_operation() {
    println!("Doing something dangerous!");
    // In a real scenario, this would involve one of the other unsafe superpowers,
    // like dereferencing a raw pointer.
}
```

The `unsafe` on the function signature does **not** make the entire body an `unsafe` block. If you want to perform an unsafe operation inside an `unsafe fn`, you still need to use an `unsafe` block. This encourages keeping the actual unsafe part of the code as small as possible.[^9][^1]

```rust
use std::slice;

/// An unsafe function that creates a slice from a raw pointer.
///
/// # SAFETY
/// The caller MUST ensure that the `ptr` points to a valid block of memory
/// of `T` that is at least `len` elements long, and that the memory remains
/// valid for the lifetime of the returned slice.
unsafe fn get_slice_from_raw_parts<'a, T>(ptr: *const T, len: usize) -> &'a [T] {
    // We still need an unsafe block to call `from_raw_parts`,
    // which is itself an unsafe function.
    unsafe {
        slice::from_raw_parts(ptr, len)
    }
}
```


### The `SAFETY` Documentation Contract

This is the most critical part of writing `unsafe` functions. You **must** document the requirements that the caller needs to fulfill to use the function correctly. This is conventionally done in a `SAFETY` block in the doc comments. This documentation is the "manual" that the caller promises they have read.[^7][^1]

## How to Call an Unsafe Function

To call a function marked as `unsafe`, you must wrap the call in an `unsafe` block.

```rust
fn main() {
    // This will not compile!
    // dangerous_operation();
    // error[E0133]: call to unsafe function `dangerous_operation` is unsafe and requires unsafe block

    // This is how you do it correctly:
    unsafe {
        dangerous_operation();
    }
}

unsafe fn dangerous_operation() {
    println!("Danger zone!");
}
```

By placing the call inside an `unsafe` block, you are making an assertion to the compiler: "I have read the `SAFETY` documentation for this function, and I guarantee that I am meeting all of its preconditions."

## A Practical Example: Interfacing with a C Library (FFI)

The most common reason to use `unsafe` functions is for Foreign Function Interface (FFI)—calling functions from libraries written in other languages, like C. The Rust compiler has no way to know what a C function does, so all calls to C functions must be marked `unsafe`.[^3][^6]

Let's imagine we are calling the `abs` function from the C standard library.

```rust
// Link to the C standard library (libc)
#[link(name = "c")]
extern "C" {
    // This declares a function signature from an external C library.
    // It's `unsafe` because Rust can't verify what it does.
    fn abs(input: i32) -> i32;
}

fn main() {
    let number = -10;
    let absolute_value;

    // We must wrap the call to the C function in an `unsafe` block.
    unsafe {
        absolute_value = abs(number);
    }

    println!("The absolute value of {} is {}", number, absolute_value);
}
```

Here, the call to `abs` is considered unsafe because the Rust compiler cannot analyze the C code to guarantee its safety.

## Creating a Safe Abstraction Around Unsafe Code

The ultimate goal when using `unsafe` is not to sprinkle `unsafe` blocks throughout your application. Instead, the best practice is to **build a safe abstraction around the unsafe code**.

You create a module or a type that uses `unsafe` internally to implement its logic, but exposes a completely safe public API. The internal `unsafe` code is carefully written to uphold all the necessary invariants, so that anyone using the safe public API cannot cause undefined behavior. `Vec<T>` is a perfect example of this: internally, it uses `unsafe` code to manage raw pointers and memory allocation, but the public API it provides (`push`, `pop`, `get`, etc.) is completely safe.[^6]

**Example: A Safe Wrapper Around Our FFI `abs` call**

```rust
#[link(name = "c")]
extern "C" {
    fn abs(input: i32) -> i32;
}

/// A safe Rust function that wraps the unsafe C `abs` function.
fn safe_abs(input: i32) -> i32 {
    // The unsafe operation is contained within this function.
    // We know this specific call is safe because `abs` in C
    // has well-defined behavior for any i32 input.
    unsafe {
        abs(input)
    }
}

fn main() {
    // Now, anyone can call `safe_abs` without needing an `unsafe` block.
    let absolute_value = safe_abs(-20);
    println!("Safely calculated absolute value: {}", absolute_value);
}
```

This is the core principle of `unsafe` Rust: **limit the `unsafe` code to the smallest possible area, audit it carefully, and expose a safe, ergonomic API to the rest of your program.** This gives you the power of low-level programming when you need it, without sacrificing the safety and confidence that Rust provides everywhere else.

1. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
2. https://geo-ant.github.io/blog/2023/unsafe-rust-exploration/
3. https://www.youtube.com/watch?v=-l0W_T8taZA
4. https://users.rust-lang.org/t/learn-rust-the-dangerous-way-the-unsafe-first-tutorial/35806
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/unsafe.html
6. https://www.youtube.com/watch?v=9E2v8pCUc48
7. https://www.apriorit.com/dev-blog/interoperability-unsafe-rust
8. https://codedamn.com/news/rust/understanding-unsafe-rust-guide-working-with-raw-pointers
9. https://stackoverflow.com/questions/57522200/what-are-best-practices-for-unsafe-functions-in-which-only-a-small-part-of-the
