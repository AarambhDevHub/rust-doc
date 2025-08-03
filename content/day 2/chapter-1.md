+++
title = "Variables and Mutability in Rust"
description = "Explore how Rust handles variable declarations, immutability, the use of 'mut', shadowing, and constants, along with best practices for writing safe and efficient code."
date = 2025-08-01
draft = false
weight = 1
tags = ["rust", "variables", "immutability", "mutability", "shadowing"]
+++

# Variables and Mutability in Rust

One of Rust's most distinctive features is its approach to **variables and mutability**. Understanding these concepts is crucial for writing safe, efficient Rust code and is fundamental to how Rust prevents common programming errors.

## Variables are Immutable by Default

In Rust, **variables are immutable by default**.

## Basic Variable Declaration

Variables are declared using the `let` keyword:

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    
    // This would cause a compilation error:
    // x = 6; // cannot assign twice to immutable variable `x`
}
```

If you try to reassign an immutable variable, Rust will give you a clear compilation error:

```rust
error[E0384]: cannot assign twice to immutable variable `x`
--> main.rs:7:5
  |
3 | let x = 1;
  |     -
  |     |
  |     first assignment to `x`
  |     help: consider making this binding mutable: `mut x`
...
7 | x = 2;
  | ^^^^^ cannot assign twice to immutable variable

```

## Making Variables Mutable with `mut`

When you need to change a variable's value, you must explicitly opt into mutability using the **`mut`****&#x20;keyword:&#x20;**

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    
    x = 6; // This is now allowed
    println!("The value of x is: {}", x);
}
```

Output:

```
The value of x is: 5
The value of x is: 6
```

The `mut` keyword serves two important purposes::

1. **Enables mutability** - Allows you to change the variable's value
2. **Communicates intent** - Signals to other developers that this variable's value will change

## Why Immutability by Default?

Rust's immutability-by-default approach provides several benefits:

**Memory Safety**

When variables are immutable, there's no risk of accidentally modifying data that other parts of your code depend on remaining unchanged.

**Concurrency Safety**

Immutable data can be safely shared between threads without synchronization, preventing data races.

**Code Predictability**

When you see a variable declaration without `mut`, you know its value won't change, making code easier to reason about.

**Bug Prevention**

Many bugs stem from unexpected changes to data. Immutability prevents these issues at compile time.

Type Annotations

While Rust can often infer variable types, you can explicitly specify them using type annotations:

```rust
fn main() {
    let x: i32 = 42;        // Explicitly typed as 32-bit integer
    let message: &str = "Hello, Rust!";  // String reference
    let mut counter: u32 = 0;            // Mutable unsigned integer
}
```

## Variable Shadowing

Rust allows **shadowing**, where you can declare a new variable with the same name, effectively hiding the previous one:

```rust
fn main() {
    let x = 5;          // x is an integer
    let x = x + 1;      // x is now 6 (still integer)
    let x = "hello";    // x is now a string
    
    println!("The value of x is: {}", x); // Prints: hello
}
```

## Shadowing vs Mutability

Shadowing is different from mutability:

* **Shadowing** creates a new variable with the same name
* **Mutability** changes the value of an existing variable
* Shadowing can change the **type** of the variable
* Mutability keeps the same type

```rust
fn main() {
    // Shadowing - creates new variables
    let spaces = "   ";        // string
    let spaces = spaces.len(); // number
    
    // This would work with shadowing but NOT with mut:
    // let mut spaces = "   ";
    // spaces = spaces.len(); // ERROR: can't change type
}
```

## Constants vs Variables

Constants are declared with the `const` keyword and are different from immutable variables:

```rust
const MAX_POINTS: u32 = 100_000;

fn main() {
    println!("Maximum points: {}", MAX_POINTS);
}
```

## Key Differences:

| Feature         | Variables (let)                | Constants (const)              |
| --------------- | ------------------------------ | ------------------------------ |
| Mutability      | Can be made mutable with `mut` | Always immutable               |
| Type annotation | Optional (inferred)            | Required                       |
| Scope           | Block-scoped                   | Can be global                  |
| Shadowing       | Allowed                        | Not Allowed                    |
| Runtime values  | Allowed                        | Must be compile-time constants |

## Best Practices

## **Start with Immutability**

Always declare variables as immutable first. Only add `mut` when you actually need to change the value.

## **Use Descriptive Names**

When using `mut`, choose variable names that clearly indicate the data will change:

```rust
let mut counter = 0;        // Good
let mut temp_result = Vec::new();  // Good
```

## **Minimize Mutable Scope**

Keep mutable variables in the smallest possible scope to reduce the risk of unintended changes.

## **Consider Shadowing for Transformations**

When you need to transform data while keeping the same conceptual variable name, shadowing can be cleaner than mutability:

```rust
// Using shadowing for data transformation
let input = "42";
let input: i32 = input.parse().unwrap();

// vs using mutability (less clear)
let mut input = "42";
let parsed_input: i32 = input.parse().unwrap();
// input is still a string, which might be confusing
```

## Common Patterns

## Conditional Mutability

```rust
fn main() {
    let mut x = 5;
    
    if some_condition() {
        x = 10;  // Only modify when needed
    }
    
    println!("Final value: {}", x);
}
```

## Building Collections

```rust
fn main() {
    let mut numbers = Vec::new();
    
    for i in 0..5 {
        numbers.push(i);
    }
    
    // numbers is now immutable for the rest of the scope
    println!("Numbers: {:?}", numbers);
}

```

Understanding variables and mutability is fundamental to writing effective Rust code. The language's immutability-by-default approach might feel restrictive at first, especially coming from languages like Python or JavaScript, but it provides powerful guarantees about code safety and helps prevent entire classes of bugs before your program even runs.
