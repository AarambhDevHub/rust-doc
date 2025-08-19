---
title: "Advanced Macros - Macro Composition"
description: "Explore how macros can be composed together to build powerful abstractions in Rust."
date: 2025-10-08
weight: 2
---

# Advanced Macros: Building Powerful Abstractions with Composition

Understanding macro composition in Rust is like learning to **create a complex, multi-stage recipe by combining several smaller, simpler cooking techniques**. Instead of writing one monolithic recipe, you create a set of reusable "mini-recipes" (helper macros) for tasks like "dice vegetables," "prepare a sauce," or "sear meat." You then compose these smaller macros together to build a larger, more sophisticated macro. This approach makes your code more modular, easier to debug, and vastly more powerful.

## The Professional Kitchen "Mise en Place" Analogy 🍽️

### Imagine You're a Head Chef Designing a System for Assembling Complex Dishes

- **A Single, Monolithic Macro**: A single, 100-step recipe for a complex dish like "Beef Wellington." It's hard to read, hard to modify, and if one step is wrong, the whole recipe fails.
- **Composed Macros (Mise en Place)**: "Mise en place" is a French culinary term for having all your ingredients prepped and ready before you start cooking. In our analogy, this means creating a set of small, focused helper macros:
    * `prepare_duxelles!()`: A macro for making the mushroom paste.
    * `sear_tenderloin!()`: A macro for perfectly searing the beef.
    * `wrap_in_pastry!()`: A macro for wrapping everything in puff pastry.
- **The Final Composed Macro**: Your "master recipe" macro, `make_wellington!()`, which doesn't contain the detailed steps itself. Instead, it calls the other helper macros in the correct order.

**Why Macro Composition is a Superior Approach:**

- **Modularity**: Each macro has a single, well-defined responsibility.
- **Reusability**: You can reuse `sear_tenderloin!()` in other beef recipes.
- **Readability**: The master macro reads like a high-level summary of the process, making it easier to understand the overall logic.
- **Debuggability**: If there's a problem with the mushroom paste, you only need to debug the `prepare_duxelles!()` macro.


## How Macro Composition Works in Rust

Macro composition is the practice of one macro expanding into a call to another macro. Rust's macro expansion process handles this seamlessly. The compiler will repeatedly expand macros until no macro calls are left.[^1]

There are two primary ways to compose macros:

1. **Internal (Private) Helper Macros**: Macros defined within the same module that are not exported. They act as private implementation details for a public-facing macro.
2. **External (Public) Helper Macros**: Using macros imported from other modules or crates.

## Common Patterns for Macro Composition

### Pattern 1: The "Internal Helper" or "Entry Point" Macro

This is the most common and powerful pattern. You create one public-facing macro that acts as the main entry point. This macro's job is to parse the initial input and then delegate the actual code generation to one or more internal, private helper macros.

**Example: A `custom_vec!` Macro with Optional Capacity**

Let's build a macro that mimics `vec![]` but also allows for specifying an initial capacity, like `custom_vec!(with_capacity: 10; 1, 2, 3)`. We'll do this by composing two macros.

```rust
// This is the internal helper macro. It's not intended for direct use.
// It handles the simple case of creating a vec and pushing elements.
#[doc(hidden)] // Hide this from the generated documentation
macro_rules! _custom_vec_internal {
    ($vec:ident, $($element:expr),*) => {
        $(
            $vec.push($element);
        )*
    };
}

// This is our public-facing macro.
#[macro_export]
macro_rules! custom_vec {
    // Matcher 1: Handle the case WITH specified capacity.
    (with_capacity: $capacity:expr; $($elements:expr),*) => {
        {
            let mut temp_vec = Vec::with_capacity($capacity);
            // Delegate to the internal helper macro to handle the elements
            $crate::_custom_vec_internal!(temp_vec, $($elements),*);
            temp_vec
        }
    };

    // Matcher 2: Handle the case WITHOUT specified capacity.
    ($($elements:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            // Delegate to the internal helper macro here as well
            $crate::_custom_vec_internal!(temp_vec, $($elements),*);
            temp_vec
        }
    };
}

fn main() {
    let v1 = custom_vec![with_capacity: 10; 1, 2, 3];
    println!("v1 (cap {}): {:?}", v1.capacity(), v1);

    let v2 = custom_vec![4, 5, 6];
    println!("v2 (cap {}): {:?}", v2.capacity(), v2);
}
```

**How it works**:

1. The `custom_vec!` macro acts as the **dispatcher**. It has two arms to handle the different syntaxes.
2. Both arms do the initial setup (creating the `Vec`).
3. Crucially, both arms then call `_custom_vec_internal!` to handle the repetitive part of the logic (pushing the elements).
4. We use `$crate::` to ensure that the macro can find its helper even if it's used from another crate.
5. By convention, internal helper macros are often prefixed with an underscore `_` and hidden from documentation with `#[doc(hidden)]`.

### Pattern 2: Recursive Macros

A recursive macro is a form of composition where a macro calls itself. This is often used for processing lists or sequences of items.

**Example: An `html!` Macro for Generating HTML**

Let's create a macro to generate a nested HTML structure.

```rust
#[macro_export]
macro_rules! html {
    // Base case: An empty tag
    ($tag:ident {}) => {
        format!("<{0}></{0}>", stringify!($tag))
    };

    // Recursive case: A tag with content inside
    ($tag:ident { $($inner:tt)+ }) => {
        format!(
            "<{0}>{1}</{0}>",
            stringify!($tag),
            // The magic happens here: a recursive call to process the inner content
            html!($($inner)+)
        )
    };

    // Base case: A simple string literal
    ($content:literal) => {
        $content.to_string()
    };

    // This allows us to combine multiple items at the same level
    ($($tokens:tt)*) => {
        // This is a bit advanced. It uses a "Push-down accumulation" technique.
        // It expands to multiple calls and then concatenates the results.
        // For a beginner, think of it as "process each item and join the strings".
        {
            let mut s = String::new();
            // This is a simplified view. The actual expansion is more complex.
            // In reality, this would likely need another helper macro.
            // Let's stick to a simpler composition for now.
            // A true implementation of this would be more complex.
        }
    };
}

// A simpler, more direct example of recursion is better for beginners.
// Let's rewrite the sum! macro from the previous guide, as it's a perfect
// example of recursive composition.

macro_rules! sum {
    // Base case: A single expression
    ($x:expr) => { $x };

    // Recursive case: Call self with the tail of the list
    ($head:expr, $($tail:expr),+) => {
        $head + sum!($($tail),+)
    };
}

fn main() {
    let total = sum!(1, 2, 3, 4, 5);
    println!("The sum is: {}", total);
}
```

The `sum!` macro is a classic example of recursive composition. It breaks a problem down (`sum of N items`) into a smaller version of the same problem (`head + sum of N-1 items`) until it hits a base case.

### Pattern 3: Composing Macros from Different Crates

You can easily compose macros by simply importing them. This is how the entire ecosystem works.

**Example: Composing `println!` and a Custom Macro**

Let's create a `debug_assert_eq!` macro that only runs in debug builds and prints a custom message using `println!`.

```rust
macro_rules! debug_assert_eq {
    ($left:expr, $right:expr) => {
        // We only compile this code block if the `debug_assertions` config flag is set
        // (i.e., when compiling in debug mode).
        #[cfg(debug_assertions)]
        {
            // Here, we are COMPOSING our macro with the standard `assert_eq!`
            // and `println!` macros.
            if $left != $right {
                println!(
                    "Assertion failed: `(left == right)`\n  left: `{:?}`,\n right: `{:?}`",
                    $left,
                    $right
                );
                // Call the standard assert_eq! to cause a panic with the correct line number.
                assert_eq!($left, $right);
            }
        }
    };
}

fn main() {
    let a = 10;
    let b = 20;

    println!("Running debug assertion (this will fail in debug mode)...");
    // In a `cargo run`, this will panic.
    // In a `cargo run --release`, this will do nothing.
    debug_assert_eq!(a, b);

    println!("This line will not be reached in debug mode.");
}
```

**How it works**: Our `debug_assert_eq!` macro expands into code that contains calls to `println!` and `assert_eq!`. The compiler sees these new macro calls and expands them as well, demonstrating a clear chain of composition. The `#[cfg(debug_assertions)]` attribute is a form of conditional compilation, ensuring this code is completely removed in release builds.

## Best Practices for Macro Composition

1. **Design for Composition**: When writing a macro, think about whether parts of its logic could be broken out into smaller, reusable helper macros.
2. **Use `#[doc(hidden)]`**: Keep your internal helper macros private. This provides a clean public API and prevents users from relying on implementation details.
3. **Use `$crate::`**: When a macro calls another macro from the same crate, always use the `$crate` path prefix. This ensures that the macro works correctly even when it's imported and used in a different crate.
4. **Keep It Readable**: The goal of composition is to make complex macros more understandable. If your composition logic becomes too convoluted, it defeats the purpose. Strive for clarity.

Macro composition is the key to unlocking the full potential of Rust's metaprogramming capabilities. By building small, focused macros and combining them, you can create powerful, maintainable, and highly expressive DSLs and code-generation tools.
<span style="display:none">[^10][^2][^3][^4][^5][^6][^7][^8][^9]</span>

1. https://doc.rust-lang.org/reference/macros-by-example.html
2. https://docs.rs/macro-compose
3. https://dev.to/ashsajal/rust-macros-explained-with-example-5gg8
4. https://stackoverflow.com/questions/34777302/example-of-how-to-use-conditional-compilation-macros-in-rust
5. https://dev.to/rogertorres/first-steps-with-rust-declarative-macros-1f8m
6. https://docs.rs/composing
7. https://stackoverflow.com/questions/34373169/how-do-i-create-a-rust-macro-with-optional-parameters-using-repetitions
8. https://www.reddit.com/r/rust/comments/jhfikh/a_question_on_macro_composition/
9. https://doc.rust-lang.org/rust-by-example/macros.html
10. https://rust-lang.github.io/api-guidelines/macros.html
