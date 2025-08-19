---
title: "Unsafe Rust Basics - When and Why to Use Unsafe"
description: "Learn scenarios where unsafe Rust is necessary, and the trade-offs involved."
date: 2025-10-13
weight: 1
---

# Unsafe Rust Basics: When and Why to Use It

Understanding `unsafe` Rust is like learning about the **emergency override panel in a high-tech, automated kitchen**. 99% of the time, the kitchen's safety systems (the Rust compiler and borrow checker) work perfectly, preventing spills, collisions, and other accidents. However, there are rare, specific situations where a master chef needs to manually override these safeties to perform a complex technique that the automated system doesn't understand. `unsafe` is that override panel: it's powerful, occasionally necessary, but must be used with extreme care and expertise.

## The Kitchen Override Panel Analogy 🍽️

- **Safe Rust**: The fully automated kitchen. The borrow checker is the system that ensures chefs never try to use the same knife at the same time, preventing accidents.
- **The `unsafe` block**: The big red button under a glass cover. When you press it, you tell the system, "I am disabling the automatic collision avoidance. I am a trained professional, and I will take personal responsibility for ensuring that what I'm about to do is safe."
- **An `unsafe` operation**: A specific, "dangerous" action that is only possible with the safeties disabled, like manually operating a high-speed slicer or working with raw, uninspected ingredients from an outside supplier.

***

## Why Does `unsafe` Rust Even Exist?

Rust's primary goal is to guarantee memory safety *at compile time*. However, this static analysis is inherently conservative. There are some valid programs that the compiler cannot *prove* are safe, so it rejects them. `unsafe` provides a way for the programmer to vouch for the code's safety in these specific cases.[^1]

There are two main reasons `unsafe` is necessary:

1. **To Do Things the Compiler Can't Understand**: Sometimes, you have more knowledge about the code's invariants than the compiler does. A classic example is implementing a data structure like `Vec<T>`, which internally uses a raw pointer that it manages manually. The compiler can't verify that this pointer management is correct, so the implementation of `Vec<T>` uses `unsafe` blocks to handle it, while still exposing a completely safe public API.
2. **To Interact with the "Unsafe" World**: The world outside of Rust—like the operating system, hardware, or code written in other languages like C—does not follow Rust's safety rules. To interact with these systems, you must use `unsafe` to bridge the gap.[^1]

***

## What `unsafe` Allows You to Do: The Five "Superpowers"

Entering an `unsafe` block gives you access to five new capabilities that are normally forbidden by the compiler.[^6][^1]


| **Superpower** | **Kitchen Analogy** | **Description** |
| :-- | :-- | :-- |
| **1. Dereference a raw pointer** | Handling a raw, uninspected ingredient from an unknown supplier. | You can turn a raw pointer (`*const T` or `*mut T`) into a reference (`&T` or `&mut T`), but you are responsible for ensuring the pointer is valid. |
| **2. Call an `unsafe` function or method** | Following a special, "dangerous" recipe from a master chef. | You can call functions marked `unsafe`, but you must uphold the safety contract documented by that function's author. |
| **3. Access or modify a mutable `static` variable** | Changing a global setting for the entire kitchen while it's running. | You can read or write to a global `static mut` variable, which is inherently unsafe due to potential data races. |
| **4. Implement an `unsafe` trait** | Certifying that your new custom kitchen appliance follows safety standards. | You can implement traits marked `unsafe` (like `Send` and `Sync`), promising the compiler that your type upholds the trait's safety guarantees. |
| **5. Access fields of a `union`** | Using a multi-purpose tool that can be either a whisk or a knife. | You can access the fields of a `union`, where the compiler cannot know which field is currently active and valid. |

**Crucial Point**: `unsafe` does **not** turn off the borrow checker. If you create a reference inside an `unsafe` block, it is still subject to all of Rust's normal lifetime and borrowing rules. `unsafe` only gives you access to these five specific, otherwise-forbidden actions.[^2]

***

## When to Use `unsafe`: The Three Main Scenarios

As a beginner, you will rarely need to write `unsafe` code. It is primarily for library authors and systems programmers. However, it's essential to understand *why* it's used.

### Scenario 1: Interfacing with Foreign Code (FFI)

This is the most common and straightforward use of `unsafe`. When calling a function from a C library, Rust cannot guarantee that the C code is memory-safe. Therefore, all calls through a Foreign Function Interface (FFI) must be wrapped in an `unsafe` block.

**Example: Calling a C Function**

Imagine we are using a C library that has a function `double_input(int x)`.

**C Code:**

```c
// a_c_library.h
int double_input(int x);
```

**Rust Code:**

```rust
// Link to the C library
#[link(name = "a_c_library")]
extern "C" {
    // Declare the C function's signature.
    // It's marked `unsafe` because Rust can't verify its implementation.
    fn double_input(x: i32) -> i32;
}

fn main() {
    let input = 5;
    let result;

    // We must use an `unsafe` block to call the foreign function.
    // We are telling the compiler: "I trust that this C function is safe to call."
    unsafe {
        result = double_input(input);
    }

    println!("The result from the C library is: {}", result);
}
```


### Scenario 2: High-Performance Code with Raw Pointers

When you need to squeeze out every last drop of performance, you might need to bypass Rust's abstractions and work with raw pointers. This is common in low-level data structures.

**Example: A Simplified Version of `Vec::split_at_mut`**
The standard library's `split_at_mut` function takes one mutable slice and returns two mutable slices.

```rust
fn split_at_mut<'a, T>(slice: &'a mut [T], mid: usize) -> (&'a mut [T], &'a mut [T]) {
    // ...
}
```

If you tried to implement this in safe Rust, the borrow checker would stop you, because you cannot have two mutable references to the same slice at the same time. However, we *know* that the two resulting slices will point to different, non-overlapping parts of the original memory. `unsafe` allows us to express this.

```rust
use std::slice;

fn my_split_at_mut<T>(slice: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr(); // Get a raw mutable pointer to the start of the data.

    assert!(mid <= len);

    // We use an unsafe block because we are creating slices from a raw pointer.
    // We are guaranteeing to the compiler that these two slices will not overlap.
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5, 6];
    let (left, right) = my_split_at_mut(&mut numbers, 3);

    println!("Left side: {:?}", left);   // [1, 2, 3]
    println!("Right side: {:?}", right); // [4, 5, 6]
}
```


### Scenario 3: Interacting with Hardware

When writing drivers or operating system kernels, you often need to read from or write to specific, fixed memory addresses that correspond to hardware devices. Safe Rust has no concept of this, so you must use `unsafe` to work with these memory-mapped registers.

## Best Practices: The `unsafe` Code of Conduct

If you must use `unsafe`, follow these rules to minimize risk:

1. **Keep `unsafe` Blocks Small**: Your `unsafe` block should be as small as possible, containing only the specific operations that require it. This makes it easy to audit and verify.[^1]
2. **Wrap `unsafe` in a Safe Abstraction**: This is the most important rule. The *implementation* of a function might use `unsafe` internally, but it should present a *safe* public API to its users. The `my_split_at_mut` example does exactly this. The function itself is safe to call, because the `unsafe` logic has been encapsulated and verified by the author.[^6]
3. **Document Why It's Safe**: Add detailed comments to every `unsafe` block explaining *why* the code is sound. What are the invariants you are upholding that the compiler can't see? What are the contracts of the unsafe functions you are calling?

By understanding that `unsafe` is a carefully controlled tool for specific, expert-level tasks—not a general-purpose "off switch" for safety—you can appreciate its role in making Rust a language that is both incredibly safe and powerfully low-level.

1. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
2. https://geo-ant.github.io/blog/2023/unsafe-rust-exploration/
3. https://users.rust-lang.org/t/learn-rust-the-dangerous-way-the-unsafe-first-tutorial/35806
4. https://www.youtube.com/watch?v=-l0W_T8taZA
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/unsafe.html
6. https://www.youtube.com/watch?v=9E2v8pCUc48
7. https://www.reddit.com/r/rust/comments/1cuvyyg/unsafe_code_in_rust/
8. https://users.rust-lang.org/t/learn-rust-the-dangerous-way-the-unsafe-first-tutorial/35806?page=2
9. https://www.reddit.com/r/rust/comments/14yaheu/unsafe_rust_advice_for_beginner/
