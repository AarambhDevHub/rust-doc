---
title: "Async Fundamentals - Future trait"
description: "Learn about Rust’s Future trait, how it models values that are not immediately available, and its role in async programming."
date: 2025-09-15
weight: 2
---

# Async Fundamentals: The `Future` Trait in Rust

Understanding the `Future` trait in Rust is like learning about an **order ticket in a professional restaurant kitchen**. When a customer places an order, they don't get their food immediately. Instead, they get a ticket—a *promise* that their food will be ready in the future. They can check on this ticket periodically to see if the food is ready. This "order ticket" is exactly what a `Future` is in Rust: a placeholder for a value that is being computed asynchronously and will be available later.[^2]

## The Professional Kitchen Order Ticket Analogy 🍽️

### Imagine You're a Waiter Managing Orders

1. **Placing an Order (Creating a `Future`)**: A waiter takes an order and gives it to the kitchen. The kitchen gives back an order ticket (a `Future`). This ticket represents the "future meal."
2. **Checking on the Order (Polling the `Future`)**: The waiter doesn't just stand there staring at the kitchen. They go and do other things (like taking other orders). Periodically, they come back and ask the kitchen, "Is order \#123 ready yet?" This check is called **polling**.
3. **The Kitchen's Response (The `Poll` Enum)**: The kitchen can give one of two answers:
    * **`Poll::Pending`**: "Not yet, it's still cooking. Come back later." The waiter leaves and continues with other tasks. The kitchen promises to notify the waiter (via a `Waker`) when it's worthwhile to check again.
    * **`Poll::Ready(meal)`**: "Yes, here is the finished meal!" The order is complete. The waiter takes the meal (`Output`) and the ticket is fulfilled.

**Why This Model Is Powerful:**

- **Non-Blocking**: The waiter (the program's executor) is never idle. If one order isn't ready, it moves on to another task instead of blocking.[^8]
- **Efficiency**: The system can handle many orders (asynchronous operations) at once, making progress on whichever ones are ready.
- **State Management**: Each order ticket (`Future`) keeps track of its own progress (e.g., "chopping vegetables," "grilling steak").[^2]


## What is the `Future` Trait?

In Rust, a `Future` is any type that implements the `std::future::Future` trait. This trait is at the very core of Rust's `async/await` syntax. It tells the Rust compiler how to check on and make progress on an asynchronous task.[^1][^5]

The trait is surprisingly simple. It looks like this:

```rust
pub trait Future {
    // The type of value that this Future will produce when it's done.
    type Output;

    // The method that the async executor calls to "check on" the future.
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Let's break down the key parts:

- **`type Output`**: This is the value that the future will eventually produce. For a future that fetches a web page, the `Output` might be `String`. For a timer, it might be `()`.
- **`fn poll(...)`**: This is the most important method. The async runtime (the "waiter") calls `poll` repeatedly on a `Future` to drive it to completion.
    - `self: Pin<&mut Self>`: A pinned mutable reference to the future itself. Pinning is a complex topic, but for now, just know that it guarantees the future won't move in memory, which is crucial for safety in `async` code.[^1]
    - `cx: &mut Context<'_>`: The context of the current task. This is how the future communicates with the executor. Most importantly, it contains a `Waker`.
- **`Poll<Self::Output>`**: This is the `enum` that `poll` returns. It has two variants:
    - `Poll::Ready(value)`: The future is complete, and here is the final `Output` value.
    - `Poll::Pending`: The future is not ready yet. The executor should check back later.


### The Role of the `Waker`

When a `Future` returns `Poll::Pending`, it's making a promise: "I will notify you when you should poll me again." It does this by cloning a `Waker` from the `Context` and storing it. When the event it's waiting for happens (e.g., a network socket receives data), it calls `.wake()` on the stored `Waker`. This tells the executor to schedule the future to be polled again soon.[^1]

## `async/await`: The Magic That Creates Futures

You will rarely implement the `Future` trait by hand. Instead, the Rust compiler does the hard work for you when you use the `async` and `await` keywords.[^7][^2]

- **`async fn`**: When you declare a function with `async fn`, you are telling Rust: "This function doesn't return a value directly. Instead, it returns an anonymous struct that implements the `Future` trait".[^6]

```rust
// This function does NOT block and immediately returns a Future.
async fn fetch_web_page(url: &str) -> String {
    // ... logic to fetch the page asynchronously ...
    "<html>...</html>".to_string()
}

// Calling it creates a future but doesn't run it yet.
let page_future = fetch_web_page("https://example.com");
```

- **`.await`**: The `.await` keyword is used *inside* an `async` function to pause its execution until another `Future` is ready. This is a point where the function can "yield" control back to the executor, allowing other tasks to run.[^7]

```rust
async fn get_two_pages() {
    println!("Fetching page 1...");
    // .await pauses get_two_pages until the first page is ready
    let page1 = fetch_web_page("https://example.com").await;
    println!("Page 1 finished. Fetching page 2...");

    // .await pauses again for the second page
    let page2 = fetch_web_page("https://another.com").await;
    println!("Page 2 finished.");
}
```


### How the Compiler Translates `async fn`

Under the hood, the Rust compiler transforms an `async fn` into a **state machine**. This state machine is an `enum` that represents the different states of the asynchronous operation. Each `.await` point in your function becomes a new state in the enum.[^5][^6]

Consider this simple `async` function:

```rust
async fn get_user_and_friends() {
    let user = get_user_from_db().await;
    let friends = get_friends_for_user(user).await;
}
```

The compiler turns this into something conceptually like this:

```rust
// A simplified, conceptual view of the generated state machine
enum GetUserAndFriendsFuture {
    // State 0: We haven't started yet.
    Start,
    // State 1: We are waiting for the user from the database.
    WaitingForUser(impl Future<Output = User>),
    // State 2: We have the user and are waiting for their friends.
    WaitingForFriends(impl Future<Output = Vec<Friend>>),
    // State 3: We are done.
    Done,
}
```

When `poll` is called on this state machine, it checks its current state and tries to advance to the next one. If it has to wait for a sub-future (like `get_user_from_db`), it returns `Poll::Pending`. If it reaches the end, it returns `Poll::Ready`. This is the "magic" of `async/await`: it's just syntactic sugar for generating these complex state machines for you.[^5]

## A Complete Example: A Simple Timer `Future`

Let's manually implement a simple `Future` that completes after a certain amount of time. This will help solidify how `poll`, `Context`, and `Waker` work together.

**Add to `Cargo.toml`:**
You don't need any special dependencies for this example, as `Future` is in the standard library. However, to *run* a future, you need an executor. We'll use the one from the `futures` crate.

```toml
[dependencies]
futures = "0.3"
```

**The Code (`src/main.rs`):**

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll, Waker};
use std::thread;
use std::time::{Duration, Instant};

/// A Future that completes after a specified duration.
pub struct TimerFuture {
    duration: Duration,
    start_time: Option<Instant>,
}

impl TimerFuture {
    pub fn new(duration: Duration) -> Self {
        TimerFuture {
            duration,
            start_time: None,
        }
    }
}

impl Future for TimerFuture {
    type Output = (); // This future produces no value when it's done.

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // First time poll is called, record the start time.
        if self.start_time.is_none() {
            self.start_time = Some(Instant::now());
        }

        // Check if the required duration has elapsed.
        if self.start_time.unwrap().elapsed() >= self.duration {
            // Yes, the time has passed. The future is complete.
            println!("Timer finished!");
            Poll::Ready(())
        } else {
            // Not yet. We need to tell the executor to poll us again later.
            // We clone the Waker and ask a new thread to wake us up after a short time.
            let waker = cx.waker().clone();
            let duration = self.duration - self.start_time.unwrap().elapsed();

            thread::spawn(move || {
                thread::sleep(duration);
                waker.wake(); // "Wake up!" - tells the executor to poll this future again.
            });

            println!("Timer not finished yet. Will wake up later.");
            Poll::Pending
        }
    }
}

fn main() {
    println!("Setting a timer for 2 seconds...");
    let timer = TimerFuture::new(Duration::from_secs(2));

    // To run a future, we need an executor.
    // The `futures::executor::block_on` function is a simple one
    // that will block the current thread until the future is complete.
    futures::executor::block_on(timer);

    println!("Main function finished after timer completed.");
}
```

This example shows the fundamental polling mechanism. Real-world executors like Tokio are much more sophisticated and efficient, but the core principle is the same.

## Summary and Key Takeaways

- **A `Future` is a value that represents an asynchronous computation** that will produce a result at some point in the future.
- The `Future` trait has a single, crucial method: `poll()`. The `poll` method is how an async executor makes progress on the future.
- `poll()` can return one of two states:
    - `Poll::Ready(value)`: The future is done, and here is its output.
    - `Poll::Pending`: The future is not done yet. The executor should wait to be woken up before polling again.
- The `async/await` syntax is **syntactic sugar** that tells the Rust compiler to automatically generate a state machine that implements the `Future` trait for you.
- The `Future` itself doesn't do anything. It must be actively driven to completion by an **async executor** (like Tokio, async-std, or `futures::executor`). The executor is responsible for polling futures when they are ready to make progress.
<span style="display:none">[^3][^4]</span>

1. https://rust-lang.github.io/async-book/02_execution/02_future.html
2. https://doc.rust-lang.org/book/ch17-01-futures-and-syntax.html
3. https://www.youtube.com/watch?v=PabDPIrt9fk
4. https://doc.rust-lang.org/book/ch17-05-traits-for-async.html
5. https://tokio.rs/tokio/tutorial/async
6. https://www.eventhelix.com/rust/rust-to-assembly-async-await
7. https://www.freecodecamp.org/news/how-asynchronous-programming-works-in-rust/
8. https://rust-lang.github.io/async-book/
