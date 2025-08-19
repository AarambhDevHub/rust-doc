---
title: "Async Fundamentals - Async vs threads"
description: "Compare async programming with traditional multi-threading, and learn when to use each approach."
date: 2025-09-15
weight: 5
---

# Async vs. Threads in Rust: A Beginner's Guide to Concurrency

Understanding the difference between `async` programming and traditional multi-threading in Rust is like learning to **manage the workflow of a professional kitchen** – you have two primary strategies for handling many tasks at once, and choosing the right one is crucial for efficiency. Do you hire more chefs to work in parallel, or do you have a single, highly efficient chef who can juggle multiple dishes at once? Both are valid approaches, but they excel in different scenarios.

## The Professional Kitchen Workflow Analogy 🍽️

### Imagine You're the Head Chef Deciding How to Handle a Rush of Orders

- **Threads**: Your team of chefs. Each chef is an independent worker (an OS thread).
- **Async Tasks**: A single, highly-skilled "super-chef" who can work on multiple dishes at once.
- **CPU Core**: A cooking station (e.g., a stovetop). You have a limited number of these.

***

### Strategy 1: Multi-Threading (Hiring More Chefs)

In this model, you assign **one chef to one dish**. If you have 10 orders, you hire 10 chefs.

- **How it works**: Each chef (thread) takes an order and works on it from start to finish. If they need to wait for something (e.g., water to boil), they are still occupying their station (CPU time and memory) and cannot do anything else.
- **Strengths**:
    * **Simple to reason about**: Each chef's work is straightforward and independent.
    * **Great for CPU-bound work**: If a dish requires intense, continuous chopping for 10 minutes, giving a dedicated chef to that task is the most efficient way to get it done.
- **Weaknesses**:
    * **Expensive**: Hiring a chef (spawning a thread) is costly in terms of time and resources. Each chef needs their own tools and space (stack memory, OS resources).[^1][^6]
    * **Scalability Limit**: You can't hire a million chefs to work in a normal-sized kitchen. Modern systems can handle tens of thousands of threads, but not millions.[^7]

**Code Analogy:**

```rust
use std::thread;
use std::time::Duration;

// Task: "Download" two different web pages (simulated with a sleep)
fn download(url: &str) {
    println!("Starting download from: {}", url);
    thread::sleep(Duration::from_secs(1)); // Simulate a blocking network request
    println!("Finished download from: {}", url);
}

fn main() {
    // We spawn a new OS thread for each download task
    let thread1 = thread::spawn(|| download("https://example.com"));
    let thread2 = thread::spawn(|| download("https://anothersite.com"));

    // Wait for both threads to finish
    thread1.join().unwrap();
    thread2.join().unwrap();
}
```


***

### Strategy 2: Asynchronous Programming (The Juggling Super-Chef)

In this model, you have **one super-chef (or a small team of them) who juggles many dishes simultaneously**.

- **How it works**: The chef starts preparing Dish A. When they get to a point where they have to wait (e.g., water boiling), they don't just stand there. They immediately put Dish A aside and start working on Dish B. When the water for Dish A boils, they are notified and can switch back to it at the next available moment. This is **non-blocking concurrency**.
- **Strengths**:
    * **Extremely Lightweight**: Starting a new "task" (deciding to work on another dish) is incredibly cheap. A single thread can manage hundreds of thousands or even millions of concurrent tasks.[^6]
    * **Great for I/O-bound work**: Perfect for tasks with a lot of waiting, like network requests, database queries, or reading files.[^2]
- **Weaknesses**:
    * **More Complex Code**: The "juggling" logic requires a special `async`/`await` syntax and an "async runtime" (like `tokio` or `async-std`) to manage the tasks.[^7]
    * **Not a Magic Bullet for CPU-bound Work**: If a task requires 10 minutes of non-stop chopping, the super-chef is still stuck on that one task and can't juggle anything else.

**Code Analogy:**

```rust
// Add `tokio` to your Cargo.toml: `tokio = { version = "1", features = ["full"] }`
use tokio::time::{sleep, Duration};

// The function is marked `async`, meaning it can be "paused" and "resumed"
async fn download_async(url: &str) {
    println!("Starting async download from: {}", url);
    // `sleep` is an async function. `.await` pauses this task,
    // allowing the runtime to work on other tasks.
    sleep(Duration::from_secs(1)).await;
    println!("Finished async download from: {}", url);
}

#[tokio::main]
async fn main() {
    // We define two "futures" (tasks to be done)
    let future1 = download_async("https://example.com");
    let future2 = download_async("https://anothersite.com");

    // `join!` runs both futures concurrently on a small number of threads
    tokio::join!(future1, future2);
}
```


## Detailed Comparison: Async vs. Threads

| **Feature** | **Multi-Threading** | **Asynchronous Programming (`async`/`await`)** |
| :-- | :-- | :-- |
| **Primary Use Case** | **CPU-bound tasks**: Heavy computations, data processing, video encoding, cryptography [^5]. | **I/O-bound tasks**: Web servers, database clients, file I/O, network services [^2][^3]. |
| **Concurrency Model** | **Preemptive Multitasking**: The Operating System decides when to switch between threads. This can happen at any time. | **Cooperative Multitasking**: Tasks explicitly yield control at `.await` points, giving the runtime a chance to run other tasks. |
| **Resource Cost** | **Heavyweight**: Each thread has its own stack and is managed by the OS. Spawning is slow (~17µs) [^1][^6]. | **Lightweight**: Tasks are managed by the async runtime within a few OS threads. Spawning is very fast (~0.3µs) [^6]. |
| **Scalability** | Scales to hundreds or thousands of concurrent tasks. Limited by OS thread limits [^7]. | Scales to hundreds of thousands or millions of concurrent tasks [^6]. |
| **Code Complexity** | **Simpler to write**: The programming model is linear and straightforward. No special syntax is needed [^2]. | **More complex**: Requires `async`/`await` syntax, Futures, and an async runtime (e.g., `tokio`). |
| **Context Switch Cost** | **High**: An OS context switch is relatively slow (~1.7µs) [^6]. | **Low**: Switching between async tasks is much faster (~0.2µs) as it happens within a single thread [^6]. |
| **Ecosystem** | Can use any standard library or crate without modification. | Requires `async`-compatible libraries (e.g., `reqwest` for HTTP, `sqlx` for databases). |

## How to Choose: A Practical Guide

Here’s a simple decision-making framework for your Rust project:

### 1. Is your task primarily **CPU-bound**?

- Are you doing intense calculations, processing large amounts of data in memory, or running complex algorithms?
- **YES**: **Use threads**. `std::thread` or a thread pool library like `rayon` is your best choice. The overhead of `async` will not help here and may even slow things down. Parallelism is what you need.


### 2. Is your task primarily **I/O-bound**?

- Is your program spending most of its time waiting for network requests, database queries, or file operations to complete?
- **YES**: **Use `async`/`await`**. This is the classic use case for `async`. It allows a single thread to manage thousands of waiting tasks efficiently, which is perfect for building high-performance network services.


### 3. Do you need to handle a **massive number of concurrent connections** (e.g., >10,000)?

- Are you building a web server, a chat application, or an IoT gateway that needs to handle C10K/C100K problem?
- **YES**: **Use `async`/`await`**. This is where `async` truly shines, as creating a thread for each connection is not feasible.


### 4. Is your code simple and you have a **small, fixed number of concurrent tasks**?

- Are you just trying to run a few background tasks simultaneously?
- **YES**: **Threads are often simpler**. If you only need to run 2-10 concurrent tasks and performance isn't absolutely critical, `std::thread` can be easier to write and reason about than setting up a full async runtime.[^2]


### 5. Can I use both? **YES!**

You don't have to choose just one. A very common and powerful pattern in Rust is to combine both approaches :[^5]

- Use an **`async` runtime** like `tokio` to handle the high-level I/O-bound tasks (like accepting web requests).
- When a request requires a heavy, blocking, CPU-bound computation, use `tokio::task::spawn_blocking` to move that specific computation onto a dedicated thread pool. This prevents the heavy work from blocking the entire async runtime.

This "best of both worlds" approach allows you to build applications that are both highly concurrent (handling many I/O tasks) and highly performant (efficiently executing CPU-bound work).
<span style="display:none">[^10][^4][^8][^9]</span>

1. https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html
2. https://www.reddit.com/r/rust/comments/jgpvi3/asyncawait_vs_threadsatomics_and_when_you_use_each/
3. https://stackoverflow.com/questions/78541829/async-thread-vs-std-thread
4. https://users.rust-lang.org/t/should-i-use-async-or-use-a-separate-thread/22770
5. https://doc.rust-lang.org/book/ch17-06-futures-tasks-threads.html
6. https://github.com/jimblandy/context-switch
7. https://corrode.dev/blog/async/
8. https://notgull.net/why-not-threads/
9. https://news.ycombinator.com/item?id=39812969
10. https://news.ycombinator.com/item?id=34567550
