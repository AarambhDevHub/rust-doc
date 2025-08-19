---
title: "Attribute Macros"
description: "Learn how to build and use attribute macros for adding custom annotations in Rust."
date: 2025-10-06
weight: 5
---

# Attribute Macros in Rust: A Beginner's Guide to Custom Annotations

Understanding attribute macros in Rust is like learning to **create your own custom "special instruction" stickers for your kitchen recipes**. Instead of just following the standard recipe format, you can add a sticker like `#[ExtraSpicy]` or `#[MakeVegan]` to a dish, and your assistant (the Rust compiler) will automatically modify the recipe based on that instruction. Attribute macros allow you to write code that operates on other code, enabling powerful abstractions and reducing boilerplate.

## The Professional Kitchen Recipe Analogy 🍽️

### Imagine You're a Head Chef Creating a Dynamic Recipe Book

- **A `struct` or `fn`**: A standard recipe card for a dish.
- **An Attribute Macro**: A special, magical sticker you can place on the recipe card.
- **The Compiler**: Your highly-trained kitchen assistant.

***

### The Problem without Attribute Macros

You have a standard recipe for "Spaghetti Bolognese." Now, you want a vegan version. You would have to copy the entire recipe and manually change the ingredients: beef becomes lentils, cheese becomes a nut-based substitute, etc. This is repetitive and error-prone.

### The Power of Attribute Macros

Instead of rewriting the recipe, you just take the standard "Spaghetti Bolognese" card and add a `#[MakeVegan]` sticker to it. Your assistant (the compiler) sees the sticker and, at "compile time" (before cooking begins), generates a *new*, modified recipe card with all the vegan substitutions already made.

This is the essence of attribute macros: they are annotations that take an item (like a function or a struct) as input, transform it, and output a new, modified version of that item.[^4]

## Attribute Macros vs. Other Macro Types

Rust has two main categories of macros, and it's important to understand the difference :[^2]

1. **Declarative Macros (`macro_rules!`)**:
    * These are "macros by example." They match patterns and generate code based on those patterns.
    * **Analogy**: A simple find-and-replace template for your recipes.
    * **Examples**: `vec![]`, `println![]`.
2. **Procedural Macros**:
    * These are more powerful and complex. They are actual Rust functions that operate on the code's structure (its Abstract Syntax Tree or AST).[^6]
    * **Analogy**: Your intelligent kitchen assistant who can read, understand, and rewrite entire recipes.
    * **Attribute macros are a type of procedural macro.** The other types are "derive" macros (like `#[derive(Debug)]`) and "function-like" macros (which look like function calls, e.g., `sqlx::query!(...)`).

This guide focuses exclusively on **attribute macros**.

## How to Build an Attribute Macro: A Step-by-Step Guide

Building a procedural macro requires a special setup. It must live in its own separate crate with a specific configuration.

### Step 1: Project Setup

Let's create a project that will have two crates: one for the macro itself (`chef_macros`) and one for using the macro (`my_kitchen`).

1. **Create a new Cargo workspace:**

```bash
mkdir recipe_project
cd recipe_project
cargo new my_kitchen
cargo new chef_macros --lib
```

2. **Configure the workspace in a root `Cargo.toml`:**
**`recipe_project/Cargo.toml`**

```toml
[workspace]
members = [
    "my_kitchen",
    "chef_macros",
]
```

3. **Configure the macro crate (`chef_macros/Cargo.toml`):**
This is the most important step. We need to tell Rust that this crate provides a procedural macro, and we need to add the `syn` and `quote` crates, which are essential for parsing and generating Rust code.

**`recipe_project/chef_macros/Cargo.toml`**

```toml
[package]
name = "chef_macros"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true  # This line is crucial!

[dependencies]
syn = { version = "1.0", features = ["full"] } # For parsing Rust code
quote = "1.0" # For generating Rust code
```

4. **Add a dependency in the user crate (`my_kitchen/Cargo.toml`):**
Our main application needs to depend on the macro crate.

**`recipe_project/my_kitchen/Cargo.toml`**

```toml
[dependencies]
chef_macros = { path = "../chef_macros" }
```


### Step 2: Writing the Attribute Macro

Let's create a simple `#[log_entry_exit]` macro. When we apply this to a function, it will automatically add `println!` statements at the beginning and end of the function.

**`recipe_project/chef_macros/src/lib.rs`**

```rust
// We need to bring these into scope
extern crate proc_macro;
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

/// This is our attribute macro. It will be used like `#[log_entry_exit]`.
#[proc_macro_attribute]
pub fn log_entry_exit(
    _attr: TokenStream, // We're not using attributes for now, so we ignore this.
    item: TokenStream,  // This is the function the attribute is attached to.
) -> TokenStream {
    // 1. Parse the input `TokenStream` into a structured Rust item (a function).
    // `syn` provides `ItemFn` to represent a function's structure.
    let mut func = parse_macro_input!(item as ItemFn);

    // 2. Get the name of the function for our log message.
    let func_name = &func.sig.ident;
    let func_name_str = func_name.to_string();

    // 3. Get the original block of code from the function.
    let original_block = &func.block;

    // 4. Generate the new code using the `quote!` macro.
    // The `quote!` macro lets us write Rust code and inject variables.
    let new_block: syn::Block = syn::parse2(quote! {
        {
            // This is our new function body
            println!("Entering function: {}", #func_name_str);

            // Here we include the original code of the function.
            let result = #original_block;

            println!("Exiting function: {}", #func_name_str);

            // Return the original result
            result
        }
    }).expect("Failed to parse new block");

    // 5. Replace the old function block with our new, expanded block.
    func.block = Box::new(new_block);

    // 6. Convert the modified function back into a `TokenStream` and return it.
    let output = quote! { #func };
    output.into()
}
```


### Step 3: Using the Attribute Macro

Now, let's use our new macro in the `my_kitchen` crate.

**`recipe_project/my_kitchen/src/main.rs`**

```rust
// Import the macro from our other crate
use chef_macros::log_entry_exit;

/// This is our custom attribute in action!
#[log_entry_exit]
fn prepare_dish(dish_name: &str) -> String {
    println!("  -> Preparing {}...", dish_name);
    format!("{} is ready!", dish_name)
}

fn main() {
    let dish = prepare_dish("Spaghetti");
    println!("Final dish: {}", dish);
}
```


### Step 4: Running the Code

Navigate to the `my_kitchen` directory and run the project:

```bash
cd my_kitchen
cargo run
```

**Expected Output:**

```
Entering function: prepare_dish
  -> Preparing Spaghetti...
Exiting function: prepare_dish
Final dish: Spaghetti is ready!
```

**How It Worked**: At compile time, the Rust compiler saw `#[log_entry_exit]` above `prepare_dish`. It invoked our macro, passing the `prepare_dish` function's code to it as a `TokenStream`. Our macro then modified that token stream to inject the `println!` statements and returned the new version of the function, which was then compiled as if we had written it that way manually.

## Taking it Further: Adding Arguments to Attributes

Attribute macros can also take arguments, making them even more powerful. Let's modify our macro to take a custom log level as an argument, like `#[log_entry_exit(level = "DEBUG")]`.

To do this, we need to parse the `_attr` token stream. The `syn` crate is essential for this. This is a more advanced topic, but here is a conceptual overview:

1. **Define a Struct for Arguments**: You would create a struct that represents the expected arguments (e.g., `struct LogArgs { level: syn::LitStr }`).
2. **Parse the `_attr` Stream**: In your macro function, you would parse the `_attr: TokenStream` into your `LogArgs` struct.
3. **Use the Arguments**: You can then use the parsed arguments (like `args.level`) inside your `quote!` block to customize the generated code.

## When to Use Attribute Macros

- **Adding Functionality**: When you want to wrap a function or struct with additional behavior, like logging, benchmarking, transaction management, or permission checks.
- **Code Transformation**: When you need to transform the code in some way, such as making a synchronous function asynchronous (as some libraries do) or adding serialization/deserialization logic.
- **Creating Frameworks**: Frameworks like `actix-web` and `rocket` use attribute macros extensively to define routes (e.g., `#[get("/")]`). This provides a clean, declarative API for users.
- **Reducing Boilerplate**: Similar to declarative macros, they can be used to generate repetitive `impl` blocks or other boilerplate code based on the structure of an item.

Attribute macros are one of Rust's most powerful (and complex) features. They enable library authors to create highly ergonomic and expressive APIs, effectively extending the language itself to solve specific problems. While they have a steeper learning curve than declarative macros, they unlock a new level of abstraction and code generation capabilities.
<span style="display:none">[^1][^3][^5][^7][^8]</span>

1. https://www.freecodecamp.org/news/procedural-macros-in-rust/
2. https://doc.rust-lang.org/book/ch19-06-macros.html
3. https://earthly.dev/blog/rust-macros/
4. https://rareskills.io/post/rust-attribute-derive-macro
5. https://www.youtube.com/watch?v=GFijwucFJqw
6. https://doc.rust-lang.org/reference/procedural-macros.html
7. https://blog.logrocket.com/macros-in-rust-a-tutorial-with-examples/
8. https://www.youtube.com/watch?v=Zmoy65pcHlk
