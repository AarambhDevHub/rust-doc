+++
title = "Understanding Cargo Project Structure"
description = "Learn about the standard Cargo project layout in Rust, including files like Cargo.toml, src/main.rs, and how to organize large applications with examples, tests, and benchmarks."
date = 2025-08-01
draft = false
weight = 4
template = "page.html"
+++

# Understanding Cargo Project Structure

Now that you've created your first Rust project with `cargo new hello_world`, let's explore the **standard Cargo project structure** and understand what each file and directory is for.

## Basic Project Structure

When you run `cargo new hello_world`, Cargo creates a standardized directory layout that follows Rust community conventions:

```
hello_world/
├── Cargo.toml
├── Cargo.lock
└── src/
    └── main.rs
```

## Core Files

**Cargo.toml** - This is the **project manifest file** that contains all metadata Cargo needs to compile your project. It includes:

* Package information (name, version, edition)
* Dependencies
* Build configuration
* Author information

**Cargo.lock** - This file **locks specific versions** of dependencies to ensure reproducible builds. It's automatically generated and updated by Cargo.

**src/main.rs** - This is the **crate root** for binary crates and serves as the entry point for your application.

## Complete Project Layout

For more complex projects, Cargo supports this comprehensive directory structure:

```
project_name/
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs              # Default library file
│   ├── main.rs             # Default executable file
│   └── bin/                # Additional executables
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/                # Benchmarks
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/               # Example programs
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/                  # Integration tests
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

## Directory Purposes

## src/ Directory

The **src** directory is where all your source code lives:

* **src/main.rs** - Entry point for **binary crates** (applications)
* **src/lib.rs** - Entry point for **library crates** (reusable code)
* **src/bin/** - Additional executable files that become separate binaries

## Other Directories

**tests/** - Contains **integration tests** that test your crate from the outside, as if it were an external dependency

**examples/** - Contains **example programs** that demonstrate how to use your crate. These can be run with `cargo run --example example_name`.

**benches/** - Contains **benchmark tests** for performance testing using tools like Criterion.

**target/** - This directory is **automatically created** when you build your project and contains compiled artifacts:

```
target/
├── debug/          # Debug builds (cargo build)
└── release/        # Release builds (cargo build --release)
```

## Binary vs Library Crates

Cargo supports two types of crates:

Binary Crate

```bash
cargo new my_app --bin    # or just cargo new my_app
```

Creates:

```
├── Cargo.toml
└── src/
    └── main.rs
```

Library Crate

```bash
cargo new my_lib --lib
```

Creates:

```
├── Cargo.toml
└── src/
    └── lib.rs
```

## Naming Conventions

Cargo follows specific naming conventions:

* **Binaries, examples, benchmarks, and integration tests**: Use `kebab-case` (e.g., `my-awesome-tool`)
* **Modules within targets**: Use `snake_case` (e.g., `my_module`)

## Working with Multiple Files

As your project grows, you can organize code into modules:

```
src/
├── main.rs
├── lib.rs
└── modules/
    ├── mod.rs
    ├── user.rs
    └── database.rs
```

## Project Organization Tips

1. **Keep the top-level directory clean** - Use it only for README files, license information, and configuration files
2. **Follow Cargo conventions** - This makes your project familiar to other Rust developers
3. **Separate concerns** - Use different directories for different types of code (tests, examples, benchmarks)
4. **Use workspaces for large projects** - When your project becomes complex, consider using Cargo workspaces to manage multiple related crates

This standardized structure helps you organize your projects effectively and makes it easy for other Rust developers to understand and contribute to your code. The conventions ensure that tools like `cargo build`, `cargo test`, and `cargo run` work seamlessly with your project.
