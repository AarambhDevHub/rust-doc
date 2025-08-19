---
title: "Unsafe Practice - Applying Unsafe in Real Projects"
description: "Hands-on exercises with unsafe Rust: wrappers, FFI projects, performance optimizations, and safety audits."
date: 2025-10-17
weight: 60
---

# Unsafe Practice: A Workshop on Applying Unsafe Rust in Real Projects

Welcome to the `unsafe` workshop. Think of this as your **advanced certification for handling dangerous machinery**. You've learned the theory of why the safety guards exist (`safe` Rust), what the emergency shut-off does (`unsafe` keyword), and the catastrophic consequences of misuse (Undefined Behavior). Now, it's time for hands-on practice under controlled conditions.

This guide will walk you through practical, real-world exercises where `unsafe` is necessary and powerful. We will build safe abstractions, interface with C libraries, and even explore performance optimization, all while keeping a critical eye on the safety invariants we must uphold.

## The "Building a Safety Cage" Analogy 🏗️

Every `unsafe` project should follow one core principle: you are not disabling safety, you are **building a new kind of safety**. You are stepping into a dangerous area to construct a sturdy, reliable safety cage (`safe` abstraction) around a dangerous machine (low-level operations). Once the cage is built, everyone else can use the machine via the cage's safe remote controls without ever being exposed to the danger.

This workshop is about learning how to build those cages correctly.

***

### Exercise 1: Building a Safe Wrapper - `MyVec<T>`

The most fundamental data structure, `Vec<T>`, cannot be implemented in 100% safe Rust. It requires `unsafe` to manage raw memory allocations. Let's build a simplified version to understand how.

**The Goal**: Create a `MyVec<T>` that can grow and store elements on the heap.
**`unsafe` Superpowers Needed**:

- Interacting with the memory allocator.
- Writing to uninitialized memory.
- Managing capacity vs. length with raw pointers.

**Step 1: The Struct Definition**

```rust
use std::ptr::NonNull;
use std::mem;
use std::alloc::{self, Layout};

// Our simplified Vec.
pub struct MyVec<T> {
    ptr: NonNull<T>, // NonNull is like *mut T, but guaranteed non-null.
    len: usize,      // Number of initialized elements
    capacity: usize, // Number of elements we have space for
}
```

**Step 2: `new` and `drop` (The Lifecycle)**

```rust
impl<T> MyVec<T> {
    pub fn new() -> Self {
        assert!(mem::size_of::<T>() != 0, "We do not handle ZSTs");
        Self {
            // A dangling pointer that is non-null. This is a common pattern for
            // an empty collection, as it doesn't point to allocated memory yet.
            ptr: NonNull::dangling(),
            len: 0,
            capacity: 0,
        }
    }
}

// We must implement Drop to deallocate our memory and prevent leaks.
impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        if self.capacity != 0 {
            unsafe {
                // SAFETY: ptr is valid for `self.capacity` elements and was
                // allocated by us with the same layout.
                let layout = Layout::array::<T>(self.capacity).unwrap();
                alloc::deallocate(self.ptr.as_ptr() as *mut u8, layout);
            }
        }
    }
}
```

**Step 3: `push` (The Unsafe Core Logic)**
This is where the real danger lies. We need to handle re-allocation when we run out of space.

```rust
// (Inside impl<T> MyVec<T>)
pub fn push(&mut self, item: T) {
    if self.len == self.capacity {
        // We're out of space, we need to grow.
        let new_capacity = if self.capacity == 0 { 4 } else { self.capacity * 2 };

        let new_layout = Layout::array::<T>(new_capacity).unwrap();
        let new_ptr = if self.capacity == 0 {
            unsafe { alloc::alloc(new_layout) }
        } else {
            // Reallocate the memory.
            let old_layout = Layout::array::<T>(self.capacity).unwrap();
            unsafe {
                alloc::realloc(self.ptr.as_ptr() as *mut u8, old_layout, new_layout.size())
            }
        };

        // Update our pointer and capacity.
        self.ptr = match NonNull::new(new_ptr as *mut T) {
            Some(p) => p,
            None => alloc::handle_alloc_error(new_layout), // Abort on allocation failure
        };
        self.capacity = new_capacity;
    }

    // Now we have space. Write the new item to the end.
    unsafe {
        // SAFETY: `self.len < self.capacity`, so the pointer at this offset
        // is valid and within our allocated block. We are writing to
        // potentially uninitialized memory, which is allowed.
        let end = self.ptr.as_ptr().add(self.len);
        std::ptr::write(end, item);
    }

    // Increment the length.
    self.len += 1;
}
```

**The Safety Audit**:

- **Wrapper**: We have created a `MyVec` with a safe `push` method. The user of `MyVec` never needs to write `unsafe`.
- **Invariants**: We uphold the invariant that `len <= capacity`. Our `drop` implementation correctly uses the `capacity` to deallocate the memory we allocated. The `push` method carefully reallocates and copies memory, ensuring the pointer is always valid.

***

### Exercise 2: A Real FFI Project - Safe Wrapper for `zlib`

**The Goal**: Create a safe Rust function that compresses data using the common C library, `zlib`.
**`unsafe` Superpowers Needed**:

- Calling an external C function.
- Passing raw pointers to buffers.
- Translating C-style error codes into a Rust `Result`.

**Step 1: Setup (`build.rs`)**
To link with a system library like `zlib`, we can use a build script.
**`build.rs`**

```rust
fn main() {
    // Tell cargo to link against the system's zlib library.
    println!("cargo:rustc-link-lib=z");
}
```

**Step 2: FFI Declarations**
We need to declare the `compressBound` and `compress` functions from `zlib.h`.

```rust
use libc::{c_int, c_ulong, c_void};

// Corresponds to the `zlib` library.
#[link(name = "z")]
extern "C" {
    // Returns an upper bound on the required buffer size for compression.
    fn compressBound(sourceLen: c_ulong) -> c_ulong;

    // Compresses data.
    fn compress(
        dest: *mut u8,
        destLen: *mut c_ulong,
        source: *const u8,
        sourceLen: c_ulong,
    ) -> c_int;
}

// Define zlib error codes for clarity.
const Z_OK: c_int = 0;
const Z_BUF_ERROR: c_int = -5;
```

**Step 3: Building the Safe Wrapper**
Now, we build our "safety cage."

```rust
// Our custom, safe error type.
#[derive(Debug)]
pub enum CompressionError {
    BufferTooSmall,
    Unknown(i32),
}

/// A safe Rust function that compresses a byte slice.
pub fn compress_data(source: &[u8]) -> Result<Vec<u8>, CompressionError> {
    let source_len = source.len() as c_ulong;

    // 1. Ask zlib how large our destination buffer should be.
    let mut dest_len = unsafe { compressBound(source_len) };

    // 2. Create the destination buffer.
    let mut dest: Vec<u8> = vec![0; dest_len as usize];

    // 3. Call the unsafe C function.
    let result = unsafe {
        // SAFETY: We are passing pointers to valid memory.
        // - `dest` is a mutable Vec, so `as_mut_ptr` is valid.
        // - `source` is a valid slice.
        // - The lengths are derived from the slices themselves.
        // - We have allocated `dest` with a size recommended by zlib itself.
        compress(
            dest.as_mut_ptr(),
            &mut dest_len,
            source.as_ptr(),
            source_len,
        )
    };

    // 4. Translate the C error code into a Rust Result.
    match result {
        Z_OK => {
            // Shrink the vector to the actual compressed size.
            dest.truncate(dest_len as usize);
            Ok(dest)
        }
        Z_BUF_ERROR => Err(CompressionError::BufferTooSmall),
        other => Err(CompressionError::Unknown(other)),
    }
}
```

**The Safety Audit**:

- **Wrapper**: The `compress_data` function is 100% safe to call.
- **FFI**: All interactions with the C library happen inside a single `unsafe` block.
- **Pointers**: We create pointers from valid Rust slices (`&[u8]`) and vectors (`Vec<u8>`), so we know they are not null or dangling.
- **Error Handling**: We explicitly check the return code from the C function and convert it into a meaningful Rust `enum`, propagating errors safely.

***

### Exercise 3: Performance Optimization - Ditching Bounds Checks

**The Goal**: Speed up a "hot loop" by removing bounds checks.
**Warning**: This is an advanced technique. **Always profile first!** Only do this if you have proven that bounds checks are a significant bottleneck.

**The Scenario**: Summing all numbers in a very large vector.

```rust
// The safe, idiomatic way. The compiler is often smart enough to
// optimize away the bounds checks here, but let's pretend it isn't.
pub fn sum_safe(values: &[u64]) -> u64 {
    values.iter().sum()
}

// The unsafe, manually optimized way.
pub fn sum_unsafe(values: &[u64]) -> u64 {
    let mut total = 0;
    let len = values.len();
    let ptr = values.as_ptr();

    // Iterate by index.
    for i in 0..len {
        unsafe {
            // SAFETY: The loop runs from `0` to `len - 1`. `i` will therefore
            // always be a valid index into the slice. We are using `add` and
            // `read` on a raw pointer, which is faster as it doesn't
            // perform a bounds check on every iteration.
            total += *ptr.add(i);
        }
    }
    total
}
```

**The Safety Audit**:

- **Wrapper**: The function `sum_unsafe` still presents a safe public API.
- **Invariant**: The entire safety of this function rests on one single invariant: `i` must never be greater than or equal to `len`. Our `for i in 0..len` loop structure guarantees this. If the loop was written as `0..=len`, this would be **instant Undefined Behavior**.
- **Realism Check**: In modern Rust, the optimizer (`LLVM`) is extremely good. For a simple loop like this, it is very likely that the safe version `sum_safe` will be optimized to the same machine code as `sum_unsafe`. This technique is more relevant for complex access patterns where the optimizer cannot prove the indices are safe. **Always trust the compiler first.**


## A Final Word: The `unsafe` Mindset

Using `unsafe` responsibly is not about being a daredevil; it's about being a meticulous engineer. It requires discipline, a deep respect for the rules, and a commitment to building safety on top of a foundation of carefully controlled danger. By mastering these patterns, you can unlock the full power of Rust, from the highest-level abstractions down to the bare metal.

1. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
2. https://www.apriorit.com/dev-blog/interoperability-unsafe-rust
3. https://doc.rust-lang.org/nomicon/ffi.html
4. https://www.reddit.com/r/rust/comments/1esj2v8/unsafe_rust/
5. https://moldstud.com/articles/p-common-unsafe-rust-patterns-use-cases-and-implementation-techniques
6. https://stackoverflow.com/questions/61280518/rust-wrapping-user-provided-safe-function-into-unsafe-ffi-function-for-ffi-cal
7. https://discourse.haskell.org/t/expert-advice-needed-best-resources-for-unsafe-ffi-memory-safety-real-time-audio-threads-and-ghc-core-optimization/9574
8. https://google.github.io/comprehensive-rust/unsafe-rust/exercise.html
