---
title: "Conditional Compilation"
description: "Use conditional compilation with cfg attributes to build code differently depending on platform or features."
date: 2025-10-08
weight: 3
---

# Conditional Compilation in Rust: A Beginner's Guide to `cfg`

Understanding conditional compilation in Rust is like learning to **create a recipe book that automatically adapts to the kitchen it's in**. Imagine a recipe that says, "If you are in a French kitchen, use crème fraîche. If you are in an American kitchen, use sour cream." Conditional compilation allows you to write code that includes or excludes certain parts based on the "environment" it's being compiled for, such as the operating system, the computer's architecture, or custom features you define.

## The Adaptive Recipe Book Analogy 🍽️

### Imagine You're Writing a Recipe for a Global Audience

- **A `fn` or `struct`**: A specific step or ingredient list in your recipe.
- **A `#[cfg(...)]` attribute**: A special note to the chef (the compiler) that says, "Only include this next part if a certain condition is true."
- **The Compilation Target**: The kitchen where the recipe is being prepared (e.g., a Windows PC, a Linux server, a macOS laptop).

***

### The Problem without Conditional Compilation

If you wanted to support multiple platforms, you might have to maintain separate, nearly identical recipe books: one for Windows, one for macOS, and one for Linux. This is a maintenance nightmare.

### The Power of Conditional Compilation

With the `#[cfg]` attribute, you can have a single, unified recipe book. You annotate specific parts of the code to indicate where they should be used:

```rust
// A single, unified recipe (code file)
fn get_user_home_directory() -> String {
    // This part is only included if we are compiling for Windows
    #[cfg(target_os = "windows")]
    {
        return "C:\\Users\\...".to_string();
    }

    // This part is only included if we are compiling for a Unix-like system
    #[cfg(unix)]
    {
        return "/home/...".to_string();
    }
}
```

This is the core of conditional compilation: **writing code that adapts itself at compile time to different targets and features**. The unused code is not just ignored; it's as if it was never written in the source file, which means smaller, more efficient binaries.[^4][^5]

## How It Works: The `cfg` Attribute

The primary tool for conditional compilation is the `#[cfg(...)]` attribute. It can be applied to almost any item in Rust, including functions, structs, modules, and even individual expressions.[^3]

**Basic Syntax**:
`#[cfg(condition)]`

If the `condition` is true for the current compilation target, the item the attribute is attached to is included. If it's false, the item is completely removed from the code before the compiler does its main analysis.[^1]

### Common Conditions

Rust provides a number of built-in configuration options that you can check against.[^5]


| **Condition** | **Description** | **Example** |
| :-- | :-- | :-- |
| `target_os = "..."` | Checks the target operating system. Common values are `"windows"`, `"macos"`, `"linux"`, `"android"`. | `#[cfg(target_os = "windows")]` |
| `target_arch = "..."` | Checks the CPU architecture. Common values are `"x86_64"`, `"aarch64"`. | `#[cfg(target_arch = "aarch64")]` |
| `target_endian = "..."` | Checks the byte order (`"little"` or `"big"`). | `#[cfg(target_endian = "little")]` |
| `target_pointer_width = "..."` | Checks the pointer size (`"32"` or `"64"`). | `#[cfg(target_pointer_width = "64")]` |
| `unix` / `windows` | A convenient shorthand for checking if the target is a Unix-like system or a Windows system. | `#[cfg(unix)]` |
| `feature = "..."` | Checks if a specific feature flag has been enabled in `Cargo.toml`. This is user-defined. | `#[cfg(feature = "my_special_feature")]` |
| `test` | Checks if the code is being compiled for tests (i.e., with `cargo test`). | `#[cfg(test)]` |
| `debug_assertions` | Checks if debug assertions are enabled (the default for `cargo build`, but not for `cargo build --release`). | `#[cfg(debug_assertions)]` |

### Combining Conditions with `all`, `any`, and `not`

You can create more complex rules by combining conditions.[^1]

- **`all(...)`**: True only if *all* sub-conditions are true.
- **`any(...)`**: True if *at least one* sub-condition is true.
- **`not(...)`**: Inverts the result of a sub-condition.

**Example**:

```rust
// This function will only be included on 64-bit Linux or 64-bit macOS.
#[cfg(all(unix, target_pointer_width = "64", any(target_os = "linux", target_os = "macos")))]
fn for_modern_unix_desktops() {
    println!("Running on a 64-bit Linux or macOS system.");
}

// This function will be included on any system that is NOT Windows.
#[cfg(not(target_os = "windows"))]
fn for_non_windows_systems() {
    println!("This is not a Windows machine.");
}
```


## Practical Examples and Patterns

### 1. Platform-Specific Implementations

This is the most common use case. You have a function that needs to be implemented differently for different operating systems.

```rust
pub mod file_system {
    #[cfg(unix)]
    pub fn get_separator() -> char {
        '/'
    }

    #[cfg(windows)]
    pub fn get_separator() -> char {
        '\\'
    }
}

fn main() {
    println!("The path separator on this OS is: '{}'", file_system::get_separator());
}
```

In this example, only one of the `get_separator` functions will ever exist in the final compiled program.

### 2. Optional Features with Cargo

This is an extremely powerful pattern for library authors. You can allow users of your library to enable or disable certain features, which can reduce compile times and binary sizes.

**In your library's `Cargo.toml`:**

```toml
[package]
name = "my_image_library"
version = "0.1.0"
edition = "2021"

[features]
# By default, no extra image formats are supported.
default = []
# Users can opt-in to support for PNG and JPEG.
png_support = []
jpeg_support = []
```

**In your library's code:**

```rust
pub struct Image {
    // ... fields ...
}

impl Image {
    pub fn new() -> Self {
        // ...
    }

    #[cfg(feature = "png_support")]
    pub fn from_png(data: &[u8]) -> Self {
        println!("Parsing PNG data...");
        // ... PNG-specific logic here ...
        Image { /* ... */ }
    }

    #[cfg(feature = "jpeg_support")]
    pub fn from_jpeg(data: &[u8]) -> Self {
        println!("Parsing JPEG data...");
        // ... JPEG-specific logic here ...
        Image { /* ... */ }
    }
}
```

A user of your library can then choose what they need in their own `Cargo.toml`:

```toml
[dependencies]
my_image_library = { version = "0.1", features = ["png_support"] }
```

In this case, the `from_jpeg` function will not even be compiled into their program.

### 3. Including Debugging Code or Test Modules

The `#[cfg(test)]` and `#[cfg(debug_assertions)]` attributes are essential for separating test code and debug logic from your release builds.

```rust
pub fn add(a: i32, b: i32) -> i32 {
    #[cfg(debug_assertions)]
    {
        // This log message will only appear in debug builds.
        println!("Debugging: add({}, {})", a, b);
    }
    a + b
}

// The entire `tests` module is only compiled when running `cargo test`.
#[cfg(test)]
mod tests {
    use super::*; // Bring the parent module's items into scope

    #[test]
    fn it_works() {
        assert_eq!(add(2, 2), 4);
    }
}
```


### The `cfg!` Macro vs. the `#[cfg]` Attribute

It's important not to confuse `#[cfg]` with `cfg!`.

- **`#[cfg]` (attribute)**: **Removes code at compile time**. The code inside a `#[cfg(false)]` block doesn't even need to be valid Rust code (though it must be syntactically well-formed).
- **`cfg!` (macro)**: **Evaluates to `true` or `false` at compile time**. It's used within a function's code. The surrounding code (e.g., the `if` statement) must always be valid, regardless of whether the macro returns `true` or `false`.[^6]

**Example:**

```rust
fn show_info() {
    // Use the cfg! macro to get a boolean value at compile time.
    if cfg!(target_os = "linux") {
        println!("Running on Linux!");
    } else {
        println!("Not running on Linux.");
    }

    // This is different from:
    // #[cfg(target_os = "linux")]
    // println!("Running on Linux!");
    // The line above would not compile on Windows, because the `else` case is missing.
}
```

By mastering conditional compilation, you gain fine-grained control over your code, allowing you to build highly optimized, platform-aware, and feature-rich applications and libraries in Rust.

1. https://doc.rust-lang.org/reference/conditional-compilation.html
2. https://stackoverflow.com/questions/60287822/can-i-create-my-own-conditional-compilation-attributes
3. https://doc.rust-lang.org/rust-by-example/attribute/cfg.html
4. https://masteringbackend.com/posts/cfg-conditional-compilation-in-rust
5. https://docs.wasmtime.dev/contributing-conditional-compilation.html
6. https://labex.io/tutorials/rust-conditional-compilation-with-rust-s-cfg-attribute-99342
7. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/conditional-compilation.html
8. https://dev-doc.rust-lang.org/beta/reference/conditional-compilation.html
9. https://www.geeksforgeeks.org/rust/rust-configuration-conditional-checks/
10. https://bitshifter.github.io/2020/05/07/conditional-compilation-in-rust/
