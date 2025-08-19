---
title: "Async main function"
description: "Understand how to define and run an async main function in Rust with Tokio."
date: "2025-09-15"
weight: 2
---

# Async Main Function in Rust with Tokio: A Beginner's Guide

Understanding how to define and run an `async` main function in Rust is like learning to **start the engine of a high-tech, automated restaurant kitchen**. A standard Rust `main` function is like a traditional kitchen that can only do one thing at a time. An `async` kitchen, however, is designed for massive concurrency—juggling hundreds of orders at once. But this advanced kitchen doesn't start on its own; you need a special "engine," called an **async runtime**, to power it up and manage all the concurrent tasks. Tokio provides the most popular and powerful engine for this job.

## The Automated Kitchen Analogy 🍽️

### Imagine You're Starting Up Your Restaurant for the Day

- **Standard `fn main()`**: A traditional kitchen. One chef, one task at a time. Simple, but not built for high volume.
- **`async fn main()`**: An ultra-modern, automated kitchen designed to handle thousands of concurrent orders. It has specialized stations for tasks that involve waiting (like baking or simmering).
- **The Async Runtime (Tokio)**: The central computer system and power grid for the automated kitchen. It schedules which dish gets worked on, manages all the timers and sensors, and ensures the kitchen is running at peak efficiency. **Without the runtime, the `async` kitchen is just a room full of unpowered equipment.**
- **`#[tokio::main]`**: The "master switch" that powers on the entire runtime and tells it to start processing the day's first order (the code inside your `async fn main`).


## Why Can't `main` Just Be `async`?

A standard Rust program needs a single, clear entry point that the operating system can call to start the program. This entry point, `fn main()`, is synchronous. It runs from top to bottom and then exits.

`async` functions, on the other hand, are different. They don't run on their own. When you call an `async` function, it returns a `Future`—a value that represents a computation that *might not be finished yet*. To actually run this `Future` to completion, you need a special scheduler called an **async runtime**. The runtime is responsible for:

- Polling `Future`s to see if they can make progress.
- Suspending tasks that are waiting for I/O (e.g., a network response) and running other tasks in the meantime.
- Waking tasks up when their I/O event is ready.

The `#[tokio::main]` attribute macro is the bridge that connects the synchronous world of the standard `fn main()` to the asynchronous world managed by the Tokio runtime.[^1]

## How to Write an `async` Main Function with Tokio

Writing an `async` main function is incredibly straightforward thanks to Tokio's powerful macros.

### Step 1: Add Tokio to Your `Cargo.toml`

First, you need to add the `tokio` crate as a dependency in your `Cargo.toml` file. The `full` feature flag enables all of Tokio's capabilities, including the `main` macro, which is a great starting point for beginners.[^6]

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```


### Step 2: Use the `#[tokio::main]` Macro

Now, you can write your `main` function with the `async` keyword and annotate it with `#[tokio::main]`.[^4][^1]

**File: `src/main.rs`**

```rust
use tokio::time::{sleep, Duration};

// This function is async, meaning it can be paused and resumed.
async fn prepare_dish(dish_name: &str) {
    println!("Chef has started preparing '{}'.", dish_name);
    // Use tokio's async-aware sleep function.
    // .await pauses this task, allowing the runtime to work on other tasks.
    sleep(Duration::from_millis(500)).await;
    println!("Chef has finished preparing '{}'!", dish_name);
}

#[tokio::main]
async fn main() {
    println!("Restaurant is open! The manager is giving out orders.");

    // Calling an async function returns a Future.
    // We need to `.await` it to run it to completion.
    prepare_dish("Paella").await;
    prepare_dish("Salad").await;

    println!("All orders for this sequence are complete.");
}
```


### How It Works: The Magic of the `#[tokio::main]` Macro

The `#[tokio::main]` attribute is a **procedural macro**. When you compile your code, this macro rewrites your `async fn main()` into a regular, synchronous `fn main()` that does all the setup work for you.[^5][^1]

What you write:

```rust
#[tokio::main]
async fn main() {
    println!("Hello, async world!");
}
```

What the compiler sees (conceptually):

```rust
fn main() {
    // 1. Create a new Tokio runtime instance.
    let runtime = tokio::runtime::Builder::new_multi_thread()
        .enable_all() // Enable I/O, timers, etc.
        .build()
        .unwrap();

    // 2. Use the runtime to execute your async code.
    //    `block_on` will run the future until it's complete,
    //    blocking the main thread in the process.
    runtime.block_on(async {
        println!("Hello, async world!");
    })
}
```

This powerful abstraction means you can start writing `async` code immediately without needing to manually configure and manage the runtime yourself.[^4]

## A More Realistic Example: Concurrent Downloads

The real power of `async` becomes clear when you need to perform multiple I/O-bound tasks concurrently. Let's simulate downloading two files at the same time.

```rust
use tokio::time::{sleep, Duration};
use std::time::Instant;

async fn download_file(filename: &str, duration_ms: u64) {
    println!("Starting download for: {}", filename);
    sleep(Duration::from_millis(duration_ms)).await;
    println!("Finished download for: {}", filename);
}

#[tokio::main]
async fn main() {
    let start = Instant::now();

    // --- The Sequential Way (for comparison) ---
    // download_file("report.pdf", 1000).await;
    // download_file("image.jpg", 1500).await;
    // This would take ~2500ms.

    // --- The Concurrent Way ---
    // Create two futures (tasks to be done).
    let future1 = download_file("report.pdf", 1000);
    let future2 = download_file("image.jpg", 1500);

    // Use `tokio::join!` to run both futures concurrently.
    // The runtime will juggle them, running them "at the same time".
    tokio::join!(future1, future2);

    let duration = start.elapsed();
    println!("Concurrent downloads finished in: {:?}", duration);
    // This will take ~1500ms, the time of the longest task.
}
```

In this example, `tokio::join!` tells the runtime to work on both `download_file` futures. When `future1` hits its `sleep(...).await`, the runtime pauses it and starts working on `future2`. It efficiently juggles these tasks until both are complete, dramatically reducing the total execution time compared to running them one after the other.

## Customizing the Tokio Runtime

While `#[tokio::main]` is great for most use cases, you can also customize its behavior. For example, you can choose between a multi-threaded runtime (the default) and a lighter-weight, single-threaded runtime.[^4]

- **Multi-Threaded Runtime (`multi_thread`)**: The default. It uses a pool of worker threads and can execute multiple tasks in parallel on different CPU cores. Best for general-purpose applications.
- **Current-Thread Runtime (`current_thread`)**: A single-threaded scheduler that runs all tasks on the same thread that the runtime was started on. It's lighter and faster if you don't need true parallelism, only I/O concurrency.

**Example: Using the current-thread runtime.**

```rust
#[tokio::main(flavor = "current_thread")]
async fn main() {
    println!("Running on the current-thread scheduler.");
    // ... your async code ...
}
```


## Summary and Key Takeaways

### **Mental Model: Starting the Automated Kitchen**

- **`async fn main()`**: The plan for your advanced, automated kitchen. It contains a list of concurrent tasks to be performed.
- **Tokio Runtime**: The powerful engine and central computer that runs the kitchen.
- **`#[tokio::main]`**: The "master on-switch" that powers up the Tokio runtime and tells it to start executing your `async main` plan.
- **`.await`**: A point in a recipe where a chef can pause and let the runtime work on another dish while waiting for something to finish (e.g., an oven to preheat).


### **Core Concepts**

1. **Rust's `fn main()` is Synchronous**: By default, Rust requires a standard, synchronous entry point for programs.
2. **`async` Functions Need a Runtime**: `async` code returns `Future`s, which must be managed and executed by an async runtime like Tokio.[^6]
3. **`#[tokio::main]` is the Bridge**: This attribute macro transforms your `async fn main()` into a synchronous `fn main()` that sets up and runs the Tokio runtime for you, making it easy to get started.[^1]

### **The Workflow for Beginners**

1. **Add Tokio**: Add `tokio = { version = "1", features = ["full"] }` to your `Cargo.toml`.
2. **Annotate `main`**: Write your `main` function as `async fn main()` and put `#[tokio::main]` on top of it.
3. **Write `async` Code**: Call and `.await` other `async` functions inside `main`.
4. **Run Concurrently**: Use utilities like `tokio::join!` or `tokio::spawn` to run multiple `async` tasks concurrently.

By following these simple steps, you can harness the power of asynchronous programming in Rust to build highly concurrent, scalable, and efficient applications, especially those that deal with a lot of I/O-bound work.
<span style="display:none">[^2][^3][^7][^8][^9]</span>

<div style="text-align: center">⁂</div>
1. https://tokio.rs/tokio/tutorial/hello-tokio
2. https://tokio.rs/tokio/tutorial/async
3. https://stackoverflow.com/questions/71116502/how-to-use-async-await-in-rust-when-you-cant-make-main-function-async
4. https://docs.rs/tokio/latest/tokio/attr.main.html
5. https://www.reddit.com/r/learnrust/comments/z17xib/question_how_does_tokio_allow_an_async_main/
6. https://rust-lang.github.io/async-book/part-guide/async-await.html
7. https://docs.rs/tokio
8. https://users.rust-lang.org/t/how-to-implement-async-await-in-main/38007
9. https://tokio.rs/tokio/topics/bridging
