---
title: "TokenStream Processing"
description: "Learn how procedural macros use TokenStream for input and output processing."
date: 2025-10-06
weight: 2
---

# Procedural Macros and `TokenStream`: A Beginner's Guide

Understanding how procedural macros process `TokenStream` in Rust is like learning to be a **master chef who works with a deconstructed recipe**. Instead of a finished dish, you are given a tray of all the raw ingredients, precisely measured and labeled (a `TokenStream`). Your job is to analyze these ingredients, possibly add new ones or transform existing ones, and then arrange them into a new, complete recipe (another `TokenStream`) for another chef to cook. Procedural macros operate on the very structure of Rust code itself, giving you the power to create powerful, intelligent abstractions.

## The Deconstructed Recipe Analogy 🍽️

### Imagine You're a Chef Who Writes Recipes for Other Chefs

- **Rust Code**: A finished, human-readable recipe.
- **The Rust Compiler**: The head chef who oversees the entire kitchen.
- **A Procedural Macro**: You, the "recipe architect."
- **A `TokenStream`**: A deconstructed recipe. The compiler takes a piece of code (like a `struct` or `fn`) and breaks it down into its fundamental components—words, numbers, symbols—and hands them to you on a tray. This tray is a `TokenStream`.

**Your Workflow as the Recipe Architect:**

1. **Receive the `TokenStream`**: The compiler hands you a tray of ingredients for the `struct User { ... }`. This includes the words `struct`, `User`, the `{` and `}` symbols, etc.
2. **Parse the `TokenStream`**: You analyze the ingredients. You recognize the pattern of a `struct` definition. You identify its name (`User`) and its fields.
3. **Generate a new `TokenStream`**: Based on your analysis, you create a new recipe. For example, you might add the entire recipe for a "default constructor" (`impl User { fn new() -> Self { ... } }`) to the tray.
4. **Return the new `TokenStream`**: You hand the expanded tray of ingredients back to the compiler, which then cooks (compiles) the final, combined recipe.

## What is a `TokenStream`?

A `TokenStream` is the fundamental input and output of all procedural macros. It's essentially a sequence of `TokenTree`s. A `TokenTree` can be one of four things :[^4]

- **`Ident`**: An identifier, like a variable name, function name, or keyword (`user`, `fn`, `struct`).
- **`Punct`**: A punctuation character, like `+`, `->`, or `,`.
- **`Literal`**: A literal value, like `42`, `"hello"`, or `true`.
- **`Group`**: A delimited sequence of tokens. The delimiters can be `()` (parentheses), `[]` (brackets), or `{}` (braces). The content inside the group is another `TokenStream`.

When you write `#[derive(Debug)]` on a struct, the compiler takes the entire `struct` definition, converts it into a `TokenStream`, and passes it to your `Debug` derive macro.[^2]

## The Three Types of Procedural Macros

All three types of procedural macros work by processing `TokenStream`s, but they are triggered differently :[^5]

1. **Function-like Macros**: `my_macro!(...)` - Takes the tokens inside the `()` as its input `TokenStream`.
2. **Derive Macros**: `#[derive(MyTrait)]` - Takes the `struct` or `enum` it's attached to as its input `TokenStream`.
3. **Attribute Macros**: `#[my_attribute]` - Takes the item it's attached to as its `item` `TokenStream` and any arguments to the attribute as its `attr` `TokenStream`.

## The Essential Crates for Procedural Macros

Writing procedural macros from scratch by manually iterating over `TokenTree`s is extremely tedious. The Rust community has developed three essential crates that make this process much easier:

1. **`proc-macro2`**: Provides an improved version of the standard `proc_macro::TokenStream`. It can be used in your macro's logic and then easily converted back to the standard library's type when you return. This allows for better testing and error handling.
2. **`syn`**: The **parser** crate. Its job is to parse a `TokenStream` into a structured, easily navigable Rust Abstract Syntax Tree (AST). For example, it can turn a `TokenStream` representing a function into an `ItemFn` struct, where you can easily access the function's name, arguments, and body.[^5]
3. **`quote`**: The **generator** crate. It does the reverse of `syn`. It provides a `quote!` macro that lets you write what looks like regular Rust code, and it will generate the corresponding `TokenStream` for you.[^1]

**The Standard Workflow**:
**`proc_macro::TokenStream` -> `syn` (parse) -> AST -> `quote` (generate) -> `proc_macro2::TokenStream` -> `proc_macro::TokenStream`**

## Example Project: A Simple `Builder` Derive Macro

Let's build a procedural macro that automatically creates a builder pattern for any struct.

**Goal**:
Given this code:

```rust
#[derive(Builder)]
pub struct Command {
    executable: String,
    args: Vec<String>,
    env: Vec<String>,
    current_dir: String,
}
```

Our macro should automatically generate this code:

```rust
pub struct CommandBuilder {
    executable: Option<String>,
    args: Option<Vec<String>>,
    // ... and so on for all fields
}

impl CommandBuilder {
    pub fn executable(&mut self, executable: String) -> &mut Self {
        self.executable = Some(executable);
        self
    }
    // ... builder methods for all fields ...

    pub fn build(&mut self) -> Result<Command, Box<dyn std::error::Error>> {
        // ... build logic ...
    }
}
```


### Step 1: Setting up the Crate

Procedural macros must be in their own special type of crate.

**`Cargo.toml`**:

```toml
[package]
name = "my-builder-macro"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true # This is essential!

[dependencies]
syn = { version = "2.0", features = ["full"] }
quote = "1.0"
proc-macro2 = "1.0"
```


### Step 2: Writing the Macro Code

**`src/lib.rs`**:

```rust
// The proc-macro crate is special and provided by the compiler.
extern crate proc_macro;

use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Ident};
use quote::quote;
use proc_macro2::Span;

#[proc_macro_derive(Builder)]
pub fn derive_builder(input: TokenStream) -> TokenStream {
    // 1. PARSE: Use `syn` to parse the input TokenStream into an AST.
    // `DeriveInput` is a struct from `syn` that represents a struct or enum definition.
    let input = parse_macro_input!(input as DeriveInput);

    // Extract the name of the struct we're deriving for (e.g., "Command").
    let struct_name = &input.ident;

    // Create the name for our new builder struct (e.g., "CommandBuilder").
    let builder_name = Ident::new(&format!("{}Builder", struct_name), Span::call_site());

    // Extract the fields from the struct definition.
    let fields = if let syn::Data::Struct(s) = &input.data {
        &s.fields
    } else {
        panic!("Builder derive macro can only be used on structs.");
    };

    // 2. GENERATE: Use `quote` to generate the new code as TokenStreams.

    // --- Generate the fields for the builder struct ---
    // e.g., `executable: Option<String>,`
    let builder_fields = fields.iter().map(|f| {
        let field_name = &f.ident;
        let field_type = &f.ty;
        quote! {
            #field_name: Option<#field_type>,
        }
    });

    // --- Generate the builder methods ---
    // e.g., `pub fn executable(&mut self, executable: String) -> &mut Self { ... }`
    let builder_methods = fields.iter().map(|f| {
        let field_name = &f.ident;
        let field_type = &f.ty;
        quote! {
            pub fn #field_name(&mut self, #field_name: #field_type) -> &mut Self {
                self.#field_name = Some(#field_name);
                self
            }
        }
    });

    // --- Generate the `build()` method logic ---
    // This part is more complex. We need to check that every field has been set.
    let build_field_checks = fields.iter().map(|f| {
        let field_name = &f.ident;
        quote! {
            #field_name: self.#field_name.take()
                .ok_or_else(|| format!("Field '{}' is missing", stringify!(#field_name)))?,
        }
    });

    // --- Assemble all the generated pieces into the final TokenStream ---
    let expanded = quote! {
        // This is the main template for the code we want to generate.
        // `#...` is how `quote` interpolates variables.
        // `#(#...)*` is how `quote` iterates over a collection.

        pub struct #builder_name {
            #(#builder_fields)*
        }

        impl #builder_name {
            #(#builder_methods)*

            pub fn build(&mut self) -> Result<#struct_name, Box<dyn std::error::Error>> {
                Ok(#struct_name {
                    #(#build_field_checks)*
                })
            }
        }

        // Add a convenience method to the original struct
        impl #struct_name {
            pub fn builder() -> #builder_name {
                #builder_name {
                    #(#builder_fields_init)*
                }
            }
        }
    };

    // This is a bit of a hack for the example. Let's create the init fields.
    let builder_fields_init = fields.iter().map(|f| {
        let field_name = &f.ident;
        quote! { #field_name: None, }
    });

    // Re-quote with the init fields. A real macro would be structured better.
    let final_expanded = quote! {
        pub struct #builder_name {
            #(#builder_fields)*
        }

        impl #builder_name {
            #(#builder_methods)*

            pub fn build(&mut self) -> Result<#struct_name, Box<dyn std::error::Error>> {
                Ok(#struct_name {
                    #(#build_field_checks)*
                })
            }
        }

        impl #struct_name {
            pub fn builder() -> #builder_name {
                #builder_name {
                    #(#builder_fields_init)*
                }
            }
        }
    };

    // 3. RETURN: Convert the generated `quote` TokenStream back into the
    // `proc_macro::TokenStream` that the compiler expects.
    TokenStream::from(final_expanded)
}
```


### Step 3: Using the Macro in another Crate

**`Cargo.toml` of the user crate:**

```toml
[dependencies]
my-builder-macro = { path = "../my-builder-macro" }
```

**`src/main.rs` of the user crate:**

```rust
use my_builder_macro::Builder;

#[derive(Builder)]
pub struct Command {
    executable: String,
    args: Vec<String>,
}

fn main() {
    let mut builder = Command::builder();
    let command = builder
        .executable("cargo".to_string())
        .args(vec!["build".to_string(), "--release".to_string()])
        .build()
        .unwrap();

    assert_eq!(command.executable, "cargo");
}
```

When you compile this code, the compiler sees `#[derive(Builder)]`, invokes our procedural macro with the `TokenStream` for the `Command` struct, and our macro generates all the `CommandBuilder` code, which is then compiled as if we had written it by hand. This powerful `TokenStream`-based workflow allows you to write code that writes code, creating safe, reusable, and intelligent abstractions.
<span style="display:none">[^10][^3][^6][^7][^8][^9]</span>

1. https://petanode.com/posts/rust-proc-macro/
2. https://doc.rust-lang.org/reference/procedural-macros.html
3. https://www.freecodecamp.org/news/procedural-macros-in-rust/
4. https://blog.jetbrains.com/rust/2022/03/18/procedural-macros-under-the-hood-part-i/
5. http://developerlife.com/2022/03/30/rust-proc-macro/
6. https://towardsdatascience.com/nine-rules-for-creating-procedural-macros-in-rust-595aa476a7ff/
7. https://docs.swmansion.com/scarb/docs/reference/procedural-macro.html
8. https://earthly.dev/blog/rust-macros/
9. https://stackoverflow.com/questions/75571171/how-to-parse-arbitrary-tokens-in-rusts-procedural-macros
10. https://dev.to/dandyvica/rust-procedural-macros-step-by-step-tutorial-36n8
