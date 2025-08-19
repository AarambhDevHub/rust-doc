---
title: "Advanced Async Patterns - Pinning and Unpin"
description: "Understand pinning, the Unpin trait, and why they are essential for safe async programming in Rust."
date: 2025-10-02
weight: 2
---

# Advanced Async in Rust: Pinning and the `Unpin` Trait Explained

Understanding `Pin` and `Unpin` in Rust is like learning the **safety rules for handling delicate, half-finished dishes in a professional kitchen**. Imagine a chef is building a complex, multi-layered cake. If someone comes along and moves the cake to a different table while the chef isn't looking, the layers could shift, and the whole structure could collapse. **Pinning** is the safety mechanism that says: "This cake is currently being worked on. Do *not* move it from this spot in memory, or something terrible will happen."

This concept is absolutely essential for safe `async` programming in Rust because `async` code often creates self-referential data structures that, like our delicate cake, will break if they are moved in memory.[^3]

## The Delicate, Self-Referential Cake Analogy 🎂

### Imagine a Chef Building a Multi-Layered Cake

- **The Cake**: An `async` task (a `Future`).
- **The Layers**: The state of the `async` task.
- **A Pointer to a Layer**: A reference from one part of the cake's structure to another (a self-reference). For example, a pointer from the top layer to the bottom layer.
- **Moving the Cake**: Moving the `async` task in memory, which the `async` runtime (like `tokio`) does all the time to manage tasks efficiently.

**The Problem: Moving a Self-Referential Struct**

1. A chef starts building a cake at **Table A**. They add the bottom layer and then add a decorative element on top that has a small pointer (like a toothpick) pointing directly to the center of the bottom layer at **Table A**.
2. Now, an overzealous kitchen assistant moves the entire cake to **Table B** to make space.
3. The cake's structure is now at **Table B**, but the decorative toothpick is still pointing to the old location at **Table A**! The pointer is now **dangling**, pointing to empty space. If the chef tries to use that toothpick to check the bottom layer, they will find nothing, and the cake's integrity is compromised (undefined behavior).

This is exactly what can happen with `async` tasks in Rust. An `async fn` is compiled into a state machine (a `struct`) that can hold references to its own fields. If the `async` runtime moves this `struct` in memory, those internal references become invalid, leading to memory safety violations.[^3]

## What is `Pin`? The "Do Not Move" Sign

`Pin` is a smart pointer wrapper in Rust, provided by `std::pin::Pin`. It wraps another pointer (like `&mut T` or `Box<T>`) and provides a crucial guarantee to the compiler: **the value `T` that this pointer points to will *never* be moved again for the rest of its lifetime**.[^5]

- It's like putting a "Do Not Move This Cake" sign on the table.
- By "pinning" a value to its location in memory, we can safely create and work with self-referential structs, which are the backbone of `async`/`await`.

The most important place you'll see this is in the `Future` trait's `poll` method:

```rust
trait Future {
    type Output;
    // The `self` argument is not `&mut Self`, it's `Pin<&mut Self>`!
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

This signature is a contract: "You can only poll this future if you promise me it is pinned and will not be moved." The `async` runtime is responsible for upholding this promise before it calls `.poll()`.[^6]

## What is `Unpin`? The "It's Safe to Move" Sign

So if pinning is so important, why does most `async` code "just work" without you having to think about `Pin`? The answer is the `Unpin` trait.

`Unpin` is an auto-trait (like `Send` or `Sync`). The compiler automatically implements it for any type that is safe to be moved, even after being pinned.[^2]

- **What types are `Unpin`?** Almost everything! `i32`, `String`, `Vec<T>`, and most structs you define are `Unpin`. This is because they do **not** contain self-references. If you move a `String`, its internal pointer to the heap buffer moves with it, so everything stays valid.
- **What types are `!Unpin` (not `Unpin`)?** Types that are explicitly designed to be self-referential. You have to opt-out of `Unpin` manually, usually by including a special marker type like `std::marker::PhantomPinned`.

**The Mental Model:**

- `Unpin`: This dish (type) is robust. You can move it from table to table freely without it falling apart. It's "un-pinnable."
- `!Unpin`: This dish is a delicate, self-referential cake. Once you start working on it, you must **pin it down** and not move it.

Because most `async fn` you write only contain `Unpin` types, the resulting `Future` is also `Unpin`. This means the runtime can move it around freely without any danger, and you don't need to manually pin it.

You only need to care about `Pin` when you are either:

1. **Writing a `Future` manually** that contains self-references.
2. **Using a library** that gives you a `!Unpin` future.
3. **Getting a compiler error** that mentions `Unpin`.

## Practical Examples: When Pinning Becomes Necessary

### Scenario 1: The Compiler Error You Might Actually See

This is the most common way a beginner will encounter `Pin`. Let's say you try to use `tokio::select!` on a future that is `!Unpin`.

```rust
use tokio::select;

// Imagine some_library gives us a future that is !Unpin
// This means it's a self-referential future that cannot be moved safely.
struct SpecialFuture; // Let's pretend this is `!Unpin`

async fn run_special_future() {
    let my_future = SpecialFuture;

    // This code will NOT compile if `my_future` is `!Unpin`.
    // The `select!` macro needs to be able to move the futures it manages.
    // select! {
    //     _ = my_future => {}
    // }

    // The error would look something like:
    // `the trait "Unpin" is not implemented for "SpecialFuture"`
}
```

**How to Fix It: Pinning the Future**

To solve this, you need to pin the `!Unpin` future to a stable memory location. The most common way to do this is to put it on the heap using `Box::pin`.

```rust
use tokio::select;

struct SpecialFuture; // Pretend this is `!Unpin`

async fn run_special_future_fixed() {
    let my_future = SpecialFuture;

    // Pin the future to the heap. `Box::pin` creates a `Pin<Box<SpecialFuture>>`.
    // The `Box` ensures a stable memory address on the heap.
    // The `Pin` guarantees it won't be moved from that address.
    let pinned_future = Box::pin(my_future);

    // Now, `select!` can work with the pinned future.
    // We can safely get a mutable reference `&mut` to it because
    // `Pin<Box<T>>` allows this safely.
    select! {
        // _ = pinned_future.as_mut() => {}
    }
}
```

**`Box::pin(value)`** is the magic tool for taking a `!Unpin` value and putting it somewhere stable (the heap) so you can work with it safely.

### Scenario 2: Writing a Self-Referential Future Manually

This is an advanced topic, but seeing it makes the "why" of `Pin` click.

```rust
use std::future::Future;
use std::marker::PhantomPinned;
use std::pin::Pin;
use std::task::{Context, Poll};

/// A future that holds a value and a pointer to that value.
/// This is a classic self-referential struct.
struct SelfReferentialFuture {
    value: String,
    // This pointer will point to `self.value`. This is what makes it self-referential.
    pointer_to_value: *const u8,
    // This marker tells the Rust compiler: "This struct is !Unpin".
    _pin: PhantomPinned,
}

impl Future for SelfReferentialFuture {
    type Output = ();

    fn poll(self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<()> {
        // Because `self` is `Pin<&mut Self>`, we are guaranteed that the memory
        // location of `SelfReferentialFuture` will not change.

        // We need `unsafe` to get a mutable reference from a `Pin`.
        // This is us promising the compiler we won't move the data.
        let this = unsafe { self.get_unchecked_mut() };

        // We can now safely set our internal pointer, because we know
        // `this.value` will not be moved.
        this.pointer_to_value = this.value.as_ptr();

        println!("Polling future. Pointer is valid!");
        Poll::Ready(())
    }
}

#[tokio::main]
async fn main() {
    let my_future = SelfReferentialFuture {
        value: "hello".to_string(),
        pointer_to_value: std::ptr::null(),
        _pin: PhantomPinned, // Necessary for !Unpin
    };

    // We must pin it before we can `.await` it!
    let pinned_future = Box::pin(my_future);

    // The runtime can now poll our future safely.
    pinned_future.await;
}
```


## Summary for Beginners: The Key Takeaways

1. **What is the problem `Pin` solves?** `async` tasks can be moved around in memory by the runtime. If an `async` task contains pointers to its own data (self-references), moving it will break those pointers, causing undefined behavior.
2. **What is `Pin`?** It's a "Do Not Move" sign for data. `Pin` is a wrapper around a pointer that guarantees the data it points to will not be moved.
3. **What is `Unpin`?** It's a "Safe to Move" sign. Most types in Rust are `Unpin` by default, meaning they don't have self-references and are safe to move even if they are temporarily pinned. This is why you usually don't have to worry about `Pin`.
4. **When will I encounter this?** You will most likely encounter `Pin`/`Unpin` when you get a compiler error saying a type "does not implement `Unpin`." This usually happens when you are working with an advanced library or trying to do something complex with `select!` or storing futures in a `struct`.
5. **How do I fix it?** The most common solution is to pin the `!Unpin` value to the heap using **`Box::pin(your_future)`**. This gives it a stable memory address and wraps it in a `Pin`, satisfying the compiler.

While the details are complex, the day-to-day takeaway for most `async` Rust programmers is simple: if the compiler complains about `Unpin`, `Box::pin` is very likely your friend.
<span style="display:none">[^1][^4][^7][^8][^9]</span>

1. https://doc.rust-lang.org/book/ch17-05-traits-for-async.html
2. https://users.rust-lang.org/t/pin-unpin-explained/100612
3. https://rust-dd.com/post/async-rust-explained-pinning-part-2
4. https://without.boats/blog/pin/
5. https://blog.logrocket.com/pinning-rust-async-data-types-memory-safety/
6. https://rust-book.cs.brown.edu/ch17-05-traits-for-async.html
7. https://rust-lang.github.io/book/ch17-05-traits-for-async.html
8. https://www.reddit.com/r/rust/comments/1c0onwb/having_a_hard_time_understanding_pin_and_future/
9. https://leapcell.io/blog/async-programming-in-rust-futures-executors-and-task-scheduling
