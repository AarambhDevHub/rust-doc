+++
title = "Functional Error Handling"
description = "Learn functional approaches to error handling in Rust using combinators like map, and_then, and unwrap_or."
date = "2025-08-13"
weight = 4
+++

# Functional Error Handling in Rust: Comprehensive Documentation for Beginners

Understanding functional error handling in Rust is like learning to **design an ultra-reliable, data-driven kitchen management system** - instead of relying on unpredictable emergencies (exceptions) that halt operations, you build a system where every potential problem is anticipated, represented explicitly in the data flow, and handled methodically as part of the normal operation. Just as a master restaurant manager handles potential issues (like ingredients not arriving, equipment failure, or a dish not turning out right) by having clear contingency plans and alternative pathways built into the workflow, Rust's functional error handling, primarily through the `Result` enum and the `?` operator, makes error handling explicit, type-safe, and composable.

## The Professional Kitchen Management System Analogy 🍳

### Imagine You're Designing an Ultra-Reliable, Data-Driven Kitchen

**The Problem with Unpredictable Emergencies (Exceptions):**

```rust
// ❌ Exception-based approach - like relying on emergency "panic!" buttons
fn prepare_dish_risky() {
    // If ingredient missing, "panic!" button pressed, operations halt!
    // Chef throws hands up, service stops, customers leave!
    panic!("Ingredient not found!"); // Unrecoverable, stops everything
}
```

**The Power of Functional Error Handling - Anticipated Problems, Clear Pathways:**

```rust
// ✅ Functional approach - anticipate problems, provide clear pathways
#[derive(Debug)]
enum IngredientProblem {
    NotFound(String),
    Expired(String),
    Contaminated(String),
}

// Function returns potential success OR potential problem explicitly
fn get_ingredient(name: &str) -> Result<String, IngredientProblem> {
    match name {
        "fresh_tomatoes" => Ok("Perfect quality tomatoes".to_string()),
        "expired_milk" => Err(IngredientProblem::Expired("Milk past its date".to_string())),
        "missing_basil" => Err(IngredientProblem::NotFound("Basil not in stock".to_string())),
        _ => Ok(format!("{} obtained", name)),
    }
}

// Chef's workflow handles both success and problem explicitly
fn prepare_salad() -> Result<String, IngredientProblem> {
    let tomatoes = get_ingredient("fresh_tomatoes")?; // Success → continue; Problem → propagate
    let basil = get_ingredient("missing_basil")?; // This will fail and propagate

    // This code only runs if all ingredients are successfully obtained
    Ok(format!("Salad prepared with {} and {}", tomatoes, basil))
}

fn kitchen_management_demo() {
    match prepare_salad() {
        Ok(dish) => println!("🍽️ Successfully prepared: {}", dish),
        Err(problem) => println!("🚨 Problem preparing salad: {:?}", problem),
    }
}
```

**Why Functional Error Handling Is Revolutionary:**

- 🛡️ **Explicitness** - Compiler forces you to acknowledge all possible outcomes
- 🎯 **Type safety** - Errors are part of the function signature, checked at compile time
- 🔄 **Composability** - Easily chain operations that might fail without nested `if` statements
- 📈 **Predictability** - No hidden control flow jumps, errors are data
- ⚡ **Zero runtime overhead** - No exceptions or stack unwinding unless explicit `panic!`


## Understanding `Result<T, E>`

### The Dual-Outcome Data Container

`Result<T, E>` is an enum that represents either success (`Ok`) or failure (`Err`):

```rust
fn demonstrate_result_enum() {
    println!("📦 `Result<T, E>` Enum - The Dual-Outcome Data Container");
    println!("{:=<70}", "");

    use std::io::{self, Read};
    use std::fs::File;

    // Result is like a special dish container: either it has the dish (Ok), or a problem report (Err)
    println!("📋 `Result` Enum Structure:");
    println!("   `enum Result<T, E> { Ok(T), Err(E), }`");
    println!("   • `T` represents the type of the successful value.");
    println!("   • `E` represents the type of the error value.");

    // Example 1: Basic `Result` Usage - File Operations
    println!("\n1️⃣ Basic `Result` Usage - File Operations:");

    fn open_and_read_file(filename: &str) -> Result<String, io::Error> {
        let mut file = File::open(filename)?; // Try to open file
        let mut contents = String::new();
        file.read_to_string(&mut contents)?; // Try to read contents
        Ok(contents) // Return success
    }

    let existing_file_name = "existing_recipe.txt";
    let non_existing_file_name = "non_existent_recipe.txt";

    // Create a dummy file for demonstration
    std::fs::write(existing_file_name, "Spicy Vegan Chili Recipe").unwrap();

    // Test with existing file
    match open_and_read_file(existing_file_name) {
        Ok(contents) => println!("   ✅ File contents: '{}'", contents),
        Err(error) => println!("   ❌ Error reading file: {}", error),
    }

    // Test with non-existing file
    match open_and_read_file(non_existing_file_name) {
        Ok(contents) => println!("   ✅ File contents: '{}'", contents),
        Err(error) => println!("   ❌ Error reading file: {}", error),
    }

    // Example 2: Common `Result` Methods - Unpacking the Dish Container
    println!("\n2️⃣ Common `Result` Methods - Unpacking the Dish Container:");

    fn get_chef_status(chef_id: u32) -> Result<String, String> {
        if chef_id == 1 {
            Ok("Chef Alice - Available".to_string())
        } else {
            Err("Chef not found".to_string())
        }
    }

    // `unwrap()`: Panic on Err
    // Use only when you are absolutely certain the operation won't fail (e.g., tests)
    let status_unwrap = get_chef_status(1).unwrap();
    println!("   `unwrap()`: {}", status_unwrap);

    // `expect()`: Panic on Err with custom message
    let status_expect = get_chef_status(1).expect("Chef should always be available!");
    println!("   `expect()`: {}", status_expect);
    // get_chef_status(2).expect("Chef should always be available!"); // This would panic!

    // `is_ok()` / `is_err()`: Check outcome without unwrapping
    let status_check = get_chef_status(3);
    println!("   `is_ok()`: {}", status_check.is_ok());
    println!("   `is_err()`: {}", status_check.is_err());

    // `ok()` / `err()`: Convert to Option
    let status_option = get_chef_status(1).ok();
    println!("   `ok()`: {:?}", status_option);

    // `unwrap_or()`: Provide a default value on Err
    let status_default = get_chef_status(2).unwrap_or("Chef Bob - Not Available".to_string());
    println!("   `unwrap_or()`: {}", status_default);

    // `map()`: Transform Ok value
    let dish_map = Ok("Spaghetti".to_string())
        .map(|s| format!("Prepared {}", s));
    println!("   `map()`: {:?}", dish_map);

    // `map_err()`: Transform Err value
    let problem_map = Err("No pasta".to_string())
        .map_err(|e| format!("Urgent: {}", e));
    println!("   `map_err()`: {:?}", problem_map);

    // `and_then()`: Chain operations that return Result
    fn prepare_ingredient(name: &str) -> Result<String, String> {
        if name == "garlic" { Ok("Peeled garlic".to_string()) } else { Err("Ingredient not found".to_string()) }
    }

    let garlic_result = prepare_ingredient("garlic")
        .and_then(|g| Ok(format!("Minced {}", g)));
    println!("   `and_then()`: {:?}", garlic_result);

    let onion_result = prepare_ingredient("onion")
        .and_then(|o| Ok(format!("Chopped {}", o)));
    println!("   `and_then()`: {:?}", onion_result);

    println!("\n🎯 `Result` Key Features:");
    println!("   🛡️ Explicitly represents success or failure");
    println!("   📝 Forces caller to handle both outcomes");
    println!("   🔄 Enables powerful functional composition");
    println!("   ⚡ Zero runtime overhead for error handling");
    println!("   📦 Used extensively in Rust's standard library");
}

fn main() {
    demonstrate_result_enum();
}
```


## The `?` Operator - The Error Propagation System

The `?` operator is a concise way to propagate `Result` errors up the call stack:

```rust
fn demonstrate_question_mark_operator() {
    println!("❓ The `?` Operator - The Error Propagation System");
    println!("{:=<70}", "");

    use std::io::{self, Read};
    use std::fs::File;
    use std::path::Path;

    // The `?` operator is like a smart assistant who escalates problems automatically
    println!("📋 `?` Operator Functionality:");
    println!("   ✅ Extracts `Ok` value and continues execution");
    println!("   ❌ Returns `Err` value from the function immediately");
    println!("   🔄 Automates error propagation up the call stack");
    println!("   📝 Reduces boilerplate `match` statements");

    // Example 1: Basic `?` Operator Usage
    println!("\n1️⃣ Basic `?` Operator Usage - Automated Problem Escalation:");

    // Function that reads file contents using `?`
    fn read_file_contents(filename: &str) -> Result<String, io::Error> {
        let mut file = File::open(filename)?; // If Err, return Err immediately
        let mut contents = String::new();
        file.read_to_string(&mut contents)?; // If Err, return Err immediately
        Ok(contents) // If Ok, return contents
    }

    // Test with existing and non-existing files
    std::fs::write("daily_report.txt", "Today's Special: Pasta").unwrap();

    match read_file_contents("daily_report.txt") {
        Ok(contents) => println!("   ✅ Report contents: '{}'", contents),
        Err(e) => println!("   ❌ Error reading report: {}", e),
    }

    match read_file_contents("missing_report.txt") {
        Ok(contents) => println!("   ✅ Report contents: '{}'", contents),
        Err(e) => println!("   ❌ Error reading report: {}", e),
    }

    println!("   💡 `?` eliminates tedious `match` blocks for error propagation.");

    // Example 2: `?` Operator with Custom Error Types
    println!("\n2️⃣ `?` Operator with Custom Error Types - Structured Problem Reports:");

    // Custom error types for better error reporting
    #[derive(Debug)]
    enum RecipeError {
        FileNotFound(String),
        InvalidFormat(String),
        IngredientMissing(String),
        CookingFailure(String),
    }

    // Implement `From` trait for automatic conversion
    impl From<io::Error> for RecipeError {
        fn from(error: io::Error) -> Self {
            RecipeError::FileNotFound(error.to_string())
        }
    }

    fn load_recipe_from_disk(recipe_name: &str) -> Result<String, RecipeError> {
        let path = Path::new(&recipe_name).with_extension("recipe");
        let mut file = File::open(&path)?; // `io::Error` automatically converted to `RecipeError::FileNotFound`
        let mut contents = String::new();
        file.read_to_string(&mut contents)?; // Again, `io::Error` conversion
        Ok(contents)
    }

    fn parse_recipe_string(recipe_string: String) -> Result<Vec<String>, RecipeError> {
        if !recipe_string.contains("Ingredients:") {
            return Err(RecipeError::InvalidFormat("Missing 'Ingredients:' section".to_string()));
        }

        let ingredients: Vec<String> = recipe_string
            .lines()
            .filter(|line| line.starts_with("- "))
            .map(|line| line[2..].to_string())
            .collect();

        if ingredients.is_empty() {
            return Err(RecipeError::IngredientMissing("No ingredients listed".to_string()));
        }

        Ok(ingredients)
    }

    fn prepare_dish(dish_name: &str) -> Result<String, RecipeError> {
        let recipe_contents = load_recipe_from_disk(dish_name)?; // Propagates RecipeError
        let ingredients = parse_recipe_string(recipe_contents)?; // Propagates RecipeError

        if ingredients.contains(&"mystery_ingredient".to_string()) {
            return Err(RecipeError::CookingFailure("Mystery ingredient makes dish explode".to_string()));
        }

        Ok(format!("Dish '{}' prepared using ingredients: {:?}", dish_name, ingredients))
    }

    // Test recipe preparation
    std::fs::write("pasta_carbonara.recipe", "Ingredients:\n- pasta\n- eggs\n- cheese").unwrap();
    std::fs::write("exploding_dish.recipe", "Ingredients:\n- mystery_ingredient").unwrap();

    match prepare_dish("pasta_carbonara") {
        Ok(dish) => println!("   ✅ Prepared: {}", dish),
        Err(e) => println!("   ❌ Preparation problem: {:?}", e),
    }

    match prepare_dish("exploding_dish") {
        Ok(dish) => println!("   ✅ Prepared: {}", dish),
        Err(e) => println!("   ❌ Preparation problem: {:?}", e),
    }

    match prepare_dish("non_existent_dish") {
        Ok(dish) => println!("   ✅ Prepared: {}", dish),
        Err(e) => println!("   ❌ Preparation problem: {:?}", e),
    }

    println!("\n   💡 `?` operator ensures error type matches function's `Result` type.");
    println!("   💡 Implement `From<OtherError>` for your custom error to enable automatic conversion.");

    // Example 3: `?` Operator with Option<T>
    println!("\n3️⃣ `?` Operator with `Option<T>` - Handling Missing Data:");

    fn find_manager_email(employee_id: u32) -> Option<String> {
        let employee_data: HashMap<u32, (String, Option<String>)> = HashMap::from([
            (1, ("Alice".to_string(), Some("alice@example.com".to_string()))),
            (2, ("Bob".to_string(), None)), // Bob has no email
            (3, ("Charlie".to_string(), Some("charlie@example.com".to_string()))),
        ]);

        let (_, email_option) = employee_data.get(&employee_id)?; // If None, return None
        let email = email_option.clone()?; // If None, return None
        Some(email) // If Some, return email
    }

    match find_manager_email(1) {
        Some(email) => println!("   ✅ Manager email for ID 1: {}", email),
        None => println!("   ❌ Manager email not found for ID 1"),
    }

    match find_manager_email(2) {
        Some(email) => println!("   ✅ Manager email for ID 2: {}", email),
        None => println!("   ❌ Manager email not found for ID 2"),
    }

    match find_manager_email(4) {
        Some(email) => println!("   ✅ Manager email for ID 4: {}", email),
        None => println!("   ❌ Manager email not found for ID 4"),
    }

    println!("\n   💡 `?` can also be used with `Option<T>` for concise missing data propagation.");

    println!("\n🎯 `?` Operator Key Features:");
    println!("   ✅ Concise error propagation syntax");
    println!("   🛡️ Type-safe - compiler ensures `Err` type compatibility");
    println!("   🔄 Automatic error conversion via `From` trait");
    println!("   📦 Works with both `Result` and `Option`");
    println!("   🎨 Enables clean, linear error-handling pipelines");
}

fn main() {
    demonstrate_question_mark_operator();
}
```


## Functional Error Handling Patterns

### Composing Operations with Predictable Outcomes

**Leveraging `Result` and `?` with other functional constructs for robust error handling:**

```rust
fn demonstrate_functional_error_patterns() {
    println!("🎨 Functional Error Handling Patterns - Composing Operations");
    println!("{:=<70}", "");

    // Functional error handling patterns are like building a robust workflow with clear paths
    println!("📋 Functional Patterns for Error Handling:");

    // Pattern 1: Chaining Operations with `and_then()` and `map()`
    println!("\n1️⃣ Chaining Operations - Building a Fault-Tolerant Workflow:");

    #[derive(Debug)]
    enum KitchenError {
        IngredientPrep(String),
        Cooking(String),
        Plating(String),
    }

    // Simulate kitchen steps that can fail
    fn chop_vegetables_fn(veg: &str) -> Result<String, KitchenError> {
        if veg == "rotten_broccoli" {
            Err(KitchenError::IngredientPrep("Rotten broccoli found".to_string()))
        } else {
            Ok(format!("Chopped {}", veg))
        }
    }

    fn cook_dish_fn(prepped_ingredients: String) -> Result<String, KitchenError> {
        if prepped_ingredients.contains("raw_chicken") {
            Err(KitchenError::Cooking("Raw chicken cooked improperly".to_string()))
        } else {
            Ok(format!("Cooked {}", prepped_ingredients))
        }
    }

    fn plate_dish_fn(cooked_dish: String) -> Result<String, KitchenError> {
        if cooked_dish.contains("broken_plate") {
            Err(KitchenError::Plating("Broken plate used".to_string()))
        } else {
            Ok(format!("Plated {}", cooked_dish))
        }
    }

    // Chain functions using `and_then()`
    fn prepare_meal_chained(ingredient: &str) -> Result<String, KitchenError> {
        chop_vegetables_fn(ingredient)
            .and_then(|chopped| cook_dish_fn(chopped))
            .and_then(|cooked| plate_dish_fn(cooked))
    }

    println!("   Chaining operations with `and_then()`:");

    match prepare_meal_chained("carrots") {
        Ok(meal) => println!("     ✅ Meal prepared: {}", meal),
        Err(e) => println!("     ❌ Meal preparation failed: {:?}", e),
    }

    match prepare_meal_chained("rotten_broccoli") {
        Ok(meal) => println!("     ✅ Meal prepared: {}", meal),
        Err(e) => println!("     ❌ Meal preparation failed: {:?}", e),
    }

    match prepare_meal_chained("raw_chicken") { // Will fail at cook_dish_fn
        Ok(meal) => println!("     ✅ Meal prepared: {}", meal),
        Err(e) => println!("     ❌ Meal preparation failed: {:?}", e),
    }

    println!("   💡 `and_then()` continues the chain only on `Ok`, propagating `Err`.");
    println!("   💡 `map()` transforms `Ok` values, `map_err()` transforms `Err` values.");

    // Pattern 2: `or_else()` for Fallback Mechanisms
    println!("\n2️⃣ `or_else()` for Fallback Mechanisms - Contingency Planning:");

    fn get_primary_supplier_stock(item: &str) -> Result<u32, String> {
        if item == "organic_quinoa" { Ok(50) } else { Err("Primary supplier out of stock".to_string()) }
    }

    fn get_secondary_supplier_stock(item: &str) -> Result<u32, String> {
        if item == "organic_quinoa" { Ok(20) } else { Err("Secondary supplier also out of stock".to_string()) }
    }

    fn get_emergency_supplier_stock(item: &str) -> Result<u32, String> {
        if item == "organic_quinoa" { Ok(5) } else { Err("Emergency supplier failed".to_string()) }
    }

    // Try suppliers in order of preference
    fn get_stock_with_fallback(item: &str) -> Result<u32, String> {
        get_primary_supplier_stock(item)
            .or_else(|_| {
                println!("     🔄 Primary failed, trying secondary supplier for {}", item);
                get_secondary_supplier_stock(item)
            })
            .or_else(|_| {
                println!("     🚨 Secondary failed, trying emergency supplier for {}", item);
                get_emergency_supplier_stock(item)
            })
    }

    println!("   Contingency planning with `or_else()`:");

    match get_stock_with_fallback("organic_quinoa") {
        Ok(stock) => println!("     ✅ Stock found: {} units", stock),
        Err(e) => println!("     ❌ All suppliers failed: {}", e),
    }

    match get_stock_with_fallback("non_existent_item") {
        Ok(stock) => println!("     ✅ Stock found: {} units", stock),
        Err(e) => println!("     ❌ All suppliers failed: {}", e),
    }

    println!("   💡 `or_else()` provides a clean way to implement fallbacks.");

    // Pattern 3: `filter_map()` and `flat_map()` for Error-Aware Iteration
    println!("\n3️⃣ Error-Aware Iteration - Processing Batches with Failures:");

    let raw_order_lines = vec![
        "Pizza Margherita, 15.99",
        "Salad Caesar, 12.50",
        "Invalid Line, ERROR",
        "Pasta Carbonara, 18.00",
        "Missing Price",
    ];

    #[derive(Debug)]
    struct ProcessedOrderLine {
        name: String,
        price: f64,
    }

    impl std::str::FromStr for ProcessedOrderLine {
        type Err = String;
        fn from_str(s: &str) -> Result<Self, Self::Err> {
            let parts: Vec<&str> = s.split(',').collect();
            if parts.len() != 2 {
                return Err(format!("Invalid format: {}", s));
            }
            let name = parts[^0].trim().to_string();
            let price: f64 = parts[^5].trim().parse().map_err(|e| format!("Invalid price: {}", e))?;
            Ok(ProcessedOrderLine { name, price })
        }
    }

    // `filter_map()` to skip errors and collect only successful parses
    let valid_order_lines: Vec<ProcessedOrderLine> = raw_order_lines.iter()
        .filter_map(|&line| {
            line.parse::<ProcessedOrderLine>().ok() // Only keep Ok results
        })
        .collect();

    println!("   `filter_map()` to collect valid entries:");
    println!("     Valid order lines: {:?}", valid_order_lines);

    // `flat_map()` to chain operations where each item can produce a Result
    fn get_item_reviews(item_name: &str) -> Result<Vec<String>, String> {
        match item_name {
            "Pizza Margherita" => Ok(vec!["Great pizza!".to_string(), "Delicious!".to_string()]),
            "Salad Caesar" => Ok(vec!["Healthy choice.".to_string()]),
            "Pasta Carbonara" => Err("No reviews for this item.".to_string()),
            _ => Err("Item not found.".to_string()),
        }
    }

    let processed_menu = vec!["Pizza Margherita", "Salad Caesar", "Pasta Carbonara", "Burger"];

    let all_reviews: Vec<String> = processed_menu.iter()
        .flat_map(|&item| get_item_reviews(item).unwrap_or_else(|_| Vec::new())) // Get reviews, default to empty Vec on Err
        .collect();

    println!("   `flat_map()` to collect all reviews (skipping items with no reviews):");
    println!("     All collected reviews: {:?}", all_reviews);

    println!("   💡 `filter_map()` for optional results, `flat_map()` for nested results.");

    // Pattern 4: Using `Result` in Structs and Enums
    println!("\n4️⃣ `Result` in Data Structures - Structured Problem Reporting:");

    #[derive(Debug)]
    enum DishPreparationResult {
        Success(String),
        Failure(String),
    }

    #[derive(Debug)]
    struct DailyKitchenReport {
        date: String,
        prepared_dishes: Vec<DishPreparationResult>,
        total_successful: u32,
        total_failed: u32,
    }

    fn generate_daily_report(date: &str) -> DailyKitchenReport {
        let mut report = DailyKitchenReport {
            date: date.to_string(),
            prepared_dishes: Vec::new(),
            total_successful: 0,
            total_failed: 0,
        };

        let dish_attempts = vec![
            ("Pizza", Ok("Perfectly baked pizza".to_string())),
            ("Salad", Ok("Freshly chopped salad".to_string())),
            ("Pasta", Err("Pasta burnt to a crisp".to_string())),
            ("Soup", Ok("Rich and creamy soup".to_string())),
            ("Steak", Err("Steak undercooked".to_string())),
        ];

        for (dish_name, result) in dish_attempts {
            let prepared_result = match result {
                Ok(msg) => {
                    report.total_successful += 1;
                    DishPreparationResult::Success(format!("{}: {}", dish_name, msg))
                }
                Err(msg) => {
                    report.total_failed += 1;
                    DishPreparationResult::Failure(format!("{}: {}", dish_name, msg))
                }
            };
            report.prepared_dishes.push(prepared_result);
        }

        report
    }

    let report = generate_daily_report("2024-01-15");
    println!("   Daily Kitchen Report: {:#?}", report);
    println!("   Total successful dishes: {}", report.total_successful);
    println!("   Total failed dishes: {}", report.total_failed);

    println!("\n🎯 Functional Pattern Benefits:");
    println!("   🛡️ Explicit error representation forces handling");
    println!("   🔄 Composable operations enable clean error propagation");
    println!("   🎨 Expressive code makes error handling readable");
    println!("   📈 Efficient processing of collections with potential failures");
    println!("   📦 `Result` in data structures for structured problem reporting");
}

fn main() {
    demonstrate_functional_error_patterns();
}
```


## Real-World Functional Error Handling Applications

### Complete Restaurant Management System Implementation

**Practical examples showing functional error handling in real applications:**

```rust
fn demonstrate_real_world_functional_error_handling() {
    println!("🏢 Real-World Functional Error Handling - Restaurant Management Systems");
    println!("{:=<75}", "");

    use std::collections::HashMap;
    use std::io;
    use std::fs;
    use std::path::PathBuf;

    // Real-world applications demonstrate functional error handling in complete systems
    println!("💼 Professional Functional Error Handling Applications:");

    // Application 1: Order Processing with Detailed Error Reporting
    println!("\n1️⃣ Order Processing with Detailed Error Reporting:");

    #[derive(Debug)]
    enum OrderProcessError {
        ItemNotFound(String),
        InsufficientStock { item: String, requested: u32, available: u32 },
        PaymentFailed(String),
        DatabaseError(String),
        InvalidInput(String),
    }

    impl From<io::Error> for OrderProcessError {
        fn from(error: io::Error) -> Self {
            OrderProcessError::DatabaseError(error.to_string())
        }
    }

    // Simulate database lookup
    fn get_item_price(item_name: &str) -> Result<f64, OrderProcessError> {
        let prices: HashMap<&str, f64> = HashMap::from([
            ("Pizza".to_string(), 15.99),
            ("Salad".to_string(), 10.00),
            ("Pasta".to_string(), 12.50),
        ]);
        prices.get(item_name).cloned().ok_or(OrderProcessError::ItemNotFound(item_name.to_string()))
    }

    // Simulate inventory check
    fn check_inventory(item_name: &str, quantity: u32) -> Result<(), OrderProcessError> {
        let stock: HashMap<&str, u32> = HashMap::from([
            ("Pizza".to_string(), 10),
            ("Salad".to_string(), 5),
            ("Pasta".to_string(), 2), // Low stock
        ]);

        let available = stock.get(item_name).cloned().ok_or(OrderProcessError::ItemNotFound(item_name.to_string()))?;

        if available < quantity {
            Err(OrderProcessError::InsufficientStock { item: item_name.to_string(), requested: quantity, available })
        } else {
            Ok(())
        }
    }

    // Simulate payment gateway
    fn process_payment(amount: f64) -> Result<String, OrderProcessError> {
        if amount > 100.0 {
            Err(OrderProcessError::PaymentFailed("Amount exceeds limit".to_string()))
        } else if amount < 0.0 {
            Err(OrderProcessError::InvalidInput("Negative amount".to_string()))
        } else {
            Ok(format!("Payment of ${:.2} processed", amount))
        }
    }

    // Process an order using `?` and `Result` combinators
    fn place_new_order(customer_id: u32, item_name: &str, quantity: u32) -> Result<String, OrderProcessError> {
        println!("   --- Placing order for Customer {} - {} x {} ---", customer_id, quantity, item_name);

        // Step 1: Get item price
        let item_price = get_item_price(item_name)?;

        // Step 2: Check inventory
        check_inventory(item_name, quantity)?;

        // Step 3: Calculate total and process payment
        let total_amount = item_price * quantity as f64;
        let payment_confirmation = process_payment(total_amount)?;

        // Step 4: Simulate saving order to DB
        fs::write(format!("order_{}.txt", customer_id), format!("Order: {} x {}", item_name, quantity))?; // Converts io::Error

        Ok(format!("Order for {} x {} placed successfully! {}", quantity, item_name, payment_confirmation))
    }

    // Test various order scenarios
    match place_new_order(1, "Pizza", 1) {
        Ok(conf) => println!("   ✅ Success: {}", conf),
        Err(err) => println!("   ❌ Failed: {:?}", err),
    }

    match place_new_order(2, "Salad", 10) { // Insufficient stock
        Ok(conf) => println!("   ✅ Success: {}", conf),
        Err(err) => println!("   ❌ Failed: {:?}", err),
    }

    match place_new_order(3, "Truffle Pasta", 1) { // Item not found
        Ok(conf) => println!("   ✅ Success: {}", conf),
        Err(err) => println!("   ❌ Failed: {:?}", err),
    }

    match place_new_order(4, "Pasta", 10) { // Amount exceeds limit
        Ok(conf) => println!("   ✅ Success: {}", conf),
        Err(err) => println!("   ❌ Failed: {:?}", err),
    }

    // Application 2: Configuration Loading and Validation
    println!("\n2️⃣ Configuration Loading and Validation:");

    #[derive(Debug)]
    enum ConfigError {
        Io(io::Error),
        Parse(String),
        Validation(String),
    }

    impl From<io::Error> for ConfigError {
        fn from(error: io::Error) -> Self {
            ConfigError::Io(error)
        }
    }

    struct RestaurantConfig {
        name: String,
        max_tables: u32,
        open_hours: (u32, u32),
        features: HashSet<String>,
    }

    // Load config from a pseudo-file
    fn load_config_from_file(path: &PathBuf) -> Result<String, ConfigError> {
        let contents = fs::read_to_string(path)?;
        Ok(contents)
    }

    // Parse config string
    fn parse_config_string(config_str: String) -> Result<RestaurantConfig, ConfigError> {
        let mut config_map = HashMap::new();
        for line in config_str.lines() {
            let parts: Vec<&str> = line.split('=').map(|s| s.trim()).collect();
            if parts.len() == 2 {
                config_map.insert(parts[^0].to_string(), parts[^1].to_string());
            } else if !line.is_empty() {
                return Err(ConfigError::Parse(format!("Invalid config line: {}", line)));
            }
        }

        let name = config_map.get("name").cloned().ok_or(ConfigError::Validation("Missing 'name'".to_string()))?;
        let max_tables_str = config_map.get("max_tables").cloned().ok_or(ConfigError::Validation("Missing 'max_tables'".to_string()))?;
        let max_tables = max_tables_str.parse::<u32>().map_err(|e| ConfigError::Parse(format!("Invalid max_tables: {}", e)))?;

        let hours_str = config_map.get("open_hours").cloned().ok_or(ConfigError::Validation("Missing 'open_hours'".to_string()))?;
        let hours_parts: Vec<u32> = hours_str.split('-').filter_map(|s| s.parse().ok()).collect();
        if hours_parts.len() != 2 {
            return Err(ConfigError::Parse("Invalid open_hours format".to_string()));
        }

        let features_str = config_map.get("features").cloned().unwrap_or_default();
        let features: HashSet<String> = features_str.split(',').filter(|s| !s.is_empty()).map(|s| s.trim().to_string()).collect();

        Ok(RestaurantConfig { name, max_tables, open_hours: (hours_parts[^0], hours_parts[^1]), features })
    }

    // Validate config logic
    fn validate_config(config: RestaurantConfig) -> Result<RestaurantConfig, ConfigError> {
        if config.max_tables == 0 {
            return Err(ConfigError::Validation("Max tables must be greater than 0".to_string()));
        }
        if config.open_hours.0 >= config.open_hours.1 {
            return Err(ConfigError::Validation("Closing hour must be after opening hour".to_string()));
        }
        Ok(config)
    }

    // Full config pipeline
    fn get_restaurant_config(path: &PathBuf) -> Result<RestaurantConfig, ConfigError> {
        load_config_from_file(path)
            .and_then(parse_config_string)
            .and_then(validate_config)
    }

    // Test config scenarios
    let valid_config_path = PathBuf::from("valid_config.txt");
    let invalid_config_path = PathBuf::from("invalid_config.txt");
    let missing_config_path = PathBuf::from("missing_config.txt");

    fs::write(&valid_config_path, "name=Bistro\nmax_tables=50\nopen_hours=9-22\nfeatures=wifi,parking").unwrap();
    fs::write(&invalid_config_path, "name=BadConfig\nmax_tables=0\nopen_hours=10-8").unwrap();

    match get_restaurant_config(&valid_config_path) {
        Ok(config) => println!("   ✅ Config loaded: {:?}", config),
        Err(err) => println!("   ❌ Config error: {:?}", err),
    }

    match get_restaurant_config(&invalid_config_path) {
        Ok(config) => println!("   ✅ Config loaded: {:?}", config),
        Err(err) => println!("   ❌ Config error: {:?}", err),
    }

    match get_restaurant_config(&missing_config_path) {
        Ok(config) => println!("   ✅ Config loaded: {:?}", config),
        Err(err) => println!("   ❌ Config error: {:?}", err),
    }

    // Application 3: Event Stream Processing
    println!("\n3️⃣ Event Stream Processing:");

    #[derive(Debug, Clone)]
    enum RestaurantEvent {
        OrderPlaced { id: u32, total: f64 },
        PaymentReceived { order_id: u32, amount: f64 },
        KitchenError { message: String },
        CustomerFeedback { rating: u8, comment: String },
    }

    // Simulate processing an event stream, handling errors functionally
    fn process_event_stream(events: Vec<RestaurantEvent>) -> Vec<Result<String, String>> {
        events.into_iter()
            .map(|event| match event {
                RestaurantEvent::OrderPlaced { id, total } => {
                    if total < 0.0 {
                        Err(format!("Invalid order total for ID {}", id))
                    } else {
                        Ok(format!("Order {} placed, total: ${:.2}", id, total))
                    }
                }
                RestaurantEvent::PaymentReceived { order_id, amount } => {
                    if amount <= 0.0 {
                        Err(format!("Invalid payment amount for order {}", order_id))
                    } else {
                        Ok(format!("Payment received for order {}, amount: ${:.2}", order_id, amount))
                    }
                }
                RestaurantEvent::KitchenError { message } => {
                    Err(format!("Kitchen fault: {}", message))
                }
                RestaurantEvent::CustomerFeedback { rating, comment } => {
                    if rating < 3 {
                        Err(format!("Low feedback ({} stars): {}", rating, comment))
                    } else {
                        Ok(format!("Positive feedback ({} stars): {}", rating, comment))
                    }
                }
            })
            .collect()
    }

    let event_stream = vec![
        RestaurantEvent::OrderPlaced { id: 1, total: 25.0 },
        RestaurantEvent::PaymentReceived { order_id: 1, amount: 25.0 },
        RestaurantEvent::KitchenError { message: "Oven malfunction".to_string() },
        RestaurantEvent::OrderPlaced { id: 2, total: -5.0 }, // Invalid order
        RestaurantEvent::CustomerFeedback { rating: 5, comment: "Amazing service".to_string() },
        RestaurantEvent::CustomerFeedback { rating: 2, comment: "Slow delivery".to_string() }, // Low rating
    ];

    let processed_results = process_event_stream(event_stream);

    println!("   Processed event stream:");
    for result in processed_results {
        match result {
            Ok(msg) => println!("   ✅ {}", msg),
            Err(err) => println!("   ❌ {}", err),
        }
    }

    // Clean up dummy files
    fs::remove_file(valid_config_path).ok();
    fs::remove_file(invalid_config_path).ok();
    fs::remove_file("daily_report.txt").ok();
    fs::remove_file("pasta_carbonara.recipe").ok();
    fs::remove_file("exploding_dish.recipe").ok();

    println!("\n🎯 Real-World Functional Error Handling Benefits:");
    println!("   🛡️ Compile-time guarantees for error handling pathways");
    println!("   📈 Composable pipelines for complex operations");
    println!("   📊 Structured error reporting for better debugging and logging");
    println!("   🔄 Flexible error recovery strategies (e.g., fallbacks)");
    println!("   🎨 Clear, readable code that integrates error handling seamlessly");

    println!("\n💡 Professional Implementation Guidelines:");
    println!("   • Define custom error enums that encapsulate domain-specific problems.");
    println!("   • Implement `From` trait for error types to enable `?` conversion.");
    println!("   • Use `?` operator for concise error propagation in pipelines.");
    println!("   • Leverage `Result` combinators (`and_then`, `or_else`, `map`, `map_err`) for functional flows.");
    println!("   • Document error scenarios and recovery procedures clearly.");
}

fn main() {
    demonstrate_real_world_functional_error_handling();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Kitchen Management System**

Remember our comprehensive professional kitchen management analogy:

- 🛡️ **Functional error handling** = **Ultra-reliable, data-driven kitchen** - Anticipating problems and building clear, explicit pathways
- 📦 **`Result<T, E>`** = **Dual-outcome dish container** - Explicitly holds either success (the dish) or a problem report
- ❓ **`?` operator** = **Smart assistant** - Automatically escalates problems up the chain, making the process seamless
- 🎨 **Functional patterns** = **Composed workflows** - Building robust operations by chaining predictable outcomes
- 📈 **Real-world applications** = **Complete management systems** - Implementing fault-tolerant operations in practice


### **Essential Functional Error Handling Concepts**

**`Result<T, E>` Enum:**

- `Ok(T)`: Represents success, carrying a value of type `T`.
- `Err(E)`: Represents failure, carrying an error value of type `E`.
- Makes error handling explicit and part of the type system.

**`?` Operator:**

- A concise way to propagate `Result` errors.
- If `Result` is `Ok(value)`, `value` is unwrapped and execution continues.
- If `Result` is `Err(error)`, `error` is returned from the current function immediately.
- Works similarly with `Option<T>` for missing data.
- Requires the error type to implement `From` for automatic conversion.


### **Key `Result` Methods**

| **Method** | **Purpose** | **Behavior on `Ok`** | **Behavior on `Err`** |
| :-- | :-- | :-- | :-- |
| `unwrap()` | Get value | Returns `T` | `panic!` |
| `expect()` | Get value | Returns `T` | `panic!` with message |
| `is_ok()` / `is_err()` | Check state | `true`/`false` | `false`/`true` |
| `ok()` / `err()` | Convert to `Option` | `Some(T)`/`None` | `None`/`Some(E)` |
| `unwrap_or(default)` | Get value or default | Returns `T` | Returns `default` |
| `map(f)` | Transform `Ok` value | Returns `Ok(f(T))` | Returns `Err(E)` |
| `map_err(f)` | Transform `Err` value | Returns `Ok(T)` | Returns `Err(f(E))` |
| `and_then(f)` | Chain `Result` operations | Returns `f(T)` | Returns `Err(E)` |
| `or_else(f)` | Chain `Result` operations | Returns `Ok(T)` | Returns `f(E)` |

### **Functional Error Handling Patterns**

**1. Chaining Operations (`and_then`, `map`, `?`):**

```rust
fn process_pipeline() -> Result<String, CustomError> {
    step1()?                        // Propagate Err from step1
        .and_then(|data1| step2(data1))?  // Propagate Err from step2
        .and_then(|data2| step3(data2))   // Final step
}
```

**2. Fallback Mechanisms (`or_else`):**

```rust
fn get_resource() -> Result<Resource, Error> {
    try_primary_source()
        .or_else(|_| try_secondary_source()) // Try secondary if primary fails
        .or_else(|_| try_cache())            // Try cache if secondary fails
}
```

**3. Error-Aware Iteration (`filter_map`, `flat_map`):**

```rust
let valid_items: Vec<T> = raw_items.into_iter()
    .filter_map(|item| item.parse().ok()) // Skip items that fail to parse
    .collect();
```

**4. Structured Error Types:**

- Define custom `enum`s for domain-specific errors (e.g., `OrderProcessError`).
- Implement `From<OtherError>` for automatic `?` conversion.


### **Why Functional Error Handling Is Powerful**

- **Explicitness**: Errors are part of the type signature, forcing handling.
- **Type Safety**: Compiler ensures error types are compatible.
- **Composability**: `Result` and `?` allow chaining fallible operations seamlessly.
- **Predictability**: No hidden exceptions or non-local control flow.
- **Efficiency**: Zero runtime overhead for the `Result` enum itself.


### **Best Practices Checklist**

**✅ Design Guidelines:**

- Use `Result<T, E>` for all recoverable errors.
- Define custom error `enum`s for domain-specific problems.
- Implement the `From` trait for your custom error `enum` to enable automatic conversion from other error types.
- Distinguish between recoverable errors (use `Result`) and unrecoverable bugs (use `panic!`).

**✅ Implementation Guidelines:**

- Prefer the `?` operator for concise error propagation.
- Use `Result` combinators (`and_then`, `or_else`, `map`, `map_err`) for functional composition.
- Avoid `unwrap()` and `expect()` in production code where failure is possible.
- Return `Result` values from functions that can fail, letting the caller decide how to handle the error.

**✅ Debugging Guidelines:**

- Read compiler error messages carefully; they often suggest fixes.
- Use `Debug` trait for custom error types to easily print them for debugging.
- Trace the flow of `Ok` and `Err` variants through your functional chains.

**❌ Common Pitfalls:**

- Using `panic!` for recoverable errors.
- Ignoring `Result` values or just `unwrap()`ing them blindly.
- Not implementing `From` trait for custom errors, leading to verbose `match` blocks.
- Building complex nested `match` statements instead of using `?` or combinators.


### **The Professional Advantage**

**Mastering functional error handling in Rust is like having complete control over your professional kitchen management system** - every potential problem is anticipated, explicitly managed, and integrated into a robust, predictable workflow:

- 🛡️ **Guaranteed robustness** - Compiler ensures all errors are handled.
- 📈 **Streamlined workflows** - Concise, readable code for fallible operations.
- 📊 **Structured problem reporting** - Detailed error types for better diagnosis.
- 🔄 **Flexible recovery** - Easily build fallbacks and alternative paths.
- ⚡ **High performance** - Error handling without runtime overhead.

**Understanding functional error handling transforms you from a programmer who fears errors to an architect** who designs resilient, fault-tolerant systems. Just as a master restaurant manager anticipates every challenge and builds contingency plans into daily operations, a skilled Rust programmer leverages `Result` and the `?` operator to create applications that are inherently robust and predictable.

This comprehensive understanding of functional error handling - from basic `Result` usage through advanced combinators and real-world applications - enables you to write Rust code that is not just correct, but resilient and maintainable, ready to gracefully handle any challenge thrown its way, much like a well-oiled, efficient restaurant during peak hours!

1. https://doc.rust-lang.org/book/ch09-00-error-handling.html
2. https://www.reddit.com/r/rust/comments/1bb7dco/error_handling_goodbest_practices/
3. https://andreabergia.com/blog/2023/05/error-handling-patterns/
4. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/error-handling.html
5. https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
6. https://www.youtube.com/watch?v=j-VQCYP7wyw
7. https://internals.rust-lang.org/t/explanation-of-the-error-handling/21506
8. https://news.ycombinator.com/item?id=41541883
