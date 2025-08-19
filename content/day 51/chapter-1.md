---
title: "Declarative Macros - macro_rules! syntax"
description: "Learn the basics of declarative macros in Rust using macro_rules! syntax."
date: 2025-10-06
weight: 1
---

# Declarative Macros (`macro_rules!`) in Rust: A Beginner's Guide

Understanding declarative macros in Rust is like learning to **create custom, reusable blueprints for your professional kitchen**. Instead of manually writing out the same boilerplate code every time you need a new recipe card or a specialized calculation, you create a smart template (`macro_rules!`) that can generate the code for you based on a simple pattern. This is a form of **metaprogramming**—writing code that writes other code—and it's one of Rust's most powerful features.[^1][^2]

## The Professional Kitchen Blueprint Analogy 🍽️

### Imagine You're the Head Chef Designing Reusable Recipe Blueprints

- **The Problem**: You find yourself constantly writing out similar function definitions or data structures with only minor variations. For example, creating a function to add two numbers, another to add three, another for four, and so on.
- **The Solution**: You create a single, smart blueprint (`macro_rules!`) called `sum!`. This blueprint can accept any number of ingredients (expressions) and will automatically generate the correct addition code for you.

***

### The Naive Approach (Manual Repetition)

```rust
// ❌ Writing repetitive code manually
fn add_two(a: i32, b: i32) -> i32 { a + b }
fn add_three(a: i32, b: i32, c: i32) -> i32 { a + b + c }
// This is tedious, error-prone, and doesn't scale.
```


### The Power of Declarative Macros (A Smart Blueprint)

```rust
// ✅ Using a macro to generate the code for us
macro_rules! sum {
    // A rule to match one or more expressions separated by commas
    ( $( $x:expr ),+ ) => {
        // The code that will be generated
        {
            0 $(+ $x)*
        }
    };
}

fn main() {
    // The macro expands to `0 + 1 + 2` at compile time
    let total_two = sum!(1, 2);
    // The macro expands to `0 + 10 + 20 + 30` at compile time
    let total_three = sum!(10, 20, 30);

    println!("Sum of two: {}", total_two);
    println!("Sum of three: {}", total_three);
}
```

**Why Macros Are More Powerful Than Functions:**

- **Variadic Arguments**: They can take a variable number of arguments, like our `sum!` macro or the built-in `println!` and `vec!` macros. Functions in Rust require a fixed number of arguments.
- **Code Generation**: They operate on the code itself *before* it's fully compiled, allowing them to generate structures, functions, and expressions that a normal function cannot.
- **Domain-Specific Languages (DSLs)**: They can be used to create mini-languages within Rust, making certain tasks more expressive (e.g., custom `html!` or `query!` macros).


## The Anatomy of `macro_rules!`

Declarative macros work like a `match` expression for your code's syntax. You define one or more "rules" (or "arms"), and the compiler tries to match the macro invocation against these rules.[^5]

The basic structure is: `( matcher ) => { transcriber }`.[^1]

- **The Matcher**: This is the pattern that the macro input must conform to. It describes the syntax you expect.
- **The Transcriber**: This is the template for the code that will be generated if the matcher succeeds. It's the "output" of the macro.


### A Simple "Hello" Macro

Let's start with the simplest possible macro.

```rust
// 1. Define the macro
macro_rules! say_hello {
    // This is the matcher: it matches an empty invocation like `say_hello!()`
    () => {
        // This is the transcriber: the code to be generated
        println!("Hello, Rust Macro!");
    };
}

fn main() {
    // 2. Call the macro
    // At compile time, this line is replaced with `println!("Hello, Rust Macro!");`
    say_hello!();
}
```


### Capturing Arguments: Metavariables and Fragments

To make macros useful, we need to capture parts of the input. We do this using **metavariables**, which are denoted with a `$` (e.g., `$x`), and **fragment specifiers**, which define what kind of code fragment the metavariable should match.[^5]

- **`$x:expr`**: Matches any Rust **expression**.
- **`$i:ident`**: Matches any **identifier** (like a variable or function name).
- **`$t:ty`**: Matches any **type**.
- **`$stmt:stmt`**: Matches a **statement**.
- **`$p:path`**: Matches a **path** (e.g., `std::collections::HashMap`).
- **`$l:literal`**: Matches a **literal** (e.g., `"hello"`, `42`).
- **`$tt:tt`**: Matches a single **token tree**. This is a catch-all for more complex patterns.

**Example: A Macro that Doubles a Value**

```rust
macro_rules! double {
    // Matcher: Captures any expression and names it `$val`
    ( $val:expr ) => {
        // Transcriber: Use the captured expression in the generated code
        $val * 2
    };
}

fn main() {
    // `double!(5)` expands to `5 * 2`
    // `double!(10 + 5)` expands to `(10 + 5) * 2`
    println!("5 doubled is {}", double!(5));
    println!("(10 + 5) doubled is {}", double!(10 + 5));
}
```


### Handling Repetition: `$(...)*` and `$(...)+`

This is where the true power of declarative macros shines. You can match repeating patterns.[^5]

- `$(...)*`: Match the pattern inside **zero or more** times.
- `$(...)+`: Match the pattern inside **one or more** times.
- You can specify a separator, like `$(...),*`.

**Example: A Custom `vec!` Macro**

Let's re-implement a simplified version of the `vec!` macro to see how repetition works.

```rust
macro_rules! my_vec {
    // Matcher: Match zero or more expressions, separated by commas.
    // The captured expressions are collected under the metavariable `$x`.
    ( $( $x:expr ),* ) => {
        {
            // Transcriber: Create a new, mutable vector
            let mut temp_vec = Vec::new();
            // And for each captured expression `$x`...
            $(
                // ...push it into the vector.
                temp_vec.push($x);
            )*
            // Finally, the block returns the vector.
            temp_vec
        }
    };
}

fn main() {
    let v1 = my_vec![1, 2, 3];
    let v2 = my_vec!["hello", "world"];
    let v3 = my_vec![];

    assert_eq!(v1, vec![1, 2, 3]);
    assert_eq!(v2, vec!["hello", "world"]);
    assert_eq!(v3, Vec::<i32>::new());
}
```


### Multiple Rules: Overloading Your Macro

Just like a `match` expression, `macro_rules!` can have multiple arms. The compiler will try them in order until it finds one that matches. This allows you to create "overloaded" macros that behave differently based on the input syntax.

The standard `vec!` macro uses this to support both `vec!` and `vec![0; 5]` (create a vec with five `0`s).[^2][^3][^1]

**Example: An Overloaded `create_user` Macro**

```rust
struct User {
    username: String,
    id: u32,
}

macro_rules! create_user {
    // Rule 1: Match a username only. Assign a default ID.
    ( $username:expr ) => {
        User {
            username: $username.to_string(),
            id: 0, // Default ID
        }
    };
    // Rule 2: Match a username AND an ID.
    ( $username:expr, $id:expr ) => {
        User {
            username: $username.to_string(),
            id: $id,
        }
    };
}

fn main() {
    let user1 = create_user!("Alice"); // Matches Rule 1
    let user2 = create_user!("Bob", 101); // Matches Rule 2

    assert_eq!(user1.id, 0);
    assert_eq!(user2.id, 101);
}
```


## Best Practices and Common Pitfalls

1. **Prefer Functions When Possible**: Macros are powerful, but they are also more complex and harder to debug than functions. If you can solve your problem with a regular function or a generic function, do that first. Use macros only when you need their unique capabilities (like variadic arguments).
2. **Use Hygienic Macros**: Rust macros are mostly "hygienic." This means that variables created inside a macro won't accidentally clash with variables of the same name outside the macro. This is a major safety feature.
3. **Wrap Transcribers in Braces (`{...}`)**: It's a very good practice to wrap your transcriber in `{}`. This creates a new scope and ensures that the macro expands to a single expression, which prevents surprising behavior related to operator precedence.
4. **Use `@` for Internal Rules**: For complex recursive macros, it's a common pattern to create "internal" rules that are not meant to be called directly by the user. These rules often start with an `@` token to make them distinct.
5. **Exporting Macros**: If you want users of your library to be able to use your macro, you must annotate it with `#[macro_export]`.

By mastering declarative macros, you gain the ability to reduce boilerplate, create powerful and flexible APIs, and write code that is more expressive and concise, all while leveraging the safety and performance of the Rust compiler.
<span style="display:none">[^4][^6][^7][^8]</span>

1. https://dev.to/rogertorres/first-steps-with-rust-declarative-macros-1f8m
2. https://doc.rust-lang.org/book/ch19-06-macros.html
3. https://suchprogramming.com/rusty-macros/
4. https://www.youtube.com/watch?v=KsJHlqULpO4
5. https://earthly.dev/blog/rust-macros/
6. https://www.freecodecamp.org/news/procedural-macros-in-rust/
7. https://www.reddit.com/r/rust/comments/18afxup/declarative_rust_macros_explanation/
8. https://betterprogramming.pub/rust-generic-trait-declarative-and-procedural-macros-6ff9cd8016de
