---
title: "Macro Workshop - DSL Creation with Macros"
description: "Explore how to design Domain Specific Languages (DSLs) using Rust macros for expressive and readable code."
date: 2025-10-09
weight: 2
---

# Macro Workshop: Designing Domain-Specific Languages (DSLs) in Rust

Understanding how to build a Domain-Specific Language (DSL) with Rust macros is like learning to **invent your own custom shorthand for writing recipes**. Instead of writing every instruction in long, formal English, you create a compact, specialized language for yourself. `salt(1tsp)` is faster to write and clearer than "add one teaspoon of salt." A DSL in Rust does the same thing: it uses macros to create a mini-language tailored to a specific problem, making your code more expressive, readable, and less prone to errors.

## The Custom Recipe Shorthand Analogy ✍️

### Imagine You're a Chef Designing a Faster Way to Write Recipes

- **A Standard Function Call**: Writing out a full, formal instruction: `add_ingredient("salt", 1.0, "teaspoon")`. This is clear but verbose.
- **A DSL**: Inventing your own shorthand: `ingredient!(salt, 1, tsp)`. This is concise and, within the context of your kitchen, perfectly understandable.

***

### Why Create a DSL?

1. **Expressiveness and Readability**: A well-designed DSL makes the code look more like the problem it's solving. Code for building HTML can look like HTML; code for defining a state machine can look like a state diagram.[^1]
2. **Reduced Boilerplate**: DSLs abstract away repetitive setup and teardown code, allowing you to focus on the core logic.
3. **Enforcing Structure**: A DSL can guide users into providing information in a specific, required order, reducing the chance of logical errors.
4. **Type Safety**: Because the DSL is built with Rust macros, it expands into regular, type-checked Rust code. You get the benefit of a custom language without sacrificing Rust's safety guarantees.[^2]

## How to Build a DSL: The `macro_rules!` Toolkit

Declarative macros (`macro_rules!`) are the primary tool for building "internal" DSLs—languages that live inside your Rust code. They work by matching patterns of Rust syntax and transforming them into other Rust code.[^3]

Let's build a simple DSL for creating HTML. Our goal is to turn this Rust macro call:

```rust
html! {
    div(class="container") {
        h1 { "Hello, DSL!" }
        p { "This is generated from our custom Rust DSL." }
    }
}
```

...into this HTML string:

```html
<div class="container"><h1>Hello, DSL!</h1><p>This is generated from our custom Rust DSL.</p></div>
```


### Step 1: The Basic Building Block - A Single Tag

Let's start with a macro that can create a single, empty HTML tag.

```rust
macro_rules! html {
    // Matcher: Matches a tag name like `div` or `p`
    ($tag:ident) => {
        // Transcriber: Generates the HTML string
        format!("<{}></{}>", stringify!($tag), stringify!($tag))
    };
}

fn main() {
    let my_div = html!(div);
    assert_eq!(my_div, "<div></div>");
    println!("{}", my_div);
}
```

- `$tag:ident`: This captures a single **identifier** (like `div`) and names it `$tag`.
- `stringify!($tag)`: This is a crucial built-in macro that converts the identifier `div` into a string literal `"div"`.


### Step 2: Adding Content to the Tag

Now let's add the ability to put text inside the tag.

```rust
macro_rules! html {
    ($tag:ident) => { ... }; // Keep the old rule as a fallback

    // New Matcher: Matches a tag name followed by a block containing an expression
    ($tag:ident { $content:expr }) => {
        format!("<{}>{}</{}>", stringify!($tag), $content, stringify!($tag))
    };
}

fn main() {
    let my_p = html!(p { "Hello, World!" });
    assert_eq!(my_p, "<p>Hello, World!</p>");
    println!("{}", my_p);
}
```

- `$content:expr`: This captures any valid Rust **expression** inside the curly braces `{}`.


### Step 3: Handling Nested Elements (The "Language" Emerges)

This is where the magic happens. We want our DSL to handle nested `html!` calls. To do this, we need to capture a "token tree" (`tt`) and recursively process it. A token tree is any block of code surrounded by `()`, `[]`, or `{}`.

```rust
macro_rules! html {
    ($tag:ident) => { ... };
    ($tag:ident { $content:expr }) => { ... };

    // New Matcher: Matches a tag with a block of token trees
    // The repetition `$()*` means "match zero or more of this pattern"
    ($tag:ident { $($inner:tt)* }) => {
        // The transcriber recursively calls the `html!` macro on the inner content
        format!("<{}>{}</{}>", stringify!($tag), html!($($inner)*), stringify!($tag))
    };

    // Base case for recursion: An empty macro call produces an empty string
    () => { String::new() };

    // Recursive step: Process one html element and recurse on the rest
    ($tag:ident { $($content:tt)* } $($rest:tt)*) => {
        format!("{}{}", html!($tag { $($content)* }), html!($($rest)*))
    };
}

fn main() {
    let nested_html = html! {
        div {
            h1 { "Title" }
            p { "Paragraph" }
        }
    };
    assert_eq!(nested_html, "<div><h1>Title</h1><p>Paragraph</p></div>");
    println!("{}", nested_html);
}
```

This is a big leap, so let's break down the new rules:

1. `($tag:ident { $($inner:tt)* })`: This is the main "container" rule. It captures a tag and its entire content block as token trees. It then recursively calls `html!` on that inner content.
2. `()`: This is the base case. When the recursion reaches the end of the content, it hits an empty `html!()` call, which produces an empty string.
3. `($tag:ident { ... } $($rest:tt)*)`: This is the crucial "sibling" rule. It matches one full element (`div { ... }`) and then captures everything else in `$rest`. It generates the HTML for the first element and then concatenates it with a recursive call to `html!` on the rest of the elements. This is what allows `h1` and `p` to appear one after the other.

### Step 4: Adding Attributes

Let's add support for attributes like `class="container"`.

```rust
macro_rules! html {
    // --- Previous rules go here ---
    // (We'll simplify for clarity, combining rules)

    // New "entry point" rule for a tag
    ($tag:ident $(($($attr_name:ident=$attr_val:expr),*))? { $($content:tt)* } $($rest:tt)*) => {
        {
            let mut attrs = String::new();
            // This `if let` handles the optional attribute block
            if let Some(_) = Some(()) { // A bit of a trick to handle optionality
                $( // This block only runs if attributes were provided
                    $(
                        attrs.push_str(&format!(" {}=\\\"{}\\\"", stringify!($attr_name), $attr_val));
                    )*
                )?
            }

            format!("<{}{}>{}</{}>{}",
                stringify!($tag),
                attrs,
                html!($($content)*),
                stringify!($tag),
                html!($($rest)*)
            )
        }
    };

    // Base case and simple content rules
    () => { String::new() };
    ($e:expr) => { $e.to_string() };
}

fn main() {
    let final_html = html! {
        div(class="container", id="main") {
            h1 { "Welcome!" }
            p { "This is our complete DSL." }
        }
    };
    let expected = r#"<div class="container" id="main"><h1>Welcome!</h1><p>This is our complete DSL.</p></div>"#;
    assert_eq!(final_html, expected);
    println!("{}", final_html);
}
```

**How it works**:

- `$(($($attr_name:ident=$attr_val:expr),*))?`: This complex-looking pattern breaks down as:
    - `(...)` : The outer parentheses for the attributes.
    - `?`: Makes the entire attribute block optional.
    - `$(...),*`: Inside, it matches zero or more key-value pairs separated by commas.
- The transcriber then builds up the attribute string and injects it into the final `format!`.


## Best Practices for Designing a DSL

1. **Start Small and Iterate**: Begin with the simplest possible version of your DSL and gradually add features like attributes, nesting, and more complex rules.
2. **Use "Keywords" for Clarity**: Inside your macro, you can match on literal identifiers to make your DSL feel more like a real language.

```rust
macro_rules! builder {
    (create $name:ident with speed = $speed:expr) => {
        // ... logic ...
    };
}
// Usage: builder!(create spaceship with speed = 99);
```

3. **Provide Good Error Messages**: Macros can produce confusing error messages. If possible, add compile-time checks or use `compile_error!` to guide the user when they use the DSL incorrectly.
4. **Embrace Recursion**: Recursive macro calls are the standard pattern for handling nested or sequential structures in a DSL. Always make sure you have a base case to terminate the recursion!
5. **Debug with `trace_macros!`**: If you're struggling to understand how your macro is expanding, you can enable tracing by putting `trace_macros!(true);` before your macro call. The compiler will print out each step of the expansion.

By using declarative macros, you can build powerful, expressive, and safe Domain-Specific Languages that elevate your Rust code, making it a perfect tool for solving the unique problems of your specific domain.
<span style="display:none">[^4][^5][^6][^7][^8]</span>

1. https://codedamn.com/news/rust/implementing-domain-specific-languages-rust-practical-guide
2. https://dev.to/aniket_botre/day-31-macro-mania-supercharge-your-rust-code-479p
3. https://doc.rust-lang.org/rust-by-example/macros/dsl.html
4. https://www.reddit.com/r/rust/comments/14f5zzj/what_is_the_state_of_the_art_for_creating/
5. https://blog.logrocket.com/macros-in-rust-a-tutorial-with-examples/
6. https://earthly.dev/blog/rust-macros/
7. https://users.rust-lang.org/t/rust-tutorials-on-dsl-creation-and-proc-macros/79497
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/rust-by-example/macros/dsl.html
