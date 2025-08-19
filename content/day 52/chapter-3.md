---
title: "syn and quote Crates"
description: "Understand how the syn crate parses Rust code and the quote crate generates Rust code."
date: 2025-10-06
weight: 3
---

# `syn` and `quote`: The Power Duo of Rust Procedural Macros

Understanding the `syn` and `quote` crates is like learning the two fundamental skills of a master chef who writes cookbooks: **deconstruction** and **reconstruction**. First, the chef must be able to taste a complex dish and break it down into its core ingredients and steps (parsing with `syn`). Second, they must be able to take those abstract steps and write them down clearly in a recipe so someone else can recreate the dish (code generation with `quote`). In the world of Rust's procedural macros, `syn` is the tool for deconstructing Rust code, and `quote` is the tool for reconstructing it.

## The Cookbook Author Analogy 🍽️

### Imagine You're Writing a Cookbook for Other Chefs

- **Procedural Macro**: A "master recipe" in your cookbook that can automatically generate other, more specific recipes based on simple instructions.
- **A `TokenStream`**: A raw, messy stream of cooking instructions, like "add `struct` then `MyStruct` then `{` then `field`..." It's just a sequence of words and symbols.
- **`syn` (The Parser)**: Your ability to read a messy set of instructions (`TokenStream`) and understand its structure, organizing it into a clear, hierarchical list: "This is a `struct` named `MyStruct`, it has one `field` named `x` of `type` `i32`..." This structured representation is called an **Abstract Syntax Tree (AST)**.[^2]
- **`quote` (The Code Generator)**: Your ability to take that structured list (the AST) and write out a new, complete recipe (`TokenStream`) in perfect, compilable Rust syntax.

Without these tools, you'd be stuck trying to manually manipulate the raw, messy stream of words, which is incredibly difficult and error-prone.

## What are Procedural Macros?

Before diving into `syn` and `quote`, it's important to know *where* they are used. **Procedural macros** are an advanced type of macro in Rust that operate on the code itself. Unlike declarative macros (`macro_rules!`), they are actual Rust functions that take a stream of code tokens as input and produce a new stream of tokens as output.[^6]

There are three types of procedural macros:

1. **Function-like macros**: `my_macro!(...)`
2. **Derive macros**: `#[derive(MyTrait)]`
3. **Attribute macros**: `#[my_attribute]`

`syn` and `quote` are the essential tools for writing all three types.

## `syn`: Parsing Rust Code into a Syntax Tree

The `syn` crate is a powerful parsing library that can take a `TokenStream` and convert it into a structured Rust data type, an Abstract Syntax Tree (AST). This turns a flat stream of tokens into a hierarchical tree that represents the code's meaning.[^2]

### Key `syn` Data Structures

`syn` provides data structures for every part of the Rust language. Some of the most common ones you'll encounter are:[^3]

- `syn::File`: Represents an entire Rust source file.
- `syn::Item`: Represents an "item" like a `struct`, `enum`, `fn`, or `mod`.
- `syn::DeriveInput`: A special struct that represents the input to a `#[derive]` macro (which can only be a `struct`, `enum`, or `union`).
- `syn::Ident`: An identifier, like a variable or function name.
- `syn::Type`: A type annotation, like `i32` or `String`.
- `syn::Expr`: An expression, like `2 + 2`.


### Example: Parsing a `struct` with `syn`

Let's see what `syn` does. Imagine our macro is given this code:

```rust
struct Point {
    x: i32,
    y: i32,
}
```

The macro receives this as a `TokenStream`. We use `syn::parse` to turn it into a structured `DeriveInput` object.

```rust
// This code goes inside a special "proc-macro" crate.
// Add to Cargo.toml:
// [lib]
// proc-macro = true
//
// [dependencies]
// syn = { version = "2.0", features = ["full"] }
// quote = "1.0"

use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput};

// This is a dummy derive macro that just prints the parsed structure.
#[proc_macro_derive(DebugPrint)]
pub fn debug_print_derive(input: TokenStream) -> TokenStream {
    // Use `parse_macro_input!` to parse the TokenStream into a DeriveInput struct.
    // If parsing fails, it will automatically return a helpful compiler error.
    let ast = parse_macro_input!(input as DeriveInput);

    // `ast` is now a structured representation of our `struct Point { ... }`
    println!("Parsed struct identifier: {}", ast.ident);

    // We can inspect the fields of the struct
    if let syn::Data::Struct(data_struct) = ast.data {
        for field in data_struct.fields.iter() {
            let field_name = field.ident.as_ref().unwrap();
            let field_type = &field.ty;
            // To print the type, we need `quote`!
            println!("  - Field '{}' has type: {}", field_name, quote::quote! { #field_type });
        }
    }

    // A real macro would return a new TokenStream here.
    TokenStream::new()
}
```

When we run `cargo build` on a crate that uses `#[derive(DebugPrint)]` on `struct Point`, the compiler will print:

```
Parsed struct identifier: Point
  - Field 'x' has type: i32
  - Field 'y' has type: i32
```

As you can see, `syn` has transformed the raw code into a data structure that we can easily inspect and manipulate.

## `quote`: Generating Rust Code from a Syntax Tree

The `quote` crate does the reverse of `syn`: it takes structured data and generates a `TokenStream` of Rust code. It provides a `quote!` macro that feels like string interpolation but for Rust code tokens.

### The `quote!` Macro

- **Syntax**: It works like `println!`, but it produces code.
- **Interpolation**: You can "splice" variables and code fragments into the output using `#variable_name`.
- **Repetition**: You can loop over collections to generate repetitive code using `#(...)*`.


### Example: Generating a `new` function with `quote`

Let's build a procedural macro that automatically creates a `new` constructor for any struct it's applied to.

```rust
// Continuing in our proc-macro crate...
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput, Data, Fields};
use quote::quote;

#[proc_macro_derive(New)]
pub fn new_derive(input: TokenStream) -> TokenStream {
    // 1. Parse the input code into an AST using `syn`.
    let ast = parse_macro_input!(input as DeriveInput);

    // Get the name of the struct (e.g., `Point`).
    let struct_name = &ast.ident;

    // Extract the fields from the struct.
    let fields = match &ast.data {
        Data::Struct(data_struct) => &data_struct.fields,
        _ => panic!("The New macro can only be used on structs."),
    };

    // We only want named fields for our constructor.
    let named_fields = match fields {
        Fields::Named(fields_named) => &fields_named.named,
        _ => panic!("The New macro only works on structs with named fields."),
    };

    // 2. Generate the code for the `new` function using `quote`.

    // Create lists of the field names and their types for the function signature.
    // e.g., `x` and `y`
    let field_names = named_fields.iter().map(|f| &f.ident);
    // e.g., `i32` and `i32`
    let field_types = named_fields.iter().map(|f| &f.ty);

    // The `quote!` macro generates the new code.
    let generated_code = quote! {
        // `impl Point { ... }`
        impl #struct_name {
            // `pub fn new(x: i32, y: i32) -> Self { ... }`
            // The `#( ... ),*` syntax repeats the pattern for each element in the iterators.
            pub fn new( #( #field_names: #field_types ),* ) -> Self {
                // We need to clone the iterators because `quote!` consumes them.
                let field_names_clone = named_fields.iter().map(|f| &f.ident);
                // `Self { x, y }`
                Self {
                    #( #field_names_clone ),*
                }
            }
        }
    };

    // 3. Convert the generated code back into a TokenStream and return it.
    generated_code.into()
}
```

**How to Use It:**

```rust
use my_proc_macros::New; // Assuming our proc-macro crate is named `my_proc_macros`

#[derive(New)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    // Our macro generated this `new` function for us!
    let p = Point::new(10, 20);
    assert_eq!(p.x, 10);
    assert_eq!(p.y, 20);
}
```


## Summary: The `syn` and `quote` Workflow

Writing a procedural macro follows a clear, three-step process:

1. **Parse (`syn`)**: Take the input `proc_macro::TokenStream` and parse it into a structured `syn` data type (like `DeriveInput`). This is the **deconstruction** step.

```rust
let ast = parse_macro_input!(input as DeriveInput);
```

2. **Analyze and Transform**: Inspect the AST to gather the information you need (struct name, field names, types, attributes, etc.). This is where you apply your logic.
3. **Generate (`quote`)**: Use the `quote!` macro to build a new `proc_macro2::TokenStream` containing the Rust code you want to add or generate. This is the **reconstruction** step.

```rust
let generated_code = quote! {
    // ... new code using #variables ...
};
```

4. **Return**: Convert the generated `TokenStream` back to the required `proc_macro::TokenStream` type.

```rust
generated_code.into()
```


By mastering `syn` for parsing and `quote` for code generation, you unlock the full power of procedural macros, enabling you to extend Rust's syntax, eliminate boilerplate, and build incredibly powerful and expressive libraries.
<span style="display:none">[^1][^10][^4][^5][^7][^8][^9]</span>

1. https://www.reddit.com/r/rust/comments/1gl7trt/rust_macros_with_syn_the_guide_you_didnt_know_you/
2. https://docs.rs/syn
3. https://github.com/dtolnay/syn
4. https://stackoverflow.com/questions/57408862/how-to-parse-rust-code-without-using-procedural-macros
5. https://github.com/sharnoff/derive-syn-parse
6. http://developerlife.com/2022/03/30/rust-proc-macro/
7. https://docs.rs/syn/latest/syn/parse/index.html
8. https://www.freecodecamp.org/news/procedural-macros-in-rust/
9. https://users.rust-lang.org/t/rust-syn-and-quote/110802
10. https://users.rust-lang.org/t/how-to-construct-syntax-trees-robustly/96921
