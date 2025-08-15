+++
title = "Generic Programming Exercises"
description = "Practice exercises to reinforce concepts of generics, traits, and type constraints in Rust."
date = "2025-08-15"
weight = 2
+++

# Generic Programming Exercises in Rust: Comprehensive Practice Guide for Beginners

Understanding generics, traits, and type constraints in Rust through practice is like **learning to design and manage a complete professional restaurant training program** - you start with basic skills and gradually build up to complex, real-world scenarios that combine multiple concepts. Just as a professional chef training program progresses from basic knife skills to creating complex multi-course meals that satisfy various dietary requirements, these exercises will take you from simple generic functions to sophisticated type-safe systems that handle complex constraints and interactions.

## The Professional Restaurant Training Program Analogy 🏪

### Imagine You're Creating a Comprehensive Chef Training Curriculum

**The Training Philosophy:**

```rust
// Like a complete restaurant training program, we'll build skills progressively:
// 1. Basic tools and techniques (simple generics)
// 2. Quality standards and certifications (traits)
// 3. Complex menu requirements (type constraints)
// 4. Professional kitchen management (advanced patterns)
// 5. Master chef certification (real-world applications)
```

**Why Practice Exercises Are Essential:**

- 🎯 **Hands-on learning** - Understanding comes through doing, not just reading
- 📈 **Progressive difficulty** - Each exercise builds on previous concepts
- 🔄 **Reinforcement** - Multiple exercises cement understanding
- 🛠️ **Problem-solving** - Learn to apply concepts to real scenarios
- 🏆 **Confidence building** - Success in exercises builds programming confidence


## Exercise Set 1: Basic Generic Fundamentals

### Learning Basic Kitchen Tools (Simple Generics)

**Exercise 1.1: Generic Container**
*Difficulty: Beginner*

Create a generic container that can hold any type of restaurant item.

```rust
// 🎯 EXERCISE: Complete the generic container implementation
#[derive(Debug)]
struct RestaurantContainer<T> {
    // TODO: Add field to store an item of type T
    item: T,
    label: String,
}

impl<T> RestaurantContainer<T> {
    // TODO: Implement new() method
    fn new(item: T, label: String) -> Self {
        RestaurantContainer { item, label }
    }

    // TODO: Implement get_item() method that returns a reference to the item
    fn get_item(&self) -> &T {
        &self.item
    }

    // TODO: Implement get_label() method
    fn get_label(&self) -> &str {
        &self.label
    }
}

fn exercise_1_1_solution() {
    println!("🥄 Exercise 1.1: Generic Container");
    println!("{:=<50}", "");

    // Test with different types
    let vegetable_container = RestaurantContainer::new("Fresh Tomatoes", "Vegetables".to_string());
    let quantity_container = RestaurantContainer::new(50, "Inventory Count".to_string());
    let price_container = RestaurantContainer::new(15.99, "Menu Price".to_string());

    println!("Vegetable: {} ({})", vegetable_container.get_item(), vegetable_container.get_label());
    println!("Quantity: {} ({})", quantity_container.get_item(), quantity_container.get_label());
    println!("Price: ${} ({})", price_container.get_item(), price_container.get_label());

    println!("\n📚 What you learned:");
    println!("• Generic structs can hold any type T");
    println!("• Methods on generic types need impl<T>");
    println!("• Same struct definition works with different types");
    println!("• Type safety is maintained - no mixing types accidentally\n");
}
```

**Exercise 1.2: Generic Function**
*Difficulty: Beginner*

Write a generic function that can process any type of restaurant data.

```rust
// 🎯 EXERCISE: Complete the generic function
fn process_restaurant_item<T>(item: T, description: &str) -> String
where
    T: std::fmt::Debug,  // We need Debug to format the item
{
    // TODO: Return a formatted string showing the processed item
    format!("Processing {}: {:?}", description, item)
}

fn exercise_1_2_solution() {
    println!("🔧 Exercise 1.2: Generic Function");
    println!("{:=<50}", "");

    // Test with various restaurant data types
    let menu_item = "Quinoa Buddha Bowl";
    let table_number = 42;
    let price = 15.99;
    let available = true;

    println!("{}", process_restaurant_item(menu_item, "menu item"));
    println!("{}", process_restaurant_item(table_number, "table"));
    println!("{}", process_restaurant_item(price, "price"));
    println!("{}", process_restaurant_item(available, "availability"));

    println!("\n📚 What you learned:");
    println!("• Generic functions work with any type T");
    println!("• where clauses specify what T can do");
    println!("• Debug trait lets us print any type");
    println!("• One function definition handles multiple types\n");
}
```

**Exercise 1.3: Multiple Generic Parameters**
*Difficulty: Beginner+*

Create a generic pairing system for restaurant operations.

```rust
// 🎯 EXERCISE: Complete the generic pair implementation
#[derive(Debug)]
struct RestaurantPair<T, U> {
    // TODO: Add fields for first and second items
    first: T,
    second: U,
}

impl<T, U> RestaurantPair<T, U> {
    // TODO: Implement new() method
    fn new(first: T, second: U) -> Self {
        RestaurantPair { first, second }
    }

    // TODO: Implement get_first() method
    fn get_first(&self) -> &T {
        &self.first
    }

    // TODO: Implement get_second() method
    fn get_second(&self) -> &U {
        &self.second
    }

    // TODO: Implement swap() method that returns a new pair with swapped types
    fn swap(self) -> RestaurantPair<U, T> {
        RestaurantPair {
            first: self.second,
            second: self.first,
        }
    }
}

fn exercise_1_3_solution() {
    println!("👥 Exercise 1.3: Multiple Generic Parameters");
    println!("{:=<50}", "");

    // Different type combinations
    let menu_price = RestaurantPair::new("Caesar Salad", 12.99);
    let table_capacity = RestaurantPair::new(5, "people");
    let ingredient_quantity = RestaurantPair::new("Tomatoes", 25);

    println!("Menu-Price: {} costs ${}", menu_price.get_first(), menu_price.get_second());
    println!("Table-Capacity: Table {} seats {}", table_capacity.get_first(), table_capacity.get_second());
    println!("Ingredient-Quantity: {} quantity: {}", ingredient_quantity.get_first(), ingredient_quantity.get_second());

    // Test swapping
    let swapped = menu_price.swap();
    println!("After swap: Price ${} for {}", swapped.get_first(), swapped.get_second());

    println!("\n📚 What you learned:");
    println!("• Multiple generic parameters: <T, U>");
    println!("• Each parameter can be a different type");
    println!("• Methods can transform type relationships");
    println!("• Type safety prevents incorrect combinations\n");
}
```


## Exercise Set 2: Trait Implementation and Usage

### Learning Professional Standards (Traits)

**Exercise 2.1: Basic Trait Definition and Implementation**
*Difficulty: Intermediate*

Create a trait for restaurant items and implement it for different types.

```rust
// 🎯 EXERCISE: Complete the trait and implementations
trait RestaurantItem {
    // TODO: Define methods that all restaurant items should have
    fn name(&self) -> &str;
    fn category(&self) -> &str;
    fn prepare_time(&self) -> u32; // minutes

    // Default implementation for description
    fn description(&self) -> String {
        format!("{} from {} category ({}min prep)",
                self.name(), self.category(), self.prepare_time())
    }
}

// TODO: Implement RestaurantItem for different food types
#[derive(Debug)]
struct Appetizer {
    name: String,
    ingredients: Vec<String>,
}

impl RestaurantItem for Appetizer {
    fn name(&self) -> &str {
        &self.name
    }

    fn category(&self) -> &str {
        "Appetizers"
    }

    fn prepare_time(&self) -> u32 {
        5 // Quick preparation
    }
}

#[derive(Debug)]
struct MainCourse {
    name: String,
    complexity: u32,
    cooking_method: String,
}

impl RestaurantItem for MainCourse {
    fn name(&self) -> &str {
        &self.name
    }

    fn category(&self) -> &str {
        "Main Courses"
    }

    fn prepare_time(&self) -> u32 {
        self.complexity * 8 // More complex = longer prep
    }
}

#[derive(Debug)]
struct Dessert {
    name: String,
    temperature: String, // "hot" or "cold"
}

impl RestaurantItem for Dessert {
    fn name(&self) -> &str {
        &self.name
    }

    fn category(&self) -> &str {
        "Desserts"
    }

    fn prepare_time(&self) -> u32 {
        if self.temperature == "hot" { 12 } else { 5 }
    }
}

fn exercise_2_1_solution() {
    println!("🏷️ Exercise 2.1: Basic Trait Implementation");
    println!("{:=<50}", "");

    let appetizer = Appetizer {
        name: "Bruschetta".to_string(),
        ingredients: vec!["bread".to_string(), "tomato".to_string(), "basil".to_string()],
    };

    let main_course = MainCourse {
        name: "Quinoa Buddha Bowl".to_string(),
        complexity: 3,
        cooking_method: "assembly".to_string(),
    };

    let dessert = Dessert {
        name: "Tiramisu".to_string(),
        temperature: "cold".to_string(),
    };

    println!("{}", appetizer.description());
    println!("{}", main_course.description());
    println!("{}", dessert.description());

    println!("\n📚 What you learned:");
    println!("• Traits define shared behavior across types");
    println!("• Different types can implement the same trait differently");
    println!("• Traits can have default implementations");
    println!("• Shared interface enables polymorphic behavior\n");
}
```

**Exercise 2.2: Generic Functions with Trait Bounds**
*Difficulty: Intermediate*

Write functions that work with any type implementing your trait.

```rust
// 🎯 EXERCISE: Complete the generic functions with trait bounds
fn process_item<T>(item: &T) -> String
where
    T: RestaurantItem,  // TODO: Add trait bound
{
    // TODO: Use trait methods to process the item
    format!("📋 Processing: {} | Category: {} | Prep time: {}min",
            item.name(), item.category(), item.prepare_time())
}

fn find_quick_items<T>(items: &[T]) -> Vec<&T>
where
    T: RestaurantItem,  // TODO: Add trait bound
{
    // TODO: Filter items that take 10 minutes or less to prepare
    items.iter()
         .filter(|item| item.prepare_time() <= 10)
         .collect()
}

fn total_prep_time<T>(items: &[T]) -> u32
where
    T: RestaurantItem,  // TODO: Add trait bound
{
    // TODO: Sum up all preparation times
    items.iter()
         .map(|item| item.prepare_time())
         .sum()
}

fn exercise_2_2_solution() {
    println!("⚙️ Exercise 2.2: Generic Functions with Trait Bounds");
    println!("{:=<50}", "");

    let items: Vec<Box<dyn RestaurantItem>> = vec![
        Box::new(Appetizer {
            name: "Calamari".to_string(),
            ingredients: vec!["squid".to_string(), "flour".to_string()],
        }),
        Box::new(MainCourse {
            name: "Pasta Carbonara".to_string(),
            complexity: 4,
            cooking_method: "stovetop".to_string(),
        }),
        Box::new(Dessert {
            name: "Ice Cream".to_string(),
            temperature: "cold".to_string(),
        }),
    ];

    // Process each item
    for item in &items {
        println!("{}", process_item(item.as_ref()));
    }

    println!("\n📚 What you learned:");
    println!("• Trait bounds constrain generic types");
    println!("• Functions can work with any type implementing a trait");
    println!("• Box<dyn Trait> enables storing different types together");
    println!("• Generic functions provide code reuse with type safety\n");
}
```

**Exercise 2.3: Multiple Trait Bounds**
*Difficulty: Intermediate+*

Combine multiple traits for more sophisticated operations.

```rust
// 🎯 EXERCISE: Complete the implementation with multiple trait bounds
use std::fmt::{Debug, Display};

trait Priceable {
    fn base_price(&self) -> f64;
    fn tax_rate(&self) -> f64 { 0.08 } // 8% default tax

    fn total_price(&self) -> f64 {
        self.base_price() * (1.0 + self.tax_rate())
    }
}

// TODO: Implement Priceable for your restaurant items
impl Priceable for Appetizer {
    fn base_price(&self) -> f64 {
        8.99 + (self.ingredients.len() as f64 * 0.50) // Base + ingredient cost
    }
}

impl Priceable for MainCourse {
    fn base_price(&self) -> f64 {
        12.00 + (self.complexity as f64 * 2.00) // Base + complexity cost
    }
}

impl Priceable for Dessert {
    fn base_price(&self) -> f64 {
        if self.temperature == "hot" { 7.99 } else { 5.99 }
    }
}

// TODO: Implement Display for better formatting
impl Display for Appetizer {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} (Appetizer with {} ingredients)", self.name, self.ingredients.len())
    }
}

impl Display for MainCourse {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} (Main course, {} method)", self.name, self.cooking_method)
    }
}

impl Display for Dessert {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{} ({} dessert)", self.name, self.temperature)
    }
}

// Generic function with multiple trait bounds
fn create_menu_description<T>(item: &T) -> String
where
    T: RestaurantItem + Priceable + Display + Debug,
{
    format!("🍽️ {}\n   {} - ${:.2}\n   Debug: {:?}",
            item,                    // Uses Display
            item.description(),      // Uses RestaurantItem
            item.total_price(),      // Uses Priceable
            item)                    // Uses Debug
}

fn exercise_2_3_solution() {
    println!("🔗 Exercise 2.3: Multiple Trait Bounds");
    println!("{:=<50}", "");

    let appetizer = Appetizer {
        name: "Stuffed Mushrooms".to_string(),
        ingredients: vec!["mushrooms".to_string(), "cheese".to_string(), "herbs".to_string()],
    };

    let main_course = MainCourse {
        name: "Grilled Salmon".to_string(),
        complexity: 5,
        cooking_method: "grilled".to_string(),
    };

    println!("{}", create_menu_description(&appetizer));
    println!("{}", create_menu_description(&main_course));

    println!("\n📚 What you learned:");
    println!("• Multiple trait bounds combine requirements");
    println!("• Use + syntax for multiple bounds: T: Trait1 + Trait2");
    println!("• Each trait adds specific capabilities to the type");
    println!("• All bounds must be satisfied for the function to work\n");
}
```


## Exercise Set 3: Advanced Type Constraints and Where Clauses

### Learning Complex Kitchen Operations (Advanced Constraints)

**Exercise 3.1: Where Clauses for Complex Constraints**
*Difficulty: Advanced*

Use where clauses to manage complex constraint relationships.

```rust
// 🎯 EXERCISE: Complete the complex constraint system
use std::collections::HashMap;

trait Storable {
    type Key: Clone + std::hash::Hash + Eq + Debug;
    fn storage_key(&self) -> Self::Key;
}

trait Trackable {
    fn track_usage(&self) -> String;
}

// TODO: Implement Storable for restaurant items
impl Storable for Appetizer {
    type Key = String;

    fn storage_key(&self) -> Self::Key {
        format!("APP_{}", self.name.to_uppercase().replace(" ", "_"))
    }
}

impl Storable for MainCourse {
    type Key = String;

    fn storage_key(&self) -> Self::Key {
        format!("MAIN_{}", self.name.to_uppercase().replace(" ", "_"))
    }
}

// TODO: Implement Trackable
impl Trackable for Appetizer {
    fn track_usage(&self) -> String {
        format!("Tracked appetizer: {} with {} ingredients", self.name, self.ingredients.len())
    }
}

impl Trackable for MainCourse {
    fn track_usage(&self) -> String {
        format!("Tracked main course: {} (complexity: {})", self.name, self.complexity)
    }
}

// Complex generic function with where clause
fn manage_restaurant_inventory<T, U>(
    items: Vec<T>,
    storage: &mut HashMap<T::Key, U>,
    processor: fn(&T) -> U,
) -> Vec<String>
where
    T: Storable + Trackable + Debug,
    U: Debug + Clone,
    T::Key: Display,
{
    let mut tracking_results = Vec::new();

    for item in items {
        // Store the item
        let key = item.storage_key();
        let processed_value = processor(&item);
        storage.insert(key.clone(), processed_value.clone());

        // Track the item
        let tracking_info = item.track_usage();
        tracking_results.push(format!("Key: {} | {} | Stored: {:?}",
                                     key, tracking_info, processed_value));
    }

    tracking_results
}

fn exercise_3_1_solution() {
    println!("📊 Exercise 3.1: Complex Where Clauses");
    println!("{:=<50}", "");

    let items = vec![
        Appetizer {
            name: "Spring Rolls".to_string(),
            ingredients: vec!["wrapper".to_string(), "vegetables".to_string()],
        },
        MainCourse {
            name: "Beef Stir Fry".to_string(),
            complexity: 3,
            cooking_method: "wok".to_string(),
        },
    ];

    let mut storage: HashMap<String, String> = HashMap::new();

    let results = manage_restaurant_inventory(
        items,
        &mut storage,
        |item| format!("Processed: {:?}", item)
    );

    for result in results {
        println!("{}", result);
    }

    println!("\nStorage contents:");
    for (key, value) in storage {
        println!("  {}: {}", key, value);
    }

    println!("\n📚 What you learned:");
    println!("• Where clauses handle complex constraint relationships");
    println!("• Associated types can have their own constraints");
    println!("• Multiple generic parameters can have different constraints");
    println!("• Complex systems become manageable with proper constraints\n");
}
```

**Exercise 3.2: Generic Collections with Constraints**
*Difficulty: Advanced*

Build a type-safe collection system for restaurant management.

```rust
// 🎯 EXERCISE: Complete the generic collection with constraints
use std::collections::{HashMap, VecDeque};

trait Identifiable {
    type Id: Clone + Eq + std::hash::Hash + Display + Debug;
    fn id(&self) -> &Self::Id;
}

trait Processable {
    type Output: Debug + Clone;
    type Error: Debug + Display;

    fn process(&self) -> Result<Self::Output, Self::Error>;
}

// Generic collection manager
struct RestaurantManager<T>
where
    T: Identifiable + Processable + Debug + Clone,
{
    items: HashMap<T::Id, T>,
    processing_queue: VecDeque<T::Id>,
    processed_results: HashMap<T::Id, T::Output>,
    name: String,
}

impl<T> RestaurantManager<T>
where
    T: Identifiable + Processable + Debug + Clone,
    T::Output: Debug + Clone,
    T::Error: Debug + Display,
{
    fn new(name: String) -> Self {
        RestaurantManager {
            items: HashMap::new(),
            processing_queue: VecDeque::new(),
            processed_results: HashMap::new(),
            name,
        }
    }

    fn add_item(&mut self, item: T) {
        let id = item.id().clone();
        self.processing_queue.push_back(id.clone());
        self.items.insert(id, item);
    }

    fn process_next(&mut self) -> Option<Result<String, String>> {
        let id = self.processing_queue.pop_front()?;

        if let Some(item) = self.items.get(&id) {
            match item.process() {
                Ok(output) => {
                    self.processed_results.insert(id.clone(), output.clone());
                    Some(Ok(format!("✅ Processed {}: {:?}", id, output)))
                }
                Err(error) => {
                    Some(Err(format!("❌ Failed to process {}: {}", id, error)))
                }
            }
        } else {
            Some(Err(format!("❌ Item {} not found", id)))
        }
    }

    fn get_stats(&self) -> String {
        format!("{}: {} items, {} in queue, {} processed",
                self.name,
                self.items.len(),
                self.processing_queue.len(),
                self.processed_results.len())
    }
}

// TODO: Create types that implement the required traits
#[derive(Debug, Clone)]
struct Order {
    id: u32,
    items: Vec<String>,
    customer: String,
    status: String,
}

impl Identifiable for Order {
    type Id = u32;

    fn id(&self) -> &Self::Id {
        &self.id
    }
}

impl Processable for Order {
    type Output = String;
    type Error = String;

    fn process(&self) -> Result<Self::Output, Self::Error> {
        if self.items.is_empty() {
            Err("Order has no items".to_string())
        } else if self.customer.is_empty() {
            Err("Order has no customer".to_string())
        } else {
            Ok(format!("Order {} processed: {} items for {}",
                      self.id, self.items.len(), self.customer))
        }
    }
}

fn exercise_3_2_solution() {
    println!("🗂️ Exercise 3.2: Generic Collections with Constraints");
    println!("{:=<50}", "");

    let mut order_manager = RestaurantManager::new("Order Processing".to_string());

    // Add orders
    order_manager.add_item(Order {
        id: 1001,
        items: vec!["Pizza".to_string(), "Salad".to_string()],
        customer: "John Doe".to_string(),
        status: "pending".to_string(),
    });

    order_manager.add_item(Order {
        id: 1002,
        items: vec![],  // Empty - will cause error
        customer: "Jane Smith".to_string(),
        status: "pending".to_string(),
    });

    order_manager.add_item(Order {
        id: 1003,
        items: vec!["Burger".to_string()],
        customer: "Bob Johnson".to_string(),
        status: "pending".to_string(),
    });

    println!("Initial stats: {}", order_manager.get_stats());

    // Process all items
    while let Some(result) = order_manager.process_next() {
        match result {
            Ok(success) => println!("{}", success),
            Err(error) => println!("{}", error),
        }
    }

    println!("Final stats: {}", order_manager.get_stats());

    println!("\n📚 What you learned:");
    println!("• Generic collections can manage any type meeting constraints");
    println!("• Associated types provide flexible trait design");
    println!("• Complex generic systems maintain type safety");
    println!("• Error handling integrates seamlessly with generics\n");
}
```


## Exercise Set 4: Real-World Integration Challenges

### Master Chef Certification (Professional Applications)

**Exercise 4.1: Plugin System Architecture**
*Difficulty: Expert*

Build a generic plugin system that demonstrates advanced generic programming.

```rust
// 🎯 EXERCISE: Complete the generic plugin system
trait Plugin<Input, Output> {
    type Config: Debug + Clone;
    type Error: Debug + Display;

    fn name(&self) -> &str;
    fn configure(&mut self, config: Self::Config) -> Result<(), Self::Error>;
    fn process(&self, input: Input) -> Result<Output, Self::Error>;
    fn priority(&self) -> u8; // 1-10, higher = more important
}

trait PluginManager<I, O> {
    type PluginType: Plugin<I, O>;

    fn register_plugin(&mut self, plugin: Self::PluginType);
    fn process_with_plugins(&self, input: I) -> Vec<Result<O, String>>;
}

// Generic plugin system
struct RestaurantPluginSystem<I, O, P>
where
    P: Plugin<I, O>,
    P::Error: Debug + Display,
{
    plugins: Vec<P>,
    system_name: String,
    _phantom: std::marker::PhantomData<(I, O)>,
}

impl<I, O, P> RestaurantPluginSystem<I, O, P>
where
    P: Plugin<I, O>,
    P::Error: Debug + Display,
    I: Clone + Debug,
    O: Debug,
{
    fn new(system_name: String) -> Self {
        RestaurantPluginSystem {
            plugins: Vec::new(),
            system_name,
            _phantom: std::marker::PhantomData,
        }
    }

    fn add_plugin(&mut self, plugin: P) {
        self.plugins.push(plugin);
        // Sort by priority (highest first)
        self.plugins.sort_by(|a, b| b.priority().cmp(&a.priority()));
    }

    fn process_input(&self, input: I) -> Vec<String> {
        let mut results = Vec::new();

        for plugin in &self.plugins {
            let result = match plugin.process(input.clone()) {
                Ok(output) => format!("✅ [{}] Success: {:?}", plugin.name(), output),
                Err(error) => format!("❌ [{}] Error: {}", plugin.name(), error),
            };
            results.push(result);
        }

        results
    }
}

// TODO: Implement concrete plugins
#[derive(Debug)]
struct MenuValidationPlugin {
    name: String,
    min_price: f64,
}

impl Plugin<(String, f64), String> for MenuValidationPlugin {
    type Config = f64;  // Minimum price configuration
    type Error = String;

    fn name(&self) -> &str {
        &self.name
    }

    fn configure(&mut self, config: Self::Config) -> Result<(), Self::Error> {
        if config <= 0.0 {
            Err("Minimum price must be positive".to_string())
        } else {
            self.min_price = config;
            Ok(())
        }
    }

    fn process(&self, input: (String, f64)) -> Result<String, Self::Error> {
        let (item_name, price) = input;

        if item_name.is_empty() {
            Err("Item name cannot be empty".to_string())
        } else if price < self.min_price {
            Err(format!("Price ${:.2} below minimum ${:.2}", price, self.min_price))
        } else {
            Ok(format!("Menu item '{}' validated at ${:.2}", item_name, price))
        }
    }

    fn priority(&self) -> u8 {
        10 // High priority - validate first
    }
}

#[derive(Debug)]
struct PricingPlugin {
    name: String,
    tax_rate: f64,
}

impl Plugin<(String, f64), (String, f64)> for PricingPlugin {
    type Config = f64;  // Tax rate configuration
    type Error = String;

    fn name(&self) -> &str {
        &self.name
    }

    fn configure(&mut self, config: Self::Config) -> Result<(), Self::Error> {
        if config < 0.0 || config > 1.0 {
            Err("Tax rate must be between 0.0 and 1.0".to_string())
        } else {
            self.tax_rate = config;
            Ok(())
        }
    }

    fn process(&self, input: (String, f64)) -> Result<(String, f64), Self::Error> {
        let (item_name, base_price) = input;
        let final_price = base_price * (1.0 + self.tax_rate);
        Ok((item_name, final_price))
    }

    fn priority(&self) -> u8 {
        5 // Medium priority
    }
}

fn exercise_4_1_solution() {
    println!("🔌 Exercise 4.1: Generic Plugin System");
    println!("{:=<50}", "");

    // Create validation system
    let mut validation_system = RestaurantPluginSystem::new("Menu Validation".to_string());

    let mut validator = MenuValidationPlugin {
        name: "Price Validator".to_string(),
        min_price: 5.0,
    };
    validator.configure(8.0).unwrap();

    validation_system.add_plugin(validator);

    // Test validation
    let test_items = vec![
        ("Caesar Salad".to_string(), 12.99),
        ("".to_string(), 10.00),  // Invalid name
        ("Soup".to_string(), 3.99),  // Too cheap
    ];

    for item in test_items {
        let results = validation_system.process_input(item);
        for result in results {
            println!("{}", result);
        }
    }

    // Create pricing system
    let mut pricing_system = RestaurantPluginSystem::new("Pricing System".to_string());

    let mut pricer = PricingPlugin {
        name: "Tax Calculator".to_string(),
        tax_rate: 0.08,
    };
    pricer.configure(0.085).unwrap();

    pricing_system.add_plugin(pricer);

    // Test pricing
    println!("\nPricing results:");
    let pricing_results = pricing_system.process_input(("Pizza Margherita".to_string(), 15.99));
    for result in pricing_results {
        println!("{}", result);
    }

    println!("\n📚 What you learned:");
    println!("• Generic plugin systems enable flexible architecture");
    println!("• Associated types provide configuration flexibility");
    println!("• Complex generic constraints enable type-safe extensibility");
    println!("• Real-world systems benefit from generic design patterns\n");
}
```

**Exercise 4.2: Complete Restaurant Management System**
*Difficulty: Master*

Integrate all concepts into a comprehensive system.

```rust
// 🎯 EXERCISE: Complete the master-level restaurant system
use std::collections::{HashMap, BTreeMap};

// Core traits for the system
trait Entity {
    type Id: Clone + Eq + std::hash::Hash + Debug + Display + Ord;
    fn id(&self) -> &Self::Id;
    fn validate(&self) -> Result<(), String>;
}

trait Processable<Context> {
    type Output: Debug + Clone;
    type Error: Debug + Display;

    fn process(&self, context: &Context) -> Result<Self::Output, Self::Error>;
}

trait Schedulable {
    fn estimated_duration(&self) -> u32; // minutes
    fn priority(&self) -> u8; // 1-10
    fn dependencies(&self) -> Vec<String>; // What must be completed first
}

// Master restaurant management system
struct RestaurantSystem<E, C>
where
    E: Entity + Processable<C> + Schedulable + Debug + Clone,
    E::Output: Debug + Clone,
    E::Error: Debug + Display,
    C: Debug + Clone,
{
    entities: HashMap<E::Id, E>,
    processing_context: C,
    schedule: BTreeMap<u32, Vec<E::Id>>, // time -> entities to process
    completed: HashMap<E::Id, E::Output>,
    system_name: String,
}

impl<E, C> RestaurantSystem<E, C>
where
    E: Entity + Processable<C> + Schedulable + Debug + Clone,
    E::Output: Debug + Clone,
    E::Error: Debug + Display,
    C: Debug + Clone,
{
    fn new(name: String, context: C) -> Self {
        RestaurantSystem {
            entities: HashMap::new(),
            processing_context: context,
            schedule: BTreeMap::new(),
            completed: HashMap::new(),
            system_name: name,
        }
    }

    fn add_entity(&mut self, entity: E) -> Result<(), String> {
        entity.validate()?;

        let id = entity.id().clone();
        let priority_time = (entity.priority() as u32) * 100 + entity.estimated_duration();

        self.schedule
            .entry(priority_time)
            .or_insert_with(Vec::new)
            .push(id.clone());

        self.entities.insert(id, entity);
        Ok(())
    }

    fn process_all(&mut self) -> Vec<String> {
        let mut results = Vec::new();

        // Process in priority order
        let schedule_keys: Vec<u32> = self.schedule.keys().cloned().collect();

        for time_slot in schedule_keys {
            if let Some(entity_ids) = self.schedule.remove(&time_slot) {
                for entity_id in entity_ids {
                    if let Some(entity) = self.entities.get(&entity_id) {
                        match entity.process(&self.processing_context) {
                            Ok(output) => {
                                self.completed.insert(entity_id.clone(), output.clone());
                                results.push(format!("✅ Processed {}: {:?}", entity_id, output));
                            }
                            Err(error) => {
                                results.push(format!("❌ Failed {}: {}", entity_id, error));
                            }
                        }
                    }
                }
            }
        }

        results
    }

    fn get_system_stats(&self) -> String {
        format!("{}: {} entities, {} completed, {} scheduled",
                self.system_name,
                self.entities.len(),
                self.completed.len(),
                self.schedule.values().map(|v| v.len()).sum::<usize>())
    }
}

// TODO: Implement a complete entity type
#[derive(Debug, Clone)]
struct RestaurantTask {
    id: String,
    description: String,
    task_type: String,
    complexity: u32,
    required_skills: Vec<String>,
}

#[derive(Debug, Clone)]
struct ProcessingContext {
    available_staff: u32,
    time_of_day: String,
    equipment_status: HashMap<String, bool>,
}

impl Entity for RestaurantTask {
    type Id = String;

    fn id(&self) -> &Self::Id {
        &self.id
    }

    fn validate(&self) -> Result<(), String> {
        if self.description.is_empty() {
            Err("Task description cannot be empty".to_string())
        } else if self.complexity > 10 {
            Err("Task complexity cannot exceed 10".to_string())
        } else {
            Ok(())
        }
    }
}

impl Processable<ProcessingContext> for RestaurantTask {
    type Output = String;
    type Error = String;

    fn process(&self, context: &ProcessingContext) -> Result<Self::Output, Self::Error> {
        // Check if we have enough staff
        let required_staff = (self.complexity + 1) / 2;
        if context.available_staff < required_staff {
            return Err(format!("Need {} staff, only {} available", required_staff, context.available_staff));
        }

        // Check if required equipment is available
        for skill in &self.required_skills {
            if let Some(&available) = context.equipment_status.get(skill) {
                if !available {
                    return Err(format!("Equipment for {} not available", skill));
                }
            }
        }

        Ok(format!("Task '{}' completed successfully during {} with {} staff",
                  self.description, context.time_of_day, required_staff))
    }
}

impl Schedulable for RestaurantTask {
    fn estimated_duration(&self) -> u32 {
        self.complexity * 5 // 5 minutes per complexity point
    }

    fn priority(&self) -> u8 {
        match self.task_type.as_str() {
            "urgent" => 10,
            "important" => 8,
            "normal" => 5,
            "maintenance" => 3,
            _ => 1,
        }
    }

    fn dependencies(&self) -> Vec<String> {
        // For this example, no dependencies
        vec![]
    }
}

fn exercise_4_2_solution() {
    println!("🏆 Exercise 4.2: Master Restaurant Management System");
    println!("{:=<50}", "");

    let context = ProcessingContext {
        available_staff: 3,
        time_of_day: "lunch rush".to_string(),
        equipment_status: [
            ("grill".to_string(), true),
            ("fryer".to_string(), true),
            ("oven".to_string(), false), // Broken
        ].iter().cloned().collect(),
    };

    let mut restaurant_system = RestaurantSystem::new(
        "Main Kitchen Operations".to_string(),
        context
    );

    // Add various tasks
    let tasks = vec![
        RestaurantTask {
            id: "TASK001".to_string(),
            description: "Prepare lunch specials".to_string(),
            task_type: "urgent".to_string(),
            complexity: 6,
            required_skills: vec!["grill".to_string()],
        },
        RestaurantTask {
            id: "TASK002".to_string(),
            description: "Clean equipment".to_string(),
            task_type: "maintenance".to_string(),
            complexity: 2,
            required_skills: vec![],
        },
        RestaurantTask {
            id: "TASK003".to_string(),
            description: "Bake desserts".to_string(),
            task_type: "normal".to_string(),
            complexity: 8,
            required_skills: vec!["oven".to_string()], // Will fail - oven broken
        },
    ];

    // Add tasks to system
    for task in tasks {
        match restaurant_system.add_entity(task) {
            Ok(()) => println!("✅ Task added successfully"),
            Err(e) => println!("❌ Failed to add task: {}", e),
        }
    }

    println!("\nInitial stats: {}", restaurant_system.get_system_stats());

    // Process all tasks
    println!("\nProcessing results:");
    let results = restaurant_system.process_all();
    for result in results {
        println!("{}", result);
    }

    println!("\nFinal stats: {}", restaurant_system.get_system_stats());

    println!("\n🎓 Master Level Achievement Unlocked!");
    println!("📚 What you mastered:");
    println!("• Complex generic system architecture");
    println!("• Multiple trait bounds working together");
    println!("• Associated types providing flexibility");
    println!("• Real-world constraint management");
    println!("• Professional-grade error handling");
    println!("• Type-safe polymorphism at scale\n");
}
```


## Running the Complete Exercise Set

### Progressive Learning Path

```rust
fn main() {
    println!("🎯 Rust Generic Programming: Complete Exercise Set");
    println!("=====================================================\n");

    println!("Starting your journey from basic concepts to master level...\n");

    // Exercise Set 1: Fundamentals
    println!("🎓 LEVEL 1: BASIC GENERICS");
    println!("=" .repeat(40));
    exercise_1_1_solution();
    exercise_1_2_solution();
    exercise_1_3_solution();

    // Exercise Set 2: Traits
    println!("🎓 LEVEL 2: TRAITS & IMPLEMENTATIONS");
    println!("=" .repeat(40));
    exercise_2_1_solution();
    exercise_2_2_solution();
    exercise_2_3_solution();

    // Exercise Set 3: Advanced Constraints
    println!("🎓 LEVEL 3: ADVANCED CONSTRAINTS");
    println!("=" .repeat(40));
    exercise_3_1_solution();
    exercise_3_2_solution();

    // Exercise Set 4: Master Level
    println!("🎓 LEVEL 4: MASTER APPLICATIONS");
    println!("=" .repeat(40));
    exercise_4_1_solution();
    exercise_4_2_solution();

    println!("🏆 CONGRATULATIONS!");
    println!("You've completed the comprehensive Rust generic programming exercise set!");
    println!("You're now equipped to build sophisticated, type-safe systems in Rust.");
}
```


## Summary and Key Learning Outcomes

### **Mental Model: The Complete Professional Restaurant Training Achievement**

Through these progressive exercises, you've built skills like a **complete professional restaurant training program** - from basic kitchen tools to master chef capabilities:

- 🥄 **Basic Tools** - Simple generics and type parameters
- 🍳 **Professional Standards** - Traits and implementations
- 👨🍳 **Complex Operations** - Advanced constraints and where clauses
- 🏆 **Master Chef** - Real-world systems integration


### **Essential Skills Gained**

**Generic Programming Mastery:**

- Creating and using generic functions, structs, and enums
- Understanding type parameters and their constraints
- Implementing methods on generic types
- Working with multiple generic parameters

**Trait System Expertise:**

- Defining traits for shared behavior
- Implementing traits for different types
- Using trait bounds to constrain generic types
- Combining multiple traits effectively

**Advanced Constraint Management:**

- Complex where clauses for sophisticated requirements
- Associated types for flexible trait design
- Generic collections with type safety
- Professional error handling patterns


### **The Professional Advantage**

**Completing these exercises transforms you from a beginner to a confident Rust generic programmer** who can build sophisticated, type-safe systems:

- 🎯 **Practical Experience** - Hands-on practice with real-world scenarios
- 📈 **Progressive Mastery** - Each exercise builds on previous concepts
- 🛡️ **Type Safety** - Deep understanding of Rust's type system guarantees
- ⚡ **Performance** - Knowledge of zero-cost abstractions and optimization
- 🏗️ **Architecture** - Ability to design scalable, maintainable systems

**Understanding generics through practice enables you to write code that is not just correct and fast, but also flexible, reusable, and maintainable** - the hallmarks of professional Rust development. Whether you're building simple utilities or complex enterprise systems, these skills form the foundation of excellent Rust programming!

1. https://www.linkedin.com/pulse/mastering-generics-rust-hands-onguide-luis-soares-m-sc-
2. https://doc.rust-lang.org/book/ch10-01-syntax.html
3. https://practice.course.rs/generics-traits/generics.html
4. https://rust-exercises.com/100-exercises/
5. https://www.geeksforgeeks.org/rust/rust-generic-traits/
6. https://dev.to/ajtech0001/mastering-generic-types-and-traits-in-rust-a-complete-guide-4e2n
7. https://rust-exercises.com/100-exercises-to-learn-rust.pdf
8. https://www.programiz.com/rust/generics
9. https://www.hyperexponential.com/blog/rust-language-exercises/
10. https://www.reddit.com/r/rust/comments/1ctepo1/100_exercises_to_learn_rust_a_new_learnbydoing/
11. https://rust-exercises.com/100-exercises/04_traits/01_trait.html
12. https://stackoverflow.com/questions/40776020/is-there-any-way-to-restrict-a-generic-type-to-one-of-several-types
13. https://practice.course.rs
14. https://teach-rs.trifectatech.org/full/traits-and-generics.html
15. https://users.rust-lang.org/t/confused-about-traits-generics-and-versus-associated-types/77356
16. https://updraft.cyfrin.io/courses/rust-programming-basics/generic-types-and-traits/generic-types-exercises
17. https://updraft.cyfrin.io/courses/rust-programming-basics/generic-types-and-traits/lifetime-exercises
18. https://github.com/skyzh/type-exercise-in-rust
