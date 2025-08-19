---
title: "Unsafe Rust Basics - Memory Safety Invariants"
description: "Discover memory safety invariants and how unsafe Rust relies on developers to uphold them."
date: 2025-10-13
weight: 5
---

# Unsafe Rust Basics: Understanding Memory Safety Invariants

To understand `unsafe` Rust, you must first appreciate what "safe" Rust guarantees. Safe Rust is like a **professional, high-security kitchen with a rigorous inspection system**. Every ingredient (data), every tool (type), and every action (operation) is constantly checked by an inspector (the borrow checker) to ensure a set of strict safety rules are followed. This guarantees that certain classes of dangerous mistakes—like using spoiled ingredients (dangling pointers) or two chefs trying to use the same knife at once (data races)—are impossible. These fundamental rules are Rust's **memory safety invariants**.

## The High-Security Kitchen Analogy: Safety Invariants 🍽️

Imagine the core safety rules (invariants) of our high-security kitchen:

1. **Valid Ingredients Only (No Dangling Pointers)**: Any ingredient you use must be valid and exist. You can't use an ingredient that has already been thrown away. In Rust, this means a reference must always point to valid, allocated memory.[^1]
2. **One Chef per Cutting Board (No Data Races)**: You can have many chefs *looking* at a recipe (`&T`, a shared reference), but only **one** chef can be actively *modifying* it at a time (`&mut T`, a mutable reference). This "Aliasing XOR Mutability" rule is the cornerstone of Rust's concurrency story.[^5]
3. **Properly Labeled Ingredients (Valid Values)**: An ingredient must be what its label says it is. A jar labeled "Salt" must contain salt, not sugar. In Rust, a `bool` must be either `true` or `false` (1 or 0), not the bit pattern for the number `3`. A non-null type like `Box<T>` must never be null.[^4][^5]
4. **Ingredients Are Used Correctly (Ownership and Lifetimes)**: You can't use an ingredient after you've given it to another chef (move semantics). You can't borrow an ingredient for longer than the chef who owns it is working (lifetimes).[^2]

Safe Rust **compiles** these rules into your program. The borrow checker is the inspector that enforces them at compile time, guaranteeing that if your code compiles, it upholds these invariants.

## Introducing `unsafe`: Entering the "No-Inspection Zone"

The `unsafe` keyword in Rust is like a special, key-card access door in the kitchen that leads to an "experimental prep area."

- **Inside this area, the automated inspection system is turned off.**
- The head chef (you, the programmer) can perform powerful, low-level operations that are normally forbidden, like using a custom-made, untested knife (dereferencing a raw pointer) or mixing ingredients from outside the kitchen (calling C functions).

Crucially, the `unsafe` keyword does not "turn off" the borrow checker entirely; it only gives you access to a few extra "superpowers" that are not available in safe Rust.[^7]

**The Five Superpowers of `unsafe` Rust:**

1. Dereferencing a raw pointer (`*const T` or `*mut T`).
2. Calling an `unsafe` function or method (including FFI).
3. Accessing or modifying a mutable `static` variable.
4. Implementing an `unsafe` trait.
5. Accessing fields of a `union`.

## The `unsafe` Contract: The Developer's Promise

When you open that `unsafe` door and step into the experimental area, you are making a solemn promise to the rest of the kitchen (the Rust compiler and all the safe code that will call your code):

> "I, the programmer, have manually inspected this small, specific block of code. I personally guarantee that even though I am using dangerous tools, the result of my work will not violate any of the kitchen's fundamental safety invariants when it is used by the rest of the safe code."

This is the core contract of `unsafe` Rust. You are telling the compiler, "Trust me. I know what I'm doing here, and I will uphold the rules myself". If you break this promise, you can introduce **Undefined Behavior (UB)**—the very class of bugs Rust was designed to prevent.[^3]

## Examples of Upholding Invariants in `unsafe` Code

Let's look at how this works in practice.

### Example 1: Dereferencing Raw Pointers Safely

Imagine you need to split a mutable slice into two halves. The borrow checker won't let you do this safely because you can't have two mutable borrows of the same slice at once. We can use `unsafe` to bypass this check, but we must promise that our two new slices don't overlap.

```rust
use std::slice;

/// A safe function that uses an `unsafe` block internally.
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr(); // Get a raw pointer to the slice's data

    assert!(mid <= len, "mid must be less than or equal to len");

    // We are now entering the "No-Inspection Zone".
    unsafe {
        // SAFETY: We are making a promise here.
        // 1. We have asserted that `mid <= len`, so the pointers we create
        //    will be within the bounds of the original slice.
        // 2. The first slice is from the start to `mid`, and the second is
        //    from `mid` to the end. They are guaranteed not to overlap.
        // Therefore, creating two separate mutable slices from these
        // non-overlapping parts is safe.
        let left = slice::from_raw_parts_mut(ptr, mid);
        let right = slice::from_raw_parts_mut(ptr.add(mid), len - mid);
        (left, right)
    }
}

fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5, 6];
    let (left, right) = split_at_mut(&mut numbers, 3);

    left = 10;
    right = 40;

    println!("Left: {:?}", left);   // Output: [10, 2, 3]
    println!("Right: {:?}", right); // Output: [40, 5, 6]
}
```

In this example, the `unsafe` block is small and contained. The function `split_at_mut` presents a safe interface to the outside world. The `SAFETY` comment is a crucial convention explaining *why* the `unsafe` block is correct and upholds Rust's invariants.

### Example 2: Interfacing with a C Library (FFI)

When you call a C function, Rust cannot guarantee anything about what that function does. It could have bugs, access invalid memory, or violate any number of invariants. Therefore, all FFI calls are inherently `unsafe`.

```rust
// A C library might expose this function.
// extern "C" {
//     fn abs(input: i32) -> i32;
// }

// In Rust, we declare the external function signature.
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    let x = -5;

    // We must wrap the call in an `unsafe` block.
    unsafe {
        // SAFETY: We are promising that the external `abs` function from the C
        // standard library is safe to call with an i32 and that it will
        // correctly return an i32 without causing any side effects that
        // would violate Rust's memory safety.
        println!("The absolute value of {} is {}", x, abs(x));
    }
}
```

Here, our `SAFETY` promise is based on our external knowledge of the C standard library. We trust that the `abs` function is well-behaved. If we were calling a less-trusted C library, we would need to be much more careful.

## Why `unsafe` is Necessary

If `unsafe` is so dangerous, why does it exist?

1. **Low-Level Control**: Sometimes, you need to perform operations that are provably safe to a human but that the borrow checker is too conservative to allow. This is common when implementing low-level data structures like `Vec` or `BTreeMap`.
2. **Performance**: For extremely performance-critical code, dropping into `unsafe` can allow for optimizations that the compiler can't make on its own (though this is rare and should be a last resort).
3. **Hardware and OS Interaction**: Talking to hardware or operating system APIs often requires raw pointer manipulation.
4. **Foreign Function Interface (FFI)**: As seen above, interacting with libraries written in other languages (like C) is inherently `unsafe`.

The goal of `unsafe` Rust is not to abandon safety, but to **confine the danger**. By requiring the `unsafe` keyword, Rust forces you to explicitly mark the exact parts of your code where you are making manual safety guarantees. This makes auditing for potential memory safety bugs much easier—you only need to scrutinize the code inside the `unsafe` blocks.

1. https://stanford-cs242.github.io/f18/lectures/05-1-rust-memory-safety.html
2. https://www.reddit.com/r/rust/comments/131knig/how_does_rust_achieve_complete_memory_safety_as/
3. https://internals.rust-lang.org/t/two-kinds-of-invariants-safety-and-validity/8264
4. https://www.ralfj.de/blog/2018/08/22/two-kinds-of-invariants.html
5. https://www.usenix.org/publications/loginonline/memory-safety-merely-table-stakes
6. https://www.micahlerner.com/2021/10/31/rudra-finding-memory-safety-bugs-in-rust-at-the-ecosystem-scale.html
7. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
8. https://rustfoundation.org/media/unsafe-rust-in-the-wild-notes-on-the-current-state-of-unsafe-rust/
