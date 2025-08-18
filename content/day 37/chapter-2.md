+++
title = "Testing Public API"
description = "Understanding how to test the public API of your Rust crates effectively."
date = "2025-08-13"
weight = 2
+++

# Testing the Public API of a Rust Crate: Comprehensive Documentation for Beginners

Understanding how to test the public API of your Rust crate is like learning to **conduct a final quality inspection of a professional restaurant's service from a customer's perspective**. While you might have internal checks for how the kitchen operates (unit tests), the most important validation is ensuring that a customer who walks in the door gets the correct, high-quality experience they expect. Just as a restaurant manager would test the entire process from ordering to dining, testing a public API in Rust involves verifying that the crate's exposed functionality works correctly and seamlessly from an external user's point of view.

## The Professional Restaurant Quality Inspection Analogy 🍽️

### Imagine You're Inspecting Your Own Restaurant as a Customer

**The Problem with Only Internal Testing (Unit Tests):**

```rust
// ❌ Internal checks only - like tasting individual ingredients but never the final dish
// This is like a unit test:
fn test_chopped_vegetables() {
    // The vegetables are chopped correctly.
}
// But this doesn't guarantee the final dish is good, or even that the menu is understandable.
```

**The Power of Public API Testing (Integration Tests) - The Customer Experience:**

```rust
// ✅ Customer experience testing - like ordering a meal from the menu and evaluating the entire process
// This is like an integration test:
fn test_order_and_receive_pizza() {
    let mut restaurant = Restaurant::new();
    let menu = restaurant.get_menu();

    // Interact with the restaurant just like a customer would
    let order_result = restaurant.place_order(menu.pizza);

    assert!(order_result.is_ok());
    assert_eq!(order_result.unwrap().name, "Delicious Pizza");
}
// This validates the entire public-facing process, from menu to final dish.
```

**Why Testing the Public API Is Critical:**

- 🛡️ **User-Centric Validation**: It confirms that your crate works as advertised for its users.[^1]
- 🤝 **API Contract Guarantee**: It ensures that you don't accidentally break your public API, which is crucial for maintaining backward compatibility.[^2]
- 🚀 **Confidence in Releases**: A strong suite of API tests gives you confidence that new versions of your crate will work correctly for your users.
- 🎯 **Clear Separation of Concerns**: It encourages a clean separation between your internal implementation details and your stable public interface.[^3]


## How to Test a Public API in Rust: Integration Tests

In Rust, the primary way to test your crate's public API is through **integration tests**. Unlike unit tests, which are placed inside your `src` directory, integration tests live in a separate `tests` directory at the root of your project.[^4][^1]

**Project Structure for Integration Tests:**

```
my_restaurant_crate/
├── Cargo.toml
├── src/
│   └── lib.rs  // Your crate's main source code
└── tests/
    └── public_api_test.rs // Your integration test file
```


### The Key Difference: How Integration Tests Work

* **External Perspective**: Each file in the `tests` directory is compiled as a separate, temporary crate that depends on your main crate. This means the test code interacts with your library in the exact same way an external user would.[^1]
* **Public Access Only**: Integration tests can only access items that are marked `pub` in your crate's API. They cannot access private functions, structs, or modules.[^1]
* **Simulates Real Usage**: This setup forces you to write tests that reflect how your crate will actually be used, which is perfect for validating your public API.


## Practical Example: Building and Testing a Restaurant Order System

Let's create a simple library for managing restaurant orders and then write an integration test to validate its public API.

**Project Setup:**

1. Create a new Rust library: `cargo new restaurant_manager --lib`
2. Navigate into the project: `cd restaurant_manager`

### Step 1: Define the Public API in `src/lib.rs`

First, let's define a simple API for our `restaurant_manager` crate.

**File: `src/lib.rs`**

```rust
// A menu item that is part of our public API
#[derive(Debug, Clone, PartialEq)]
pub struct MenuItem {
    pub name: String,
    pub price: f64,
}

// An order, also part of the public API
#[derive(Debug, PartialEq)]
pub struct Order {
    items: Vec<MenuItem>,
    total: f64,
    is_confirmed: bool,
}

// Our public API implementation for Order
impl Order {
    /// Creates a new, empty order.
    pub fn new() -> Self {
        Order {
            items: Vec::new(),
            total: 0.0,
            is_confirmed: false,
        }
    }

    /// Adds a MenuItem to the order.
    pub fn add_item(&mut self, item: MenuItem) {
        if !self.is_confirmed {
            self.total += item.price;
            self.items.push(item);
        }
    }

    /// Finalizes the order, preventing further changes.
    pub fn confirm_order(&mut self) -> Result<(), String> {
        if self.items.is_empty() {
            Err("Cannot confirm an empty order.".to_string())
        } else {
            self.is_confirmed = true;
            Ok(())
        }
    }

    /// Returns the total price of the order.
    pub fn get_total(&self) -> f64 {
        self.total
    }

    // An internal, private function that is NOT part of the public API
    fn log_internal_state(&self) {
        println!("Internal order state: {:?}", self.items);
    }
}
```

* `MenuItem` and `Order` are public structs.
* The methods `new`, `add_item`, `confirm_order`, and `get_total` are the public API of our `Order` struct.
* `log_internal_state` is a private function and **cannot** be accessed by an integration test.


### Step 2: Write an Integration Test for the Public API

Now, let's create our integration test file.

1. Create a `tests` directory: `mkdir tests`
2. Create a test file inside it: `touch tests/order_api_test.rs`

**File: `tests/order_api_test.rs`**

```rust
// Import our crate just like an external user would
use restaurant_manager::{MenuItem, Order};

#[test]
fn test_full_order_workflow_from_public_api() {
    // 1. Create a new order using the public `new` function
    let mut my_order = Order::new();
    assert_eq!(my_order.get_total(), 0.0);

    // 2. Create some menu items
    let pizza = MenuItem { name: "Pizza".to_string(), price: 15.99 };
    let salad = MenuItem { name: "Salad".to_string(), price: 8.50 };

    // 3. Add items to the order using the public `add_item` method
    my_order.add_item(pizza.clone());
    my_order.add_item(salad.clone());

    // 4. Check the total using the public `get_total` method
    assert_eq!(my_order.get_total(), 24.49);

    // 5. Confirm the order using the public `confirm_order` method
    assert!(my_order.confirm_order().is_ok());

    // 6. Try to add another item after confirmation (should fail silently)
    let another_pizza = MenuItem { name: "Another Pizza".to_string(), price: 15.99 };
    my_order.add_item(another_pizza);

    // The total should NOT have changed
    assert_eq!(my_order.get_total(), 24.49);

    // This line would fail to compile because `items` is a private field:
    // assert_eq!(my_order.items.len(), 2);

    // This line would fail to compile because `log_internal_state` is a private method:
    // my_order.log_internal_state();
}

#[test]
fn test_cannot_confirm_empty_order() {
    let mut empty_order = Order::new();

    // Trying to confirm an empty order should return an error
    let result = empty_order.confirm_order();

    assert!(result.is_err());
    assert_eq!(result.unwrap_err(), "Cannot confirm an empty order.");
}
```


### Step 3: Run the Tests

Now, run the tests using the standard Cargo command:

```bash
cargo test
```

**Expected Output:**
Cargo will first compile your `restaurant_manager` library, then compile `order_api_test.rs` as a separate crate that links against your library, and finally run the tests.

```
   Compiling restaurant_manager v0.1.0 (...)
    Finished test [unoptimized + debuginfo] target(s) in ...s
     Running unittests src/lib.rs (target/debug/deps/restaurant_manager-...)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/order_api_test.rs (target/debug/deps/order_api_test-...)

running 2 tests
test test_cannot_confirm_empty_order ... ok
test test_full_order_workflow_from_public_api ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests restaurant_manager

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

This output confirms that our integration tests, which exercise the public API, are passing successfully.

## Unit Tests vs. Integration Tests for API Validation

While integration tests are the primary tool for testing the public API, unit tests still play a crucial role.


| **Aspect** | **Unit Tests (`src/lib.rs`)** | **Integration Tests (`tests/`)** |
| :-- | :-- | :-- |
| **Perspective** | **Internal (Developer)** | **External (User)** |
| **Access** | Can access `pub`, `pub(crate)`, and private items. | Can only access `pub` items. |
| **Purpose** | Test small, isolated pieces of logic, including internal implementation details. | Test how the public parts of the crate work together from a user's perspective. |
| **Typical Use Case** | Testing complex algorithms, edge cases of private functions. | Testing the end-to-end workflow of your crate's public API. |
| **Best Practice** | Keep them small and focused. | Treat them as examples of how to use your crate. |

A good testing strategy uses both:

* **Integration tests** to guarantee the stability and correctness of your public API.
* **Unit tests** to cover the internal logic, edge cases, and private helper functions that your public API relies on.


## Best Practices for Testing Your Public API

- **Prioritize Integration Tests**: For validating the public API, integration tests are the most important tool because they mimic real-world usage.
- **Test the "Happy Path"**: Ensure the most common, successful workflow through your API works as expected (like our `test_full_order_workflow_from_public_api` example).
- **Test Edge Cases and Errors**: Test how your API behaves with invalid inputs or in error conditions (like our `test_cannot_confirm_empty_order` example).
- **Keep Tests Independent**: Each integration test should be self-contained and not depend on the state left by other tests.
- **Use a `common` Module for Shared Setup**: If you have setup code that needs to be shared across multiple integration test files, you can create a `tests/common/mod.rs` file to house it.[^1]
- **Consider `public-api` Crate**: For advanced use cases, the `public-api` crate can be used to programmatically check for breaking changes in your API between versions, often used in CI pipelines.[^5][^6]


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Quality Inspection System**

Remember our comprehensive professional restaurant quality inspection analogy:

- 🍽️ **Public API Testing** = **Customer Experience Inspection** - Evaluating the entire service from a customer's point of view.
- 📋 **Integration Tests** = **The Customer's Journey** - A step-by-step test of ordering from the menu, receiving the food, and paying the bill.
- 🧑🍳 **Unit Tests** = **Internal Kitchen Checks** - Ensuring individual ingredients are fresh and each station operates correctly.
- 🛡️ **Confidence** = **Five-Star Rating** - Knowing your restaurant (and your crate) delivers a high-quality, reliable experience.


### **How to Test Your Public API in Rust**

1. **Create a `tests` Directory**: At the root of your crate, create a directory named `tests`.
2. **Write Test Files**: Add `.rs` files inside the `tests` directory. Each file will be compiled as a separate crate that depends on your main library.
3. **Import Your Crate**: In each test file, use `use your_crate_name::*` to import the public items from your library.
4. **Write `#[test]` Functions**: Write test functions that call your public API and use `assert!` macros to verify the behavior.
5. **Run with `cargo test`**: Cargo will automatically find and run both your unit tests and integration tests.

### **Key Differences: Unit vs. Integration Tests**

- **Unit Tests (`src/`)**:
    - Test internal logic.
    - Can access private items.
    - Good for testing individual components in isolation.
- **Integration Tests (`tests/`)**:
    - Test the public API.
    - Can only access public items.
    - Good for testing how different parts of your API work together.


### **Benefits of Testing Your Public API**

- **Ensures Correctness**: Verifies that your crate does what it promises.
- **Prevents Regressions**: Catches breaking changes to your public API before you release them.
- **Acts as Documentation**: Your integration tests serve as practical examples of how to use your crate.
- **Improves Design**: Writing tests from a user's perspective can help you design a more ergonomic and intuitive API.


### **The Professional Advantage**

**Mastering public API testing in Rust is like having a complete professional restaurant quality assurance program** that ensures every customer has a perfect experience:

- 🛡️ **User-Focused Quality**: You are building and verifying your software from the perspective of those who will use it.
- 🔄 **Stable and Reliable API**: Your crate becomes a dependable building block for other developers.
- 🚀 **Confident Development**: You can refactor internal code and add new features with confidence, knowing your public contract is protected by tests.
- 🏆 **Professional Grade**: A well-tested public API is a hallmark of a high-quality, professional Rust crate.

**Understanding how to effectively test your public API transforms you from a programmer who writes code that works to an engineer** who builds reliable, maintainable, and user-friendly libraries. Just as a top-tier restaurant meticulously ensures every aspect of the customer experience is flawless, a skilled Rust developer uses integration tests to guarantee that their crate's public API is robust, correct, and a pleasure to use.

1. https://doc.rust-lang.org/rust-by-example/testing/integration_testing.html
2. https://towardsdatascience.com/nine-rules-for-elegant-rust-library-apis-9b986a465247/
3. https://users.rust-lang.org/t/crate-level-tests/96565
4. https://blog.jetbrains.com/rust/2024/04/02/rust-unit-and-integration-testing-in-rustrover/
5. https://crates.io/crates/public-api
6. https://docs.rs/public-api
7. https://crates.io/crates/rust-api-test
8. https://zerotomastery.io/blog/complete-guide-to-testing-code-in-rust/
9. https://www.rapidinnovation.io/post/testing-and-debugging-rust-code
10. https://users.rust-lang.org/t/integration-tests-for-binary-crates/21373
