+++
title = "Error Propagation Patterns in Rust"
description = "Explore common error propagation techniques in Rust, including the use of the `?` operator, `map_err`, and custom conversions to simplify and standardize error handling."
date = 2025-08-13
weight = 5
keywords = ["Rust", "error handling", "error propagation", "Result", "question mark operator", "map_err", "From trait", "custom error conversion"]
+++

# Error Propagation Patterns in Rust: Comprehensive Documentation for Beginners

Understanding error propagation patterns in Rust is like learning to **design a complete emergency communication system for your vegetarian restaurant chain** - you need efficient ways to pass critical information from the point where problems are discovered (like a kitchen staff member finding spoiled ingredients) all the way up through the management hierarchy (line cook → sous chef → head chef → restaurant manager → regional manager) until it reaches someone who can make the right decision. Just as a professional restaurant needs clear protocols for escalating different types of problems to the appropriate decision-makers, Rust's error propagation patterns provide systematic ways to pass error information up through your function call stack until it reaches code that can handle it appropriately.

## The Professional Restaurant Emergency Communication System Analogy 🏢

### Imagine You're Designing Communication Protocols for Your Restaurant Empire

**The Problem with Ad-Hoc Error Handling:**

```rust
// ❌ No systematic error propagation - like having no communication protocols
fn handle_order_chaos() -> String {
    // Kitchen discovers problem but doesn't know who to tell
    let ingredients_result = check_ingredients();
    if ingredients_result == "spoiled" {
        // Panic! Everyone runs around confused!
        return "HELP! SOMETHING IS WRONG!".to_string();
    }

    // No clear escalation path
    let cooking_result = cook_food();
    if cooking_result == "burned" {
        // More chaos!
        return "FIRE! FIRE!".to_string();
    }

    "Maybe it worked?".to_string()
}
```

**The Power of Systematic Error Propagation:**

```rust
// ✅ Professional error propagation - like having clear communication protocols
fn handle_order_professionally() -> Result<String, RestaurantError> {
    let ingredients = check_ingredients()?;  // → Reports to next level if problem
    let prepped = prepare_ingredients(ingredients)?;  // → Escalates if needed
    let cooked = cook_food(prepped)?;  // → Passes problem up if cooking fails
    let plated = plate_dish(cooked)?;  // → Reports presentation issues

    Ok(format!("Order completed: {}", plated))  // Success flows through naturally
}
```

**Why Error Propagation Patterns Are Essential:**

- 📡 **Clear communication channels** - Problems reach the right decision-makers
- ⚡ **Efficient escalation** - No time wasted on inappropriate handling attempts
- 🎯 **Appropriate responses** - Different problems get different solutions
- 🔄 **Consistent protocols** - Everyone knows how to handle and pass along issues
- 🛡️ **No lost information** - Error context travels with the problem


## Understanding Error Propagation Fundamentals

### What is Error Propagation?

Error propagation is the process of passing error information up through the call stack until it reaches code capable of handling it:

```rust
fn demonstrate_error_propagation_basics() {
    println!("📡 Error Propagation Fundamentals - Restaurant Communication System");
    println!("{:=<70}", "");

    // Error propagation is like a restaurant communication chain
    println!("💡 What Error Propagation Does:");
    println!("   • 🔍 Detects problems at the source");
    println!("   • 📡 Transmits error information upward");
    println!("   • 🎯 Reaches appropriate handling level");
    println!("   • 🔄 Maintains error context and details");
    println!("   • ⚡ Enables clean separation of concerns");

    // Basic restaurant hierarchy for error handling
    #[derive(Debug)]
    enum RestaurantError {
        IngredientIssue(String),
        CookingProblem(String),
        ServiceError(String),
        SystemFailure(String),
    }

    impl std::fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                RestaurantError::IngredientIssue(msg) => write!(f, "Ingredient problem: {}", msg),
                RestaurantError::CookingProblem(msg) => write!(f, "Cooking issue: {}", msg),
                RestaurantError::ServiceError(msg) => write!(f, "Service problem: {}", msg),
                RestaurantError::SystemFailure(msg) => write!(f, "System failure: {}", msg),
            }
        }
    }

    // Level 1: Kitchen staff discovers problems
    fn check_ingredient_quality(ingredient: &str) -> Result<String, RestaurantError> {
        match ingredient {
            "fresh_tomatoes" => Ok("High quality tomatoes".to_string()),
            "spoiled_lettuce" => Err(RestaurantError::IngredientIssue(
                "Lettuce is wilted and unusable".to_string()
            )),
            "expired_milk" => Err(RestaurantError::IngredientIssue(
                "Milk is past expiration date".to_string()
            )),
            _ => Ok(format!("Standard quality {}", ingredient)),
        }
    }

    // Level 2: Line cook processes ingredients
    fn prepare_ingredient(ingredient: &str) -> Result<String, RestaurantError> {
        let quality_check = check_ingredient_quality(ingredient)?;  // Propagate if problem

        match ingredient {
            "fresh_tomatoes" => Ok("Tomatoes washed and diced".to_string()),
            item if item.contains("difficult") => Err(RestaurantError::CookingProblem(
                format!("Don't know how to prepare {}", item)
            )),
            _ => Ok(format!("Prepared {}", ingredient)),
        }
    }

    // Level 3: Sous chef coordinates dish preparation
    fn prepare_dish(dish_name: &str, ingredients: &[&str]) -> Result<String, RestaurantError> {
        let mut prepared_ingredients = Vec::new();

        for ingredient in ingredients {
            let prepped = prepare_ingredient(ingredient)?;  // Stop if any ingredient fails
            prepared_ingredients.push(prepped);
        }

        if dish_name == "impossible_dish" {
            return Err(RestaurantError::CookingProblem(
                "Recipe not in our cookbook".to_string()
            ));
        }

        Ok(format!("Dish '{}' prepared with: {}",
                  dish_name,
                  prepared_ingredients.join(", ")))
    }

    // Level 4: Head chef manages complete order
    fn complete_order(order: &[(&str, &[&str])]) -> Result<String, RestaurantError> {
        let mut completed_dishes = Vec::new();

        for (dish_name, ingredients) in order {
            let dish = prepare_dish(dish_name, ingredients)?;  // Escalate any dish problems
            completed_dishes.push(dish);
        }

        if completed_dishes.len() > 10 {
            return Err(RestaurantError::ServiceError(
                "Order too large for current staff capacity".to_string()
            ));
        }

        Ok(format!("Order completed: {} dishes ready", completed_dishes.len()))
    }

    // Level 5: Restaurant manager handles customer requests
    fn process_customer_order(customer_id: &str, order: &[(&str, &[&str])]) -> Result<String, RestaurantError> {
        if customer_id.is_empty() {
            return Err(RestaurantError::ServiceError(
                "Customer ID required".to_string()
            ));
        }

        println!("   📋 Processing order for customer: {}", customer_id);

        let order_result = complete_order(order)?;  // Let kitchen problems bubble up

        Ok(format!("Customer {} order: {}", customer_id, order_result))
    }

    println!("🧪 Testing Error Propagation Chain:");

    // Test successful propagation
    println!("\n✅ Successful Order (No Errors to Propagate):");
    let good_order = [
        ("Quinoa Salad", &["fresh_tomatoes", "quinoa"][..]),
        ("Veggie Wrap", &["spinach", "carrots"][..]),
    ];

    match process_customer_order("CUST_001", &good_order) {
        Ok(result) => println!("   🎉 SUCCESS: {}", result),
        Err(error) => println!("   ❌ ERROR: {}", error),
    }

    // Test error propagation at different levels
    println!("\n❌ Error Propagation Examples:");

    let test_scenarios = [
        // Level 1 error (ingredient quality)
        ("Ingredient Quality Issue", "CUST_002", vec![
            ("Salad", &["spoiled_lettuce", "tomatoes"][..])
        ]),

        // Level 2 error (preparation problem)
        ("Preparation Problem", "CUST_003", vec![
            ("Complex Dish", &["difficult_ingredient"][..])
        ]),

        // Level 3 error (unknown recipe)
        ("Recipe Problem", "CUST_004", vec![
            ("impossible_dish", &["normal_ingredient"][..])
        ]),

        // Level 4 error (service capacity)
        ("Service Capacity", "CUST_005", vec![
            ("Simple Dish", &["ingredient1"]);  // Repeat 12 times
            12
        ].into_iter().flatten().collect()),

        // Level 5 error (missing customer info)
        ("Customer Info Missing", "", vec![
            ("Normal Dish", &["normal_ingredient"][..])
        ]),
    ];

    for (scenario_name, customer_id, order) in test_scenarios {
        println!("\n   🔍 Testing: {}", scenario_name);

        match process_customer_order(customer_id, &order) {
            Ok(result) => println!("     ✅ Unexpected success: {}", result),
            Err(error) => {
                println!("     ❌ Error propagated: {}", error);
                println!("     📊 Error traveled up the chain and was caught at the manager level");
            }
        }
    }

    println!("\n🎯 Error Propagation Benefits Demonstrated:");
    println!("   📡 Problems detected at source level (ingredient check)");
    println!("   ⬆️ Errors automatically flow up the management chain");
    println!("   🎯 All errors handled at appropriate management level");
    println!("   🔄 Consistent error information maintained throughout");
    println!("   ⚡ No intermediate levels need error handling logic");
    println!("   🛡️ Type safety ensures no errors are lost or ignored");
}

fn main() {
    demonstrate_error_propagation_basics();
}
```


## Core Error Propagation Patterns

### 1. Manual Error Propagation with Match

**The explicit approach - like having detailed communication protocols:**

```rust
fn demonstrate_manual_error_propagation() {
    println!("🔧 Manual Error Propagation - Detailed Communication Protocols");
    println!("{:=<65}", "");

    use std::collections::HashMap;

    // Restaurant operation that requires explicit error handling at each step
    #[derive(Debug)]
    enum KitchenError {
        IngredientNotFound(String),
        InsufficientQuantity { item: String, needed: u32, available: u32 },
        EquipmentFailure(String),
        SafetyViolation(String),
    }

    impl std::fmt::Display for KitchenError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                KitchenError::IngredientNotFound(item) =>
                    write!(f, "Ingredient '{}' not found in inventory", item),
                KitchenError::InsufficientQuantity { item, needed, available } =>
                    write!(f, "Need {} of {}, but only {} available", needed, item, available),
                KitchenError::EquipmentFailure(equipment) =>
                    write!(f, "Equipment failure: {}", equipment),
                KitchenError::SafetyViolation(details) =>
                    write!(f, "Safety violation: {}", details),
            }
        }
    }

    // Inventory system
    struct Inventory {
        items: HashMap<String, u32>,
    }

    impl Inventory {
        fn new() -> Self {
            let mut items = HashMap::new();
            items.insert("tomatoes".to_string(), 50);
            items.insert("lettuce".to_string(), 30);
            items.insert("quinoa".to_string(), 25);
            items.insert("avocado".to_string(), 0);  // Out of stock

            Inventory { items }
        }

        fn check_item(&self, item: &str, quantity: u32) -> Result<(), KitchenError> {
            match self.items.get(item) {
                Some(&available) => {
                    if available >= quantity {
                        Ok(())
                    } else {
                        Err(KitchenError::InsufficientQuantity {
                            item: item.to_string(),
                            needed: quantity,
                            available,
                        })
                    }
                }
                None => Err(KitchenError::IngredientNotFound(item.to_string())),
            }
        }
    }

    // Kitchen operations with manual error propagation
    fn check_equipment(equipment: &str) -> Result<String, KitchenError> {
        match equipment {
            "stove" => Ok("Stove is operational".to_string()),
            "broken_oven" => Err(KitchenError::EquipmentFailure("Oven temperature sensor failed".to_string())),
            "refrigerator" => Ok("Refrigerator running normally".to_string()),
            _ => Ok(format!("{} is available", equipment)),
        }
    }

    fn validate_food_safety(process: &str) -> Result<String, KitchenError> {
        match process {
            "raw_handling" => Err(KitchenError::SafetyViolation("Cross-contamination risk with raw ingredients".to_string())),
            "temperature_check" => Ok("Temperature within safe range".to_string()),
            "hygiene_protocol" => Ok("Hygiene protocols followed".to_string()),
            _ => Ok(format!("Safety check passed for {}", process)),
        }
    }

    // Manual error propagation - explicit handling at each step
    fn prepare_dish_manual(
        dish_name: &str,
        ingredients: &[(&str, u32)],
        equipment: &[&str]
    ) -> Result<String, KitchenError> {
        println!("   🔧 Manual preparation of: {}", dish_name);

        let inventory = Inventory::new();

        // Step 1: Check all ingredients (manual propagation)
        for (ingredient, quantity) in ingredients {
            println!("     🔍 Checking ingredient: {} ({})", ingredient, quantity);

            match inventory.check_item(ingredient, *quantity) {
                Ok(()) => {
                    println!("       ✅ {} available", ingredient);
                }
                Err(error) => {
                    println!("       ❌ Ingredient problem detected");
                    return Err(error);  // Manual error propagation
                }
            }
        }

        // Step 2: Check all equipment (manual propagation)
        for equipment_item in equipment {
            println!("     🔧 Checking equipment: {}", equipment_item);

            match check_equipment(equipment_item) {
                Ok(status) => {
                    println!("       ✅ {}", status);
                }
                Err(error) => {
                    println!("       ❌ Equipment problem detected");
                    return Err(error);  // Manual error propagation
                }
            }
        }

        // Step 3: Validate safety protocols (manual propagation)
        let safety_checks = ["temperature_check", "hygiene_protocol"];
        for safety_process in safety_checks {
            println!("     🛡️ Safety check: {}", safety_process);

            match validate_food_safety(safety_process) {
                Ok(status) => {
                    println!("       ✅ {}", status);
                }
                Err(error) => {
                    println!("       ❌ Safety violation detected");
                    return Err(error);  // Manual error propagation
                }
            }
        }

        // If all checks pass, complete the dish
        Ok(format!("Dish '{}' prepared successfully with {} ingredients",
                  dish_name, ingredients.len()))
    }

    // Alternative pattern: Early return strategy
    fn prepare_dish_early_return(
        dish_name: &str,
        ingredients: &[(&str, u32)]
    ) -> Result<String, KitchenError> {
        println!("   ⚡ Early return preparation of: {}", dish_name);

        let inventory = Inventory::new();

        // Early return pattern - check and return immediately on error
        for (ingredient, quantity) in ingredients {
            let check_result = inventory.check_item(ingredient, *quantity);
            if let Err(error) = check_result {
                println!("     ❌ Early return triggered by: {}", ingredient);
                return Err(error);  // Immediate return on first error
            }
            println!("     ✅ {} validated", ingredient);
        }

        // Equipment check with early return
        let required_equipment = ["stove", "refrigerator"];
        for equipment in required_equipment {
            let equipment_result = check_equipment(equipment);
            if let Err(error) = equipment_result {
                println!("     ❌ Early return triggered by equipment: {}", equipment);
                return Err(error);
            }
        }

        Ok(format!("Dish '{}' completed with early return pattern", dish_name))
    }

    println!("🧪 Testing Manual Error Propagation Patterns:");

    // Test successful manual propagation
    println!("\n1️⃣ Successful Manual Propagation:");
    let good_ingredients = [("tomatoes", 5), ("lettuce", 3), ("quinoa", 10)];
    let equipment = ["stove", "refrigerator"];

    match prepare_dish_manual("Quinoa Salad", &good_ingredients, &equipment) {
        Ok(result) => println!("   🎉 SUCCESS: {}", result),
        Err(error) => println!("   ❌ UNEXPECTED ERROR: {}", error),
    }

    // Test error propagation at different stages
    println!("\n2️⃣ Error Propagation at Ingredient Stage:");
    let bad_ingredients = [("tomatoes", 5), ("avocado", 10)];  // Avocado out of stock

    match prepare_dish_manual("Impossible Salad", &bad_ingredients, &equipment) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error caught and propagated: {}", error);
            println!("   📊 Error originated from ingredient check, propagated to dish preparation");
        }
    }

    // Test equipment failure propagation
    println!("\n3️⃣ Error Propagation at Equipment Stage:");
    let broken_equipment = ["stove", "broken_oven"];

    match prepare_dish_manual("Baked Dish", &good_ingredients, &broken_equipment) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error caught and propagated: {}", error);
            println!("   📊 Error originated from equipment check, propagated upward");
        }
    }

    // Test early return pattern
    println!("\n4️⃣ Early Return Pattern:");
    let mixed_ingredients = [("tomatoes", 5), ("missing_ingredient", 1), ("quinoa", 10)];

    match prepare_dish_early_return("Test Dish", &mixed_ingredients) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Early return triggered: {}", error);
            println!("   📊 Processing stopped immediately when error encountered");
        }
    }

    println!("\n🎯 Manual Error Propagation Patterns:");
    println!("   🔧 Explicit error handling with match statements");
    println!("   ⚡ Early return on first error encountered");
    println!("   📊 Full control over error handling flow");
    println!("   🎯 Clear visibility into each error checking step");
    println!("   💡 Good for learning and debugging error flow");

    println!("\n📋 When to Use Manual Propagation:");
    println!("   🔍 When you need detailed control over error handling");
    println!("   📊 When you want to log or handle specific errors differently");
    println!("   🎯 When learning error handling concepts");
    println!("   🔧 When debugging complex error scenarios");
    println!("   ⚡ When performance is critical (avoid ? operator overhead)");
}

fn main() {
    demonstrate_manual_error_propagation();
}
```


### 2. The ? Operator - Automatic Error Propagation

**The elegant approach - like having smart communication systems:**

```rust
fn demonstrate_question_mark_propagation() {
    println!("❓ ? Operator Error Propagation - Smart Communication Systems");
    println!("{:=<65}", "");

    use std::collections::HashMap;

    // Restaurant system with automatic error propagation
    #[derive(Debug)]
    enum RestaurantError {
        InventoryError(String),
        PreparationError(String),
        ServiceError(String),
        QualityError(String),
    }

    impl std::fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
            match self {
                RestaurantError::InventoryError(msg) => write!(f, "Inventory: {}", msg),
                RestaurantError::PreparationError(msg) => write!(f, "Preparation: {}", msg),
                RestaurantError::ServiceError(msg) => write!(f, "Service: {}", msg),
                RestaurantError::QualityError(msg) => write!(f, "Quality: {}", msg),
            }
        }
    }

    // Restaurant operations using ? operator
    struct Restaurant {
        inventory: HashMap<String, u32>,
        equipment_status: HashMap<String, bool>,
        quality_standards: HashMap<String, f32>,
    }

    impl Restaurant {
        fn new() -> Self {
            let mut inventory = HashMap::new();
            inventory.insert("tomatoes".to_string(), 50);
            inventory.insert("lettuce".to_string(), 30);
            inventory.insert("quinoa".to_string(), 25);
            inventory.insert("cheese".to_string(), 0);  // Out of stock

            let mut equipment_status = HashMap::new();
            equipment_status.insert("oven".to_string(), true);
            equipment_status.insert("mixer".to_string(), true);
            equipment_status.insert("broken_grill".to_string(), false);

            let mut quality_standards = HashMap::new();
            quality_standards.insert("freshness".to_string(), 8.5);
            quality_standards.insert("temperature".to_string(), 165.0);
            quality_standards.insert("presentation".to_string(), 7.0);

            Restaurant {
                inventory,
                equipment_status,
                quality_standards,
            }
        }

        // Level 1: Basic operations that can fail
        fn check_inventory(&self, item: &str, needed: u32) -> Result<(), RestaurantError> {
            let available = self.inventory.get(item).unwrap_or(&0);

            if *available >= needed {
                Ok(())
            } else {
                Err(RestaurantError::InventoryError(
                    format!("Need {} {}, only {} available", needed, item, available)
                ))
            }
        }

        fn check_equipment(&self, equipment: &str) -> Result<(), RestaurantError> {
            match self.equipment_status.get(equipment) {
                Some(true) => Ok(()),
                Some(false) => Err(RestaurantError::PreparationError(
                    format!("Equipment '{}' is out of order", equipment)
                )),
                None => Err(RestaurantError::PreparationError(
                    format!("Equipment '{}' not found", equipment)
                )),
            }
        }

        fn check_quality_standard(&self, standard: &str, value: f32) -> Result<(), RestaurantError> {
            match self.quality_standards.get(standard) {
                Some(&threshold) => {
                    if value >= threshold {
                        Ok(())
                    } else {
                        Err(RestaurantError::QualityError(
                            format!("{} {} below standard {}", standard, value, threshold)
                        ))
                    }
                }
                None => Err(RestaurantError::QualityError(
                    format!("Unknown quality standard: {}", standard)
                )),
            }
        }

        // Level 2: Ingredient preparation using ? operator
        fn prepare_ingredient(&self, ingredient: &str, quantity: u32) -> Result<String, RestaurantError> {
            println!("     🔍 Preparing {} units of {}", quantity, ingredient);

            // ? operator automatically propagates inventory errors
            self.check_inventory(ingredient, quantity)?;

            // ? operator automatically propagates quality errors
            let freshness_score = match ingredient {
                "tomatoes" => 9.0,
                "lettuce" => 8.0,
                "bad_ingredient" => 5.0,  // Below standard
                _ => 8.5,
            };

            self.check_quality_standard("freshness", freshness_score)?;

            Ok(format!("Prepared {} {}", quantity, ingredient))
        }

        // Level 3: Dish preparation using ? operator
        fn prepare_dish(&self, dish_name: &str, recipe: &[(&str, u32)]) -> Result<String, RestaurantError> {
            println!("   🍽️ Preparing dish: {}", dish_name);

            let mut prepared_ingredients = Vec::new();

            // ? operator propagates errors from each ingredient preparation
            for (ingredient, quantity) in recipe {
                let prep_result = self.prepare_ingredient(ingredient, *quantity)?;
                prepared_ingredients.push(prep_result);
            }

            // Check required equipment based on dish type
            let required_equipment = match dish_name {
                "Grilled Salad" => "broken_grill",  // This will fail
                "Baked Quinoa" => "oven",
                "Mixed Salad" => "mixer",
                _ => "oven",
            };

            // ? operator propagates equipment errors
            self.check_equipment(required_equipment)?;

            Ok(format!("Dish '{}' completed with: {}",
                      dish_name,
                      prepared_ingredients.join(", ")))
        }

        // Level 4: Order processing using ? operator
        fn process_order(&self, customer: &str, dishes: &[(&str, &[(&str, u32)])]) -> Result<String, RestaurantError> {
            println!(" 📋 Processing order for customer: {}", customer);

            if customer.is_empty() {
                return Err(RestaurantError::ServiceError(
                    "Customer name required".to_string()
                ));
            }

            let mut completed_dishes = Vec::new();

            // ? operator propagates errors from each dish preparation
            for (dish_name, recipe) in dishes {
                let dish_result = self.prepare_dish(dish_name, recipe)?;
                completed_dishes.push(dish_result);
            }

            // Final quality check
            if completed_dishes.len() > 5 {
                return Err(RestaurantError::ServiceError(
                    "Order too large for current capacity".to_string()
                ));
            }

            Ok(format!("Order completed for {}: {} dishes ready",
                      customer, completed_dishes.len()))
        }

        // Level 5: Complete restaurant operation chain
        fn handle_customer_request(&self, customer: &str, special_requests: &[&str]) -> Result<String, RestaurantError> {
            println!("🏪 Handling customer request from: {}", customer);

            // Validate special requests
            for request in special_requests {
                if request.contains("impossible") {
                    return Err(RestaurantError::ServiceError(
                        format!("Cannot accommodate request: {}", request)
                    ));
                }
            }

            // Standard order processing
            let standard_order = [
                ("House Salad", &[("lettuce", 2), ("tomatoes", 3)][..]),
                ("Quinoa Bowl", &[("quinoa", 5), ("tomatoes", 2)][..]),
            ];

            // ? operator propagates any errors from order processing
            let order_result = self.process_order(customer, &standard_order)?;

            Ok(format!("Customer request fulfilled: {}", order_result))
        }
    }

    println!("🧪 Testing ? Operator Error Propagation:");

    let restaurant = Restaurant::new();

    // Test successful propagation (no errors)
    println!("\n1️⃣ Successful Operation (No Errors to Propagate):");
    match restaurant.handle_customer_request("Alice", &[]) {
        Ok(result) => println!("   🎉 SUCCESS: {}", result),
        Err(error) => println!("   ❌ UNEXPECTED ERROR: {}", error),
    }

    // Test error propagation at different levels
    println!("\n2️⃣ Error Propagation from Inventory Level:");
    let cheese_order = [("Cheese Dish", &[("cheese", 5)][..])];  // Out of stock

    match restaurant.process_order("Bob", &cheese_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error propagated from inventory: {}", error);
            println!("   📊 Error flow: inventory check → ingredient prep → dish prep → order processing");
        }
    }

    // Test equipment failure propagation
    println!("\n3️⃣ Error Propagation from Equipment Level:");
    let grilled_order = [("Grilled Salad", &[("lettuce", 2)][..])];  // Broken grill

    match restaurant.process_order("Carol", &grilled_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error propagated from equipment: {}", error);
            println!("   📊 Error flow: equipment check → dish prep → order processing");
        }
    }

    // Test quality standard propagation
    println!("\n4️⃣ Error Propagation from Quality Level:");
    let bad_order = [("Bad Dish", &[("bad_ingredient", 1)][..])];  // Below quality standard

    match restaurant.process_order("Diana", &bad_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error propagated from quality check: {}", error);
            println!("   📊 Error flow: quality check → ingredient prep → dish prep → order processing");
        }
    }

    // Test service level error
    println!("\n5️⃣ Error Propagation from Service Level:");
    match restaurant.handle_customer_request("Emma", &["impossible_request"]) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error at service level: {}", error);
            println!("   📊 Error caught at highest level before processing began");
        }
    }

    // Compare ? operator vs manual propagation
    println!("\n📊 ? Operator vs Manual Comparison:");

    // Manual version (what ? operator replaces)
    fn manual_chain_example() -> Result<String, RestaurantError> {
        let restaurant = Restaurant::new();

        match restaurant.check_inventory("tomatoes", 5) {
            Ok(()) => {
                match restaurant.check_equipment("oven") {
                    Ok(()) => {
                        match restaurant.check_quality_standard("freshness", 9.0) {
                            Ok(()) => Ok("Manual chain completed".to_string()),
                            Err(e) => Err(e),
                        }
                    }
                    Err(e) => Err(e),
                }
            }
            Err(e) => Err(e),
        }
    }

    // ? operator version (clean and elegant)
    fn question_mark_chain_example() -> Result<String, RestaurantError> {
        let restaurant = Restaurant::new();

        restaurant.check_inventory("tomatoes", 5)?;
        restaurant.check_equipment("oven")?;
        restaurant.check_quality_standard("freshness", 9.0)?;

        Ok("? operator chain completed".to_string())
    }

    println!("   🔧 Manual version: Verbose, nested, error-prone");
    println!("   ❓ ? operator: Clean, linear, maintainable");
    println!("   ⚡ Both approaches produce identical error propagation behavior");

    println!("\n🎯 ? Operator Benefits:");
    println!("   ⚡ Automatic error propagation with minimal syntax");
    println!("   📝 Clean, linear code flow");
    println!("   🔄 Consistent error handling patterns");
    println!("   🛡️ Type-safe error propagation");
    println!("   🎯 Early termination on first error");
    println!("   📊 Preserved error context and information");

    println!("\n💡 How ? Operator Works:");
    println!("   1. 🔍 Evaluates the Result expression");
    println!("   2. ✅ If Ok(value) → extracts value, continues execution");
    println!("   3. ❌ If Err(error) → immediately returns Err(error)");
    println!("   4. 🔄 Automatically converts error types using From trait");
    println!("   5. ⚡ Skips remaining code in function on error");
}

fn main() {
    demonstrate_question_mark_propagation();
}
```


### 3. Error Conversion and Type Transformation

**Unified communication systems - like having universal translation protocols:**

```rust
fn demonstrate_error_conversion_propagation() {
    println!("🔄 Error Conversion Propagation - Universal Translation Systems");
    println!("{:=<70}", "");

    use std::collections::HashMap;
    use std::fmt;
    use std::error::Error;

    // Different restaurant subsystems with their own error types
    #[derive(Debug)]
    enum KitchenError {
        IngredientMissing(String),
        EquipmentBroken(String),
        RecipeNotFound(String),
    }

    #[derive(Debug)]
    enum ServiceError {
        TableUnavailable,
        StaffShortage(u32),
        CustomerComplaint(String),
    }

    #[derive(Debug)]
    enum InventoryError {
        OutOfStock(String),
        SupplierIssue(String),
        CountingError(String),
    }

    #[derive(Debug)]
    enum PaymentError {
        CardDeclined,
        InsufficientFunds(f64),
        ProcessingError(String),
    }

    // Unified restaurant error type that can represent any subsystem error
    #[derive(Debug)]
    enum RestaurantError {
        Kitchen(KitchenError),
        Service(ServiceError),
        Inventory(InventoryError),
        Payment(PaymentError),
        GeneralError(String),
    }

    // Implement Display for all error types
    impl fmt::Display for KitchenError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                KitchenError::IngredientMissing(item) => write!(f, "Missing ingredient: {}", item),
                KitchenError::EquipmentBroken(equipment) => write!(f, "Broken equipment: {}", equipment),
                KitchenError::RecipeNotFound(dish) => write!(f, "Recipe not found: {}", dish),
            }
        }
    }

    impl fmt::Display for ServiceError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                ServiceError::TableUnavailable => write!(f, "No tables available"),
                ServiceError::StaffShortage(needed) => write!(f, "Need {} more staff members", needed),
                ServiceError::CustomerComplaint(issue) => write!(f, "Customer complaint: {}", issue),
            }
        }
    }

    impl fmt::Display for InventoryError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                InventoryError::OutOfStock(item) => write!(f, "Out of stock: {}", item),
                InventoryError::SupplierIssue(supplier) => write!(f, "Supplier issue: {}", supplier),
                InventoryError::CountingError(details) => write!(f, "Counting error: {}", details),
            }
        }
    }

    impl fmt::Display for PaymentError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                PaymentError::CardDeclined => write!(f, "Credit card declined"),
                PaymentError::InsufficientFunds(amount) => write!(f, "Insufficient funds: need ${:.2}", amount),
                PaymentError::ProcessingError(details) => write!(f, "Payment processing error: {}", details),
            }
        }
    }

    impl fmt::Display for RestaurantError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                RestaurantError::Kitchen(e) => write!(f, "Kitchen Error: {}", e),
                RestaurantError::Service(e) => write!(f, "Service Error: {}", e),
                RestaurantError::Inventory(e) => write!(f, "Inventory Error: {}", e),
                RestaurantError::Payment(e) => write!(f, "Payment Error: {}", e),
                RestaurantError::GeneralError(msg) => write!(f, "General Error: {}", msg),
            }
        }
    }

    // Implement Error trait
    impl Error for KitchenError {}
    impl Error for ServiceError {}
    impl Error for InventoryError {}
    impl Error for PaymentError {}
    impl Error for RestaurantError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            match self {
                RestaurantError::Kitchen(e) => Some(e),
                RestaurantError::Service(e) => Some(e),
                RestaurantError::Inventory(e) => Some(e),
                RestaurantError::Payment(e) => Some(e),
                RestaurantError::GeneralError(_) => None,
            }
        }
    }

    // Automatic error conversion using From trait
    impl From<KitchenError> for RestaurantError {
        fn from(error: KitchenError) -> Self {
            RestaurantError::Kitchen(error)
        }
    }

    impl From<ServiceError> for RestaurantError {
        fn from(error: ServiceError) -> Self {
            RestaurantError::Service(error)
        }
    }

    impl From<InventoryError> for RestaurantError {
        fn from(error: InventoryError) -> Self {
            RestaurantError::Inventory(error)
        }
    }

    impl From<PaymentError> for RestaurantError {
        fn from(error: PaymentError) -> Self {
            RestaurantError::Payment(error)
        }
    }

    // Convert from standard library errors
    impl From<std::num::ParseFloatError> for RestaurantError {
        fn from(error: std::num::ParseFloatError) -> Self {
            RestaurantError::GeneralError(format!("Number parsing error: {}", error))
        }
    }

    // Subsystem implementations
    struct KitchenSystem;
    struct ServiceSystem;
    struct InventorySystem;
    struct PaymentSystem;

    impl KitchenSystem {
        fn check_ingredient(&self, ingredient: &str) -> Result<String, KitchenError> {
            match ingredient {
                "tomatoes" => Ok("Fresh tomatoes available".to_string()),
                "quinoa" => Ok("Quinoa in stock".to_string()),
                "missing_ingredient" => Err(KitchenError::IngredientMissing(ingredient.to_string())),
                _ => Ok(format!("{} available", ingredient)),
            }
        }

        fn check_equipment(&self, equipment: &str) -> Result<String, KitchenError> {
            match equipment {
                "oven" => Ok("Oven operational".to_string()),
                "broken_mixer" => Err(KitchenError::EquipmentBroken("Mixer motor failed".to_string())),
                _ => Ok(format!("{} working", equipment)),
            }
        }

        fn find_recipe(&self, dish: &str) -> Result<String, KitchenError> {
            match dish {
                "salad" => Ok("Salad recipe loaded".to_string()),
                "unknown_dish" => Err(KitchenError::RecipeNotFound(dish.to_string())),
                _ => Ok(format!("{} recipe found", dish)),
            }
        }
    }

    impl ServiceSystem {
        fn check_table_availability(&self) -> Result<String, ServiceError> {
            // Simulate full restaurant
            Err(ServiceError::TableUnavailable)
        }

        fn check_staff_level(&self, required: u32) -> Result<String, ServiceError> {
            let available = 3;
            if available >= required {
                Ok(format!("{} staff members available", available))
            } else {
                Err(ServiceError::StaffShortage(required - available))
            }
        }
    }

    impl InventorySystem {
        fn check_stock(&self, item: &str) -> Result<u32, InventoryError> {
            match item {
                "lettuce" => Ok(50),
                "out_of_stock_item" => Err(InventoryError::OutOfStock(item.to_string())),
                _ => Ok(25),
            }
        }
    }

    impl PaymentSystem {
        fn process_payment(&self, amount: f64, method: &str) -> Result<String, PaymentError> {
            match method {
                "declined_card" => Err(PaymentError::CardDeclined),
                "insufficient_card" => Err(PaymentError::InsufficientFunds(amount)),
                "valid_card" => Ok(format!("Payment of ${:.2} processed", amount)),
                _ => Err(PaymentError::ProcessingError("Unknown payment method".to_string())),
            }
        }
    }

    // Main restaurant operation that uses error conversion
    fn process_complete_order(
        customer: &str,
        dish: &str,
        payment_method: &str,
        amount_str: &str
    ) -> Result<String, RestaurantError> {
        println!("   📋 Processing complete order for: {}", customer);

        let kitchen = KitchenSystem;
        let service = ServiceSystem;
        let inventory = InventorySystem;
        let payment = PaymentSystem;

        // Each ? automatically converts specific errors to RestaurantError

        // Kitchen operations (KitchenError → RestaurantError)
        kitchen.find_recipe(dish)?;
        println!("     ✅ Recipe found");

        kitchen.check_ingredient("tomatoes")?;
        println!("     ✅ Ingredients checked");

        kitchen.check_equipment("oven")?;
        println!("     ✅ Equipment verified");

        // Service operations (ServiceError → RestaurantError)
        service.check_staff_level(2)?;
        println!("     ✅ Staff availability confirmed");

        // Skip table check for this example as it always fails
        // service.check_table_availability()?;

        // Inventory operations (InventoryError → RestaurantError)
        let stock_level = inventory.check_stock("lettuce")?;
        println!("     ✅ Inventory checked: {} units", stock_level);

        // Payment operations (PaymentError → RestaurantError)
        let amount: f64 = amount_str.parse()?;  // ParseFloatError → RestaurantError
        let payment_result = payment.process_payment(amount, payment_method)?;
        println!("     ✅ Payment processed: {}", payment_result);

        Ok(format!("Order completed for {}: {} (${:.2})", customer, dish, amount))
    }

    // Error analysis helper
    fn analyze_error_source(error: &RestaurantError) -> (&'static str, &'static str) {
        match error {
            RestaurantError::Kitchen(_) => ("Kitchen", "Contact head chef"),
            RestaurantError::Service(_) => ("Service", "Contact restaurant manager"),
            RestaurantError::Inventory(_) => ("Inventory", "Contact supply manager"),
            RestaurantError::Payment(_) => ("Payment", "Contact payment processor"),
            RestaurantError::GeneralError(_) => ("General", "Contact system administrator"),
        }
    }

    println!("🧪 Testing Error Conversion Propagation:");

    let test_scenarios = [
        ("Success Case", "Alice", "salad", "valid_card", "25.50"),
        ("Kitchen Recipe Error", "Bob", "unknown_dish", "valid_card", "20.00"),
        ("Kitchen Equipment Error", "Carol", "salad", "valid_card", "15.00"),  // Would need broken_mixer test
        ("Service Staff Error", "Diana", "salad", "valid_card", "30.00"),     // Would need staff shortage
        ("Inventory Error", "Emma", "salad", "valid_card", "18.00"),          // Would need out_of_stock test
        ("Payment Card Error", "Frank", "salad", "declined_card", "22.00"),
        ("Payment Insufficient Error", "Grace", "salad", "insufficient_card", "35.00"),
        ("Parse Error", "Henry", "salad", "valid_card", "invalid_amount"),
    ];

    for (scenario_name, customer, dish, payment_method, amount) in test_scenarios {
        println!("\n🔍 Testing: {}", scenario_name);

        match process_complete_order(customer, dish, payment_method, amount) {
            Ok(result) => {
                println!("   🎉 SUCCESS: {}", result);
            }
            Err(error) => {
                println!("   ❌ ERROR: {}", error);

                let (subsystem, action) = analyze_error_source(&error);
                println!("   📊 Error originated from: {} subsystem", subsystem);
                println!("   🔧 Recommended action: {}", action);

                // Show error chain if available
                if let Some(source) = error.source() {
                    println!("   🔗 Root cause: {}", source);
                }
            }
        }
    }

    // Demonstrate manual error conversion vs automatic
    println!("\n📊 Manual vs Automatic Error Conversion:");

    // Manual conversion (what From trait replaces)
    fn manual_conversion_example() -> Result<String, RestaurantError> {
        let kitchen = KitchenSystem;

        match kitchen.check_ingredient("missing_ingredient") {
            Ok(result) => Ok(result),
            Err(kitchen_error) => Err(RestaurantError::Kitchen(kitchen_error)),  // Manual conversion
        }
    }

    // Automatic conversion (using From trait and ?)
    fn automatic_conversion_example() -> Result<String, RestaurantError> {
        let kitchen = KitchenSystem;

        let result = kitchen.check_ingredient("missing_ingredient")?;  // Automatic conversion
        Ok(result)
    }

    println!("   🔧 Manual: Explicit conversion required for each error");
    println!("   ⚡ Automatic: ? operator handles conversion seamlessly");

    println!("\n🎯 Error Conversion Benefits:");
    println!("   🔄 Unified error handling at application boundaries");
    println!("   ⚡ Automatic conversion with From trait implementation");
    println!("   🎯 Clean ? operator usage across different subsystems");
    println!("   📊 Preserved error context and source information");
    println!("   🏗️ Scalable error handling for complex systems");

    println!("\n💡 Error Conversion Best Practices:");
    println!("   🔄 Implement From trait for automatic conversions");
    println!("   🎯 Design error hierarchies that match system architecture");
    println!("   📊 Preserve error source information for debugging");
    println!("   ⚡ Use ? operator for clean error propagation");
    println!("   🏗️ Create unified error types for application boundaries");
}

fn main() {
    demonstrate_error_conversion_propagation();
}
```


## Advanced Error Propagation Patterns

### 1. Chain Operations with Context Enhancement

**Building sophisticated error trails - like detailed incident reports:**

```rust
fn demonstrate_contextual_error_propagation() {
    println!("📋 Contextual Error Propagation - Detailed Incident Reporting");
    println!("{:=<70}", "");

    use std::collections::HashMap;
    use std::fmt;
    use std::error::Error;

    // Enhanced error types with context
    #[derive(Debug)]
    struct ErrorContext {
        operation: String,
        location: String,
        timestamp: String,
        additional_info: HashMap<String, String>,
    }

    impl ErrorContext {
        fn new(operation: &str, location: &str) -> Self {
            ErrorContext {
                operation: operation.to_string(),
                location: location.to_string(),
                timestamp: "2024-01-15T14:30:00Z".to_string(), // Simplified
                additional_info: HashMap::new(),
            }
        }

        fn with_info(mut self, key: &str, value: &str) -> Self {
            self.additional_info.insert(key.to_string(), value.to_string());
            self
        }
    }

    #[derive(Debug)]
    struct ContextualError {
        error_type: String,
        message: String,
        context: ErrorContext,
        source: Option<Box<dyn Error>>,
    }

    impl fmt::Display for ContextualError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            write!(f, "[{}] {} in {}: {}",
                   self.context.timestamp,
                   self.error_type,
                   self.context.location,
                   self.message)
        }
    }

    impl Error for ContextualError {
        fn source(&self) -> Option<&(dyn Error + 'static)> {
            self.source.as_ref().map(|e| e.as_ref())
        }
    }

    impl ContextualError {
        fn new(error_type: &str, message: &str, context: ErrorContext) -> Self {
            ContextualError {
                error_type: error_type.to_string(),
                message: message.to_string(),
                context,
                source: None,
            }
        }

        fn with_source<E: Error + 'static>(mut self, source: E) -> Self {
            self.source = Some(Box::new(source));
            self
        }
    }

    // Restaurant operations with context enhancement
    struct RestaurantOperation {
        operation_id: String,
    }

    impl RestaurantOperation {
        fn new(operation_id: &str) -> Self {
            RestaurantOperation {
                operation_id: operation_id.to_string(),
            }
        }

        // Level 1: Basic operations with context
        fn validate_ingredient(&self, ingredient: &str, quality_score: f32) -> Result<String, ContextualError> {
            let context = ErrorContext::new("ingredient_validation", "kitchen_prep_station")
                .with_info("operation_id", &self.operation_id)
                .with_info("ingredient", ingredient)
                .with_info("quality_score", &quality_score.to_string());

            if quality_score < 7.0 {
                return Err(ContextualError::new(
                    "QUALITY_FAILURE",
                    &format!("Ingredient {} quality score {:.1} below minimum 7.0", ingredient, quality_score),
                    context
                ));
            }

            Ok(format!("Ingredient {} validated (quality: {:.1})", ingredient, quality_score))
        }

        fn check_equipment(&self, equipment: &str) -> Result<String, ContextualError> {
            let context = ErrorContext::new("equipment_check", "kitchen_equipment_bay")
                .with_info("operation_id", &self.operation_id)
                .with_info("equipment", equipment);

            match equipment {
                "broken_oven" => Err(ContextualError::new(
                    "EQUIPMENT_FAILURE",
                    "Oven temperature sensor malfunction",
                    context
                )),
                "maintenance_grill" => Err(ContextualError::new(
                    "EQUIPMENT_UNAVAILABLE",
                    "Grill undergoing scheduled maintenance",
                    context
                )),
                _ => Ok(format!("Equipment {} operational", equipment)),
            }
        }

        // Level 2: Operations with context propagation and enhancement
        fn prepare_base_ingredients(&self, ingredients: &[(&str, f32)]) -> Result<Vec<String>, ContextualError> {
            let context = ErrorContext::new("base_ingredient_prep", "prep_kitchen")
                .with_info("operation_id", &self.operation_id)
                .with_info("ingredient_count", &ingredients.len().to_string());

            let mut prepared = Vec::new();

            for (ingredient, quality) in ingredients {
                // Add context to propagated errors
                let ingredient_result = self.validate_ingredient(ingredient, *quality)
                    .map_err(|e| ContextualError::new(
                        "PREP_STAGE_FAILURE",
                        &format!("Base ingredient preparation failed for {}", ingredient),
                        context.clone()
                    ).with_source(e))?;

                prepared.push(ingredient_result);
            }

            Ok(prepared)
        }

        fn setup_cooking_station(&self, required_equipment: &[&str]) -> Result<String, ContextualError> {
            let context = ErrorContext::new("cooking_station_setup", "main_kitchen")
                .with_info("operation_id", &self.operation_id)
                .with_info("equipment_count", &required_equipment.len().to_string());

            for equipment in required_equipment {
                // Enhance context for equipment errors
                self.check_equipment(equipment)
                    .map_err(|e| ContextualError::new(
                        "STATION_SETUP_FAILURE",
                        &format!("Cooking station setup failed due to {}", equipment),
                        context.clone()
                    ).with_source(e))?;
            }

            Ok(format!("Cooking station ready with {} pieces of equipment", required_equipment.len()))
        }

        // Level 3: Complex operations with full context chain
        fn execute_recipe(&self, recipe_name: &str, ingredients: &[(&str, f32)], equipment: &[&str]) -> Result<String, ContextualError> {
            let context = ErrorContext::new("recipe_execution", "main_kitchen")
                .with_info("operation_id", &self.operation_id)
                .with_info("recipe", recipe_name)
                .with_info("complexity", "high");

            println!("     🔍 Executing recipe: {}", recipe_name);

            // Step 1: Prepare ingredients with context enhancement
            let _prepared_ingredients = self.prepare_base_ingredients(ingredients)
                .map_err(|e| ContextualError::new(
                    "RECIPE_PREP_FAILURE",
                    &format!("Recipe {} failed during ingredient preparation", recipe_name),
                    context.clone()
                ).with_source(e))?;

            println!("       ✅ Ingredients prepared");

            // Step 2: Setup cooking station with context enhancement
            let _station_status = self.setup_cooking_station(equipment)
                .map_err(|e| ContextualError::new(
                    "RECIPE_EQUIPMENT_FAILURE",
                    &format!("Recipe {} failed during station setup", recipe_name),
                    context.clone()
                ).with_source(e))?;

            println!("       ✅ Cooking station ready");

            // Step 3: Recipe-specific validation
            if recipe_name == "impossible_recipe" {
                return Err(ContextualError::new(
                    "RECIPE_NOT_FOUND",
                    "Recipe not available in current menu",
                    context
                ));
            }

            Ok(format!("Recipe {} executed successfully", recipe_name))
        }

        // Level 4: Full order processing with comprehensive context
        fn process_full_order(&self, customer: &str, orders: &[(&str, &[(&str, f32)], &[&str])]) -> Result<String, ContextualError> {
            let context = ErrorContext::new("full_order_processing", "restaurant_main")
                .with_info("operation_id", &self.operation_id)
                .with_info("customer", customer)
                .with_info("order_count", &orders.len().to_string());

            if customer.is_empty() {
                return Err(ContextualError::new(
                    "CUSTOMER_INFO_MISSING",
                    "Customer identification required for order processing",
                    context
                ));
            }

            println!("   📋 Processing full order for customer: {}", customer);

            let mut completed_dishes = Vec::new();

            for (recipe_name, ingredients, equipment) in orders {
                // Execute each recipe with full context propagation
                let dish_result = self.execute_recipe(recipe_name, ingredients, equipment)
                    .map_err(|e| ContextualError::new(
                        "ORDER_DISH_FAILURE",
                        &format!("Order processing failed on dish: {}", recipe_name),
                        context.clone()
                    ).with_source(e))?;

                completed_dishes.push(dish_result);
            }

            if completed_dishes.len() > 8 {
                return Err(ContextualError::new(
                    "ORDER_SIZE_EXCEEDED",
                    &format!("Order size {} exceeds kitchen capacity of 8 dishes", completed_dishes.len()),
                    context
                ));
            }

            Ok(format!("Full order completed for {}: {} dishes prepared", customer, completed_dishes.len()))
        }
    }

    // Error chain analysis helper
    fn analyze_error_chain(error: &ContextualError, depth: usize) {
        let indent = "  ".repeat(depth);
        println!("{}📋 Error Level {}: {}", indent, depth, error.error_type);
        println!("{}   Message: {}", indent, error.message);
        println!("{}   Operation: {}", indent, error.context.operation);
        println!("{}   Location: {}", indent, error.context.location);

        if !error.context.additional_info.is_empty() {
            println!("{}   Additional Info:", indent);
            for (key, value) in &error.context.additional_info {
                println!("{}     {}: {}", indent, key, value);
            }
        }

        if let Some(source) = error.source() {
            if let Some(contextual_source) = source.downcast_ref::<ContextualError>() {
                println!("{}   ⬇️ Caused by:", indent);
                analyze_error_chain(contextual_source, depth + 1);
            } else {
                println!("{}   Root cause: {}", indent, source);
            }
        }
    }

    println!("🧪 Testing Contextual Error Propagation:");

    let operation = RestaurantOperation::new("OP_12345");

    // Test successful operation
    println!("\n1️⃣ Successful Operation with Context:");
    let good_order = [
        ("Quinoa Salad", &[("quinoa", 8.5), ("tomatoes", 9.0)][..], &["prep_station"][..]),
    ];

    match operation.process_full_order("Alice", &good_order) {
        Ok(result) => println!("   🎉 SUCCESS: {}", result),
        Err(error) => {
            println!("   ❌ UNEXPECTED ERROR:");
            analyze_error_chain(&error, 0);
        }
    }

    // Test error propagation with context enhancement
    println!("\n2️⃣ Error Propagation with Context Chain:");
    let quality_failure_order = [
        ("Problematic Dish", &[("quinoa", 8.5), ("bad_tomatoes", 5.0)][..], &["prep_station"][..]),
    ];

    match operation.process_full_order("Bob", &quality_failure_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Error with Full Context Chain:");
            analyze_error_chain(&error, 0);
        }
    }

    // Test equipment failure propagation
    println!("\n3️⃣ Equipment Failure with Context:");
    let equipment_failure_order = [
        ("Baked Dish", &[("flour", 8.0)][..], &["broken_oven"][..]),
    ];

    match operation.process_full_order("Carol", &equipment_failure_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Equipment Error with Context:");
            analyze_error_chain(&error, 0);
        }
    }

    // Test missing customer information
    println!("\n4️⃣ Service Level Error:");
    let normal_order = [
        ("Simple Dish", &[("ingredient", 8.0)][..], &["prep_station"][..]),
    ];

    match operation.process_full_order("", &normal_order) {
        Ok(result) => println!("   ✅ Unexpected success: {}", result),
        Err(error) => {
            println!("   ❌ Service Error:");
            analyze_error_chain(&error, 0);
        }
    }

    println!("\n🎯 Contextual Error Propagation Benefits:");
    println!("   📋 Rich context preserved through error chain");
    println!("   🔗 Complete error causation history");
    println!("   📊 Detailed debugging information at each level");
    println!("   🎯 Operation tracking and correlation");
    println!("   ⏰ Timestamp information for incident analysis");
    println!("   🏗️ Hierarchical error understanding");

    println!("\n💡 Context Enhancement Patterns:");
    println!("   🔄 Add context at each error handling level");
    println!("   📋 Include operation-specific information");
    println!("   🔗 Chain errors to preserve causation");
    println!("   📊 Enhance context without losing original error");
    println!("   🎯 Design context for operational requirements");
}

fn main() {
    demonstrate_contextual_error_propagation();
}
```


## Best Practices and Professional Guidelines

### Professional Error Propagation Design

**Essential patterns for building robust error propagation systems:**

```rust
fn demonstrate_error_propagation_best_practices() {
    println!("✅ Error Propagation Best Practices - Professional Guidelines");
    println!("{:=<70}", "");

    use std::fmt;
    use std::error::Error;
    use std::collections::HashMap;

    // Best Practice 1: Design error hierarchies that match your architecture
    println!("🏗️ Best Practice 1: Architecture-Aligned Error Hierarchies");

    // ✅ Good: Error structure matches system architecture
    #[derive(Debug)]
    enum RestaurantSystemError {
        // Front-of-house errors (customer-facing)
        FrontOfHouse(FrontOfHouseError),

        // Back-of-house errors (kitchen and prep)
        BackOfHouse(BackOfHouseError),

        // Infrastructure errors (systems and equipment)
        Infrastructure(InfrastructureError),

        // Business logic errors (rules and policies)
        BusinessLogic(BusinessLogicError),
    }

    #[derive(Debug)]
    enum FrontOfHouseError {
        ServiceIssue { table: u32, issue: String },
        CustomerComplaint { customer_id: String, details: String },
        ReservationConflict { time: String, party_size: u32 },
    }

    #[derive(Debug)]
    enum BackOfHouseError {
        IngredientIssue { ingredient: String, problem: String },
        CookingFailure { dish: String, stage: String },
        KitchenEquipment { equipment: String, error_code: u32 },
    }

    #[derive(Debug)]
    enum InfrastructureError {
        DatabaseConnection(String),
        NetworkTimeout { operation: String, duration_ms: u32 },
        SystemOverload { cpu_percent: f32, memory_percent: f32 },
    }

    #[derive(Debug)]
    enum BusinessLogicError {
        PolicyViolation { policy: String, violation: String },
        CapacityExceeded { requested: u32, available: u32 },
        InvalidOperation { operation: String, reason: String },
    }

    // Best Practice 2: Implement consistent error propagation patterns
    impl fmt::Display for RestaurantSystemError {
        fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
            match self {
                RestaurantSystemError::FrontOfHouse(e) => write!(f, "Front of House: {:?}", e),
                RestaurantSystemError::BackOfHouse(e) => write!(f, "Back of House: {:?}", e),
                RestaurantSystemError::Infrastructure(e) => write!(f, "Infrastructure: {:?}", e),
                RestaurantSystemError::BusinessLogic(e) => write!(f, "Business Logic: {:?}", e),
            }
        }
    }

    impl Error for RestaurantSystemError {}

    // Automatic error conversion
    impl From<FrontOfHouseError> for RestaurantSystemError {
        fn from(error: FrontOfHouseError) -> Self {
            RestaurantSystemError::FrontOfHouse(error)
        }
    }

    impl From<BackOfHouseError> for RestaurantSystemError {
        fn from(error: BackOfHouseError) -> Self {
            RestaurantSystemError::BackOfHouse(error)
        }
    }

    impl From<InfrastructureError> for RestaurantSystemError {
        fn from(error: InfrastructureError) -> Self {
            RestaurantSystemError::Infrastructure(error)
        }
    }

    impl From<BusinessLogicError> for RestaurantSystemError {
        fn from(error: BusinessLogicError) -> Self {
            RestaurantSystemError::BusinessLogic(error)
        }
    }

    // Best Practice 3: Use meaningful error context and recovery guidance
    trait ErrorRecovery {
        fn recovery_suggestions(&self) -> Vec<String>;
        fn is_retryable(&self) -> bool;
        fn escalation_level(&self) -> u8; // 1-5, 5 being highest
    }

    impl ErrorRecovery for RestaurantSystemError {
        fn recovery_suggestions(&self) -> Vec<String> {
            match self {
                RestaurantSystemError::FrontOfHouse(_) => vec![
                    "Apologize to customer".to_string(),
                    "Offer compensation if appropriate".to_string(),
                    "Escalate to manager if needed".to_string(),
                ],
                RestaurantSystemError::BackOfHouse(_) => vec![
                    "Check ingredient availability".to_string(),
                    "Use alternative cooking method".to_string(),
                    "Contact supplier if ingredient issue".to_string(),
                ],
                RestaurantSystemError::Infrastructure(_) => vec![
                    "Check network connectivity".to_string(),
                    "Restart affected services".to_string(),
                    "Contact IT support".to_string(),
                ],
                RestaurantSystemError::BusinessLogic(_) => vec![
                    "Review business rules".to_string(),
                    "Check for policy updates".to_string(),
                    "Consult with management".to_string(),
                ],
            }
        }

        fn is_retryable(&self) -> bool {
            match self {
                RestaurantSystemError::Infrastructure(_) => true,
                RestaurantSystemError::BackOfHouse(_) => true,
                _ => false,
            }
        }

        fn escalation_level(&self) -> u8 {
            match self {
                RestaurantSystemError::FrontOfHouse(_) => 2,
                RestaurantSystemError::BackOfHouse(_) => 3,
                RestaurantSystemError::Infrastructure(_) => 4,
                RestaurantSystemError::BusinessLogic(_) => 1,
            }
        }
    }

    // Best Practice 4: Strategic error handling and propagation
    struct RestaurantManager {
        error_count: HashMap<String, u32>,
    }

    impl RestaurantManager {
        fn new() -> Self {
            RestaurantManager {
                error_count: HashMap::new(),
            }
        }

        // ✅ Good: Strategic error handling with context
        fn handle_service_request(&mut self, request: &str) -> Result<String, RestaurantSystemError> {
            println!("   📋 Processing service request: {}", request);

            // Use ? for clean error propagation
            let validation_result = self.validate_request(request)?;
            let processing_result = self.process_request(&validation_result)?;
            let completion_result = self.complete_request(processing_result)?;

            Ok(completion_result)
        }

        fn validate_request(&self, request: &str) -> Result<String, RestaurantSystemError> {
            if request.is_empty() {
                return Err(BusinessLogicError::InvalidOperation {
                    operation: "request_validation".to_string(),
                    reason: "Empty request not allowed".to_string(),
                }.into());
            }

            if request.contains("impossible") {
                return Err(BusinessLogicError::PolicyViolation {
                    policy: "service_limitations".to_string(),
                    violation: "Impossible requests not supported".to_string(),
                }.into());
            }

            Ok(format!("Validated request: {}", request))
        }

        fn process_request(&self, validated_request: &str) -> Result<String, RestaurantSystemError> {
            // Simulate different types of processing errors
            if validated_request.contains("kitchen_error") {
                return Err(BackOfHouseError::CookingFailure {
                    dish: "test_dish".to_string(),
                    stage: "preparation".to_string(),
                }.into());
            }

            if validated_request.contains("service_error") {
                return Err(FrontOfHouseError::ServiceIssue {
                    table: 5,
                    issue: "Spilled water on customer".to_string(),
                }.into());
            }

            if validated_request.contains("system_error") {
                return Err(InfrastructureError::DatabaseConnection(
                    "Connection timeout after 30 seconds".to_string()
                ).into());
            }

            Ok(format!("Processed: {}", validated_request))
        }

        fn complete_request(&self, processed_request: String) -> Result<String, RestaurantSystemError> {
            if processed_request.len() > 100 {
                return Err(BusinessLogicError::CapacityExceeded {
                    requested: processed_request.len() as u32,
                    available: 100,
                }.into());
            }

            Ok(format!("Completed: {}", processed_request))
        }

        // Best Practice 5: Error analysis and metrics
        fn handle_error(&mut self, error: &RestaurantSystemError) {
            let error_type = match error {
                RestaurantSystemError::FrontOfHouse(_) => "front_of_house",
                RestaurantSystemError::BackOfHouse(_) => "back_of_house",
                RestaurantSystemError::Infrastructure(_) => "infrastructure",
                RestaurantSystemError::BusinessLogic(_) => "business_logic",
            };

            *self.error_count.entry(error_type.to_string()).or_insert(0) += 1;

            println!("     📊 Error Analysis:");
            println!("       Type: {}", error_type);
            println!("       Escalation Level: {}/5", error.escalation_level());
            println!("       Retryable: {}", error.is_retryable());
            println!("       Recovery Suggestions:");
            for (i, suggestion) in error.recovery_suggestions().iter().enumerate() {
                println!("         {}. {}", i + 1, suggestion);
            }
        }

        fn get_error_summary(&self) -> Vec<(String, u32)> {
            self.error_count.iter()
                .map(|(error_type, &count)| (error_type.clone(), count))
                .collect()
        }
    }

    // Best Practice 6: Comprehensive error propagation testing
    fn test_error_propagation_patterns() {
        println!("\n🧪 Testing Error Propagation Patterns:");

        let mut manager = RestaurantManager::new();

        let test_requests = [
            ("normal_request", "Should succeed"),
            ("", "Empty request error"),
            ("impossible_request", "Policy violation"),
            ("kitchen_error_request", "Kitchen processing error"),
            ("service_error_request", "Service error"),
            ("system_error_request", "Infrastructure error"),
        ];

        for (request, description) in test_requests {
            println!("\n  🔍 Testing: {}", description);

            match manager.handle_service_request(request) {
                Ok(result) => {
                    println!("     ✅ SUCCESS: {}", result);
                }
                Err(error) => {
                    println!("     ❌ ERROR: {}", error);
                    manager.handle_error(&error);
                }
            }
        }

        // Error summary
        println!("\n📊 Error Summary:");
        let error_summary = manager.get_error_summary();
        for (error_type, count) in error_summary {
            println!("   {} errors: {}", error_type, count);
        }
    }

    test_error_propagation_patterns();

    println!("\n📋 Error Propagation Best Practices Summary:");
    println!("   🏗️ Design error hierarchies matching system architecture");
    println!("   🔄 Implement From trait for automatic error conversion");
    println!("   ❓ Use ? operator for clean error propagation");
    println!("   📋 Include rich context and recovery guidance");
    println!("   📊 Implement error analysis and metrics collection");
    println!("   🎯 Strategic error handling based on error types");
    println!("   🔄 Design for retry and recovery scenarios");

    println!("\n💡 Professional Guidelines:");
    println!("   🎯 Match error granularity to operational needs");
    println!("   📊 Include error codes for systematic categorization");
    println!("   🔄 Design error types for monitoring and alerting");
    println!("   📋 Document error propagation paths");
    println!("   ⚡ Consider performance impact of error context");
    println!("   🏗️ Plan error handling strategy from system design phase");
}

fn main() {
    demonstrate_error_propagation_best_practices();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Restaurant Emergency Communication System**

Remember our comprehensive restaurant emergency communication analogy:

- 📡 **Error propagation** = **Professional communication protocols** - Problems flow efficiently to decision-makers
- ⬆️ **Upward flow** = **Management escalation** - Each level passes problems to appropriate authority
- 🔄 **Automatic conversion** = **Translation services** - Different departments speak the same language
- 📋 **Rich context** = **Detailed incident reports** - Complete information travels with problems
- 🎯 **Strategic handling** = **Appropriate responses** - Different problems get different solutions


### **Essential Error Propagation Patterns**

**Manual Propagation Pattern:**

```rust
fn manual_error_propagation() -> Result<String, Error> {
    match risky_operation_1() {
        Ok(result1) => {
            match risky_operation_2(result1) {
                Ok(result2) => {
                    match risky_operation_3(result2) {
                        Ok(final_result) => Ok(final_result),
                        Err(e) => Err(e),  // Manual propagation
                    }
                }
                Err(e) => Err(e),  // Manual propagation
            }
        }
        Err(e) => Err(e),  // Manual propagation
    }
}
```

**? Operator Pattern:**

```rust
fn automatic_error_propagation() -> Result<String, Error> {
    let result1 = risky_operation_1()?;  // Automatic propagation
    let result2 = risky_operation_2(result1)?;  // Automatic propagation
    let final_result = risky_operation_3(result2)?;  // Automatic propagation
    Ok(final_result)
}
```

**Error Conversion Pattern:**

```rust
// Automatic conversion with From trait
impl From<SpecificError> for GeneralError {
    fn from(error: SpecificError) -> Self {
        GeneralError::Specific(error)
    }
}

fn unified_error_handling() -> Result<String, GeneralError> {
    let result1 = specific_operation_1()?;  // SpecificError → GeneralError
    let result2 = specific_operation_2(result1)?;  // AnotherError → GeneralError
    Ok(result2)
}
```


### **Error Propagation Decision Matrix**

| **Scenario** | **Pattern** | **Use Case** | **Benefits** |
| :-- | :-- | :-- | :-- |
| **Learning/Debugging** | Manual with match | Understanding error flow | Full visibility and control |
| **Production Code** | ? operator | Clean error propagation | Concise, maintainable code |
| **Multi-system** | Error conversion | Unified error handling | Consistent error interface |
| **Enterprise** | Contextual errors | Rich error information | Detailed debugging and monitoring |

### **Best Practices Checklist**

**✅ DO:**

- Use ? operator as primary propagation mechanism[^1][^2]
- Design error hierarchies matching system architecture
- Implement From trait for automatic error conversion
- Include rich context in error types
- Document error propagation paths
- Plan error handling strategy during design phase
- Use appropriate error granularity for operational needs

**❌ DON'T:**

- Ignore errors or use unwrap() in production
- Create overly complex error hierarchies
- Lose error context during propagation
- Use generic string errors for structured systems
- Forget to implement standard error traits
- Mix error propagation patterns inconsistently


### **Error Propagation Flow Patterns**

```rust
// Pattern 1: Linear error propagation
fn linear_flow() -> Result<String, Error> {
    let step1 = operation1()?;
    let step2 = operation2(step1)?;
    let step3 = operation3(step2)?;
    Ok(step3)
}

// Pattern 2: Conditional error propagation
fn conditional_flow() -> Result<String, Error> {
    let result = risky_operation()?;

    if result.needs_validation() {
        validate_result(&result)?;
    }

    Ok(result.finalize())
}

// Pattern 3: Parallel error aggregation
fn parallel_flow() -> Result<Vec<String>, Error> {
    let results: Result<Vec<_>, _> = vec!["a", "b", "c"]
        .iter()
        .map(|item| process_item(item))
        .collect();  // Stops at first error

    results
}

// Pattern 4: Error recovery and fallback
fn recovery_flow() -> Result<String, Error> {
    primary_operation()
        .or_else(|_| fallback_operation())
        .or_else(|_| emergency_operation())
}
```


### **Performance Considerations**

**Error Propagation Performance:**

- **? operator**: Near-zero overhead in success case
- **Manual propagation**: Slightly more overhead due to explicit handling
- **Error conversion**: Small cost for type conversion
- **Rich context**: Consider memory and construction costs
- **Error chaining**: Balance detail vs performance


### **The Professional Advantage**

**Mastering error propagation patterns in Rust is like having a world-class restaurant communication system** that ensures problems are handled efficiently and appropriately:

- 📡 **Efficient communication** - Errors reach the right handlers quickly
- 🎯 **Appropriate responses** - Different error types trigger different actions
- 🔄 **Consistent protocols** - Standardized error handling across the system
- 📊 **Rich information** - Complete context travels with errors
- ⚡ **Clean code** - ? operator eliminates boilerplate
- 🛡️ **Type safety** - Compiler ensures no errors are ignored

**Understanding error propagation patterns transforms you from a programmer who struggles with error handling to a systems expert** who builds robust, maintainable Rust applications. Just as a professional restaurant has clear protocols for escalating different types of problems to the appropriate management levels, a skilled Rust programmer uses error propagation patterns to build software that handles failures gracefully and provides clear information about what went wrong and how to fix it.

This comprehensive understanding of error propagation - from basic manual handling through advanced contextual patterns - makes your Rust programs more reliable, your debugging more effective, and your development process more efficient, whether you're building simple applications or complex distributed systems that need to handle thousands of operations with grace and precision![^2][^3][^1]

1. https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
2. https://masteringbackend.com/posts/rust-error-handling-80-20-guide
3. https://dev.to/leapcell/rusts-result-type-error-handling-made-easy-58i
4. https://notes.kodekloud.com/docs/Rust-Programming/Collections-Error-Handling/Creating-Custom-Error-Types
5. https://users.rust-lang.org/t/understanding-control-flow-and-error-propagation-patterns-novice/119061
6. http://essay.utwente.nl/100758/1/kas_BA_EEMCS.pdf
7. https://dzone.com/articles/different-ways-for-error-propagation-in-rust
8. https://blog.logrocket.com/error-handling-rust/
9. https://www.lpalmieri.com/posts/error-handling-rust/
10. https://users.rust-lang.org/t/proper-error-propagation-in-rust/52713
11. https://stackoverflow.com/questions/71104550/propagating-details-of-an-error-upwards-in-rust
12. https://www.reddit.com/r/rust/comments/6konyi/error_propagation_of_different_types/
13. https://www.capitalone.com/tech/software-engineering/rust-error-handling/
14. https://blog.frankel.ch/error-management-rust-libs/
15. https://www.reddit.com/r/rust/comments/1ed42mm/error_propagation_with_context_missing_an_expect/
16. https://dev.to/ajtech0001/mastering-error-handling-in-rust-a-complete-guide-32aj
17. https://www.woodruff.dev/error-propagation-with-so-simple-so-smart/
18. https://stackoverflow.com/questions/68064390/understanding-ok-with-error-propagation-operator
19. https://www.reddit.com/r/rust/comments/1bb7dco/error_handling_goodbest_practices/
20. https://bitfieldconsulting.com/posts/rust-errors-option-result
21. https://learning-rust.github.io/docs/error-and-none-propagation/
22. https://users.rust-lang.org/t/understanding-best-practices-for-propagating-different-errors-in-libraries/120269
23. https://technorely.com/insights/effective-error-handling-in-rust-cli-apps-best-practices-examples-and-advanced-techniques
24. https://dev-state.com/posts/error_handling/
25. https://www.w3resource.com/rust/error_handling/error-propagation.php
26. https://www.rustfinity.com/practice/rust/challenges/error-propagation/result
27. https://www.reddit.com/r/rust/comments/1eudm6a/error_propagation_and_handling/
28. https://stackoverflow.com/questions/78217448/simple-error-handling-for-precondition-argument-checking-in-rust
29. https://andreabergia.com/blog/2023/05/error-handling-patterns/
30. https://www.sheshbabu.com/posts/rust-error-handling/
