---
title: "Introduction to Embedded Rust"
description: "Get started with Rust in embedded systems, using unsafe code responsibly for hardware interactions."
date: 2025-10-16
weight: 5
---

# Unsafe Abstractions: An Introduction to Embedded Rust

Getting started with embedded Rust is like learning to be an **astronaut landing on a new planet**. The standard support systems you're used to on Earth (like an operating system's standard library, `std`) are gone. You are in a `#[no_std]` environment. Your tools are limited to a "survival kit" (the `core` library), and you must interact directly with the raw, untamed environment (the hardware). To do this, you must occasionally step outside your pressurized rover (safe Rust) and manually operate the planet's machinery. This is where `unsafe` Rust becomes not just useful, but essential.

## The Astronaut on a New Planet Analogy 🧑🚀

- **Embedded System (Microcontroller)**: The new planet you've landed on. It has its own unique rules and physics.
- **Safe Rust**: The inside of your high-tech, safety-guaranteed rover. The compiler checks everything to make sure you can't hurt yourself.
- **`#[no_std]` Environment**: You can't call mission control (`std`) for help. You only have your personal survival kit (`core` library).[^1]
- **Hardware Peripherals (GPIO, Timers, etc.)**: The planet's mysterious control panels. They are powerful but have no safety labels.
- `unsafe` **Rust**: Stepping outside the rover in your spacesuit. You have the freedom to interact with the control panels directly, but you are now personally responsible for your own safety.[^2]


### Why is `unsafe` Necessary in Embedded?

The underlying computer hardware is inherently unsafe. The Rust compiler, in its mission to guarantee safety, cannot possibly know that writing the number `1` to the specific memory address `0x48001014` will turn on an LED. To the compiler, that's just a dangerous, arbitrary memory write.[^3]

`unsafe` is the keyword that lets you, the programmer, tell the compiler:
> "Trust me. I have read the planet's manual (the microcontroller's datasheet), and I know that this specific action is how you operate this specific control panel."

The fundamental goal of embedded Rust is to use these necessary `unsafe` operations to build **safe, high-level abstractions**. You step out of the rover, figure out how a dangerous panel works, and build a safe remote control for it so that you (and other users of your code) never have to go outside again.

## A Practical Example: Blinking an LED (The "Hello, World!" of Embedded)

Let's walk through the process of turning on a single LED on a microcontroller (like an STM32). This is the quintessential first step in embedded programming.

### Step 1: Read the "Planet's Manual" (The Datasheet)

Every microcontroller comes with a datasheet, a massive PDF that is the ultimate source of truth. It tells you the "magic addresses" for all the hardware peripherals. For blinking an LED, we need to interact with a **GPIO** (General-Purpose Input/Output) pin.

The datasheet might tell us:

1. To turn on the GPIO port, we must first enable its clock by writing to a register at address `0x4002104C`.
2. To set the pin as an output, we must write to a mode register at address `0x48001000`.
3. To set the pin's state to "high" (turning the LED on), we must write to an output data register at address `0x48001014`.

These addresses are our "control panel switches."

### Step 2: The Raw `unsafe` Approach (Stepping Outside the Rover)

To interact with these addresses, we must use **raw pointers**. Writing to a raw pointer is an `unsafe` superpower.

```rust
// These "magic" addresses come directly from the hardware datasheet.
const GPIOE_MODER: *mut u32 = 0x48001000 as *mut u32;
const GPIOE_ODR:   *mut u32 = 0x48001014 as *mut u32;
const RCC_AHB2ENR: *mut u32 = 0x4002104C as *mut u32;

fn turn_on_led_raw() {
    // We must wrap all direct hardware manipulation in an `unsafe` block.
    unsafe {
        // SAFETY: This is our promise to the compiler.
        // 1. The datasheet guarantees these addresses are valid for this hardware.
        // 2. We are writing specific bit patterns documented to configure the
        //    GPIO pin and set its state without causing side effects.

        // Enable the clock for GPIO Port E
        *RCC_AHB2ENR |= 1 << 4;

        // Configure Pin 8 of Port E as a general-purpose output
        *GPIOE_MODER &= !(0b11 << 16); // Clear bits 17 and 16
        *GPIOE_MODER |=   0b01 << 16; // Set to 01 for output mode

        // Set Pin 8 high to turn the LED on
        *GPIOE_ODR |= 1 << 8;
    }
}
```

This code works, but it's dangerous and hard to read. Anyone using it needs to understand the datasheet, and a typo in an address could cause catastrophic failure. This is not a reusable or safe way to write code.

### Step 3: Building a Safe Abstraction (The Remote Control)

Now for the most important part: we encapsulate this `unsafe` logic inside a safe, easy-to-use API.

```rust
use volatile::Volatile; // A crate to ensure writes are not optimized away

pub struct Led {
    // We store the pointer to the Output Data Register for this specific pin.
    output_register: Volatile<&'static mut u32>,
    pin_number: u8,
}

impl Led {
    /// Creates a new LED controller.
    ///
    /// # Safety
    /// The caller must ensure that the GPIO clock is enabled and the pin is
    /// configured as an output before calling this. This function still contains
    /// unsafe operations to create the initial object.
    pub unsafe fn new(port_address: usize, pin: u8) -> Self {
        Led {
            // Create a mutable reference from the raw address.
            // This is unsafe because we are claiming exclusive access to this memory.
            output_register: Volatile::new(&mut *(port_address as *mut u32)),
            pin_number: pin,
        }
    }

    /// Turns the LED on.
    pub fn turn_on(&mut self) {
        // This method is SAFE to call. The `unsafe` logic is hidden inside.
        // The internal logic is guaranteed to be correct by our implementation.
        self.output_register.update(|reg| *reg |= 1 << self.pin_number);
    }

    /// Turns the LED off.
    pub fn turn_off(&mut self) {
        self.output_register.update(|reg| *reg &= !(1 << self.pin_number));
    }
}

// --- Usage ---
fn use_safe_abstraction() {
    // Unsafe setup code still needs to be run once at the beginning.
    unsafe {
        // (Clock and mode configuration from before)
        *RCC_AHB2ENR |= 1 << 4;
        *GPIOE_MODER &= !(0b11 << 16);
        *GPIOE_MODER |=   0b01 << 16;
    }

    // Now we can create our safe LED object.
    let mut led = unsafe { Led::new(0x48001014, 8) };

    // From this point on, everything is 100% safe!
    // No more `unsafe` blocks are needed to control the LED.
    led.turn_on();
    // wait a bit...
    led.turn_off();
}
```

*Note on `volatile`*: When dealing with hardware registers, you must use volatile reads and writes (`read_volatile`, `write_volatile` or a crate like `volatile`). This tells the compiler that these memory locations can change unexpectedly, preventing it from optimizing away crucial reads or writes.

## The Result: A Zero-Cost Abstraction

We have successfully created a **zero-cost abstraction**. The safe `led.turn_on()` method compiles down to the exact same single machine instruction as the raw `unsafe` pointer write. We have lost no performance, but we have gained enormous safety and ergonomics.

This is the core philosophy of embedded Rust:

1. Dive into the `unsafe` world to understand and control the hardware.
2. Build a safe, high-level, zero-cost API around that `unsafe` core.
3. Do the vast majority of your application development using the safe API you created.

This approach allows Rust to provide the low-level control of C while still delivering the high-level safety guarantees that make it such a powerful and productive language, even on the bare metal of an embedded system.

1. https://www.trust-in-soft.com/resources/blogs/rusts-hidden-dangers-unsafe-embedded-and-ffi-risks
2. https://opentitan.org/book/doc/rust_for_c_devs.html
3. https://doc.rust-lang.org/book/ch20-01-unsafe-rust.html
4. https://www.sciencedirect.com/science/article/pii/S0167642325000206
5. https://arxiv.org/html/2311.05063v2
6. https://www.youtube.com/shorts/cFXX_A_-Lc4
7. https://www.embedded.com/memory-safety-in-rust/
8. https://docs.rust-embedded.org/book/concurrency/
9. https://www.reddit.com/r/rust/comments/1eaw93c/unsafe_rust_everywhere_really/
