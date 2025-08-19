---
title: "Advanced Async Patterns - Custom Futures"
description: "Explore how to implement your own futures and understand the inner workings of Rust’s async system."
date: 2025-10-02
weight: 3
---

# Advanced Async Patterns: Implementing Custom Futures in Rust

Understanding how to implement your own `Future` in Rust is like learning to **write your own recipe from scratch for a professional kitchen**. While most of the time you can use existing recipes (`async` functions provided by libraries), writing your own gives you ultimate control and a deep understanding of how the cooking process (the `async` system) actually works. It demystifies `async/await` by revealing the state machine at its core.

## The Professional Chef's Recipe Analogy 🍽️

### Imagine You're Writing a Recipe with Waiting Steps

- **A `Future`**: The recipe card itself. It's a plan for a dish that isn't ready yet.
- **The `poll` method**: You, the chef, checking on the recipe. "Is this step done yet?"
- **The `Poll` Enum**: The answer to your question:
    * `Poll::Pending`: "No, the water isn't boiling yet. Come back later."
    * `Poll::Ready(value)`: "Yes, the sauce has reduced! Here it is."
- **The `Waker`**: A kitchen timer. When you leave a step to wait, you set a timer. When it goes off, it "wakes" you up and tells you it's time to `poll` (check on) that recipe again.
- **The Executor**: The head chef (like `tokio`). They manage all the recipes (futures), and when a timer (`Waker`) goes off, they know to send a chef (`poll`) to check on that specific dish.
- **`Pin`**: A skewer that holds the ingredients for a dish in place on the counter. It guarantees that the half-finished dish won't be moved around, which could mess up the recipe.


## The Core of `async`: The `Future` Trait

At its heart, every `async` operation in Rust is a type that implements the `std::future::Future` trait. The `async/await` syntax is just convenient sugar for this.[^1]

The trait is surprisingly simple:

```rust
pub trait Future {
    type Output; // The value the future will produce when it's done

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

To create a custom future, you just need to implement this one method, `poll`.

### The `poll` Method: The Engine of `async`

The `poll` method is the heart of every `Future`. The async executor calls it repeatedly to drive the future towards completion. Your job when implementing `poll` is to:

1. Check if your task is complete.
2. If **yes**, return `Poll::Ready(your_output_value)`.
3. If **no**, arrange for the `Waker` (found inside the `Context`) to be called later when progress can be made, and then return `Poll::Pending`.

## Example 1: A Simple State-Machine Future

Let's build the simplest possible `Future`: one that completes after it has been polled a certain number of times. This demonstrates the basic state machine concept.

**File: `src/main.rs`**

```rust
// Add `futures` to Cargo.toml: `futures = "0.3"`
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

/// A custom future that will be ready after being polled twice.
struct ReadyAfterTwoPolls {
    polled_once: bool,
}

impl ReadyAfterTwoPolls {
    fn new() -> Self {
        ReadyAfterTwoPolls { polled_once: false }
    }
}

impl Future for ReadyAfterTwoPolls {
    type Output = &'static str;

    /// This is where the magic happens.
    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("Polling the future...");

        if self.polled_once {
            // We've been polled before, so we are ready now.
            println!("  -> Ready!");
            Poll::Ready("Future is complete!")
        } else {
            // This is the first time we're polled. We are not ready yet.
            self.polled_once = true; // Update our internal state
            println!("  -> Pending...");
            // We don't use the Waker here because we know the executor
            // will poll us again. In a real future, we would.
            Poll::Pending
        }
    }
}

#[tokio::main]
async fn main() {
    println!("Before awaiting the future...");

    let my_future = ReadyAfterTwoPolls::new();
    let result = my_future.await; // The executor will poll this for us

    println!("After awaiting the future...");
    println!("Result: {}", result);
}
```

**Running this code will produce:**

```
Before awaiting the future...
Polling the future...
  -> Pending...
Polling the future...
  -> Ready!
After awaiting the future...
Result: Future is complete!
```

This output clearly shows the executor polling our future twice. The first time, it returned `Pending`, and the second time, `Ready`.

## Example 2: A More Realistic `TimerFuture`

The previous example was simple but not very useful. A real `Future` usually waits for an external event (like a timer, a network packet, or a file I/O completion). To do this, it needs to use the `Waker` to tell the executor when to poll it again.

Let's build a `TimerFuture` that completes after a set duration.

**File: `src/main.rs`**

```rust
use std::future::Future;
use std::pin::Pin;
use std::sync::{Arc, Mutex};
use std::task::{Context, Poll, Waker};
use std::thread;
use std::time::Duration;

/// A future that completes after a specified duration.
pub struct TimerFuture {
    // We need shared state between the future and the timer thread.
    // Arc allows shared ownership, Mutex allows safe mutation.
    shared_state: Arc<Mutex<SharedState>>,
}

/// The state shared between the future and the waiting thread.
struct SharedState {
    completed: bool,
    waker: Option<Waker>,
}

impl TimerFuture {
    pub fn new(duration: Duration) -> Self {
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));

        // Clone the Arc to move it into the new thread.
        let thread_shared_state = Arc::clone(&shared_state);

        // --- This is the external event source ---
        thread::spawn(move || {
            println!("[Timer Thread] Waiting for {:?}...", duration);
            thread::sleep(duration);
            println!("[Timer Thread] Time's up!");

            let mut state = thread_shared_state.lock().unwrap();
            state.completed = true;
            // If the future has given us a waker, wake it up!
            if let Some(waker) = state.waker.take() {
                println!("[Timer Thread] Waking up the future...");
                waker.wake();
            }
        });

        TimerFuture { shared_state }
    }
}

impl Future for TimerFuture {
    type Output = &'static str;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("[Executor] Polling TimerFuture...");
        let mut state = self.shared_state.lock().unwrap();

        if state.completed {
            println!("[Executor]  -> Ready!");
            Poll::Ready("Timer is done!")
        } else {
            // The future is not ready yet. We must store the Waker so the
            // timer thread can wake us up when it's time.
            state.waker = Some(cx.waker().clone());
            println!("[Executor]  -> Pending. Stored waker.");
            Poll::Pending
        }
    }
}

#[tokio::main]
async fn main() {
    println!("[Main] Setting up a 2-second timer...");
    let timer = TimerFuture::new(Duration::from_secs(2));
    let result = timer.await;
    println!("[Main] Result: {}", result);
}
```

**How this works:**

1. **`TimerFuture::new()`**: Creates the shared state and spawns a new thread that will sleep for the specified duration.
2. **First `poll`**: The `tokio` executor polls our `TimerFuture`. The `completed` flag is `false`, so it stores a clone of the `Waker` in the shared state and returns `Poll::Pending`. The executor now puts this future to sleep.
3. **The Timer Thread**: After 2 seconds, the background thread wakes up, locks the shared state, sets `completed` to `true`, and calls `.wake()` on the stored `Waker`.
4. **Waking Up**: The `wake()` call signals to the `tokio` executor that our `TimerFuture` is ready to make progress. The executor schedules it to be polled again.
5. **Second `poll`**: The executor polls our future again. This time, `state.completed` is `true`, so it returns `Poll::Ready(...)`. The `.await` completes, and the program continues.

## A Note on `Pin<&mut Self>`: The "Do Not Move" Rule

You might have noticed the strange `self: Pin<&mut Self>` receiver in the `poll` method.

- **Why is it needed?** Many futures store pointers to their own internal data (they are "self-referential"). If the `Future` struct were moved in memory, these internal pointers would become invalid, leading to memory safety bugs.
- **What does `Pin` do?** `Pin` is a special wrapper that "pins" the `Future` to its location in memory. It guarantees to the compiler that the future's memory location will not change, making it safe to work with self-referential structs.

For beginners, you don't need to understand all the deep details of `Pin`. Just know that it's a safety mechanism required by the `Future` trait, and you must use it in the `poll` method signature.

## Summary and Best Practices

- **Futures are State Machines**: At their core, futures are state machines that are driven forward by an executor calling their `poll` method.
- **`Poll` is the Return Value**: The `poll` method must return `Poll::Pending` if work is ongoing, or `Poll::Ready(value)` when the work is complete.
- **The `Waker` is Key**: For any future that waits for an external event, it must store the `Waker` from the `Context` and call `.wake()` on it when it's ready to be polled again.
- **`Pin` Ensures Memory Safety**: `Pin` is a crucial safety feature that prevents self-referential futures from being moved.
- **You Rarely Need to Write Your Own `Future`**: The `async/await` syntax handles all of this for you. Implementing `Future` manually is an advanced pattern, primarily useful for library authors or for integrating with non-`async` systems. However, understanding how it works is invaluable for debugging and deeply understanding the `async` ecosystem.

By building your own custom futures, you pull back the curtain on Rust's `async/await` magic, revealing the elegant and powerful state machine system that makes modern, high-performance concurrency possible.
<span style="display:none">[^10][^11][^12][^2][^3][^4][^5][^6][^7][^8][^9]</span>

1. https://rust-lang.github.io/async-book/02_execution/02_future.html
2. https://doc.rust-lang.org/book/ch17-01-futures-and-syntax.html
3. https://stackoverflow.com/questions/74586510/implement-future-trait-based-on-future-available-inside-the-struct
4. https://www.youtube.com/watch?v=PabDPIrt9fk
5. https://leapcell.io/blog/async-programming-in-rust-futures-executors-and-task-scheduling
6. https://docs.rs/future-by-example
7. https://doc.rust-lang.org/book/ch17-03-more-futures.html
8. https://itnext.io/asyncwrite-and-a-tale-of-four-implementations-e63aef8397f7
9. https://dev.to/mindflavor/rust-futures-an-uneducated-short-and-hopefully-not-boring-tutorial---part-1-3k3
10. https://www.youtube.com/watch?v=eQkFrww3udQ
11. https://fasterthanli.me/articles/understanding-rust-futures-by-going-way-too-deep
12. https://github.com/rust-lang/rfcs/issues/2900
