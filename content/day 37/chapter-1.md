+++
title = "Integration Test Structure"
description = "Learn how to structure integration tests in Rust using the `tests` directory."
date = "2025-08-13"
weight = 1
+++

# Structuring Integration Tests in Rust: A Beginner's Guide to the `tests` Directory

Understanding how to structure integration tests in Rust is like learning to **design a comprehensive inspection of your entire restaurant's operation** – you need to test how different systems (like ordering, cooking, and payment) work together seamlessly, just as a real customer would experience them. While unit tests check individual components in isolation (like testing if a single oven heats up correctly), integration tests ensure the whole workflow is correct. In Rust, this is achieved by using the special `tests` directory, which allows you to test your library's public API from an external perspective.

## The Professional Restaurant Workflow Inspection Analogy 🍽️

### Imagine You're Inspecting the Entire Customer Experience

**The Problem with Only Unit Tests:**

```rust
// ❌ Unit tests only - like testing each kitchen appliance in isolation
// #[test] fn test_oven_heats_up() { ... }
// #[test] fn test_register_opens() { ... }
// #[test] fn test_dishwasher_runs() { ... }

// You know each part works on its own, but you don't know if an order
// can be successfully taken, cooked, served, and paid for.
```

**The Power of Integration Tests - Inspecting the Full Workflow:**

```rust
// ✅ Integration tests - testing the end-to-end process
// #[test]
// fn test_customer_orders_pizza_and_pays() {
//     let mut kitchen = Kitchen::new();
//     let mut register = Register::new();
//
//     let order = register.take_order("Pizza"); // Ordering system
//     let dish = kitchen.prepare_dish(order);     // Kitchen system
//     let final_bill = register.calculate_bill(dish); // Payment system
//
//     assert_eq!(final_bill, 15.99); // Verify the entire process works
// }
```

**Why Integration Test Structure Is Important in Rust:**

- 📦 **Black-Box Testing**: Tests your library's public API just like a real user would, ensuring it's ergonomic and correct from the outside.[^1]
- 🤝 **Verifies Component Interaction**: Ensures different parts of your code work together as expected.[^1]
- 🏗️ **Clear Separation**: Keeps tests of the public API separate from internal unit tests, leading to better organization.[^2]
- 🚀 **Independent Compilation**: Each integration test file is compiled as its own separate crate, providing a clean testing environment.[^2]
- 🎯 **Focused Testing**: Allows you to test specific user-facing workflows and features.


## The `tests` Directory: Rust's Integration Testing Home

Rust has a special convention for integration tests: any code placed in a `tests` directory at the root of your project is treated as an integration test.[^2]

**Standard Project Structure:**

```
my_restaurant_project/
├── Cargo.toml
├── src/
│   └── lib.rs  // Your library's source code
└── tests/
    └── order_workflow.rs // An integration test file
```

**Key Characteristics of the `tests` Directory:**

* It sits at the top level of your project, next to `src`.[^2]
* Cargo automatically knows to look for integration tests here.[^2]
* Each `.rs` file inside `tests` is compiled as a **separate crate**.[^2]
* Because they are separate crates, you must import your library's public items using `use my_crate_name::...;`.[^3]


## A Step-by-Step Guide to Structuring Integration Tests

Let's build a simple restaurant order management system and write an integration test for it.

**Project Setup:**

1. Create a new Rust library project: `cargo new restaurant_manager --lib`
2. Navigate into the project: `cd restaurant_manager`

### Step 1: Write the Library Code in `src/lib.rs`

First, let's create a simple public API for our restaurant manager.

Open `src/lib.rs` and add the following code:

```rust
// src/lib.rs

use std::collections::HashMap;

#[derive(Debug, Clone, PartialEq)]
pub struct MenuItem {
    pub name: String,
    pub price: f64,
}

#[derive(Debug, Default)]
pub struct Order {
    items: Vec<MenuItem>,
    total: f64,
    is_finalized: bool,
}

impl Order {
    pub fn new() -> Self {
        Order::default()
    }

    pub fn add_item(&mut self, item: MenuItem) {
        if !self.is_finalized {
            self.total += item.price;
            self.items.push(item);
        }
    }

    pub fn finalize(&mut self) -> f64 {
        self.is_finalized = true;
        self.total
    }

    pub fn get_total(&self) -> f64 {
        self.total
    }
}

// A simple menu for our restaurant
pub fn get_menu() -> HashMap<String, MenuItem> {
    let mut menu = HashMap::new();
    menu.insert("Pizza".to_string(), MenuItem { name: "Pizza".to_string(), price: 15.99 });
    menu.insert("Salad".to_string(), MenuItem { name: "Salad".to_string(), price: 9.50 });
    menu
}
```

This code defines our public API: `MenuItem`, `Order`, and `get_menu`.

### Step 2: Create the `tests` Directory and an Integration Test File

Now, let's create our integration test.

1. Create a `tests` directory in the root of your `restaurant_manager` project.
2. Inside the `tests` directory, create a new file named `full_order_process.rs`.

Your directory structure should now look like this:

```
restaurant_manager/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── full_order_process.rs
```


### Step 3: Write the Integration Test

Open `tests/full_order_process.rs` and add the following test code. Note how we use `restaurant_manager::*` to import our library's public API.

```rust
// tests/full_order_process.rs

use restaurant_manager::*; // Import our library's public items

#[test]
fn test_customer_creates_and_finalizes_order() {
    // 1. Get the menu (as a customer would)
    let menu = get_menu();
    let pizza = menu.get("Pizza").unwrap().clone();
    let salad = menu.get("Salad").unwrap().clone();

    // 2. Create a new order
    let mut order = Order::new();
    assert_eq!(order.get_total(), 0.0);

    // 3. Add items to the order
    order.add_item(pizza);
    assert_eq!(order.get_total(), 15.99);

    order.add_item(salad);
    assert_eq!(order.get_total(), 25.49); // 15.99 + 9.50

    // 4. Finalize the order
    let final_bill = order.finalize();
    assert_eq!(final_bill, 25.49);

    // 5. Verify no more items can be added after finalization
    let extra_pizza = menu.get("Pizza").unwrap().clone();
    order.add_item(extra_pizza);
    assert_eq!(order.get_total(), 25.49); // Total should not change
}
```

This test simulates a real user's workflow: getting the menu, creating an order, adding items, finalizing it, and verifying the total.

### Step 4: Run the Tests

Now, run the tests from your project's root directory:

```bash
cargo test
```

**Expected Output:**
You will see output indicating that Cargo is running both unit tests (if any) and integration tests.

```
   Compiling restaurant_manager v0.1.0 (...)
    Finished test [unoptimized + debuginfo] target(s) in ...s
     Running unittests src/lib.rs (target/debug/deps/restaurant_manager-...)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/full_order_process.rs (target/debug/deps/full_order_process-...)

running 1 test
test test_customer_creates_and_finalizes_order ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests restaurant_manager

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The output shows a separate section for `Running tests/full_order_process.rs`, confirming that our integration test was found and executed successfully.

## Sharing Code Between Integration Tests

As your test suite grows, you might need to share setup code or helper functions across multiple integration test files.

**The Problem**: If you create a `tests/common.rs` file, Cargo will treat it as another integration test crate and try to run tests inside it, which is not what you want.

**The Solution**: Create a module inside the `tests` directory. The standard way to do this is to create a subdirectory and a `mod.rs` file inside it.[^2]

**Example of Sharing Code:**

1. Create a new directory: `tests/common`
2. Create a file inside it: `tests/common/mod.rs`

Your directory structure will look like this:

```
restaurant_manager/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    ├── common/
    │   └── mod.rs          // Shared helper code
    └── full_order_process.rs // Your main integration test
```

Now, let's add a helper function to `tests/common/mod.rs`:

```rust
// tests/common/mod.rs
use restaurant_manager::*;

// This function can be shared by multiple integration tests
pub fn create_order_with_items() -> Order {
    let mut order = Order::new();
    let menu = get_menu();

    order.add_item(menu.get("Pizza").unwrap().clone());
    order.add_item(menu.get("Salad").unwrap().clone());

    order
}
```

To use this shared code in `tests/full_order_process.rs`, declare it as a module:

```rust
// tests/full_order_process.rs

use restaurant_manager::*;

mod common; // This makes the `common` module available

#[test]
fn test_order_creation_with_helper() {
    // Use the shared helper function
    let order = common::create_order_with_items();

    assert_eq!(order.get_total(), 25.49);
}

// ... other tests ...
```

By placing shared code in a `mod.rs` file inside a subdirectory, you prevent Cargo from treating it as a test crate, while still allowing you to share code across your integration tests.

## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Workflow Inspection**

Remember our comprehensive professional restaurant workflow inspection analogy:

- 📦 **Integration Tests** = **End-to-End Workflow Inspection** - Verifying that all systems (ordering, kitchen, payment) work together correctly.
- 🏗️ **`tests` Directory** = **The Inspector's Office** - A separate space where tests are designed to interact with the restaurant just like a customer.
- 🤝 **Shared Code (`tests/common/mod.rs`)** = **The Inspector's Toolkit** - Reusable tools and checklists that can be used across multiple inspections.
- 🚀 **`cargo test`** = **Running the Inspection** - The official process for executing all checks and reporting the results.


### **Key Rules for Integration Test Structure in Rust**

| **Rule** | **Explanation** | **Reference** |
| :-- | :-- | :-- |
| **Location** | Create a `tests` directory at the root of your project, next to `src`. |  |
| **Compilation** | Each `.rs` file in the `tests` directory is compiled as a separate crate. | [^2] |
| **Accessing Your Library** | You must import your library's public API using `use my_crate_name::...;`. | [^3] |
| **No `#[cfg(test)]` Needed** | The `#[cfg(test)]` attribute is not needed for integration tests, as Cargo knows they are only for testing. | [^2] |
| **Sharing Code** | To share code, create a subdirectory (e.g., `tests/common`) and place the shared code in a `mod.rs` file inside it. | [^2] |
| **Running Tests** | `cargo test` automatically finds and runs all integration tests. | [^1] |

### **Unit Tests vs. Integration Tests: A Quick Comparison**

| **Aspect** | **Unit Tests** | **Integration Tests** |
| :-- | :-- | :-- |
| **Location** | In `src`, inside a `#[cfg(test)] mod tests`. | In the root `tests` directory. |
| **Access** | Can access private functions and types. | Can only access the public API. |
| **Purpose** | Test individual components in isolation. | Test how components work together. |
| **Compilation** | Compiled as part of the library crate. | Each file is compiled as a separate crate. |
| **Use Case** | Testing internal logic and edge cases. | Testing public-facing workflows and features. |

### **Best Practices for Structuring Integration Tests**

- **Organize by Feature**: Create separate `.rs` files in the `tests` directory for different features or user stories (e.g., `user_authentication.rs`, `order_management.rs`).
- **Use Helper Modules**: For complex projects, use a `common` module (or other appropriately named modules) to share setup code, test data factories, or common assertions.
- **Test the Public API**: Focus your integration tests on what a user of your library would do. Avoid testing internal implementation details.
- **Keep Tests Independent**: Each integration test should be ableto run independently of others. Use setup and teardown logic within each test (or with helper functions) to ensure a clean state.
- **Use Descriptive File Names**: Name your test files clearly to indicate what functionality they are testing.


### **The Professional Advantage**

**Mastering integration test structure in Rust is like having a complete professional restaurant quality assurance system** that ensures your entire operation runs smoothly from the customer's perspective:

- 🛡️ **Confidence in Public API**: Ensures your library works as advertised for your users.
- 🤝 **Robust Integration**: Guarantees that different parts of your system work together correctly.
- 🏗️ **Clean and Organized Testing**: Separates internal implementation tests from external workflow tests.
- 🔄 **Safe Refactoring**: Allows you to change internal implementations with confidence, knowing that the public API remains correct.
- 🏆 **Professional Reliability**: Delivers software that is not only correct in its parts but also as a whole.

**Understanding how to structure integration tests transforms you from a programmer who just tests functions to an engineer** who validates entire systems. Just as a master restaurant manager ensures a seamless customer experience from start to finish, a skilled Rust programmer uses integration tests to build software that is robust, reliable, and correct in its real-world application.

1. https://doc.rust-lang.org/rust-by-example/testing/integration_testing.html
2. https://doc.rust-lang.org/book/ch11-03-test-organization.html
3. https://stackoverflow.com/questions/76979070/how-to-properly-use-a-tests-folder-in-a-rust-project
4. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch11-03-test-organization.html
5. https://blog.jetbrains.com/rust/2024/04/02/rust-unit-and-integration-testing-in-rustrover/
6. https://zerotomastery.io/blog/complete-guide-to-testing-code-in-rust/
7. https://www.reddit.com/r/rust/comments/qk77iu/best_way_to_organise_tests_in_rust/
8. https://www.youtube.com/watch?v=MiLp0InGOkI
9. https://users.rust-lang.org/t/how-to-properly-divide-tests-into-unit-and-integration-tests/92679
