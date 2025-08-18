+++
title = "Test Organization"
description = "Best practices for organizing tests in Rust projects, including unit and integration tests."
date = "2025-08-13"
weight = 3
+++

# Test Organization in Rust: Comprehensive Documentation for Beginners

Organizing tests in a Rust project is like **setting up a comprehensive quality control system for a professional restaurant** – you need different types of checks to ensure the highest quality at every stage. Just as a restaurant has separate procedures for tasting individual ingredients (unit tests), verifying that a complete dish is assembled correctly (integration tests), and ensuring the public menu is accurate (documentation tests), Rust provides a structured way to organize your tests to maintain code quality, prevent regressions, and ensure your project works as expected.

## The Professional Restaurant Quality Control Analogy 🍽️

### Imagine You're Establishing a Quality Control System for Your Restaurant

**The Problem with Disorganized Quality Control:**

```rust
// ❌ Disorganized approach - like cooks randomly tasting things
// - No clear distinction between ingredient checks and final dish checks
// - Hard to know what's been tested
// - Inefficient and prone to missing issues
```

**The Power of Structured Testing in Rust - A Professional Quality System:**

```rust
// ✅ Structured approach - different checks for different purposes
// 1. Ingredient Tasting (Unit Tests): Check individual components
//    - "Does this sauce have the right salt level?"
//
// 2. Final Dish Verification (Integration Tests): Check how components work together
//    - "Is the final plated pizza correct and delicious?"
//
// 3. Menu Accuracy Checks (Documentation Tests): Ensure public information is correct
//    - "Does the menu description match the actual dish?"
```

**Why Structured Test Organization Is Critical:**

- 🎯 **Clarity**: Everyone knows where to find and add tests.
- 🚀 **Efficiency**: Run different types of tests separately (e.g., fast unit tests vs. slower integration tests).
- 🛡️ **Confidence**: Ensures all parts of your code, both internal and public, are working correctly.
- 📚 **Documentation**: Tests serve as examples of how to use your code.
- 📈 **Maintainability**: A well-organized test suite is easier to maintain as your project grows.


## Understanding Test Organization Fundamentals in Rust

### The Three Pillars of Rust Testing: Unit, Integration, and Doc Tests

**Rust's testing framework is built around three main types of tests, each with a specific purpose and location in your project:**

```rust
fn demonstrate_test_organization_fundamentals() {
    println!("🍽️ Test Organization in Rust - The Three Pillars of Quality Control");
    println!("{:=<70}", "");

    // Rust testing is divided into three main categories
    println!("📋 The Three Main Categories of Tests in Rust:");
    println!("   1️⃣ Unit Tests - Tasting Individual Ingredients 🧪");
    println!("   2️⃣ Integration Tests - Verifying the Final Dish 🍕");
    println!("   3️⃣ Documentation Tests - Checking the Public Menu 📜");

    // Example 1: Project Structure Overview
    println!("\n1️⃣ Project Structure Overview:");
    println!("   📁 A typical Rust project with organized tests:");
    println!("   ``` ");
    println!("   my_restaurant/");
    println!("   ├── Cargo.toml");
    println!("   ├── src/");
    println!("   │   ├── lib.rs       // Contains unit tests and doc tests");
    println!("   │   └── main.rs      // For binary crates");
    println!("   └── tests/");
    println!("       └── orders.rs    // Contains integration tests");
    println!("   ```");

    // Example 2: Unit Tests - Tasting Individual Ingredients
    println!("\n2️⃣ Unit Tests - Testing Individual Components:");
    println!("   🎯 Purpose: To test a small, specific piece of code (like a single function or method) in isolation.");
    println!("   📍 Location: Inside the same file as the code being tested, within a `#[cfg(test)] mod tests { ... }` block [^237][^241].");
    println!("   ✅ Key Feature: Can access private functions and variables of the module, allowing for detailed 'white-box' testing [^2][^7].");

    // Simulate `src/lib.rs` with a unit test
    println!("\n   🧪 Example: `src/lib.rs`");
    println!("   ``` ");
    println!("   // src/lib.rs");
    println!("   ");
    println!("   /// Adds two numbers. Private helper function.");
    println!("   fn add_private(a: i32, b: i32) -> i32 {");
    println!("       a + b");
    println!("   }");
    println!("   ");
    println!("   /// Public function to calculate total cost.");
    println!("   pub fn calculate_total(price: i32, tax: i32) -> i32 {");
    println!("       add_private(price, tax)");
    println!("   }");
    println!("   ");
    println!("   #[cfg(test)] // This module only compiles when testing");
    println!("   mod tests {");
    println!("       use super::*; // Import from the parent module");
    println!("   ");
    println!("       #[test]");
    println!("       fn test_private_addition() {");
    println!("           // We can test the private `add_private` function here");
    println!("           assert_eq!(add_private(2, 3), 5);");
    println!("       }");
    println!("   ");
    println!("       #[test]");
    println!("       fn test_calculate_total() {");
    println!("           assert_eq!(calculate_total(10, 2), 12);");
    println!("       }");
    println!("   }");
    println!("   ```");

    // Example 3: Integration Tests - Verifying the Final Dish
    println!("\n3️⃣ Integration Tests - Testing How Components Work Together:");
    println!("   🎯 Purpose: To test the public API of your library crate as an external user would, ensuring different parts work together correctly [^237].");
    println!("   📍 Location: Inside a separate `tests` directory at the root of your project. Each `.rs` file in `tests/` is a separate integration test crate [^6].");
    println!("   ✅ Key Feature: Can only access `pub` items from your library, providing true 'black-box' testing of your public interface [^3][^7].");

    // Simulate `tests/orders.rs` with an integration test
    println!("\n   🍕 Example: `tests/orders.rs`");
    println!("   ``` ");
    println!("   // tests/orders.rs");
    println!("   ");
    println!("   // Import the library crate being tested");
    println!("   use my_restaurant;");
    println!("   ");
    println!("   #[test]");
    println!("   fn test_order_total_calculation() {");
    println!("       // We can only use the public `calculate_total` function");
    println!("       let final_price = my_restaurant::calculate_total(100, 15);");
    println!("       assert_eq!(final_price, 115);");
    println!("       ");
    println!("       // We cannot call private functions from here:");
    println!("       // my_restaurant::add_private(2, 2); // This would be a compile error!");
    println!("   }");
    println!("   ```");

    // Example 4: Documentation Tests - Checking the Public Menu
    println!("\n4️⃣ Documentation Tests - Ensuring Examples Are Correct:");
    println!("   🎯 Purpose: To provide examples in your documentation that also double as tests, ensuring they remain correct as your code evolves [^238].");
    println!("   📍 Location: Inside documentation comments (`///`) in your `src` files.");
    println!("   ✅ Key Feature: Makes your documentation more reliable and serves as a great way to show how to use your public API.");

    // Simulate `src/lib.rs` with a doc test
    println!("\n   📜 Example: Doc test in `src/lib.rs`");
    println!("   ``` ");
    println!("   // src/lib.rs");
    println!("   ");
    println!("   /// Calculates the total cost of an item, including tax.");
    println!("   ///");
    println!("   /// # Examples");
    println!("   ///");
    println!("   /// ```");
    println!("   /// let total = my_restaurant::calculate_total(50, 5);");
    println!("   /// assert_eq!(total, 55);");
    println!("   /// ``` ");
    println!("   pub fn calculate_total(price: i32, tax: i32) -> i32 {");
    println!("       price + tax");
    println!("   }");
    println!("   ```");

    println!("\n🎯 Test Organization Fundamentals Summary:");
    println!("   • **Unit Tests**: In `src/`, test private details, close to the code.");
    println!("   • **Integration Tests**: In `tests/`, test public API, like a user.");
    println!("   • **Doc Tests**: In `///` comments, provide examples that are also tests.");
}

fn main() {
    demonstrate_test_organization_fundamentals();
}
```


## How to Organize Tests in Your Project

### A Practical Guide to Structuring Your Quality Control System

**Follow these best practices to create a well-organized and maintainable test suite for your Rust projects.**

```rust
fn how_to_organize_tests_in_practice() {
    println!("🔧 How to Organize Tests in Practice - A Quality Control Guide");
    println!("{:=<70}", "");

    // This section provides a practical guide to structuring tests in a typical Rust project
    println!("📋 Step-by-Step Guide to Test Organization:");

    // Step 1: Library vs. Binary Crate Structure
    println!("\n1️⃣ Step 1: Structure Your Project with a Library and a Binary Crate:");
    println!("   🎯 Best Practice: For projects with both a library and an executable, separate the core logic into a library crate (`src/lib.rs`) and keep the binary crate (`src/main.rs`) as a thin wrapper [^3].");
    println!("   ✅ Why? This makes your core logic easily testable through integration tests, as binary crates do not have a public API that can be tested from the `tests` directory [^3].");
    println!("   ``` ");
    println!("   my_restaurant_app/");
    println!("   ├── Cargo.toml");
    println!("   ├── src/");
    println!("   │   ├── lib.rs       // Core logic, fully testable");
    println!("   │   └── main.rs      // Thin wrapper, calls library functions");
    println!("   └── tests/");
    println!("       └── public_api.rs // Integration tests for the library");
    println!("   ```");

    // Step 2: Unit Tests for Internal Logic
    println!("\n2️⃣ Step 2: Place Unit Tests in a `tests` Submodule:");
    println!("   🎯 Best Practice: Inside each module of your `src` directory, create a `#[cfg(test)] mod tests { ... }` block for unit tests related to that module [^1][^6].");
    println!("   ✅ Why? This keeps tests close to the code they are testing, making them easy to find and update. It also allows you to test private functions and data structures [^2].");
    println!("\n   🧪 Example: `src/kitchen.rs`");
    println!("   ``` ");
    println!("   // src/kitchen.rs");
    println!("   ");
    println!("   fn chop_vegetables() -> bool { /* ... */ true }");
    println!("   fn mix_ingredients() -> bool { /* ... */ true }");
    println!("   ");
    println!("   pub fn prepare_dish() -> String {");
    println!("       if chop_vegetables() && mix_ingredients() {");
    println!("           \"Dish prepared\".to_string()");
    println!("       } else {");
    println!("           \"Preparation failed\".to_string()");
    println!("       }");
    println!("   }");
    println!("   ");
    println!("   #[cfg(test)]");
    println!("   mod tests {");
    println!("       use super::*;");
    println!("   ");
    println!("       #[test]");
    println!("       fn test_chopping() {");
    println!("           assert!(chop_vegetables());");
    println!("       }");
    println!("   ");
    println!("       #[test]");
    println!("       fn test_mixing() {");
    println!("           assert!(mix_ingredients());");
    println!("       }");
    println!("   }");
    println!("   ```");

    // Step 3: Integration Tests for Public API
    println!("\n3️⃣ Step 3: Create a `tests` Directory for Integration Tests:");
    println!("   🎯 Best Practice: At the root of your project, create a `tests` directory. Each `.rs` file inside this directory becomes a separate integration test crate [^241].");
    println!("   ✅ Why? This tests your library from an external perspective, ensuring your public API is correct, usable, and stable. It forces you to think like a user of your library [^2].");
    println!("\n   🍕 Example: `tests/public_api_tests.rs`");
    println!("   ``` ");
    println!("   // tests/public_api_tests.rs");
    println!("   ");
    println!("   // Use your library crate");
    println!("   use my_restaurant_app;");
    println!("   ");
    println!("   #[test]");
    println!("   fn test_prepare_dish_publicly() {");
    println!("       let dish = my_restaurant_app::kitchen::prepare_dish();");
    println!("       assert_eq!(dish, \"Dish prepared\");");
    println!("   }");
    println!("   ```");
    println!("   💡 Tip: You can have multiple integration test files, e.g., `tests/orders.rs`, `tests/kitchen.rs`, to organize tests by feature.");

    // Step 4: Documentation Tests for Examples
    println!("\n4️⃣ Step 4: Add Documentation Tests for Public API Examples:");
    println!("   🎯 Best Practice: Write code examples in your documentation comments (`///`) for all public functions, structs, and modules [^238].");
    println!("   ✅ Why? `cargo test` automatically runs these examples, ensuring your documentation is always up-to-date and correct. This provides excellent, tested examples for your users [^238].");
    println!("\n   📜 Example: Doc test in `src/lib.rs`");
    println!("   ``` ");
    println!("   // src/lib.rs");
    println!("   ");
    println!("   /// Prepares a dish using internal kitchen logic.");
    println!("   ///");
    println!("   /// # Examples");
    println!("   ///");
    println!("   /// ```");
    println!("   /// let my_dish = my_restaurant_app::kitchen::prepare_dish();");
    println!("   /// assert_eq!(my_dish, \"Dish prepared\");");
    println!("   /// ``` ");
    println!("   pub mod kitchen {");
    println!("       // ... kitchen module code here ... ");
    println!("   }");
    println!("   ```");

    // Step 5: Handling Tests for Binary Crates
    println!("\n5️⃣ Step 5: Handling Tests for Binary-Only Crates:");
    println!("   🎯 Challenge: Binary crates (`src/main.rs`) don't have a library to link against for integration tests.");
    println!("   ✅ Solution: If your binary is complex, refactor the core logic into a library (`src/lib.rs`) and have `main.rs` call it. This makes the logic testable [^238].");
    println!("   💡 For simple binaries, you can still write unit tests within `src/main.rs` using the same `#[cfg(test)] mod tests` pattern.");

    println!("\n🎯 Practical Test Organization Summary:");
    println!("   • **Keep unit tests close**: In a `tests` submodule in the same file as the code.");
    println!("   • **Separate integration tests**: In the `tests` directory at the project root.");
    println!("   • **Document with tests**: Use doc tests (`///`) for public API examples.");
    println!("   • **Structure for testability**: Prefer a library + binary structure for complex applications.");
}

fn main() {
    how_to_organize_tests_in_practice();
}
```


## Running Your Tests

### Executing Your Quality Control Checks

**Cargo, Rust's build tool and package manager, makes running tests straightforward.**

```rust
fn how_to_run_tests() {
    println!("🚀 Running Your Tests - Executing Quality Control Checks");
    println!("{:=<70}", "");

    println!("📋 Cargo Commands for Running Tests:");

    // 1. Run All Tests
    println!("\n1️⃣ Run All Tests (Unit, Integration, and Doc Tests):");
    println!("   ``` ");
    println!("   cargo test");
    println!("   ```");
    println!("   💡 This command compiles your code in test mode and runs all tests in parallel.");

    // 2. Run Only Unit Tests
    println!("\n2️⃣ Run Only Unit Tests:");
    println!("   ``` ");
    println!("   cargo test --lib");
    println!("   ```");
    println!("   💡 Use this for quick checks on internal logic.");

    // 3. Run Only Integration Tests
    println!("\n3️⃣ Run Only Integration Tests:");
    println!("   ``` ");
    println!("   cargo test --test <TEST_FILE_NAME>");
    println!("   ```");
    println!("   Example: `cargo test --test public_api_tests`");
    println!("   💡 Use this to test a specific integration test file.");
    println!("   To run all integration tests: you can omit the test name, but it might run others as well.");

    // 4. Run Only Doc Tests
    println!("\n4️⃣ Run Only Doc Tests:");
    println!("   ``` ");
    println!("   cargo test --doc");
    println!("   ```");
    println!("   💡 Use this to verify that your documentation examples are correct.");

    // 5. Run a Specific Test
    println!("\n5️⃣ Run a Specific Test by Name:");
    println!("   ``` ");
    println!("   cargo test <TEST_NAME>");
    println!("   ```");
    println!("   Example: `cargo test test_chopping` will run the test function named `test_chopping`.");
    println!("   💡 This is useful for focusing on a single test while debugging.");

    // 6. Filtering Tests
    println!("\n6️⃣ Filtering Tests:");
    println!("   ``` ");
    println!("   cargo test <PART_OF_TEST_NAME>");
    println!("   ```");
    println!("   Example: `cargo test kitchen` will run all tests whose names contain `kitchen`.");

    // 7. Ignoring Tests
    println!("\n7️⃣ Ignoring Slow or Expensive Tests:");
    println!("   🎯 Use the `#[ignore]` attribute on a test function to exclude it from `cargo test` by default.");
    println!("   ``` ");
    println!("   #[test]");
    println!("   #[ignore]");
    println!("   fn slow_integration_test() {");
    println!("       // This test takes a long time");
    println!("   }");
    println!("   ```");
    println!("   💡 To run only the ignored tests:");
    println!("   ``` ");
    println!("   cargo test -- --ignored");
    println!("   ```");
    println!("   💡 To run all tests, including ignored ones:");
    println!("   ``` ");
    println!("   cargo test -- --include-ignored");
    println!("   ```");
}

fn main() {
    how_to_run_tests();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Quality Control System**

Remember our comprehensive professional restaurant quality control analogy:

- 🍽️ **Test Organization**: A structured quality control system for your code.
- 🧪 **Unit Tests**: Tasting individual ingredients (`src/your_module.rs`).
- 🍕 **Integration Tests**: Verifying the final dish (`tests/your_feature.rs`).
- 📜 **Doc Tests**: Checking the public menu (`///` comments in `src`).
- 🚀 **`cargo test`**: The quality control manager who runs all the checks.


### **Best Practices for Test Organization in Rust**

1. **Separate Unit and Integration Tests**: This is the most fundamental and important practice.[^2]
    * **Unit Tests**: Keep them in the same file as the code they're testing, inside a `#[cfg(test)] mod tests { ... }` block. They test private implementation details.
    * **Integration Tests**: Place them in the `tests` directory at the root of your project. They test your library's public API as a user would.
2. **Structure for Testability**: For applications with a binary executable, refactor the core logic into a library crate (`src/lib.rs`). This allows your core logic to be tested via integration tests, which is not possible for a binary-only crate.[^3]
3. **Document with Tests**: Use documentation tests (`///`) to provide clear, tested examples of how to use your public API. This ensures your documentation never goes out of date.[^3]
4. **Organize by Feature**: For larger projects, organize your tests by feature.
    * In `src`, each module can have its own `tests` submodule.
    * In the `tests` directory, you can have multiple files, e.g., `tests/orders.rs`, `tests/kitchen.rs`, `tests/menu.rs`, to group related integration tests.
5. **Use Descriptive Test Names**: Name your tests clearly to indicate what they are testing. Avoid generic names like `test1`. For example, `test_calculate_total_with_tax` is much better than `test_total`.[^5]
6. **Use `#[ignore]` for Slow Tests**: Mark tests that are slow or require special setup (e.g., network access, database) with `#[ignore]`. This allows you to run most of your tests quickly during normal development and run the slow tests separately when needed.

### **Test Organization Quick Reference Table**

| **Test Type** | **Location** | **Purpose** | **Access** | **Best For** |
| :-- | :-- | :-- | :-- | :-- |
| **Unit Tests** | `src/module.rs` inside `#[cfg(test)] mod tests` | Test individual components in isolation. | Public and private items of module. | Testing internal logic, private functions. |
| **Integration Tests** | `tests/feature.rs` at project root | Test the library's public API as a user. | Public items of the library crate. | Testing how modules work together, public API. |
| **Doc Tests** | `///` comments in `src` files | Provide examples that are also tests. | Public items of the library crate. | Documenting public API with tested examples. |

### **The Professional Advantage**

**Mastering test organization in Rust is like building a robust quality control system for a professional restaurant** - it ensures that every part of your project, from individual components to the final product, is of the highest quality:

- 🛡️ **Confidence**: A well-organized test suite gives you confidence to refactor and add new features without breaking existing functionality.
- 🚀 **Efficiency**: Separating tests allows you to run fast checks during development and slower, more comprehensive checks when needed.
- 📚 **Clarity**: A clear testing structure makes your project easier to understand for new contributors.
- 📈 **Maintainability**: Organized tests are easier to maintain and update as your codebase grows.
- 🎯 **Professionalism**: Demonstrates a commitment to quality and a mature approach to software development.

By following these best practices, you can create a testing strategy that is not only effective but also a pleasure to work with, leading to higher-quality, more reliable Rust applications.

1. https://users.rust-lang.org/t/real-world-tips-for-organising-unit-tests-for-larger-projects-and-files/130749
2. https://doc.rust-lang.org/book/ch11-03-test-organization.html
3. https://www.reddit.com/r/rust/comments/qk77iu/best_way_to_organise_tests_in_rust/
4. https://zerotomastery.io/blog/complete-guide-to-testing-code-in-rust/
5. https://rustc-dev-guide.rust-lang.org/tests/best-practices.html
6. https://stackoverflow.com/questions/76979070/how-to-properly-use-a-tests-folder-in-a-rust-project
7. https://blog.logrocket.com/how-to-organize-rust-tests/
8. https://forum.dfinity.org/t/good-practice-testing-structure-for-a-rust-project/37390
