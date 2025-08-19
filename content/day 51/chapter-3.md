---
title: "Declarative Macros - Repetition and Optional Patterns"
description: "Explore repetition and optional patterns in macros to handle lists, sequences, and optional arguments."
date: 2025-10-06
weight: 3
---

# Declarative Macros in Rust: Repetition and Optional Patterns

Understanding repetition and optional patterns in Rust's declarative macros is like learning to **write a flexible recipe for a professional kitchen**. Instead of writing one recipe for a single serving and another for a party of ten, you want one master recipe that can handle *any* number of ingredients or even optional ones. Declarative macros, with their special repetition (`$()*`) and optional (`$()?`) syntax, allow you to write this kind of flexible, powerful code that adapts to different inputs.

## The Professional Kitchen Recipe Analogy 🍽️

### Imagine You're Writing a Master Recipe for Your Restaurant

- **A Simple Macro**: A recipe for making exactly one pancake. It takes one set of ingredients and produces one pancake.
- **Repetition (`$()*`)**: A recipe for making *any number* of pancakes. It says, "For each set of ingredients you are given, make one pancake." This allows you to handle a list or sequence of inputs.
- **Optionality (`$()?`)**: A recipe that says, "You can optionally add chocolate chips." It provides a way to handle an argument that may or may not be present.

This guide will teach you how to write these flexible "master recipes" using Rust's `macro_rules!`.

## The Foundation: A Quick Macro Refresher

Declarative macros in Rust are defined with `macro_rules!`. They work like a `match` expression for your code.[^3]

- **Matcher**: The pattern on the left side of `=>` that defines the syntax your macro accepts.
- **Transcriber**: The code on the right side of `=>` that is generated when the pattern matches.
- **Fragment Specifiers**: Placeholders like `$name:expr` that capture a piece of Rust code (in this case, an expression).[^5]

```rust
macro_rules! say_hello {
    // Matcher: captures one expression named `$name`
    ($name:expr) => {
        // Transcriber: generates a println! statement
        println!("Hello, {}!", $name);
    };
}

fn main() {
    say_hello!("World"); // Expands to: println!("Hello, {}!", "World");
}
```


## Repetition: Handling Lists and Sequences (`$()*`)

The real power of macros comes from their ability to handle repeated patterns. This is done using the `$()*` syntax, which means "match the preceding pattern zero or more times".[^4]

- **Syntax**: `$( ... )*` or `$( ... ),*` (with a separator)
- **How it works**:

1. The pattern inside the `$(...)` is matched against the input.
2. The `*` (or `+` for one or more) tells the macro to keep matching this pattern as long as it can.
3. In the transcriber, you use `$()*` again to generate code for each matched instance.


### Example 1: A `sum!` Macro

Let's create a macro that can sum any number of expressions. This is something a regular function cannot do, as functions have a fixed number of arguments.

```rust
macro_rules! sum {
    // The matcher:
    // 1. `$( $item:expr )`   : Match any expression and call it `$item`.
    // 2. `$( ... ),*`         : The pattern can repeat, separated by commas.
    ( $( $item:expr ),* ) => {
        {
            // We start with a base value of 0.
            let mut total = 0;
            // The transcriber:
            // 1. `$( ... )*`      : For each item that was matched...
            // 2. `total += $item;`: ...generate this line of code.
            $(
                total += $item;
            )*
            total // Return the final total
        }
    };
}

fn main() {
    let my_sum = sum!(1, 2, 3 + 3, 10);
    println!("The sum is: {}", my_sum); // Prints: The sum is: 19

    let another_sum = sum!(); // Also works with zero arguments
    println!("An empty sum is: {}", another_sum); // Prints: An empty sum is: 0
}
```

**How the `sum!(1, 2, 6)` expansion works:**

1. **Matcher**:
    - `$item` matches `1`.
    - `$item` matches `2`.
    - `$item` matches `6`.
2. **Transcriber**: The `$( total += $item; )*` block expands to:

```rust
total += 1;
total += 2;
total += 6;
```

The final macro call becomes:

```rust
{
    let mut total = 0;
    total += 1;
    total += 2;
    total += 6;
    total
}
```


### Example 2: A `make_functions!` Macro

Let's create a macro to reduce boilerplate when defining many simple functions.

```rust
macro_rules! make_functions {
    // Match one or more function names, separated by commas.
    ( $( $func_name:ident ),+ ) => {
        // For each matched function name...
        $(
            // ...generate a new function with that name.
            fn $func_name() {
                println!("You called function: {}", stringify!($func_name));
            }
        )+
    };
}

// Now, create three functions with one macro call
make_functions!(cook_pizza, serve_drinks, clean_tables);

fn main() {
    cook_pizza();    // Prints: You called function: cook_pizza
    serve_drinks();  // Prints: You called function: serve_drinks
    clean_tables();  // Prints: You called function: clean_tables
}
```

**Note**: We used `+` instead of `*` to require **one or more** matches.

## Optional Patterns: Handling Optional Arguments (`$()?`)

Sometimes, you want a macro to accept an argument that may or may not be there. The `$()?` syntax handles this, meaning "match the preceding pattern zero or one time."

### Example 3: A Flexible `create_order` Macro

Let's create a macro to initialize an `Order` struct. We want to be able to optionally specify a priority level.

```rust
struct Order {
    id: u32,
    customer: String,
    is_priority: bool,
}

macro_rules! create_order {
    // Matcher:
    // 1. `$id:expr, $customer:expr` : Required arguments.
    // 2. `$(, priority)?`          : An optional `, priority` token sequence.
    //    - The `?` means this group can appear zero or one time.
    ($id:expr, $customer:expr $(, priority)?) => {
        {
            // By default, is_priority is false.
            let mut priority_flag = false;

            // Transcriber:
            // If the optional pattern was matched...
            $(
                // ...this code gets generated.
                priority_flag = true;
            )?

            Order {
                id: $id,
                customer: $customer.to_string(),
                is_priority: priority_flag,
            }
        }
    };
}

fn main() {
    // Call without the optional argument
    let standard_order = create_order!(101, "Alice");
    assert_eq!(standard_order.is_priority, false);
    println!("Standard Order Priority: {}", standard_order.is_priority);

    // Call with the optional argument
    let priority_order = create_order!(102, "Bob", priority);
    assert_eq!(priority_order.is_priority, true);
    println!("Priority Order Priority: {}", priority_order.is_priority);
}
```

**How the `create_order!(102, "Bob", priority)` expansion works:**

1. **Matcher**: The `$(, priority)?` group successfully matches the `, priority` tokens in the input.
2. **Transcriber**: Because the optional group was matched, the code inside `$(...)?` is generated. The macro expands to:

```rust
{
    let mut priority_flag = false;
    priority_flag = true; // This line was generated
    Order {
        id: 102,
        customer: "Bob".to_string(),
        is_priority: priority_flag,
    }
}
```


## Combining Repetition and Optionality

You can nest these patterns to create incredibly powerful macros. For example, you could create a macro that defines a `struct` with a variable number of fields, where some fields might be optional.

## Best Practices and Common Pitfalls

1. **Wrap Transcribers in Braces**: Always wrap your transcriber in `{ ... }`. This ensures that the generated code has its own scope, preventing variable name collisions and making the macro behave more predictably like an expression.
2. **Use Separators**: When using repetition, specify a separator (like `,` or `;`) inside the matcher: `$( ... ),*`. This makes your macro syntax cleaner and easier to use.
3. **Start Simple**: Macros can get complex quickly. Start with a simple pattern and build up from there. Test each change as you go.
4. **Use `stringify!` for Debugging**: If your macro isn't working, `stringify!` can be a lifesaver. It converts a token into a string literal, allowing you to print out what your macro is actually receiving.

```rust
macro_rules! debug_tokens {
    ($($tokens:tt)*) => {
        println!("{}", stringify!($($tokens)*));
    };
}
debug_tokens!(hello 123 ::world); // Prints: "hello 123 :: world"
```

5. **Know When Not to Use a Macro**: If you can solve a problem with a function or a generic type, that is usually the better choice. Macros should be reserved for tasks that are impossible with regular Rust syntax, such as handling a variable number of arguments or generating code structures.

By mastering repetition and optional patterns, you can unlock the full potential of Rust's declarative macros, allowing you to write more expressive, less repetitive, and more powerful code.
<span style="display:none">[^1][^2][^6][^7][^8]</span>

1. https://dev.to/rogertorres/first-steps-with-rust-declarative-macros-1f8m
2. https://doc.rust-lang.org/book/ch19-06-macros.html
3. https://suchprogramming.com/rusty-macros/
4. https://www.youtube.com/watch?v=KsJHlqULpO4
5. https://earthly.dev/blog/rust-macros/
6. https://www.freecodecamp.org/news/procedural-macros-in-rust/
7. https://www.reddit.com/r/rust/comments/18afxup/declarative_rust_macros_explanation/
8. https://betterprogramming.pub/rust-generic-trait-declarative-and-procedural-macros-6ff9cd8016de
