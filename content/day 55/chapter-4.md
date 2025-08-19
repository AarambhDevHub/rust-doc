---
title: "Macro Workshop - Real-World Macro Examples"
description: "Analyze real-world examples of macros in popular Rust libraries and learn practical applications."
date: 2025-10-09
weight: 4
---

# Macro Workshop: Analyzing Real-World Examples in Popular Rust Libraries

To truly understand the power and purpose of macros in Rust, it's best to see how they are used in the wild by popular, battle-tested libraries. This workshop will dissect iconic macros from `log`, `serde`, `tokio`, and `sqlx`, explaining the problem they solve, how they work, and why a macro was the only tool for the job.

## The Professional Kitchen Analogy: Specialized Tools for Expert Chefs 🍽️

As a chef's skills grow, they move beyond standard recipes and begin creating their own specialized tools and techniques.

- A **declarative macro** is like a chef creating a reusable template for a common task, like "mince and season."
- A **procedural macro** is like a chef building a custom kitchen gadget that can automatically debone a fish or perfectly portion pasta, performing complex tasks that would be tedious and error-prone to do manually.

Let's look at the custom tools built by some of the best "chefs" in the Rust ecosystem.

***

### 1. `log::info!` — The Declarative Macro for a Simple DSL

The `log` crate provides a standard logging facade for Rust applications. Its macros (`info!`, `warn!`, `debug!`, etc.) are a perfect example of declarative macros used to create a simple, effective Domain-Specific Language (DSL).

**The Problem**: You need a consistent way to write log messages that supports different severity levels and can format arguments, just like `println!`.

**The "Before" Picture (Without Macros)**
If you were to implement this with functions, your code would be verbose and clunky:

```rust
// ❌ Tedious, manual logging
fn log_something(user_id: u32) {
    // Manually check if the "INFO" level is even enabled
    if log::log_enabled!(log::Level::Info) {
        println!("[INFO] User {} has logged in", user_id);
    }
}
```

**The "After" Picture (Using the `log` Macro)**
The `log` macros provide a clean and intuitive interface:

```rust
use log::info;

// ✅ Clean, expressive, and efficient
fn log_something(user_id: u32) {
    info!("User {} has logged in", user_id);
}
```

**How It Works (Under the Hood)**
The `info!` macro is a declarative macro defined with `macro_rules!`. It's a simple wrapper around the more general `log!` macro. Its definition looks something like this (simplified):

```rust
macro_rules! info {
    // Match a format string and a list of arguments, just like println!
    ($($arg:tt)+) => {
        // Expand to a call to the core log! macro with the "Info" level
        log!(Level::Info, $($arg)+)
    }
}
```

The macro handles the check to see if the log level is enabled, so if you have logging disabled in your release build, the call to `info!` compiles down to absolutely nothing—no runtime cost.

**Why It *Must* Be a Macro:**

1. **Variable Arguments**: A function cannot accept a variable number of arguments in the way `info!("User: {}, Action: {}", id, action)` does.
2. **Compile-Time Disabling**: The macro can check the log level at compile time and generate no code if logging for that level is disabled, which is more performant than a runtime `if` check.
3. **Capturing Context**: The underlying `log!` macro captures the file and line number where it was called using `file!` and `line!`, something a function cannot do.

***

### 2. `serde::Serialize` — The Custom Derive Macro for Boilerplate Removal

`serde` is the gold standard for serialization and deserialization in Rust. Its `#[derive(Serialize, Deserialize)]` is arguably the most famous and impactful use of procedural macros in the ecosystem.

**The Problem**: Manually writing the code to convert a `struct` to and from formats like JSON, YAML, or Bincode is incredibly repetitive and error-prone.

**The "Before" Picture (Without Macros)**
For a simple `User` struct, you would have to write all of this by hand:

```rust
// ❌ Writing manual serialization is a nightmare
use serde::{ser::{Serialize, Serializer, SerializeStruct}};

struct User {
    id: u32,
    username: String,
}

impl Serialize for User {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let mut state = serializer.serialize_struct("User", 2)?;
        state.serialize_field("id", &self.id)?;
        state.serialize_field("username", &self.username)?;
        state.end()
    }
}
```

**The "After" Picture (Using the `serde` Macro)**
The derive macro reduces all of that boilerplate to a single line:

```rust
use serde::Serialize;

// ✅ The macro generates the entire `impl` block for you!
#[derive(Serialize)]
struct User {
    id: u32,
    username: String,
}
```

**How It Works (Under the Hood)**

1. When the compiler sees `#[derive(Serialize)]`, it invokes the `Serialize` procedural macro.
2. The macro receives the code for the `User` struct as a `TokenStream`.
3. It uses the `syn` crate to parse this token stream into a structured representation, identifying the struct's name (`User`) and its fields (`id`, `username`) and their types.
4. It then uses the `quote` crate to generate the entire `impl Serialize for User { ... }` block, just like the manual version above.
5. Finally, it hands this newly generated code back to the compiler, which seamlessly integrates it into the rest of the program.

**Why It *Must* Be a Macro:**
A function or generic has no ability to **introspect** or **reflect** on the fields of a struct at compile time. Only a procedural macro can take a type definition as input and generate a new `impl` block tailored to that specific type.

***

### 3. `tokio::main` — The Attribute Macro for Ergonomics

`tokio` is the leading asynchronous runtime in Rust. Its `#[tokio::main]` attribute macro is a brilliant example of how to simplify complex setup boilerplate into a clean, intuitive annotation.

**The Problem**: To run an `async` function, you need to create and configure an async runtime, and then tell it to run your main future to completion. This involves several lines of setup code.

**The "Before" Picture (Without Macros)**
Manually setting up the `tokio` runtime looks like this:

```rust
// ❌ The manual setup for an async main function
fn main() {
    let rt = tokio::runtime::Runtime::new().unwrap();
    rt.block_on(async {
        println!("Hello from async world!");
    })
}
```

**The "After" Picture (Using the `tokio` Macro)**
The attribute macro makes the process feel native to the language:

```rust
// ✅ Clean, intuitive, and hides the complexity
#[tokio::main]
async fn main() {
    println!("Hello from async world!");
}
```

**How It Works (Under the Hood)**
The `#[tokio::main]` macro is an attribute macro. It takes the entire `async fn main()` function definition as its input. It then **rewrites** the function. The code it generates looks roughly like this:

```rust
// The macro transforms your async main into a regular main...
fn main() {
    // ...and puts the boilerplate inside it!
    tokio::runtime::Builder::new_multi_thread()
        .enable_all()
        .build()
        .unwrap()
        .block_on(async {
            // Your original async code goes here
            println!("Hello from async world!");
        })
}
```

**Why It *Must* Be a Macro:**
A function cannot take another function's definition and rewrite it. The ability to consume a function, transform it, and replace it is a unique power of attribute macros. It allows `tokio` to provide a user experience that feels like `async main` is a built-in Rust feature.

***

### 4. `sqlx::query!` — The Function-Like Macro for Compile-Time Safety

`sqlx` is a modern, async-native SQL toolkit for Rust. Its `query!` macro (and variants) are a groundbreaking use of procedural macros to bring compile-time safety to database interactions.

**The Problem**: Traditional database queries use raw strings. A typo in a SQL query (like `SLECT` instead of `SELECT` or a wrong column name) is only caught when you run the code, often causing a runtime panic.

**The "Before" Picture (Without Macros, or with other libraries)**

```rust
// ❌ This typo in "username" would only be found at runtime!
let query_string = "SELECT user_id, user_name FROM users WHERE id = ?";
// ... code to execute the query ...
```

**The "After" Picture (Using the `sqlx` Macro)**
The `sqlx::query!` macro checks your SQL at **compile time**.

```rust
use sqlx::Row;

async fn get_user(pool: &sqlx::PgPool, id: i32) -> Result<(), sqlx::Error> {
    // ✅ If you misspell a column, this code will FAIL TO COMPILE!
    let user = sqlx::query!("SELECT user_id, username FROM users WHERE user_id = $1", id)
        .fetch_one(pool)
        .await?;

    // The macro also infers the return type for user.username as String
    println!("Found user: {}", user.username);
    Ok(())
}
```

**How It Works (Under the Hood)**
This is the most "magical" macro of the group:

1. The `sqlx::query!` macro is a function-like procedural macro.
2. To use it, you must first set a `DATABASE_URL` environment variable.
3. During compilation, the macro **connects to your live database**, parses your SQL query string, and sends it to the database to be validated.
4. It checks for SQL syntax errors, verifies that tables and columns exist, and even checks that the types of your placeholders (`$1`) and return values match your Rust code.
5. If anything is wrong, it generates a detailed **compile-time error**. If everything is correct, it generates highly optimized runtime code to execute the query.

**Why It *Must* Be a Macro:**
This is impossible with any other feature. A function cannot execute code (like connecting to a database) during compilation. This requires a procedural macro's ability to run arbitrary code and interact with the outside world at build time, bringing a level of safety to database work that was previously unheard of in most languages.

1. https://www.reddit.com/r/rust/comments/bfwmqy/6_useful_rust_macros_that_you_might_not_have_seen/
2. https://github.com/rust-unofficial/awesome-rust
3. https://earthly.dev/blog/rust-macros/
4. https://doc.rust-lang.org/book/ch19-06-macros.html
5. https://technorely.com/insights/writing-and-optimizing-custom-derives-with-rusts-proc-macro-for-code-generation
6. https://dev.to/aniket_botre/day-31-macro-mania-supercharge-your-rust-code-479p
7. https://www.freecodecamp.org/news/procedural-macros-in-rust/
8. https://www.oreilly.com/library/view/write-powerful-rust/9781633437494/
