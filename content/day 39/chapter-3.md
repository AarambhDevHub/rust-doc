+++
title = "Documentation Tests"
description = "Learn how to embed and run examples in your documentation as tests to ensure accuracy."
date = "2025-08-13"
weight = 3
+++

# Documentation Tests in Rust: Comprehensive Documentation for Beginners

Understanding documentation tests in Rust is like learning that **the serving suggestions and pictures on a professional restaurant menu are not just for show—they are actual, verifiable dishes that the kitchen is tested on every day**. This ensures that what the customer sees on the menu (the documentation) is exactly what they get on their plate (the code's actual behavior). Just as a restaurant guarantees its menu is accurate by regularly preparing the dishes shown, Rust guarantees its documentation is accurate by compiling and running the code examples within it as part of its standard testing process.

## The Verifiable Restaurant Menu Analogy 🍽️

### Imagine You're Managing a Restaurant with a "What You See Is What You Get" Guarantee

**The Problem with Unverified Documentation (A Menu with Inaccurate Pictures):**

```rust
// ❌ Unverified example - like a menu photo that doesn't match the dish
/// Calculates the total cost of an order.
///
/// # Examples
///
/// ```rust
/// // This example might be outdated or simply wrong!
/// let total = calculate_order_total(10.0, 20.0);
/// assert_eq!(total, 30.0); // Maybe the logic changed to include tax?
/// ```
fn calculate_order_total(a: f64, b: f64) -> f64 {
    // Oops, the implementation now includes a 10% tax
    (a + b) * 1.10
}
// This leads to confused and unhappy customers (developers).
```

**The Power of Documentation Tests (A Menu That Is Tested Daily):**

```rust
// ✅ Verifiable example - the menu photo is generated from a real dish made today
/// Calculates the total cost of an order, including a 10% tax.
///
/// # Examples
///
/// ```
/// use my_restaurant::calculate_order_total;
///
/// let total = calculate_order_total(10.0, 20.0);
/// // This example is automatically tested, so it must be correct!
/// assert_eq!(total, 33.0);
/// ```
pub fn calculate_order_total(a: f64, b: f64) -> f64 {
    (a + b) * 1.10
}
// When you run `cargo test`, Rust ensures this example is correct.
```

**Why Documentation Tests Are a Game-Changer in Rust:**

- 🛡️ **Guaranteed Correctness**: Ensures your documentation examples are never outdated or incorrect.[^1][^2]
- 📖 **Living Documentation**: Your documentation evolves with your code, preventing "doc rot".[^1]
- 🚀 **Excellent User Experience**: Provides users with reliable, copy-pasteable examples that are guaranteed to work.[^2]
- ✅ **Implicit Testing**: Serves as a primary way to test your public API, ensuring it's ergonomic and works as expected.[^2]
- 🔄 **Simplified Maintenance**: When you change a function's behavior, the failing doc test immediately reminds you to update the documentation.


## How to Write and Run Documentation Tests

Documentation tests (or "doctests") are simply Rust code blocks written inside your documentation comments. Rust's tooling automatically extracts, compiles, and runs them.[^1]

### Step 1: Writing a Basic Documentation Test

You write a code example within a documentation comment using triple backticks (`````). `rustdoc` will treat this as a Rust code block and test it [^2].

**Example:**

```
// In src/lib.rs

/// Adds two integers together.
///
/// # Examples
///
/// ```
/// use my_library::add; // Assuming this code is in a library named `my_library`
///
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

**How it works:**

* `rustdoc` sees the code block inside the `///` comments.
* It implicitly wraps this code in a `fn main() { ... }` block [^5].
* It adds an `extern crate <your_crate_name>;` at the top so you can use your library's functions [^3].
* It then compiles and runs this new temporary test program. If the program runs without panicking, the test passes.


### Step 2: Running Documentation Tests

You don't need a special command to run your doctests. They are run automatically with `cargo test`.

```
cargo test
```

The output will show your regular unit tests and your documentation tests separately.

**Example Output:**

```
running 0 unit tests

test result: ok. 0 passed; 0 failed; ...

running 1 test
test src/lib.rs - add (line 5) ... ok

test result: ok. 1 passed; 0 failed; ...
```

If you *only* want to run documentation tests, you can use the `--doc` flag [^3]:

```
cargo test --doc
```


## Advanced Documentation Test Features

`rustdoc` provides special annotations to control how your code examples are tested [^5].

### 1. Hiding Code from the Documentation Output

Sometimes, you need setup code for your example that isn't relevant to the user. You can hide these lines from the final HTML documentation by starting them with a `# ` (a hash and a space) [^5].

**Example:**

```
/// Creates a greeting for a given name.
///
/// # Examples
///
/// ```
/// # use my_library::create_greeting; // This line is hidden in the docs, but still runs in the test.
/// let greeting = create_greeting("Alice");
/// assert_eq!(greeting, "Hello, Alice!");
/// ```
pub fn create_greeting(name: &str) -> String {
    format!("Hello, {}!", name)
}
```

In the generated HTML, the user will only see the last two lines, making the example clean and focused. However, the `use` statement is still included when the test is compiled, allowing it to work correctly.

### 2. Testing for Panics (`should_panic`)

If your function is expected to panic under certain conditions, you can test this behavior by adding the `should_panic` attribute to your code block [^2].

**Example:**

```
/// Divides two numbers.
///
/// # Panics
///
/// This function panics if the second argument is zero.
///
/// # Examples
///
/// ```rust,should_panic
/// use my_library::div;
///
/// // This test passes because the function panics as expected.
/// div(10, 0);
/// ```
pub fn div(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division by zero!");
    }
    a / b
}
```

The test will pass only if the code inside panics. If it runs successfully, the test will fail.

### 3. Ignoring Code (`ignore`)

If you have an example that shouldn't be run as a test (e.g., it's just illustrative pseudocode or requires a special environment), you can use the `ignore` attribute [^5].

**Example:**

```
/// Connects to a network service.
///
/// # Examples
///
/// ```rust,ignore
/// // This example is not run as a test because it requires a live network connection.
/// let connection = connect_to_server("127.0.0.1:8080");
/// assert!(connection.is_ok());
/// ```
```


### 4. Compiling but Not Running (`no_run`)

This is useful for examples that demonstrate something that should compile but might run forever, or require user interaction [^5].

**Example:**

```
/// Starts an infinite event loop.
///
/// # Examples
///
/// ```rust,no_run
/// // This code is compiled to check for errors, but not executed.
/// loop {
///     println!("Processing events...");
/// }
/// ```
```


### 5. Testing Code That Should Fail to Compile (`compile_fail`)

This is an advanced feature used to demonstrate what happens when your API is used incorrectly, ensuring that certain misuses result in a compile-time error [^4].

**Example:**

```
/// This function only accepts types that implement the `Display` trait.
///
/// # Examples
///
/// This example will fail to compile because `MyStruct` does not implement `Display`.
///
/// ```rust,compile_fail
/// struct MyStruct; // Doesn't implement Display
///
/// print_thing(MyStruct);
/// ```
pub fn print_thing<T: std::fmt::Display>(thing: T) {
    println!("{}", thing);
}
```

This test passes only if the code inside *fails* to compile.

## Best Practices for Writing Documentation Tests

1. **Focus on the Public API**: Your doctests should demonstrate how a user would typically interact with your library's public functions, structs, and methods.
2. **Keep Examples Simple and Clear**: Each example should demonstrate one specific feature or use case. Avoid overly complex examples.
3. **Use `assert!` and `assert_eq!`**: Your examples should not just run; they should verify that the output is correct using assertions. This makes them true tests, not just demonstrations.
4. **Hide Unnecessary Setup Code**: Use `#` to hide `use` statements or other setup code that distracts from the core example.
5. **Document Expected Failures**: If your code is designed to `panic` or return an `Err`, write doctests that explicitly test for this behavior.
6. **Run `cargo test` Frequently**: Make it a habit to run your tests often. This ensures that your documentation and code are always in sync.

## Summary and Key Takeaways

### **Mental Model: The Complete Verifiable Restaurant Menu System**

Remember our comprehensive verifiable restaurant menu analogy:

- 📖 **Documentation Comments** = **The Restaurant Menu** - A guide for your users.
- ✅ **Documentation Tests** = **A "What You See Is What You Get" Guarantee** - Ensures the menu is always accurate.
- `cargo test` = **The Daily Kitchen Inspection** - The process that verifies every dish on the menu.
- `#` **(Hidden Code)** = **The "Chef's Prep"** - The necessary setup that customers don't need to see.
- `should_panic` = **Testing the "Extra Spicy" Dish** - Ensuring the dish delivers the expected (and warned-about) kick.


### **The Documentation Testing Workflow**

1. **Write Documentation**: Use `///` to document your public API items.
2. **Embed Code Examples**: Inside your documentation, add Rust code blocks using ````````.
3. **Add Assertions**: Use `assert_eq!`, `assert!`, etc., within your examples to verify correctness.
4. **Run Tests**: Execute `cargo test`. `rustdoc` will automatically find, compile, and run your examples.
5. **Use Attributes for Control**: Use annotations like `#`, `should_panic`, and `ignore` to fine-tune how your examples are tested.

### **Why Documentation Tests are a Core Feature of Rust**

- **Reliability**: They provide a strong guarantee that your documentation is not misleading [^1].
- **Usability**: They serve as practical, working examples that users can trust and learn from [^2].
- **Maintainability**: They force you to keep your documentation in sync with your code, preventing "doc rot" [^1].


### **The Professional Advantage**

**Mastering documentation tests in Rust is like having a complete quality assurance system built directly into your communication process**:

- 🛡️ **Trust and Confidence**: Users of your library can trust your documentation completely.
- 🚀 **Faster Onboarding**: New developers can learn your API quickly from reliable, working examples.
- 🔄 **Living Documentation**: Your documentation becomes a dynamic, verifiable asset rather than a static, outdated document.
- 🏆 **Professional Polish**: It signals a high level of quality and care in your software engineering practice.

**Understanding documentation tests transforms you from a programmer who just writes documentation to an engineer** who builds a complete, verifiable, and trustworthy user experience. Just as a master chef stands behind every item on their menu with a guarantee of quality, a skilled Rust programmer uses documentation tests to stand behind their code with a guarantee of correctness and usability.

1. https://doc.rust-lang.org/rustdoc/documentation-tests.html
2. https://doc.rust-lang.org/rust-by-example/testing/doc_testing.html
3. https://doc.rust-lang.org/rust-by-example/meta/doc.html
4. http://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/rustdoc/documentation-tests.html
5. https://ebarnard.github.io/2019-06-03-rust-smaller-trait-implementers-docs/rustdoc/documentation-tests.html
6. https://www.jetbrains.com/help/rust/rust-doctest-support.html
7. https://stackoverflow.com/questions/66804618/run-doc-test-in-examples-folder-with-cargo
8. https://www.reddit.com/r/rust/comments/i8hewj/how_should_i_generate_documentation_for_tests/
9. https://doc.rust-lang.org/rust-by-example/testing.html
10. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/rust-by-example/testing/doc_testing.html
