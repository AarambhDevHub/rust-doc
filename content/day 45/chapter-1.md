---
title: "Day 45 (Friday): Concurrency Project"
description: "Build a multi-threaded file processor using the producer-consumer pattern, perform performance benchmarking, and debug race conditions."
date: 2025-09-12
weight: 1
---

# Concurrency Project: Building a Multi-Threaded File Processor in Rust

This project provides a comprehensive guide to building a multi-threaded file processor in Rust. We will use the **producer-consumer pattern** with channels to process multiple files concurrently, benchmark its performance against a single-threaded approach, and discuss how this pattern helps prevent and debug race conditions.

## The Professional Kitchen Workflow Analogy 🍽️

Imagine you're the head chef of a large restaurant that just received a big stack of order tickets (files). You need to process them as quickly as possible.

- **The Naive Approach (Single-Threaded)**: One chef reads an order, prepares it, and sends it out. They can only work on one order at a time. This is slow and inefficient.
- **The Concurrent Approach (Producer-Consumer)**:
    * **Producers (The Order Takers)**: You have several staff members (`producer` threads) whose only job is to read the order tickets (`files`) and put them onto a central conveyor belt (`channel`).
    * **Consumers (The Line Cooks)**: You have a team of chefs (`consumer` threads) waiting at the end of the conveyor belt. Their only job is to grab an order as it arrives, process it (e.g., count words, transform text), and finish it.
    * **The Conveyor Belt (The Channel)**: This system ensures that orders are passed safely from the takers to the cooks without anyone bumping into each other. It's the safe communication path.

This project will build that efficient kitchen workflow in Rust.

## Project Overview

**Goal**: To build a program that reads multiple text files, processes each line (e.g., converts it to uppercase), and measures the total time taken.

**Core Concepts Covered**:

1. **Multi-threading**: Using multiple threads to perform work in parallel.
2. **Producer-Consumer Pattern**: Decoupling the work of reading data (producing) from processing it (consuming).
3. **Channels (`mpsc`)**: Safely communicating data between threads.
4. **Synchronization (`Arc<Mutex<T>>`)**: Safely sharing state (the channel receiver) among multiple consumer threads.
5. **Performance Benchmarking**: Comparing the multi-threaded version to a single-threaded version to see the benefits of parallelism.
6. **Race Condition Prevention**: Understanding how channels and ownership prevent common concurrency bugs.

## The Complete Project Code

This single `main.rs` file contains the entire project, including dummy file creation for a self-contained example.

**File: `src/main.rs`**

```rust
use std::fs::{self, File};
use std::io::{self, BufRead, BufReader, Write};
use std::sync::{mpsc, Arc, Mutex};
use std::thread;
use std::time::Instant;

// --- Setup: Helper function to create dummy files for processing ---
/// Creates `num_files` dummy text files, each with `lines_per_file` lines.
/// Returns a Vec of the created filenames.
fn create_dummy_files(num_files: usize, lines_per_file: usize) -> io::Result<Vec<String>> {
    fs::create_dir_all("test_data")?; // Ensure the directory exists
    let mut filenames = Vec::new();
    for i in 0..num_files {
        let filename = format!("test_data/file_{}.txt", i);
        let mut file = File::create(&filename)?;
        for j in 0..lines_per_file {
            writeln!(file, "This is line {} from file {}", j, i)?;
        }
        filenames.push(filename);
    }
    Ok(filenames)
}

// --- The work to be done by a consumer on each line ---
/// A simple function that simulates processing a line of text.
fn process_line(line: &str) -> String {
    // In a real application, this could be complex parsing, computation, etc.
    line.to_uppercase()
}

// --- Single-Threaded Implementation for Benchmarking ---
fn single_threaded_processor(filenames: &[String]) -> (usize, usize) {
    let mut lines_processed = 0;
    let mut chars_processed = 0;
    for filename in filenames {
        let file = File::open(filename).unwrap();
        let reader = BufReader::new(file);
        for line in reader.lines() {
            let line = line.unwrap();
            let processed_line = process_line(&line);
            lines_processed += 1;
            chars_processed += processed_line.len();
        }
    }
    (lines_processed, chars_processed)
}

// --- Multi-Threaded Implementation (Producer-Consumer) ---
fn multi_threaded_processor(filenames: Vec<String>, num_consumers: usize) -> (usize, usize) {
    // 1. Create the channel (the "conveyor belt")
    let (tx, rx) = mpsc::channel::<String>();

    // We need to share the receiver among multiple consumers.
    // To do this safely, we wrap it in a Mutex (to ensure only one thread
    // accesses it at a time) and an Arc (to allow multiple threads to own it).
    let rx = Arc::new(Mutex::new(rx));

    // --- 2. Spawn Producer Threads ---
    let mut producer_handles = Vec::new();
    for filename in filenames {
        let tx = tx.clone(); // Clone the sender for each producer
        let handle = thread::spawn(move || {
            let file = File::open(filename).unwrap();
            let reader = BufReader::new(file);
            for line in reader.lines() {
                tx.send(line.unwrap()).unwrap();
            }
        });
        producer_handles.push(handle);
    }

    // Drop the original sender. The channel will close only when ALL senders
    // (including this one) are dropped. This is crucial for the consumers to know when to stop.
    drop(tx);

    // --- 3. Spawn Consumer Threads ---
    let mut consumer_handles = Vec::new();
    let total_lines = Arc::new(Mutex::new(0));
    let total_chars = Arc::new(Mutex::new(0));

    for _ in 0..num_consumers {
        let rx_clone = Arc::clone(&rx);
        let lines_clone = Arc::clone(&total_lines);
        let chars_clone = Arc::clone(&total_chars);

        let handle = thread::spawn(move || {
            loop {
                // Acquire a lock on the receiver to get the next message.
                // The `recv()` call will block until a message is available
                // or return an error if the channel is closed.
                let message = rx_clone.lock().unwrap().recv();

                match message {
                    Ok(line) => {
                        let processed_line = process_line(&line);
                        // Lock the shared counters to update them safely
                        *lines_clone.lock().unwrap() += 1;
                        *chars_clone.lock().unwrap() += processed_line.len();
                    }
                    Err(_) => {
                        // The channel is empty and all senders have been dropped.
                        // This consumer's work is done.
                        break;
                    }
                }
            }
        });
        consumer_handles.push(handle);
    }

    // --- 4. Wait for all threads to finish ---
    for handle in producer_handles {
        handle.join().unwrap();
    }
    for handle in consumer_handles {
        handle.join().unwrap();
    }

    // Return the final counts
    let lines = *total_lines.lock().unwrap();
    let chars = *total_chars.lock().unwrap();
    (lines, chars)
}

fn main() -> io::Result<()> {
    // --- Setup and Configuration ---
    let num_files = 10;
    let lines_per_file = 10_000;
    let num_consumers = 4; // Ideally, match the number of CPU cores

    println!("Setting up {} dummy files with {} lines each...", num_files, lines_per_file);
    let filenames = create_dummy_files(num_files, lines_per_file)?;
    println!("Setup complete.");

    // --- Performance Benchmarking ---

    // 1. Benchmark Single-Threaded Version
    println!("\n--- Running Single-Threaded Processor ---");
    let start_single = Instant::now();
    let (lines_single, chars_single) = single_threaded_processor(&filenames);
    let duration_single = start_single.elapsed();
    println!("Processed {} lines ({} chars).", lines_single, chars_single);
    println!("Time taken: {:?}", duration_single);

    // 2. Benchmark Multi-Threaded Version
    println!("\n--- Running Multi-Threaded Processor ({} Consumers) ---", num_consumers);
    let start_multi = Instant::now();
    let (lines_multi, chars_multi) = multi_threaded_processor(filenames.clone(), num_consumers);
    let duration_multi = start_multi.elapsed();
    println!("Processed {} lines ({} chars).", lines_multi, chars_multi);
    println!("Time taken: {:?}", duration_multi);

    // --- Performance Comparison ---
    println!("\n--- Performance Comparison ---");
    let speedup = duration_single.as_secs_f64() / duration_multi.as_secs_f64();
    println!("Multi-threaded version was {:.2}x faster than the single-threaded version.", speedup);

    // Clean up the dummy files
    fs::remove_dir_all("test_data")?;
    Ok(())
}
```


### How It Works: A Step-by-Step Breakdown

1. **Setup (`create_dummy_files`)**: The program starts by creating a `test_data` directory with several large text files. This gives us a realistic workload to process.
2. **The Channel (`mpsc::channel`)**: We create a channel, which gives us a `Sender` (`tx`) and a `Receiver` (`rx`). This is our "conveyor belt."
3. **Producers (The Order Takers)**:
    * We loop through our list of filenames and spawn a new thread for each one.
    * Each producer thread gets a **clone** of the `Sender` (`tx.clone()`).
    * The producer's job is simple: open a file, read it line by line, and send each line down the channel using `tx.send()`.
    * When a producer finishes its file, its thread terminates, and its `Sender` is dropped.
4. **Consumers (The Line Cooks)**:
    * We decide how many consumer threads we want (e.g., `num_consumers = 4`).
    * **Crucially**, we can't clone the `Receiver`. To share it among multiple consumers, we wrap it in `Arc<Mutex<...>>`.
        * `Arc` (Atomically Referenced Counter) allows multiple threads to have shared ownership of the receiver.
        * `Mutex` (Mutual Exclusion) ensures that only one consumer can try to `recv()` a message from the channel at a time, preventing data races on the receiver itself.
    * Each consumer thread runs an infinite loop. Inside the loop, it locks the mutex and calls `recv()`.
        * `Ok(line)` means it successfully received a message, which it then processes.
        * `Err(_)` means the channel is empty *and* all senders have been dropped. This is the signal for the consumer to break its loop and shut down.
5. **Synchronization and Shutdown**:
    * The `main` thread `drop(tx)`s its original sender.
    * The consumer loop `for msg in rx` (or `rx.recv()` in a `loop`) will only terminate when *all* `Sender` clones have been dropped. This provides a clean and automatic shutdown mechanism.
    * We use `.join()` on all thread handles to ensure the `main` function waits for all work to be completed before exiting.

### Performance Benchmarking

The `main` function demonstrates a simple but effective way to benchmark:

1. It runs the entire task using a simple, single-threaded `for` loop and measures the time with `std::time::Instant`.
2. It then runs the same task using our multi-threaded producer-consumer implementation and measures the time again.
3. By comparing the two durations, we can see the **speedup** provided by parallelism. For CPU-bound tasks like this, you should see a significant performance improvement on a multi-core machine.

### Race Condition Debugging and Prevention

This project is a masterclass in *preventing* race conditions by design.[^1]

- **What is a Race Condition?**: A bug that occurs when multiple threads try to access and modify shared data at the same time, leading to unpredictable and incorrect results. Imagine two chefs trying to update the total on the same order ticket simultaneously – the final number could be wrong.[^2]

**How Our Pattern Prevents Race Conditions**:

1. **Message Passing over Shared State**: The primary strategy here is to **avoid sharing mutable state** in the first place. Instead of having all threads work on a shared `Vec` of lines protected by a `Mutex`, we pass ownership of each `String` (line) through the channel. The compiler's ownership rules ensure that once a line is sent, the sender can no longer access it. This makes data races on the lines themselves impossible.[^3]
2. **Protecting the Single Point of Contention**: The only piece of state we *do* share is the channel's `Receiver`. If multiple consumers tried to call `.recv()` on it at the same time without protection, that would be a race condition. The solution is `Arc<Mutex<mpsc::Receiver>>`.
    * The `Mutex` acts as a lock, ensuring that only one consumer thread can "ask the conveyor belt for the next item" at any given moment. While this introduces a small point of contention, it's extremely fast and only locks for the brief moment it takes to receive a pointer to the next message. The actual processing of the message happens *outside* the lock, allowing consumers to work in parallel.
3. **Atomic Counters (Alternative for results)**: The example uses `Arc<Mutex<usize>>` to aggregate results. For simple counting, `Arc<AtomicUsize>` would be even more performant, as atomic operations avoid the overhead of OS-level locking.

By following this pattern, you are guided by the Rust compiler to write concurrent code that is free of data races by construction.

1. https://www.linkedin.com/pulse/understanding-race-conditions-preventing-them-rust-michael-putong-dq9nc
2. https://stackoverflow.com/questions/34510/what-is-a-race-condition
3. https://microsoft.github.io/rust-for-dotnet-devs/latest/threading/producer-consumer.html
4. https://users.rust-lang.org/t/producer-consumer-problem-in-rust/75987
5. https://www.w3resource.com/rust/channels/rust-message-passing-exercise-4.php
6. https://blog.softwaremill.com/multithreading-in-rust-with-mpsc-multi-producer-single-consumer-channels-db0fc91ae3fa
7. https://doc.rust-lang.org/book/ch16-02-message-passing.html
8. https://users.rust-lang.org/t/how-to-benchmark-multi-threaded-method-is-criterion-only-useful-for-single-threaded-code/52362
9. https://www.reddit.com/r/rust/comments/5w7fwo/whats_the_best_way_to_do_producer_consumer_in_rust/
10. https://docs.rs/many_cpus_benchmarking
11. https://stackoverflow.com/questions/65871238/multithreaded-performance-with-rust
