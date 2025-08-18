+++
title = "Writing Test Functions"
description = "Learn how to write test functions in Rust using the built-in testing framework."
date = "2025-08-13"
weight = 1
+++

<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Writing Test Functions in Rust: Comprehensive Documentation for Beginners

Understanding how to write test functions in Rust is like learning to **implement a rigorous quality control system for your professional restaurant chain** – you need systematic procedures to verify that every dish, ingredient, and process meets the highest standards before it reaches the customer. Just as a professional kitchen has dedicated procedures for tasting dishes, checking ingredient freshness, and verifying order accuracy, Rust's built-in testing framework provides powerful tools to write, organize, and run tests that ensure your code is correct, reliable, and bug-free.

## The Professional Kitchen Quality Control System Analogy 👨🍳

### Imagine You're Designing a Quality Control System for a High-End Restaurant

**The Problem without Systematic Testing:**

```rust
// ❌ Manual "taste testing" - like manually checking every order before it goes out
// - Inconsistent: Different chefs might check differently
// - Inefficient: Takes a lot of time and effort
// - Not scalable: Hard to keep up during peak hours
// - Error-prone: Easy to miss small mistakes
// Result: Inconsistent quality, customer complaints, and potential failures
```

**The Power of Rust's Built-in Testing Framework - Automated Quality Control:**

```rust
// ✅ Automated quality control - like having a dedicated testing station
// with automated checks for every dish

// The dish we're testing
fn prepare_dish(ingredient: &str) -> String {
    format!("Perfectly prepared {}", ingredient)
}

// The automated test
#[test]
fn check_dish_preparation() {
    let ingredient = "Wagyu Steak";
    let expected_result = "Perfectly prepared Wagyu Steak";
    let actual_result = prepare_dish(ingredient);

    // Automated check: Does the result match our expectation?
    assert_eq!(actual_result, expected_result, "Dish preparation did not meet standards!");
}

// Result: Consistent, reliable, and automated quality control for every dish!
```

**Why Rust's Testing Framework Is Essential:**

- 🛡️ **Reliability**: Ensures your code works correctly and as expected.[^5]
- 🚀 **Confidence**: Allows you to refactor and add new features without fear of breaking existing functionality.[^4]
- 🔄 **Regression Prevention**: Catches bugs that might be introduced by new changes.[^5]
- 📋 **Documentation**: Tests serve as living documentation, showing how your code is intended to be used.[^3]
- ⚡ **Efficiency**: Automated tests are much faster and more reliable than manual checks.


## Understanding Test Functions Fundamentals

### The Dedicated Quality Control Station

**In Rust, tests are functions marked with the `#[test]` attribute, organized to verify your code's correctness:**

```rust
fn demonstrate_test_function_fundamentals() {
    println!("🧪 Test Function Fundamentals - The Dedicated Quality Control Station");
    println!("{:=<70}", "");

    println!("📋 Core Concepts of Rust Testing:");
    println!("   • `#[test]` Attribute: Marks a function as a test case [^4].");
    println!("   • `#[cfg(test)]` Attribute: Conditionally compiles test code only when running tests [^5].");
    println!("   • `mod tests`: A conventional module for organizing unit tests within a file [^3].");
    println!("   • Assertion Macros: Macros like `assert!`, `assert_eq!`, and `assert_ne!` to verify conditions [^1].");
    println!("   • `cargo test`: The command to run all tests in your project [^4].");

    // Example 1: Anatomy of a Simple Test Function
    println!("\n1️⃣ Anatomy of a Simple Test - Checking a Single Ingredient:");

    // The function we are testing
    fn is_ingredient_fresh(ingredient: &str) -> bool {
        !ingredient.contains("expired")
    }

    // A simple test function
    #[test]
    fn test_fresh_ingredient() {
        let ingredient = "Fresh Tomato";
        // `assert!` macro checks if the expression is true.
        // If it's false, the test panics and fails.
        assert!(is_ingredient_fresh(ingredient), "The ingredient should be fresh!");
    }

    println!("   💡 A test function is a normal Rust function marked with `#[test]`.");
    println!("   💡 It uses assertion macros to check for expected outcomes.");
    println!("   💡 A test passes if it runs without panicking, and fails if it panics.");

    // Example 2: Using Assertion Macros
    println!("\n2️⃣ Using Assertion Macros - Verifying Dish Quality:");

    // Function to test
    fn combine_ingredients(ingredient1: &str, ingredient2: &str) -> String {
        format!("{} & {}", ingredient1, ingredient2)
    }

    #[test]
    fn test_ingredient_combination() {
        let ingredient1 = "Tomato";
        let ingredient2 = "Basil";
        let expected_dish = "Tomato & Basil";

        let actual_dish = combine_ingredients(ingredient1, ingredient2);

        // `assert_eq!` checks if two values are equal.
        assert_eq!(actual_dish, expected_dish, "The ingredients were combined incorrectly!");
    }

    #[test]
    fn test_different_ingredients_are_not_equal() {
        let ingredient1 = "Tomato";
        let ingredient2 = "Potato";

        // `assert_ne!` checks if two values are NOT equal.
        assert_ne!(ingredient1, ingredient2, "These ingredients should not be the same!");
    }

    println!("   💡 `assert!(expression)`: Checks for `true`.");
    println!("   💡 `assert_eq!(left, right)`: Checks for equality.");
    println!("   💡 `assert_ne!(left, right)`: Checks for inequality.");
    println!("   💡 All assertion macros accept an optional custom failure message.");

    // Example 3: Organizing Tests in a `tests` Module
    println!("\n3️⃣ Organizing Tests with `mod tests` - The Quality Control Department:");

    println!("   📁 It's a common convention to place unit tests in a `tests` module inside the same file as the code they are testing.");
    println!("   ``` ");
    println!("   // Your main code (e.g., in src/lib.rs or src/main.rs)");
    println!("   pub fn add(a: i32, b: i32) -> i32 { a + b }");
    println!("");
    println!("   // A dedicated module for tests");
    println!("   #[cfg(test)]");
    println!("   mod tests {");
    println!("       // Import the code you want to test");
    println!("       use super::*;");
    println!("");
    println!("       #[test]");
    println!("       fn test_addition() {");
    println!("           assert_eq!(add(2, 2), 4);");
    println!("       }");
    println!("   }");
    println!("   ```");
    println!("   💡 `#[cfg(test)]` ensures the test module is only compiled when you run `cargo test`, keeping your final binary clean [^240].");
    println!("   💡 `use super::*;` brings all items from the parent module into the test module's scope.");

    // Example 4: Testing for Panics
    println!("\n4️⃣ Testing for Panics - Ensuring Safety Systems Work:");

    // Function that should panic under certain conditions
    fn serve_dish(dish_name: &str) {
        if dish_name == "Forbidden Dish" {
            panic!("Cannot serve the Forbidden Dish!");
        }
        println!("Serving {}", dish_name);
    }

    #[test]
    #[should_panic] // This attribute expects the test to panic
    fn test_serving_forbidden_dish_panics() {
        serve_dish("Forbidden Dish");
    }

    #[test]
    #[should_panic(expected = "out of ingredients")] // Optional: Check for specific panic message
    fn test_specific_panic_message() {
        panic!("The kitchen is out of ingredients!");
    }

    println!("   💡 `#[should_panic]` makes a test pass only if the code inside it panics.");
    println!("   💡 You can also check for a specific panic message with `expected = \"...\"`.");

    // Example 5: Using `Result<T, E>` in Tests
    println!("\n5️⃣ Using `Result<T, E>` in Tests - Checking for Expected Success or Failure:");

    // A function that can either succeed or fail
    fn prepare_complex_dish(ingredient_count: u32) -> Result<String, String> {
        if ingredient_count < 5 {
            Err("Not enough ingredients to prepare the complex dish.".to_string())
        } else {
            Ok("Complex dish prepared successfully.".to_string())
        }
    }

    #[test]
    fn test_complex_dish_preparation_succeeds() -> Result<(), String> {
        let result = prepare_complex_dish(10)?; // The `?` operator will propagate an `Err` and fail the test
        assert_eq!(result, "Complex dish prepared successfully.");
        Ok(()) // Return `Ok(())` to indicate success
    }

    #[test]
    fn test_complex_dish_preparation_fails() {
        let result = prepare_complex_dish(3);
        assert!(result.is_err()); // Check that the result is an error
    }

    println!("   💡 You can return a `Result<T, E>` from a test function.");
    println!("   💡 `Ok(())` indicates the test passed.");
    println!("   💡 An `Err` value indicates the test failed.");
    println!("   💡 This is useful for testing functions that return `Result` types.");

    println!("\n🎯 Test Function Fundamentals Summary:");
    println!("   • `#[test]`: Marks a function as a test.");
    println!("   • `mod tests { ... }`: A conventional way to organize tests.");
    println!("   • Assertion Macros (`assert!`, `assert_eq!`, etc.): Verify outcomes.");
    println!("   • `#[should_panic]`: Tests for expected panics.");
    println!("   • `Result<T, E>`: Tests can return `Result` for more expressive failure cases.");
}

fn main() {
    demonstrate_test_function_fundamentals();
}
```


## How to Write and Run Tests: A Practical Walkthrough

### From the Chef's Recipe to the Quality Control Checklist

**A step-by-step guide to creating and running tests in a Rust project:**

```rust
fn demonstrate_writing_and_running_tests() {
    println!("🚀 How to Write and Run Tests - From Recipe to Quality Control");
    println!("{:=<70}", "");

    println!("📋 Step-by-Step Guide to Writing Tests:");

    // Step 1: Create a New Library Project
    println!("\n1️⃣ Step 1: Create a New Library Project with `cargo`");
    println!("   In your terminal, run:");
    println!("   ``` ");
    println!("   cargo new restaurant_lib --lib");
    println!("   cd restaurant_lib");
    println!("   ```");
    println!("   💡 Cargo automatically creates a `src/lib.rs` file with a default test module.");

    // Step 2: Write the Function to be Tested
    println!("\n2️⃣ Step 2: Write the Business Logic to be Tested");
    println!("   Let's create a simple `restaurant` module in `src/lib.rs`:");
    println!("   ``` ");
    println!("   // In src/lib.rs");
    println!("   pub mod restaurant {");
    println!("       #[derive(Debug, PartialEq)]");
    println!("       pub struct Order {");
    println!("           pub id: u32,");
    println!("           pub item: String,");
    println!("           pub quantity: u32,");
    println!("       }");
    println!("");
    println!("       pub fn create_order(id: u32, item: String, quantity: u32) -> Order {");
    println!("           if quantity == 0 {");
    println!("               panic!(\"Cannot create an order with zero quantity!\");");
    println!("           }");
    println!("           Order { id, item, quantity }");
    println!("       }");
    println!("");
    println!("       pub fn is_large_order(order: &Order) -> bool {");
    println!("           order.quantity > 10");
    println!("       }");
    println!("   }");
    println!("   ```");

    // Step 3: Write the Test Functions
    println!("\n3️⃣ Step 3: Write the Test Functions in a `tests` Module");
    println!("   Add this `tests` module to the bottom of `src/lib.rs`:");
    println!("   ``` ");
    println!("   // At the bottom of src/lib.rs");
    println!("   #[cfg(test)]");
    println!("   mod tests {");
    println!("       // Import the necessary items from our library");
    println!("       use super::restaurant::{create_order, is_large_order, Order};");
    println!("");
    println!("       // Test case 1: Test successful order creation");
    println!("       #[test]");
    println!("       fn test_create_order_success() {");
    println!("           let order = create_order(1, \"Pizza\".to_string(), 2);");
    println!("           assert_eq!(order.id, 1);");
    println!("           assert_eq!(order.item, \"Pizza\");");
    println!("           assert_eq!(order.quantity, 2);");
    println!("       }");
    println!("");
    println!("       // Test case 2: Test if a small order is correctly identified");
    println!("       #[test]");
    println!("       fn test_small_order_is_not_large() {");
    println!("           let small_order = Order { id: 2, item: \"Salad\".to_string(), quantity: 3 };");
    println!("           assert!(!is_large_order(&small_order));");
    println!("       }");
    println!("");
    println!("       // Test case 3: Test if a large order is correctly identified");
    println!("       #[test]");
    println!("       fn test_large_order_is_large() {");
    println!("           let large_order = Order { id: 3, item: \"Fries\".to_string(), quantity: 15 };");
    println!("           assert!(is_large_order(&large_order));");
    println!("       }");
    println!("");
    println!("       // Test case 4: Test that creating an order with zero quantity panics");
    println!("       #[test]");
    println!("       #[should_panic(expected = \"Cannot create an order with zero quantity!\")]");
    println!("       fn test_create_order_with_zero_quantity_panics() {");
    println!("           create_order(4, \"Coke\".to_string(), 0);");
    println!("       }");
    println!("   }");
    println!("   ```");

    // Step 4: Run the Tests
    println!("\n4️⃣ Step 4: Run the Tests Using `cargo test`");
    println!("   In your terminal, from the `restaurant_lib` directory, run:");
    println!("   ``` ");
    println!("   cargo test");
    println!("   ```");

    // Example 5: Understanding the Test Output
    println!("\n5️⃣ Step 5: Understand the Test Output");
    println!("   You will see output similar to this:");
    println!("   ``` ");
    println!("   running 4 tests");
    println!("   test tests::test_create_order_success ... ok");
    println!("   test tests::test_large_order_is_large ... ok");
    println!("   test tests::test_small_order_is_not_large ... ok");
    println!("   test tests::test_create_order_with_zero_quantity_panics ... ok");
    println!("");
    println!("   test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s");
    println!("   ```");
    println!("   💡 Each `... ok` line indicates a passed test.");
    println!("   💡 The summary at the end provides an overview of the test run.");

    // Example 6: Controlling Test Execution
    println!("\n6️⃣ Step 6: Control Which Tests Are Run");
    println!("   You can run specific tests or tests that match a pattern:");
    println!("   ``` ");
    println!("   # Run only tests with 'large' in their name");
    println!("   cargo test large");
    println!("");
    println!("   # Run a single specific test by its full path");
    println!("   cargo test tests::test_create_order_success");
    println!("");
    println!("   # Run tests in a single thread for sequential execution");
    println!("   cargo test -- --test-threads=1");
    println!("");
    println!("   # Show output from passed tests (e.g., from `println!`)");
    println!("   cargo test -- --show-output");
    println!("   ```");
    println!("   💡 These command-line flags give you fine-grained control over your test runs.");

    println!("\n🎯 Practical Testing Workflow Summary:");
    println!("   • Write your business logic in a library or application.");
    println!("   • Create a `#[cfg(test)] mod tests` module.");
    println!("   • Write test functions using `#[test]` and assertion macros.");
    println!("   • Run `cargo test` to execute your tests and verify your code.");
    println!("   • Use command-line flags to filter and control test execution.");
}

fn main() {
    demonstrate_writing_and_running_tests();
}
```


## Real-World Testing Strategies and Organization

### From Unit Tests to Integration Tests

**Organizing your tests effectively is key to maintaining a robust and scalable project:**

```rust
fn demonstrate_testing_strategies_and_organization() {
    println!("🗂️ Testing Strategies and Organization - The Complete Quality Control Program");
    println!("{:=<75}", "");

    println!("📋 Types of Tests in a Rust Project:");
    println!("   • **Unit Tests**: Test individual components in isolation. Placed in `src/` files [^238].");
    println!("   • **Integration Tests**: Test how different parts of your library work together. Placed in a separate `tests/` directory [^3].");
    println!("   • **Documentation Tests**: Test the code examples in your documentation [^3].");

    // Strategy 1: Unit Tests
    println!("\n1️⃣ Unit Tests - Checking Individual Ingredients and Recipes:");
    println!("   • **Location**: In the same file as the code they are testing, inside a `#[cfg(test)] mod tests` block.");
    println!("   • **Purpose**: To test private functions and the internal logic of a module.");
    println!("   • **Example**: (As shown in the previous section) Testing `create_order` and `is_large_order` in `src/lib.rs`.");
    println!("   💡 Unit tests are for fine-grained, focused testing of individual components.");

    // Strategy 2: Integration Tests
    println!("\n2️⃣ Integration Tests - Testing the Full Dining Experience:");
    println!("   • **Location**: In a `tests` directory at the root of your project (next to `src/`).");
    println!("   • **Purpose**: To test the public API of your library as a user would.");
    println!("   • **File Structure**:");
    println!("   ``` ");
    println!("   restaurant_lib/");
    println!("   ├── Cargo.toml");
    println!("   ├── src/");
    println!("   │   └── lib.rs");
    println!("   └── tests/");
    println!("       └── order_integration_tests.rs");
    println!("   ```");
    println!("   • **Example Integration Test (`tests/order_integration_tests.rs`):**");
    println!("   ``` ");
    println!("   // In tests/order_integration_tests.rs");
    println!("");
    println!("   // Use the public API of your crate");
    println!("   use restaurant_lib::restaurant::{create_order, is_large_order};");
    println!("");
    println!("   #[test]");
    println!("   fn test_full_order_workflow() {");
    println!("       // Create an order using the public API");
    println!("       let order = create_order(101, \"Full Meal\".to_string(), 12);");
    println!("");
    println!("       // Check its properties");
    println!("       assert_eq!(order.id, 101);");
    println!("       assert_eq!(order.item, \"Full Meal\");");
    println!("");
    println!("       // Use another public function to test its state");
    println!("       assert!(is_large_order(&order));");
    println!("   }");
    println!("   ```");
    println!("   💡 Integration tests ensure that different parts of your library work together correctly and that your public API is solid.");

    // Strategy 3: Documentation Tests
    println!("\n3️⃣ Documentation Tests - Ensuring Your Menu Descriptions Are Accurate:");
    println!("   • **Location**: Inside documentation comments (`///` or `/** ... */`) in your `src/` files.");
    println!("   • **Purpose**: To provide examples of how to use your code and to ensure those examples are always correct.");
    println!("   • **Example Documentation Test (in `src/lib.rs`):**");
    println!("   ``` ");
    println!("   /// Creates a new order with the given details.");
    println!("   ///");
    println!("   /// # Examples");
    println!("   ///");
    println!("   /// ```");
    println!("   /// use restaurant_lib::restaurant::create_order;");
    println!("   ///");
    println!("   /// let order = create_order(1, \"Pizza\".to_string(), 2);");
    println!("   /// assert_eq!(order.id, 1);");
    println!("   /// ``` ");
    println!("   pub fn create_order(id: u32, item: String, quantity: u32) -> Order {");
    println!("       // ... implementation ...");
    println!("   }");
    println!("   ``` ");
    println!("   💡 `cargo test` will automatically run the code inside ``` ");
    println!("   💡 This keeps your examples up-to-date and provides excellent, verifiable documentation.");

    // Best Practices for Organizing Tests
    println!("\n📋 Best Practices for Test Organization:");
    println!("   -  **Use Unit Tests** for testing internal logic, private functions, and edge cases of individual components.");
    println!("   -  **Use Integration Tests** for testing the public API and the interaction between different modules.");
    println!("   -  **Use Documentation Tests** to provide clear, runnable examples for users of your library.");
    println!("   -  **Create Helper Functions** within your `tests` module to reduce code duplication in your tests.");
    println!("   -  **Keep Tests Focused**: Each test should ideally verify one specific piece of behavior.");
    println!("   -  **Descriptive Test Names**: Name your tests clearly to indicate what they are testing (e.g., `test_function_name_when_condition_then_behavior`).");

    println!("\n🎯 Testing Strategy Summary:");
    println!("   -  A combination of unit, integration, and documentation tests provides a comprehensive quality control program.");
    println!("   -  Unit tests check the small parts, integration tests check the whole system, and documentation tests ensure your instructions are correct.");
    println!("   -  Rust's built-in framework makes it easy to write and manage all three types of tests seamlessly.");
}

fn main() {
    demonstrate_testing_strategies_and_organization();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Kitchen Quality Control System**

Remember our comprehensive professional kitchen quality control analogy:

- 🧪 **Writing tests** = **Implementing a quality control system** - Systematic procedures to ensure excellence.
- 🎯 **Test functions** = **Individual quality checks** - Specific tests for dishes, ingredients, or processes.
- 🚀 **`cargo test`** = **Running the quality control program** - Executing all checks automatically.
- 🗂️ **Test organization** = **Structuring the quality program** - Separating unit, integration, and documentation checks.
- 🛡️ **Reliability** = **Customer satisfaction** - Ensuring a high-quality, bug-free final product.


### **How to Write a Test Function in Rust**

1. **The `#[test]` Attribute**: Mark any function with `#[test]` to turn it into a test case. Rust's test runner will execute this function when you run `cargo test`.
2. **The `tests` Module Convention**: It's a common practice to place your unit tests inside a module named `tests` at the bottom of the file you are testing. This module should be annotated with `#[cfg(test)]` to ensure it is only compiled when testing.

```rust
// In src/lib.rs
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*; // Import items from the parent module

    #[test]
    fn test_addition() {
        assert_eq!(add(2, 2), 4);
    }
}
```

3. **Assertion Macros**: Use these macros to verify that your code behaves as expected. A test fails if an assertion macro panics.
    * `assert!(expression, "optional message")`: Panics if `expression` is `false`.
    * `assert_eq!(left, right, "optional message")`: Panics if `left` is not equal to `right`.
    * `assert_ne!(left, right, "optional message")`: Panics if `left` is equal to `right`.
4. **Testing for Panics**: Use the `#[should_panic]` attribute to assert that a piece of code should panic.

```rust
#[test]
#[should_panic(expected = "invalid input")]
fn test_function_panics_on_invalid_input() {
    my_function_that_panics("invalid");
}
```

5. **Using `Result<T, E>` in Tests**: Test functions can return a `Result<T, E>`, which allows you to use the `?` operator for more expressive tests. An `Ok(())` indicates success, while an `Err` indicates failure.

### **Running Your Tests**

- **Run all tests**:

```
cargo test
```

- **Run tests that match a name**:

```
cargo test my_test_name
```

- **Run tests sequentially**:

```
cargo test -- --test-threads=1
```

- **Show output from passed tests**:

```
cargo test -- --show-output
```


### **Types of Tests and Their Organization**

| **Test Type** | **Location** | **Purpose** |
| :-- | :-- | :-- |
| **Unit Tests** | In `src/` files, inside `mod tests { ... }` | Test individual components, including private functions. |
| **Integration Tests** | In the `tests/` directory at the project root | Test the public API of your library as a whole. |
| **Documentation Tests** | In documentation comments (`///`) in `src/` files | Ensure code examples in your documentation are correct. |

### **Best Practices Checklist**

**✅ Writing Effective Tests:**

- Each test should focus on a single piece of functionality.
- Use descriptive test names that explain what is being tested.
- Write tests for both the "happy path" (expected inputs) and edge cases (invalid inputs, boundary conditions).
- Use helper functions within your `tests` module to avoid code duplication.

**✅ Organization:**

- Use unit tests for internal logic and integration tests for your public API.
- Keep your documentation examples up-to-date and testable.
- For larger projects, consider organizing integration tests into multiple files within the `tests/` directory.

**❌ Common Pitfalls:**

- Writing tests that are too complex or test too many things at once.
- Not testing edge cases or error conditions.
- Letting documentation examples become outdated.
- Not separating unit and integration tests, leading to confusion about what is being tested.


### **The Professional Advantage**

**Mastering Rust's built-in testing framework is like having a complete professional kitchen quality control system** that ensures every part of your application meets the highest standards of quality and reliability:

- 🛡️ **Robust Code**: A comprehensive test suite provides confidence that your code is correct and reliable.
- 🔄 **Safe Refactoring**: Tests allow you to make changes to your code with the assurance that you haven't broken existing functionality.
- 📋 **Living Documentation**: Your tests serve as practical, verifiable examples of how your code should be used.
- ⚡ **Efficient Development**: Automated tests catch bugs early in the development process, saving time and effort.
- 🏆 **Professional Quality**: A well-tested codebase is a hallmark of professional software engineering.

**Understanding how to write and organize tests in Rust transforms you from a programmer who writes code that works to an engineer** who builds robust, reliable, and maintainable software. Just as a master chef ensures every dish is perfect before it leaves the kitchen, a skilled Rust programmer uses testing to ensure every line of code is correct before it ships to production.

This comprehensive understanding of writing test functions in Rust - from basic assertions to advanced organization strategies - empowers you to build high-quality software with the confidence that comes from a thorough and automated testing practice
1. https://zerotomastery.io/blog/complete-guide-to-testing-code-in-rust/
2. https://doc.rust-lang.org/book/ch11-00-testing.html
3. https://reintech.io/blog/rusts-test-framework-comprehensive-guide
4. https://doc.rust-lang.org/book/ch11-01-writing-tests.html
5. https://www.c-sharpcorner.com/article/how-to-write-tests-in-rust/
6. https://dev.to/tramposo/testing-in-rust-a-quick-guide-to-unit-tests-integration-tests-and-benchmarks-2bah
7. https://www.youtube.com/watch?v=UyJ1mEqKMvE
8. https://www.swiftorial.com/tutorials/programming_languages/rust/testing/introduction_to_testing/
9. https://rust-cli.github.io/book/tutorial/testing.html
