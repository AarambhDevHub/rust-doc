---
title: "Advanced Unsafe Operations - Pointer Arithmetic"
description: "Learn how to perform pointer arithmetic in Rust and when it might be necessary in low-level programming."
date: 2025-10-14
weight: 2
---

# Advanced Unsafe Operations: Pointer Arithmetic in Rust

Learning pointer arithmetic in Rust is like a head chef learning to **work with raw, unprocessed ingredients directly from the farm**, bypassing the kitchen's usual sanitized and pre-packaged supplies. It's a powerful, low-level skill that provides ultimate control but requires immense care and responsibility. One wrong move—one miscalculation—and you could ruin the entire dish (cause Undefined Behavior). This is an advanced technique, firmly inside the `unsafe` world, used only when absolutely necessary for performance or interoperability.

## The Raw Ingredients Analogy: Why This is Dangerous 🍽️

- **A Safe Rust Slice (`&[T]`)**: A pre-packaged, sealed container of ingredients. It has a label telling you exactly what's inside and how much there is (length). You can't accidentally grab more than what's in the container.
- **A Raw Pointer (`*mut T`)**: A giant, open sack of flour in the middle of the floor. You have a scoop (the pointer) and you can take flour from anywhere in the sack. But:
    * There's no label telling you how much flour is in the sack (no bounds checking).
    * You could accidentally scoop from the floor next to the sack (access out-of-bounds memory).
    * You could scoop from a sack of salt, thinking it's flour (type confusion).

**Pointer arithmetic is the act of moving your scoop around that floor**, from one sack to another, or even to places where there is no sack at all. It's your responsibility to know where the sacks are, what's in them, and how big they are.

## What is Pointer Arithmetic?

Pointer arithmetic is the process of adding an integer offset to a pointer to calculate a new memory address. Crucially, the offset is **scaled by the size of the type the pointer points to**.

- If you have a pointer `ptr` to a `u32` (which is 4 bytes), `ptr.add(1)` does **not** add 1 byte to the address. It adds `1 * size_of::<u32>()` bytes, moving the pointer to the memory address of the *next* `u32`.

This is the fundamental operation for manually navigating arrays or other contiguous blocks of memory.[^1]

## The Tools for Pointer Arithmetic in Rust

All pointer arithmetic must happen inside an `unsafe` block. The primary methods are defined on raw pointers (`*const T` and `*mut T`).[^4]


| Method | Description | Safety Condition |
| :-- | :-- | :-- |
| `offset(count: isize)` (unsafe) | Moves the pointer by `count` elements. `count` can be negative. | **Strictly enforced**: The start and end pointers **must** be within the bounds of the same allocated object (or one-past-the-end). Violating this is **instant Undefined Behavior (UB)** [^6]. |
| `add(count: usize)` (unsafe) | Moves the pointer forward by `count` elements. A convenience wrapper around `offset`. | Same as `offset`. |
| `sub(count: usize)` (unsafe) | Moves the pointer backward by `count` elements. A convenience wrapper around `offset`. | Same as `offset`. |
| `wrapping_offset(count: isize)` (safe) | Moves the pointer by `count` elements, but allows the address to wrap around (like integer wrapping). **The method itself is safe.** | **Loosely enforced**: The resulting pointer must not be dereferenced if it points outside a valid allocation. Dereferencing an invalid pointer is UB, but just *creating* it is not [^6]. |

**Key Takeaway**: Use `offset`, `add`, or `sub` when you can guarantee you are staying within the bounds of a single allocated object. Use `wrapping_offset` for more complex address calculations where you might temporarily go out of bounds, but you promise not to dereference the invalid pointers.[^7]

## Example: Manually Iterating Through an Array

Let's implement a simplified version of an iterator using raw pointers. This demonstrates the core concept of moving a pointer through a contiguous block of memory.

```rust
fn main() {
    let data: [i32; 5] = [10, 20, 30, 40, 50];

    // Get a raw pointer to the start of the array.
    let ptr: *const i32 = data.as_ptr();
    let len = data.len();

    println!("Manually iterating through the array with pointer arithmetic:");

    for i in 0..len {
        // We need an unsafe block to perform pointer arithmetic and dereference.
        unsafe {
            // Calculate the address of the i-th element.
            // This is equivalent to `&data[i]` but done manually.
            let current_element_ptr = ptr.add(i);

            // Dereference the pointer to get the value.
            let value = *current_element_ptr;

            println!("  - At index {}: value = {}", i, value);

            // SAFETY: The loop runs from 0 to len - 1.
            // `ptr.add(i)` is guaranteed to stay within the bounds of `data`
            // because `i` is always less than `len`. Therefore, this is safe.
        }
    }
}
```


## When is Pointer Arithmetic Necessary?

In idiomatic, safe Rust, you should almost never need to use pointer arithmetic. Slices, iterators, and other safe abstractions handle it for you. However, it becomes essential in a few specific low-level scenarios :[^3]

### 1. Implementing Custom Data Structures

When you build a data structure that manages its own memory, like a custom `Vec` or a `HashMap`, you are working with raw, uninitialized memory. Pointer arithmetic is the only way to navigate and initialize this memory.

**Conceptual Example: A simplified `MyVec::push`**

```rust
// This is a conceptual, simplified example.
struct MyVec<T> {
    ptr: *mut T, // Pointer to the start of the allocation
    len: usize,
    capacity: usize,
}

impl<T> MyVec<T> {
    fn push(&mut self, value: T) {
        // (Assume we have already checked capacity and reallocated if needed)
        unsafe {
            // Calculate the address of the next empty slot.
            let end_ptr = self.ptr.add(self.len);

            // Write the new value to that memory location without reading
            // or dropping the old (uninitialized) memory.
            std::ptr::write(end_ptr, value);

            // Increment the length.
            self.len += 1;
        }
    }
}
```


### 2. Interfacing with C Libraries (FFI)

C APIs often work with arrays by passing a pointer to the first element and a length. To work with such an API in Rust, you'll need to use pointer arithmetic to access the elements of the C array.

**Example: Receiving an array from C**

```rust
/// A hypothetical C function that fills a buffer and returns the number of items written.
// extern "C" fn fill_buffer(buffer: *mut i32, max_len: usize) -> usize;

// A safe Rust wrapper around the unsafe C function.
fn get_data_from_c() -> Vec<i32> {
    let mut buffer = Vec::with_capacity(10);
    let buffer_ptr = buffer.as_mut_ptr();
    let max_len = buffer.capacity();
    let items_written;

    unsafe {
        // SAFETY: We are calling a C function. We trust it to not write
        // more than `max_len` items into our buffer.
        // items_written = fill_buffer(buffer_ptr, max_len);

        // For this example, let's simulate the C function's behavior
        items_written = 5;
        for i in 0..items_written {
            *buffer_ptr.add(i) = (i as i32) * 10;
        }

        // We must update the Vec's length to reflect the initialized data.
        buffer.set_len(items_written);
    }
    buffer
}
```

Here, `buffer.set_len()` is a critical `unsafe` function. We are promising the `Vec` that we have personally initialized the first `items_written` elements, so it's safe for it to manage them.

### 3. High-Performance Code and SIMD

In some rare cases, manual pointer manipulation can be used to perform optimizations that the compiler might miss, especially when dealing with explicit SIMD (Single Instruction, Multiple Data) operations or other very low-level memory layouts. This is an extremely advanced topic and should only be approached after extensive profiling has proven it necessary.

## Summary: A Tool of Last Resort

| **Abstraction Level** | **Tool** | **Analogy** |
| :-- | :-- | :-- |
| **Highest (Safest)** | Iterators (`.iter()`, `.map()`) | Following a precise, tested, step-by-step recipe. |
| **Mid-Level (Safe)** | Slice Indexing (`my_slice[i]`) | Grabbing a pre-packaged ingredient from a clearly labeled shelf. |
| **Low-Level (Unsafe)** | Pointer Arithmetic (`ptr.add(i)`) | Scooping raw ingredients from giant, unlabeled sacks on the floor. |

Pointer arithmetic is a fundamental concept in systems programming, and Rust provides the tools to do it. However, it places the full responsibility for memory safety squarely on your shoulders. It's a powerful tool, but like a chef's sharpest knife, it demands respect, precision, and a deep understanding of the safety rules you are temporarily bypassing. Always prefer safe abstractions and only use pointer arithmetic when the problem genuinely demands it.
<span style="display:none">[^2][^5][^8][^9]</span>

1. https://blog.devgenius.io/playing-with-pointer-arithmetic-in-rust-8b80cec52890
2. https://stackoverflow.com/questions/24759028/how-should-you-do-pointer-arithmetic-in-rust
3. https://users.rust-lang.org/t/pointer-arithmetic-like-in-c/87236
4. https://doc.rust-lang.org/std/primitive.pointer.html
5. https://users.rust-lang.org/t/is-the-pointer-arithmetic-on-a-pointer-that-does-not-point-to-an-element-of-an-array-undefined-behavior/105420
6. https://model-checking.github.io/verify-rust-std/challenges/0003-pointer-arithmentic.html
7. https://www.reddit.com/r/rust/comments/1k92p50/why_does_rust_standard_library_use_wrapping_math/
8. https://users.rust-lang.org/t/pointer-arithmetic/5717
9. https://news.ycombinator.com/item?id=13474237
