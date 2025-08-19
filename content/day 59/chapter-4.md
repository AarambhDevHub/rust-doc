---
title: "No-std Programming in Rust"
description: "Learn how to write Rust programs without the standard library, useful for OS dev and embedded systems."
date: 2025-10-16
weight: 4
---

# No-std Programming in Rust: Building from the Ground Up

Understanding `no-std` (no standard library) programming in Rust is like learning the difference between building a house in a bustling city versus building a cabin in the wilderness.

- **`std` Programming (The City)**: You have a fully-stocked hardware store (`std`, the standard library) right next door. It provides everything you need off-the-shelf: plumbing (networking), electricity (threading), pre-built components (`Vec`, `String`, `HashMap`), and a construction crew (OS runtime) that knows exactly how to start building your `main` house. You rely on the city's infrastructure (the Operating System).
- **`no-std` Programming (The Wilderness)**: You are dropped in a forest with only a basic, high-quality toolkit (`core`, the core library). There is no hardware store, no power grid, and no construction crew. You have to build everything from the raw materials around you (the bare-metal hardware). You don't get a `main` house to start with; you have to designate the very first tree to chop down yourself (the entry point).

`no-std` programming is for environments where there is no underlying Operating System, such as embedded microcontrollers or when writing an OS kernel itself.

## What You Lose When You Go `no-std`

The standard library (`std`) is fundamentally an abstraction layer over operating system services. By choosing `#![no_std]`, you are telling the compiler, "I cannot assume an OS exists," and therefore, you lose access to all the features that depend on one :[^1]

1. **Heap Allocation**: This is the most significant change. There is no default memory allocator, so you cannot use `Box`, `Vec`, `String`, `HashMap`, or any other type that requires dynamic memory. You must work with fixed-size arrays and data structures on the stack.
2. **OS Services**:
    * **File I/O**: No `std::fs`. You can't read or write files.
    * **Networking**: No `std::net`. You can't open TCP or UDP sockets.
    * **Threading**: No `std::thread`. The concept of OS-managed threads doesn't exist.
3. **Standard Input \& Output**: The `println!` and `print!` macros are gone because there is no standard console to print to.
4. **A Default `main` Entry Point**: The `std` library provides a small runtime that sets up stacks, processes command-line arguments, and then calls your `main` function. Without it, you must define the program's true entry point yourself.[^2]
5. **Default Panic Handling**: When a program in a `std` environment panics, the runtime catches it and prints a backtrace to the console. In `no-std`, you must define what happens during a panic yourself.[^2]

## What's Left? The `core` Crate

You are not left with nothing. When you use `#![no_std]`, you still have access to the **`core` library**. `core` is the platform-agnostic, fundamental subset of `std`. It's your essential toolkit in the wilderness.[^3]

**`core` provides:**

- Primitive types like `i32`, `f64`, `bool`, and `char`.
- Essential enums like `Option<T>` and `Result<T, E>`.
- Iterators and the traits that power them.
- Core traits like `Copy`, `Clone`, `Send`, `Sync`, and `Debug`.
- Macros for control flow like `if`, `for`, and `match`.
- APIs for working with pointers and memory (`std::ptr`, `std::mem`).

In essence, you keep the heart of the Rust language, but lose all the parts that talk to an operating system.

## How to Write a `no_std` Program: A "Blinky" Example

Let's create the "Hello, World!" of embedded systems: a program that blinks an LED. This requires direct hardware manipulation and demonstrates the key components of a `no_std` application.

### Step 1: The Crate Attributes

Your `main.rs` (or `lib.rs`) file must start with these attributes:

```rust
// 1. Tell the compiler we won't be using the standard library.
#![no_std]
// 2. Tell the compiler we're defining our own entry point, not using the standard `main`.
#![no_main]
```


### Step 2: Define a Panic Handler

Since there's no OS to handle panics, you must link a panic handler. The simplest way is to use a crate that provides one. `panic-halt` simply puts the processor into an infinite loop if a panic occurs, which is a safe state.[^2]

**In `Cargo.toml`:**

```toml
[dependencies]
panic-halt = "0.2.0"
```

Your code doesn't need to do anything else; just including the crate is enough to provide the handler.

### Step 3: Define the Entry Point

Without the `std` runtime, we need to tell the processor where to start executing code. For ARM Cortex-M microcontrollers (the most common in the embedded Rust world), the `cortex-m-rt` crate provides an `#[entry]` attribute for this.[^2]

**In `Cargo.toml`:**

```toml
[dependencies]
cortex-m-rt = "0.7.3"
# Plus the HAL for your specific board
stm32f4xx-hal = { version = "0.17", features = ["stm32f401"] }
```


### Step 4: The Full "Blinky" Program

This example targets an STM32 "Blue Pill" board, a popular choice for beginners.

**`src/main.rs`**

```rust
// No standard library, no main function.
#![no_std]
#![no_main]

// Import the panic handler and entry point attribute.
use panic_halt as _;
use cortex_m_rt::entry;

// Import the Hardware Abstraction Layer (HAL) for our specific microcontroller.
// The HAL provides safe, high-level APIs to control the hardware.
use stm32f4xx_hal::{pac, prelude::*};

// This is the true entry point of our program.
#[entry]
fn main() -> ! {
    // Get a handle to the microcontroller's peripherals. This is a safe
    // way to access the hardware registers provided by the HAL.
    let dp = pac::Peripherals::take().unwrap();

    // Configure the clock system. We need to tell the microcontroller
    // how fast to run.
    let rcc = dp.RCC.constrain();
    let clocks = rcc.cfgr.sysclk(48.mhz()).freeze();

    // Get a handle to the GPIO port C, which is where the LED is connected.
    let gpioc = dp.GPIOC.split();

    // Configure pin PC13 as a push-pull output pin. This is our LED.
    let mut led = gpioc.pc13.into_push_pull_output();

    // Create a delay timer based on the system clock.
    let mut delay = dp.TIM1.delay_ms(&clocks);

    // The main loop. Since our function must never return (`-> !`),
    // we loop forever.
    loop {
        // Turn the LED on (by setting the pin low on this board)
        led.set_low();
        delay.delay_ms(500);

        // Turn the LED off (by setting the pin high)
        led.set_high();
        delay.delay_ms(500);
    }
}
```

This code, when compiled for the correct hardware target and flashed onto the microcontroller, will run directly on the "bare metal" without any OS involvement.

## The `no_std` Ecosystem: Rebuilding Abstractions

While you lose `std`, you are not alone. The embedded Rust community has built a rich ecosystem of crates to provide the functionality you need :[^4]

- **Hardware Abstraction Layers (HALs)**: Crates like `stm32f4xx-hal` provide safe APIs for controlling hardware, so you don't have to write `unsafe` code to manipulate memory-mapped registers directly.
- **The `alloc` Crate and Allocators**: If you need heap allocation, you can explicitly add the `alloc` crate and an allocator implementation (like `alloc-cortex-m`). This gives you back `Box`, `Vec`, and `String`.
- **Concurrency Frameworks**: Since `std::thread` is gone, concurrency is handled differently, often through interrupts. Frameworks like **RTIC** (Real-Time Interrupt-driven Concurrency) provide a safe and efficient way to build multi-tasking applications.
- **Drivers**: There are hundreds of platform-agnostic drivers for common sensors, displays, and other external components that are all designed to work in a `no_std` environment.

`no_std` programming represents a shift in mindset. It forces you to be acutely aware of memory, hardware, and every resource your program uses. It's a powerful paradigm that enables Rust to run on the smallest of devices, giving you performance and safety in environments where it was previously very difficult to achieve both.

1. https://docs.rust-embedded.org/book/intro/no-std.html
2. https://github.com/Hunterdii/30-Days-Of-Rust/blob/main/25_Rust%20on%20Embedded%20Systems/25_rust_on_embedded_systems.md
3. https://www.linkedin.com/learning/introduction-to-embedded-systems-with-rust/writing-your-first-program-using-no-std
4. https://github.com/rust-embedded/awesome-embedded-rust
5. https://users.rust-lang.org/t/how-to-use-the-std-crate-on-embedded-platforms/12512
6. https://hackmd.io/@alxiong/rust-no-std
7. https://docs.rust-embedded.org
8. https://blog.timhutt.co.uk/std-embedded-rust/index.html
