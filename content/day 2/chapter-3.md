
+++
title = "Constants vs Variables"
description = "Understand the fundamental differences between Rust's constants and immutable variables, including compile-time evaluation, memory behavior, shadowing, type annotations, and usage patterns."
date = 2025-08-01
draft = false
weight = 3
+++

# Constants vs Variables in Rust: A Detailed Comparison

Building on your understanding of data types and variables, let's explore the **fundamental differences between constants and immutable variables** in Rust. While both appear unchangeable on the surface, they serve different purposes and have distinct characteristics that affect how they're used in your programs.

## Key Differences Overview

| Feature                     | Variables (`let`)                                    | Constants (`const`)            |
| --------------------------- | ---------------------------------------------------- | ------------------------------ |
| **Keyword**                 | `let`                                                | `const`                        |
| **Mutability**              | Immutable by default, can be made mutable with `mut` | Always immutable               |
| **Type annotation**         | Optional (inferred)                                  | Required                       |
| **Compile-time vs Runtime** | Value computed at runtime                            | Value computed at compile time |
| **Shadowing**               | Allowed                                              | Not allowed                    |
| **Scope**                   | Block-scoped                                         | Can be global or any scope     |
| **Memory behavior**         | Has a memory location                                | **Inlined at compile time**    |



## Compile-Time vs Runtime Evaluation

The **most fundamental difference** between constants and variables is when their values are determined:

## Variables (Runtime)

```rust
fn main() {
    let user_input = get_user_input();  // Value determined at runtime
    let current_time = std::time::SystemTime::now();  // Runtime value
    let calculation = 5 * 10 + user_input;  // Can use runtime values
}

```

## Constants (Compile-Time)

```rust
const MAX_POINTS: u32 = 100_000;  // Must be known at compile time
const CALCULATION: i32 = 5 * 10 + 2;  // Constant expression only
const PI: f64 = 3.14159265359;  // Literal value

fn main() {
    // This would be an error - cannot use runtime values:
    // const INVALID: i32 = get_user_input();  // Error!
}

```

**Constants must be evaluable at compile time**[1](https://www.reddit.com/r/rust/comments/pj2ier/the_main_difference_between_unmutable_variables/). This means you cannot use function calls, user input, or any value that requires runtime computation to initialize a constant.

## Memory Behavior and Inlining

## Constants are Inlined

**Constants are inlined everywhere they're used**[1](https://www.reddit.com/r/rust/comments/pj2ier/the_main_difference_between_unmutable_variables/). When you use a constant, the compiler replaces it with its actual value:

```rust
const VALUE: i32 = 42;

fn main() {
    let x = VALUE;  // Compiler replaces this with: let x = 42;
    let y = VALUE;  // Compiler replaces this with: let y = 42;
}
```

This means constants **don't have a memory address** - they're copied to every location where they're used[2](https://stackoverflow.com/questions/52751597/what-is-the-difference-between-a-constant-and-a-static-variable-and-which-should).

## Variables Have Memory Locations

Immutable variables, even when they never change, still **occupy memory** and have a specific location:

```rust
fn main() {
    let value = 42;
    let reference = &value;  // Can take a reference to the variable
    println!("Address: {:p}", reference);
}

```

## Type Annotations

## Constants Require Explicit Types

```rust
const MAX_SIZE: usize = 1000;  // Type annotation required
const RATE: f64 = 0.08;        // Must specify type

// This would cause an error:
// const INVALID = 100;  // Error: missing type annotation

```

## Variables Support Type Inference

```rust
fn main() {
    let max_size = 1000;      // Type inferred as i32
    let rate = 0.08;          // Type inferred as f64
    let name: &str = "Rust";  // Optional explicit type
}

```

## Shadowing

## Variables Support Shadowing

Variables can be **redeclared with the same name**, effectively creating new variables:

```rust
fn main() {
    let value = 5;          // Original variable
    let value = value + 1;  // New variable shadows the first
    let value = "hello";    // Can even change type
    
    println!("{}", value);  // Prints: hello
}

```

## Constants Cannot Be Shadowed

**Constants cannot be redeclared** in the same scope[5](https://iq.opengenus.org/constant-vs-variable-in-rust/):

```rust
const VALUE: i32 = 5;

fn main() {
    // This would cause an error:
    // const VALUE: i32 = 10;  // Error: duplicate definition
}

```

## Scope and Declaration

## Variable Scope

Variables are **block-scoped** and must be declared within functions or other code blocks:

```rust
fn main() {
    let local_var = 42;  // Function-scoped
    
    {
        let block_var = 100;  // Block-scoped
    }
    // block_var is not accessible here
}

```

## Constant Scope

Constants can be declared **at any scope level**, including globally[3](https://doc.rust-lang.org/rust-by-example/custom_types/constants.html)[7](https://www.tutorialspoint.com/rust/rust_constant.htm):

```rust
// Global constants
const GLOBAL_MAX: i32 = 1000;
const APP_NAME: &str = "MyApp";

fn main() {
    const LOCAL_CONST: f32 = 3.14;  // Local constant
    
    println!("Global: {}", GLOBAL_MAX);
    println!("Local: {}", LOCAL_CONST);
}

```

## Mutability Differences

## Variables Can Be Made Mutable

```rust
fn main() {
    let mut counter = 0;      // Mutable variable
    counter += 1;             // Can change value
    
    let immutable = 42;       // Immutable variable
    // immutable = 43;        // Error: cannot assign twice
}

```

## Constants Are Always Immutable

**Constants cannot be made mutable** under any circumstances[5](https://iq.opengenus.org/constant-vs-variable-in-rust/):

```rust
// This is invalid syntax:
// const mut INVALID: i32 = 100;  // Error: unexpected `mut`

const ALWAYS_IMMUTABLE: i32 = 100;
// No way to change this value

```

## Performance Implications

## Constants

* **No runtime overhead** - values are inlined at compile time
* **No memory allocation** - exist only during compilation
* **Optimal for frequently used values**

## Variables

* **Runtime memory allocation** on the stack
* **Memory access overhead** when reading values
* **Better for values that need memory addresses**

## When to Use Each

## Use Constants When:

```rust
// Mathematical constants
const PI: f64 = 3.14159265359;
const E: f64 = 2.71828;

// Configuration values known at compile time
const MAX_USERS: usize = 1000;
const DEFAULT_TIMEOUT: u64 = 30;

// String literals used throughout the application
const APP_VERSION: &str = "1.0.0";
const ERROR_MESSAGE: &str = "Invalid input provided";

```

## Use Immutable Variables When:

```rust
fn main() {
    // Runtime-computed values
    let user_input = read_user_input();
    let file_contents = std::fs::read_to_string("config.txt").unwrap();
    
    // Complex calculations
    let result = expensive_computation();
    
    // Values that need memory addresses
    let data = vec![1, 2, 3, 4, 5];
    process_data(&data);  // Need to pass a reference
}

```

## Naming Conventions

## Constants use SCREAMING\_SNAKE\_CASE

```rust
const MAX_BUFFER_SIZE: usize = 8192;
const DEFAULT_CONNECTION_TIMEOUT: u64 = 5000;
const PI_APPROXIMATION: f64 = 3.14159;

```

## Variables use snake\_case

```rust
fn main() {
    let max_retries = 3;
    let user_name = "Alice";
    let is_authenticated = false;
}

```

## Common Patterns

## Using Constants for Array Sizes

```rust
const BUFFER_SIZE: usize = 1024;
const MAX_CONNECTIONS: usize = 100;

fn main() {
    let buffer: [u8; BUFFER_SIZE] = [0; BUFFER_SIZE];
    let connections: [Option<Connection>; MAX_CONNECTIONS] = 
        [None; MAX_CONNECTIONS];
}

```

## Constants in Match Expressions

```rust
const HTTP_OK: u16 = 200;
const HTTP_NOT_FOUND: u16 = 404;
const HTTP_SERVER_ERROR: u16 = 500;

fn handle_response(status_code: u16) {
    match status_code {
        HTTP_OK => println!("Success!"),
        HTTP_NOT_FOUND => println!("Not found"),
        HTTP_SERVER_ERROR => println!("Server error"),
        _ => println!("Unknown status"),
    }
}

```

Understanding these differences helps you make the right choice between constants and variables. **Use constants for compile-time known values that won't change and need to be inlined for performance**. **Use immutable variables for runtime values that don't need to change after initialization**. This distinction is fundamental to writing efficient and idiomatic Rust code.
