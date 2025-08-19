---
title: "Unsafe Rust Basics - Raw Pointers"
description: "Understand raw pointers in Rust, their use cases, and differences from references."
date: 2025-10-13
weight: 2
---

# Unsafe Rust Basics: A Beginner's Guide to Raw Pointers

Understanding raw pointers in Rust is like learning that a professional kitchen has a set of **unlabeled, razor-sharp, specialized knives** stored in a locked cabinet. Most of the time, chefs use the standard, safe-handled knives (Rust's references and smart pointers). But for very specific, high-stakes tasks—like preparing fugu or performing a complex butchering technique—they might need to use these dangerous, specialized tools. This requires explicit permission (`unsafe` block), intense focus, and accepting full responsibility for the outcome.

## The Professional Kitchen Knife Analogy 🔪

| **Tool** | **Kitchen Analogy** | **Key Characteristics** |
| :-- | :-- | :-- |
| **References (`&`, `&mut`)** | Standard, ergonomic, safety-guarded knives | **Safe**. The borrow checker guarantees they are always valid, non-null, and follow strict aliasing rules. |
| **Smart Pointers (`Box`, `Rc`)** | Specialized tools with built-in safety features | **Safe**. Manage memory and ownership automatically (e.g., `Box` handles cleanup). |
| **Raw Pointers (`*const`, `*mut`)** | Unlabeled, unguarded, specialized knives | **Unsafe**. The compiler cannot guarantee anything about them. They are your responsibility. |


***

## What Are Raw Pointers?

Raw pointers are memory addresses without the safety guarantees and compile-time checks that Rust's references (`&T`, `&mut T`) provide. They are most similar to pointers in C/C++.[^3]

There are two types of raw pointers :[^5]

- `*const T`: An immutable raw pointer. You are not supposed to write to the memory it points to through this pointer.
- `*mut T`: A mutable raw pointer. You can read from and write to the memory it points to.


### The Fundamental Difference: The Compiler's Trust

- **References**: You are making a **promise to the compiler** that your code follows the borrow checker rules. The compiler verifies this promise.
- **Raw Pointers**: You are telling the compiler, "I know what I'm doing. **Turn off your safety checks** for this specific operation. I will personally guarantee that this pointer is valid."


## The Dangers: What Guarantees Do You Lose?

When you use raw pointers, you are stepping outside of Safe Rust and giving up the core guarantees that make Rust safe :[^1]

- **No Guaranteed Validity**: They can be null, dangling (pointing to memory that has been freed), or point to an unaligned memory address. Dereferencing an invalid pointer is undefined behavior.[^6]
- **No Borrow Checking**: The aliasing rules do not apply. You can have multiple mutable raw pointers to the same data, creating a risk of data races.
- **No Automatic Cleanup**: They do not have lifetimes and do not automatically manage resources like `Box<T>` does. This can lead to memory leaks if not handled carefully.
- **No Ownership Model**: They are "plain-old-data" and can be freely copied. They do not move ownership.[^1]

Because of these dangers, any operation that involves **dereferencing** a raw pointer must be done inside an `unsafe` block.[^2]

## How to Work with Raw Pointers

### 1. Creating Raw Pointers (This is Safe)

You can create raw pointers without an `unsafe` block. The act of creating a pointer is not dangerous; using it is. The most common way is to coerce a reference.[^2]

```rust
fn main() {
    let my_number: i32 = 42;
    let mut my_speed: i32 = 88;

    // Create an immutable raw pointer from a reference.
    let num_ptr: *const i32 = &my_number;

    // Create a mutable raw pointer from a mutable reference.
    let speed_ptr: *mut i32 = &mut my_speed;

    // You can also create a null pointer.
    let null_ptr: *const i32 = std::ptr::null();

    println!("Raw pointers created (but not yet used).");
}
```


### 2. Dereferencing Raw Pointers (This is `unsafe`)

To read or write the data a raw pointer points to, you must use the dereference operator (`*`) inside an `unsafe` block. This is you telling the compiler, "I take full responsibility."

```rust
fn main() {
    let mut my_number = 10;
    let num_ptr: *mut i32 = &mut my_number;

    // To read the value, we need an unsafe block.
    unsafe {
        println!("The value at the pointer is: {}", *num_ptr);
    }

    // To write to the value, we also need an unsafe block.
    unsafe {
        *num_ptr = 20; // Modify the data through the raw pointer.
    }

    // The original variable is changed.
    println!("The original number is now: {}", my_number); // Prints 20
}
```


### 3. Converting Between Pointers and References

- **Reference to Raw Pointer**: This is safe and happens implicitly (coercion) or explicitly with `as`.[^6]

```rust
let x = 5;
let x_ptr: *const i32 = &x;
```

- **Raw Pointer to Reference**: This is **highly unsafe** because you are promising the compiler that the pointer is valid and adheres to all of the reference's safety rules (non-null, aligned, correct lifetime, etc.). The preferred way is `&*`.[^1]

```rust
fn main() {
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    unsafe {
        // Here, we are asserting to the compiler that r1 is a valid,
        // live pointer, and we are creating a safe reference from it.
        let safe_ref: &i32 = &*r1;
        println!("Value via safe reference: {}", safe_ref);
    }
}
```


## When Are Raw Pointers Necessary?

If they are so dangerous, why do they exist? There are a few critical use cases where Safe Rust's rules are too restrictive, and you need to drop down to a lower level of control.[^4]

### 1. Interfacing with C Code (FFI)

When calling functions from a C library, you must use raw pointers because C code doesn't understand Rust's references and lifetimes. Rust's `*const T` and `*mut T` are designed to be compatible with C's `const T*` and `T*`.[^1]

```rust
// A function declaration from a C library
extern "C" {
    fn process_data(data: *const u8, length: usize);
}

fn call_c_function() {
    let my_data: &[u8] = &[1, 2, 3];
    let data_ptr: *const u8 = my_data.as_ptr(); // Get a raw pointer to the slice's data

    // Calling the C function is unsafe because the C code could do anything.
    unsafe {
        process_data(data_ptr, my_data.len());
    }
}
```


### 2. Building Safe Abstractions

This is the most important use case within the Rust ecosystem. The building blocks of many safe data structures, like `Box<T>`, `Rc<T>`, `Vec<T>`, and even `Mutex<T>`, are implemented using `unsafe` code and raw pointers internally. The authors use `unsafe` code to build a data structure with a specific behavior (like shared ownership or a growable array), and then they expose a completely safe public API.

**Example: A Simplified `Box`**
A `Box` needs to manage a raw pointer to heap-allocated memory. The `Box` struct itself is safe, but its internal implementation uses `unsafe` to allocate and deallocate the memory.

### 3. Interacting with Hardware

When writing low-level code like device drivers or kernels, you often need to read from or write to specific, fixed memory addresses that correspond to hardware registers. Raw pointers are the only way to do this.

```rust
fn write_to_hardware_register() {
    // A hypothetical memory-mapped hardware register
    let register_address = 0xDEADBEEF as *mut u32;

    unsafe {
        // Write a value directly to that memory address.
        *register_address = 0x12345678;
    }
}
```


## Summary: A Hierarchy of Choice

When dealing with pointers and memory in Rust, follow this hierarchy:

1. **Always prefer safe references (`&`, `&mut`)**. This should be your default for 99% of your code. Let the borrow checker be your guide.
2. **Use smart pointers (`Box`, `Rc`, `Arc`)** when you need different ownership semantics (heap allocation, shared ownership). These are safe abstractions.
3. **Only use `unsafe` and raw pointers** when you are absolutely certain that you cannot achieve your goal with safe constructs. This is typically limited to FFI, building low-level data structures, or hardware interaction. When you do, wrap your `unsafe` code in the smallest possible block and provide extensive documentation explaining why it is necessary and how you are upholding the safety invariants.
<span style="display:none">[^7][^8]</span>

1. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/raw-pointers.html
2. https://codedamn.com/news/rust/understanding-unsafe-rust-guide-working-with-raw-pointers
3. https://users.rust-lang.org/t/understanding-of-rust-raw-pointers/88996
4. https://www.reddit.com/r/rust/comments/litwzk/best_resource_to_understand_pointers_in_rust/
5. https://reintech.io/blog/understanding-implementing-rusts-raw-pointers
6. https://doc.rust-lang.org/std/primitive.pointer.html
7. https://corner.buka.sh/understanding-box-and-raw-pointers-in-rust-a-guide-to-safe-and-unsafe-memory-access/
8. https://www.youtube.com/watch?v=f3Es83evRhY
