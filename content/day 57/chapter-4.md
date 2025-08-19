---
title: "Advanced Unsafe Operations - Inline Assembly"
description: "Discover how to use inline assembly in Rust for system-level programming and optimization."
date: 2025-10-14
weight: 4
---

# Advanced Unsafe Operations: An Introduction to Inline Assembly in Rust

Using inline assembly in Rust is like a master chef deciding to **forge their own, custom kitchen knife** for a very specific task. While the standard set of knives (the Rust compiler and its optimizations) is excellent for 99.9% of jobs, there are rare occasions where you need absolute, low-level control over the hardware that only hand-forged assembly code can provide. This is a powerful, dangerous, and highly specialized tool used in system-level programming and for ultimate performance optimization.

## The Custom-Forged Kitchen Knife Analogy 🔪

### Imagine You're a Chef with an Extremely Demanding Task

- **Standard Rust Code**: A set of high-quality, professionally made chef's knives. They are safe, reliable, and incredibly effective for almost every task.
- **The Rust Compiler (LLVM)**: A master sharpener who knows how to optimize your knife techniques for maximum efficiency.
- **Inline Assembly (`asm!`)**: A forge in the back of the kitchen where you can take raw steel and hammer out your own blade. You can create a knife with a specific curve or weight that no manufacturer provides, but you are also entirely responsible for its safety and effectiveness. If you make a mistake, the knife could break or cause serious injury.

***

### Why and When Would You Use Inline Assembly?

You should only reach for inline assembly when there is **no other way** to accomplish your goal. The Rust compiler's optimizer (LLVM) is exceptionally good, and it's very difficult to write assembly by hand that is faster than what the compiler can generate.

However, there are a few valid use cases :[^1][^8]

1. **Accessing CPU-Specific Instructions**: Some CPUs have special instructions that are not exposed through Rust's standard library. Examples include `rdtsc` (to read the timestamp counter for high-precision timing), `popcnt` (to count set bits), or advanced vector extensions (SIMD) that are not yet available as intrinsics.[^5]
2. **Interacting Directly with the Operating System**: Making raw system calls (syscalls) by setting up registers and triggering a software interrupt. This is common in low-level kernel development or when building specialized runtime environments.[^10]
3. **Hardware Interaction in Embedded Systems**: Directly manipulating hardware registers or I/O ports on a microcontroller where performance and timing are critical.
4. **Security and Cryptography**: Implementing cryptographic algorithms where you need to guarantee that operations are performed in a specific way to prevent side-channel attacks.

**Important**: All inline assembly must be placed inside an `unsafe` block. You are bypassing all of Rust's safety guarantees and telling the compiler, "I am taking full responsibility for the correctness of these machine instructions."[^8]

## The `asm!` Macro: Rust's Gateway to Assembly

Rust provides inline assembly through the `std::arch::asm!` macro (or `core::arch::asm!` for `no_std` environments).[^2][^1]

### Basic Syntax

The `asm!` macro has a flexible structure that allows you to specify the assembly code, inputs, outputs, and other options.

```rust
use std::arch::asm;

unsafe {
    asm!(
        "assembly template string(s)",
        in(<register_spec>) <rust_variable_in>,
        out(<register_spec>) <rust_variable_out>,
        inout(<register_spec>) <rust_variable_inout>,
        // ... more operands
        options(<option_flags>),
    );
}
```

- **Assembly Template**: A string literal containing the assembly instructions. You can have multiple strings, which are treated as separate lines.[^1]
- **Operands (`in`, `out`, `inout`)**: These declare how Rust variables are passed to and from the assembly code. They specify a register (`reg`), a constant (`const`), or a memory location (`sym`).
- **Options**: Flags that tell the compiler about the side effects of your assembly code (e.g., `nostack`, `preserves_flags`).


## A Step-by-Step Example: Adding Two Numbers

Let's start with a simple example to see how the pieces fit together. We will add two numbers using x86-64 assembly.

```rust
use std::arch::asm;

fn add_with_asm(a: u64, b: u64) -> u64 {
    let mut sum: u64; // The Rust variable to hold the output

    // Inline assembly is always unsafe
    unsafe {
        asm!(
            // 1. Assembly instructions as string literals
            "mov rax, {a}",   // Move the value of `a` into the `rax` register
            "add rax, {b}",   // Add the value of `b` to the `rax` register
            "mov {sum}, rax", // Move the result from `rax` into our output variable

            // 2. Operand declarations
            a = in(reg) a,        // Pass the Rust variable `a` in any general-purpose register
            b = in(reg) b,        // Pass the Rust variable `b` in any general-purpose register
            sum = out(reg) sum,   // The result will be in a register and then moved to `sum`

            // We can also specify which registers are "clobbered" (modified)
            // but the compiler is smart enough to infer this in many cases.
            // We are using rax, so we could explicitly state:
            // out("rax") _,
        );
    }
    sum
}

fn main() {
    let result = add_with_asm(10, 20);
    println!("10 + 20 = {}", result); // Prints "10 + 20 = 30"
}
```

**Simplified Version using `inout`**:
The compiler is smart. We can make this more concise by telling it to use the same register for an input and the output.

```rust
use std::arch::asm;

fn add_with_asm_concise(a: u64, b: u64) -> u64 {
    let mut result = a; // Start with `a` in our result variable
    unsafe {
        asm!(
            // "add the value of `b` to the register holding `result`"
            "add {0}, {1}",
            inout(reg) result, // `result` is both an input and an output
            in(reg) b,
        );
    }
    result
}

fn main() {
    let result = add_with_asm_concise(10, 20);
    println!("10 + 20 = {}", result);
}
```

In this version, `result` starts with the value of `a`. The compiler places it in a register. The `add` instruction adds `b` to that same register, and the final value is then available in the `result` variable.

## Real-World Example: Making a Linux System Call

A more practical example is making a direct system call, which is how programs request services from the operating system kernel. Let's make a `write` syscall on Linux to print "Hello, world!" to the console.

- **Syscall number for `write`**: 1
- **Arguments**:

1. `rdi`: File descriptor (`1` for `stdout`)
2. `rsi`: Pointer to the message buffer
3. `rdx`: Message length

```rust
use std::arch::asm;

fn main() {
    let message = "Hello from inline assembly!\n";
    let message_ptr = message.as_ptr();
    let message_len = message.len();

    unsafe {
        asm!(
            "syscall", // The instruction to trigger a system call

            // System call number goes in `rax`
            in("rax") 1,
            // First argument goes in `rdi`
            in("rdi") 1, // 1 = stdout
            // Second argument goes in `rsi`
            in("rsi") message_ptr,
            // Third argument goes in `rdx`
            in("rdx") message_len,

            // Tell the compiler that the syscall instruction will modify (clobber)
            // certain registers. This is crucial for correctness. `rcx` and `r11`
            // are used by the syscall mechanism itself.
            out("rcx") _,
            out("r11") _,
        );
    }
}
```

When you run this code on a 64-bit Linux system, it will directly ask the kernel to write the string to your terminal, bypassing the standard library's `println!`.

## Best Practices and Dangers

1. **Avoid It If You Can**: This cannot be stressed enough. The compiler is almost always smarter than you. Only use `asm!` if you have profiled your code and have identified a bottleneck that can *only* be solved with a specific instruction.
2. **Read the Documentation**: Inline assembly syntax is complex. The [official Rust Reference](https://doc.rust-lang.org/reference/inline-assembly.html) is your best friend.
3. **Specify Clobbers**: You **must** tell the compiler which registers your assembly code modifies (other than the outputs). If you don't, the compiler might assume a register still holds a value that you've overwritten, leading to subtle and catastrophic bugs.
4. **Use `options(nostack)`**: If you are certain your assembly code does not use the stack, add this option. It can help the compiler generate more efficient code around your `asm!` block.
5. **Wrap It in a Safe Abstraction**: If you must use inline assembly, hide it inside a safe function. Perform all necessary checks in safe Rust code, then call into a small, well-documented `unsafe` block. This minimizes the "blast radius" of any potential mistakes.

Inline assembly is the ultimate tool for low-level control, but it comes with the ultimate responsibility. It's a fascinating and powerful feature that showcases Rust's commitment to being a true systems programming language.
<span style="display:none">[^3][^4][^6][^7][^9]</span>

1. https://doc.rust-lang.org/rust-by-example/unsafe/asm.html
2. https://doc.rust-lang.org/reference/inline-assembly.html
3. https://codedamn.com/news/rust/optimizing-rust-code-inline-assembly-performance-boosts
4. https://github.com/uggla/asm
5. https://rust-dd.com/post/inline-assembly-in-rust
6. https://www.linkedin.com/pulse/inline-assembly-inrust-luis-soares-m-sc--xyvcf
7. https://blog.devgenius.io/inline-assembly-in-rust-22020b44834b
8. https://rust-lang.github.io/rfcs/2873-inline-asm.html
9. https://dev.to/luzero/templating-inline-asm-in-rust-37ee
10. https://www.youtube.com/watch?v=IJn-6Xo_kCM
