---
title: "Declarative Macros - Common Macro Patterns"
description: "Discover common macro patterns and use cases to simplify repetitive Rust code."
date: 2025-10-06
weight: 5
---

# Declarative Macros in Rust: A Beginner's Guide to Common Patterns

Understanding declarative macros in Rust is like learning to **create your own custom kitchen shortcuts** – instead of manually writing out the same complex sequence of steps for every similar dish, you create a template (a macro) that does the repetitive work for you. This saves time, reduces errors, and makes your code more concise and readable. Just as a chef might have a shortcut for "dice and sauté," a Rust programmer can create a macro to handle common, repetitive code patterns.

## The Professional Kitchen Shortcut Analogy 🍽️

### Imagine You're a Chef Tired of Writing Out the Same Prep Steps Over and Over

**The Problem with Repetitive Code (Manual Prep):**

```rust
// ❌ Writing out the same boilerplate code repeatedly
// Check if a user has permission level "Admin"
let user1_permission = "Admin";
if user1_permission != "Admin" {
    return Err("Permission Denied: Not an Admin".to_string());
}

// Check if another user has permission level "Editor"
let user2_permission = "Viewer";
if user2_permission != "Editor" {
    return Err("Permission Denied: Not an Editor".to_string());
}

// This is verbose, error-prone, and hard to maintain.
```

**The Power of Declarative Macros (A Reusable Kitchen Shortcut):**

```rust
// ✅ Creating a shortcut to handle the repetitive check
macro_rules! ensure_permission {
    ($user_permission:expr, $required_permission:expr) => {
        if $user_permission != $required_permission {
            return Err(format!("Permission Denied: Expected '{}'", $required_permission));
        }
    };
}

// Now, using the shortcut is clean and clear
fn check_permissions() -> Result<(), String> {
    let user1_permission = "Admin";
    ensure_permission!(user1_permission, "Admin");

    let user2_permission = "Viewer";
    ensure_permission!(user2_permission, "Editor"); // This will correctly return an Err

    Ok(())
}
```

**Why Macros Are a Powerful Tool:**

- **DRY (Don't Repeat Yourself)**: Eliminate repetitive boilerplate code.[^2]
- **Domain-Specific Languages (DSLs)**: Create mini-languages tailored to your application's needs (like `vec![]` or `println!`).[^2]
- **Syntactic Abstraction**: Create syntax that wouldn't be possible with functions alone (e.g., taking a variable number of arguments).
- **Compile-Time Code Generation**: Macros write code for you at compile time, meaning there is no runtime performance cost.[^6]


## How Declarative Macros Work: `macro_rules!`

Declarative macros (also known as "macros by example") are defined with the `macro_rules!` macro. They work like a powerful `match` statement for your code's structure.[^5][^2]

**Basic Structure:**

```rust
macro_rules! macro_name {
    // A "matcher" arm
    ( pattern_to_match ) => {
        // The "transcriber" - the code to generate
        // ... code to output ...
    };
    // You can have multiple arms, just like a `match` statement
    ( another_pattern ) => {
        // ... different code to output ...
    };
}
```

- **Matcher**: The pattern on the left of `=>`. It describes the syntax that the macro call should have.
- **Transcriber**: The code on the right of `=>`. This is the template for the code that the macro will expand into.


### Fragment Specifiers: The Types of Code You Can Match

When you define a pattern, you capture parts of the input using `$` followed by a variable name (e.g., `$my_var`). You must also specify what *kind* of code fragment you expect. This is called a **fragment specifier**.


| Specifier | Matches... | Example |
| :-- | :-- | :-- |
| `expr` | An **expression** (e.g., `2 + 2`, `my_func()`) | `println!("{}", $my_expr:expr)` |
| `ident` | An **identifier** (e.g., a variable or function name) | `struct $MyStruct:ident { ... }` |
| `ty` | A **type** (e.g., `i32`, `String`, `&str`) | `let x: $MyType:ty;` |
| `stmt` | A **statement** (e.g., `let x = 5;`) | `$my_stmt:stmt` |
| `pat` | A **pattern** (e.g., `(a, b)`, `Some(x)`) | `match x { $my_pat:pat => ... }` |
| `item` | An **item** (e.g., a `struct`, `fn`, `enum`) | `$my_item:item` |
| `tt` | A **token tree** (any single token or a balanced group of `()`, `[]`, `{}`) | Captures almost anything. Used for complex cases. |
| `*` and `+` | Repetition (zero or more, one or more) | `$( $x:expr ),*` matches `1, 2, 3` |

## Common Macro Patterns and Use Cases

### Pattern 1: Reducing Boilerplate

This is the most common use case: creating a shortcut for repetitive code. Our `ensure_permission!` macro from the beginning is a perfect example.

**Another Example: A Quick `HashMap` Initializer**

Instead of writing `map.insert(...)` multiple times, let's create a macro to initialize a `HashMap` more cleanly.

```rust
use std::collections::HashMap;

macro_rules! map {
    // The pattern matches key-value pairs separated by commas
    // `$(...)*` means "repeat the inner pattern zero or more times"
    ( $( $key:expr => $value:expr ),* ) => {
        { // Use a block to create a new scope and return the map
            let mut temp_map = HashMap::new();
            // `$(...)*` in the transcriber repeats the code for each match
            $(
                temp_map.insert($key, $value);
            )*
            temp_map
        }
    };
}

fn main() {
    // A much cleaner way to create a HashMap
    let my_menu = map! {
        "Pizza" => 15.99,
        "Salad" => 10.50,
        "Coke" => 2.50
    };

    println!("Pizza costs: ${}", my_menu.get("Pizza").unwrap());
}
```

**How it works**: The `$(...),*` syntax is the key here. It tells the macro to match zero or more occurrences of the pattern inside (a key expression, `=>`, and a value expression), separated by commas. In the transcriber, the same `$(...)*` syntax loops through all the captured pairs and generates an `insert` call for each one.

### Pattern 2: Creating a Simple Domain-Specific Language (DSL)

Macros can create a more natural, readable syntax for a specific problem domain. The `vec!` macro is a classic example of this.

**Example: A Simple SQL-like `query!` Macro**

Let's imagine we want a simpler way to express database queries.

```rust
macro_rules! query {
    (SELECT $columns:expr FROM $table:ident WHERE $field:ident = $value:expr) => {
        format!(
            "SELECT {} FROM {} WHERE {} = '{}';",
            $columns,
            stringify!($table), // stringify! turns an identifier into a string
            stringify!($field),
            $value
        )
    };
}

fn main() {
    let user_id = 101;
    let sql_query = query!(SELECT "username, email" FROM users WHERE id = user_id);

    println!("Generated SQL: {}", sql_query);
    // Note: This is a simplified example. Real SQL queries should use
    // parameterized queries to prevent SQL injection!
}
```

**How it works**: This macro defines a very specific syntax. It will only match a macro call that literally contains `SELECT`, `FROM`, and `WHERE`. This makes the macro call read almost like natural language. `stringify!` is a built-in macro that converts an identifier like `users` into a string literal `"users"`.

### Pattern 3: Handling a Variable Number of Arguments

Functions in Rust require a fixed number of arguments. Macros do not. This is one of the most powerful features of macros and is used by `println!` and `vec!`.

**Example: A `sum!` Macro to Add Multiple Numbers**

Let's create a macro that can sum any number of expressions.

```rust
macro_rules! sum {
    // The base case: if called with a single expression, just return it.
    ($x:expr) => { $x };

    // The recursive case: match the first expression and the "rest" of them.
    // `$head` captures the first expression.
    // `$($tail:expr),+` captures one or more remaining expressions.
    ($head:expr, $($tail:expr),+) => {
        // The macro expands into the head plus a recursive call to itself
        // with the rest of the arguments.
        $head + sum!($($tail),+)
    };
}

fn main() {
    let total = sum!(1, 5, 3, 10);
    println!("Sum is: {}", total); // Prints "Sum is: 19"

    let single = sum!(42);
    println!("Single sum is: {}", single); // Prints "Single sum is: 42"
}
```

**How it works**: This is a **recursive macro**.

1. `sum!(1, 5, 3, 10)` matches the second arm: `$head` is `1`, and `$tail` is `5, 3, 10`.
2. It expands to: `1 + sum!(5, 3, 10)`.
3. The inner `sum!(5, 3, 10)` matches the second arm again: `$head` is `5`, `$tail` is `3, 10`.
4. It expands to: `1 + (5 + sum!(3, 10))`.
5. This continues until we have `1 + (5 + (3 + sum!(10)))`.
6. `sum!(10)` matches the **base case** (the first arm) and just expands to `10`.
7. The final expanded code is `1 + 5 + 3 + 10`, which the compiler then evaluates.

### Best Practices for Writing Declarative Macros

1. **Start with Functions**: Before writing a macro, ask yourself: "Can I do this with a function?" Functions are simpler, easier to debug, and better understood by tooling. Only use macros when a function won't work (e.g., for variable arguments or DSLs).
2. **Keep Macros Simple**: A complex macro can be very difficult to debug. If a macro becomes too large, consider breaking it into smaller helper macros.
3. **Use `stringify!` and `concat!`**: These built-in macros are very useful. `stringify!` turns a token into a string, and `concat!` concatenates string literals.
4. **Export with `#[macro_export]`**: If you want your macro to be usable outside of your crate, you must annotate it with `#[macro_export]` above the `macro_rules!` definition.
5. **Document Your Macros**: Write `///` doc comments for your macros just like you would for functions. Explain what they do and provide examples of how to use them.

By mastering these common patterns, you can leverage Rust's powerful macro system to write more expressive, concise, and maintainable code, effectively extending the language to suit the needs of your project.
<span style="display:none">[^1][^3][^4][^7][^8]</span>

1. https://doc.rust-lang.org/book/ch19-06-macros.html
2. https://dev.to/rogertorres/first-steps-with-rust-declarative-macros-1f8m
3. https://suchprogramming.com/rusty-macros/
4. https://www.youtube.com/watch?v=KsJHlqULpO4
5. https://earthly.dev/blog/rust-macros/
6. https://www.youtube.com/watch?v=dHv8FhcAl5U
7. https://www.freecodecamp.org/news/procedural-macros-in-rust/
8. https://www.reddit.com/r/rust/comments/18afxup/declarative_rust_macros_explanation/
