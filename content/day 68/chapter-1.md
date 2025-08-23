---
title: "Memory Layout Optimization in Rust"
description: "Learn how to structure data for optimal memory usage and performance in Rust."
date: 2025-11-12
weight: 1
---

# Memory Layout Optimization in Rust: A Beginner's Guide

Understanding how to structure your data in memory is a key skill for writing high-performance Rust code. While the Rust compiler is incredibly smart, thinking about memory layout can unlock significant speed improvements, especially in performance-critical applications. Think of it like **organizing a professional mechanic's toolbox**. A well-organized toolbox allows the mechanic (the CPU) to find and use tools (data) much faster, leading to a more efficient workflow.

## The Mechanic's Toolbox Analogy: Why Layout Matters 🛠️

- **Your Data (`struct`s)**: The tools in your toolbox.
- **Memory (RAM)**: The toolbox itself.
- **The CPU**: The mechanic who needs to grab tools.
- **The CPU Cache**: A small, super-fast tray on the mechanic's workbench. Grabbing a tool from this tray is instant. Grabbing one from the main toolbox is much slower.

The goal is to organize the toolbox (memory) so that when the mechanic (CPU) needs one tool, the other tools they'll need next are pulled onto the fast tray (cache) at the same time. This is called **cache-friendliness**. A messy, disorganized layout leads to poor cache performance, forcing the CPU to constantly make slow trips back to main memory.

## The Default Behavior: What Rust Does Automatically

By default, the Rust compiler is a very smart assistant. If you define a struct, it will automatically reorder the fields to try to make the struct as small as possible by minimizing wasted space.[^1][^2]

```rust
struct MyStruct {
    a: u8,   // 1 byte
    b: u64,  // 8 bytes
    c: u32,  // 4 bytes
}
```

You might expect this to be laid out in the order `a`, `b`, `c`. However, Rust is free to reorder it to something like `b, c, a` to pack it more tightly in memory. For most cases, this is exactly what you want!

## The Core Concepts: Alignment and Padding

To understand *why* layout matters, you need to know about two key concepts:

1. **Alignment**: This is a rule that says a piece of data of a certain size must be stored at a memory address that is a multiple of its size (or a power of two). For example, a `u64` (8 bytes) must start at a memory address that is divisible by 8.[^2][^3]
    - **Analogy**: A large wrench must sit in a slot that starts at a 10cm mark in the drawer; it can't start at the 13cm mark.
2. **Padding**: To satisfy alignment rules, the compiler sometimes has to insert empty, unused bytes between fields. This is called padding.[^4]
    - **Analogy**: If you place a small screwdriver (1 byte) next to a large wrench (8 bytes) that needs 8-byte alignment, you might have to leave 7cm of empty space after the screwdriver to ensure the wrench starts in the right spot. This empty space is padding.

### Example: The Cost of Poor Padding

Let's force Rust to lay out a struct in a specific, suboptimal order using `#[repr(C)]`.

```rust
use std::mem::{size_of, align_of};

// We use #[repr(C)] to tell Rust to NOT reorder fields.
// This guarantees a C-like, sequential layout.

// Suboptimal ordering
#[repr(C)]
struct Unoptimized {
    a: u8,   // 1 byte
    // 7 bytes of padding will be inserted here by the compiler
    // to ensure `b` starts on an 8-byte boundary.
    b: u64,  // 8 bytes
    c: u16,  // 2 bytes
    // 6 bytes of padding at the end to make the total size a multiple of the
    // struct's alignment (which is 8, because of `b`).
}

// Optimal ordering
#[repr(C)]
struct Optimized {
    b: u64,  // 8 bytes
    c: u16,  // 2 bytes
    a: u8,   // 1 byte
    // 5 bytes of padding at the end.
}

fn main() {
    println!("Size of Unoptimized: {} bytes", size_of::<Unoptimized>()); // Output: 24 bytes
    println!("Size of Optimized:   {} bytes", size_of::<Optimized>());   // Output: 16 bytes
}
```

By simply reordering the fields from **largest to smallest**, we saved 8 bytes of memory! This is because we minimized the amount of internal padding the compiler needed to add. For an array of a million of these structs, that's a saving of 8 megabytes.

## Taking Control: The `#[repr]` Attribute

While Rust's default layout is great, sometimes you need explicit control. The `#[repr]` attribute lets you dictate the memory layout strategy.[^5]

### 1. `#[repr(C)]`

This is the most common and useful layout attribute. It guarantees that the fields are laid out in the order you declare them, with C-compatible padding rules.

- **When to Use It**:
    - **FFI (Foreign Function Interface)**: Absolutely essential when passing structs to and from C/C++ code, as both sides must agree on the layout.
    - **Predictable Layout**: When you need to cast raw bytes to your struct type or perform other low-level memory manipulations.
- **Best Practice**: When using `#[repr(C)]`, **always manually order your fields from largest to smallest** to minimize padding.


### 2. `#[repr(packed)]`

This is a dangerous but powerful attribute. It tells the compiler to remove **all** padding.

- **When to Use It**:
    - When you need to match a specific memory layout defined by a network protocol or hardware device, where no padding is present.
    - When saving every single byte is critical.
- **The Danger**: Using `#[repr(packed)]` can create **unaligned fields**. Accessing an unaligned field can be very slow or, on some architectures, can cause a program crash. In Rust, taking a reference (`&`) to an unaligned field is **instant Undefined Behavior**. You must use safe pointer reads (`read_unaligned`/`write_unaligned`) to access the fields, which requires `unsafe` code.[^6]
- **Analogy**: This is like cramming all your tools into a bag with no organizers. It saves space, but trying to pull out a specific tool is slow and you might cut yourself.


### 3. `#[repr(align(N))]`

This attribute lets you specify a *minimum* alignment for your struct, where `N` must be a power of two.

- **When to Use It**:
    - **SIMD (Single Instruction, Multiple Data)**: SIMD operations often require data to be aligned to 16, 32, or even 64-byte boundaries for maximum performance.
    - **Interacting with specific hardware** that has strict alignment requirements.


## Practical Optimization Strategies

1. **Let Rust Do the Work**: For most internal data structures, trust Rust's default layout (`#[repr(Rust)]`). It's smart and will usually produce the most compact layout.
2. **Order Fields for `#[repr(C)]`**: If you need a stable layout, use `#[repr(C)]` and manually order fields from largest to smallest.
3. **Group Small Fields**: If you have multiple small fields (like `u8` or `bool`), group them together. This allows the compiler to pack them into the padding left over from larger fields.

```rust
#[repr(C)]
struct Good {
    big: u64,
    small1: bool,
    small2: u8,
    small3: u8,
} // Likely 16 bytes
```

4. **Use `std::mem` to Verify**: Don't guess! Use `size_of::<T>()` and `align_of::<T>()` to check the actual size and alignment of your structs. This will help you understand the impact of your changes.
5. **Think about Cache Lines (Data-Oriented Design)**: This is a more advanced topic, but a key to high performance is organizing data so that the information you need together is also close together in memory. This often means preferring a "Struct of Arrays" (SoA) over an "Array of Structs" (AoS) for hot loops.

By paying attention to how your data is structured, you can write Rust code that is not only safe and correct but also incredibly performant, making the most efficient use of the underlying hardware.

1. https://www.youtube.com/watch?v=2vN_KGL4Bco
2. https://brainhive.nl/blog/memory-alignment
3. https://users.rust-lang.org/t/type-alignment-understanding-memory-layout/126503
4. https://dev.to/gritmax/memory-allocations-in-rust-3m7l
5. https://www.linkedin.com/pulse/rust-memory-layouts-inpractice-luis-soares-m-sc--zcfwf
6. https://webreference.com/rust/ownership/memory-management/
7. https://www.rapidinnovation.io/post/performance-optimization-techniques-in-rust
8. https://users.rust-lang.org/t/memory-layout-optimization-opportunity/107522
9. https://gendignoux.com/blog/2024/12/02/rust-data-oriented-design.html
10. https://dev.to/chetanmittaldev/10-best-ways-to-optimize-code-performance-using-rusts-memory-management-33jl
11. https://www.youtube.com/watch?v=-6cnnNlAvNk
