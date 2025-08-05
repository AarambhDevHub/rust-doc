+++
title = "Result<T, E> Enum in Rust"
description = "Understand how Rust's Result<T, E> enum is used for error handling and safe propagation of success or failure in operations."
date = 2025-08-05
draft = false
weight = 4
+++

# Result<T, E> Enum in Rust: Comprehensive Documentation for Beginners

Understanding `Result<T, E>` is like learning to **run a professional vegetarian kitchen where every cooking step can either succeed beautifully or fail spectacularly** - and you need to know not just *if* something went wrong, but *exactly why* it failed so you can fix it. Unlike `Option<T>` which simply tells you "something is missing," `Result<T, E>` is your kitchen's incident report system that captures both successes and detailed failure information.

## The Professional Vegetarian Kitchen Analogy 👨🍳

### Imagine You're Head Chef in a Busy Vegetarian Restaurant

**Option<T> Approach (Limited Information):**

```rust
// ❌ Only tells you if something failed, not why
fn cook_dish() -> Option<String> {
    // Either returns Some("Delicious pasta") or None
    // But if None, you don't know what went wrong!
}
```

**Result<T, E> Approach (Full Kitchen Report):**

```rust
// ✅ Tells you exactly what happened - success or detailed failure
fn cook_dish() -> Result<String, CookingError> {
    // Returns either:
    // Ok("Perfect quinoa bowl") - success with the dish
    // Err(CookingError::OutOfIngredient("quinoa")) - specific problem
    // Err(CookingError::OverheatedOven(450)) - another specific problem
}
```

**Why This Is Better:**

- 🎯 **Success details** - Know exactly what you accomplished
- 🔍 **Specific error info** - Understand exactly what went wrong
- 🛠️ **Actionable feedback** - Can take specific steps to fix problems
- 📊 **Kitchen management** - Track patterns in what goes wrong
- 🚫 **No silent failures** - Every problem is captured and must be handled


## What is Result<T, E>?

### Core Concept

`Result<T, E>` is Rust's primary error-handling mechanism. It's an enum with exactly two variants:

```rust
enum Result<T, E> {
    Ok(T),      // Success: contains the successful result of type T
    Err(E),     // Failure: contains the error information of type E
}
```

**Key Benefits:**

- **Explicit error handling** - Must handle both success and failure cases
- **Rich error information** - Errors can carry detailed context
- **No exceptions** - No hidden control flow or crashes
- **Composable** - Can chain operations and propagate errors elegantly
- **Type safe** - Compiler ensures you handle all cases


### The Kitchen Incident System

Think of `Result<T, E>` like a comprehensive kitchen reporting system:

```rust
#[derive(Debug)]
enum CookingError {
    MissingIngredient(String),
    EquipmentFailure(String),
    TemperatureTooHigh(u32),
    TemperatureTooLow(u32),
    TimingError { step: String, expected: u32, actual: u32 },
    ContaminationRisk(String),
}

#[derive(Debug)]
struct Dish {
    name: String,
    ingredients: Vec<String>,
    cook_time_minutes: u32,
}

fn prepare_quinoa_bowl() -> Result<Dish, CookingError> {
    // Check if we have quinoa
    if !pantry_has_ingredient("quinoa") {
        return Err(CookingError::MissingIngredient("quinoa".to_string()));
    }

    // Check if stove is working
    if !equipment_functional("stove") {
        return Err(CookingError::EquipmentFailure("stove not heating".to_string()));
    }

    // Check temperature
    let stove_temp = get_stove_temperature();
    if stove_temp > 400 {
        return Err(CookingError::TemperatureTooHigh(stove_temp));
    }

    // If everything is fine, return success
    Ok(Dish {
        name: "Quinoa Power Bowl".to_string(),
        ingredients: vec!["quinoa".to_string(), "vegetables".to_string()],
        cook_time_minutes: 25,
    })
}

fn pantry_has_ingredient(_ingredient: &str) -> bool { true } // Simplified
fn equipment_functional(_equipment: &str) -> bool { true }   // Simplified
fn get_stove_temperature() -> u32 { 350 }                   // Simplified

fn main() {
    match prepare_quinoa_bowl() {
        Ok(dish) => {
            println!("✅ Success! Made {} in {} minutes", dish.name, dish.cook_time_minutes);
            println!("   Ingredients: {:?}", dish.ingredients);
        }
        Err(CookingError::MissingIngredient(ingredient)) => {
            println!("❌ Can't cook: missing {}", ingredient);
            println!("   Action: Add {} to shopping list", ingredient);
        }
        Err(CookingError::EquipmentFailure(issue)) => {
            println!("❌ Equipment problem: {}", issue);
            println!("   Action: Call maintenance team");
        }
        Err(CookingError::TemperatureTooHigh(temp)) => {
            println!("❌ Stove too hot: {}°F", temp);
            println!("   Action: Lower heat and wait to cool down");
        }
        Err(other_error) => {
            println!("❌ Other cooking error: {:?}", other_error);
        }
    }
}
```

Each kitchen operation tells you exactly what happened - no guessing required!

## Basic Result<T, E> Usage

### Simple Operations with Results

```rust
// Basic arithmetic that can fail
fn safe_divide(numerator: f64, denominator: f64) -> Result<f64, String> {
    if denominator == 0.0 {
        Err("Cannot divide by zero - that's mathematically impossible!".to_string())
    } else if denominator.abs() < f64::EPSILON {
        Err("Denominator too close to zero - results would be unreliable".to_string())
    } else {
        Ok(numerator / denominator)
    }
}

// Parsing ingredients from strings
fn parse_ingredient_amount(input: &str) -> Result<f64, String> {
    input.parse::<f64>().map_err(|_| {
        format!("'{}' is not a valid number - please use format like '2.5'", input)
    })
}

// Converting temperature units
fn fahrenheit_to_celsius(f: f64) -> Result<f64, String> {
    if f < -459.67 {
        Err(format!("Temperature {}°F is below absolute zero!", f))
    } else {
        Ok((f - 32.0) * 5.0 / 9.0)
    }
}

fn main() {
    // Test division
    match safe_divide(10.0, 2.0) {
        Ok(result) => println!("✅ 10 ÷ 2 = {}", result),
        Err(error) => println!("❌ Division failed: {}", error),
    }

    match safe_divide(10.0, 0.0) {
        Ok(result) => println!("✅ 10 ÷ 0 = {}", result),
        Err(error) => println!("❌ Division failed: {}", error),
    }

    // Test parsing
    let amounts = ["2.5", "invalid", "0.75"];
    for amount_str in &amounts {
        match parse_ingredient_amount(amount_str) {
            Ok(amount) => println!("✅ Parsed '{}' as {} cups", amount_str, amount),
            Err(error) => println!("❌ Parse error: {}", error),
        }
    }

    // Test temperature conversion
    let temperatures = [32.0, 100.0, -500.0];
    for temp_f in &temperatures {
        match fahrenheit_to_celsius(*temp_f) {
            Ok(temp_c) => println!("✅ {}°F = {:.1}°C", temp_f, temp_c),
            Err(error) => println!("❌ Conversion error: {}", error),
        }
    }
}
```


### Pattern Matching with Result<T, E>

**The most fundamental way to work with `Result<T, E>` is pattern matching:**

```rust
#[derive(Debug)]
enum RecipeError {
    MissingIngredients(Vec<String>),
    InvalidQuantity(String),
    UnsuitableEquipment(String),
    TimingConflict(String),
}

#[derive(Debug)]
struct Recipe {
    name: String,
    servings: u32,
    estimated_time: u32,
}

fn validate_recipe_request(
    recipe_name: &str,
    requested_servings: u32,
    available_ingredients: &[&str]
) -> Result<Recipe, RecipeError> {
    // Check for valid servings
    if requested_servings == 0 || requested_servings > 20 {
        return Err(RecipeError::InvalidQuantity(
            format!("Cannot make {} servings - range is 1-20", requested_servings)
        ));
    }

    // Check required ingredients based on recipe
    let required_ingredients = match recipe_name {
        "Quinoa Bowl" => vec!["quinoa", "vegetables", "olive oil"],
        "Pasta Primavera" => vec!["pasta", "vegetables", "tomato sauce"],
        "Lentil Soup" => vec!["lentils", "vegetables", "vegetable broth"],
        _ => return Err(RecipeError::UnsuitableEquipment(
            format!("Recipe '{}' not in our cookbook", recipe_name)
        )),
    };

    let missing: Vec<String> = required_ingredients
        .into_iter()
        .filter(|ingredient| !available_ingredients.contains(ingredient))
        .map(|s| s.to_string())
        .collect();

    if !missing.is_empty() {
        return Err(RecipeError::MissingIngredients(missing));
    }

    // Estimate cooking time
    let base_time = match recipe_name {
        "Quinoa Bowl" => 25,
        "Pasta Primavera" => 30,
        "Lentil Soup" => 45,
        _ => unreachable!(), // We already checked above
    };

    let total_time = base_time + (requested_servings - 1) * 5; // Extra time for larger batches

    Ok(Recipe {
        name: recipe_name.to_string(),
        servings: requested_servings,
        estimated_time: total_time,
    })
}

fn process_recipe_request(
    recipe_name: &str,
    servings: u32,
    available_ingredients: &[&str]
) {
    match validate_recipe_request(recipe_name, servings, available_ingredients) {
        Ok(recipe) => {
            println!("✅ Recipe approved!");
            println!("   📖 {}", recipe.name);
            println!("   👥 {} servings", recipe.servings);
            println!("   ⏰ {} minutes estimated", recipe.estimated_time);
        }
        Err(RecipeError::MissingIngredients(missing)) => {
            println!("❌ Cannot make recipe - missing ingredients:");
            for ingredient in missing {
                println!("   🛒 Need: {}", ingredient);
            }
        }
        Err(RecipeError::InvalidQuantity(message)) => {
            println!("❌ Invalid serving size: {}", message);
        }
        Err(RecipeError::UnsuitableEquipment(message)) => {
            println!("❌ Recipe not available: {}", message);
        }
        Err(RecipeError::TimingConflict(message)) => {
            println!("❌ Timing issue: {}", message);
        }
    }
}

fn main() {
    let pantry_items = ["quinoa", "vegetables", "olive oil", "pasta"];

    println!("🍳 Recipe Request Processing System");
    println!("Available ingredients: {:?}", pantry_items);
    println!("{:=<50}", "");

    // Test different scenarios
    let requests = [
        ("Quinoa Bowl", 4, "Should work"),
        ("Pasta Primavera", 6, "Missing tomato sauce"),
        ("Lentil Soup", 2, "Missing lentils and broth"),
        ("Pizza", 4, "Recipe not available"),
        ("Quinoa Bowl", 25, "Too many servings"),
    ];

    for (recipe, servings, note) in requests {
        println!("\n🍽️ Request: {} for {} people ({})", recipe, servings, note);
        process_recipe_request(recipe, servings, &pantry_items);
    }
}
```


## Essential Result<T, E> Methods

### Checking and Unwrapping Methods

```rust
fn demonstrate_result_methods() {
    let success: Result<i32, String> = Ok(42);
    let failure: Result<i32, String> = Err("Something went wrong".to_string());

    // Checking methods - safe to use
    println!("✅ Checking methods:");
    println!("   success.is_ok(): {}", success.is_ok());     // true
    println!("   success.is_err(): {}", success.is_err());   // false
    println!("   failure.is_ok(): {}", failure.is_ok());     // false
    println!("   failure.is_err(): {}", failure.is_err());   // true

    // Safe unwrapping with defaults
    println!("\n🔒 Safe unwrapping:");
    println!("   success.unwrap_or(0): {}", success.unwrap_or(0));         // 42
    println!("   failure.unwrap_or(0): {}", failure.unwrap_or(0));         // 0

    // Unwrapping with computed defaults
    println!("   failure.unwrap_or_else(|_| -1): {}",
             failure.unwrap_or_else(|_| -1));                              // -1

    // Getting references to inner values
    println!("\n👀 Peeking at values:");
    if let Ok(val) = &success {
        println!("   Success contains: {}", val);
    }
    if let Err(err) = &failure {
        println!("   Error contains: {}", err);
    }

    // Dangerous methods - use with caution!
    println!("\n⚠️ Dangerous methods (use carefully):");
    println!("   success.unwrap(): {}", success.unwrap());    // 42 - OK
    // println!("   failure.unwrap(): {}", failure.unwrap());  // Would PANIC!

    println!("   success.expect('Should work'): {}",
             success.expect("Should work"));                   // 42 - OK
    // println!("   failure.expect('Should work'): {}",
    //          failure.expect("Should work"));                // Would PANIC!
}

fn main() {
    demonstrate_result_methods();
}
```


### Transformation Methods

```rust
fn demonstrate_transformations() {
    let cooking_temp: Result<u32, String> = Ok(350);
    let broken_thermometer: Result<u32, String> = Err("Sensor malfunction".to_string());

    // Map - transform success values
    println!("🔄 Transformation methods:");

    let temp_celsius = cooking_temp.map(|f| (f - 32) * 5 / 9);
    println!("   350°F in Celsius: {:?}", temp_celsius);

    let broken_celsius = broken_thermometer.map(|f| (f - 32) * 5 / 9);
    println!("   Broken sensor in Celsius: {:?}", broken_celsius);

    // Map with string formatting
    let temp_display = cooking_temp.map(|temp| format!("{}°F", temp));
    println!("   Formatted temperature: {:?}", temp_display);

    // Map_err - transform error values
    let better_error = broken_thermometer.map_err(|err| {
        format!("Kitchen equipment error: {} - call maintenance", err)
    });
    println!("   Better error message: {:?}", better_error);

    // and_then - chain operations that can fail
    println!("\n🔗 Chaining operations:");

    let recipe_result = cooking_temp.and_then(|temp| {
        if temp >= 300 && temp <= 400 {
            Ok(format!("Perfect for roasting vegetables at {}°F", temp))
        } else if temp < 300 {
            Err(format!("{}°F too low for roasting - increase heat", temp))
        } else {
            Err(format!("{}°F too high for roasting - reduce heat", temp))
        }
    });

    match recipe_result {
        Ok(instruction) => println!("   Recipe instruction: {}", instruction),
        Err(warning) => println!("   Temperature warning: {}", warning),
    }

    // or_else - provide alternative on error
    let backup_result = broken_thermometer.or_else(|_| {
        println!("   Thermometer broken, using visual cues...");
        Ok(375) // Estimated temperature
    });
    println!("   Backup temperature reading: {:?}", backup_result);
}

fn main() {
    demonstrate_transformations();
}
```


## The Question Mark Operator (?) - Error Propagation

### Simplifying Error Handling

The `?` operator is like having a **sous chef who automatically handles all kitchen emergencies** - if anything goes wrong at any step, they immediately report it up the chain of command without you having to manually check each step.

```rust
use std::collections::HashMap;

#[derive(Debug)]
enum KitchenError {
    MissingIngredient(String),
    InvalidQuantity(String),
    EquipmentFailure(String),
    TemperatureError(String),
}

struct Kitchen {
    ingredients: HashMap<String, u32>, // ingredient -> quantity in grams
    equipment_status: HashMap<String, bool>, // equipment -> working status
}

impl Kitchen {
    fn new() -> Self {
        let mut ingredients = HashMap::new();
        ingredients.insert("quinoa".to_string(), 500);
        ingredients.insert("olive oil".to_string(), 250);
        ingredients.insert("vegetables".to_string(), 800);
        ingredients.insert("salt".to_string(), 100);

        let mut equipment = HashMap::new();
        equipment.insert("stove".to_string(), true);
        equipment.insert("oven".to_string(), true);
        equipment.insert("blender".to_string(), false); // Broken!

        Kitchen {
            ingredients,
            equipment_status: equipment,
        }
    }

    // Check ingredient availability
    fn check_ingredient(&self, name: &str, needed: u32) -> Result<(), KitchenError> {
        match self.ingredients.get(name) {
            Some(available) if *available >= needed => Ok(()),
            Some(available) => Err(KitchenError::InvalidQuantity(
                format!("Need {}g of {}, but only have {}g", needed, name, available)
            )),
            None => Err(KitchenError::MissingIngredient(name.to_string())),
        }
    }

    // Check equipment availability
    fn check_equipment(&self, name: &str) -> Result<(), KitchenError> {
        match self.equipment_status.get(name) {
            Some(true) => Ok(()),
            Some(false) => Err(KitchenError::EquipmentFailure(
                format!("{} is not working", name)
            )),
            None => Err(KitchenError::EquipmentFailure(
                format!("{} not available in kitchen", name)
            )),
        }
    }

    // Validate cooking temperature
    fn validate_temperature(&self, temp: u32, method: &str) -> Result<(), KitchenError> {
        match method {
            "sauté" if temp < 300 || temp > 400 => {
                Err(KitchenError::TemperatureError(
                    format!("Sautéing requires 300-400°F, got {}°F", temp)
                ))
            }
            "boil" if temp < 212 => {
                Err(KitchenError::TemperatureError(
                    format!("Water boils at 212°F, got {}°F", temp)
                ))
            }
            _ => Ok(()),
        }
    }

    // WITHOUT ? operator - verbose and error-prone
    fn prepare_quinoa_bowl_verbose(&self) -> Result<String, KitchenError> {
        // Check quinoa
        match self.check_ingredient("quinoa", 200) {
            Ok(()) => {},
            Err(e) => return Err(e),
        }

        // Check vegetables
        match self.check_ingredient("vegetables", 300) {
            Ok(()) => {},
            Err(e) => return Err(e),
        }

        // Check olive oil
        match self.check_ingredient("olive oil", 30) {
            Ok(()) => {},
            Err(e) => return Err(e),
        }

        // Check stove
        match self.check_equipment("stove") {
            Ok(()) => {},
            Err(e) => return Err(e),
        }

        // Check temperature
        match self.validate_temperature(350, "sauté") {
            Ok(()) => {},
            Err(e) => return Err(e),
        }

        Ok("Quinoa bowl prepared successfully!".to_string())
    }

    // WITH ? operator - clean and readable!
    fn prepare_quinoa_bowl(&self) -> Result<String, KitchenError> {
        // Each ? automatically returns the error if something fails
        self.check_ingredient("quinoa", 200)?;       // Auto-return on error
        self.check_ingredient("vegetables", 300)?;   // Auto-return on error
        self.check_ingredient("olive oil", 30)?;     // Auto-return on error
        self.check_equipment("stove")?;              // Auto-return on error
        self.validate_temperature(350, "sauté")?;    // Auto-return on error

        Ok("Quinoa bowl prepared successfully!".to_string())
    }

    // Chain multiple operations with ?
    fn prepare_smoothie(&self) -> Result<String, KitchenError> {
        self.check_ingredient("vegetables", 150)?;   // Spinach, kale, etc.
        self.check_equipment("blender")?;            // This will fail!
        self.validate_temperature(32, "blend")?;     // Cold blending

        Ok("Green smoothie ready!".to_string())
    }

    // Complex recipe with multiple steps
    fn prepare_full_meal(&self) -> Result<Vec<String>, KitchenError> {
        let mut dishes = Vec::new();

        // Try to make quinoa bowl
        let quinoa_result = self.prepare_quinoa_bowl()?; // Propagates error if fails
        dishes.push(quinoa_result);

        // Try to make smoothie (will fail due to broken blender)
        let smoothie_result = self.prepare_smoothie()?; // Error will bubble up
        dishes.push(smoothie_result);

        Ok(dishes)
    }
}

fn main() {
    let kitchen = Kitchen::new();

    println!("🍳 Kitchen Status Check");
    println!("Ingredients: {:?}", kitchen.ingredients);
    println!("Equipment: {:?}", kitchen.equipment_status);
    println!("{:=<50}", "");

    // Test individual recipe
    println!("\n🥗 Making Quinoa Bowl:");
    match kitchen.prepare_quinoa_bowl() {
        Ok(success) => println!("✅ {}", success),
        Err(error) => println!("❌ Failed: {:?}", error),
    }

    // Test recipe that will fail
    println!("\n🥤 Making Smoothie:");
    match kitchen.prepare_smoothie() {
        Ok(success) => println!("✅ {}", success),
        Err(error) => println!("❌ Failed: {:?}", error),
    }

    // Test full meal (will fail at smoothie step)
    println!("\n🍽️ Preparing Full Meal:");
    match kitchen.prepare_full_meal() {
        Ok(dishes) => {
            println!("✅ Meal completed!");
            for dish in dishes {
                println!("   - {}", dish);
            }
        }
        Err(error) => {
            println!("❌ Meal preparation failed: {:?}", error);
            println!("   The error occurred during one of the recipe steps");
        }
    }
}
```

**Benefits of the `?` Operator:**

- 🎯 **Concise code** - No need for manual match statements
- 🔄 **Automatic propagation** - Errors bubble up automatically
- 📖 **Readable flow** - Focus on the happy path
- ✅ **Type safe** - Compiler ensures error types match
- 🚀 **Performance** - Zero runtime cost


## Advanced Result<T, E> Patterns

### Custom Error Types with Context

```rust
use std::fmt;

#[derive(Debug)]
enum RestaurantError {
    Inventory {
        item: String,
        required: u32,
        available: u32,
    },
    Equipment {
        device: String,
        issue: String,
        maintenance_needed: bool,
    },
    Staff {
        problem: String,
        shift_affected: String,
    },
    Customer {
        complaint: String,
        severity: u8, // 1-10 scale
        table_number: Option<u32>,
    },
    Supplier {
        supplier_name: String,
        delivery_issue: String,
        expected_date: String,
    },
}

impl fmt::Display for RestaurantError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            RestaurantError::Inventory { item, required, available } => {
                write!(f, "Inventory shortage: need {} of {}, but only have {}",
                       required, item, available)
            }
            RestaurantError::Equipment { device, issue, maintenance_needed } => {
                let action = if *maintenance_needed {
                    " - maintenance required"
                } else {
                    " - minor issue"
                };
                write!(f, "Equipment problem with {}: {}{}", device, issue, action)
            }
            RestaurantError::Staff { problem, shift_affected } => {
                write!(f, "Staff issue during {} shift: {}", shift_affected, problem)
            }
            RestaurantError::Customer { complaint, severity, table_number } => {
                let location = table_number
                    .map(|t| format!(" at table {}", t))
                    .unwrap_or_else(|| " (takeout)".to_string());
                write!(f, "Customer complaint{} (severity {}): {}",
                       location, severity, complaint)
            }
            RestaurantError::Supplier { supplier_name, delivery_issue, expected_date } => {
                write!(f, "Supplier {} delivery problem: {} (expected: {})",
                       supplier_name, delivery_issue, expected_date)
            }
        }
    }
}

struct VegetarianRestaurant {
    name: String,
    daily_operations: Vec<String>,
}

impl VegetarianRestaurant {
    fn new(name: String) -> Self {
        VegetarianRestaurant {
            name,
            daily_operations: Vec::new(),
        }
    }

    fn check_morning_inventory(&mut self) -> Result<(), RestaurantError> {
        // Simulate inventory check
        let inventory_items = [
            ("quinoa", 150, 200),      // (item, available, required)
            ("vegetables", 80, 500),   // This will fail
            ("olive oil", 300, 100),
        ];

        for (item, available, required) in inventory_items {
            if available < required {
                return Err(RestaurantError::Inventory {
                    item: item.to_string(),
                    required,
                    available,
                });
            }
        }

        self.daily_operations.push("✅ Morning inventory check complete".to_string());
        Ok(())
    }

    fn test_equipment(&mut self) -> Result<(), RestaurantError> {
        // Simulate equipment checks
        let equipment_checks = [
            ("oven", "working", false),
            ("blender", "motor making noise", true), // Needs maintenance
            ("refrigerator", "working", false),
        ];

        for (device, status, needs_maintenance) in equipment_checks {
            if status != "working" {
                return Err(RestaurantError::Equipment {
                    device: device.to_string(),
                    issue: status.to_string(),
                    maintenance_needed: needs_maintenance,
                });
            }
        }

        self.daily_operations.push("✅ Equipment check complete".to_string());
        Ok(())
    }

    fn process_customer_feedback(&mut self) -> Result<(), RestaurantError> {
        // Simulate customer complaint
        return Err(RestaurantError::Customer {
            complaint: "Food arrived cold".to_string(),
            severity: 7,
            table_number: Some(12),
        });
    }

    fn run_daily_operations(&mut self) -> Result<String, RestaurantError> {
        println!("🌅 Starting daily operations for {}", self.name);

        // Chain multiple operations - first error stops the chain
        self.check_morning_inventory()?;
        self.test_equipment()?;
        self.process_customer_feedback()?; // This will fail

        self.daily_operations.push("✅ All operations completed successfully".to_string());

        Ok(format!("Daily operations completed for {}", self.name))
    }

    fn get_operations_log(&self) -> &[String] {
        &self.daily_operations
    }
}

fn handle_restaurant_error(error: &RestaurantError) -> String {
    match error {
        RestaurantError::Inventory { item, required, available } => {
            format!("🛒 URGENT: Order {} more {} (need {}, have {})",
                   required - available, item, required, available)
        }
        RestaurantError::Equipment { device, maintenance_needed, .. } => {
            if *maintenance_needed {
                format!("🔧 MAINTENANCE: Schedule repair for {}", device)
            } else {
                format!("⚡ EQUIPMENT: Quick fix needed for {}", device)
            }
        }
        RestaurantError::Staff { shift_affected, .. } => {
            format!("👥 STAFFING: Address {} shift issue immediately", shift_affected)
        }
        RestaurantError::Customer { severity, table_number, .. } => {
            let urgency = if *severity >= 8 { "CRITICAL" } else { "HIGH" };
            let location = table_number
                .map(|t| format!("table {}", t))
                .unwrap_or_else(|| "takeout customer".to_string());
            format!("📞 {} PRIORITY: Resolve complaint from {}", urgency, location)
        }
        RestaurantError::Supplier { supplier_name, expected_date, .. } => {
            format!("🚚 LOGISTICS: Contact {} about delivery (due {})",
                   supplier_name, expected_date)
        }
    }
}

fn main() {
    let mut restaurant = VegetarianRestaurant::new("Green Garden Bistro".to_string());

    match restaurant.run_daily_operations() {
        Ok(success_message) => {
            println!("🎉 {}", success_message);
            println!("\n📋 Operations Log:");
            for log_entry in restaurant.get_operations_log() {
                println!("   {}", log_entry);
            }
        }
        Err(error) => {
            println!("❌ Operations failed: {}", error);
            println!("📋 Action required: {}", handle_restaurant_error(&error));

            println!("\n📝 Completed operations before failure:");
            for log_entry in restaurant.get_operations_log() {
                println!("   {}", log_entry);
            }
        }
    }
}
```


### Working with Collections of Results

```rust
fn process_multiple_orders() {
    let order_requests = [
        ("Quinoa Bowl", 2),
        ("Pasta Primavera", 4),
        ("Invalid Dish", 1),
        ("Lentil Soup", 6),
    ];

    // collect() on Results - stops at first error
    println!("🍽️ Processing all orders (stop on first error):");
    let all_orders_result: Result<Vec<_>, _> = order_requests
        .iter()
        .map(|(dish, servings)| validate_order(dish, *servings))
        .collect();

    match all_orders_result {
        Ok(orders) => {
            println!("✅ All orders validated successfully:");
            for order in orders {
                println!("   {}", order);
            }
        }
        Err(error) => {
            println!("❌ Order processing stopped at first error: {}", error);
        }
    }

    // partition() - separate successes and failures
    println!("\n📊 Processing all orders (collect successes and failures):");
    let (successes, failures): (Vec<_>, Vec<_>) = order_requests
        .iter()
        .map(|(dish, servings)| validate_order(dish, *servings))
        .partition(Result::is_ok);

    let successful_orders: Vec<_> = successes.into_iter().map(Result::unwrap).collect();
    let failed_orders: Vec<_> = failures.into_iter().map(Result::unwrap_err).collect();

    println!("✅ Successful orders ({}):", successful_orders.len());
    for order in successful_orders {
        println!("   {}", order);
    }

    println!("❌ Failed orders ({}):", failed_orders.len());
    for error in failed_orders {
        println!("   {}", error);
    }

    // filter_map() - keep only successes
    println!("\n🎯 Only successful orders:");
    let good_orders: Vec<_> = order_requests
        .iter()
        .filter_map(|(dish, servings)| {
            validate_order(dish, *servings).ok()
        })
        .collect();

    for order in good_orders {
        println!("   ✅ {}", order);
    }
}

fn validate_order(dish: &str, servings: u32) -> Result<String, String> {
    if servings == 0 {
        return Err("Cannot make 0 servings".to_string());
    }
    if servings > 10 {
        return Err(format!("Cannot make {} servings - maximum is 10", servings));
    }

    match dish {
        "Quinoa Bowl" | "Pasta Primavera" | "Lentil Soup" => {
            Ok(format!("{} for {} people", dish, servings))
        }
        _ => Err(format!("'{}' is not on our menu", dish)),
    }
}

fn main() {
    process_multiple_orders();
}
```


### Result with Custom Types and Conversions

```rust
use std::str::FromStr;

#[derive(Debug)]
struct Temperature {
    celsius: f64,
}

#[derive(Debug)]
enum TemperatureError {
    TooHot(f64),
    TooCold(f64),
    InvalidFormat(String),
    OutOfRange(String),
}

impl fmt::Display for TemperatureError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            TemperatureError::TooHot(temp) => {
                write!(f, "Temperature {}°C is dangerously hot for cooking", temp)
            }
            TemperatureError::TooCold(temp) => {
                write!(f, "Temperature {}°C is too cold for proper cooking", temp)
            }
            TemperatureError::InvalidFormat(input) => {
                write!(f, "'{}' is not a valid temperature format", input)
            }
            TemperatureError::OutOfRange(msg) => {
                write!(f, "Temperature out of range: {}", msg)
            }
        }
    }
}

impl Temperature {
    fn new(celsius: f64) -> Result<Self, TemperatureError> {
        if celsius < -273.15 {
            Err(TemperatureError::OutOfRange(
                format!("{}°C is below absolute zero", celsius)
            ))
        } else if celsius < 0.0 {
            Err(TemperatureError::TooCold(celsius))
        } else if celsius > 300.0 {
            Err(TemperatureError::TooHot(celsius))
        } else {
            Ok(Temperature { celsius })
        }
    }

    fn from_fahrenheit(fahrenheit: f64) -> Result<Self, TemperatureError> {
        let celsius = (fahrenheit - 32.0) * 5.0 / 9.0;
        Self::new(celsius)
    }

    fn to_fahrenheit(&self) -> f64 {
        self.celsius * 9.0 / 5.0 + 32.0
    }

    fn is_suitable_for_cooking(&self) -> bool {
        self.celsius >= 60.0 && self.celsius <= 250.0
    }

    fn get_cooking_method(&self) -> &str {
        match self.celsius {
            temp if temp < 60.0 => "too cold for cooking",
            60.0..=80.0 => "suitable for warming",
            81.0..=100.0 => "good for simmering",
            101.0..=150.0 => "perfect for sautéing",
            151.0..=200.0 => "ideal for roasting",
            201.0..=250.0 => "high heat cooking",
            _ => "too hot - dangerous",
        }
    }
}

impl FromStr for Temperature {
    type Err = TemperatureError;

    fn from_str(s: &str) -> Result<Self, Self::Err> {
        let s = s.trim();

        if s.ends_with("°C") {
            let num_part = &s[..s.len()-2];
            match num_part.parse::<f64>() {
                Ok(celsius) => Self::new(celsius),
                Err(_) => Err(TemperatureError::InvalidFormat(s.to_string())),
            }
        } else if s.ends_with("°F") {
            let num_part = &s[..s.len()-2];
            match num_part.parse::<f64>() {
                Ok(fahrenheit) => Self::from_fahrenheit(fahrenheit),
                Err(_) => Err(TemperatureError::InvalidFormat(s.to_string())),
            }
        } else {
            // Assume Celsius if no unit specified
            match s.parse::<f64>() {
                Ok(celsius) => Self::new(celsius),
                Err(_) => Err(TemperatureError::InvalidFormat(s.to_string())),
            }
        }
    }
}

fn process_cooking_temperatures(temp_strings: &[&str]) {
    println!("🌡️ Processing Cooking Temperatures:");
    println!("{:=<60}", "");

    for temp_str in temp_strings {
        match temp_str.parse::<Temperature>() {
            Ok(temp) => {
                println!("✅ {}: {:.1}°C ({:.1}°F)",
                         temp_str, temp.celsius, temp.to_fahrenheit());
                println!("   Cooking method: {}", temp.get_cooking_method());
                if temp.is_suitable_for_cooking() {
                    println!("   🍳 Safe for cooking");
                } else {
                    println!("   ⚠️ Not suitable for cooking");
                }
            }
            Err(error) => {
                println!("❌ {}: {}", temp_str, error);

                // Provide helpful suggestions based on error type
                match error {
                    TemperatureError::TooHot(_) => {
                        println!("   💡 Suggestion: Lower the heat for safety");
                    }
                    TemperatureError::TooCold(_) => {
                        println!("   💡 Suggestion: Increase temperature for proper cooking");
                    }
                    TemperatureError::InvalidFormat(_) => {
                        println!("   💡 Suggestion: Use format like '180°C' or '350°F'");
                    }
                    TemperatureError::OutOfRange(_) => {
                        println!("   💡 Suggestion: Check your thermometer calibration");
                    }
                }
            }
        }
        println!();
    }
}

fn main() {
    let temperature_inputs = [
        "180°C",      // Good cooking temperature
        "350°F",      // Good cooking temperature in Fahrenheit
        "25°C",       // Too cold
        "400°C",      // Too hot
        "-300°C",     // Below absolute zero
        "abc°C",      // Invalid format
        "200",        // No unit (assumes Celsius)
        "invalid",    // Invalid
    ];

    process_cooking_temperatures(&temperature_inputs);
}
```


## Best Practices and Patterns

### 1. Prefer Specific Error Types

```rust
// ❌ Generic string errors - not helpful
fn bad_parse_recipe(input: &str) -> Result<Recipe, String> {
    if input.is_empty() {
        Err("bad input".to_string())  // Not descriptive
    } else {
        // ...
        Err("something wrong".to_string())  // Vague
    }
}

// ✅ Specific error types with context
#[derive(Debug)]
enum RecipeParseError {
    EmptyInput,
    MissingName,
    InvalidServings { input: String, reason: String },
    MissingIngredients,
    InvalidIngredient { line: usize, content: String },
    UnsupportedFormat { expected: String, found: String },
}

impl fmt::Display for RecipeParseError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            RecipeParseError::EmptyInput => {
                write!(f, "Recipe input cannot be empty")
            }
            RecipeParseError::MissingName => {
                write!(f, "Recipe must have a name on the first line")
            }
            RecipeParseError::InvalidServings { input, reason } => {
                write!(f, "Invalid servings '{}': {}", input, reason)
            }
            RecipeParseError::MissingIngredients => {
                write!(f, "Recipe must have at least one ingredient")
            }
            RecipeParseError::InvalidIngredient { line, content } => {
                write!(f, "Invalid ingredient on line {}: '{}'", line, content)
            }
            RecipeParseError::UnsupportedFormat { expected, found } => {
                write!(f, "Expected {} format, but found {}", expected, found)
            }
        }
    }
}

fn good_parse_recipe(input: &str) -> Result<Recipe, RecipeParseError> {
    if input.trim().is_empty() {
        return Err(RecipeParseError::EmptyInput);
    }

    let lines: Vec<&str> = input.lines().collect();

    if lines.is_empty() {
        return Err(RecipeParseError::MissingName);
    }

    let name = lines[^0].trim();
    if name.is_empty() {
        return Err(RecipeParseError::MissingName);
    }

    // More parsing logic...
    Ok(Recipe {
        name: name.to_string(),
        servings: 4,
        estimated_time: 30,
    })
}
```


### 2. Use ? for Clean Error Propagation

```rust
// Chain operations that can fail
fn prepare_complex_dish() -> Result<String, KitchenError> {
    let ingredients = gather_ingredients()?;
    let prepared_ingredients = prep_ingredients(ingredients)?;
    let cooked_components = cook_components(prepared_ingredients)?;
    let final_dish = plate_dish(cooked_components)?;

    Ok(format!("Successfully prepared: {}", final_dish))
}

// Individual steps that can fail
fn gather_ingredients() -> Result<Vec<String>, KitchenError> {
    // Check pantry, return list of available ingredients
    Ok(vec!["quinoa".to_string(), "vegetables".to_string()])
}

fn prep_ingredients(ingredients: Vec<String>) -> Result<Vec<String>, KitchenError> {
    // Wash, chop, measure ingredients
    Ok(ingredients) // Simplified
}

fn cook_components(ingredients: Vec<String>) -> Result<Vec<String>, KitchenError> {
    // Cook each component
    Ok(ingredients) // Simplified
}

fn plate_dish(components: Vec<String>) -> Result<String, KitchenError> {
    // Combine and present
    Ok("Beautiful quinoa bowl".to_string())
}
```


### 3. Provide Meaningful Error Context

```rust
impl From<std::num::ParseIntError> for RecipeParseError {
    fn from(error: std::num::ParseIntError) -> Self {
        RecipeParseError::InvalidServings {
            input: "parsing error".to_string(),
            reason: format!("Not a valid number: {}", error),
        }
    }
}

// This allows automatic conversion with ?
fn parse_servings(input: &str) -> Result<u32, RecipeParseError> {
    let servings = input.parse::<u32>()?; // Automatically converts ParseIntError

    if servings == 0 {
        Err(RecipeParseError::InvalidServings {
            input: input.to_string(),
            reason: "Must serve at least 1 person".to_string(),
        })
    } else if servings > 50 {
        Err(RecipeParseError::InvalidServings {
            input: input.to_string(),
            reason: "Cannot serve more than 50 people".to_string(),
        })
    } else {
        Ok(servings)
    }
}
```


### 4. Handle Results at the Right Level

```rust
// Low-level functions return Results
fn read_ingredient_database() -> Result<Database, DatabaseError> {
    // ... implementation
}

fn find_recipe(db: &Database, name: &str) -> Result<Recipe, RecipeError> {
    // ... implementation
}

// High-level functions handle multiple Results
fn prepare_meal_plan(recipe_names: &[&str]) -> Vec<MealPlanResult> {
    let database = match read_ingredient_database() {
        Ok(db) => db,
        Err(e) => {
            return vec![MealPlanResult::DatabaseError(e.to_string())];
        }
    };

    recipe_names
        .iter()
        .map(|name| {
            match find_recipe(&database, name) {
                Ok(recipe) => MealPlanResult::Success(recipe),
                Err(e) => MealPlanResult::RecipeError {
                    recipe_name: name.to_string(),
                    error: e.to_string(),
                },
            }
        })
        .collect()
}

#[derive(Debug)]
enum MealPlanResult {
    Success(Recipe),
    RecipeError { recipe_name: String, error: String },
    DatabaseError(String),
}
```


## Common Mistakes to Avoid

### Mistake 1: Using unwrap() Carelessly

```rust
// ❌ Dangerous - can crash your program
fn dangerous_cooking() {
    let temp_str = "invalid";
    let temperature = temp_str.parse::<u32>().unwrap(); // 💥 PANIC!
    println!("Cooking at {}°F", temperature);
}

// ✅ Safe - handle the error properly
fn safe_cooking() {
    let temp_str = "invalid";
    match temp_str.parse::<u32>() {
        Ok(temperature) => {
            println!("✅ Cooking at {}°F", temperature);
        }
        Err(error) => {
            println!("❌ Invalid temperature '{}': {}", temp_str, error);
            println!("💡 Please provide a valid number");
        }
    }
}

// ✅ Also safe - provide a default
fn safe_cooking_with_default() {
    let temp_str = "invalid";
    let temperature = temp_str.parse::<u32>().unwrap_or(350);
    println!("🍳 Cooking at {}°F (default if invalid)", temperature);
}
```


### Mistake 2: Not Providing Helpful Error Messages

```rust
// ❌ Vague error messages
#[derive(Debug)]
enum BadError {
    Failed,
    Wrong,
    Problem,
}

// ✅ Specific, actionable error messages
#[derive(Debug)]
enum GoodError {
    InsufficientIngredients {
        required: HashMap<String, u32>,
        available: HashMap<String, u32>,
        missing: Vec<String>,
    },
    EquipmentUnavailable {
        equipment: String,
        reason: String,
        alternative: Option<String>,
    },
    TemperatureOutOfRange {
        current: u32,
        min_required: u32,
        max_required: u32,
        adjustment_needed: String,
    },
}
```


### Mistake 3: Mixing Error Handling Styles

```rust
// ❌ Inconsistent - some functions panic, others return Results
fn inconsistent_function_1(input: &str) -> String {
    input.parse::<u32>().unwrap().to_string() // Can panic!
}

fn inconsistent_function_2(input: &str) -> Result<String, String> {
    match input.parse::<u32>() {
        Ok(num) => Ok(num.to_string()),
        Err(e) => Err(e.to_string()),
    }
}

// ✅ Consistent - all functions use Result for error handling
fn consistent_function_1(input: &str) -> Result<String, ParseError> {
    let num = input.parse::<u32>()
        .map_err(|e| ParseError::InvalidNumber(input.to_string(), e.to_string()))?;
    Ok(num.to_string())
}

fn consistent_function_2(input: &str) -> Result<String, ParseError> {
    let num = input.parse::<u32>()
        .map_err(|e| ParseError::InvalidNumber(input.to_string(), e.to_string()))?;
    Ok(format!("Processed: {}", num))
}

#[derive(Debug)]
enum ParseError {
    InvalidNumber(String, String),
    EmptyInput,
    OutOfRange(String),
}
```


## Summary and Key Takeaways

### **Mental Model: The Professional Kitchen Incident Report System**

Remember the kitchen analogy:

- 🎯 **Ok(success)** = Dish prepared perfectly with details of what was made
- ❌ **Err(error)** = Something went wrong with specific details about the problem
- 📋 **Pattern matching** = Kitchen manager reviewing incident reports
- 🔄 **Error propagation (?)** = Problems automatically reported up the chain of command
- 🛠️ **Error handling** = Taking specific action based on what went wrong


### **Essential Principles**

1. **Explicit error handling** - Every operation that can fail returns a Result
2. **Rich error information** - Errors contain context about what went wrong
3. **Composable operations** - Chain operations safely with `?` operator
4. **Type safety** - Compiler ensures you handle both success and error cases
5. **No hidden failures** - All potential failures are visible in the type system

### **When to Use Result<T, E>**

| **Use Case** | **Example** | **Why Result Works** |
| :-- | :-- | :-- |
| **I/O operations** | Reading files, network requests | Operations can fail for many reasons |
| **Parsing** | String to number conversion | Input might be invalid |
| **User input** | Form validation | Users can enter invalid data |
| **External services** | API calls, database queries | Services can be unavailable |
| **Business logic** | Order processing | Business rules can be violated |
| **Resource allocation** | Memory, file handles | Resources can be exhausted |

### **Result<T, E> Method Quick Reference**

```rust
let success: Result<i32, String> = Ok(42);
let failure: Result<i32, String> = Err("oops".to_string());

// Checking
success.is_ok()                    // true
success.is_err()                   // false

// Safe extraction
success.unwrap_or(0)               // 42
failure.unwrap_or(0)               // 0
failure.unwrap_or_else(|_| -1)     // -1

// Transformation
success.map(|x| x * 2)             // Ok(84)
failure.map_err(|e| format!("Error: {}", e))

// Chaining
success.and_then(|x| Ok(x + 1))    // Ok(43)
success.and_then(|x| if x > 50 { Ok(x) } else { Err("too small".to_string()) })

// Error propagation
fn chain_operations() -> Result<i32, String> {
    let a = might_fail_1()?;       // Early return on error
    let b = might_fail_2(a)?;      // Early return on error
    let c = might_fail_3(b)?;      // Early return on error
    Ok(c)
}
```


### **Best Practices Summary**

**✅ DO:**

- Use specific error types with context
- Leverage the `?` operator for clean propagation
- Handle errors at appropriate levels
- Provide actionable error messages
- Use `match` for complex error handling
- Design errors to be helpful for debugging

**❌ DON'T:**

- Use `unwrap()` without good reason
- Create vague or generic error messages
- Mix panicking and Result-returning functions
- Ignore error context
- Make errors that don't help users understand what went wrong


### **The Bigger Picture**

**Result<T, E> is like having a world-class incident management system** in your vegetarian restaurant kitchen:

- 📊 **Complete transparency** - Every success and failure is documented
- 🎯 **Actionable information** - Errors tell you exactly what to fix
- 🔄 **Scalable handling** - Errors automatically propagate where they need to go
- ✅ **Guaranteed coverage** - Compiler ensures no error goes unhandled
- 📈 **Continuous improvement** - Rich error data helps improve processes

**Result<T, E> eliminates entire categories of bugs** by making failure modes explicit and forcing you to think about what can go wrong. It's the foundation of Rust's reliability - like having a kitchen that never serves a bad dish because every step of the cooking process is monitored, validated, and error-checked. When something does go wrong, you know exactly what happened and can fix it immediately!

This explicit error handling might feel like extra work at first, but it leads to incredibly robust programs that handle real-world problems gracefully - just like a professional kitchen that runs smoothly even when ingredients run out, equipment breaks, or unexpected orders come in, because every possible problem has been anticipated and planned for.

1. https://doc.rust-lang.org/rust-by-example/std/result.html
2. https://doc.rust-lang.org/std/result/enum.Result.html
3. https://doc.rust-lang.org/std/result/
4. https://www.geeksforgeeks.org/rust/rust-result-type/
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/std/result/enum.Result.html
6. https://codedamn.com/news/rust/advanced-error-handling-rust-exploring-power-result-type
7. https://dev.to/leapcell/rusts-result-type-error-handling-made-easy-58i
8. https://blog.logrocket.com/understanding-rust-option-results-enums/
9. https://reintech.io/blog/understanding-implementing-rust-option-result-types
10. https://labex.io/tutorials/rust-exploring-rust-s-result-type-99239
11. https://labex.io/tutorials/rust-exploring-rust-s-result-enum-99257
12. https://doc.rust-lang.org/rust-by-example/error/result.html
13. https://www.youtube.com/watch?v=NSqN2r0h8DE
14. https://stackoverflow.com/questions/69218879/rust-extend-built-in-enum-result
15. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/std/result/index.html
16. https://www.youtube.com/watch?v=s5S2Ed5T-dc
17. https://oida.dev/rust-enums-wrapping-errors/
18. https://www.reddit.com/r/rust/comments/10c23qc/what_does_result_mean/
19. https://dhghomon.github.io/easy_rust/Chapter_31.html
20. https://en.wikipedia.org/wiki/Result_type
