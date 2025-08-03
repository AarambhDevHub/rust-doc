+++
title = "Setting up Development (VS Code + Hello World)"
description = "Learn how to set up your Rust development environment using VS Code and rust-analyzer. We'll also write and run our first Rust program: Hello, World!"
date = 2025-08-01
draft = false
weight = 3
template = "page.html"
+++

# Setting up Development Environment (VS Code + rust-analyzer) and First "Hello, World!" Program

Now that you have Rust installed with rustup and cargo, let's set up a complete development environment using Visual Studio Code and create your first Rust program.

## Setting up VS Code with rust-analyzer

## 1. Install Visual Studio Code

If you haven't already, download and install [Visual Studio Code](https://code.visualstudio.com/) for your operating system.

## 2. Install the rust-analyzer Extension

The **rust-analyzer** extension is the official and recommended Rust extension for VS Code

. Here's how to install it:

1. Open VS Code
2. Press `Ctrl+Shift+X` (Windows/Linux) or `⇧⌘X` (macOS) to open the Extensions view
3. Search for "rust-analyzer"
4. Install the **Release Version** of the rust-analyzer extension

**Important**: There's an older deprecated Rust extension (`rust-lang.rust`) in the marketplace, but rust-analyzer is the recommended choice by rust-lang.org

.

## 3. Install Additional Helpful Extensions

For a complete Rust development experience, consider installing these optional extensions:

* **CodeLLDB**: For debugging Rust programs
* **Even Better TOML**: For better `Cargo.toml` file support
* **Dependi**: For crate dependency management and suggestions

## 4. Verify Your Setup

After installing the extensions, verify your Rust installation is working:

1. Open a new terminal in VS Code (`Ctrl+`` or `View > Terminal\`)
2. Run the following commands to confirm everything is installed:

```bash
rustc --version
cargo --version
rustup --version
```

You should see version information for each tool.



## Creating Your First "Hello, World!" Program

## Method 1: Using Cargo (Recommended)

Cargo is Rust's build system and package manager. It's the preferred way to create and manage Rust projects:

1. **Create a new project**:

```bash
cargo new hello_world
cd hello_world
```

1. **Open the project in VS Code**:

```bash
code .
```

1. **Examine the project structure**:
   1. `Cargo.toml`: Project configuration file
   2. `src/main.rs`: Your main Rust source file
2. **View the default code**: Open `src/main.rs` and you'll see:

```rust
fn main() {
    println!("Hello, world!");
}
```

1. **Run the program**:

```bash
cargo run
```

## Method 2: Direct Compilation (Alternative)

You can also create a simple Rust file and compile it directly:

1. **Create a Rust file**: Create a new file called `main.rs`.
2. **Write the Hello World code:**

```rust
fn main() {
    println!("Hello, World!");
}
```

1. **Compile the program**:

```bash
rustc main.rs
```

1. **Run the executable**:
   * On Linux/macOS: `./main`
   * On Windows: `main.exe`

## Understanding the Hello World Program

Let's break down what each part of the code does:

The main() Function:

```rust
fn main() {
```

* `fn` declares a function
* `main()` is the entry point of every Rust program - it's always the first code that runs
* The curly braces `{}` contain the function body.

The Print Statement

```rust
println!("Hello, World!");
```

* `println!` is a **macro**, not a function (notice the `!` at the end).
* Macros in Rust are used for metaprogramming and always end with `!`.
* The string `"Hello, World!"` is passed as an argument to the macro.
* The semicolon `;` ends the statement - required in Rust.



## Running and Debugging in VS Code

## Running Your Program

You have several options to run your Rust code in VS Code:

* Use the integrated terminal: `cargo run`
* Use the **Run** button that appears above the `main()` function
* Press `F5` to run with debugging

## Setting up Debugging

When you first try to debug a Rust program in VS Code

:

1. Put a breakpoint on any line by clicking in the left margin
2. Press `F5` or go to **Run > Start Debugging**
3. If prompted about launch configuration, click **Yes** to generate launch configurations for Cargo targets
4. VS Code will automatically create the necessary debugging configuration

## Expected Output

When you run your Hello World program, you should see:

```
Hello, World!
```



## VS Code Features with rust-analyzer

Once everything is set up, you'll have access to powerful development features:

* **Syntax highlighting** and **error detection**
* **Code completion** and **IntelliSense**
* **Type information** on hover
* **Go to definition** and **Find references**
* **Code formatting** with rustfmt
* **Linting** with clippy integration
* **Inline documentation**

## Troubleshooting

If rust-analyzer shows errors about "sysroot" or can't find a Rust workspace:

* Make sure you installed Rust using `rustup` (not package managers like `apt`)
* Ensure you're in a proper Cargo project directory with a `Cargo.toml` file
* Restart VS Code after installing Rust or the extension

Congratulations! You now have a complete Rust development environment set up with VS Code and have successfully created and run your first Rust program.
