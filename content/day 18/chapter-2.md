+++
title = "Result<T, E> in Depth"
description = "Dive deep into Rust’s Result<T, E> for recoverable errors: construction, matching, the ? operator, mapping/combinators, custom error types, and best practices."
date = 2025-08-13
weight = 2
keywords = ["Rust", "Result", "error handling", "? operator", "map", "map_err", "combinators", "propagation"]
+++

# Result<T, E> in Rust: Comprehensive Documentation for Beginners

Understanding `Result<T, E>` in Rust is like learning to **manage a professional vegetarian restaurant's complete error handling and quality control system** - you need a systematic way to handle everything that can go wrong during daily operations while keeping the restaurant running smoothly. Just as a professional restaurant manager must distinguish between recoverable problems (like running out of an ingredient - substitute it) and unrecoverable emergencies (like a kitchen fire - evacuate immediately), Rust's `Result<T, E>` type provides a powerful, type-safe way to handle errors that can be recovered from, allowing your program to continue operating even when things don't go exactly as planned.

## The Professional Restaurant Quality Control System Analogy 🏪

### Imagine You're Operating a High-End Vegetarian Restaurant's Complete Error Management System

**The Two Possible Outcomes for Every Operation:**

```rust
// Every restaurant operation has two possible outcomes
enum RestaurantOperation<Success, Problem> {
    Success(Success),    // ✅ Everything went perfectly
    Problem(Problem),    // ⚠️ Something went wrong, but we can handle it
}

// In Rust, this is exactly what Result<T, E> represents!
use std::result::Result;

// T = Success type, E = Error type
fn prepare_dish(ingredient: &str) -> Result<String, String> {
    match ingredient {
        "quinoa" => Ok("Delicious Quinoa Buddha Bowl ready!".to_string()),
        "tomatoes" => Ok("Fresh Caprese Salad prepared!".to_string()),
        "unicorn_meat" => Err("Ingredient not available in vegetarian restaurant".to_string()),
        "" => Err("Cannot prepare dish without ingredients".to_string()),
        _ => Ok(format!("Creative dish made with {}", ingredient)),
    }
}
```

**Why Result<T, E> is Superior to Simple Error Codes:**

- ✅ **Type safety** - Compiler forces you to handle both success and error cases
- 🎯 **Clear intentions** - Function signatures tell you exactly what can go wrong
- 🔄 **Composable** - Chain operations together elegantly
- 🛡️ **No silent failures** - Can't accidentally ignore errors
- 📊 **Rich error information** - Carry detailed error context


## Understanding Result<T, E> Fundamentals

### What is Result<T, E>?

`Result<T, E>` is Rust's primary error-handling type - an enum with two variants:[^1][^2]

```rust
fn demonstrate_result_basics() {
    println!("🎯 Result<T, E> Fundamentals - Restaurant Quality Control");
    println!("{:=<60}", "");

    // Result is defined as an enum with two variants:
    // enum Result<T, E> {
    //     Ok(T),   // Success case containing value of type T
    //     Err(E),  // Error case containing error of type E
    // }

    println!("📊 Result<T, E> Structure:");
    println!("   • T = Type of the success value");
    println!("   • E = Type of the error value");
    println!("   • Ok(T) = Operation succeeded with value");
    println!("   • Err(E) = Operation failed with error");

    // Example 1: Simple restaurant operations
    fn check_table_availability(table_number: u32) -> Result<String, String> {
        match table_number {
            1..=20 => Ok(format!("Table {} is available for reservation", table_number)),
            21..=25 => Err(format!("Table {} is currently occupied", table_number)),
            _ => Err(format!("Table {} does not exist in restaurant", table_number)),
        }
    }

    println!("\n🪑 Table Availability Check:");

    // Test different scenarios
    let test_tables = [5, 23, 50];

    for table in test_tables {
        match check_table_availability(table) {
            Ok(message) => println!("   ✅ {}", message),
            Err(error) => println!("   ❌ Error: {}", error),
        }
    }

    // Example 2: More complex restaurant operation
    #[derive(Debug)]
    struct Order {
        id: u32,
        dish: String,
        price: f64,
    }

    #[derive(Debug)]
    enum OrderError {
        InvalidDish,
        PriceOutOfRange,
        KitchenClosed,
    }

    fn create_order(dish: &str, price: f64, hour: u8) -> Result<Order, OrderError> {
        // Check if kitchen is open
        if hour < 11 || hour > 22 {
            return Err(OrderError::KitchenClosed);
        }

        // Validate dish
        let valid_dishes = ["Quinoa Bowl", "Veggie Burger", "Lentil Soup", "Green Salad"];
        if !valid_dishes.contains(&dish) {
            return Err(OrderError::InvalidDish);
        }

        // Validate price range
        if price < 5.0 || price > 30.0 {
            return Err(OrderError::PriceOutOfRange);
        }

        // All validations passed - create order
        Ok(Order {
            id: 12345,
            dish: dish.to_string(),
            price,
        })
    }

    println!("\n📋 Order Creation Examples:");

    let order_attempts = [
        ("Quinoa Bowl", 15.99, 19),    // Valid order
        ("Pizza", 12.99, 19),          // Invalid dish
        ("Veggie Burger", 45.0, 19),   // Price too high
        ("Lentil Soup", 9.99, 23),     // Kitchen closed
    ];

    for (dish, price, hour) in order_attempts {
        match create_order(dish, price, hour) {
            Ok(order) => println!("   ✅ Order created: {:?}", order),
            Err(error) => println!("   ❌ Order failed: {:?}", error),
        }
    }
}

fn main() {
    demonstrate_result_basics();
}
```


### The Anatomy of Result<T, E>

**Understanding the type parameters and variants:**

```rust
fn demonstrate_result_anatomy() {
    println!("🔬 Result<T, E> Anatomy - Deep Dive into Success and Error Types");
    println!("{:=<65}", "");

    // Different Result types for different restaurant operations

    // Example 1: String success, String error (simple)
    fn simple_menu_lookup(item_id: u32) -> Result<String, String> {
        match item_id {
            1 => Ok("Quinoa Buddha Bowl".to_string()),
            2 => Ok("Mediterranean Wrap".to_string()),
            3 => Ok("Lentil Curry".to_string()),
            _ => Err(format!("Menu item {} not found", item_id))
        }
    }

    // Example 2: Complex success type, enum error type
    #[derive(Debug)]
    struct CustomerAccount {
        email: String,
        loyalty_points: u32,
        membership_tier: String,
    }

    #[derive(Debug)]
    enum AccountError {
        NotFound,
        Suspended,
        InvalidEmail,
        DatabaseError(String),
    }

    fn get_customer_account(email: &str) -> Result<CustomerAccount, AccountError> {
        // Validate email format
        if !email.contains('@') {
            return Err(AccountError::InvalidEmail);
        }

        // Simulate database lookup
        match email {
            "alice@email.com" => Ok(CustomerAccount {
                email: email.to_string(),
                loyalty_points: 1250,
                membership_tier: "Gold".to_string(),
            }),
            "banned@email.com" => Err(AccountError::Suspended),
            "db_error@test.com" => Err(AccountError::DatabaseError("Connection timeout".to_string())),
            _ => Err(AccountError::NotFound),
        }
    }

    // Example 3: Unit success type (just indicating success/failure)
    fn send_welcome_email(email: &str) -> Result<(), String> {
        if email.is_empty() {
            return Err("Email address cannot be empty".to_string());
        }

        if !email.contains('@') {
            return Err("Invalid email format".to_string());
        }

        // Simulate sending email
        println!("   📧 Welcome email sent to {}", email);
        Ok(()) // Success with no additional data
    }

    // Example 4: Numeric success type with custom error
    #[derive(Debug)]
    enum CalculationError {
        DivisionByZero,
        Overflow,
        InvalidInput,
    }

    fn calculate_tip_percentage(bill_total: f64, tip_amount: f64) -> Result<f64, CalculationError> {
        if bill_total <= 0.0 || tip_amount < 0.0 {
            return Err(CalculationError::InvalidInput);
        }

        if bill_total == 0.0 {
            return Err(CalculationError::DivisionByZero);
        }

        let percentage = (tip_amount / bill_total) * 100.0;

        if percentage > 1000.0 {  // Unreasonably high tip
            return Err(CalculationError::Overflow);
        }

        Ok(percentage)
    }

    println!("🧪 Testing Different Result Types:");

    // Test simple string Result
    println!("\n1️⃣ Simple String Result:");
    match simple_menu_lookup(1) {
        Ok(dish) => println!("   Found dish: {}", dish),
        Err(error) => println!("   Lookup error: {}", error),
    }

    // Test complex struct Result
    println!("\n2️⃣ Complex Struct Result:");
    match get_customer_account("alice@email.com") {
        Ok(account) => println!("   Account: {:?}", account),
        Err(error) => println!("   Account error: {:?}", error),
    }

    // Test unit Result
    println!("\n3️⃣ Unit Success Result:");
    match send_welcome_email("newcustomer@email.com") {
        Ok(()) => println!("   ✅ Email sent successfully"),
        Err(error) => println!("   ❌ Email error: {}", error),
    }

    // Test numeric Result
    println!("\n4️⃣ Numeric Result:");
    match calculate_tip_percentage(50.0, 10.0) {
        Ok(percentage) => println!("   Tip percentage: {:.1}%", percentage),
        Err(error) => println!("   Calculation error: {:?}", error),
    }

    println!("\n📊 Result Type Patterns:");
    println!("   Result<String, String>     → Simple success/error messages");
    println!("   Result<Struct, Enum>       → Rich data with structured errors");
    println!("   Result<(), Error>          → Success/failure with no success data");
    println!("   Result<Number, Error>      → Calculations with error handling");
    println!("   Result<Vec<T>, Error>      → Collections with potential failures");
}

fn main() {
    demonstrate_result_anatomy();
}
```


## Core Result<T, E> Operations - Restaurant Management

### 1. Creating and Returning Results

**Different ways to create and return Results in your restaurant operations:**

```rust
fn demonstrate_result_creation() {
    println!("🏗️ Creating Results - Restaurant Operation Patterns");
    println!("{:=<55}", "");

    // Pattern 1: Early returns for error conditions
    fn validate_reservation(party_size: u8, date: &str, time: &str) -> Result<String, String> {
        // Check party size first
        if party_size == 0 {
            return Err("Party size cannot be zero".to_string());
        }

        if party_size > 12 {
            return Err("Party size too large - maximum 12 guests".to_string());
        }

        // Check date format (simplified)
        if date.len() != 10 {
            return Err("Date must be in YYYY-MM-DD format".to_string());
        }

        // Check time format
        if !time.contains(':') {
            return Err("Time must include colon (e.g., 19:30)".to_string());
        }

        // All validations passed
        Ok(format!("Reservation confirmed for {} guests on {} at {}", party_size, date, time))
    }

    // Pattern 2: Match-based Result creation
    fn get_dish_category(dish: &str) -> Result<String, String> {
        match dish.to_lowercase().as_str() {
            "quinoa bowl" | "buddha bowl" | "grain bowl" => Ok("Bowls".to_string()),
            "veggie burger" | "portobello burger" => Ok("Burgers".to_string()),
            "lentil soup" | "tomato soup" | "mushroom soup" => Ok("Soups".to_string()),
            "caesar salad" | "greek salad" | "garden salad" => Ok("Salads".to_string()),
            "" => Err("Dish name cannot be empty".to_string()),
            _ => Err(format!("Unknown dish category for '{}'", dish))
        }
    }

    // Pattern 3: Conditional Result creation
    fn check_ingredient_stock(ingredient: &str, required_amount: u32) -> Result<u32, String> {
        // Simulate inventory lookup
        let stock_levels = [
            ("quinoa", 50),
            ("tomatoes", 30),
            ("lettuce", 45),
            ("avocado", 12),
            ("lentils", 35),
        ];

        for (item, stock) in stock_levels {
            if item == ingredient.to_lowercase() {
                if stock >= required_amount {
                    return Ok(stock - required_amount); // Remaining after use
                } else {
                    return Err(format!("Insufficient {}: need {}, have {}", ingredient, required_amount, stock));
                }
            }
        }

        Err(format!("Ingredient '{}' not found in inventory", ingredient))
    }

    // Pattern 4: Result from external operations
    fn parse_customer_age(age_str: &str) -> Result<u8, String> {
        match age_str.parse::<u8>() {
            Ok(age) => {
                if age > 120 {
                    Err("Age seems unrealistic".to_string())
                } else {
                    Ok(age)
                }
            }
            Err(_) => Err(format!("'{}' is not a valid age", age_str))
        }
    }

    // Pattern 5: Nested validation with multiple error points
    fn process_order_payment(amount_str: &str, payment_method: &str) -> Result<f64, String> {
        // Parse amount
        let amount = amount_str.parse::<f64>()
            .map_err(|_| format!("Invalid amount format: '{}'", amount_str))?;

        // Validate amount range
        if amount <= 0.0 {
            return Err("Amount must be positive".to_string());
        }

        if amount > 500.0 {
            return Err("Amount exceeds daily limit of $500".to_string());
        }

        // Validate payment method
        match payment_method.to_lowercase().as_str() {
            "cash" | "card" | "mobile" => {
                println!("   💳 Processing ${:.2} via {}", amount, payment_method);
                Ok(amount)
            }
            _ => Err(format!("Payment method '{}' not accepted", payment_method))
        }
    }

    println!("🧪 Testing Result Creation Patterns:");

    // Test reservation validation
    println!("\n📅 Reservation Validation:");
    let reservations = [
        (4, "2024-12-25", "19:30"),   // Valid
        (0, "2024-12-25", "19:30"),   // Invalid party size
        (8, "invalid-date", "19:30"), // Invalid date
        (6, "2024-12-25", "invalid"), // Invalid time
    ];

    for (size, date, time) in reservations {
        match validate_reservation(size, date, time) {
            Ok(confirmation) => println!("   ✅ {}", confirmation),
            Err(error) => println!("   ❌ {}", error),
        }
    }

    // Test dish categorization
    println!("\n🍽️ Dish Categorization:");
    let dishes = ["Quinoa Bowl", "Veggie Burger", "Unknown Dish", ""];

    for dish in dishes {
        match get_dish_category(dish) {
            Ok(category) => println!("   '{}' → Category: {}", dish, category),
            Err(error) => println!("   '{}' → Error: {}", dish, error),
        }
    }

    // Test inventory checking
    println!("\n📦 Inventory Checking:");
    let inventory_checks = [("quinoa", 20), ("tomatoes", 40), ("caviar", 5)];

    for (ingredient, needed) in inventory_checks {
        match check_ingredient_stock(ingredient, needed) {
            Ok(remaining) => println!("   {} → ✅ Used {}, {} remaining", ingredient, needed, remaining),
            Err(error) => println!("   {} → ❌ {}", ingredient, error),
        }
    }

    // Test age parsing
    println!("\n👤 Age Parsing:");
    let ages = ["25", "150", "abc", ""];

    for age_str in ages {
        match parse_customer_age(age_str) {
            Ok(age) => println!("   '{}' → ✅ Valid age: {}", age_str, age),
            Err(error) => println!("   '{}' → ❌ {}", age_str, error),
        }
    }

    // Test payment processing
    println!("\n💳 Payment Processing:");
    let payments = [("25.50", "card"), ("invalid", "cash"), ("1000.00", "card")];

    for (amount, method) in payments {
        match process_order_payment(amount, method) {
            Ok(processed) => println!("   ${} via {} → ✅ Processed ${:.2}", amount, method, processed),
            Err(error) => println!("   ${} via {} → ❌ {}", amount, method, error),
        }
    }

    println!("\n🎯 Result Creation Best Practices:");
    println!("   ✅ Use early returns for clear error conditions");
    println!("   ✅ Match on input values for categorization");
    println!("   ✅ Validate inputs step by step");
    println!("   ✅ Convert external errors with map_err()");
    println!("   ✅ Use ? operator for clean error propagation");
}

fn main() {
    demonstrate_result_creation();
}
```


### 2. Handling Results - Safe Error Management

**Different ways to safely handle Result values in your restaurant operations:**

```rust
fn demonstrate_result_handling() {
    println!("🛡️ Handling Results - Safe Restaurant Error Management");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Setup some restaurant operations that return Results
    fn lookup_menu_price(item: &str) -> Result<f64, String> {
        let menu = HashMap::from([
            ("quinoa bowl", 15.99),
            ("veggie burger", 12.49),
            ("lentil soup", 9.75),
            ("green salad", 8.99),
        ]);

        menu.get(item.to_lowercase().as_str())
            .copied()
            .ok_or_else(|| format!("Menu item '{}' not found", item))
    }

    fn calculate_tax(subtotal: f64, tax_rate: f64) -> Result<f64, String> {
        if subtotal < 0.0 {
            return Err("Subtotal cannot be negative".to_string());
        }

        if tax_rate < 0.0 || tax_rate > 1.0 {
            return Err("Tax rate must be between 0.0 and 1.0".to_string());
        }

        Ok(subtotal * tax_rate)
    }

    // Method 1: Pattern matching - The explicit way
    println!("🎯 Method 1: Pattern Matching (Explicit Handling)");

    let dish = "quinoa bowl";
    let price_result = lookup_menu_price(dish);

    match price_result {
        Ok(price) => {
            println!("   ✅ Found {}: ${:.2}", dish, price);

            // Chain another operation
            match calculate_tax(price, 0.08) {
                Ok(tax) => println!("   📊 Tax (8%): ${:.2}", tax),
                Err(tax_error) => println!("   ❌ Tax calculation failed: {}", tax_error),
            }
        }
        Err(error) => {
            println!("   ❌ Price lookup failed: {}", error);
        }
    }

    // Method 2: if let - For when you only care about success
    println!("\n✨ Method 2: if let (Success-focused handling)");

    if let Ok(price) = lookup_menu_price("veggie burger") {
        println!("   ✅ Veggie burger costs: ${:.2}", price);

        if let Ok(tax) = calculate_tax(price, 0.08) {
            let total = price + tax;
            println!("   💰 Total with tax: ${:.2}", total);
        }
    } else {
        println!("   ❌ Could not find veggie burger price");
    }

    // Method 3: unwrap_or - Provide default values
    println!("\n🔄 Method 3: unwrap_or (Default Values)");

    let unknown_dish_price = lookup_menu_price("pizza").unwrap_or(0.0);
    println!("   Pizza price (with default): ${:.2}", unknown_dish_price);

    let default_tax = calculate_tax(15.99, 0.08).unwrap_or(0.0);
    println!("   Tax calculation (with default): ${:.2}", default_tax);

    // Method 4: unwrap_or_else - Computed defaults
    println!("\n🧮 Method 4: unwrap_or_else (Computed Defaults)");

    let computed_default = lookup_menu_price("mystery_dish").unwrap_or_else(|error| {
        println!("   ⚠️ Lookup failed: {}, using average price", error);
        13.50 // Average menu price
    });
    println!("   Mystery dish price: ${:.2}", computed_default);

    // Method 5: map and map_err - Transform success and error values
    println!("\n🔄 Method 5: map and map_err (Value Transformation)");

    let formatted_price = lookup_menu_price("lentil soup")
        .map(|price| format!("${:.2}", price))
        .map_err(|error| format!("PRICE ERROR: {}", error));

    match formatted_price {
        Ok(formatted) => println!("   Formatted price: {}", formatted),
        Err(error) => println!("   {}", error),
    }

    // Method 6: and_then - Chain operations that can fail
    println!("\n🔗 Method 6: and_then (Operation Chaining)");

    let total_with_tax = lookup_menu_price("green salad")
        .and_then(|price| {
            println!("   Found salad: ${:.2}", price);
            calculate_tax(price, 0.08)
                .map(|tax| price + tax)
        });

    match total_with_tax {
        Ok(total) => println!("   Final total: ${:.2}", total),
        Err(error) => println!("   Calculation failed: {}", error),
    }

    // Method 7: or_else - Fallback operations
    println!("\n🚀 Method 7: or_else (Fallback Operations)");

    let price_with_fallback = lookup_menu_price("unavailable_dish")
        .or_else(|_| {
            println!("   First lookup failed, trying fallback...");
            lookup_menu_price("quinoa bowl") // Fallback to popular item
        })
        .or_else(|_| {
            println!("   Fallback failed, using emergency default");
            Ok(10.00) // Emergency default
        });

    match price_with_fallback {
        Ok(price) => println!("   Final price: ${:.2}", price),
        Err(error) => println!("   All fallbacks failed: {}", error),
    }

    // Method 8: Question mark operator (?) - Clean error propagation
    println!("\n❓ Method 8: Question Mark Operator (Clean Propagation)");

    fn calculate_order_total(dish1: &str, dish2: &str) -> Result<f64, String> {
        let price1 = lookup_menu_price(dish1)?; // Propagate error if it occurs
        let price2 = lookup_menu_price(dish2)?;

        let subtotal = price1 + price2;
        let tax = calculate_tax(subtotal, 0.08)?;

        Ok(subtotal + tax)
    }

    match calculate_order_total("quinoa bowl", "veggie burger") {
        Ok(total) => println!("   Order total: ${:.2}", total),
        Err(error) => println!("   Order calculation failed: {}", error),
    }

    // Method 9: Multiple Result handling
    println!("\n📦 Method 9: Multiple Result Handling");

    let order_items = ["quinoa bowl", "veggie burger", "mystery_item", "lentil soup"];
    let mut successful_prices = Vec::new();
    let mut failed_items = Vec::new();

    for item in order_items {
        match lookup_menu_price(item) {
            Ok(price) => {
                successful_prices.push((item, price));
            }
            Err(error) => {
                failed_items.push((item, error));
            }
        }
    }

    println!("   ✅ Found prices for {} items:", successful_prices.len());
    for (item, price) in successful_prices {
        println!("     {} → ${:.2}", item, price);
    }

    if !failed_items.is_empty() {
        println!("   ❌ Failed to find {} items:", failed_items.len());
        for (item, error) in failed_items {
            println!("     {} → {}", item, error);
        }
    }

    println!("\n🎯 Result Handling Best Practices:");
    println!("   ✅ Use pattern matching for explicit error handling");
    println!("   ✅ Use if let when you only care about success");
    println!("   ✅ Use unwrap_or for simple default values");
    println!("   ✅ Use and_then to chain fallible operations");
    println!("   ✅ Use ? operator for clean error propagation");
    println!("   ✅ Use or_else for fallback operations");
    println!("   ❌ Avoid unwrap() in production code");
}

fn main() {
    demonstrate_result_handling();
}
```


## Advanced Result<T, E> Patterns and Techniques

### 1. Error Propagation and the ? Operator

**Elegant error handling in complex restaurant operations:**

```rust
fn demonstrate_error_propagation() {
    println!("🔗 Error Propagation - Restaurant Operation Chains");
    println!("{:=<55}", "");

    use std::collections::HashMap;

    // Restaurant database simulation
    struct RestaurantDB {
        menu_prices: HashMap<String, f64>,
        inventory: HashMap<String, u32>,
        customers: HashMap<String, CustomerInfo>,
    }

    #[derive(Debug, Clone)]
    struct CustomerInfo {
        name: String,
        loyalty_points: u32,
        dietary_restrictions: Vec<String>,
    }

    #[derive(Debug)]
    enum RestaurantError {
        MenuItemNotFound(String),
        InsufficientInventory { item: String, needed: u32, available: u32 },
        CustomerNotFound(String),
        InvalidPrice(f64),
        PaymentProcessingError(String),
    }

    impl std::fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                RestaurantError::MenuItemNotFound(item) =>
                    write!(f, "Menu item '{}' not found", item),
                RestaurantError::InsufficientInventory { item, needed, available } =>
                    write!(f, "Not enough {}: need {}, have {}", item, needed, available),
                RestaurantError::CustomerNotFound(email) =>
                    write!(f, "Customer '{}' not found", email),
                RestaurantError::InvalidPrice(price) =>
                    write!(f, "Invalid price: ${:.2}", price),
                RestaurantError::PaymentProcessingError(msg) =>
                    write!(f, "Payment failed: {}", msg),
            }
        }
    }

    impl RestaurantDB {
        fn new() -> Self {
            let mut menu_prices = HashMap::new();
            menu_prices.insert("quinoa_bowl".to_string(), 15.99);
            menu_prices.insert("veggie_burger".to_string(), 12.49);
            menu_prices.insert("lentil_soup".to_string(), 9.75);

            let mut inventory = HashMap::new();
            inventory.insert("quinoa".to_string(), 50);
            inventory.insert("vegetables".to_string(), 30);
            inventory.insert("lentils".to_string(), 25);

            let mut customers = HashMap::new();
            customers.insert("alice@email.com".to_string(), CustomerInfo {
                name: "Alice Johnson".to_string(),
                loyalty_points: 150,
                dietary_restrictions: vec!["vegan".to_string()],
            });

            RestaurantDB { menu_prices, inventory, customers }
        }

        // Method without ? operator (verbose)
        fn get_menu_price_verbose(&self, item: &str) -> Result<f64, RestaurantError> {
            match self.menu_prices.get(item) {
                Some(&price) => Ok(price),
                None => Err(RestaurantError::MenuItemNotFound(item.to_string())),
            }
        }

        // Method with ? operator (clean)
        fn get_menu_price(&self, item: &str) -> Result<f64, RestaurantError> {
            self.menu_prices.get(item)
                .copied()
                .ok_or_else(|| RestaurantError::MenuItemNotFound(item.to_string()))
        }

        fn check_inventory(&self, item: &str, needed: u32) -> Result<(), RestaurantError> {
            let available = self.inventory.get(item)
                .copied()
                .ok_or_else(|| RestaurantError::MenuItemNotFound(item.to_string()))?;

            if available >= needed {
                Ok(())
            } else {
                Err(RestaurantError::InsufficientInventory {
                    item: item.to_string(),
                    needed,
                    available
                })
            }
        }

        fn get_customer(&self, email: &str) -> Result<&CustomerInfo, RestaurantError> {
            self.customers.get(email)
                .ok_or_else(|| RestaurantError::CustomerNotFound(email.to_string()))
        }
    }

    // Complex operation chain using ? operator
    fn process_complete_order(
        db: &RestaurantDB,
        customer_email: &str,
        dish: &str,
        quantity: u32
    ) -> Result<String, RestaurantError> {
        // Step 1: Validate customer exists
        let customer = db.get_customer(customer_email)?;

        // Step 2: Get menu price
        let unit_price = db.get_menu_price(dish)?;

        // Step 3: Check inventory
        let ingredient = match dish {
            "quinoa_bowl" => "quinoa",
            "veggie_burger" => "vegetables",
            "lentil_soup" => "lentils",
            _ => return Err(RestaurantError::MenuItemNotFound(dish.to_string())),
        };

        db.check_inventory(ingredient, quantity)?;

        // Step 4: Calculate total
        let total = unit_price * quantity as f64;

        // Step 5: Apply loyalty discount
        let discount = if customer.loyalty_points > 100 { 0.10 } else { 0.0 };
        let final_total = total * (1.0 - discount);

        // Step 6: Process payment (simulated)
        process_payment(final_total)?;

        Ok(format!(
            "Order completed for {}: {} x {} = ${:.2} ({}% discount applied)",
            customer.name,
            quantity,
            dish,
            final_total,
            (discount * 100.0) as u32
        ))
    }

    fn process_payment(amount: f64) -> Result<(), RestaurantError> {
        if amount <= 0.0 {
            return Err(RestaurantError::InvalidPrice(amount));
        }

        if amount > 1000.0 {
            return Err(RestaurantError::PaymentProcessingError(
                "Amount exceeds daily limit".to_string()
            ));
        }

        // Simulate payment processing
        println!("   💳 Processing payment: ${:.2}", amount);
        Ok(())
    }

    // Compare verbose vs clean error handling
    println!("📊 Comparing Error Handling Approaches:");

    let db = RestaurantDB::new();

    // Verbose approach (without ? operator)
    fn verbose_order_processing(db: &RestaurantDB, dish: &str) -> Result<f64, RestaurantError> {
        match db.get_menu_price_verbose(dish) {
            Ok(price) => {
                match db.check_inventory("quinoa", 1) {
                    Ok(()) => {
                        match process_payment(price) {
                            Ok(()) => Ok(price),
                            Err(e) => Err(e),
                        }
                    }
                    Err(e) => Err(e),
                }
            }
            Err(e) => Err(e),
        }
    }

    // Clean approach (with ? operator)
    fn clean_order_processing(db: &RestaurantDB, dish: &str) -> Result<f64, RestaurantError> {
        let price = db.get_menu_price(dish)?;
        db.check_inventory("quinoa", 1)?;
        process_payment(price)?;
        Ok(price)
    }

    println!("\n🧪 Testing Order Processing:");

    let test_orders = [
        ("alice@email.com", "quinoa_bowl", 2),     // Valid order
        ("unknown@email.com", "quinoa_bowl", 1),   // Unknown customer
        ("alice@email.com", "pizza", 1),           // Unknown dish
        ("alice@email.com", "quinoa_bowl", 100),   // Insufficient inventory
    ];

    for (email, dish, quantity) in test_orders {
        println!("\n   📋 Order: {} x {} for {}", quantity, dish, email);

        match process_complete_order(&db, email, dish, quantity) {
            Ok(confirmation) => println!("     ✅ {}", confirmation),
            Err(error) => println!("     ❌ Order failed: {}", error),
        }
    }

    // Demonstrate error propagation through function chains
    println!("\n🔗 Error Propagation Chain Example:");

    fn validate_and_prepare_order(
        dish: &str,
        quantity: u32,
        customer_email: &str
    ) -> Result<String, RestaurantError> {
        let db = RestaurantDB::new();

        // Each ? operator can short-circuit and return the error
        let customer = db.get_customer(customer_email)?;
        println!("     ✓ Customer validated: {}", customer.name);

        let price = db.get_menu_price(dish)?;
        println!("     ✓ Price found: ${:.2}", price);

        let ingredient = match dish {
            "quinoa_bowl" => "quinoa",
            "lentil_soup" => "lentils",
            _ => "vegetables",
        };

        db.check_inventory(ingredient, quantity)?;
        println!("     ✓ Inventory sufficient");

        let total = price * quantity as f64;
        process_payment(total)?;
        println!("     ✓ Payment processed");

        Ok(format!("Order ready: {} x {} for ${:.2}", quantity, dish, total))
    }

    match validate_and_prepare_order("quinoa_bowl", 1, "alice@email.com") {
        Ok(result) => println!("   🎉 Success: {}", result),
        Err(error) => println!("   💥 Failed at some step: {}", error),
    }

    println!("\n🎯 ? Operator Benefits:");
    println!("   ✅ Reduces boilerplate code dramatically");
    println!("   ✅ Makes error propagation explicit and clean");
    println!("   ✅ Stops execution at first error encountered");
    println!("   ✅ Maintains readability in complex operations");
    println!("   ✅ Works with any Result<T, E> or Option<T>");
}

fn main() {
    demonstrate_error_propagation();
}
```


### 2. Custom Error Types and Error Conversion

**Building sophisticated error handling systems for restaurant operations:**

```rust
fn demonstrate_custom_error_types() {
    println!("🏗️ Custom Error Types - Professional Restaurant Error System");
    println!("{:=<65}", "");

    use std::fmt;

    // Define a comprehensive error system for the restaurant
    #[derive(Debug)]
    enum RestaurantError {
        // Customer-related errors
        Customer(CustomerError),

        // Menu-related errors
        Menu(MenuError),

        // Kitchen-related errors
        Kitchen(KitchenError),

        // Payment-related errors
        Payment(PaymentError),

        // System-related errors
        System(SystemError),
    }

    #[derive(Debug)]
    enum CustomerError {
        NotFound { email: String },
        AccountSuspended { reason: String },
        InvalidReservation { details: String },
    }

    #[derive(Debug)]
    enum MenuError {
        ItemNotAvailable { item: String },
        CategoryNotFound { category: String },
        PriceError { item: String, invalid_price: f64 },
    }

    #[derive(Debug)]
    enum KitchenError {
        InsufficientIngredients { item: String, needed: u32, available: u32 },
        EquipmentFailure { equipment: String },
        StaffShortage { required: u32, available: u32 },
    }

    #[derive(Debug)]
    enum PaymentError {
        InsufficientFunds { required: f64, available: f64 },
        InvalidPaymentMethod { method: String },
        ProcessingFailed { transaction_id: String, reason: String },
    }

    #[derive(Debug)]
    enum SystemError {
        DatabaseConnection { details: String },
        ConfigurationMissing { setting: String },
        NetworkTimeout { operation: String, timeout_seconds: u32 },
    }

    // Implement Display trait for user-friendly error messages
    impl fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                RestaurantError::Customer(e) => write!(f, "Customer Error: {}", e),
                RestaurantError::Menu(e) => write!(f, "Menu Error: {}", e),
                RestaurantError::Kitchen(e) => write!(f, "Kitchen Error: {}", e),
                RestaurantError::Payment(e) => write!(f, "Payment Error: {}", e),
                RestaurantError::System(e) => write!(f, "System Error: {}", e),
            }
        }
    }

    impl fmt::Display for CustomerError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                CustomerError::NotFound { email } =>
                    write!(f, "Customer '{}' not found in our system", email),
                CustomerError::AccountSuspended { reason } =>
                    write!(f, "Account suspended: {}", reason),
                CustomerError::InvalidReservation { details } =>
                    write!(f, "Invalid reservation: {}", details),
            }
        }
    }

    impl fmt::Display for MenuError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                MenuError::ItemNotAvailable { item } =>
                    write!(f, "Menu item '{}' is currently unavailable", item),
                MenuError::CategoryNotFound { category } =>
                    write!(f, "Menu category '{}' does not exist", category),
                MenuError::PriceError { item, invalid_price } =>
                    write!(f, "Invalid price ${:.2} for item '{}'", invalid_price, item),
            }
        }
    }

    impl fmt::Display for KitchenError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                KitchenError::InsufficientIngredients { item, needed, available } =>
                    write!(f, "Not enough {}: need {}, have {}", item, needed, available),
                KitchenError::EquipmentFailure { equipment } =>
                    write!(f, "{} is currently out of order", equipment),
                KitchenError::StaffShortage { required, available } =>
                    write!(f, "Staff shortage: need {}, have {}", required, available),
            }
        }
    }

    impl fmt::Display for PaymentError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                PaymentError::InsufficientFunds { required, available } =>
                    write!(f, "Insufficient funds: need ${:.2}, have ${:.2}", required, available),
                PaymentError::InvalidPaymentMethod { method } =>
                    write!(f, "Payment method '{}' is not accepted", method),
                PaymentError::ProcessingFailed { transaction_id, reason } =>
                    write!(f, "Payment processing failed ({}): {}", transaction_id, reason),
            }
        }
    }

    impl fmt::Display for SystemError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                SystemError::DatabaseConnection { details } =>
                    write!(f, "Database connection failed: {}", details),
                SystemError::ConfigurationMissing { setting } =>
                    write!(f, "Required configuration '{}' is missing", setting),
                SystemError::NetworkTimeout { operation, timeout_seconds } =>
                    write!(f, "Network timeout for '{}' after {} seconds", operation, timeout_seconds),
            }
        }
    }

    // Implement From trait for automatic error conversion
    impl From<CustomerError> for RestaurantError {
        fn from(error: CustomerError) -> Self {
            RestaurantError::Customer(error)
        }
    }

    impl From<MenuError> for RestaurantError {
        fn from(error: MenuError) -> Self {
            RestaurantError::Menu(error)
        }
    }

    impl From<KitchenError> for RestaurantError {
        fn from(error: KitchenError) -> Self {
            RestaurantError::Kitchen(error)
        }
    }

    impl From<PaymentError> for RestaurantError {
        fn from(error: PaymentError) -> Self {
            RestaurantError::Payment(error)
        }
    }

    impl From<SystemError> for RestaurantError {
        fn from(error: SystemError) -> Self {
            RestaurantError::System(error)
        }
    }

    // Restaurant operations using custom error types
    struct Restaurant {
        name: String,
    }

    impl Restaurant {
        fn new(name: &str) -> Self {
            Restaurant { name: name.to_string() }
        }

        fn find_customer(&self, email: &str) -> Result<Customer, CustomerError> {
            match email {
                "alice@email.com" => Ok(Customer {
                    email: email.to_string(),
                    name: "Alice Johnson".to_string(),
                    balance: 50.0,
                }),
                "banned@email.com" => Err(CustomerError::AccountSuspended {
                    reason: "Multiple no-shows".to_string()
                }),
                _ => Err(CustomerError::NotFound {
                    email: email.to_string()
                }),
            }
        }

        fn check_menu_item(&self, item: &str) -> Result<MenuItem, MenuError> {
            match item {
                "quinoa_bowl" => Ok(MenuItem {
                    name: item.to_string(),
                    price: 15.99,
                    available: true,
                }),
                "sold_out_special" => Err(MenuError::ItemNotAvailable {
                    item: item.to_string()
                }),
                _ => Err(MenuError::ItemNotAvailable {
                    item: item.to_string()
                }),
            }
        }

        fn prepare_dish(&self, item: &str, quantity: u32) -> Result<(), KitchenError> {
            match item {
                "quinoa_bowl" => {
                    let available_quinoa = 20;
                    if available_quinoa >= quantity {
                        Ok(())
                    } else {
                        Err(KitchenError::InsufficientIngredients {
                            item: "quinoa".to_string(),
                            needed: quantity,
                            available: available_quinoa,
                        })
                    }
                }
                _ => Err(KitchenError::EquipmentFailure {
                    equipment: "main prep station".to_string()
                }),
            }
        }

        fn process_payment(&self, customer: &Customer, amount: f64) -> Result<String, PaymentError> {
            if customer.balance < amount {
                return Err(PaymentError::InsufficientFunds {
                    required: amount,
                    available: customer.balance,
                });
            }

            // Simulate payment processing
            Ok(format!("TXN_{}", rand::random::<u32>()))
        }

        // Combined operation that can fail at multiple points
        fn complete_order(&self, customer_email: &str, item: &str, quantity: u32) -> Result<String, RestaurantError> {
            // Each step can produce a different type of error
            // The ? operator automatically converts them using From trait

            let customer = self.find_customer(customer_email)?;  // CustomerError -> RestaurantError
            let menu_item = self.check_menu_item(item)?;         // MenuError -> RestaurantError
            self.prepare_dish(item, quantity)?;                  // KitchenError -> RestaurantError

            let total_amount = menu_item.price * quantity as f64;
            let transaction_id = self.process_payment(&customer, total_amount)?; // PaymentError -> RestaurantError

            Ok(format!(
                "Order completed: {} x {} for {} (Transaction: {})",
                quantity, item, customer.name, transaction_id
            ))
        }
    }

    #[derive(Debug)]
    struct Customer {
        email: String,
        name: String,
        balance: f64,
    }

    #[derive(Debug)]
    struct MenuItem {
        name: String,
        price: f64,
        available: bool,
    }

    // Add a simple random number generator for transaction IDs
    mod rand {
        static mut SEED: u32 = 42;

        pub fn random<T>() -> T where T: From<u32> {
            unsafe {
                SEED = SEED.wrapping_mul(1664525).wrapping_add(1013904223);
                T::from(SEED % 10000)
            }
        }
    }

    println!("🧪 Testing Custom Error System:");

    let restaurant = Restaurant::new("Green Garden Bistro");

    let test_orders = [
        ("alice@email.com", "quinoa_bowl", 1),      // Should succeed
        ("unknown@email.com", "quinoa_bowl", 1),    // Customer not found
        ("alice@email.com", "pizza", 1),            // Menu item not available
        ("alice@email.com", "quinoa_bowl", 50),     // Insufficient ingredients
        ("banned@email.com", "quinoa_bowl", 1),     // Account suspended
    ];

    for (email, item, quantity) in test_orders {
        println!("\n   📋 Processing order: {} x {} for {}", quantity, item, email);

        match restaurant.complete_order(email, item, quantity) {
            Ok(confirmation) => {
                println!("     ✅ Success: {}", confirmation);
            }
            Err(error) => {
                println!("     ❌ Failed: {}", error);

                // Demonstrate error handling by type
                match error {
                    RestaurantError::Customer(CustomerError::NotFound { .. }) => {
                        println!("     💡 Suggestion: Ask customer to register");
                    }
                    RestaurantError::Menu(MenuError::ItemNotAvailable { .. }) => {
                        println!("     💡 Suggestion: Recommend similar items");
                    }
                    RestaurantError::Kitchen(KitchenError::InsufficientIngredients { .. }) => {
                        println!("     💡 Suggestion: Offer alternative dishes or wait time");
                    }
                    RestaurantError::Customer(CustomerError::AccountSuspended { .. }) => {
                        println!("     💡 Suggestion: Contact customer service");
                    }
                    _ => {
                        println!("     💡 Suggestion: Please try again later");
                    }
                }
            }
        }
    }

    println!("\n🎯 Custom Error Type Benefits:");
    println!("   ✅ Type-safe error categorization");
    println!("   ✅ Rich error context and details");
    println!("   ✅ Automatic error conversion with From trait");
    println!("   ✅ User-friendly error messages");
    println!("   ✅ Easy error handling by category");
    println!("   ✅ Extensible error system design");

    println!("\n💡 Design Principles:");
    println!("   🎯 Each error type represents a specific domain");
    println!("   📝 Include relevant context in error variants");
    println!("   🔄 Implement From traits for automatic conversion");
    println!("   👤 Implement Display for user-friendly messages");
    println!("   🏗️ Structure errors hierarchically when appropriate");
}

fn main() {
    demonstrate_custom_error_types();
}
```


## Result<T, E> vs panic! - Making the Right Choice

### Understanding When to Use Each Error Handling Approach

```rust
fn demonstrate_result_vs_panic() {
    println!("⚖️ Result<T, E> vs panic! - Choosing the Right Error Strategy");
    println!("{:=<65}", "");

    // Scenario comparison: When to use Result vs when to use panic!

    println!("🎯 Decision Framework:");
    println!("   📊 Expected vs Unexpected errors");
    println!("   🔄 Recoverable vs Unrecoverable situations");
    println!("   👤 User errors vs Programming errors");
    println!("   📈 Production vs Development/Testing code");

    // ✅ Use Result<T, E> for: Expected, recoverable errors
    println!("\n✅ Use Result<T, E> for Expected, Recoverable Errors:");

    // User input validation - expected to sometimes fail
    fn parse_customer_age(input: &str) -> Result<u8, String> {
        match input.trim().parse::<u8>() {
            Ok(age) if age <= 120 => Ok(age),
            Ok(age) => Err(format!("Age {} seems unrealistic", age)),
            Err(_) => Err(format!("'{}' is not a valid age", input)),
        }
    }

    // File operations - files might not exist
    fn load_menu_config(filename: &str) -> Result<String, String> {
        // Simulate file reading
        match filename {
            "menu.txt" => Ok("Quinoa Bowl,15.99\nVeggie Burger,12.49".to_string()),
            "missing.txt" => Err("File not found".to_string()),
            "corrupted.txt" => Err("File is corrupted".to_string()),
            _ => Ok("Default menu configuration".to_string()),
        }
    }

    // Network operations - networks can fail
    fn fetch_daily_specials() -> Result<Vec<String>, String> {
        let network_available = true; // Simulate network status

        if !network_available {
            return Err("Network connection unavailable".to_string());
        }

        // Simulate API call
        Ok(vec![
            "Today's special: Mushroom Risotto".to_string(),
            "Soup of the day: Butternut Squash".to_string(),
        ])
    }

    // Business rule validation - rules can be violated
    fn apply_loyalty_discount(total: f64, loyalty_points: u32) -> Result<f64, String> {
        if total <= 0.0 {
            return Err("Order total must be positive".to_string());
        }

        let discount_rate = match loyalty_points {
            0..=99 => 0.0,
            100..=499 => 0.05,
            500..=999 => 0.10,
            1000.. => 0.15,
        };

        Ok(total * (1.0 - discount_rate))
    }

    // ❌ Use panic! for: Programming errors and impossible conditions
    println!("\n🚨 Use panic! for Programming Errors and Impossible Conditions:");

    // Array bounds that should never be exceeded (programming error)
    fn get_menu_section(section_id: usize) -> &'static str {
        let sections = ["Appetizers", "Mains", "Desserts", "Beverages"];

        if section_id >= sections.len() {
            panic!("🐛 PROGRAMMING ERROR: Invalid section_id {}. Valid range: 0-{}",
                   section_id, sections.len() - 1);
        }

        sections[section_id]
    }

    // Invariant violations (should never happen if code is correct)
    fn calculate_split_bill(total: f64, people: u32) -> f64 {
        if people == 0 {
            panic!("🐛 LOGIC ERROR: Cannot split bill among 0 people");
        }

        if total < 0.0 {
            panic!("🐛 DATA ERROR: Bill total ${:.2} cannot be negative", total);
        }

        total / people as f64
    }

    // Configuration that must be valid (deployment/setup error)
    fn get_restaurant_capacity() -> u32 {
        const MAX_TABLES: u32 = 25;
        const SEATS_PER_TABLE: u32 = 4;

        if MAX_TABLES == 0 {
            panic!("🔧 CONFIG ERROR: MAX_TABLES cannot be zero");
        }

        if SEATS_PER_TABLE == 0 {
            panic!("🔧 CONFIG ERROR: SEATS_PER_TABLE cannot be zero");
        }

        MAX_TABLES * SEATS_PER_TABLE
    }

    println!("\n🧪 Testing Result<T, E> Usage (Recoverable Errors):");

    // Test user input parsing
    let age_inputs = ["25", "150", "abc", ""];
    for input in age_inputs {
        match parse_customer_age(input) {
            Ok(age) => println!("   ✅ Age '{}' → {} years old", input, age),
            Err(error) => println!("   🔄 Age '{}' → {} (can ask user to retry)", input, error),
        }
    }

    // Test file operations with fallbacks
    let config_files = ["menu.txt", "missing.txt", "backup.txt"];
    for filename in config_files {
        match load_menu_config(filename) {
            Ok(config) => println!("   ✅ Loaded {}: {} chars", filename, config.len()),
            Err(error) => println!("   🔄 {} failed: {} (can try backup)", filename, error),
        }
    }

    // Test network operations with graceful degradation
    match fetch_daily_specials() {
        Ok(specials) => {
            println!("   ✅ Daily specials loaded:");
            for special in specials {
                println!("     • {}", special);
            }
        }
        Err(error) => println!("   🔄 Specials unavailable: {} (can show cached menu)", error),
    }

    // Test business rules with user feedback
    match apply_loyalty_discount(50.0, 250) {
        Ok(discounted) => println!("   ✅ Loyalty discount applied: ${:.2}", discounted),
        Err(error) => println!("   🔄 Discount error: {} (can proceed without discount)", error),
    }

    println!("\n🧪 Testing panic! Usage (Programming Errors):");

    // These would panic in real scenarios - we'll show the concept
    println!("   🔧 Valid configuration check:");
    let capacity = get_restaurant_capacity();
    println!("     Restaurant capacity: {} guests", capacity);

    println!("   📊 Valid section access:");
    for i in 0..4 {
        let section = get_menu_section(i);
        println!("     Section {}: {}", i, section);
    }

    println!("   💰 Valid bill calculation:");
    let split_amount = calculate_split_bill(100.0, 4);
    println!("     Split bill: ${:.2} per person", split_amount);

    // Examples of what WOULD panic (we won't actually call these)
    println!("\n🚨 Examples that WOULD panic (programming errors):");
    println!("   • get_menu_section(10) → Index out of bounds");
    println!("   • calculate_split_bill(50.0, 0) → Division by zero");
    println!("   • calculate_split_bill(-10.0, 2) → Negative total");

    println!("\n🎯 Decision Guidelines Summary:");

    println!("\n✅ Use Result<T, E> when:");
    println!("   👤 Handling user input (parsing, validation)");
    println!("   📁 Working with external resources (files, network)");
    println!("   🏢 Enforcing business rules (discounts, limits)");
    println!("   🔄 The program can recover and continue");
    println!("   📊 Errors are part of normal program flow");

    println!("\n🚨 Use panic! when:");
    println!("   🐛 Programming logic is violated");
    println!("   🔧 Configuration/setup is invalid");
    println!("   📐 Mathematical impossibilities occur");
    println!("   🎯 Contract preconditions are violated");
    println!("   🧪 Writing tests, examples, or prototypes");

    println!("\n💡 Key Principle:");
    println!("   Result<T, E> = 'This might fail, and that's okay'");
    println!("   panic!       = 'This should never happen if code is correct'");

    // Practical example: Combining both approaches
    println!("\n🔗 Practical Example: Combining Both Approaches");

    fn process_restaurant_order(
        customer_email: &str,
        dish_id: usize,
        quantity_str: &str
    ) -> Result<String, String> {
        // Use panic! for programming errors
        let menu = ["Quinoa Bowl", "Veggie Burger", "Lentil Soup"];
        if dish_id >= menu.len() {
            panic!("🐛 BUG: Invalid dish_id {} passed to function", dish_id);
        }
        let dish_name = menu[dish_id]; // Safe because we validated above

        // Use Result for user input errors
        let quantity = quantity_str.parse::<u32>()
            .map_err(|_| format!("Invalid quantity: '{}'", quantity_str))?;

        if quantity == 0 {
            return Err("Quantity must be at least 1".to_string());
        }

        if quantity > 20 {
            return Err("Maximum 20 items per order".to_string());
        }

        // Use Result for business rule validation
        if customer_email.is_empty() {
            return Err("Customer email is required".to_string());
        }

        // Success case
        Ok(format!("Order: {} x {} for {}", quantity, dish_name, customer_email))
    }

    // Test the combined approach
    println!("\n   🧪 Testing combined approach:");

    let test_cases = [
        ("alice@email.com", 0, "2"),     // Valid order
        ("bob@email.com", 1, "abc"),     // Invalid quantity (Result error)
        ("", 0, "1"),                    // Missing email (Result error)
        // ("alice@email.com", 10, "1"), // Invalid dish_id (would panic!)
    ];

    for (email, dish_id, quantity) in test_cases {
        match process_restaurant_order(email, dish_id, quantity) {
            Ok(order) => println!("     ✅ {}", order),
            Err(error) => println!("     🔄 Recoverable error: {}", error),
        }
    }

    println!("\n🏆 Professional Error Handling:");
    println!("   ✅ Use Result<T, E> as your primary error handling mechanism[^1][^3]");
    println!("   ✅ Reserve panic! for truly exceptional circumstances");
    println!("   ✅ Design APIs that return Results for expected failures");
    println!("   ✅ Use panic! to catch programming bugs during development");
    println!("   ✅ Let users recover from Result errors gracefully");
}

fn main() {
    demonstrate_result_vs_panic();
}
```


## Best Practices and Professional Patterns

### Professional Result<T, E> Usage Guidelines

```rust
fn demonstrate_result_best_practices() {
    println!("✅ Result<T, E> Best Practices - Professional Error Handling");
    println!("{:=<65}", "");

    use std::collections::HashMap;
    use std::fmt;

    // Best Practice 1: Design meaningful error types
    println!("🎯 Best Practice 1: Meaningful Error Types");

    // ❌ Poor: Generic string errors
    fn bad_validate_email(email: &str) -> Result<String, String> {
        if email.contains('@') {
            Ok(email.to_string())
        } else {
            Err("Invalid email".to_string()) // Not helpful!
        }
    }

    // ✅ Good: Specific error types with context
    #[derive(Debug)]
    enum EmailError {
        Empty,
        MissingAtSymbol,
        MultipleFAtSymbols,
        InvalidDomain(String),
        TooLong { length: usize, max: usize },
    }

    impl fmt::Display for EmailError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                EmailError::Empty => write!(f, "Email address cannot be empty"),
                EmailError::MissingAtSymbol => write!(f, "Email must contain @ symbol"),
                EmailError::MultipleFAtSymbols => write!(f, "Email cannot contain multiple @ symbols"),
                EmailError::InvalidDomain(domain) => write!(f, "Invalid domain: {}", domain),
                EmailError::TooLong { length, max } =>
                    write!(f, "Email too long: {} characters (max: {})", length, max),
            }
        }
    }

    fn good_validate_email(email: &str) -> Result<String, EmailError> {
        if email.is_empty() {
            return Err(EmailError::Empty);
        }

        if email.len() > 254 {
            return Err(EmailError::TooLong { length: email.len(), max: 254 });
        }

        let at_count = email.chars().filter(|&c| c == '@').count();
        match at_count {
            0 => Err(EmailError::MissingAtSymbol),
            1 => {
                let parts: Vec<&str> = email.split('@').collect();
                let domain = parts[^1];
                if domain.is_empty() || !domain.contains('.') {
                    Err(EmailError::InvalidDomain(domain.to_string()))
                } else {
                    Ok(email.to_string())
                }
            }
            _ => Err(EmailError::MultipleFAtSymbols),
        }
    }

    // Best Practice 2: Use Result<T, E> for API boundaries
    println!("\n🔗 Best Practice 2: Result at API Boundaries");

    pub struct RestaurantAPI;

    impl RestaurantAPI {
        // ✅ Public methods return Results for error handling
        pub fn create_reservation(
            &self,
            customer_email: &str,
            party_size: u8,
            date: &str
        ) -> Result<ReservationConfirmation, ReservationError> {
            // Validate inputs
            let _validated_email = self.validate_email_internal(customer_email)?;
            let _validated_party_size = self.validate_party_size_internal(party_size)?;
            let _validated_date = self.validate_date_internal(date)?;

            // Business logic (simplified)
            Ok(ReservationConfirmation {
                id: format!("RES_{}", 12345),
                customer_email: customer_email.to_string(),
                party_size,
                date: date.to_string(),
            })
        }

        // Private methods can use panic! for internal contract violations
        fn validate_email_internal(&self, email: &str) -> Result<String, ReservationError> {
            good_validate_email(email)
                .map_err(|e| ReservationError::InvalidInput(format!("Email error: {}", e)))
        }

        fn validate_party_size_internal(&self, size: u8) -> Result<u8, ReservationError> {
            match size {
                1..=12 => Ok(size),
                0 => Err(ReservationError::InvalidInput("Party size cannot be zero".to_string())),
                _ => Err(ReservationError::InvalidInput(format!("Party size {} exceeds maximum of 12", size))),
            }
        }

        fn validate_date_internal(&self, date: &str) -> Result<String, ReservationError> {
            if date.len() == 10 && date.chars().nth(4) == Some('-') && date.chars().nth(7) == Some('-') {
                Ok(date.to_string())
            } else {
                Err(ReservationError::InvalidInput("Date must be in YYYY-MM-DD format".to_string()))
            }
        }
    }

    #[derive(Debug)]
    pub struct ReservationConfirmation {
        pub id: String,
        pub customer_email: String,
        pub party_size: u8,
        pub date: String,
    }

    #[derive(Debug)]
    pub enum ReservationError {
        InvalidInput(String),
        SystemError(String),
        NotAvailable { date: String, time: String },
    }

    impl fmt::Display for ReservationError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                ReservationError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
                ReservationError::SystemError(msg) => write!(f, "System error: {}", msg),
                ReservationError::NotAvailable { date, time } =>
                    write!(f, "No availability on {} at {}", date, time),
            }
        }
    }

    // Best Practice 3: Use combinators for clean error handling
    println!("\n🧩 Best Practice 3: Result Combinators");

    fn parse_menu_item(line: &str) -> Result<MenuItem, String> {
        let parts: Vec<&str> = line.split(',').collect();

        if parts.len() != 2 {
            return Err("Line must have exactly 2 parts separated by comma".to_string());
        }

        let name = parts[^0].trim();
        let price_str = parts[^1].trim();

        if name.is_empty() {
            return Err("Menu item name cannot be empty".to_string());
        }

        let price = price_str.parse::<f64>()
            .map_err(|_| format!("Invalid price: '{}'", price_str))?;

        if price <= 0.0 {
            return Err(format!("Price must be positive, got: {}", price));
        }

        Ok(MenuItem { name: name.to_string(), price })
    }

    fn parse_menu_file(content: &str) -> Result<Vec<MenuItem>, String> {
        content
            .lines()
            .filter(|line| !line.trim().is_empty())
            .enumerate()
            .map(|(i, line)| {
                parse_menu_item(line)
                    .map_err(|e| format!("Line {}: {}", i + 1, e))
            })
            .collect()
    }

    #[derive(Debug)]
    struct MenuItem {
        name: String,
        price: f64,
    }

    // Best Practice 4: Provide helpful error context
    println!("\n📝 Best Practice 4: Rich Error Context");

    fn process_customer_order(
        customer_id: &str,
        items: &[(String, u32)], // (item_name, quantity)
        payment_method: &str
    ) -> Result<OrderSummary, OrderError> {
        // Add context at each step
        let customer = validate_customer(customer_id)
            .map_err(|e| OrderError::CustomerValidation {
                customer_id: customer_id.to_string(),
                details: e,
            })?;

        let order_items = validate_order_items(items)
            .map_err(|e| OrderError::InvalidItems {
                customer_id: customer_id.to_string(),
                details: e,
            })?;

        let total = calculate_total(&order_items)
            .map_err(|e| OrderError::CalculationError {
                customer_id: customer_id.to_string(),
                details: e,
            })?;

        process_payment(&customer, total, payment_method)
            .map_err(|e| OrderError::PaymentFailed {
                customer_id: customer_id.to_string(),
                amount: total,
                payment_method: payment_method.to_string(),
                details: e,
            })?;

        Ok(OrderSummary {
            customer_name: customer.name,
            items: order_items,
            total,
        })
    }

    #[derive(Debug)]
    struct Customer {
        name: String,
        email: String,
    }

    #[derive(Debug)]
    struct OrderItem {
        name: String,
        quantity: u32,
        unit_price: f64,
    }

    #[derive(Debug)]
    struct OrderSummary {
        customer_name: String,
        items: Vec<OrderItem>,
        total: f64,
    }

    #[derive(Debug)]
    enum OrderError {
        CustomerValidation { customer_id: String, details: String },
        InvalidItems { customer_id: String, details: String },
        CalculationError { customer_id: String, details: String },
        PaymentFailed {
            customer_id: String,
            amount: f64,
            payment_method: String,
            details: String
        },
    }

    impl fmt::Display for OrderError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                OrderError::CustomerValidation { customer_id, details } =>
                    write!(f, "Customer validation failed for '{}': {}", customer_id, details),
                OrderError::InvalidItems { customer_id, details } =>
                    write!(f, "Invalid items for customer '{}': {}", customer_id, details),
                OrderError::CalculationError { customer_id, details } =>
                    write!(f, "Calculation error for customer '{}': {}", customer_id, details),
                OrderError::PaymentFailed { customer_id, amount, payment_method, details } =>
                    write!(f, "Payment of ${:.2} via {} failed for '{}': {}",
                           amount, payment_method, customer_id, details),
            }
        }
    }

    // Helper functions for demonstration
    fn validate_customer(customer_id: &str) -> Result<Customer, String> {
        if customer_id == "CUST_001" {
            Ok(Customer {
                name: "Alice Johnson".to_string(),
                email: "alice@email.com".to_string(),
            })
        } else {
            Err(format!("Customer '{}' not found", customer_id))
        }
    }

    fn validate_order_items(items: &[(String, u32)]) -> Result<Vec<OrderItem>, String> {
        let menu_prices = HashMap::from([
            ("Quinoa Bowl".to_string(), 15.99),
            ("Veggie Burger".to_string(), 12.49),
        ]);

        let mut order_items = Vec::new();
        for (name, quantity) in items {
            if *quantity == 0 {
                return Err(format!("Quantity for '{}' cannot be zero", name));
            }

            let unit_price = menu_prices.get(name)
                .ok_or_else(|| format!("Item '{}' not found in menu", name))?;

            order_items.push(OrderItem {
                name: name.clone(),
                quantity: *quantity,
                unit_price: *unit_price,
            });
        }

        Ok(order_items)
    }

    fn calculate_total(items: &[OrderItem]) -> Result<f64, String> {
        let total: f64 = items.iter()
            .map(|item| item.unit_price * item.quantity as f64)
            .sum();

        if total <= 0.0 {
            Err("Order total must be positive".to_string())
        } else {
            Ok(total)
        }
    }

    fn process_payment(customer: &Customer, amount: f64, method: &str) -> Result<(), String> {
        match method {
            "cash" | "card" => Ok(()),
            _ => Err(format!("Payment method '{}' not accepted", method)),
        }
    }

    println!("🧪 Testing Best Practices:");

    // Test meaningful error types
    println!("\n1️⃣ Testing Email Validation:");
    let test_emails = ["alice@example.com", "", "invalid", "user@", "a@b.c"];
    for email in test_emails {
        match good_validate_email(email) {
            Ok(valid) => println!("   ✅ Valid: {}", valid),
            Err(error) => println!("   ❌ Invalid '{}': {}", email, error),
        }
    }

    // Test API boundary handling
    println!("\n2️⃣ Testing API Boundaries:");
    let api = RestaurantAPI;
    match api.create_reservation("alice@email.com", 4, "2024-12-25") {
        Ok(confirmation) => println!("   ✅ Reservation: {:?}", confirmation),
        Err(error) => println!("   ❌ Reservation failed: {}", error),
    }

    // Test combinator usage
    println!("\n3️⃣ Testing Result Combinators:");
    let menu_content = "Quinoa Bowl,15.99\nVeggie Burger,12.49\nInvalid Line\nGreen Salad,8.99";
    match parse_menu_file(menu_content) {
        Ok(items) => {
            println!("   ✅ Parsed {} menu items:", items.len());
            for item in items {
                println!("     • {} → ${:.2}", item.name, item.price);
            }
        }
        Err(error) => println!("   ❌ Menu parsing failed: {}", error),
    }

    // Test rich error context
    println!("\n4️⃣ Testing Rich Error Context:");
    let order_items = vec![
        ("Quinoa Bowl".to_string(), 2),
        ("Veggie Burger".to_string(), 1),
    ];

    match process_customer_order("CUST_001", &order_items, "card") {
        Ok(summary) => println!("   ✅ Order processed: {:?}", summary),
        Err(error) => println!("   ❌ Order failed: {}", error),
    }

    println!("\n📋 Result<T, E> Best Practices Summary:");
    println!("   ✅ Design specific, meaningful error types with context[^1][^3]");
    println!("   ✅ Use Result<T, E> at API boundaries for caller error handling");
    println!("   ✅ Leverage Result combinators (map, and_then, or_else)");
    println!("   ✅ Provide rich error context for debugging and user feedback");
    println!("   ✅ Use ? operator for clean error propagation");
    println!("   ✅ Implement Display trait for user-friendly error messages");
    println!("   ✅ Chain operations with proper error conversion");
    println!("   ❌ Avoid generic String errors in production code");
    println!("   ❌ Don't use unwrap() without careful consideration");
}

fn main() {
    demonstrate_result_best_practices();
}
```


## Summary and Decision Guidelines

### **Mental Model: The Professional Restaurant Error Management System**

Remember our restaurant error management analogy:

- ✅ **Result<T, E>** = **Professional problem-solving system** - Handle issues gracefully to keep serving customers
- 🎯 **Ok(T)** = **Successful operation** - Everything went as planned, here's your result
- ⚠️ **Err(E)** = **Manageable problem** - Something went wrong, but we can handle it and continue
- 🔗 **? operator** = **Efficient escalation system** - Pass problems up the chain until someone can solve them
- 📊 **Custom error types** = **Detailed problem categorization** - Know exactly what went wrong and how to fix it


### **Essential Result<T, E> Patterns**

**Core Usage Pattern:**

```rust
// Basic Result pattern
fn restaurant_operation() -> Result<SuccessType, ErrorType> {
    // Validation
    if input_invalid() {
        return Err(ErrorType::InvalidInput);
    }

    // Processing that might fail
    let result = risky_operation()?; // Use ? for propagation

    // Success
    Ok(result)
}

// Handling the Result
match restaurant_operation() {
    Ok(success) => println!("Success: {:?}", success),
    Err(error) => println!("Error: {}", error),
}
```

**Common Result Combinators:**

```rust
// Transform success values
result.map(|value| format!("Processed: {}", value))

// Chain operations that can fail
result.and_then(|value| another_operation(value))

// Provide fallbacks
result.or_else(|_| fallback_operation())

// Extract values safely
let value = result.unwrap_or(default_value);
let value = result.unwrap_or_else(|| compute_default());
```


### **Decision Framework: Result vs panic!**

```
🤔 Should this be Result<T, E> or panic!?

├─ 👤 Is this error caused by external factors?
│  └─ ✅ YES → Use Result<T, E>
│
├─ 🔄 Can the program recover and continue?
│  └─ ✅ YES → Use Result<T, E>
│
├─ 📊 Is this expected during normal operation?
│  └─ ✅ YES → Use Result<T, E>
│
└─ 🐛 Is this a programming error or impossible condition?
   └─ ✅ YES → Use panic!
```


### **Result<T, E> Use Cases Quick Reference**

| **Scenario** | **Use Result** | **Example** | **Error Type** |
| :-- | :-- | :-- | :-- |
| **User Input** | ✅ | Parse age from string | `Result<u8, ParseError>` |
| **File I/O** | ✅ | Read configuration file | `Result<String, IoError>` |
| **Network** | ✅ | API request | `Result<Data, NetworkError>` |
| **Business Rules** | ✅ | Validate order total | `Result<Order, ValidationError>` |
| **Database** | ✅ | Query customer data | `Result<Customer, DbError>` |
| **Parsing** | ✅ | JSON deserialization | `Result<Struct, JsonError>` |

### **Best Practices Checklist**

**✅ DO:**

- Use `Result<T, E>` for all fallible operations[^2][^1]
- Design specific error types with meaningful context
- Use the `?` operator for clean error propagation
- Implement `Display` trait for user-friendly error messages
- Handle `Result` values explicitly with `match` or combinators
- Use `map_err()` to convert between error types
- Provide fallback mechanisms with `unwrap_or()` and `or_else()`

**❌ DON'T:**

- Use generic `String` errors in production code
- Call `unwrap()` without being certain of success
- Ignore `Result` return values (compiler will warn)
- Use `Result` for programming logic errors (use `panic!`)
- Create deeply nested error hierarchies unnecessarily
- Forget to implement error conversion traits (`From`, `Into`)


### **Professional Error Handling Patterns**

```rust
// Pattern 1: Custom error types
#[derive(Debug)]
enum RestaurantError {
    InvalidInput(String),
    NotFound(String),
    SystemError(String),
}

// Pattern 2: Error conversion
impl From<ParseIntError> for RestaurantError {
    fn from(error: ParseIntError) -> Self {
        RestaurantError::InvalidInput(error.to_string())
    }
}

// Pattern 3: Rich error context
fn process_order(data: &str) -> Result<Order, RestaurantError> {
    let parsed = parse_order_data(data)
        .map_err(|e| RestaurantError::InvalidInput(format!("Parse failed: {}", e)))?;

    validate_order(&parsed)
        .map_err(|e| RestaurantError::InvalidInput(format!("Validation failed: {}", e)))?;

    Ok(parsed)
}

// Pattern 4: Multiple error handling
fn process_multiple_orders(orders: &[String]) -> (Vec<Order>, Vec<RestaurantError>) {
    let mut successful = Vec::new();
    let mut failed = Vec::new();

    for order_data in orders {
        match process_order(order_data) {
            Ok(order) => successful.push(order),
            Err(error) => failed.push(error),
        }
    }

    (successful, failed)
}
```


### **The Professional Advantage**

**Mastering Result<T, E> in Rust is like being a world-class restaurant manager** who gracefully handles every possible problem while keeping excellent service:

- 🎯 **Explicit error handling** - Never accidentally ignore problems
- 🔄 **Graceful recovery** - Continue operating when things go wrong
- 📊 **Rich error information** - Know exactly what happened and why
- 🛡️ **Type safety** - Compiler ensures all errors are handled
- ⚡ **Performance** - Zero-cost abstractions for error handling

**Understanding Result<T, E> transforms you from a programmer who struggles with errors to a systems expert** who builds resilient, user-friendly Rust applications. Just as a professional restaurant anticipates problems and has procedures to handle them gracefully without disrupting the dining experience, a skilled Rust programmer uses `Result<T, E>` to build software that handles errors elegantly and continues providing value even when things don't go exactly as planned.

This comprehensive understanding of `Result<T, E>` - from basic usage through advanced patterns - makes your Rust programs more reliable, your error messages more helpful, and your development process more robust, whether you're building simple utilities or complex enterprise systems that need to handle millions of operations safely and efficiently!

1. https://doc.rust-lang.org/std/result/
2. https://dev.to/leapcell/rusts-result-type-error-handling-made-easy-58i
3. https://doc.rust-lang.org/rust-by-example/error/result.html
4. https://doc.rust-lang.org/std/result/enum.Result.html
5. https://www.geeksforgeeks.org/rust/rust-result-type/
6. https://stackoverflow.com/questions/57414785/example-for-using-result-instead-of-unwrap-on-function-in-rust
7. https://www.geeksforgeeks.org/rust/rust-panic-error-handling-mechanism/
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch09-02-recoverable-errors-with-result.html
9. https://bitfieldconsulting.com/posts/rust-errors-option-result
10. https://www.youtube.com/watch?v=6AkRRzcAUKk
11. https://www.newline.co/@lsunsi/a-gentle-introduction-to-results-in-rust--b0391487
12. https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
13. https://www.alexdwilson.dev/learning-in-public/result-vs-panic-in-rust
14. https://stackoverflow.com/questions/67650449/resultt-e-type-in-rust-why-reference-cant-be-passed-as-t-or-e
15. https://blog.logrocket.com/panic-vs-error-rust/
16. https://www.djamware.com/post/6881abbd2d906261a96f3386/mastering-error-handling-in-rust-result-option-and-match-explained
17. https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html
18. https://www.reddit.com/r/rust/comments/18ofh6u/when_to_panic_vs_forward_error_to_caller/
19. https://users.rust-lang.org/t/is-error-handling-in-rust-all-about-when-you-can-and-cant-afford-to-return-a-result-t-e-instance/94902
20. https://codedamn.com/news/rust/advanced-error-handling-rust-exploring-power-result-type
