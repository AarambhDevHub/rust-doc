---
title: "Declarative Macros - Pattern Matching in Macros"
description: "Understand how pattern matching works in declarative macros to create flexible macro rules."
date: 2025-10-06
weight: 2
---

# Pattern Matching in Rust's Declarative Macros: A Beginner's Guide

Understanding pattern matching in Rust's declarative macros (`macro_rules!`) is like learning to **create a highly flexible ordering system for a professional restaurant**. Instead of having just one fixed way to take an order, you want to allow waiters to use different formats—a simple order, an order with special instructions, or even a list of several orders at once. Declarative macros use a powerful pattern-matching system to recognize these different input formats and generate the correct code for each one, making them an incredibly flexible tool for reducing boilerplate.

## The Professional Restaurant Ordering System Analogy 🍽️

### Imagine You're Designing an Ordering System That Can Handle Any Request

- **A Macro Invocation**: A waiter taking an order from a customer.
- **A Macro Rule**: A specific format the waiter can use to write down the order (e.g., "Dish Name," "Dish Name with note 'extra spicy'," "List of Dishes").
- **Pattern Matching**: The kitchen's ability to read the order ticket, understand which format was used, and prepare the dish accordingly.

***

### The Problem without Flexible Patterns (A Rigid Ordering System)

If your ordering system only understands one exact format, it's brittle and hard to use.

```rust
// ❌ A rigid macro that only accepts one format
// create_order!(Pizza);
// What if we want to add a note? Or multiple items?
// create_order!(Pizza, "extra cheese"); // This would fail.
```


### The Power of Pattern Matching (A Flexible Ordering System)

With pattern matching, you can define multiple rules to handle different kinds of input.

```rust
// ✅ A flexible macro that understands multiple formats
macro_rules! create_order {
    // Rule 1: A single dish name
    ($dish:expr) => { /* ... */ };
    // Rule 2: A dish name with a special instruction
    ($dish:expr, $note:literal) => { /* ... */ };
    // Rule 3: A list of multiple dishes
    (for $customer:ident: $($dish:expr),+) => { /* ... */ };
}

// Now all of these are valid!
create_order!("Pizza");
create_order!("Steak", "medium-rare");
create_order!(for Alice: "Soup", "Salad", "Bread");
```

This flexibility is the core strength of declarative macros.

## The Anatomy of a `macro_rules!` Definition

A `macro_rules!` definition consists of one or more "rules," each with two parts separated by `=>` :[^3][^6]

```rust
macro_rules! my_macro {
    ( PATTERN_1 ) => { TRANSCRIPTION_1 };
    ( PATTERN_2 ) => { TRANSCRIPTION_2 };
    // ... more rules
}
```

- **Pattern (The "Matcher")**: This is the syntax that the macro invocation must match. It's similar to a `match` arm. The Rust compiler checks the macro invocation against these patterns from top to bottom and uses the first one that matches.[^6]
- **Transcription (The "Transcriber")**: This is the code that will be generated and substituted in place of the macro call. It can use variables captured in the pattern.


## Capturing Code with "Metavariables" and "Fragment Specifiers"

To make patterns flexible, you can "capture" parts of the input into **metavariables**. These are like variables for your macro, and they always start with a `$`.[^1]

Each metavariable must be paired with a **fragment specifier**, which tells the compiler what kind of Rust code to expect and capture.[^1]


| Fragment Specifier | What It Matches | Example Capture |
| :-- | :-- | :-- |
| `ident` | An identifier (e.g., a variable name, function name, struct name). | `my_var`, `User` |
| `expr` | An expression (e.g., `2 + 2`, `my_func()`, `"hello"`). | `"hello"` |
| `ty` | A type (e.g., `i32`, `String`, `Vec<u8>`). | `String` |
| `pat` | A pattern (like those used in `match` arms, e.g., `Some(x)`). | `(x, y)` |
| `path` | A path (e.g., `std::collections::HashMap`). | `std::vec` |
| `stmt` | A single statement (e.g., `let x = 5;`). | `let x = 5;` |
| `block` | A block of code surrounded by curly braces (`{ ... }`). | `{ ... }` |
| `item` | An item (e.g., a function, struct, or module definition). | `fn foo() {}` |
| `literal` | A literal (e.g., `"hello"`, `42`, `true`). | `true` |
| `tt` | A single "token tree" (a flexible catch-all for complex or unknown syntax) . | `+` or `foo` |

## Pattern Matching Examples: From Simple to Advanced

### Example 1: Matching a Single Expression

Let's create a `four_times!` macro that multiplies an expression by four.

```rust
macro_rules! four_times {
    // Pattern: Matches any single expression and captures it in the metavariable `$e`.
    // The fragment specifier is `expr`.
    ($e:expr) => {
        // Transcription: Generates code that multiplies the captured expression by 4.
        $e * 4
    };
}

fn main() {
    let result1 = four_times!(5); // $e becomes `5`
    let result2 = four_times!(10 + 1); // $e becomes `10 + 1`

    println!("Result 1: {}", result1); // Prints 20
    println!("Result 2: {}", result2); // Prints 44
}
```


### Example 2: Multiple Rules for Different Patterns

Let's expand our `create_order!` macro to handle different ways of ordering.

```rust
macro_rules! create_order {
    // Rule 1: A single dish.
    // Matches a single expression.
    ($dish:expr) => {
        println!("Received a simple order for: {}", $dish);
    };

    // Rule 2: A dish with a special note.
    // Matches an expression, followed by a comma, followed by a literal.
    ($dish:expr, $note:literal) => {
        println!("Received an order for: {} with note: '{}'", $dish, $note);
    };
}

fn main() {
    // This invocation matches Rule 1
    create_order!("Spaghetti");

    // This invocation matches Rule 2
    create_order!("Steak", "make it medium-rare");
}
```


### Example 3: Repetition with `$(...)*` and `$(...)+`

This is the most powerful feature of declarative macros. You can match a repeating pattern.

- `$(...)*`: Matches **zero or more** repetitions.
- `$(...)+`: Matches **one or more** repetitions.

The pattern inside `$(...)` is what repeats, and you can specify a separator (like a comma) between the repetitions.

Let's create a `make_dishes!` macro that can take a list of ingredients for multiple dishes.

```rust
macro_rules! make_dishes {
    // Pattern:
    // Matches the literal `make` followed by...
    // one or more repetitions of `(dish_name: ingredients, ...)`
    // The repetition is captured in `$dish_name` and `$($ingredient),*`.
    // The `$(...)+` means we must have at least one dish.
    (make $( $dish_name:ident: [ $($ingredient:expr),* ] );+ ) => {
        // Transcription:
        // We use the same `$(...)+` syntax to generate code for each repetition.
        $(
            println!("Preparing dish: {}", stringify!($dish_name));
            // We can even have nested repetition!
            // This `$(...)*` generates a `println!` for each ingredient.
            $(
                println!("  - Adding {}", $ingredient);
            )*
        )+
    };
}

fn main() {
    make_dishes! {
        make
        salad: ["lettuce", "tomato", "cucumber"];
        soup: ["broth", "carrots", "celery"];
        dessert: ["ice cream", "chocolate sauce"];
    };
}
```

**Output**:

```
Preparing dish: salad
  - Adding lettuce
  - Adding tomato
  - Adding cucumber
Preparing dish: soup
  - Adding broth
  - Adding carrots
  - Adding celery
Preparing dish: dessert
  - Adding ice cream
  - Adding chocolate sauce
```

This shows how you can define a complex, custom Domain-Specific Language (DSL) right inside Rust.

## Best Practices for Macro Pattern Matching

1. **Be Specific**: Use the most specific fragment specifier possible (e.g., `ident` instead of `expr` if you know you need an identifier). This provides better error messages if the macro is used incorrectly.
2. **Use Braces or Brackets for Clarity**: While macros can be invoked with `()`, `[]`, or `{}`, using `vec!{...}` or `my_macro!{...}` can improve readability and avoid certain parsing ambiguities.
3. **Start with Simple Rules First**: If you have multiple rules, place the most specific or simple ones at the top. The compiler checks them in order.
4. **Use Recursion for Complex Parsing**: For very complex patterns that can't be handled by simple repetition, you can use a "recursive" macro pattern. One macro rule processes one piece of the input and then calls itself with the rest of the input. This is an advanced technique but incredibly powerful.[^5]
5. **Document Your Macro's Syntax**: Since you are creating a custom mini-language, it's crucial to document how to use your macro, including all the valid patterns it accepts.

By mastering pattern matching, you can elevate your use of declarative macros from simple code shortcuts to creating expressive, flexible, and powerful DSLs that make your codebases more concise and readable.
<span style="display:none">[^2][^4][^7][^8]</span>

1. https://doc.rust-lang.org/book/ch19-06-macros.html
2. https://users.rust-lang.org/t/matching-rust-code-via-macro-rules/112286
3. https://pratapsharma.io/rust-macro-rules-guide/
4. https://www.reddit.com/r/rust/comments/rsfw9b/how_do_you_define_a_proc_macro_using_pattern/
5. https://palant.info/2023/04/17/processing-a-complex-syntax-with-rusts-declarative-macros/
6. https://suchprogramming.com/rusty-macros/
7. https://users.rust-lang.org/t/pattern-to-type-in-macro/85467
8. https://www.youtube.com/watch?v=Ij20OxPFrCw
