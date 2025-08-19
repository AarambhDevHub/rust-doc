---
title: "Types of Procedural Macros"
description: "Explore function-like, derive, and attribute procedural macros in Rust."
date: 2025-10-06
weight: 1
---

# Procedural Macros in Rust: A Beginner's Guide to the Three Types

Understanding procedural macros in Rust is like learning to **program the robots in your professional kitchen**. While declarative macros are like creating simple, reusable recipe templates, procedural macros are like writing the complex software that allows a robot arm to perform custom tasks—analyzing an ingredient (`TokenStream`), manipulating it, and creating something entirely new. Procedural macros operate on the code itself, allowing you to extend Rust's syntax in powerful ways.

## What Are Procedural Macros?

Procedural macros are a type of macro that acts like a function. They accept Rust code as input, operate on that code, and produce new Rust code as output. This all happens at compile time.[^2]

**Key Characteristics:**

- **Input/Output**: They take a `TokenStream` as input and return a `TokenStream` as output. A `TokenStream` is essentially a sequence of Rust code elements (like identifiers, keywords, and literals).[^1]
- **Compiler Integration**: They are like compiler plugins that you can write yourself, allowing you to extend the language's capabilities.[^5]
- **Separate Crate**: Procedural macros must be defined in their own special "procedural macro" crate. You cannot define them in the same crate where you use them.[^2]


## The Three Types of Procedural Macros

Rust has three distinct types of procedural macros, each triggered by a different syntax and used for a different purpose.[^3][^1]


| **Macro Type** | **Kitchen Analogy** | **Syntax** | **What It Does** |
| :-- | :-- | :-- | :-- |
| **Function-like** | A custom-programmed kitchen robot for a specific task | `my_macro!(...)` | Takes a block of code and transforms it into something else. |
| **Derive** | A "Make Standard" button on a kitchen appliance | `#[derive(MyTrait)]` | Automatically implements a trait for a `struct` or `enum`. |
| **Attribute** | A special instruction sticker on a recipe | `#[my_attribute]` or `#[my_attribute(...)]` | Attaches to an item (like a function or struct) and can modify or replace it. |


***

## Setting Up a Procedural Macro Crate

To create any type of procedural macro, you first need a dedicated crate.

1. **Create a new library**: `cargo new my_macros --lib`
2. **Modify `Cargo.toml`**:

```toml
[package]
name = "my_macros"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true # This line is essential!

[dependencies]
syn = { version = "2.0", features = ["full"] } # For parsing Rust code
quote = "1.0" # For generating Rust code
```

    - `proc-macro = true` tells Rust this is a procedural macro crate.[^2]
    - `syn` is a powerful library for parsing Rust code from a `TokenStream` into a structured Abstract Syntax Tree (AST).
    - `quote` is the inverse of `syn`; it helps you generate a `TokenStream` from Rust-like code.

***

## 1. Function-like Macros

These are the most similar to declarative macros in how they are called (`my_macro!(...)`), but their implementation is far more powerful. They can parse their input in any way they want.

**Analogy**: A robot designed to take a list of ingredients (`"onions", "carrots", "celery"`) and generate the full Rust code for a `struct` representing that recipe.

**Example: An SQL Macro**

Let's create a `sql!` macro that validates a very simple SQL-like syntax at compile time.

**In `my_macros/src/lib.rs`:**

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr};

/// A function-like macro to generate a (very simplified) SQL string.
/// Usage: sql!("SELECT * FROM users")
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    // Parse the input tokens as a string literal.
    // `parse_macro_input!` handles errors for us.
    let input_str = parse_macro_input!(input as LitStr);
    let sql_string = input_str.value();

    // Perform validation at compile time
    if !sql_string.to_uppercase().starts_with("SELECT") {
        // Return a compiler error if the validation fails
        return syn::Error::new(input_str.span(), "SQL must start with 'SELECT'")
            .to_compile_error()
            .into();
    }

    // If validation passes, generate the code that creates the string literal.
    // The `quote!` macro turns this back into a TokenStream.
    let expanded = quote! {
        #sql_string
    };

    expanded.into()
}
```

**How to use it in another crate:**

```rust
use my_macros::sql;

fn main() {
    let my_query = sql!("SELECT id, name FROM products");
    println!("My query is: {}", my_query);

    // This would cause a compile-time error:
    // let invalid_query = sql!("UPDATE users SET active = false");
    // --> error: SQL must start with 'SELECT'
}
```


## 2. Derive Macros

This is the most common type of procedural macro. It allows you to create new "derives" that you can add to the `#[derive(...)]` attribute on structs and enums. This is perfect for implementing traits automatically.

**Analogy**: Your oven has a `#[derive(Bake)]` button. You put a dish (`struct Cake`) in, press the button, and the oven automatically generates and executes the correct baking steps (`impl Bake for Cake`).

**Example: Deriving a Custom `Answer` Trait**

Let's create a macro that derives a trait that gives the answer to life, the universe, and everything.

**First, define the trait in a normal crate (not the macro crate):**

```rust
// In `my_app/src/main.rs`
pub trait Answer {
    fn answer() -> i32;
}
```

**Now, implement the derive macro in `my_macros/src/lib.rs`:**

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

/// A derive macro that implements the `Answer` trait.
/// Usage: #[derive(Answer)]
#[proc_macro_derive(Answer)]
pub fn answer_derive(input: TokenStream) -> TokenStream {
    // Parse the input tokens into a syntax tree.
    // This gives us structured data, like the name of the struct.
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident; // Get the identifier (name) of the struct

    // Use the `quote!` macro to generate the implementation.
    let expanded = quote! {
        // This is the code that will be generated for the user's struct
        impl Answer for #name {
            fn answer() -> i32 {
                42
            }
        }
    };

    expanded.into()
}
```

**How to use it:**

```rust
use my_macros::Answer; // The derive macro
// Remember to also bring the trait into scope
use my_app::Answer as _;

#[derive(Answer)] // Our procedural macro runs here
struct MyStruct;

fn main() {
    // We can now call the method that our macro generated!
    println!("The answer is: {}", MyStruct::answer());
}
```

This is how popular libraries like `serde` provide `#[derive(Serialize, Deserialize)]`.

## 3. Attribute Macros

Attribute macros are the most flexible type. They can be attached to almost any "item" (like a function, struct, or module) and can modify, replace, or add code.

**Analogy**: A special "Double Recipe" sticker (`#[double]`). When you put this sticker on a recipe (`fn bake_cake()`), the kitchen robot reads it and automatically doubles all the ingredient quantities in the recipe before executing it.

**Example: A Timing Attribute**

Let's create a `#[timed]` attribute that wraps a function to measure and print its execution time.

**In `my_macros/src/lib.rs`:**

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

/// An attribute macro that times the execution of a function.
/// Usage: #[timed]
#[proc_macro_attribute]
pub fn timed(_attr: TokenStream, item: TokenStream) -> TokenStream {
    // Parse the item the attribute is attached to (it must be a function).
    let func = parse_macro_input!(item as ItemFn);

    let func_name = &func.sig.ident; // The function's name
    let func_block = &func.block;    // The function's body (all its code)
    let func_sig = &func.sig;        // The function's signature (inputs, output, etc.)

    // Generate a new function that wraps the original one.
    let expanded = quote! {
        #func_sig { // Use the original function's signature
            // Start a timer before executing the original code
            let start = std::time::Instant::now();

            // Execute the original function's body
            #func_block

            // Print the elapsed time
            println!("Function '{}' executed in {:?}", stringify!(#func_name), start.elapsed());
        }
    };

    expanded.into()
}
```

**How to use it:**

```rust
use my_macros::timed;
use std::thread;
use std::time::Duration;

#[timed] // Our attribute macro modifies the function below
fn slow_operation() {
    println!("Doing some slow work...");
    thread::sleep(Duration::from_secs(1));
    println!("...done.");
}

fn main() {
    slow_operation();
    // Output will be:
    // Doing some slow work...
    // ...done.
    // Function 'slow_operation' executed in 1.00s
}
```

This is how web frameworks like `actix-web` and `rocket` provide routing attributes like `#[get("/")]`. They are attribute macros that transform a normal function into a web handler.
<span style="display:none">[^4][^6][^7][^8][^9]</span>

1. https://blog.jetbrains.com/rust/2022/03/18/procedural-macros-under-the-hood-part-i/
2. https://doc.rust-lang.org/reference/procedural-macros.html
3. https://www.freecodecamp.org/news/procedural-macros-in-rust/
4. https://petanode.com/posts/rust-proc-macro/
5. http://developerlife.com/2022/03/30/rust-proc-macro/
6. https://www.starknet.io/cairo-book/ch12-10-procedural-macros.html
7. https://doc.rust-lang.org/book/ch19-06-macros.html
8. https://www.reddit.com/r/rust/comments/145uhss/a_walkthough_on_how_to_write_derive_procedural/
9. https://docs.swmansion.com/scarb/docs/reference/procedural-macro.html
