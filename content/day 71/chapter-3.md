---
title: "Documentation Standards"
description: "Learn best practices for documenting Rust projects using rustdoc and guidelines for clear API documentation."
date: 2025-11-10
weight: 3
---

# Documentation Standards: A Beginner's Guide to `rustdoc` and Best Practices

In the Rust ecosystem, documentation is not an afterthought—it's a first-class citizen. Rust's built-in tooling, `rustdoc`, makes it easy to write, generate, and even test your documentation. This guide will walk you through the fundamentals of how `rustdoc` works and the best practices for creating clear, useful documentation for your projects.

## The "Library Tour Guide" Analogy 🏛️

Think of yourself as the tour guide for a great library (your Rust crate).

- **Your Code**: The books and shelves in the library.
- **Your Documentation**: You, the guide, explaining what each section contains, how to find specific books, and how to use the library's resources effectively.
- `rustdoc`: The tool that prints your tour script into beautiful, interactive brochures and signs for visitors.

A library with no guide is intimidating and difficult to use. A library with a great guide is welcoming and powerful. Your goal is to be a great guide for your code.

## How `rustdoc` Works: The Basics

`rustdoc` is a tool that parses your Rust source code and generates HTML documentation from special comments. It's fully integrated with Cargo, making it incredibly easy to use.[^1]

### Two Types of Documentation Comments

There are two primary ways to write documentation comments:

1. **Outer Documentation (`///`)**: Documents the **item that follows it**. This is the most common type, used for functions, structs, enums, etc.
2. **Inner Documentation (`//!` or `/*! ... */`)**: Documents the **item that contains it**. This is used for documenting modules or entire crates from within the file.
```rust
// in src/lib.rs

//! This is an **inner** comment that documents the entire crate.
//! It uses Markdown for formatting.

/// This is an **outer** comment that documents the `add_one` function.
/// It takes one number and returns that number plus one.
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```


### Generating and Viewing Your Documentation

The only command you need to know to get started is:

```bash
cargo doc --open
```

This command will:

1. Compile your documentation.
2. Generate an HTML website in the `target/doc` directory.
3. Automatically open it in your web browser.

What you'll see is a professional, searchable, and hyperlinked documentation site for your crate, all generated from your comments.

## Best Practices for Writing High-Quality Documentation

Good documentation is more than just explaining what a function does. It's about helping a user succeed with your code. Here are the most important practices to follow.[^2][^3]

### 1. The Crate-Level Summary: Your Front Page

Every library needs a good introduction. The first thing a user sees is your crate's front page. This documentation should live at the top of your `src/lib.rs` file, using **inner** `//!` comments.

**A good crate-level summary includes:**

- **A one-sentence pitch**: What does this crate do?
- **What problem it solves**: Why would someone want to use it?
- **A complete, copy-pasteable example**: Show a simple but common use case. This is the most important part!

```rust
// in src/lib.rs

//! # My Calculator Crate
//!
//! `my_calculator` is a collection of utility functions for performing
//! simple arithmetic. It is designed to be easy to use and very fast.
//!
//! ## Example
//! Here is a simple example of adding two numbers:
//! ```
//! use my_calculator::add;
//!
//! let result = add(2, 2);
//! assert_eq!(result, 4);
//! ```

// ... rest of the crate ...
pub fn add(a: i32, b: i32) -> i32 { a + b }
```


### 2. Document Every Public Item

Any part of your API that you expect users to interact with (a `pub` function, struct, enum, trait, or macro) should be documented. If it's public, it's part of your contract with the user.

- **For functions**: Explain what it does, its parameters, and what it returns.
- **For structs**: Explain the purpose of the struct and its important fields.
- **For enums**: Explain the purpose of the enum and what each variant represents.


### 3. Use the Standard Sections (`Examples`, `Panics`, `Errors`, `Safety`)

`rustdoc` recognizes special headings (using Markdown) to create standardized sections in your documentation. This makes it easy for users to find the information they need.

- `# Examples`: Provide one or more examples of how to use the item.
- `# Panics`: Clearly state any conditions under which the function will `panic!`.
- `# Errors`: If the function returns a `Result`, explain the different kinds of errors that can occur and what they mean.
- `# Safety`: For `unsafe` functions, this is the most critical section. You must explain the invariants that the caller is responsible for upholding to use the function safely.

**A Comprehensive Function Example:**

```rust
use std::num::ParseIntError;

/// Divides two numbers and returns the result.
///
/// This function provides a safe way to perform integer division.
///
/// # Examples
///
/// Basic usage:
/// ```
/// use my_crate::divide;
///
/// assert_eq!(divide(10, 2), Ok(5));
/// ```
///
/// # Errors
///
/// Returns an `Err` if the second number is zero, as division by
/// zero is not allowed.
///
/// ```
/// use my_crate::divide;
///
/// assert!(divide(10, 0).is_err());
/// ```
///
/// # Panics
///
/// This function does not panic.
pub fn divide(a: i32, b: i32) -> Result<i32, &'static str> {
    if b == 0 {
        Err("division by zero")
    } else {
        Ok(a / b)
    }
}
```


### 4. Your Examples Are Tests! (`doctests`)

One of `rustdoc`'s most powerful features is that it treats code blocks in your documentation as tests. When you run `cargo test`, Cargo will not only run your unit tests but will also **compile and run your documentation examples**.[^2]

This is a revolutionary feature because it guarantees that:

- Your documentation examples will always compile.
- Your examples will never be out of date.
- Your documentation is proven to be correct.

**Pro Tip**: You can hide lines from the final documentation output with `#`. This is useful for setup code that is necessary for the example to run but isn't relevant to the user.

```rust
/// # Examples
/// ```
/// # fn get_a_database_connection() -> i32 { 5 }
/// let db_conn = get_a_database_connection();
/// assert_eq!(db_conn, 5);
/// ```
```

In the rendered HTML, only the `let` and `assert_eq!` lines will be visible.

### 5. Use Intra-Doc Links

You can create links to other items in your documentation (or even in other crates) using a simple syntax. This helps users navigate your API and discover related functionality.

```rust
/// This function creates a [`Widget`].
///
/// It is often used with the [`process_widget`] function.
pub fn create_widget() -> Widget { /* ... */ }

/// Processes a [`Widget`].
pub fn process_widget(widget: Widget) { /* ... */ }

pub struct Widget;
```

When `rustdoc` renders this, `[`Widget`]` will become a hyperlink to the `Widget` struct's documentation page.

## The Goal of Good Documentation

Your documentation should answer three key questions for the user:

1. **What is it?** (A high-level summary)
2. **How do I use it?** (Clear, copy-pasteable examples)
3. **What do I need to worry about?** (Panics, errors, and safety considerations)

By following these standards, you can create documentation that not only explains your code but actively helps people use it successfully, making your projects more approachable and professional.

1. https://apidog.com/blog/rustdoc/
2. https://www.tangramvision.com/blog/making-great-docs-with-rustdoc
3. https://dev.to/gritmax/writing-rust-documentation-5hn5
4. https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
5. https://www.youtube.com/watch?v=G2_Tb2f2F1s
6. https://www.reddit.com/r/rust/comments/7eohmt/whats_the_best_practice_for_documenting_a_rust/
7. https://www.rust-lang.org/learn
8. https://rust-lang.github.io/api-guidelines/documentation.html
9. https://users.rust-lang.org/t/how-to-create-tutorial-guide-api-documentation-in-rust/77139
10. https://www.risein.com/blog/beginners-guide-to-learning-rust
11. https://blog.jetbrains.com/rust/2024/09/20/how-to-learn-rust/
12. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/documentation.html
13. https://doc.rust-lang.org/rust-by-example/meta/doc.html
14. https://news.ycombinator.com/item?id=22636282
