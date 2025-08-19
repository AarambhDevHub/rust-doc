---
title: "Advanced Macros - Debugging Macros"
description: "Techniques and tools for debugging declarative and procedural macros effectively."
date: 2025-10-08
weight: 3
---

# Advanced Macros: Debugging Techniques for Rust Macros

Debugging macros in Rust can be challenging because they operate at compile time, transforming your code before it's even fully compiled. A bug in a macro often results in a confusing error message that points to the code *using* the macro, not the macro's definition itself. Understanding how to peel back this layer of abstraction is key to effective debugging.

## The "Magic Trick" Analogy for Debugging 🎩

Imagine you're trying to figure out how a magician performs a magic trick.

- **The Macro Invocation**: The magician on stage performing the trick (e.g., pulling a rabbit out of a hat).
- **The Macro Expansion**: What's happening behind the scenes—the hidden compartments, the sleight of hand.
- **The Compiler Error**: The audience being baffled and saying, "That's impossible! Hats don't produce rabbits!" The complaint is about the final result, not the hidden mechanism.

To debug the trick, you can't just watch the final performance over and over. You need tools to see what's happening backstage. Similarly, to debug a macro, you need tools to see the code it generates.

***

This guide will cover the essential tools and techniques for debugging both declarative (`macro_rules!`) and procedural macros, ordered from simplest to most powerful.

## 1. The Essential Tool for All Macro Debugging: `cargo expand`

This is the single most important tool in your macro debugging toolkit. The `cargo-expand` subcommand lets you see the fully expanded source code that your macro generates, *without* compiling it.[^1]

**How to Use It:**

1. **Install it**: If you haven't already, install the tool with Cargo:

```bash
cargo install cargo-expand
```

2. **Run it**: In your project directory, run:

```bash
cargo expand
```

This will print the fully expanded code for your entire crate to the console. You can also expand specific modules (`cargo expand a::b::c`) or tests (`cargo expand --test my_test`).

**Example Scenario:**
Let's say you have a simple declarative macro that's causing a problem.

```rust
macro_rules! create_fn {
    ($fn_name:ident) => {
        fn $fn_name() {
            let x = "hello"; // Oops, forgot a semicolon
            println!("{}", x);
        }
    };
}

create_fn!(my_cool_function); // The compiler error will point to this line.

fn main() {
    my_cool_function();
}
```

The compiler error will be confusing. But if you run `cargo expand`, you will see the generated code clearly:

```rust
// Output from `cargo expand`
fn my_cool_function() {
    let x = "hello" // Ah, there's the missing semicolon!
    println!("{}", x);
}

fn main() {
    my_cool_function();
}
```

`cargo expand` is your first and best step for almost any macro bug. It shows you exactly what the compiler is seeing.

## 2. Debugging Declarative Macros (`macro_rules!`)

Declarative macros are simpler, and so are their debugging techniques.

### Technique 1: `trace_macros!` (Nightly Only)

The `trace_macros!` feature provides a step-by-step log of how a macro is expanded. It's especially useful for debugging complex or recursive macros.[^2]

**How to Use It:**

- You must be on the nightly Rust toolchain (`rustup default nightly`).
- Enable the feature at the top of your file: `#![feature(trace_macros)]`.
- Wrap the macro call you want to debug:

```rust
#![feature(trace_macros)]

macro_rules! recursive_sum {
    ($x:expr) => { $x };
    ($head:expr, $($tail:expr),+) => {
        $head + recursive_sum!($($tail),+)
    };
}

fn main() {
    trace_macros!(true); // Turn on tracing
    let total = recursive_sum!(1, 2, 3);
    trace_macros!(false); // Turn it off

    println!("{}", total);
}
```

When you compile this, the compiler will print a detailed trace showing each step of the recursive expansion, helping you pinpoint where it's going wrong.

### Technique 2: `log_syntax!` (Nightly Only)

Similar to `trace_macros!`, `log_syntax!` simply prints the tokens passed to a macro invocation. It's a quick way to see exactly what your macro is receiving.[^2]

```rust
#![feature(log_syntax)]

macro_rules! my_macro {
    ($($tokens:tt)*) => {
        log_syntax!($($tokens)*); // Print the tokens we received
    };
}

my_macro!(some tokens here 123);
```


## 3. Debugging Procedural Macros

Procedural macros are actual Rust code, which means you have more powerful debugging options, but also more complexity. Since they are compiled separately and run as part of the compiler, you can't use a traditional debugger.

### Technique 1: The "Print Debugging" Gold Standard

This is the most common and effective technique. Since a proc-macro can write to `stderr`, you can print the generated code to the console during the build process.[^3][^4]

```rust
// In your procedural macro crate
use proc_macro::TokenStream;
use quote::quote;
use syn::parse_macro_input;

#[proc_macro_derive(MyMacro)]
pub fn my_macro_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as syn::DeriveInput);
    let name = &ast.ident;

    let output = quote! {
        impl MyTrait for #name {
            fn my_trait_method() {
                println!("Hello from {}!", stringify!(#name));
            }
        }
    };

    // The key debugging line:
    eprintln!("Generated code:\n{}", output.to_string());

    output.into()
}
```

Now, whenever you compile a crate that uses `#[derive(MyMacro)]`, the full text of the generated `impl` block will be printed to your console. This, combined with `cargo expand`, gives you a very clear picture of what's happening.

### Technique 2: Using `dbg!` Inside a Procedural Macro

The trusty `dbg!` macro works inside procedural macros too! It's perfect for inspecting the `syn` data structures that represent your code.[^4][^5]

```rust
#[proc_macro_attribute]
pub fn my_attribute(_attr: TokenStream, item: TokenStream) -> TokenStream {
    let func = parse_macro_input!(item as syn::ItemFn);

    // Let's inspect the parsed function signature
    dbg!(&func.sig);

    // ... rest of the macro logic ...

    let output = quote! { #func };
    output.into()
}
```

When you compile, `dbg!` will print a detailed, `Debug`-formatted representation of the `syn::Signature` struct to `stderr`, allowing you to see exactly how `syn` parsed your function.

### Technique 3: Test-Driven Development

Because procedural macros can be difficult to debug interactively, writing thorough tests is crucial. The `proc-macro2` crate is designed to be used outside of a proc-macro context, which makes it perfect for unit testing.[^4]

You can write tests that:

1. Create a `proc_macro2::TokenStream` from a string of code.
2. Pass this token stream to your macro's implementation function.
3. Parse the resulting token stream and assert that it has the structure you expect.

This allows you to test your macro logic in a standard `cargo test` environment, which is much faster and easier than the compile-run-debug loop.

### Technique 4: When All Else Fails - The "Compiler Error" Debugger

Sometimes, the simplest way to get information is to make the macro intentionally fail to compile. You can return a `syn::Error` that contains a debug message.

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, spanned::Spanned};

#[proc_macro_derive(MyMacro)]
pub fn my_macro_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as syn::DeriveInput);

    // Create an error that points to the struct's name and contains a debug message.
    let debug_message = format!("DEBUG: The struct name is {}", ast.ident);
    return syn::Error::new(ast.span(), debug_message).to_compile_error().into();

    // ... unreachable code ...
}
```

When you try to use this macro, the compiler will halt with your custom error message, effectively turning compiler errors into a primitive debugging tool. It's ugly, but it works in a pinch.

## Summary of Debugging Strategies

| **Technique** | **Macro Type** | **When to Use** |
| :-- | :-- | :-- |
| **`cargo expand`** | Both | **Always start here.** See the final output of your macro. |
| **`trace_macros!`** | Declarative (Nightly) | For debugging complex or recursive `macro_rules!`. |
| **Print-to-`stderr`** | Procedural | To see the generated code from a proc-macro during compilation. Your primary tool. |
| **`dbg!`** | Procedural | For inspecting the intermediate `syn` data structures while parsing the input. |
| **Test-Driven Development** | Procedural | For robustly testing macro logic without needing to compile a separate crate every time. |
| **Intentional `syn::Error`** | Procedural | As a last resort to quickly get debug information out of the compiler. |


1. https://lukaswirth.dev/tlborm/syntax-extensions/debugging.html
2. https://stackoverflow.com/questions/30200374/how-do-i-debug-macros
3. https://dev.to/tramposo/understanding-rust-macros-a-comprehensive-guide-for-developers-am4
4. https://www.reddit.com/r/rust/comments/ixvbl7/how_do_you_debug_while_developing_proc_macros/
5. https://doc.rust-lang.org/std/macro.dbg.html
6. https://users.rust-lang.org/t/proc-macro-debugging-tricks/69343
7. https://palant.info/2023/04/17/processing-a-complex-syntax-with-rusts-declarative-macros/
8. https://stackoverflow.com/questions/75420558/how-to-debug-a-custom-proc-macro
9. https://www.reddit.com/r/rust/comments/1l0us8z/how_to_debug_gracefully_in_procedural_macros/
10. https://users.rust-lang.org/t/debugging-proc-macro/29821
