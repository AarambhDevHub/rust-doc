---
title: "Advanced Macros - Function-like Procedural Macros"
description: "Learn how to build and use function-like procedural macros for advanced metaprogramming in Rust."
date: 2025-10-08
weight: 1
---

# Function-like Procedural Macros in Rust: A Beginner's Guide

Understanding function-like procedural macros in Rust is like learning to **create your own intelligent kitchen appliance** that can understand a custom language. While declarative macros are like simple recipe templates, and attribute macros are like "special instruction" stickers, a function-like macro is like a smart oven that you can command with `cook!("Pizza", temp: 200, time: 15)`. It takes your custom, structured input, validates it, and then generates the complex sequence of low-level instructions needed to operate the oven correctly.

## The Smart Kitchen Appliance Analogy 🍽️

### Imagine You Want to Create a Custom Command for Your Kitchen

- **A standard function**: A button on the oven labeled "Bake." It does one thing, and you can't change its parameters easily.
- **A function-like macro**: A voice-activated command system where you can say `cook!(...)`. It can parse complex instructions, handle optional parameters, and even check your recipe for errors before it starts cooking.

***

### The Problem without Function-like Macros

If you wanted to write a function to generate and validate SQL queries, you would have to pass the query as a simple string.

```rust
// The function can't validate the SQL at compile time.
// A typo in "SELCET" would only be caught at runtime.
fn execute_sql(query: &str) {
    // ... logic to execute the query ...
}

fn main() {
    execute_sql("SELCET * FROM users WHERE id = 1"); // Error is hidden here!
}
```


### The Power of Function-like Macros

A function-like macro can parse the input tokens *at compile time*, validate the syntax, and then generate the code to execute the query.

```rust
// This is a conceptual example. `sql!` is a famous example of a function-like macro.
// In a real implementation, this macro would parse and validate the SQL.
let my_query = sql!(SELECT * FROM users WHERE id = 1);
```

Here, if you wrote `SELCET`, the macro could fail to compile and give you an immediate, clear error message. This is the superpower of function-like macros: **they allow you to create custom, domain-specific "mini-languages" inside Rust that are validated at compile time**.[^1]

## Function-like Macros vs. Other Macro Types

| **Macro Type** | **Syntax** | **Analogy** | **Key Feature** |
| :-- | :-- | :-- | :-- |
| **Declarative** | `my_macro!(...)` | Recipe template | Pattern matching (`macro_rules!`) |
| **Attribute** | `#[my_attribute]` | "Special instruction" sticker | Modifies the item it's attached to (`struct`, `fn`, etc.) |
| **Function-like** | `my_macro!(...)` | Smart kitchen appliance | Takes a custom stream of tokens as input, generates new code [^4] |

Notice that function-like macros are invoked with `!` just like declarative macros. You can't tell them apart just by looking at the invocation. The key difference is in their implementation: declarative macros use pattern matching, while function-like macros use powerful Rust code to parse and generate token streams.[^3]

## How to Build a Function-like Macro: A Step-by-Step Guide

The setup for a function-like macro is identical to that of an attribute macro. It must live in its own dedicated `proc-macro` crate.

### Step 1: Project Setup

We'll follow the same project structure as before: a workspace containing a macro crate (`chef_macros`) and a user crate (`my_kitchen`).

1. **Create the workspace and crates:**

```bash
mkdir smart_kitchen
cd smart_kitchen
cargo new my_kitchen
cargo new chef_macros --lib
```

2. **Configure the root `Cargo.toml`:**

```toml
[workspace]
members = ["my_kitchen", "chef_macros"]
```

3. **Configure the macro crate (`chef_macros/Cargo.toml`):**

```toml
[package]
name = "chef_macros"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
syn = { version = "1.0", features = ["full"] }
quote = "1.0"
```

4. **Configure the user crate (`my_kitchen/Cargo.toml`):**

```toml
[dependencies]
chef_macros = { path = "../chef_macros" }
```


### Step 2: Writing the Function-like Macro

Let's create a `recipe!` macro that takes a simple, custom syntax for defining a recipe and generates a Rust struct and an implementation for it.

**Our Goal**: We want to be able to write this in our `main.rs`:

```rust
recipe! {
    name: "Spaghetti Carbonara",
    serves: 4,
    ingredients: {
        "Pasta": "500g",
        "Guanciale": "150g",
        "Eggs": "4",
        "Pecorino Romano": "100g",
        "Black Pepper": "to taste"
    }
}
```

And have the macro generate this code:

```rust
pub struct SpaghettiCarbonara;

impl SpaghettiCarbonara {
    pub fn serves() -> u32 { 4 }
    pub fn print_ingredients() {
        println!("- Pasta: 500g");
        println!("- Guanciale: 150g");
        // ... and so on
    }
}
```

**`recipe_project/chef_macros/src/lib.rs`**

```rust
extern crate proc_macro;
use proc_macro::TokenStream;
use quote::quote;
use syn::{
    parse::{Parse, ParseStream, Result},
    parse_macro_input,
    punctuated::Punctuated,
    Expr, Ident, LitInt, LitStr, Token,
};

// 1. Define structs to represent our custom syntax.
// `syn` will help us parse the input `TokenStream` into these structs.
struct Recipe {
    name: LitStr,
    serves: LitInt,
    ingredients: Vec<(LitStr, LitStr)>,
}

// 2. Implement the `Parse` trait for our custom structs.
// This tells `syn` how to parse our `recipe!{...}` syntax.
impl Parse for Recipe {
    fn parse(input: ParseStream) -> Result<Self> {
        // This is a small parser for our custom language.
        let mut name: Option<LitStr> = None;
        let mut serves: Option<LitInt> = None;
        let mut ingredients: Option<Vec<(LitStr, LitStr)>> = None;

        while !input.is_empty() {
            let key: Ident = input.parse()?;
            input.parse::<Token![:]>()?;

            if key == "name" {
                name = Some(input.parse()?);
            } else if key == "serves" {
                serves = Some(input.parse()?);
            } else if key == "ingredients" {
                let content;
                syn::braced!(content in input);
                let parsed_ingredients =
                    Punctuated::<(LitStr, Token![:], LitStr), Token![,]>::parse_terminated(&content)?;
                ingredients = Some(
                    parsed_ingredients
                        .into_iter()
                        .map(|(k, _, v)| (k, v))
                        .collect(),
                );
            }
            // Consume optional comma
            if !input.is_empty() {
                input.parse::<Token![,]>()?;
            }
        }

        Ok(Recipe {
            name: name.ok_or_else(|| input.error("missing `name` field"))?,
            serves: serves.ok_or_else(|| input.error("missing `serves` field"))?,
            ingredients: ingredients.ok_or_else(|| input.error("missing `ingredients` field"))?,
        })
    }
}

// 3. Define the main macro function.
#[proc_macro]
pub fn recipe(input: TokenStream) -> TokenStream {
    // Parse the incoming tokens into our `Recipe` struct.
    let recipe = parse_macro_input!(input as Recipe);

    // Extract the data we need for code generation.
    let struct_name_str = recipe.name.value().replace(" ", "");
    let struct_name = Ident::new(&struct_name_str, recipe.name.span());
    let serves_val = recipe.serves;

    // Generate the `println!` lines for each ingredient.
    let ingredient_lines = recipe.ingredients.iter().map(|(name, amount)| {
        quote! {
            println!("- {}: {}", #name, #amount);
        }
    });

    // 4. Generate the final output code using `quote!`.
    let output = quote! {
        pub struct #struct_name;

        impl #struct_name {
            pub fn serves() -> u32 {
                #serves_val
            }
            pub fn print_ingredients() {
                println!("Ingredients for {}:", #struct_name_str);
                #( #ingredient_lines )*
            }
        }
    };

    output.into()
}
```


### Step 3: Using the Function-like Macro

Now, in our `my_kitchen` crate, we can use this clean, declarative syntax.

**`recipe_project/my_kitchen/src/main.rs`**

```rust
use chef_macros::recipe;

// This is our custom, domain-specific language in action!
recipe! {
    name: "Spaghetti Carbonara",
    serves: 4,
    ingredients: {
        "Pasta": "500g",
        "Guanciale": "150g",
        "Eggs": "4",
        "Pecorino Romano": "100g",
        "Black Pepper": "to taste"
    }
}

fn main() {
    println!("Serving Spaghetti Carbonara for {} people.", SpaghettiCarbonara::serves());
    SpaghettiCarbonara::print_ingredients();
}
```


### Step 4: Running the Code

From the `my_kitchen` directory:

```bash
cargo run
```

**Expected Output:**

```
Serving Spaghetti Carbonara for 4 people.
Ingredients for SpaghettiCarbonara:
- Pasta: 500g
- Guanciale: 150g
- Eggs: 4
- Pecorino Romano: 100g
- Black Pepper: to taste
```

**How It Worked**: The `recipe!` macro invocation was passed to our procedural macro as a `TokenStream`. We used `syn` to parse this stream according to our custom rules (defined in the `Parse` impl) into a structured `Recipe` object. This step allows us to validate the input and provide helpful error messages. Finally, we used the data from the `Recipe` struct with the `quote!` macro to generate a new `TokenStream` containing the `struct` and `impl` block, which replaced the original `recipe!` macro call.

## When to Use Function-like Macros

- **Creating Domain-Specific Languages (DSLs)**: This is their primary use case. When you want to create a rich, custom syntax for a specific problem (like SQL queries, HTML templating, or parser definitions).
- **Compile-Time Validation**: When you need to parse and validate complex input at compile time, which is beyond the capabilities of `macro_rules!`.
- **Interacting with External Files**: A function-like macro can read an external file (e.g., a `.sql` file or a `.html` template) at compile time and embed its contents directly into your Rust code, often after processing it. `include_str!` is a simple built-in example of this concept.
- **Advanced Code Generation**: When you need to generate code based on logic that is too complex for the simple pattern matching of declarative macros.

Function-like macros are the most flexible type of procedural macro, giving you complete control over both the input syntax and the generated output. They are a cornerstone of many advanced Rust libraries, providing ergonomic and type-safe interfaces for complex tasks.
<span style="display:none">[^2][^5][^6][^7][^8][^9]</span>

1. https://doc.rust-lang.org/reference/procedural-macros.html
2. https://www.freecodecamp.org/news/procedural-macros-in-rust/
3. https://stackoverflow.com/questions/58922119/how-do-i-create-a-function-like-procedural-macro
4. https://lukaswirth.dev/tlborm/proc-macros/methodical/function-like.html
5. https://earthly.dev/blog/rust-macros/
6. https://www.youtube.com/watch?v=crWfcA064is
7. http://developerlife.com/2022/03/30/rust-proc-macro/
8. https://rareskills.io/post/rust-function-like-macro
9. https://doc.rust-lang.org/book/ch19-06-macros.html
