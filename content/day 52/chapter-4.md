---
title: "Custom Derive Macros"
description: "Create custom derive macros to implement traits automatically."
date: 2025-10-06
weight: 4
---

# Custom Derive Macros in Rust: A Beginner's Guide

Understanding custom derive macros in Rust is like learning to **create your own custom kitchen appliance that automatically prepares ingredients for a specific type of recipe**. You've seen standard appliances like `#[derive(Debug)]` or `#[derive(Clone)]` that the language provides. A custom derive macro lets you build your *own* appliance. For example, you could create a `#[derive(MakeSandwich)]` that automatically teaches any struct how to be assembled into a sandwich. This powerful feature allows you to eliminate boilerplate code and enforce conventions by automatically implementing traits for your data structures.

## The Professional Kitchen Appliance Analogy 🍽️

### Imagine You're a Chef Who Wants to Automate Recipe Preparation

- **A `struct` or `enum`**: A set of raw ingredients (e.g., `struct Sandwich { bread: String, filling: String, has_mustard: bool }`).
- **A `trait`**: A recipe or a capability (e.g., `trait Assemblable { fn assemble(&self); }`).
- **A standard `derive`**: A common kitchen appliance provided for you (like a blender for `Debug`).
- **A custom `derive` macro**: A specialized appliance you build yourself. For example, a `#[derive(Assemblable)]` machine that knows how to take any `struct` with `bread` and `filling` fields and automatically generate the steps to assemble it.

**The Problem without Custom Derive:**
For every type of sandwich (`BLT`, `Club`, `Veggie`), you would have to manually write out the *same* assembly logic in an `impl` block. This is repetitive, error-prone, and hard to maintain.

**The Power of Custom Derive:**
You write the "assembly logic" once inside the macro. Then, you can simply add `#[derive(Assemblable)]` to any sandwich `struct`, and the compiler will automatically write the correct `impl Assemblable for ...` block for you.

## What is a Procedural Macro?

Custom derive macros are a type of **procedural macro** (or "proc macro"). Unlike declarative macros (`macro_rules!`), which work with pattern matching, procedural macros are like special compiler plugins written in Rust. They take a stream of code tokens as input, manipulate them, and produce a new stream of code tokens as output.[^4]

There are three types of procedural macros:

1. **Custom `derive`**: The most common type. Adds code based on a `struct` or `enum` (e.g., `#[derive(MyTrait)]`).
2. **Attribute-like**: A custom attribute that can be attached to any item (e.g., `#[my_attribute]` on a function).
3. **Function-like**: A macro that looks like a function call (e.g., `my_macro!(...)`).

This guide focuses on **custom derive macros**.

## How to Build a Custom Derive Macro: Step-by-Step

Let's build a simple custom derive macro from scratch. We'll create a trait `HasName` and then a `#[derive(HasName)]` macro that automatically implements this trait for any struct with a `name` field.

### Step 1: The Project Structure (Two Crates)

A procedural macro must live in its **own separate crate** with a specific `proc-macro = true` setting in its `Cargo.toml`. This is a requirement from the Rust compiler.[^1][^5]

Our project will have two crates:

1. `my_app`: The main application crate where we define our trait and use the derive macro.
2. `my_app_derive`: The `proc-macro` crate where we implement the custom derive logic.

Let's set up the directory structure:

```
my_project/
├── my_app/
│   ├── src/
│   │   └── main.rs
│   └── Cargo.toml
├── my_app_derive/
│   ├── src/
│   │   └── lib.rs
│   └── Cargo.toml
└── Cargo.toml  (Workspace)
```

**1. Workspace `Cargo.toml` (`my_project/Cargo.toml`):**

```toml
[workspace]
members = ["my_app", "my_app_derive"]
```

**2. Application Crate `Cargo.toml` (`my_app/Cargo.toml`):**

```toml
[package]
name = "my_app"
version = "0.1.0"
edition = "2021"

[dependencies]
# We depend on our derive macro crate
my_app_derive = { path = "../my_app_derive" }
```

**3. Proc Macro Crate `Cargo.toml` (`my_app_derive/Cargo.toml`):**

```toml
[package]
name = "my_app_derive"
version = "0.1.0"
edition = "2021"

# This tells Rust that this crate is a procedural macro
[lib]
proc-macro = true

[dependencies]
# These two crates are the powerhouse of procedural macros
syn = { version = "1.0", features = ["full"] } # For parsing Rust code
quote = "1.0" # For generating Rust code
```


### Step 2: Defining the Trait (in `my_app`)

In `my_app/src/main.rs`, let's define the trait we want to implement automatically.

**`my_app/src/main.rs`:**

```rust
// We import the derive macro from our other crate
use my_app_derive::HasName;

// 1. The trait we want to automatically implement
pub trait HasName {
    fn name(&self) -> &str;
}

// 2. Here's a struct we want to derive the trait for.
//    The `#[derive(HasName)]` attribute is what we are building.
#[derive(HasName)]
pub struct User {
    username: String,
    name: String, // Our macro will look for this field
}

#[derive(HasName)]
pub struct Project {
    name: String,
    owner: User,
}

fn main() {
    let user = User {
        username: "jdoe".to_string(),
        name: "John Doe".to_string(),
    };

    let project = Project {
        name: "Rust Macro Guide".to_string(),
        owner: user,
    };

    // Because of our custom derive, these methods will exist!
    println!("User's name: {}", project.owner.name());
    println!("Project's name: {}", project.name());
}
```

Right now, this code will fail to compile because `#[derive(HasName)]` doesn't exist yet. Let's build it.

### Step 3: Implementing the Derive Macro (in `my_app_derive`)

This is where the magic happens. We'll edit `my_app_derive/src/lib.rs`.

**`my_app_derive/src/lib.rs`:**

```rust
// This crate provides the `proc_macro` functionality from the compiler
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn::{self, parse_macro_input, DeriveInput};

// This is the function that the compiler will call when it sees `#[derive(HasName)]`.
// The `#[proc_macro_derive(HasName)]` attribute registers our macro.
#[proc_macro_derive(HasName)]
pub fn has_name_derive(input: TokenStream) -> TokenStream {
    // 1. Parse the input tokens into a syntax tree.
    // `DeriveInput` is a struct that represents a `struct` or `enum` definition.
    let ast = parse_macro_input!(input as DeriveInput);

    // 2. Get the name of the struct we're deriving for (e.g., "User").
    let name = &ast.ident;

    // 3. Generate the implementation code using the `quote!` macro.
    // `quote!` is like a templating engine for Rust code.
    let gen = quote! {
        // This is the code that our macro will output.
        // `#name` will be replaced with the identifier of the struct.
        impl HasName for #name {
            fn name(&self) -> &str {
                // This assumes the struct has a field named `name`.
                // A more robust macro would check for this field first.
                &self.name
            }
        }
    };

    // 4. Convert the generated code back into a TokenStream and return it.
    gen.into()
}
```


### How It Works: A Deeper Look

1. **`#[proc_macro_derive(HasName)]`**: This attribute tells the Rust compiler that this function, `has_name_derive`, is the implementation for the `HasName` derive macro.
2. **`input: TokenStream`**: The function receives the raw source code of the item it's attached to (e.g., the entire `struct User { ... }` block) as a `TokenStream`. This is just a sequence of code tokens, which is hard to work with directly.
3. **`syn::parse_macro_input!`**: This is where the `syn` crate comes in. It's a powerful parser that can take a `TokenStream` and turn it into a structured representation of Rust code, known as an **Abstract Syntax Tree (AST)**. The `DeriveInput` struct from `syn` represents a struct, enum, or union definition, giving us easy access to its name, fields, generics, etc.
4. **`quote::quote!`**: This is the counterpart to `syn`. The `quote!` macro lets us write Rust code almost like a template. It's a "quasi-quoting" macro, meaning we can write code and then "un-quote" variables from our environment into the generated code using `#variable_name`. Here, we use `#name` to inject the identifier of the struct we're working on (e.g., `User` or `Project`).
5. **`gen.into()`**: Finally, we convert the `quote!` output (which is a `proc_macro2::TokenStream`) back into the `proc_macro::TokenStream` that the compiler expects and return it. The compiler then takes this generated code and inserts it into our program.

Now, if you go back to the `my_project` root and run `cargo run -p my_app`, it will compile and run successfully! The `#[derive(HasName)]` macro expanded into:

```rust
impl HasName for User {
    fn name(&self) -> &str {
        &self.name
    }
}
impl HasName for Project {
    fn name(&self) -> &str {
        &self.name
    }
}
```


### Step 4: Making the Macro More Robust (Handling Fields)

Our current macro blindly assumes a field named `name` exists. A more robust macro would inspect the fields of the struct.

```rust
// (Inside my_app_derive/src/lib.rs)
// ... same setup ...
#[proc_macro_derive(HasName)]
pub fn has_name_derive_robust(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;

    // Get the fields from the struct
    let fields = if let syn::Data::Struct(syn::DataStruct { fields: syn::Fields::Named(fields), .. }) = ast.data {
        fields
    } else {
        panic!("HasName can only be derived for structs with named fields");
    };

    // Find a field named "name"
    let mut name_field_ident = None;
    for field in fields.named.iter() {
        if let Some(ident) = &field.ident {
            if ident == "name" {
                name_field_ident = Some(ident);
                break;
            }
        }
    }

    let name_field = match name_field_ident {
        Some(ident) => ident,
        None => panic!("Struct must have a field named 'name' to derive HasName"),
    };

    let gen = quote! {
        impl HasName for #name {
            fn name(&self) -> &str {
                // Use the identified field
                &self.#name_field
            }
        }
    };

    gen.into()
}
```

This version is much safer. It inspects the struct's fields, ensures it's a struct with named fields, finds the `name` field, and uses it. If it can't find the field, it will produce a helpful compile-time error.

By mastering custom derive macros, you can significantly reduce boilerplate, enforce API consistency, and create powerful, reusable abstractions that feel like a natural part of the Rust language.
<span style="display:none">[^10][^2][^3][^6][^7][^8][^9]</span>

1. https://cetra3.github.io/blog/creating-your-own-derive-macro/
2. https://stackoverflow.com/questions/53135923/how-to-write-a-custom-derive-macro
3. https://www.reddit.com/r/rust/comments/145uhss/a_walkthough_on_how_to_write_derive_procedural/
4. https://doc.rust-lang.org/reference/procedural-macros.html
5. https://github.com/imbolc/rust-derive-macro-guide
6. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/procedural-macros.html
7. https://www.freecodecamp.org/news/procedural-macros-in-rust/
8. https://doc.rust-lang.org/book/ch19-06-macros.html
9. https://rareskills.io/post/rust-attribute-derive-macro
10. https://users.rust-lang.org/t/i-wrote-my-first-custom-derive-macro/57887
