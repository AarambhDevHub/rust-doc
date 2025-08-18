+++
title = "Examples in Documentation"
description = "Best practices for including clear, working code examples in your Rust documentation."
date = "2025-08-13"
weight = 4
+++

# Examples in Documentation: Comprehensive Documentation for Beginners

Including clear, working code examples in your Rust documentation is like **adding a "chef's demonstration" video to a restaurant's online menu** – it goes beyond a simple description and shows your customers (developers) exactly how to prepare and enjoy the dish. Just as a cooking demonstration makes a recipe more approachable and guarantees a successful result, well-crafted documentation examples make your code easier to understand, use correctly, and integrate into other projects. This practice is a cornerstone of the Rust ecosystem, powered by `rustdoc`, which not only formats your examples but also tests them.[^2]

## The Professional Restaurant Menu Demonstration Analogy 🍽️

### Imagine You're Creating an Interactive Menu with Cooking Demonstrations

**The Problem with No Examples (A Menu with Only Text):**

```rust
// ❌ No examples - like a recipe with only a list of ingredients and complex instructions
/// Calculates the final price of an order, including discounts and taxes.
/// This function first sums the prices of all items, then applies a percentage-based
/// discount, and finally adds a flat tax rate on top of the discounted total.
fn calculate_final_price(items: &[f64], discount: f64, tax_rate: f64) -> f64 {
    // How do I actually use this? What should `discount` look like (0.1 or 10)?
    // The user has to guess and might make mistakes.
    0.0 // Placeholder
}
```

**The Power of Documentation Examples (A Menu with a Step-by-Step Video):**

```rust
// ✅ With examples - a clear, step-by-step demonstration that anyone can follow
/// Calculates the final price of an order, including discounts and taxes.
///
/// # Examples
///
/// ```
/// let items = vec![10.00, 25.50]; // A couple of items
/// let discount = 0.10; // 10% discount
/// let tax_rate = 0.05; // 5% tax
///
/// let final_price = calculate_final_price(&items, discount, tax_rate);
///
/// // (10.00 + 25.50) = 35.50
/// // 35.50 * (1.0 - 0.10) = 31.95
/// // 31.95 * (1.0 + 0.05) = 33.5475
/// assert!((final_price - 33.5475).abs() < 1e-9);
/// ```
fn calculate_final_price(items: &[f64], discount: f64, tax_rate: f64) -> f64 {
    // ... implementation ...
    0.0 // Placeholder
}
```

**Why Examples Are a Superpower in Rust Documentation:**

- 📖 **Clarity \& Usability**: Examples are often the fastest way for a developer to understand how to use your code.
- ✅ **Guaranteed Correctness**: `rustdoc` compiles and runs your examples as tests (`cargo test`), ensuring they are always correct and up-to-date.[^7]
- 🚀 **Faster Onboarding**: Users can copy-paste your examples to get started quickly, reducing friction.
- 🤝 **Living Documentation**: Because they are tested, your examples evolve with your code, preventing documentation rot.
- 🎯 **Demonstrates Intent**: Examples clearly show the intended use case for your functions and types.


## How to Write Examples in Rust Documentation

Examples are written inside standard documentation comments (`///`) using **Markdown code blocks**.

```rust
/// This is a documentation comment.
///
/// # Examples
///
/// Here is a code block containing our example:
/// ```
/// // Rust code goes here
/// let result = my_function(2, 2);
/// assert_eq!(result, 4);
/// ```
```


### The `# Examples` Section

By convention, all examples live under a level-1 Markdown heading named `# Examples`. This standard structure makes your documentation predictable and easy to navigate.

### The Core Feature: Doc Tests

When you run `cargo test`, the test harness will automatically:

1. Extract every code block from your documentation.
2. Wrap it in a `fn main() { ... }` block (if one isn't present).
3. Add `extern crate your_crate_name;` at the top.
4. Compile and run the code.

The test passes if the code compiles and runs without panicking. This is why `assert!` and `assert_eq!` are so common in documentation examples—they ensure the code produces the correct result.[^7]

## Step-by-Step Guide: Adding Examples to a Restaurant Library

Let's expand on a simple restaurant library, focusing on adding high-quality, testable examples.

**Project Setup:**

1. Create a new Rust library project: `cargo new restaurant_menu --lib`
2. Open `src/lib.rs`.

### Step 1: An Example for a Simple Function

Let's start with a simple utility function and add an example.

```rust
// src/lib.rs

//! # Restaurant Menu Library
//!
//! A simple library for managing restaurant menus and orders.

/// Converts a price in cents to a formatted dollar string.
///
/// # Examples
///
/// ```
/// use restaurant_menu::format_price;
///
/// let formatted = format_price(1599);
/// assert_eq!(formatted, "$15.99");
/// ```
pub fn format_price(cents: u32) -> String {
    format!("${:.2}", cents as f64 / 100.0)
}
```

**Run the doc test:**

```bash
cargo test
```

**Expected Output:**
The test will pass because the code in the example block is correct.

```
running 1 test
test src/lib.rs - format_price (line 12) ... ok

test result: ok. 1 passed; 0 failed; ...
```


### Step 2: An Example for a Struct and its Methods

Now, let's create a `MenuItem` struct and show how to create and use it.

```rust
// src/lib.rs (continued)

/// Represents an item on the menu.
///
/// # Examples
///
/// ```
/// use restaurant_menu::MenuItem;
///
/// let pizza = MenuItem::new("Margherita Pizza", 1250);
///
/// assert_eq!(pizza.name, "Margherita Pizza");
/// assert_eq!(pizza.get_price_cents(), 1250);
/// ```
#[derive(Debug, Clone, PartialEq)]
pub struct MenuItem {
    pub name: String,
    price_cents: u32,
}

impl MenuItem {
    /// Creates a new menu item.
    ///
    /// The price should be provided in cents to avoid floating-point inaccuracies.
    pub fn new(name: &str, price_cents: u32) -> Self {
        MenuItem {
            name: name.to_string(),
            price_cents,
        }
    }

    /// Returns the price of the item in cents.
    pub fn get_price_cents(&self) -> u32 {
        self.price_cents
    }
}
```

- The example for the `MenuItem` struct demonstrates its entire lifecycle: creation with `::new()` and usage of its methods.
- This single example effectively documents the primary use case for the whole struct.


### Step 3: Hiding Setup Code in Examples

Sometimes, you need setup code for an example to work, but you don't want to clutter the documentation with it. You can hide lines from the final HTML output by starting them with `#`. These lines are still compiled and run by `cargo test`.[^5]

**Example:**

Let's create an `Order` struct that requires a complex setup.

```rust
// src/lib.rs (continued)

/// Represents a customer's order.
pub struct Order {
    items: Vec<MenuItem>,
}

impl Order {
    /// Creates a new, empty order.
    pub fn new() -> Self {
        Order { items: Vec::new() }
    }

    /// Adds a `MenuItem` to the order.
    pub fn add_item(&mut self, item: MenuItem) {
        self.items.push(item);
    }

    /// Calculates the total price of the order in cents.
    ///
    /// # Examples
    ///
    /// ```
    /// # // This setup code is hidden from the final documentation,
    /// # // but is necessary for the example to run.
    /// # use restaurant_menu::{Order, MenuItem};
    /// #
    /// // Create a new order and add some items.
    /// let mut order = Order::new();
    /// order.add_item(MenuItem::new("Salad", 800));
    /// order.add_item(MenuItem::new("Bread", 250));
    ///
    /// // The total should be the sum of the item prices.
    /// assert_eq!(order.total_cents(), 1050);
    /// ```
    pub fn total_cents(&self) -> u32 {
        self.items.iter().map(|item| item.get_price_cents()).sum()
    }
}
```

In the generated HTML, the user will only see:

```rust
// Create a new order and add some items.
let mut order = Order::new();
order.add_item(MenuItem::new("Salad", 800));
order.add_item(MenuItem::new("Bread", 250));

// The total should be the sum of the item prices.
assert_eq!(order.total_cents(), 1050);
```

This keeps the example focused on the `total_cents` method while ensuring the test is complete and runnable.

### Step 4: Examples for Code that Should Not Compile

To show how *not* to use your API, you can use a ````ignore` code block. This code will be displayed but not tested.

```
/// # Examples of Incorrect Usage
///
/// The following code will not compile because `price_cents` is a private field.
///
/// ```ignore
/// let item = MenuItem::new("Fries", 400);
/// // This fails because price_cents is not public!
/// item.price_cents = 500;
/// ```
```


### Step 5: Examples for Fallible Functions

For functions that return a `Result`, it's good practice to show both the `Ok` and `Err` cases. You can use the `?` operator inside a helper function to keep the example clean [^5].

```
// src/lib.rs (continued)

/// Parses a menu item from a "name:price" string.
///
/// # Errors
///
/// Returns an error if the string is not formatted correctly or if the price is not a valid number.
///
/// # Examples
///
/// ```
/// # // Use a helper function to handle the Result with `?`
/// # fn main() -> Result<(), String> {
/// use restaurant_menu::{MenuItem, parse_menu_item};
///
/// let item = parse_menu_item("Pizza:1599")?;
/// assert_eq!(item, MenuItem::new("Pizza", 1599));
///
/// // Example of a failure case
/// assert!(parse_menu_item("Salad:invalid").is_err());
/// #
/// # Ok(())
/// # }
/// # main().unwrap();
/// ```
pub fn parse_menu_item(s: &str) -> Result<MenuItem, String> {
    let parts: Vec<&str> = s.split(':').collect();
    if parts.len() != 2 {
        return Err("Invalid format. Expected 'name:price'".to_string());
    }
    let name = parts;
    let price_cents = parts[^1].parse::<u32>().map_err(|_| "Invalid price".to_string())?;
    Ok(MenuItem::new(name, price_cents))
}
```

- The `# fn main() -> Result<...> { ... }` wrapper is a common pattern for testing fallible functions [^5].
- `# main().unwrap()` ensures the test panics if the `main` function returns an `Err`, making the doc test fail as expected.


## Best Practices for Writing Documentation Examples

1. **Be Clear and Simple**: Your example should be as simple as possible while still demonstrating the feature effectively. Avoid complex logic that distracts from the main point.
2. **Cover the Main Use Case**: The primary example should show the most common way to use the item.
3. **Include Assertions**: Always use `assert_eq!` or `assert!` to verify that your example code produces the correct result. This is what makes it a true test.
4. **Keep Examples Self-Contained**: Use hidden lines (`#`) to include all necessary `use` statements and setup code so the example can be copied and run directly.
5. **Document Edge Cases**: If there are important edge cases (e.g., empty inputs, maximum values), consider adding a separate, small example for them.
6. **Use `?`, Not `unwrap()`**: In examples for fallible functions, use the `?` operator inside a helper function instead of `.unwrap()`. This demonstrates best practices to your users [^277].
7. **Be Consistent**: Use a consistent style for all examples across your crate. `rustfmt` will help format the code, but you should be consistent with naming and structure.

## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Menu Demonstration System**

Remember our comprehensive professional restaurant menu demonstration analogy:

- 📖 **`# Examples` Section**: The "Cooking Demonstration" part of your menu.
- ```
```

- `assert_eq!`: The "taste test" at the end of the video, proving the dish came out perfectly.
- `#` **(Hidden Lines)**: The "mise en place" – all the pre-chopped ingredients that are ready to go but not shown in detail during the main demonstration.
- `cargo test`: The "quality control" team that watches every demonstration video to make sure it's accurate and repeatable.


### **The Documentation Example Workflow**

1. **Add an `# Examples` section** to your documentation comment.
2. **Write a Rust code block** using ` ```rust ... ```
3. **Create a simple, self-contained example** that demonstrates the most common use case.
4. **Use `assert!` or `assert_eq!`** to verify the outcome of your example.
5. **Hide necessary setup code** (like `use` statements) with `#` to keep the public-facing example clean.
6. **Run `cargo test`** to automatically test your example.
7. **Run `cargo doc --open`** to see your beautiful, interactive documentation.

### **Why Testable Examples Are a Game-Changer**

- **Trust**: Developers trust documentation that is proven to be correct.
- **Maintenance**: It forces you to keep your examples up-to-date as your code changes.
- **Quality**: It doubles as a mini-integration test for your public API, often catching bugs in how components interact.


### **The Professional Advantage**

**Mastering documentation examples in Rust is like having a complete professional training and marketing system** for your library, ensuring users have a seamless and successful experience:

- 🚀 **Effortless Onboarding**: Provides users with a clear, copy-pasteable starting point.
- 🛡️ **Guaranteed Correctness**: Builds trust by proving that your documentation is not just talk—it actually works.
- 🤝 **Improved API Design**: Writing examples forces you to use your own API, often revealing design flaws or usability issues.
- 📖 **Living Documentation**: Creates documentation that is automatically kept in sync with your code.
- 🏆 **Professional Polish**: Signals a high-quality, well-maintained project that developers will want to use.

**Understanding how to write excellent, testable examples transforms you from a programmer who just writes code to an engineer** who builds a complete, professional, and user-friendly product. Just as a master chef provides clear, foolproof demonstrations to empower home cooks, a skilled Rust programmer provides clear, testable examples to empower other developers to build amazing things with their code.

1. https://doc.rust-lang.org/rust-by-example/meta/doc.html
2. https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
3. https://www.tangramvision.com/blog/making-great-docs-with-rustdoc
4. https://dev.to/gritmax/writing-rust-documentation-5hn5
5. https://rust-lang.github.io/api-guidelines/documentation.html
6. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/documentation.html
7. https://apidog.com/blog/rustdoc/
8. https://www.reddit.com/r/rust/comments/7eohmt/whats_the_best_practice_for_documenting_a_rust/
9. https://users.rust-lang.org/t/best-practices-to-write-rust-code/110040
10. https://blog.guillaume-gomez.fr/articles/2020-03-12+Guide+on+how+to+write+documentation+for+a+Rust+crate
