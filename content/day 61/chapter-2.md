---
title: "Environment Variables in CLI Apps"
description: "Understand how to read and manage environment variables in Rust CLI applications."
date: 2025-11-03
weight: 2
---

# Environment Variables in Rust CLI Applications: A Beginner's Guide

Environment variables are a cornerstone for configuring CLI apps, letting you inject secrets, API keys, and configuration *without* hardcoding them or passing them explicitly as command-line arguments. In Rust, managing these variables is simple and powerful—with the standard library and popular third-party tools such as `dotenv`.

## Why Use Environment Variables in CLI Apps?

- **Portability**: Makes your app easy to configure without recompiling.
- **Security**: Lets you handle secrets (API keys, credentials) more securely than placing them in code.
- **Separation of Concerns**: Keeps environment-specific data separate from code logic.

***

## Reading Environment Variables: The Standard Way

Rust's `std::env` module is your main entry point for all environment-related operations.

### Reading an Environment Variable

```rust
use std::env;

fn main() {
    // Try to read an environment variable called KEY
    let result = env::var("KEY");
    match result {
        Ok(val) => println!("KEY: {val}"),
        Err(e) => println!("Couldn't read KEY: {e}"),
    }
}
```

*If `KEY` is set in the environment, you'll get its value; if not, you'll get an error (like `NotPresent`).*[^1][^2]

**Running with an environment variable:**

- Linux/macOS: `KEY=my_secret cargo run`
- Windows (PowerShell): `$env:KEY="my_secret"; cargo run`

***

### Providing a Default Value

You might want your app to use a default if the variable isn't set:

```rust
let key = env::var("KEY").unwrap_or_else(|_| "default_value".to_string());
println!("KEY is: {}", key);
```

For numbers or types, chain further parsing and defaults:

```rust
let port: u16 = env::var("PORT")
    .unwrap_or_else(|_| "8080".to_string())
    .parse::<u16>()
    .unwrap_or(8080); // Fallback to 8080 if parse fails
```

This pattern ensures your app is robust whether or not the env var is set or parses correctly.[^3]

***

### Listing All Environment Variables

To see all currently set variables:

```rust
for (key, value) in std::env::vars() {
    println!("{:?}: {:?}", key, value);
}
```

This is useful for debugging app configuration.[^4]

***

## Setting and Removing Environment Variables Programmatically

You can also set or remove a variable inside your Rust code (usually only for test/dev purposes):

```rust
// Set
std::env::set_var("NEW_KEY", "value");
// Remove
std::env::remove_var("NEW_KEY");
```


***

## Loading Environment Variables from a `.env` File

For local development, it's common to use a `.env` file to store secrets or configs. The [dotenv](https://crates.io/crates/dotenv) crate enables this pattern.

### Set Up

In `Cargo.toml`:

```toml
[dependencies]
dotenv = "0.15"
```


### Usage

1. Create a `.env` file in your project root:

```
API_KEY=my_apikey_123
```

2. Load it and read variables:

```rust
use dotenv::dotenv;
use std::env;

fn main() {
    dotenv().ok(); // Loads .env into process env variables

    let key = env::var("API_KEY").expect("API_KEY must be set");
    println!("API_KEY is {}", key);
}
```


Now, running `cargo run` will print the API key from your `.env` file.[^5]

***

## Best Practices

- **Check for missing variables**: Always handle `Err` when using `env::var`, so missing configuration doesn't panic your app unexpectedly.
- **Prefer command-line args for user input** and environment variables for secrets or defaults/config.
- **Document required variables** in your README.
- **Avoid using environment variables for sensitive data in production** unless secure OS-level protections are in place; consider vault systems for very sensitive keys.

***

## Recap: Common Patterns

| **Pattern** | **Code Example** |
| :-- | :-- |
| String with fallback | `let x = env::var("VAR").unwrap_or("default".to_string());` |
| Parsed number | `let port: u16 = env::var("PORT").unwrap_or("8080".to_string()).parse().unwrap_or(8080);` |
| Load from .env | `dotenv().ok(); let key = env::var("API_KEY").unwrap();` |
| All variables | `for (k,v) in env::vars() { println!(\"{k}: {v}\"); }` |

Rust's `std::env` and `dotenv` ecosystem make environment variable management for CLI applications straightforward, robust, and secure.[^2][^1][^5]

1. https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html
2. https://dev.to/francescoxx/3-ways-to-use-environment-variables-in-rust-4eaf
3. https://stackoverflow.com/questions/78104768/most-idiomatic-way-to-read-a-numeric-environment-variable-in-rust
4. https://doc.rust-lang.org/src/std/env.rs.html
5. https://zsiciarz.github.io/24daysofrust/book/vol2/day5.html
6. https://www.reddit.com/r/rust/comments/1ecnriw/command_line_env_vars_and_configs/
7. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch12-05-working-with-environment-variables.html
8. https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html
9. https://doc.rust-lang.org/std/env/index.html
10. https://docs.rs/load-dotenv
