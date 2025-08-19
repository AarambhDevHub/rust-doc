---
title: "Async Fundamentals - Async functions and blocks"
description: "Dive into async functions and blocks, understanding their structure and how they return Futures."
date: 2025-09-15
weight: 4
---

# Async Fundamentals in Rust: `async` Functions and Blocks

Understanding `async` fundamentals in Rust is like learning to **manage a professional kitchen where chefs can multitask efficiently**. Instead of a chef blocking an entire stovetop while waiting for water to boil (synchronous execution), they can start the water boiling, move on to chopping vegetables, and come back to the pot only when it's ready. This non-blocking approach is the core of `async` programming in Rust. It allows a single thread to handle many tasks concurrently by never waiting idly.

## The Professional Multitasking Chef Analogy 👨🍳

### Imagine You're a Chef Who Never Wastes Time

**The Old Way (Synchronous Blocking):**

```rust
// ❌ The blocking chef - one task at a time
fn make_dinner_synchronously() {
    // 1. Boil water (5 minutes) - The chef stands and waits, doing nothing else.
    //    The entire kitchen (thread) is blocked.
    boil_water();

    // 2. Chop vegetables (2 minutes) - Only starts after water is boiled.
    chop_vegetables();
}
// Total time: 7 minutes. The chef was idle for 5 minutes.
```

**The `async` Way (Asynchronous Non-Blocking):**

```rust
// ✅ The multitasking async chef
async fn make_dinner_asynchronously() {
    // 1. Start boiling water and get a "promise" (a Future) that it will be done later.
    let water_handle = boil_water_async(); // Returns immediately

    // 2. While the water is boiling, chop the vegetables. The kitchen (thread) is productive.
    chop_vegetables();

    // 3. Now, wait (`.await`) for the water to finish boiling.
    //    If it's not ready, the chef can yield and let other tasks run.
    water_handle.await;

    // Both tasks were worked on concurrently.
}
// Total time: ~5 minutes. The chopping was done *during* the water boiling.
```

**Why `async` Is a Game-Changer:**

- **Efficiency**: Allows a single thread to make progress on many tasks at once, dramatically improving throughput for I/O-bound workloads (like web servers or databases).[^4]
- **Responsiveness**: Prevents long-running operations (like network requests) from blocking the entire application.
- **Ergonomics**: `async/.await` syntax makes writing complex, non-blocking code look and feel almost like simple, synchronous code.[^5]


## What Are `async` Functions and Blocks?

In Rust, the `async` keyword is used to create asynchronous operations. It can be applied in two ways: `async fn` and `async` blocks.[^7]

### The Key Concept: They Return a `Future`

The most important thing to understand is that calling an `async` function or entering an `async` block does **not** execute the code inside it. Instead, it instantly returns a value that represents the work to be done later. This value is called a **`Future`**.[^1]

A `Future` is like a ticket you get from a deli counter. You have a promise that you'll eventually get your sandwich, but you don't have it yet. The `Future` is a placeholder for a value that will be computed in the future.[^5]

### 1. `async fn`: Asynchronous Functions

When you add `async` before a function definition, you are creating an asynchronous function.

- **What it does**: It changes the function's return type. Instead of returning `T`, it returns a `Future<Output = T>`.
- **Behavior**: The body of the function is not executed until the returned `Future` is `.await`ed.[^4]

**Example:**

```rust
// This function does NOT return a `String`.
// It returns a `Future` that will produce a `String` when it's done.
async fn fetch_data_from_db() -> String {
    println!("(Starting to fetch data from the database...)");
    // In a real app, this would involve a real database call.
    // We'll simulate a delay.
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    println!("(...Finished fetching data!)");
    "some data from the database".to_string()
}

// To run this, you need an async runtime like Tokio.
// Add `tokio = { version = "1", features = ["full"] }` to your Cargo.toml
#[tokio::main]
async fn main() {
    println!("Calling the async function...");
    let data_future = fetch_data_from_db(); // This returns a Future instantly. No code inside runs yet.
    println!("The async function returned a Future. Now we will .await it.");

    // The .await keyword is what actually runs the Future.
    // It will pause execution here and allow other tasks to run.
    // Once the Future is complete, it will resume.
    let data = data_future.await;

    println!("The awaited result is: '{}'", data);
}
```

**Running the Example:**
You will see that the "Calling..." and "The async function returned..." messages print immediately. Then there will be a 1-second pause while the `Future` is awaited, during which the "(Starting..." and "...Finished" messages will print.

### 2. `async` Blocks: Asynchronous Expressions

`async` blocks allow you to create a `Future` from an arbitrary block of code. They are useful when you want to create a `Future` inside a regular (non-`async`) function or when you want to move data into a `Future`.

- **What it does**: An `async { ... }` block captures variables from its environment and returns a `Future` that will execute the code inside the block when `.await`ed.

**Example:**

```rust
use std::time::Duration;

fn get_two_pieces_of_data() -> (impl std::future::Future<Output = String>, impl std::future::Future<Output = u32>) {
    let user_name = "Alice".to_string();

    // Create a Future using an `async` block.
    // The `move` keyword transfers ownership of `user_name` into the Future.
    let future1 = async move {
        println!("(Future 1 starting...)");
        tokio::time::sleep(Duration::from_secs(1)).await;
        println!("(Future 1 finished!)");
        format!("Hello, {}", user_name)
    };

    // Create another independent Future.
    let future2 = async {
        println!("(Future 2 starting...)");
        tokio::time::sleep(Duration::from_secs(1)).await;
        println!("(Future 2 finished!)");
        42
    };

    (future1, future2)
}

#[tokio::main]
async fn main() {
    let (f1, f2) = get_two_pieces_of_data();

    // We can run these futures concurrently!
    let (result1, result2) = tokio::join!(f1, f2);

    println!("Result 1: {}", result1);
    println!("Result 2: {}", result2);
}
```


## How `async` Works Under the Hood (A Simplified View)

The `async` keyword is "syntactic sugar." The Rust compiler takes your `async` code and transforms it into something much more complex: a **state machine**.[^8][^1]

- An `async fn` is compiled into a struct that implements the `Future` trait.
- This struct holds all the local variables of your function.
- Each `.await` point in your code becomes a state in the state machine.
- The **async runtime** (like Tokio or `async-std`) has an **executor** that repeatedly calls a `.poll()` method on your `Future`.
    - If the `Future` is waiting for something (like a network response), `.poll()` returns `Poll::Pending`. The executor then knows it can put this task aside and work on another one.
    - Once the resource is ready, the executor polls the `Future` again. This time, it can make progress past the `.await` point.
    - When the `Future` is completely finished, `.poll()` returns `Poll::Ready(value)`, and the result is available.

This polling mechanism is what allows a single thread to juggle thousands of concurrent tasks without blocking.

## The Role of the Async Runtime

A key point in Rust is that the language provides the `async/.await` syntax, but it **does not** provide a built-in async runtime. You must choose one from the ecosystem.[^5]

- **What is a runtime?**: It's the library that contains the **executor** (which runs the `Future`s), I/O utilities (for async networking, file system access), and task scheduling logic.
- **Popular Runtimes**:
    * **`tokio`**: The most popular, battle-tested runtime. Excellent for networking applications.
    * **`async-std`**: Aims to provide an `async` version of the standard library.
    * **`smol`**: A smaller, simpler runtime.

You enable a runtime by adding it to your `Cargo.toml` and using its entry point macro (e.g., `#[tokio::main]`).

## Summary and Key Takeaways

### **Mental Model: The Multitasking Chef**

- **`async` Code**: A recipe designed for multitasking.
- **`async fn` or `async {}`**: Writing down a part of the recipe. This doesn't cook anything; it just creates a "to-do" card.
- **`Future`**: The "to-do" card itself. It represents a task that can be started, paused, and resumed.
- **`.await`**: The point where the chef checks a "to-do" card. If the task (e.g., water boiling) isn't finished, the chef leaves it and picks up another card (another task).
- **Async Runtime (e.g., Tokio)**: The head chef who manages all the "to-do" cards, deciding which chef works on which task at any given moment to keep the kitchen as busy and efficient as possible.


### **Core Concepts**

| **Concept** | **Description** | **Key Point** |
| :-- | :-- | :-- |
| **`async fn`** | A function that returns a `Future` instead of a direct value. | The function body is not executed until the returned `Future` is awaited [^4]. |
| **`async {}`** | A block of code that evaluates to a `Future`. | Useful for creating `Future`s on the fly or moving data into them. |
| **`Future`** | A trait representing a value that may not be ready yet. It's a placeholder for a future result. | Futures are lazy; they do nothing until they are polled by an executor [^5]. |
| **`.await`** | The operator used to pause the current task and wait for a `Future` to complete. | It is non-blocking. It yields control of the thread so other tasks can run [^7]. |
| **Runtime** | The library (like Tokio) that schedules and executes `Future`s. | Rust does not have a built-in runtime; you must choose one. |

By understanding that `async` code creates `Future`s (promises of work) and that `.await` is the mechanism to pause and resume that work, you can begin to write highly efficient, non-blocking, and concurrent applications in Rust.
<span style="display:none">[^2][^3][^6][^9]</span>

1. https://www.eventhelix.com/rust/rust-to-assembly-async-await
2. https://doc.rust-lang.org/book/ch17-00-async-await.html
3. https://rust-lang.github.io/async-book/
4. https://www.freecodecamp.org/news/how-asynchronous-programming-works-in-rust/
5. https://dev.to/leapcell/asyncawait-in-rust-a-beginners-guide-237h
6. https://blog.ediri.io/how-to-asyncawait-in-rust-an-introduction
7. https://rust-lang.github.io/async-book/03_async_await/01_chapter.html
8. https://tokio.rs/tokio/tutorial/async
9. https://www.youtube.com/watch?v=NSgyNb0egm4
