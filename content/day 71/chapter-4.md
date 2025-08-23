---
title: "API Design Principles"
description: "Explore effective API design in Rust, focusing on ergonomics, clarity, and long-term maintainability."
date: 2025-11-10
weight: 4
---

# API Design Principles in Rust: A Guide for Crate Developers

Designing a high-quality Application Programming Interface (API) is one of the most important aspects of creating a successful Rust crate. A good API feels intuitive, guides users toward correct usage, and is a pleasure to work with. This is often referred to as **ergonomics**. In contrast, a poorly designed API can be confusing, error-prone, and a source of constant frustration.

This guide explores the core principles of effective API design in Rust, focusing on ergonomics, clarity, and long-term maintainability, with examples suitable for beginners.

## The "Well-Designed Kitchen Tool" Analogy 🔪

Think of your crate's API as a new kitchen tool you are designing.

- **A Good API (An Ergonomic Chef's Knife)**: It has a comfortable handle, a balanced weight, and a sharp blade. It feels natural to use, and its purpose is immediately obvious. It helps the chef work efficiently and safely.
- **A Bad API (A Poorly Designed Gadget)**: It's awkward to hold, its function is unclear, and it requires reading a long manual for a simple task. It's easy to use it incorrectly and cut yourself.

Our goal is to design the chef's knife, not the gadget.

## Core Principles of Ergonomic API Design

### 1. Make Correct Usage Easy and Incorrect Usage Hard

This is the most important principle of Rust API design. Leverage the type system to make invalid states unrepresentable.

**The "Typestate" Pattern**: Use different types to represent the different states of an object. This guides the user through a required sequence of steps at compile time.

**Anti-Pattern (Using a single, mutable struct):**

```rust
// ❌ This design allows the user to call `send()` before `connect()`.
pub struct BadConnection {
    is_connected: bool,
    // ...
}

impl BadConnection {
    pub fn connect(&mut self) { self.is_connected = true; }
    pub fn send(&self, data: &str) {
        if !self.is_connected {
            panic!("Not connected!"); // Runtime error!
        }
        // ... send data ...
    }
}
```

**Good Practice (Using the Typestate Pattern):**

```rust
// ✅ This design makes it impossible to call `send()` before `connect()`.
pub struct Disconnected; // A zero-sized "marker" struct for the initial state.

pub struct Connected {
    // ... connection details ...
}

impl Disconnected {
    pub fn connect(self) -> Result<Connected, std::io::Error> {
        // ... logic to connect ...
        Ok(Connected { /* ... */ })
    }
}

impl Connected {
    // The `send` method is only available on a `Connected` object.
    pub fn send(&self, data: &str) {
        // ... send data ...
    }
}

fn main() {
    let disconnected = Disconnected;
    // You CANNOT call .send() here. The compiler will stop you.
    let connected = disconnected.connect().unwrap();
    // NOW you can call .send().
    connected.send("Hello!");
}
```


### 2. Design for Clarity and Discoverability

Your API should be easy to understand and explore. Follow the official [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/checklist.html) for naming conventions.

- **Types and Traits**: `UpperCamelCase` (e.g., `HttpRequest`, `Future`).
- **Functions and Variables**: `snake_case` (e.g., `send_request`, `user_name`).
- **"Constructor" Functions**: Use `new()` for standard constructors. Use descriptive names like `from_path()` or `with_capacity()` for other ways of creating an object.
- **Getters**: Name methods that return a field `field_name()` (e.g., `user.name()`). If the operation is expensive, prefix it with `get_` (e.g., `get_user_from_database()`).
- **Boolean Logic**: Use prefixes like `is_`, `has_`, or `contains_` for methods that return a boolean (e.g., `is_empty()`, `has_permission()`).

**Builder Pattern for Complex Initialization**: If creating an object requires many configuration options, use the Builder pattern. It's more readable than a `new()` function with ten arguments.

```rust
// ✅ The Builder pattern is readable and flexible.
let request = HttpRequest::builder()
    .method("POST")
    .uri("https://example.com")
    .header("Content-Type", "application/json")
    .body("{'data': 'value'}")
    .build()?;
```


### 3. Use the Type System to Your Advantage

- **Borrow, Don't Own**: Accept slices (`&str`, `&[T]`) instead of owned types (`String`, `Vec<T>`) as function arguments whenever possible. This is more flexible for the user and avoids unnecessary allocations.
- **Return `Result` for Fallible Operations**: Never `panic!` on errors that are expected to happen (like I/O errors or parsing failures). Return a `Result<T, E>` to let the user decide how to handle the failure.[^1]
- **Use `Into<T>` for Ergonomic Arguments**: For functions that take a specific type (like a `PathBuf`), accepting `impl Into<PathBuf>` allows the user to pass in more convenient types (like `&str`), which will be converted automatically.

```rust
use std::path::{Path, PathBuf};

// This function is flexible. It can be called with a String, &str, or PathBuf.
pub fn process_file(path: impl Into<PathBuf>) {
    let path_buf: PathBuf = path.into();
    // ...
}
```


### 4. Design for Long-Term Maintainability

- **Keep Your Public API Small**: Only expose the types and functions that are essential for the user. Use `pub(crate)` or no `pub` for internal implementation details. A smaller public surface area is easier to maintain and less likely to have breaking changes.
- **Avoid Concrete Types in Public APIs**: Whenever possible, return an `impl Trait` instead of a specific, concrete type. This gives you the flexibility to change the internal implementation later without breaking your users' code.

**Anti-Pattern (Exposing an internal type):**

```rust
// ❌ The user is now coupled to the fact that you use a `Vec`.
// If you want to change to a `VecDeque` later, it's a breaking change.
pub fn get_all_users() -> Vec<User> {
    // ...
}
```

**Good Practice (Hiding the implementation detail):**

```rust
// ✅ The user only knows they get something they can iterate over.
// You can change the internal implementation freely.
pub fn get_all_users() -> impl Iterator<Item = User> {
    // ...
}
```

- **Document Everything**: Use `///` for doc comments on all public items. Explain *what* the item does, *why* it might be useful, and provide examples of how to use it. `cargo doc --open` is your best friend.


## A Final Checklist for Your Crate's API

Before you publish a new version of your crate, review your public API with these questions:

- **[Clarity]** Is the purpose of each function and type immediately obvious from its name?
- **[Ergonomics]** Does the API feel intuitive? Am I making the user think too much?
- **[Safety]** Does the type system prevent common mistakes and invalid states?
- **[Flexibility]** Am I using borrows (`&T`) and generics (`impl Trait`) where appropriate to give the user more options?
- **[Maintainability]** Have I hidden implementation details and kept the public API surface as small as possible?
- **[Documentation]** Is every public item documented with clear explanations and examples?

By following these principles, you can create Rust libraries that are not only powerful and performant but also a genuine pleasure for the community to use.

1. https://www.reddit.com/r/rust/comments/8asb4i/dark_side_of_ergonomics/
2. https://users.rust-lang.org/t/best-practices-to-design-a-c-api-that-can-be-ergonomically-used-in-rust/133151
3. https://blog.rust-lang.org/2017/03/02/lang-ergonomics.html
4. https://www.scribd.com/document/754922698/Rust-API-Guidelines
5. https://www.linkedin.com/pulse/building-scalable-modular-rust-api-axum-backend-developers-david-ak-csvlc
6. https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
7. https://internals.rust-lang.org/t/feedback-requested-book-on-api-type-patterns/13976
8. https://www.reddit.com/r/rust/comments/5xca4g/rusts_language_ergonomics_initiative_the_rust/
9. https://news.ycombinator.com/item?id=16836022
