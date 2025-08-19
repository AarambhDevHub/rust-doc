---
title: "Unsafe Abstractions - Building Safe APIs Over Unsafe Code"
description: "Learn how to wrap unsafe operations in safe Rust abstractions, ensuring memory safety while still using low-level power."
date: 2025-10-16
weight: 1
---

# Unsafe Abstractions: Building Safe APIs Over Unsafe Code

The most important principle of `unsafe` Rust is not just using its "superpowers," but **containing them**. The gold standard for `unsafe` programming is to build a safe, ergonomic abstraction around a small, well-reasoned `unsafe` block. This approach gives you the best of both worlds: the low-level power and performance when you need it, and the compile-time safety guarantees of Rust everywhere else. This is how much of Rust's own standard library is built.[^1]

## The High-Security Kitchen Analogy: The "Black Box" Dish 🍽️

Imagine you are a head chef who has invented a new, revolutionary cooking technique. It's complex and requires using a custom-built, potentially dangerous piece of equipment (the `unsafe` code).

- **The Unsafe Operation**: Your secret, complex technique.
- **The Safe Abstraction**: You don't teach every cook in your kitchen this dangerous technique. Instead, you create a "black box" machine (the safe API).
    - The machine has a simple, safe interface: a green button labeled "Press to perfectly cook steak."
    - Inside the machine, your complex, dangerous technique is performed perfectly every time.
    - The other cooks don't need to know *how* it works; they just need to know that pressing the green button is safe and will always produce a perfectly cooked steak.

Your safe API is this "green button." It exposes the *benefits* of the `unsafe` code without exposing any of the *dangers*.[^2]

## Why Build Safe Abstractions?

1. **Containment**: It dramatically reduces the surface area for bugs. If a memory safety issue occurs, you know it must be within the small, audited `unsafe` blocks of your abstraction, not in the thousands of places where the safe API is used.[^1]
2. **Ergonomics**: It provides a clean, easy-to-use interface for users of your library. They don't need to understand the complex invariants you are manually upholding.
3. **Reusability**: A well-designed safe abstraction can be used confidently throughout a large codebase, just like any other safe Rust function or type.
4. **Maintainability**: When you need to investigate or update the `unsafe` logic, you only have one place to look.

## Case Study: Re-implementing `Vec::push`

The `Vec<T>` type is a classic example of a safe abstraction over `unsafe` code. A `Vec` needs to manually manage a block of memory on the heap, which involves raw pointers. Let's build a highly simplified `MyVec` to see how this works.

Our `MyVec` will need to track three things:

- A pointer to the heap allocation.
- The number of elements currently in the vector (`len`).
- The total number of elements the allocation can hold (`capacity`).

**The `unsafe` operations we'll need are:**

- Allocating and deallocating memory.
- Writing to memory using a raw pointer, potentially past the current `len` (but not past the `capacity`).


### Step 1: Defining the Struct and the `unsafe` Core

```rust
use std::ptr::NonNull;
use std::alloc::{alloc, dealloc, Layout};

// A simplified vector implementation.
pub struct MyVec<T> {
    ptr: NonNull<T>, // A non-null raw pointer for the allocation
    len: usize,
    capacity: usize,
}

impl<T> MyVec<T> {
    pub fn new() -> Self {
        Self {
            // Start with a dangling pointer for an empty vec.
            // `NonNull::dangling()` is a special pointer that is non-null
            // but doesn't point to any allocated memory. It's safe to have,
            // but not safe to dereference.
            ptr: NonNull::dangling(),
            len: 0,
            capacity: 0,
        }
    }

    // ... other methods will go here ...
}
```


### Step 2: Building the Safe `push` Abstraction

The `push` method is the "green button." It's a safe function that users can call. Internally, it will contain the `unsafe` logic for managing the buffer.

```rust
// Continuing the impl block from above...

    pub fn push(&mut self, item: T) {
        // First, handle the safe logic: do we have enough capacity?
        if self.len == self.capacity {
            // If not, we need to grow our allocation.
            self.grow();
        }

        // Now, enter the unsafe block to write the new element.
        unsafe {
            // SAFETY: We have just ensured there is enough capacity, so `self.len`
            // is a valid offset within our allocation. `self.ptr` is guaranteed
            // to be valid and allocated (or we would have grown it).
            // We use `ptr::write` to avoid creating a temporary `&mut T`,
            // which has stricter aliasing rules.
            let end = self.ptr.as_ptr().add(self.len);
            std::ptr::write(end, item);
        }

        // We can now safely increment the length.
        self.len += 1;
    }

    // The `grow` function is a private helper, also containing unsafe logic.
    fn grow(&mut self) {
        let new_capacity = if self.capacity == 0 { 1 } else { self.capacity * 2 };

        // Calculate the new layout for the allocation.
        let new_layout = Layout::array::<T>(new_capacity).unwrap();

        let new_ptr = if self.capacity == 0 {
            // If we are empty, just allocate new memory.
            unsafe { alloc(new_layout) }
        } else {
            // If we have existing data, we need to reallocate.
            // `realloc` will try to grow the allocation in place or move it.
            let old_layout = Layout::array::<T>(self.capacity).unwrap();
            unsafe {
                std::alloc::realloc(self.ptr.as_ptr() as *mut u8, old_layout, new_layout.size())
            }
        };

        // Update our pointer and capacity. `NonNull::new` handles the null check.
        self.ptr = match NonNull::new(new_ptr as *mut T) {
            Some(p) => p,
            None => std::alloc::handle_alloc_error(new_layout), // Abort on allocation failure
        };
        self.capacity = new_capacity;
    }
```


### Step 3: Implementing `Drop` for Cleanup

Our `MyVec` owns a raw allocation. If we don't clean it up when `MyVec` is dropped, we will have a memory leak. The `Drop` trait is the safe place to put this cleanup logic.

```rust
impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        if self.capacity != 0 {
            // 1. Manually drop all the elements in the vector.
            // We must do this to run their destructors.
            unsafe {
                for i in 0..self.len {
                    std::ptr::drop_in_place(self.ptr.as_ptr().add(i));
                }
            }

            // 2. Deallocate the memory block itself.
            let layout = Layout::array::<T>(self.capacity).unwrap();
            unsafe {
                dealloc(self.ptr.as_ptr() as *mut u8, layout);
            }
        }
    }
}
```


### The Result: A Safe Abstraction

We have now created `MyVec<T>` with a safe `push` method. A user can write this code without any `unsafe` blocks:

```rust
fn main() {
    let mut vec = MyVec::new();
    vec.push(10);
    vec.push(20);
    vec.push(30);
    // When `vec` goes out of scope here, its `drop` method is called automatically,
    // cleaning up the memory and preventing leaks.
}
```

The user of `MyVec` never needs to know about pointers, allocation, or memory layouts. They just use the safe API we have provided. We have successfully contained all the necessary `unsafe` complexity within our implementation, upholding the "black box" principle.

## Checklist for Building Safe Abstractions

1. **Identify the Invariants**: What promises must your `unsafe` code uphold to be correct? (e.g., "The `len` must never be greater than the `capacity`.")
2. **Minimize the `unsafe` Surface**: Keep your `unsafe` blocks as small and targeted as possible. Do as much of the logic as you can in safe code.
3. **Provide a Safe Public API**: Expose functions and methods that do not require the user to use the `unsafe` keyword. Your public API should enforce the invariants of your `unsafe` code.
4. **Use `SAFETY` Comments**: Meticulously document *why* each `unsafe` block is correct inside the implementation. Explain how it upholds the invariants.
5. **Implement `Drop`**: If your type manages raw resources (like memory, file handles, or network sockets), you **must** implement the `Drop` trait to ensure those resources are cleaned up correctly to prevent leaks.
6. **Test Rigorously**: Write extensive tests for your safe abstraction, including edge cases, to ensure your `unsafe` logic is sound. Tools like Miri can run your tests and detect certain kinds of Undefined Behavior.
<span style="display:none">[^3][^4][^5][^6][^7][^8]</span>

1. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch19-01-unsafe-rust.html
2. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
3. https://geo-ant.github.io/blog/2023/unsafe-rust-exploration/
4. https://www.reddit.com/r/rust/comments/emae93/what_are_valid_use_cases_for_unsafe_code/
5. https://cs.stanford.edu/~aozdemir/blog/unsafe-rust-escape/
6. https://users.rust-lang.org/t/adding-safe-abstraction-into-unsafe-global-raw-pointer/61515
7. https://jam1.re/blog/why-rusts-unsafe-works
8. https://www.cscjournals.org/manuscript/Journals/IJCSS/Volume18/Issue2/IJCSS-1712.pdf
