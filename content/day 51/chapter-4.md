---
title: "Declarative Macros - Macro Hygiene"
description: "Learn about macro hygiene to avoid variable capture and ensure safe macro expansions."
date: 2025-10-06
weight: 4
---

# Declarative Macros and Hygiene in Rust: A Beginner's Guide

Understanding macro hygiene in Rust is like learning the **etiquette of being a helpful assistant in a professional kitchen**. A good assistant helps with tasks without getting in the way, using their own tools and not accidentally swapping the chef's salt for sugar. An "unhygienic" assistant, however, might grab any container they see, potentially mixing up ingredients and causing chaos. Macro hygiene is the set of rules that ensures macros act like good, "hygienic" assistants, preventing them from accidentally interfering with the code they are helping to write.

## The Professional Kitchen Assistant Analogy 🍽️

### Imagine You're a Head Chef with an Assistant (a Macro)

- **Your Code**: The chef's carefully arranged workspace, with ingredients like `salt`, `sugar`, and `flour`.
- **A Macro**: An assistant you can call upon to perform a repetitive task, like "prepare a basic dough."

***

### An Unhygienic Assistant (An Unhygienic Macro)

Imagine you tell your assistant, "Prepare the dough using `flour`." The assistant looks around, sees your special bag of "cake flour," and uses that. But you wanted them to use their own standard "bread flour." The assistant has accidentally "captured" a variable from your environment, leading to the wrong result.

This is **variable capture**, a classic hygiene problem.

### A Hygienic Assistant (A Hygienic Macro)

A well-trained, hygienic assistant knows to *only* use the tools and ingredients they brought with them. When you say, "Prepare the dough," they use *their own* `flour` from their own kit, never touching your `flour`.

Rust's declarative macros (`macro_rules!`) are designed to be **partially hygienic**, meaning they automatically handle this separation for you in most cases, especially with local variables.[^5]

## What is Macro Hygiene?

Macro hygiene is a set of principles that prevent macros from causing unintended side effects by clashing with the code where they are used. The two main problems hygiene solves are :[^4][^6]

1. **Variable Capture**: Preventing variables inside the macro from conflicting with variables outside the macro.
2. **Item/Path Ambiguity**: Ensuring that functions, types, and other items called inside the macro resolve to the correct definition, regardless of where the macro is called.

Rust's declarative macros are excellent at handling the first problem but require manual care for the second.

## How Rust's Hygiene Works for Local Variables (Automatic)

Rust's `macro_rules!` macros are hygienic with respect to **local variables**. This means that a variable defined inside a macro will *not* conflict with a variable of the same name outside the macro, and vice versa. The compiler achieves this by attaching an invisible "syntax context" or "color" to every identifier.[^5]

### Example 1: Preventing the Macro from Clashing with Your Code

Let's create a macro that uses an internal variable named `temp`.

```rust
macro_rules! double_and_log {
    ($x:expr) => {
        // This `temp` is "colored" with the macro's context
        let temp = $x * 2;
        println!("Double of {} is {}", stringify!($x), temp);
        temp
    };
}

fn main() {
    // This `temp` is "colored" with the main function's context
    let temp = "I am a local variable.";

    let result = double_and_log!(5);

    println!("Result: {}", result);
    println!("My local variable is still: '{}'", temp);
}
```

**Output:**

```
Double of 5 is 10
Result: 10
My local variable is still: 'I am a local variable.'
```

Even though both the macro and `main` have a variable named `temp`, they do not interfere with each other. Rust's hygiene keeps them separate, as if they were named `temp_macro` and `temp_main`. This is a huge safety feature![^3]

### Example 2: Preventing Your Code from Clashing with the Macro

The reverse is also true. A macro cannot accidentally access a variable from the scope where it's called.

```rust
macro_rules! add_a {
    ($e:expr) => {
        // This `a` is from the macro's context
        // and is completely separate from the `a` in `main`.
        $e + a
    }
}

fn main() {
    // This `a` is from the main function's context.
    let a = 10;

    // This will fail to compile!
    // let result = add_a!(5);
    // error[E0425]: cannot find value `a` in this scope
}
```

This fails because the `a` inside the macro is considered undefined. The macro cannot "see" the `let a = 10;` in `main`. This prevents macros from having spooky, action-at-a-distance behavior and makes them much more predictable.[^5]

## The Unhygienic Part: Paths and Items

While local variables are handled automatically, paths to functions, structs, and other items are **not** hygienic by default. This can lead to problems.[^8]

### The Problem: Ambiguous Paths

Imagine you write a macro in your crate that needs to create a `Vec`.

```rust
// In your crate `my_lib`
#[macro_export]
macro_rules! make_a_vec {
    ($($elem:expr),*) => {
        // What `Vec` is this? Is it `std::vec::Vec`?
        // Or did the user define their own `struct Vec`?
        Vec::from([$($elem),*])
    };
}
```

Now, a user of your library does this:

```rust
use my_lib::make_a_vec;

// The user mischievously defines their own `Vec`
struct Vec;

fn main() {
    // This will now fail to compile because it tries to call
    // `Vec::from` on the user's struct, not `std::vec::Vec`.
    let my_vec = make_a_vec![1, 2, 3];
}
```

This is an example of an **unhygienic path**. The meaning of `Vec` changed depending on the context where the macro was called.

### The Solution: Using Absolute Paths

To write hygienic macros, you must use **absolute paths** for all external items, starting with `::`. This tells the compiler to always start its search from the root of all crates.[^1]

**Hygienic Version:**

```rust
#[macro_export]
macro_rules! make_a_vec_hygienic {
    ($($elem:expr),*) => {
        // Always start from the crate root `::`
        ::std::vec::Vec::from([$($elem),*])
    };
}
```

Now, this macro will *always* use `std::vec::Vec`, regardless of what the user has defined in their local scope.

### The Solution: Using `$crate` for Items Within Your Crate

What if your macro needs to call another function or use a struct from *your own* crate? You might be tempted to write `my_lib::MyStruct`. But what if the user renames your crate in their `Cargo.toml`?

The special metavariable `$crate` solves this. `$crate` expands to an absolute path to the root of the crate where the macro was **defined**.[^4]

**Example:**

```rust
// In your crate `my_lib`

pub struct HelperStruct;

#[macro_export]
macro_rules! use_helper {
    () => {
        // Use `$crate` to refer to items in your own crate hygienically.
        // This will work even if the user renames `my_lib`.
        let _ = $crate::HelperStruct;
    };
}
```


## Best Practices for Writing Hygienic Declarative Macros

1. **Trust Local Variable Hygiene**: You don't need to worry about local variable names clashing. Rust handles this for you. Don't try to invent clever unique names like `__temp_my_macro`; it's unnecessary.
2. **Use Absolute Paths for External Items**: Always begin paths to items from other crates (including `std`) with `::`.
    * **Bad**: `Option<T>`, `std::option::Option<T>`
    * **Good**: `::std::option::Option<T>`
3. **Use `$crate` for Internal Items**: When a macro needs to refer to another public item within the same crate, use the `$crate` metavariable to create a robust, absolute path.
    * **Bad**: `my_crate::MyStruct`
    * **Good**: `$crate::MyStruct`
4. **Pass Functions and Types as Macro Arguments**: If a macro needs to call a function or use a type that should be provided by the *user*, pass it as a macro argument. This makes the dependency explicit and avoids hygiene issues.

**Example of Passing a Type:**

```rust
macro_rules! create_container_of {
    ($container_type:ty, $value:expr) => {
        let mut container: $container_type = ::std::default::Default::default();
        container.push($value); // Assumes a `push` method
        container
    };
}

fn main() {
    // The user explicitly provides `Vec<i32>` as the type.
    let my_vec = create_container_of!(Vec<i32>, 10);
    assert_eq!(my_vec, vec![^10]);
}
```

By following these simple rules, you can write powerful declarative macros that are safe, reliable, and "just work" in any context, embodying the safety and predictability that are core to the Rust programming language.
<span style="display:none">[^2][^7][^9]</span>

1. https://gist.github.com/Kestrer/8c05ebd4e0e9347eb05f265dfb7252e1
2. https://users.rust-lang.org/t/where-to-learn-the-precise-kind-of-hygiene-that-rust-macros-provide/81626
3. https://dev.to/tramposo/understanding-rust-macros-a-comprehensive-guide-for-developers-am4
4. https://earthly.dev/blog/rust-macros/
5. https://lukaswirth.dev/tlborm/decl-macros/minutiae/hygiene.html
6. https://www.swiftorial.com/tutorials/programming_languages/rust/macros/advanced_macro_techniques
7. https://www.c-sharpcorner.com/article/macros-in-rust/
8. https://doc.rust-lang.org/reference/macros-by-example.html
9. https://www.reddit.com/r/learnrust/comments/s3mkjs/macro_hygiene/
10. https://internals.rust-lang.org/t/explicit-monomorphization-for-compilation-time-reduction/15907
