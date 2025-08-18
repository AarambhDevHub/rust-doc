+++
title = "Comprehensive Testing Exercise"
description = "A hands-on exercise combining unit, integration, and property-based testing in Rust."
date = "2025-08-13"
weight = 1
+++

# Comprehensive Testing Exercise in Rust: A Hands-On Guide

This guide provides a hands-on exercise that combines unit, integration, and property-based testing in Rust. It's designed for beginners and will walk you through building and testing a simple "Restaurant Order" library, explaining the "how" and "why" of each testing strategy.

## The Professional Restaurant Quality Assurance Analogy 🍽️

Think of building software like running a professional restaurant. To ensure quality, you need different levels of inspection:

1. **Unit Tests (Appliance Checks)**: You test each individual kitchen appliance in isolation. Does the oven heat to the correct temperature? Does the blender turn on? These are small, focused checks.
2. **Integration Tests (Workflow Checks)**: You test the entire process of making a dish. Does the order from the front-of-house correctly reach the kitchen? Do the ingredients from the prep station move to the cooking station correctly? This ensures all the individual parts work together.
3. **Property-Based Tests (Rule Checks)**: You test fundamental rules of cooking that must *always* be true, no matter the ingredients. For example, "cooking any raw food should never result in a dish that is still raw" or "the total bill should always be the sum of its items." These tests check for universal truths with a wide range of random inputs.

By combining these three strategies, you create a comprehensive quality assurance system that guarantees a flawless dining experience (or, in our case, robust and correct software).

## Project Setup

First, let's create a new Rust library for our restaurant order system.

1. **Create the Project**:

```bash
cargo new restaurant_order_system --lib
cd restaurant_order_system
```

2. **Add Dependencies for Property-Based Testing**:
Open your `Cargo.toml` file and add `quickcheck` and `quickcheck_macros` to the `[dev-dependencies]` section.

```toml
[dev-dependencies]
quickcheck = "1.0.3"
quickcheck_macros = "1.0.0"
```


## Part 1: Unit Testing (Testing the Appliances)

Unit tests focus on the smallest components of your code in isolation. We'll write them inside the same file as the code they are testing, within a `#[cfg(test)]` module.

### Step 1: Write the First Failing Test (Red)

Let's start by defining what an `Order` should look like and writing a test to ensure a new order is created empty.

Open `src/lib.rs` and add the following test code.

```rust
// src/lib.rs

#[cfg(test)]
mod tests {
    // Note: This test will fail to compile, which is the first step in TDD!
    #[test]
    fn test_new_order_is_empty() {
        let order = Order::new(101);
        assert_eq!(order.id, 101);
        assert!(order.items.is_empty());
        assert_eq!(order.total, 0.0);
    }
}
```

Run `cargo test`. It will fail to compile because `Order` doesn't exist yet. This is the **Red** phase of Test-Driven Development (TDD).

### Step 2: Write Code to Pass the Test (Green)

Now, let's write the minimal code required to make our test pass. Add this code **above** the `tests` module in `src/lib.rs`.

```rust
// src/lib.rs

#[derive(Debug, Clone, PartialEq)]
pub struct MenuItem {
    pub name: String,
    pub price: f64,
}

#[derive(Debug, PartialEq)]
pub struct Order {
    pub id: u32,
    pub items: Vec<MenuItem>,
    pub total: f64,
}

impl Order {
    /// Creates a new, empty order with a given ID.
    pub fn new(id: u32) -> Self {
        Order {
            id,
            items: Vec::new(),
            total: 0.0,
        }
    }
}
```

Finally, bring these new types into the scope of your tests by adding `use super::*;` to the `tests` module.

```rust
#[cfg(test)]
mod tests {
    use super::*; // Add this line

    #[test]
    fn test_new_order_is_empty() {
        let order = Order::new(101);
        assert_eq!(order.id, 101);
        assert!(order.items.is_empty());
        assert_eq!(order.total, 0.0);
    }
}
```

Run `cargo test` again. The test should now pass. This is the **Green** phase.

### Step 3: Add More Unit Tests (Repeat the Cycle)

Let's add a test for adding an item to the order.

```rust
// Inside the `tests` module
#[test]
fn test_add_item_updates_total_and_items() {
    // Arrange
    let mut order = Order::new(102);
    let pizza = MenuItem { name: "Pizza".to_string(), price: 15.99 };

    // Act
    order.add_item(pizza.clone());

    // Assert
    assert_eq!(order.items.len(), 1);
    assert_eq!(order.items[0], pizza);
    assert_eq!(order.total, 15.99);
}
```

This will fail because `add_item` doesn't exist. Let's implement it inside the `impl Order` block.

```rust
// Inside the `impl Order` block
/// Adds a MenuItem to the order and updates the total price.
pub fn add_item(&mut self, item: MenuItem) {
    self.total += item.price;
    self.items.push(item);
}
```

Run `cargo test` again. Both tests should now pass. You have successfully used unit tests to drive the development of your `Order` struct.

## Part 2: Integration Testing (Checking the Workflow)

Integration tests live in the `tests` directory and are used to test your library's public API as a user would. They ensure that different parts of your code work together correctly.

### Step 1: Create an Integration Test File

Create a new directory named `tests` in the root of your project, and inside it, create a file named `workflow_test.rs`.

**Project Structure:**

```
restaurant_order_system/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── workflow_test.rs
```


### Step 2: Write the Integration Test

In `tests/workflow_test.rs`, let's write a test that simulates a full user workflow: creating an order, adding multiple items, and then getting a summary.

```rust
// tests/workflow_test.rs

// Import the structs and functions from our library crate
use restaurant_order_system::{Order, MenuItem};

#[test]
fn test_full_order_and_summary() {
    // Arrange: Create a new order and some menu items
    let mut order = Order::new(201);
    let pizza = MenuItem { name: "Margherita Pizza".to_string(), price: 18.50 };
    let salad = MenuItem { name: "Caesar Salad".to_string(), price: 12.00 };

    // Act: Add items to the order
    order.add_item(pizza);
    order.add_item(salad);

    // Act: Get a summary of the order
    let summary = order.get_summary();

    // Assert: Check that the summary is correct
    let expected_summary = "Order #201 | Items: 2 | Total: $30.50";
    assert_eq!(summary, expected_summary);
}
```

This will fail because `get_summary` does not exist. Let's add it to `src/lib.rs`.

```rust
// Inside the `impl Order` block in src/lib.rs
/// Returns a formatted string summary of the order.
pub fn get_summary(&self) -> String {
    format!(
        "Order #{} | Items: {} | Total: ${:.2}",
        self.id,
        self.items.len(),
        self.total
    )
}
```

Run `cargo test`. All three tests (two unit, one integration) should pass. You've now verified that your public API works as expected for a common user workflow.

## Part 3: Property-Based Testing (Checking the Rules)

Property-based tests check that certain "properties" or rules of your code hold true for a wide range of randomly generated inputs. This is excellent for finding edge cases.

### Step 1: Define the Properties

What are some fundamental rules about our `Order` system?

1. **Adding an item always increases the total by that item's price.**
2. **The total of an order is always equal to the sum of the prices of its items.**

### Step 2: Implement the Property Tests

We will use the `quickcheck` crate we added earlier. `quickcheck` needs to know how to generate random instances of our types, so we must implement the `Arbitrary` trait for `MenuItem` and `Order`.

Add this code to `src/lib.rs`, inside the `tests` module.

```rust
// Inside the `tests` module in src/lib.rs

// Add these dependencies for property-based testing
use quickcheck::{Arbitrary, Gen};
use quickcheck_macros::quickcheck;

// Teach quickcheck how to create random MenuItems
impl Arbitrary for MenuItem {
    fn arbitrary(g: &mut Gen) -> Self {
        MenuItem {
            name: String::arbitrary(g),
            // Generate a reasonable, non-negative price
            price: f64::arbitrary(g).abs(),
        }
    }
}

// Teach quickcheck how to create random Orders
impl Arbitrary for Order {
    fn arbitrary(g: &mut Gen) -> Self {
        let items: Vec<MenuItem> = Arbitrary::arbitrary(g);
        let total = items.iter().map(|item| item.price).sum();
        Order {
            id: u32::arbitrary(g),
            items,
            total,
        }
    }
}

// --- Property Tests ---

#[quickcheck]
fn prop_add_item_increases_total_correctly(mut order: Order, item: MenuItem) -> bool {
    let initial_total = order.total;
    order.add_item(item.clone());
    // Use a small epsilon for float comparison
    (order.total - (initial_total + item.price)).abs() < 1e-9
}

#[quickcheck]
fn prop_total_is_sum_of_item_prices(order: Order) -> bool {
    let calculated_sum: f64 = order.items.iter().map(|item| item.price).sum();
    (order.total - calculated_sum).abs() < 1e-9
}
```


### Step 3: Run and Analyze

Run `cargo test`. `quickcheck` will run each property test 100 times with random data. If it finds a failing case, it will try to "shrink" it to the simplest possible example that still fails, which is incredibly useful for debugging.

All five of our tests should pass, giving us a high degree of confidence in our `Order` system.

## Summary: Your Comprehensive Testing Strategy

You have successfully built a small library guided by a comprehensive testing strategy:

1. **Unit Tests** (`src/lib.rs`): Verified that the core components (`Order::new`, `Order::add_item`) work correctly in isolation. They are fast and great for testing internal logic.
2. **Integration Tests** (`tests/` directory): Verified that the public API (`get_summary`) provides the expected outcome for a typical user workflow, ensuring the different parts of your code work together.
3. **Property-Based Tests** (`quickcheck`): Verified that fundamental rules of your system hold true for a vast range of inputs, protecting you against edge cases that you might not have thought of.

By combining these three types of tests, you create a robust safety net that allows you to develop new features and refactor existing code with confidence. This multi-layered approach is at the heart of writing reliable, professional-grade Rust software.
