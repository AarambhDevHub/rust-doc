---
title: "Async I/O - Stream Processing"
description: "Work with async streams in Rust to process data sequences in a non-blocking manner."
date: 2025-10-01
weight: 4
---

# Async I/O with Streams in Rust: A Beginner's Guide

Understanding async streams in Rust is like learning to **manage a continuous flow of ingredients arriving at a professional restaurant kitchen**. Instead of waiting for an entire delivery truck to be unloaded before you can start cooking (synchronous I/O), you have a system where individual boxes of ingredients (data chunks) are brought to you one by one. You can start working on the first box as soon as it arrives, and while you wait for the next one, you can work on other tasks. This non-blocking, item-by-item processing is the essence of async streams.

## The Professional Kitchen Ingredient Delivery Analogy 🍽️

### Imagine You're a Chef Waiting for a Delivery of Ingredients

- **A Traditional Iterator**: The entire delivery truck. You must wait for the whole truck to be unloaded before you can see what's inside.
- **An Async Stream**: A conveyor belt from the delivery truck to your station. Ingredients arrive one box at a time.
- **`Future<Item>`**: A single box of ingredients that will arrive at some point in the future.
- **`Stream`**: The entire sequence of boxes on the conveyor belt.

***

### The Problem with Blocking I/O (Waiting for the Whole Truck)

If you have to wait for the entire delivery of 1,000 kilograms of potatoes to be unloaded before you can start peeling the first one, your kitchen will be idle and inefficient. This is like a synchronous file read – your program blocks until the entire file is read into memory.

### The Power of Async Streams (The Conveyor Belt)

With a conveyor belt, a box of potatoes arrives at your station. You start peeling them. While you wait for the next box, you can work on another task, like chopping carrots. When the next box of potatoes arrives, the system notifies you, and you can resume peeling. This is **non-blocking, asynchronous data processing**.

**Why Async Streams Are Powerful:**

- **Responsiveness**: Your application doesn't freeze while waiting for data. It can work on other tasks.[^1]
- **Memory Efficiency**: You process data in chunks instead of loading entire large files or network responses into memory at once.
- **Scalability**: A single thread can efficiently manage thousands of concurrent data streams (e.g., thousands of network connections).


## What is a `Stream`?

A `Stream` is the asynchronous equivalent of Rust's standard `Iterator`. It represents a sequence of values that will become available over time.[^2][^4]


| **Concept** | **Synchronous (`Iterator`)** | **Asynchronous (`Stream`)** |
| :-- | :-- | :-- |
| **Get next item** | `next()` -> `Option<Item>` | `next().await` -> `Option<Item>` |
| **Behavior** | **Blocking**: Returns the next item immediately. | **Non-blocking**: Returns a `Future` that will resolve to the next item when it's ready. `.await`ing it allows other tasks to run while waiting. |

To work with streams, you'll need the `futures` or `tokio-stream` crate, which provides the `Stream` trait and the `StreamExt` trait for useful adaptor methods.

```toml
# In your Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
futures = "0.3"
tokio-stream = "0.1"
```


## How to Work with Streams: A Step-by-Step Example

Let's build a simple program that simulates receiving a stream of messages from a network connection and processes them as they arrive.

### 1. Creating a Stream

Manually implementing the `Stream` trait is complex. The easiest way to create a stream for demonstration is by using helpers from `tokio-stream` or the `async-stream` crate. Here, we'll convert an iterator into a stream.[^4][^2]

```rust
use tokio_stream::StreamExt;
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    // Create a simple stream that yields a number every 100 milliseconds
    let mut stream = tokio_stream::iter(vec![1, 2, 3, 4, 5])
        .throttle(Duration::from_millis(100)); // An adaptor to add a delay

    // ... we'll consume this stream next
}
```


### 2. Consuming a Stream with `while let`

The standard way to process items from a stream is to use a `while let` loop. Since there are no `async for` loops in Rust yet, this is the idiomatic pattern.[^4]

```rust
use tokio_stream::StreamExt;

async fn sum_stream(mut stream: impl Unpin + Stream<Item = i32>) -> i32 {
    let mut total = 0;
    while let Some(item) = stream.next().await {
        println!("Received item: {}", item);
        total += item;
    }
    total
}

#[tokio::main]
async fn main() {
    // Create a simple stream of numbers
    let stream = tokio_stream::iter(vec![10, 20, 30, 40]);
    let total = sum_stream(stream).await;
    println!("Total from stream: {}", total);
}
```

**Key Components:**

- `stream.next().await`: This is the core of stream consumption. It asynchronously waits for the next item. While it's waiting, other `async` tasks can run.
- `while let Some(item) = ...`: The loop continues as long as the stream yields items. When the stream is exhausted, `next()` returns `None`, and the loop terminates.
- `impl Unpin + Stream<Item = i32>`: This is a common function signature for accepting any type of stream. `Unpin` is a marker trait often required when working with `async` types.


### 3. Transforming Streams with Adaptors

Just like iterators, streams have powerful adaptor methods (provided by `StreamExt`) that let you build processing pipelines in a functional style.[^2]


| **Adaptor** | **Iterator Equivalent** | **Description** |
| :-- | :-- | :-- |
| **`map()`** | `map()` | Applies a function to each item in the stream. |
| **`filter()`** | `filter()` | Yields only the items that satisfy a predicate. |
| **`for_each()`** | `for_each()` | Consumes the stream, calling a function for each item (for side effects). |
| **`throttle()`** | N/A | Limits the rate at which items are produced by the stream. |

**Example: Building a processing pipeline.**

```rust
use tokio_stream::{self, StreamExt};
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    // 1. Create a source stream (e.g., notifications from a service)
    let notifications_stream = tokio_stream::iter(vec![
        "new_user:alice",
        "new_post:hello",
        "new_user:bob",
        "error:timeout",
        "new_user:charlie",
    ]);

    // 2. Build a processing pipeline using adaptors
    let processed_users_stream = notifications_stream
        .filter(|msg| msg.starts_with("new_user")) // Only keep new user notifications
        .map(|msg| msg.replace("new_user:", "")) // Extract the username
        .map(|name| name.to_uppercase()); // Convert the name to uppercase

    // 3. Consume the final stream
    // `pin_mut!` is often needed when using adaptors before a loop
    tokio::pin!(processed_users_stream);

    println!("Processing new user registrations:");
    while let Some(username) = processed_users_stream.next().await {
        println!("- Welcome, {}!", username);
    }
}
```

**Why `tokio::pin!`?** When you transform a stream, the compiler often can't guarantee its location in memory. Pinning a value "fixes" it to a specific memory location, which is a requirement for many `async` operations. `tokio::pin!` is a convenient macro for this.

## A Practical Real-World Example: Processing Lines from a File

Let's tie this all together by building a program that reads a large file line by line asynchronously, processes each line, and prints the result. This is a classic use case where streams prevent you from loading the entire file into memory.

```rust
use tokio::fs::File;
use tokio::io::{self, AsyncBufReadExt, BufReader};
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() -> io::Result<()> {
    // --- Setup: Create a dummy large file ---
    let filename = "large_log_file.txt";
    let mut file = File::create(filename).await?;
    for i in 0..1000 {
        file.write_all(format!("Log entry {}\n", i).as_bytes()).await?;
    }
    drop(file); // Ensure file is closed

    // --- Async Stream Processing ---
    println!("Starting to process file asynchronously...");

    // 1. Open the file asynchronously
    let file = File::open(filename).await?;

    // 2. Create a `BufReader`, which provides a `.lines()` method that returns a `Stream`
    let reader = BufReader::new(file);
    let mut lines_stream = reader.lines();

    // 3. Process the stream
    let mut line_count = 0;
    while let Some(line_result) = lines_stream.next().await {
        match line_result {
            Ok(line) => {
                // Simulate some processing
                if line.contains("77") {
                    println!("Found a special line: {}", line.to_uppercase());
                }
                line_count += 1;
            }
            Err(e) => eprintln!("Error reading line: {}", e),
        }
    }

    println!("\nFinished processing. Total lines read: {}", line_count);

    // --- Cleanup ---
    tokio::fs::remove_file(filename).await?;

    Ok(())
}
```

This example shows how to work with I/O-bound streams in a way that is both memory-efficient and non-blocking. While the `BufReader` is waiting for the next chunk of data from the disk, your `tokio` runtime is free to work on thousands of other `async` tasks.

## Summary and Key Takeaways

### **Mental Model: The Professional Kitchen Ingredient Conveyor Belt**

- **`Stream`**: The entire conveyor belt system, delivering a sequence of ingredients over time.
- **`Future<Option<Item>>`**: The promise that the *next* box of ingredients will eventually arrive on the belt.
- **`.next().await`**: The act of the chef waiting for the next box. Crucially, the chef can do other things while waiting.
- **Adaptors (`map`, `filter`)**: Specialized stations along the conveyor belt that transform or inspect each box as it passes by.
- **`while let Some(...)`**: The chef's work loop, which continues as long as boxes are arriving on the belt.


### **Key Concepts of Async Streams**

- **Asynchronous Iterators**: A `Stream` is conceptually an iterator where fetching the next item is an `async` operation.
- **Non-Blocking**: `.await`ing a stream's next item yields control back to the async runtime, allowing other tasks to run. This is the key to handling many I/O-bound tasks concurrently.[^1]
- **Functional Pipelines**: Use adaptors like `map` and `filter` from the `StreamExt` trait to build efficient, readable data processing pipelines.[^2]
- **Consumption**: The standard way to consume a stream is with a `while let Some(item) = stream.next().await` loop.[^4]


### **When to Use Async Streams**

- **Network I/O**: Reading data from a TCP or UDP socket, where data arrives in chunks over time.
- **File I/O**: Processing large files line by line without loading the entire file into memory.
- **Event Streams**: Handling events from a message queue (like RabbitMQ or Kafka), a WebSocket, or a GUI.
- **Database Queries**: Streaming rows from a database query result instead of fetching all of them at once.

By mastering async streams, you gain the ability to write highly scalable, responsive, and resource-efficient applications in Rust, especially for network services and data-intensive tasks.
<span style="display:none">[^3][^5][^6][^7][^8]</span>

1. https://rust-lang.github.io/async-book/
2. https://www.qovery.com/blog/a-guided-tour-of-streams-in-rust/
3. https://doc.rust-lang.org/book/ch17-00-async-await.html
4. https://tokio.rs/tokio/tutorial/streams
5. https://gendignoux.com/blog/2021/04/01/rust-async-streams-futures-part1.html
6. https://www.youtube.com/watch?v=0HwrZp9CBD4
7. https://rust-lang.github.io/async-book/05_streams/01_chapter.html
8. https://tokio.rs/tokio/tutorial
