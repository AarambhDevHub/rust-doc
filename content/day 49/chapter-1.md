---
title: "Advanced Async Patterns - Async Traits"
description: "Learn how to define and use async traits to abstract asynchronous behavior in Rust."
date: 2025-10-02
weight: 1
---

# Advanced Async Patterns: Async Traits in Rust

Understanding `async` traits in Rust is like learning to **create a universal remote control for all the different appliances in a professional kitchen**. You want a single, consistent interface (a `trait`) to control various devices (a microwave, a sous-vide machine, a convection oven), even though each one operates asynchronously and takes a different amount of time. Async traits allow you to define a common set of `async` behaviors that different types can implement, enabling powerful, generic, and abstract asynchronous code.

## The Professional Kitchen Universal Remote Analogy 🍽️

### Imagine You're Designing a "Smart Kitchen" System

- **The Goal**: Create a universal `start_cooking` button that works for every appliance.
- **The Appliances**: A microwave (fast), a convection oven (medium), and a smoker (very slow). Each one is an `async` operation.
- **The `trait`**: A `Cook` trait with an `async fn cook()` method.
- **The Challenge**: Before Rust 1.75, you couldn't just put `async fn` in a trait directly. It was like having a remote control blueprint but no way to build a button that could handle the different "waiting" times of each appliance.

**The Problem without Async Traits:**
How do you write a function that takes *any* cookable appliance and starts it? Without a common `async` trait, you'd need separate logic for each type of appliance, which isn't generic or scalable.

**The Power of Async Traits:**
With `async` traits, you can define a single, elegant interface.

```rust
// The "Universal Remote" blueprint
#[async_trait::async_trait] // We'll discuss this macro later
trait Cook {
    async fn cook(&self, dish: &str);
}

// Now you can write generic functions that work with *any* appliance
async fn start_any_appliance(appliance: &impl Cook, dish: &str) {
    println!("Starting to cook {}...", dish);
    appliance.cook(dish).await; // Just call .cook() and await it!
    println!("Finished cooking {}!", dish);
}
```


## The Evolution of Async Traits in Rust

Writing `async fn` in traits has historically been one of Rust's most complex challenges. Its evolution can be broken down into three main approaches :[^8]

1. **The "Old" Manual Way (Associated `Future` Type)**: The original workaround, requiring complex boilerplate.
2. **The `async-trait` Crate**: A macro-based solution that makes it ergonomic but adds some overhead.
3. **The "New" Native Way (Stabilized in Rust 1.75)**: The modern, built-in solution, which is still evolving.

Let's explore each one.

### 1. The Manual Way: Associated Types and `Box<dyn Future>`

Before language support existed, the pattern was to define a trait method that *returned a `Future`* manually. This required boxing the `Future` and using dynamic dispatch (`dyn`).[^2][^8]

- **How it Works**: The trait method returns `Pin<Box<dyn Future<Output = ...> + Send>>`. `async` blocks are used to create the future, which is then "boxed" (allocated on the heap) and "pinned" (fixed in memory).
- **Pros**: It works on all Rust versions.
- **Cons**: Verbose, hard to read, and introduces heap allocation and dynamic dispatch overhead for every call.

**Example:**

```rust
use std::future::Future;
use std::pin::Pin;

// This is the manual, verbose way. You generally won't write this today.
trait CookManual {
    fn cook<'a>(&'a self, dish: &'a str) -> Pin<Box<dyn Future<Output = ()> + Send + 'a>>;
}

struct Microwave;
impl CookManual for Microwave {
    fn cook<'a>(&'a self, dish: &'a str) -> Pin<Box<dyn Future<Output = ()> + Send + 'a>> {
        Box::pin(async move {
            println!("Microwave: Cooking {} fast!", dish);
            tokio::time::sleep(std::time::Duration::from_secs(1)).await;
        })
    }
}
```

This pattern is complex and mostly of historical interest now, but it helps understand *why* simpler solutions were needed.

### 2. The `async-trait` Crate: The Ergonomic Workaround

The `async-trait` crate was created to solve the ergonomic nightmare of the manual approach. It provides a simple attribute macro, `#[async_trait]`, that automatically transforms your `async fn` into the complex `Pin<Box<...>>` signature behind the scenes.[^5][^6]

- **How it Works**: The macro rewrites the `async fn` in your trait and `impl` blocks into a regular function that returns a boxed `Future`.
- **Pros**: Very easy to write and read. Works on stable Rust. Supports `dyn Trait` objects out of the box.
- **Cons**: Still has the same performance overhead as the manual method (heap allocation and dynamic dispatch for every call) because it generates similar code.[^5]

**Example:**

```rust
// Add `async-trait = "0.1.77"` to your Cargo.toml
use async_trait::async_trait;
use tokio::time::{sleep, Duration};

// The Universal Remote Blueprint
#[async_trait]
trait Cook {
    async fn cook(&self, dish: &str);
}

// --- Appliance Implementations ---
struct Microwave;
#[async_trait]
impl Cook for Microwave {
    async fn cook(&self, dish: &str) {
        println!("Microwave: Cooking '{}' for 1 second...", dish);
        sleep(Duration::from_secs(1)).await;
    }
}

struct Oven;
#[async_trait]
impl Cook for Oven {
    async fn cook(&self, dish: &str) {
        println!("Oven: Cooking '{}' for 5 seconds...", dish);
        sleep(Duration::from_secs(5)).await;
    }
}

// --- Generic Function Using the Trait ---
async fn start_cooking_job(appliance: &impl Cook, dish: &str) {
    println!("Starting job for '{}'...", dish);
    appliance.cook(dish).await;
    println!("Job for '{}' is complete!", dish);
}

// --- Using Trait Objects with `dyn Trait` ---
async fn start_all_appliances(appliances: Vec<Box<dyn Cook + Send + Sync>>, dish: &str) {
    for appliance in appliances {
        // We can call .cook() on a `dyn Cook` object thanks to async-trait
        appliance.cook(dish).await;
    }
}

#[tokio::main]
async fn main() {
    let microwave = Microwave;
    let oven = Oven;

    // Using the trait with static dispatch (generic function)
    println!("--- Static Dispatch Example ---");
    start_cooking_job(&microwave, "Popcorn").await;
    start_cooking_job(&oven, "Roast Chicken").await;

    // Using the trait with dynamic dispatch (`dyn Trait`)
    println!("\n--- Dynamic Dispatch with `dyn Trait` ---");
    let kitchen_appliances: Vec<Box<dyn Cook + Send + Sync>> = vec![
        Box::new(Microwave),
        Box::new(Oven),
    ];
    start_all_appliances(kitchen_appliances, "Leftovers").await;
}
```

For years, `async-trait` has been the standard, idiomatic way to work with async traits in Rust.

### 3. Native `async fn` in Traits (Stabilized in Rust 1.75+)

As of Rust 1.75 (released in December 2023), you can now write `async fn` directly in traits without any macros. This is a huge step forward for the language.[^1][^7]

- **How it Works**: The compiler now understands `async fn` in traits and desugars it to a method that returns `impl Future`. This avoids the heap allocation (`Box`) of `async-trait`, leading to better performance (static dispatch).
- **Pros**:
    * No external dependencies needed.
    * Zero-cost abstraction: no heap allocation or dynamic dispatch when used with generics (`impl Trait`).
- **Current Limitations (as of late 2023/early 2024)**:
    * **No `dyn Trait` Support**: You cannot yet create a trait object (like `Box<dyn Cook>`) from a native async trait. This is the biggest limitation and the main reason why `async-trait` is still widely used.[^6][^7]
    * `?Send` bounds are not yet supported for non-thread-safe futures.

**Example (using native async fn):**

```rust
// This requires Rust 1.75 or later
use tokio::time::{sleep, Duration};

// No `#[async_trait]` macro needed!
trait NativeCook {
    async fn cook(&self, dish: &str);
}

struct Smoker;
impl NativeCook for Smoker {
    async fn cook(&self, dish: &str) {
        println!("Smoker: Slowly cooking '{}' for 10 seconds...", dish);
        sleep(Duration::from_secs(10)).await;
    }
}

// This generic function works perfectly with native async traits
async fn start_native_cooking_job(appliance: &impl NativeCook, dish: &str) {
    println!("Starting native job for '{}'...", dish);
    appliance.cook(dish).await;
    println!("Native job for '{}' is complete!", dish);
}

#[tokio::main]
async fn main() {
    let smoker = Smoker;
    start_native_cooking_job(&smoker, "Brisket").await;

    // The following would NOT compile with native async traits yet:
    // let appliances: Vec<Box<dyn NativeCook>> = vec![Box::new(smoker)];
}
```


## Summary: Which Approach Should You Use?

Here is a simple guide for choosing the right approach for your project today:


| **Scenario** | **Recommended Approach** | **Reasoning** |
| :-- | :-- | :-- |
| **You need to use your trait as a trait object (`dyn Trait`)** | Use the **`async-trait` crate**. | This is currently the only stable and ergonomic way to get `dyn Trait` support for async methods. It's the standard solution for cases where you need runtime polymorphism [^7]. |
| **You only need static dispatch (using generics like `impl Trait`)** | Use **native `async fn` in traits** (if on Rust 1.75+). | This is the future of Rust. It offers better performance (no allocations) and requires no dependencies. Use this whenever you don't need `dyn Trait`. |
| **You are writing a library for others to use** | **Consider `async-trait`** for your public API. | To provide maximum flexibility to your users, who might need to create trait objects from your traits, `async-trait` is still the safest bet for library authors until `dyn Trait` support is stabilized [^6]. |

The Rust language is actively working on full `dyn` support for native async traits, at which point `async-trait` will likely become less necessary. But for now, understanding both is key to navigating the modern `async` Rust ecosystem.
<span style="display:none">[^3][^4][^9]</span>

1. https://blog.rust-lang.org/2023/12/21/async-fn-rpit-in-traits.html
2. https://stackoverflow.com/questions/65921581/how-can-i-define-an-async-method-in-a-trait
3. https://doc.rust-lang.org/book/ch17-05-traits-for-async.html
4. https://rust-lang.github.io/async-book/07_workarounds/05_async_in_traits.html
5. https://docs.rs/async-trait
6. https://www.reddit.com/r/rust/comments/1k2r19x/does_rust_still_require_the_use_of_asynctrait/
7. https://google.github.io/comprehensive-rust/concurrency/async-pitfalls/async-traits.html
8. https://fasterthanli.me/articles/catching-up-with-async-rust
9. https://rust-book.cs.brown.edu/ch17-05-traits-for-async.html
