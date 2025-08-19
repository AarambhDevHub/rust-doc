---
title: "Vec Implementation Walkthrough"
description: "Understand how Rust's Vec is built internally using unsafe code and how it ensures safety for the end user."
date: 2025-10-16
weight: 2
---

# Unsafe Abstractions: A Walkthrough of Rust's `Vec<T>`

To truly appreciate Rust's philosophy, it's essential to understand how one of its most fundamental types, `Vec<T>`, is built. `Vec` provides a safe, ergonomic, growable array to all Rust developers. But underneath this safe exterior lies a carefully crafted core of `unsafe` code. This walkthrough will demystify how `Vec` uses `unsafe` to manage memory directly, and how it wraps this danger in a safe API, guaranteeing memory safety for the end user.

## The Professional Kitchen Analogy: Building a "Magic" Expanding Tray 🍽️

Imagine you are a culinary engineer designing a "magic" tray for a high-volume kitchen. This tray needs to hold a variable number of dishes.

- **A `Vec<T>`**: Your magic, self-expanding serving tray.
- **The Heap**: The vast storage warehouse next to the kitchen where you can request shelf space.
- `unsafe` **Code**: The complex, dangerous engineering work required to build the tray's internal expansion mechanism. You are working with raw materials (raw pointers) and custom tools (the memory allocator).
- **Safe API**: The simple, foolproof instructions you give to the chefs on how to use the tray (`push`, `pop`, `len`). They never need to know about the complex machinery inside.

The goal is to build a tray that is easy and safe for chefs to use, while hiding the complex and dangerous mechanics that make it work. This is precisely what `Vec` does. It encapsulates `unsafe` code to provide a safe abstraction.[^4][^5][^8]

## The Core Components of `Vec<T>`

A `Vec<T>` is, at its heart, just a pointer and two numbers. Its definition is surprisingly simple :[^7]

```rust
pub struct Vec<T> {
    ptr: *mut T,   // A raw pointer to the start of the memory allocation on the heap.
    cap: usize,    // The capacity: how many elements the allocated memory can hold.
    len: usize,    // The length: how many elements are currently in the Vec.
}
```

- **`ptr`**: A `*mut T` is a raw, mutable pointer. It's a memory address with no lifetime or ownership information. The borrow checker doesn't track it, which is why working with it is `unsafe`.
- **`cap` (Capacity)**: The amount of space we've *reserved* on the heap.
- **`len` (Length)**: The amount of space we're *currently using*.

The core invariant that `Vec` must always uphold is: **`0 <= len <= cap`**.

## Building Our Own `MyVec<T>`: A Simplified Implementation

Let's build a simplified version of `Vec` to see where `unsafe` is necessary.

```rust
use std::ptr::{self, NonNull};
use std::alloc::{self, Layout};

// Our simplified Vec
pub struct MyVec<T> {
    ptr: NonNull<T>, // We use NonNull to store the pointer, which is an optimization.
                     // It's a raw pointer that is guaranteed to never be null.
    len: usize,
    cap: usize,
}

impl<T> MyVec<T> {
    /// Creates a new, empty MyVec.
    pub fn new() -> Self {
        // For an empty Vec, we don't allocate any memory. The pointer is "dangling"
        // but that's okay because the capacity and length are 0.
        Self {
            ptr: NonNull::dangling(),
            len: 0,
            cap: 0,
        }
    }

    /// Pushes a new element onto the Vec.
    pub fn push(&mut self, elem: T) {
        // The most important method: handling growth.
        if self.len == self.cap {
            // If we're full, we need to reallocate.
            self.grow();
        }

        // Now we know we have space.
        unsafe {
            // SAFETY: We have just checked that len < cap, so `ptr.add(len)`
            // points to a valid, allocated, but uninitialized memory location.
            // `ptr::write` writes the element without reading the old memory,
            // which is crucial because the memory is uninitialized.
            ptr::write(self.ptr.as_ptr().add(self.len), elem);
        }

        // Increment the length to mark the new element as initialized.
        self.len += 1;
    }

    /// The internal growth logic.
    fn grow(&mut self) {
        // Calculate the new capacity. If we have 0 capacity, start with 4.
        // Otherwise, double it. This is a common growth strategy.
        let new_cap = if self.cap == 0 { 4 } else { self.cap * 2 };

        // Create the layout for the new memory allocation.
        let new_layout = Layout::array::<T>(new_cap).unwrap();

        let new_ptr = if self.cap == 0 {
            // If we're allocating for the first time...
            unsafe { alloc::alloc(new_layout) }
        } else {
            // If we're reallocating, we need to move the old data.
            let old_layout = Layout::array::<T>(self.cap).unwrap();
            unsafe {
                alloc::realloc(self.ptr.as_ptr() as *mut u8, old_layout, new_layout.size())
            }
        };

        // Update our pointer and capacity.
        // `NonNull::new` is used to handle the case where allocation fails.
        self.ptr = match NonNull::new(new_ptr as *mut T) {
            Some(p) => p,
            None => alloc::handle_alloc_error(new_layout),
        };
        self.cap = new_cap;
    }
}
```


### Why is `unsafe` Necessary Here?

1. **Direct Memory Management (`alloc` and `realloc`)**: Safe Rust does not allow you to directly call the global memory allocator. This is a low-level operation that requires `unsafe` because you are now manually responsible for managing that memory's lifetime. If you forget to deallocate it, you have a memory leak.
2. **Writing to Uninitialized Memory (`ptr::write`)**: When `Vec` grows, it gets a chunk of uninitialized memory from the allocator. You cannot safely write to this memory. A normal assignment (`*ptr = value`) would try to *drop* the old value at that location first, but since the memory is uninitialized, this would be Undefined Behavior. `ptr::write` is an `unsafe` function that "blindly" copies the bits of the new element into the memory location without trying to drop the old contents.
3. **Pointer Arithmetic (`ptr.add(len)`)**: Moving a raw pointer around using `.add()` is an `unsafe` operation because the compiler cannot prove that the resulting pointer will still be valid and within the allocated bounds. Our code is safe because our own logic (`len < cap`) guarantees it.

## Providing Safe Abstractions

The code above shows the dangerous, `unsafe` core. The magic of `Vec` is that it wraps this core in a completely safe API. Let's add a `get` method.

```rust
// Inside the impl<T> MyVec<T> block...

pub fn get(&self, index: usize) -> Option<&T> {
    // This is the SAFE part.
    if index < self.len {
        // We perform a bounds check. This is a runtime cost, but it
        // guarantees safety.
        unsafe {
            // SAFETY: We have just checked that `index` is within our
            // initialized length, so the pointer is valid and points to an
            // initialized element of type T.
            Some(&*self.ptr.as_ptr().add(index))
        }
    } else {
        // If the index is out of bounds, we return None.
        None
    }
}
```

The public `get` method is completely safe. It takes an index and returns an `Option<&T>`. It's impossible for a user of this method to cause Undefined Behavior. If they provide an invalid index, they get `None`. Inside, we use an `unsafe` block to perform the pointer arithmetic and dereference, but we only do so *after* a safe bounds check.

This is the fundamental pattern of `unsafe` abstractions in Rust: **use `unsafe` internally to do things the borrow checker can't understand, but wrap it in a public API that performs the necessary checks to guarantee the safety invariants are never broken.**

## Handling `Drop`

What happens when our `MyVec` goes out of scope? We need to deallocate the memory we requested from the allocator. This requires implementing the `Drop` trait.

```rust
impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        if self.cap != 0 {
            // We must first drop all the elements that are currently initialized.
            while self.len > 0 {
                self.len -= 1;
                unsafe {
                    // SAFETY: We know `len` is valid. `ptr::read` moves the value
                    // out of the Vec, allowing it to be dropped.
                    ptr::read(self.ptr.as_ptr().add(self.len));
                }
            }

            // Now that all elements are dropped, we can deallocate the memory.
            let layout = Layout::array::<T>(self.cap).unwrap();
            unsafe {
                alloc::dealloc(self.ptr.as_ptr() as *mut u8, layout);
            }
        }
    }
}
```

Dropping is also `unsafe` for two reasons:

1. We are reading from a raw pointer to get the elements to drop them.
2. We are calling the global deallocator.

Our logic ensures this is safe by only dropping the `len` initialized elements, and only deallocating if `cap` is not 0.

## Conclusion

`Vec<T>` is the perfect case study for `unsafe` Rust.

- It **must** use `unsafe` code to perform the low-level memory management that safe Rust forbids.
- It uses its own internal logic (tracking `len` and `cap`) to **manually uphold** Rust's safety invariants.
- It exposes a **100% safe public API** by adding runtime checks (like bounds checks) and using the type system (`Option` and `&T`) to prevent users from misusing it.

This is the core philosophy of `unsafe` in the standard library: contain the danger, verify its correctness with extreme care, and present a foolproof, safe interface to the world.
<span style="display:none">[^1][^10][^2][^3][^6][^9]</span>

1. https://doc.rust-lang.org/std/vec/struct.Vec.html
2. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
3. https://stackoverflow.com/questions/73419704/why-does-the-rust-standard-library-have-so-much-unsafe-code
4. https://users.rust-lang.org/t/why-is-vec-implemented-un-safely/1580
5. https://www.reddit.com/r/rust/comments/xc5wut/should_we_be_worried_about_proliferation_of/
6. https://news.ycombinator.com/item?id=27273259
7. https://d3m3vilurr.gitbooks.io/the-unsafe-rust-programming-language/vec.html
8. https://nora.codes/post/what-is-rusts-unsafe/
9. https://users.rust-lang.org/t/is-my-highly-unsafe-code-correct-in-place-mapping-a-vector/96078
10. https://cryptical.xyz/rust/unsafe
