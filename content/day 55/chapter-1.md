---
title: "Macro Workshop - Custom Derive Implementation"
description: "Learn how to implement custom derive macros to reduce boilerplate and enhance Rust code ergonomics."
date: 2025-10-09
weight: 1
---

# Macro Workshop: Implementing a Custom Derive Macro in Rust

Understanding how to build a custom `derive` macro in Rust is like learning to **create your own "instant recipe" generator for a professional kitchen**. Imagine you have dozens of similar dishes that all need to be prepared in a fundamentally similar way (e.g., they all need to be logged, have their ingredients counted, or be assigned a unique ID). Instead of manually writing out this boilerplate code for every single dish, you create a `#[derive(MyCustomAction)]` macro. When a chef uses this on a recipe, your system automatically generates all the necessary boilerplate, saving time, reducing errors, and ensuring consistency.

## The "Instant Recipe" Generator Analogy 🍽️

### Imagine You're a Head Chef Needing to Standardize a Process

- **A `struct` or `enum`**: A recipe card for a new dish.
- **A `trait`**: A standard kitchen process, like "Counting Ingredients" or "Assigning a Menu ID."
- **Implementing a `trait` manually**: A chef carefully writing out the steps for the "Counting Ingredients" process on the new recipe card.
- **A `derive` macro**: An "instant recipe" machine. The chef just stamps the recipe with `#[derive(CountIngredients)]`, and the machine automatically prints the correct, standardized implementation of the `CountIngredients` trait onto the card.

***

### Why Create a Custom `derive` Macro?

You've likely used built-in derive macros like `#[derive(Debug, Clone, PartialEq)]`. These macros automatically generate implementations for the `Debug`, `Clone`, and `PartialEq` traits. Custom derive macros allow you to do the same for your *own* traits.[^4][^5]

**Use a custom `derive` macro when:**

- You have a trait that will be implemented for many different structs in a similar, repetitive way.
- You want to reduce boilerplate code and enforce a consistent implementation pattern.
- You want to add functionality to structs based on their structure (e.g., their fields and types).


## How to Build a Custom `derive` Macro: A Step-by-Step Guide

Building a procedural macro like a custom derive requires a specific project setup. It must live in its own dedicated crate.

### Step 1: Project Setup

Let's create a workspace with two crates:

1. `auto_id_derive`: The crate that will contain our procedural macro.
2. `my_app`: The main application crate that will use the macro.

**1. Create the workspace and crates:**

```bash
mkdir macro_workshop
cd macro_workshop
cargo new my_app
cargo new auto_id_derive --lib
```

**2. Configure the workspace `Cargo.toml`:**
**`macro_workshop/Cargo.toml`**

```toml
[workspace]
members = [
    "my_app",
    "auto_id_derive",
]
```

**3. Configure the macro crate (`auto_id_derive/Cargo.toml`):**
This is the most critical part. We must declare this crate as a `proc-macro` crate and add the `syn` and `quote` dependencies, which are essential tools for parsing and generating Rust code.[^6]

**`macro_workshop/auto_id_derive/Cargo.toml`**

```toml
[package]
name = "auto_id_derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true # This tells Rust this crate is for procedural macros

[dependencies]
syn = "1.0" # A library for parsing Rust code into a syntax tree
quote = "1.0" # A library for turning syntax trees back into Rust code
```

**4. Configure the main app crate (`my_app/Cargo.toml`):**
The main application needs to depend on both the macro crate and a new crate we'll create for our trait. It's a best practice to define the trait in a separate crate from the macro so that other crates can depend on the trait without also depending on the macro's implementation details.

Let's create one more crate for our trait:

```bash
cargo new auto_id --lib
```

Add it to the workspace `Cargo.toml`. Now, configure `my_app/Cargo.toml`:

```toml
[dependencies]
auto_id = { path = "../auto_id" }
auto_id_derive = { path = "../auto_id_derive" }
```


### Step 2: Define the Trait

Let's create a simple `HasId` trait that our macro will implement.

**`macro_workshop/auto_id/src/lib.rs`**

```rust
pub trait HasId {
    /// Returns a unique identifier for the struct.
    fn id(&self) -> &str;
}
```


### Step 3: Write the Custom Derive Macro

Now for the core of the workshop. We will write the code that generates the `impl HasId for ...` block.

**`macro_workshop/auto_id_derive/src/lib.rs`**

```rust
// This brings the necessary tools into scope
extern crate proc_macro;
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

/// This is our custom derive macro function.
/// The Rust compiler calls this function when it sees `#[derive(HasId)]`.
#[proc_macro_derive(HasId)]
pub fn has_id_derive(input: TokenStream) -> TokenStream {
    // 1. Parse the input `TokenStream` into a structured representation.
    // `syn::DeriveInput` represents a struct or enum definition.
    // `parse_macro_input!` is a helper macro from `syn` that handles errors for us.
    let input = parse_macro_input!(input as DeriveInput);

    // 2. Extract the name of the struct the macro is attached to.
    let name = &input.ident;

    // 3. Generate the implementation code using the `quote!` macro.
    // `quote!` is like a templating engine for Rust code.
    // We can use `#variable` to inject variables from our Rust code into the generated code.
    let expanded = quote! {
        // This is the code that will be generated and placed into the user's crate.
        impl auto_id::HasId for #name {
            fn id(&self) -> &str {
                // For this simple example, we'll use the struct's name as its ID.
                stringify!(#name) // stringify! turns an identifier into a string literal.
            }
        }
    };

    // 4. Convert the generated `quote::Tokens` back into a `proc_macro::TokenStream`.
    TokenStream::from(expanded)
}
```


### Step 4: Use the Custom Derive Macro

Finally, let's use our new `#[derive(HasId)]` in our main application.

**`macro_workshop/my_app/src/main.rs`**

```rust
// Import the trait itself
use auto_id::HasId;
// Import the derive macro
use auto_id_derive::HasId;

// Use the derive macro on a struct.
#[derive(HasId)]
struct User {
    username: String,
    email: String,
}

// Use it on another struct.
#[derive(HasId)]
struct Product {
    name: String,
    price: f64,
}

fn print_id(item: &impl HasId) {
    println!("The ID of this item is: '{}'", item.id());
}

fn main() {
    let user = User {
        username: "Alice".to_string(),
        email: "alice@example.com".to_string(),
    };

    let product = Product {
        name: "Laptop".to_string(),
        price: 1200.0,
    };

    print_id(&user);
    print_id(&product);
}
```


### Step 5: Run the Code

Navigate to the `my_app` directory and run the project:

```bash
cd my_app
cargo run
```

**Expected Output:**

```
The ID of this item is: 'User'
The ID of this item is: 'Product'
```

**How It Worked**: At compile time, when the compiler processed `my_app`, it saw `#[derive(HasId)]` on the `User` struct. It then invoked our `has_id_derive` function from the `auto_id_derive` crate, passing it the code for the `User` struct. Our macro then generated the following code block and handed it back to the compiler:

```rust
impl auto_id::HasId for User {
    fn id(&self) -> &str {
        "User"
    }
}
```

The compiler then included this implementation as if we had written it ourselves. The same process happened for the `Product` struct.

## Expanding Your Knowledge: Handling Fields and Attributes

This was a simple example. More advanced derive macros can inspect the fields of a struct or even take helper attributes. For example, you could modify the macro to look for a field named `id` on the struct and use its value, or you could allow the user to specify the ID with a helper attribute like `#[id = "custom_id"]`. These tasks involve more detailed parsing with the `syn` crate but follow the same fundamental pattern of **parse -> transform -> generate**.

By learning to build custom derive macros, you unlock one of Rust's most powerful features for creating ergonomic, boilerplate-free libraries and applications.

1. https://cetra3.github.io/blog/creating-your-own-derive-macro/
2. https://stackoverflow.com/questions/53135923/how-to-write-a-custom-derive-macro
3. https://www.reddit.com/r/rust/comments/145uhss/a_walkthough_on_how_to_write_derive_procedural/
4. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/procedural-macros.html
5. http://www.technorely.com/insights/writing-and-optimizing-custom-derives-with-rusts-proc-macro-for-code-generation
6. https://github.com/imbolc/rust-derive-macro-guide
7. https://doc.rust-lang.org/book/ch19-06-macros.html
8. https://rareskills.io/post/rust-attribute-derive-macro
9. https://users.rust-lang.org/t/implementing-a-custom-derive-macro-to-work-in-conjunction-with-serde-serialize/104794
10. https://www.freecodecamp.org/news/procedural-macros-in-rust/
