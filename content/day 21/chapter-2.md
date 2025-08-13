+++
title = "Generic Structs and Enums"
description = "Learn how to define and use generic structs and enums to handle multiple data types efficiently."
date = "2025-08-13"
weight = 2
+++

# Generic Structs and Enums in Rust: Comprehensive Documentation for Beginners

Understanding generic structs and enums in Rust is like learning to **design flexible restaurant equipment and standardized processes that can handle different types of ingredients while maintaining the same structure and safety standards**. Just as a professional restaurant needs both flexible storage containers that can hold various ingredients (structs) and standardized process workflows that can handle different outcomes like success or failure (enums), Rust's generic structs and enums allow you to create data structures and type definitions that work with multiple data types while maintaining type safety and code reusability.

## The Professional Restaurant Flexible Systems Analogy 🏪

### Imagine You're Designing Adaptable Restaurant Infrastructure

**The Problem with Fixed-Type Infrastructure:**

```rust
// ❌ Fixed approach - like having separate containers for each ingredient type
struct TomatoContainer {
    contents: Vec<String>,  // Only tomatoes
    capacity: usize,
}

struct CheeseContainer {
    contents: Vec<String>,  // Only cheese
    capacity: usize,
}

enum PizzaResult {
    Success(String),    // Only pizza outcomes
    Failure(String),
}

enum SaladResult {
    Success(String),    // Only salad outcomes
    Failure(String),
}
// Duplicate code for every ingredient and dish type!
```

**The Power of Generic Structs and Enums - Universal Restaurant Systems:**

```rust
// ✅ Generic approach - universal containers and processes
struct Container<T> {
    contents: Vec<T>,    // Any ingredient type
    capacity: usize,
}

enum CookingResult<T, E> {
    Success(T),          // Any success type
    Failure(E),          // Any error type
}

fn universal_restaurant_demo() {
    // Same container design, different contents!
    let tomato_container = Container {
        contents: vec!["Roma Tomato", "Cherry Tomato"],
        capacity: 100,
    };

    let price_container = Container {
        contents: vec![15.99, 12.50, 18.75],  // Numbers instead of strings
        capacity: 50,
    };

    // Same result pattern, different outcome types!
    let pizza_result: CookingResult<String, String> =
        CookingResult::Success("Perfect Margherita Pizza".to_string());

    let temperature_result: CookingResult<f64, String> =
        CookingResult::Success(165.0);  // Temperature reading
}
```

**Why Generic Structs and Enums Are Essential:**

- 🔄 **Reusability** - One definition works with many types
- 🛡️ **Type safety** - Compile-time guarantees for each usage
- 📦 **Memory efficiency** - Optimized layouts for each concrete type
- 🎯 **Maintainability** - Single source of truth for structure
- ⚡ **Performance** - Zero-cost abstractions through monomorphization


## Generic Structs Fundamentals

### Universal Storage Containers for Your Restaurant

**Understanding how to create and use flexible data structures:**

```rust
fn demonstrate_generic_structs_fundamentals() {
    println!("📦 Generic Structs Fundamentals - Universal Storage Systems");
    println!("{:=<70}", "");

    // Generic structs are like universal storage containers
    println!("🏪 Generic Struct Components:");
    println!("   • struct Name<T> = Struct with type parameter T");
    println!("   • field: T = Field of generic type T");
    println!("   • impl<T> = Implementation for all types T");
    println!("   • impl Name<ConcreteType> = Implementation for specific type");

    // Example 1: Simple Generic Container - Universal Storage Box
    println!("\n1️⃣ Simple Generic Container - Universal Storage Box:");

    #[derive(Debug, Clone)]
    struct StorageBox<T> {
        item: T,
        label: String,
        capacity: usize,
    }

    impl<T> StorageBox<T> {
        fn new(item: T, label: String, capacity: usize) -> Self {
            StorageBox { item, label, capacity }
        }

        fn get_item(&self) -> &T {
            &self.item
        }

        fn get_label(&self) -> &str {
            &self.label
        }

        fn update_item(&mut self, new_item: T) {
            self.item = new_item;
        }
    }

    // Works with any type!
    let vegetable_box = StorageBox::new(
        "Fresh Spinach".to_string(),
        "Vegetables".to_string(),
        100
    );

    let quantity_box = StorageBox::new(
        25,
        "Inventory Count".to_string(),
        1000
    );

    let price_box = StorageBox::new(
        15.99,
        "Menu Prices".to_string(),
        50
    );

    println!("   Vegetable box: {:?}", vegetable_box);
    println!("   Quantity box: {:?}", quantity_box);
    println!("   Price box: {:?}", price_box);

    println!("   Vegetable item: {}", vegetable_box.get_item());
    println!("   Quantity item: {}", quantity_box.get_item());
    println!("   Price item: ${}", price_box.get_item());

    // Example 2: Multiple Generic Parameters - Multi-Compartment Storage
    println!("\n2️⃣ Multiple Generic Parameters - Multi-Compartment Storage:");

    #[derive(Debug)]
    struct DualContainer<T, U> {
        primary: T,
        secondary: U,
        container_id: String,
    }

    impl<T, U> DualContainer<T, U> {
        fn new(primary: T, secondary: U, id: &str) -> Self {
            DualContainer {
                primary,
                secondary,
                container_id: id.to_string(),
            }
        }

        fn get_primary(&self) -> &T {
            &self.primary
        }

        fn get_secondary(&self) -> &U {
            &self.secondary
        }

        fn swap_items(self) -> DualContainer<U, T> {
            DualContainer {
                primary: self.secondary,
                secondary: self.primary,
                container_id: format!("swapped-{}", self.container_id),
            }
        }
    }

    let ingredient_price_pair = DualContainer::new(
        "Organic Quinoa".to_string(),
        12.99,
        "ingredient-001"
    );

    let quantity_available_pair = DualContainer::new(
        50,      // quantity
        true,    // available
        "inventory-001"
    );

    println!("   Ingredient-Price: {:?}", ingredient_price_pair);
    println!("   Primary: {}, Secondary: ${}",
             ingredient_price_pair.get_primary(),
             ingredient_price_pair.get_secondary());

    println!("   Quantity-Available: {:?}", quantity_available_pair);

    // Swap the types
    let swapped = ingredient_price_pair.swap_items();
    println!("   After swap: {:?}", swapped);

    // Example 3: Generic Struct with Constraints - Quality-Controlled Storage
    println!("\n3️⃣ Generic Constraints - Quality-Controlled Storage:");

    use std::fmt::Display;

    #[derive(Debug)]
    struct LabeledContainer<T>
    where
        T: Display + Clone,
    {
        contents: Vec<T>,
        description: String,
        max_capacity: usize,
    }

    impl<T> LabeledContainer<T>
    where
        T: Display + Clone,
    {
        fn new(description: &str, max_capacity: usize) -> Self {
            LabeledContainer {
                contents: Vec::new(),
                description: description.to_string(),
                max_capacity,
            }
        }

        fn add_item(&mut self, item: T) -> Result<(), String> {
            if self.contents.len() >= self.max_capacity {
                return Err(format!("Container '{}' is at maximum capacity", self.description));
            }

            self.contents.push(item);
            Ok(())
        }

        fn list_contents(&self) -> String {
            if self.contents.is_empty() {
                format!("Container '{}' is empty", self.description)
            } else {
                let items: Vec<String> = self.contents
                    .iter()
                    .enumerate()
                    .map(|(i, item)| format!("  {}. {}", i + 1, item))
                    .collect();
                format!("Container '{}':\n{}", self.description, items.join("\n"))
            }
        }

        fn duplicate_item(&self, index: usize) -> Option<T> {
            self.contents.get(index).map(|item| item.clone())
        }
    }

    let mut ingredient_container = LabeledContainer::new("Fresh Vegetables", 5);
    let mut price_container = LabeledContainer::new("Daily Prices", 3);

    // Add items to containers
    ingredient_container.add_item("Tomatoes").unwrap();
    ingredient_container.add_item("Spinach").unwrap();
    ingredient_container.add_item("Carrots").unwrap();

    price_container.add_item(15.99).unwrap();
    price_container.add_item(12.50).unwrap();

    println!("{}", ingredient_container.list_contents());
    println!("{}", price_container.list_contents());

    // Test capacity limit
    match ingredient_container.add_item("Broccoli") {
        Ok(()) => println!("   Added broccoli successfully"),
        Err(e) => println!("   Failed to add: {}", e),
    }

    // Test duplication
    if let Some(duplicated) = ingredient_container.duplicate_item(0) {
        println!("   Duplicated item: {}", duplicated);
    }

    // Example 4: Nested Generic Structs - Complex Storage Systems
    println!("\n4️⃣ Nested Generic Structs - Complex Storage Systems:");

    #[derive(Debug)]
    struct Shelf<T> {
        items: Vec<StorageBox<T>>,
        shelf_id: String,
    }

    #[derive(Debug)]
    struct Warehouse<T> {
        shelves: Vec<Shelf<T>>,
        warehouse_name: String,
    }

    impl<T> Shelf<T> {
        fn new(shelf_id: &str) -> Self {
            Shelf {
                items: Vec::new(),
                shelf_id: shelf_id.to_string(),
            }
        }

        fn add_box(&mut self, storage_box: StorageBox<T>) {
            self.items.push(storage_box);
        }

        fn count_boxes(&self) -> usize {
            self.items.len()
        }
    }

    impl<T> Warehouse<T> {
        fn new(name: &str) -> Self {
            Warehouse {
                shelves: Vec::new(),
                warehouse_name: name.to_string(),
            }
        }

        fn add_shelf(&mut self, shelf: Shelf<T>) {
            self.shelves.push(shelf);
        }

        fn total_boxes(&self) -> usize {
            self.shelves.iter().map(|shelf| shelf.count_boxes()).sum()
        }

        fn get_summary(&self) -> String {
            format!("Warehouse '{}': {} shelves, {} total boxes",
                   self.warehouse_name,
                   self.shelves.len(),
                   self.total_boxes())
        }
    }

    // Create a string-based warehouse system
    let mut ingredient_warehouse = Warehouse::new("Fresh Ingredients");

    let mut vegetables_shelf = Shelf::new("A1-Vegetables");
    vegetables_shelf.add_box(StorageBox::new("Tomatoes", "Red Vegetables", 50));
    vegetables_shelf.add_box(StorageBox::new("Spinach", "Green Vegetables", 30));

    let mut herbs_shelf = Shelf::new("A2-Herbs");
    herbs_shelf.add_box(StorageBox::new("Basil", "Italian Herbs", 20));

    ingredient_warehouse.add_shelf(vegetables_shelf);
    ingredient_warehouse.add_shelf(herbs_shelf);

    println!("   {}", ingredient_warehouse.get_summary());
    println!("   Warehouse structure: {:#?}", ingredient_warehouse);

    println!("\n🎯 Generic Struct Guidelines:");
    println!("   📦 Use <T> for single type parameter");
    println!("   🔄 Use <T, U, V> for multiple type parameters");
    println!("   🛡️ Apply constraints with where clauses");
    println!("   ⚙️ Implement methods for all types or specific types");
    println!("   🎨 Design for flexibility while maintaining type safety");
}

fn main() {
    demonstrate_generic_structs_fundamentals();
}
```


## Generic Enums Fundamentals

### Standardized Process Outcomes for Your Restaurant

**Creating flexible enums that can handle different types of results and states:**

```rust
fn demonstrate_generic_enums_fundamentals() {
    println!("🔄 Generic Enums Fundamentals - Standardized Process Workflows");
    println!("{:=<70}", "");

    // Generic enums are like standardized process outcomes that work with any type
    println!("⚙️ Generic Enum Components:");
    println!("   • enum Name<T> = Enum with type parameter T");
    println!("   • Variant(T) = Variant holding type T");
    println!("   • match = Pattern matching on generic variants");
    println!("   • Result<T, E> and Option<T> = Standard library examples");

    // Example 1: Simple Generic Enum - Universal Operation Result
    println!("\n1️⃣ Simple Generic Enum - Universal Operation Result:");

    #[derive(Debug, Clone, PartialEq)]
    enum OperationResult<T> {
        Success(T),
        Pending,
        Failed(String),
    }

    impl<T> OperationResult<T> {
        fn is_success(&self) -> bool {
            matches!(self, OperationResult::Success(_))
        }

        fn is_pending(&self) -> bool {
            matches!(self, OperationResult::Pending)
        }

        fn is_failed(&self) -> bool {
            matches!(self, OperationResult::Failed(_))
        }

        fn unwrap(self) -> T {
            match self {
                OperationResult::Success(value) => value,
                OperationResult::Pending => panic!("Called unwrap on Pending result"),
                OperationResult::Failed(error) => panic!("Called unwrap on Failed result: {}", error),
            }
        }

        fn unwrap_or(self, default: T) -> T {
            match self {
                OperationResult::Success(value) => value,
                _ => default,
            }
        }
    }

    // Test with different types
    let dish_preparation: OperationResult<String> =
        OperationResult::Success("Quinoa Buddha Bowl ready".to_string());

    let temperature_check: OperationResult<f64> =
        OperationResult::Success(165.0);

    let inventory_count: OperationResult<usize> =
        OperationResult::Failed("Database connection lost".to_string());

    let order_status: OperationResult<bool> =
        OperationResult::Pending;

    println!("   Dish preparation: {:?}", dish_preparation);
    println!("   Is success: {}", dish_preparation.is_success());

    println!("   Temperature check: {:?}", temperature_check);
    println!("   Temperature value: {}°F", temperature_check.clone().unwrap());

    println!("   Inventory count: {:?}", inventory_count);
    println!("   Is failed: {}", inventory_count.is_failed());

    println!("   Order status: {:?}", order_status);
    println!("   Is pending: {}", order_status.is_pending());

    // Example 2: Multiple Generic Parameters - Complex Process Results
    println!("\n2️⃣ Multiple Generic Parameters - Complex Process Results:");

    #[derive(Debug)]
    enum ProcessingResult<T, E, M> {
        Success { data: T, metadata: M },
        PartialSuccess { data: T, warnings: Vec<String>, metadata: M },
        Failed { error: E, context: M },
    }

    impl<T, E, M> ProcessingResult<T, E, M> {
        fn is_any_success(&self) -> bool {
            matches!(self, ProcessingResult::Success { .. } | ProcessingResult::PartialSuccess { .. })
        }

        fn get_metadata(&self) -> &M {
            match self {
                ProcessingResult::Success { metadata, .. } => metadata,
                ProcessingResult::PartialSuccess { metadata, .. } => metadata,
                ProcessingResult::Failed { context, .. } => context,
            }
        }

        fn map_data<U, F>(self, f: F) -> ProcessingResult<U, E, M>
        where
            F: FnOnce(T) -> U,
        {
            match self {
                ProcessingResult::Success { data, metadata } =>
                    ProcessingResult::Success { data: f(data), metadata },
                ProcessingResult::PartialSuccess { data, warnings, metadata } =>
                    ProcessingResult::PartialSuccess { data: f(data), warnings, metadata },
                ProcessingResult::Failed { error, context } =>
                    ProcessingResult::Failed { error, context },
            }
        }
    }

    #[derive(Debug)]
    struct OrderMetadata {
        order_id: String,
        timestamp: String,
        customer_id: String,
    }

    let metadata = OrderMetadata {
        order_id: "ORD-001".to_string(),
        timestamp: "2024-01-15T14:30:00Z".to_string(),
        customer_id: "CUST-123".to_string(),
    };

    // Successful order processing
    let successful_order: ProcessingResult<String, String, OrderMetadata> =
        ProcessingResult::Success {
            data: "Order completed successfully".to_string(),
            metadata: metadata.clone(),
        };

    // Partial success with warnings
    let partial_order: ProcessingResult<String, String, OrderMetadata> =
        ProcessingResult::PartialSuccess {
            data: "Order completed with substitutions".to_string(),
            warnings: vec![
                "Requested item out of stock, substituted with similar item".to_string(),
                "Delivery delayed by 15 minutes".to_string(),
            ],
            metadata: metadata.clone(),
        };

    // Failed order
    let failed_order: ProcessingResult<String, String, OrderMetadata> =
        ProcessingResult::Failed {
            error: "Payment declined".to_string(),
            context: metadata,
        };

    println!("   Successful order: {:?}", successful_order);
    println!("   Is success: {}", successful_order.is_any_success());

    println!("   Partial order: {:?}", partial_order);

    println!("   Failed order: {:?}", failed_order);
    println!("   Order ID from metadata: {}", failed_order.get_metadata().order_id);

    // Example 3: Generic Enums with Standard Library Patterns
    println!("\n3️⃣ Standard Library Pattern Usage:");

    // Demonstrating Option<T> usage in restaurant context
    fn find_ingredient_price(ingredient: &str) -> Option<f64> {
        let prices = [
            ("tomatoes", 3.99),
            ("quinoa", 8.50),
            ("spinach", 2.75),
        ];

        prices.iter()
            .find(|(name, _)| *name == ingredient)
            .map(|(_, price)| *price)
    }

    // Demonstrating Result<T, E> usage
    fn calculate_recipe_cost(recipe: &[&str]) -> Result<f64, String> {
        let mut total_cost = 0.0;

        for ingredient in recipe {
            match find_ingredient_price(ingredient) {
                Some(price) => total_cost += price,
                None => return Err(format!("Price not found for ingredient: {}", ingredient)),
            }
        }

        if total_cost > 0.0 {
            Ok(total_cost)
        } else {
            Err("Recipe has no valid ingredients".to_string())
        }
    }

    let quinoa_recipe = ["quinoa", "tomatoes", "spinach"];
    let invalid_recipe = ["quinoa", "unicorn_tears"];

    // Test ingredient price lookup
    println!("   Price of tomatoes: {:?}", find_ingredient_price("tomatoes"));
    println!("   Price of caviar: {:?}", find_ingredient_price("caviar"));

    // Test recipe cost calculation
    match calculate_recipe_cost(&quinoa_recipe) {
        Ok(cost) => println!("   Quinoa recipe cost: ${:.2}", cost),
        Err(error) => println!("   Recipe cost error: {}", error),
    }

    match calculate_recipe_cost(&invalid_recipe) {
        Ok(cost) => println!("   Invalid recipe cost: ${:.2}", cost),
        Err(error) => println!("   Recipe error: {}", error),
    }

    // Example 4: Custom Generic Enums for Domain Logic
    println!("\n4️⃣ Custom Generic Enums for Domain Logic:");

    #[derive(Debug)]
    enum InventoryStatus<T> {
        InStock(T),
        LowStock(T, u32),  // item, threshold
        OutOfStock,
        Discontinued(String),  // reason
    }

    impl<T> InventoryStatus<T> {
        fn is_available(&self) -> bool {
            matches!(self, InventoryStatus::InStock(_) | InventoryStatus::LowStock(_, _))
        }

        fn get_item(&self) -> Option<&T> {
            match self {
                InventoryStatus::InStock(item) => Some(item),
                InventoryStatus::LowStock(item, _) => Some(item),
                _ => None,
            }
        }
    }

    let tomato_status = InventoryStatus::InStock("Fresh Roma Tomatoes".to_string());
    let quinoa_status = InventoryStatus::LowStock("Organic Quinoa".to_string(), 5);
    let truffle_status: InventoryStatus<String> = InventoryStatus::OutOfStock;
    let old_ingredient: InventoryStatus<String> =
        InventoryStatus::Discontinued("Seasonal ingredient no longer available".to_string());

    let statuses = [
        ("Tomatoes", &tomato_status),
        ("Quinoa", &quinoa_status),
        ("Truffles", &truffle_status),
        ("Old Item", &old_ingredient),
    ];

    for (name, status) in statuses {
        println!("   {} status: {:?}", name, status);
        println!("     Available: {}", status.is_available());
        if let Some(item) = status.get_item() {
            println!("     Item: {}", item);
        }
    }

    println!("\n🎯 Generic Enum Guidelines:");
    println!("   🔄 Use for representing different states with same structure");
    println!("   📦 Leverage Option<T> for nullable values");
    println!("   ⚠️ Use Result<T, E> for operations that can fail");
    println!("   🎨 Create domain-specific enums for business logic");
    println!("   ⚙️ Implement methods for common operations");
}

fn main() {
    demonstrate_generic_enums_fundamentals();
}
```


## Advanced Generic Patterns for Structs and Enums

### Professional Restaurant System Architecture

**Sophisticated patterns for building complex, type-safe restaurant management systems:**

```rust
fn demonstrate_advanced_generic_patterns() {
    println!("🚀 Advanced Generic Patterns - Professional Restaurant Architecture");
    println!("{:=<75}", "");

    use std::marker::PhantomData;
    use std::collections::HashMap;

    // Advanced patterns are like sophisticated restaurant management architectures
    println!("🏗️ Professional Generic System Patterns:");

    // Pattern 1: Generic Structs with Associated Types
    println!("\n1️⃣ Generic Structs with Associated Types:");

    trait Storage {
        type Item;
        type Key;

        fn store(&mut self, key: Self::Key, item: Self::Item) -> Result<(), String>;
        fn retrieve(&self, key: &Self::Key) -> Option<&Self::Item>;
        fn remove(&mut self, key: &Self::Key) -> Option<Self::Item>;
        fn count(&self) -> usize;
    }

    #[derive(Debug)]
    struct RestaurantStorage<K, V> {
        data: HashMap<K, V>,
        capacity: usize,
        storage_type: String,
    }

    impl<K, V> RestaurantStorage<K, V>
    where
        K: std::hash::Hash + Eq + Clone,
        V: Clone,
    {
        fn new(capacity: usize, storage_type: &str) -> Self {
            RestaurantStorage {
                data: HashMap::new(),
                capacity,
                storage_type: storage_type.to_string(),
            }
        }

        fn is_full(&self) -> bool {
            self.data.len() >= self.capacity
        }

        fn utilization(&self) -> f64 {
            if self.capacity == 0 {
                0.0
            } else {
                self.data.len() as f64 / self.capacity as f64
            }
        }
    }

    impl<K, V> Storage for RestaurantStorage<K, V>
    where
        K: std::hash::Hash + Eq + Clone,
        V: Clone,
    {
        type Item = V;
        type Key = K;

        fn store(&mut self, key: Self::Key, item: Self::Item) -> Result<(), String> {
            if self.is_full() && !self.data.contains_key(&key) {
                return Err(format!("Storage '{}' is at capacity", self.storage_type));
            }

            self.data.insert(key, item);
            Ok(())
        }

        fn retrieve(&self, key: &Self::Key) -> Option<&Self::Item> {
            self.data.get(key)
        }

        fn remove(&mut self, key: &Self::Key) -> Option<Self::Item> {
            self.data.remove(key)
        }

        fn count(&self) -> usize {
            self.data.len()
        }
    }

    // Test the storage system
    let mut ingredient_storage = RestaurantStorage::new(5, "Ingredient Inventory");
    let mut price_storage = RestaurantStorage::new(10, "Price Database");

    // Store different types
    ingredient_storage.store("tomatoes".to_string(), "Fresh Roma Tomatoes".to_string()).unwrap();
    ingredient_storage.store("quinoa".to_string(), "Organic Quinoa".to_string()).unwrap();

    price_storage.store("ITEM001".to_string(), 15.99).unwrap();
    price_storage.store("ITEM002".to_string(), 12.50).unwrap();

    println!("   Ingredient storage utilization: {:.1}%", ingredient_storage.utilization() * 100.0);
    println!("   Price storage count: {}", price_storage.count());

    if let Some(ingredient) = ingredient_storage.retrieve(&"tomatoes".to_string()) {
        println!("   Retrieved ingredient: {}", ingredient);
    }

    // Pattern 2: State Machines with Generic Types
    println!("\n2️⃣ Generic State Machines - Order Processing Workflow:");

    // State types
    trait OrderState {}

    #[derive(Debug)]
    struct Pending;
    #[derive(Debug)]
    struct Confirmed;
    #[derive(Debug)]
    struct InPreparation;
    #[derive(Debug)]
    struct Ready;
    #[derive(Debug)]
    struct Delivered;
    #[derive(Debug)]
    struct Cancelled;

    impl OrderState for Pending {}
    impl OrderState for Confirmed {}
    impl OrderState for InPreparation {}
    impl OrderState for Ready {}
    impl OrderState for Delivered {}
    impl OrderState for Cancelled {}

    #[derive(Debug)]
    struct Order<T, S: OrderState> {
        id: String,
        items: Vec<T>,
        total: f64,
        customer_info: String,
        _state: PhantomData<S>,
    }

    // Order creation
    impl<T> Order<T, Pending> {
        fn new(id: &str, items: Vec<T>, total: f64, customer_info: &str) -> Self {
            Order {
                id: id.to_string(),
                items,
                total,
                customer_info: customer_info.to_string(),
                _state: PhantomData,
            }
        }

        fn confirm(self) -> Order<T, Confirmed> {
            println!("   📋 Order {} confirmed", self.id);
            Order {
                id: self.id,
                items: self.items,
                total: self.total,
                customer_info: self.customer_info,
                _state: PhantomData,
            }
        }

        fn cancel(self, reason: &str) -> Order<T, Cancelled> {
            println!("   ❌ Order {} cancelled: {}", self.id, reason);
            Order {
                id: self.id,
                items: self.items,
                total: self.total,
                customer_info: self.customer_info,
                _state: PhantomData,
            }
        }
    }

    impl<T> Order<T, Confirmed> {
        fn start_preparation(self) -> Order<T, InPreparation> {
            println!("   🔥 Order {} preparation started", self.id);
            Order {
                id: self.id,
                items: self.items,
                total: self.total,
                customer_info: self.customer_info,
                _state: PhantomData,
            }
        }
    }

    impl<T> Order<T, InPreparation> {
        fn mark_ready(self) -> Order<T, Ready> {
            println!("   ✅ Order {} is ready", self.id);
            Order {
                id: self.id,
                items: self.items,
                total: self.total,
                customer_info: self.customer_info,
                _state: PhantomData,
            }
        }
    }

    impl<T> Order<T, Ready> {
        fn deliver(self) -> Order<T, Delivered> {
            println!("   🚚 Order {} delivered", self.id);
            Order {
                id: self.id,
                items: self.items,
                total: self.total,
                customer_info: self.customer_info,
                _state: PhantomData,
            }
        }
    }

    // Common methods available in all states
    impl<T, S: OrderState> Order<T, S> {
        fn get_id(&self) -> &str {
            &self.id
        }

        fn get_total(&self) -> f64 {
            self.total
        }

        fn get_customer(&self) -> &str {
            &self.customer_info
        }
    }

    // Demonstrate state machine
    let order_items = vec!["Quinoa Buddha Bowl", "Caesar Salad"];

    let pending_order = Order::new("ORD-001", order_items, 28.49, "Alice Johnson");
    println!("   Created order: {} for {}", pending_order.get_id(), pending_order.get_customer());

    let confirmed_order = pending_order.confirm();
    let preparation_order = confirmed_order.start_preparation();
    let ready_order = preparation_order.mark_ready();
    let delivered_order = ready_order.deliver();

    println!("   Final order total: ${:.2}", delivered_order.get_total());

    // Pattern 3: Generic Enums with Complex Variants
    println!("\n3️⃣ Complex Generic Enums - Multi-Stage Processing Results:");

    #[derive(Debug)]
    enum ProcessingStage<T, E> {
        NotStarted,
        InProgress {
            current_step: String,
            progress: f64,
            intermediate_data: Option<T>
        },
        Completed {
            result: T,
            duration_ms: u64,
            metadata: HashMap<String, String>
        },
        Failed {
            error: E,
            failed_at_step: String,
            recovery_suggestions: Vec<String>
        },
        Cancelled {
            reason: String,
            partial_result: Option<T>
        },
    }

    impl<T, E> ProcessingStage<T, E> {
        fn is_terminal(&self) -> bool {
            matches!(self,
                ProcessingStage::Completed { .. } |
                ProcessingStage::Failed { .. } |
                ProcessingStage::Cancelled { .. }
            )
        }

        fn get_progress(&self) -> f64 {
            match self {
                ProcessingStage::NotStarted => 0.0,
                ProcessingStage::InProgress { progress, .. } => *progress,
                ProcessingStage::Completed { .. } => 100.0,
                ProcessingStage::Failed { .. } => 0.0,
                ProcessingStage::Cancelled { .. } => 0.0,
            }
        }
    }

    // Simulate recipe processing stages
    fn simulate_recipe_processing() -> Vec<ProcessingStage<String, String>> {
        vec![
            ProcessingStage::NotStarted,

            ProcessingStage::InProgress {
                current_step: "Preparing ingredients".to_string(),
                progress: 25.0,
                intermediate_data: Some("Ingredients washed".to_string()),
            },

            ProcessingStage::InProgress {
                current_step: "Cooking".to_string(),
                progress: 75.0,
                intermediate_data: Some("Cooking in progress".to_string()),
            },

            ProcessingStage::Completed {
                result: "Quinoa Buddha Bowl ready to serve".to_string(),
                duration_ms: 1200,
                metadata: {
                    let mut meta = HashMap::new();
                    meta.insert("chef".to_string(), "Alice".to_string());
                    meta.insert("station".to_string(), "Hot Kitchen".to_string());
                    meta.insert("quality_score".to_string(), "95".to_string());
                    meta
                },
            },
        ]
    }

    let processing_stages = simulate_recipe_processing();

    println!("   Recipe processing simulation:");
    for (i, stage) in processing_stages.iter().enumerate() {
        println!("     Stage {}: {:?}", i, stage);
        println!("       Progress: {:.1}%", stage.get_progress());
        println!("       Terminal: {}", stage.is_terminal());
        println!();
    }

    // Pattern 4: Generic Builder Pattern for Complex Types
    println!("\n4️⃣ Generic Builder Pattern - Complex Menu Construction:");

    #[derive(Debug, Clone)]
    struct MenuItem<T> {
        name: String,
        price: f64,
        ingredients: Vec<String>,
        dietary_info: Vec<String>,
        custom_data: T,
    }

    struct MenuItemBuilder<T> {
        name: Option<String>,
        price: Option<f64>,
        ingredients: Vec<String>,
        dietary_info: Vec<String>,
        custom_data: Option<T>,
    }

    impl<T> MenuItemBuilder<T> {
        fn new() -> Self {
            MenuItemBuilder {
                name: None,
                price: None,
                ingredients: Vec::new(),
                dietary_info: Vec::new(),
                custom_data: None,
            }
        }

        fn name(mut self, name: &str) -> Self {
            self.name = Some(name.to_string());
            self
        }

        fn price(mut self, price: f64) -> Self {
            self.price = Some(price);
            self
        }

        fn ingredient(mut self, ingredient: &str) -> Self {
            self.ingredients.push(ingredient.to_string());
            self
        }

        fn ingredients(mut self, ingredients: Vec<&str>) -> Self {
            self.ingredients.extend(ingredients.into_iter().map(|s| s.to_string()));
            self
        }

        fn dietary_info(mut self, info: &str) -> Self {
            self.dietary_info.push(info.to_string());
            self
        }

        fn custom_data(mut self, data: T) -> Self {
            self.custom_data = Some(data);
            self
        }

        fn build(self) -> Result<MenuItem<T>, String> {
            let name = self.name.ok_or("Name is required")?;
            let price = self.price.ok_or("Price is required")?;
            let custom_data = self.custom_data.ok_or("Custom data is required")?;

            if self.ingredients.is_empty() {
                return Err("At least one ingredient is required".to_string());
            }

            Ok(MenuItem {
                name,
                price,
                ingredients: self.ingredients,
                dietary_info: self.dietary_info,
                custom_data,
            })
        }
    }

    // Use the builder with different custom data types
    #[derive(Debug, Clone)]
    struct NutritionalInfo {
        calories: u32,
        protein_g: f64,
        carbs_g: f64,
    }

    #[derive(Debug, Clone)]
    struct PreparationInfo {
        prep_time_minutes: u32,
        cooking_method: String,
        difficulty: String,
    }

    // Build menu item with nutritional info
    let quinoa_bowl = MenuItemBuilder::new()
        .name("Quinoa Buddha Bowl")
        .price(15.99)
        .ingredients(vec!["quinoa", "roasted vegetables", "tahini dressing"])
        .dietary_info("Vegan")
        .dietary_info("Gluten-Free")
        .custom_data(NutritionalInfo {
            calories: 520,
            protein_g: 18.5,
            carbs_g: 68.2,
        })
        .build();

    // Build menu item with preparation info
    let pasta_dish = MenuItemBuilder::new()
        .name("Pasta Marinara")
        .price(12.50)
        .ingredient("pasta")
        .ingredient("marinara sauce")
        .ingredient("fresh basil")
        .dietary_info("Vegetarian")
        .custom_data(PreparationInfo {
            prep_time_minutes: 15,
            cooking_method: "Boil and simmer".to_string(),
            difficulty: "Easy".to_string(),
        })
        .build();

    match quinoa_bowl {
        Ok(item) => {
            println!("   Built menu item: {}", item.name);
            println!("     Price: ${:.2}", item.price);
            println!("     Ingredients: {:?}", item.ingredients);
            println!("     Nutritional info: {:?}", item.custom_data);
        }
        Err(error) => println!("   Failed to build quinoa bowl: {}", error),
    }

    match pasta_dish {
        Ok(item) => {
            println!("   Built menu item: {}", item.name);
            println!("     Preparation info: {:?}", item.custom_data);
        }
        Err(error) => println!("   Failed to build pasta dish: {}", error),
    }

    println!("\n🎯 Advanced Pattern Benefits:");
    println!("   🏗️ Associated types provide flexible, constrained interfaces");
    println!("   🔄 State machines ensure correct workflow progression");
    println!("   📊 Complex enums model sophisticated business logic");
    println!("   🛠️ Builder patterns create type-safe construction APIs");
    println!("   ⚡ All patterns maintain zero-cost abstraction guarantees");
}

fn main() {
    demonstrate_advanced_generic_patterns();
}
```


## Real-World Applications and Best Practices

### Professional Restaurant Management System Implementation

**Practical applications demonstrating how generic structs and enums solve real programming challenges:**

```rust
fn demonstrate_real_world_applications() {
    println!("🏢 Real-World Applications - Complete Restaurant Management System");
    println!("{:=<75}", "");

    use std::collections::HashMap;
    use std::fmt::{Debug, Display};
    use std::hash::Hash;

    // Real-world applications show generic structs and enums in complete systems
    println!("💼 Professional Restaurant System Architecture:");

    // Application 1: Generic Repository Pattern for Data Management
    println!("\n1️⃣ Generic Repository Pattern - Universal Data Management:");

    trait Entity {
        type Id: Clone + Hash + Eq + Display;
        fn id(&self) -> &Self::Id;
    }

    trait Repository<T: Entity> {
        type Error: Debug;

        fn create(&mut self, entity: T) -> Result<(), Self::Error>;
        fn find_by_id(&self, id: &T::Id) -> Option<&T>;
        fn update(&mut self, entity: T) -> Result<(), Self::Error>;
        fn delete(&mut self, id: &T::Id) -> Result<T, Self::Error>;
        fn find_all(&self) -> Vec<&T>;
        fn count(&self) -> usize;
    }

    #[derive(Debug)]
    enum RepositoryError {
        NotFound(String),
        DuplicateId(String),
        ValidationError(String),
    }

    #[derive(Debug)]
    struct InMemoryRepository<T: Entity> {
        data: HashMap<T::Id, T>,
        name: String,
    }

    impl<T: Entity> InMemoryRepository<T> {
        fn new(name: &str) -> Self {
            InMemoryRepository {
                data: HashMap::new(),
                name: name.to_string(),
            }
        }
    }

    impl<T: Entity + Clone> Repository<T> for InMemoryRepository<T> {
        type Error = RepositoryError;

        fn create(&mut self, entity: T) -> Result<(), Self::Error> {
            let id = entity.id().clone();
            if self.data.contains_key(&id) {
                return Err(RepositoryError::DuplicateId(
                    format!("Entity with ID {} already exists in {}", id, self.name)
                ));
            }
            self.data.insert(id, entity);
            Ok(())
        }

        fn find_by_id(&self, id: &T::Id) -> Option<&T> {
            self.data.get(id)
        }

        fn update(&mut self, entity: T) -> Result<(), Self::Error> {
            let id = entity.id().clone();
            if !self.data.contains_key(&id) {
                return Err(RepositoryError::NotFound(
                    format!("Entity with ID {} not found in {}", id, self.name)
                ));
            }
            self.data.insert(id, entity);
            Ok(())
        }

        fn delete(&mut self, id: &T::Id) -> Result<T, Self::Error> {
            self.data.remove(id).ok_or_else(|| RepositoryError::NotFound(
                format!("Entity with ID {} not found in {}", id, self.name)
            ))
        }

        fn find_all(&self) -> Vec<&T> {
            self.data.values().collect()
        }

        fn count(&self) -> usize {
            self.data.len()
        }
    }

    // Define restaurant entities
    #[derive(Debug, Clone)]
    struct MenuItem {
        id: String,
        name: String,
        price: f64,
        category: String,
        available: bool,
    }

    impl Entity for MenuItem {
        type Id = String;
        fn id(&self) -> &Self::Id {
            &self.id
        }
    }

    #[derive(Debug, Clone)]
    struct Customer {
        id: u32,
        name: String,
        email: String,
        loyalty_points: u32,
    }

    impl Entity for Customer {
        type Id = u32;
        fn id(&self) -> &Self::Id {
            &self.id
        }
    }

    #[derive(Debug, Clone)]
    struct Order {
        id: String,
        customer_id: u32,
        items: Vec<String>,
        total: f64,
        status: String,
    }

    impl Entity for Order {
        type Id = String;
        fn id(&self) -> &Self::Id {
            &self.id
        }
    }

    // Test the repository system
    let mut menu_repo = InMemoryRepository::new("Menu Items");
    let mut customer_repo = InMemoryRepository::new("Customers");
    let mut order_repo = InMemoryRepository::new("Orders");

    // Add menu items
    let quinoa_bowl = MenuItem {
        id: "ITEM001".to_string(),
        name: "Quinoa Buddha Bowl".to_string(),
        price: 15.99,
        category: "Main Course".to_string(),
        available: true,
    };

    let caesar_salad = MenuItem {
        id: "ITEM002".to_string(),
        name: "Caesar Salad".to_string(),
        price: 12.50,
        category: "Salads".to_string(),
        available: true,
    };

    menu_repo.create(quinoa_bowl).unwrap();
    menu_repo.create(caesar_salad).unwrap();

    // Add customers
    let customer1 = Customer {
        id: 1,
        name: "Alice Johnson".to_string(),
        email: "alice@email.com".to_string(),
        loyalty_points: 150,
    };

    customer_repo.create(customer1).unwrap();

    // Add order
    let order1 = Order {
        id: "ORD001".to_string(),
        customer_id: 1,
        items: vec!["ITEM001".to_string()],
        total: 15.99,
        status: "Pending".to_string(),
    };

    order_repo.create(order1).unwrap();

    println!("   Repository Status:");
    println!("     Menu items: {}", menu_repo.count());
    println!("     Customers: {}", customer_repo.count());
    println!("     Orders: {}", order_repo.count());

    // Test queries
    if let Some(item) = menu_repo.find_by_id(&"ITEM001".to_string()) {
        println!("   Found menu item: {} - ${:.2}", item.name, item.price);
    }

    // Application 2: Generic Event System with Type Safety
    println!("\n2️⃣ Generic Event System - Type-Safe Restaurant Events:");

    trait Event {
        type Data: Debug;
        fn event_type(&self) -> &str;
        fn timestamp(&self) -> u64;
        fn data(&self) -> &Self::Data;
    }

    #[derive(Debug)]
    struct OrderEvent {
        timestamp: u64,
        order_data: OrderEventData,
    }

    #[derive(Debug)]
    struct OrderEventData {
        order_id: String,
        customer_id: u32,
        total: f64,
    }

    impl Event for OrderEvent {
        type Data = OrderEventData;

        fn event_type(&self) -> &str {
            "order"
        }

        fn timestamp(&self) -> u64 {
            self.timestamp
        }

        fn data(&self) -> &Self::Data {
            &self.order_data
        }
    }

    #[derive(Debug)]
    struct InventoryEvent {
        timestamp: u64,
        inventory_data: InventoryEventData,
    }

    #[derive(Debug)]
    struct InventoryEventData {
        item_id: String,
        previous_quantity: u32,
        new_quantity: u32,
        change_reason: String,
    }

    impl Event for InventoryEvent {
        type Data = InventoryEventData;

        fn event_type(&self) -> &str {
            "inventory"
        }

        fn timestamp(&self) -> u64 {
            self.timestamp
        }

        fn data(&self) -> &Self::Data {
            &self.inventory_data
        }
    }

    struct EventProcessor<T: Event> {
        events: Vec<T>,
        processor_name: String,
    }

    impl<T: Event> EventProcessor<T> {
        fn new(name: &str) -> Self {
            EventProcessor {
                events: Vec::new(),
                processor_name: name.to_string(),
            }
        }

        fn add_event(&mut self, event: T) {
            self.events.push(event);
        }

        fn process_events_since<F>(&self, since: u64, processor: F)
        where
            F: Fn(&T),
        {
            let recent_events: Vec<&T> = self.events
                .iter()
                .filter(|event| event.timestamp() >= since)
                .collect();

            println!("   Processing {} recent {} events in {}:",
                     recent_events.len(),
                     if recent_events.is_empty() { "unknown" } else { recent_events[^0].event_type() },
                     self.processor_name);

            for event in recent_events {
                processor(event);
            }
        }

        fn get_event_summary(&self) -> String {
            format!("{}: {} events processed", self.processor_name, self.events.len())
        }
    }

    // Test event system
    let mut order_processor = EventProcessor::new("Order Handler");
    let mut inventory_processor = EventProcessor::new("Inventory Manager");

    // Add events
    order_processor.add_event(OrderEvent {
        timestamp: 1000,
        order_data: OrderEventData {
            order_id: "ORD001".to_string(),
            customer_id: 1,
            total: 15.99,
        },
    });

    inventory_processor.add_event(InventoryEvent {
        timestamp: 1050,
        inventory_data: InventoryEventData {
            item_id: "ITEM001".to_string(),
            previous_quantity: 50,
            new_quantity: 49,
            change_reason: "Order fulfilled".to_string(),
        },
    });

    // Process recent events
    order_processor.process_events_since(900, |event| {
        println!("     📦 Order Event: ID={}, Customer={}, Total=${:.2}",
                 event.data().order_id, event.data().customer_id, event.data().total);
    });

    inventory_processor.process_events_since(1000, |event| {
        println!("     📊 Inventory Event: Item={}, {} → {} ({})",
                 event.data().item_id,
                 event.data().previous_quantity,
                 event.data().new_quantity,
                 event.data().change_reason);
    });

    println!("   {}", order_processor.get_event_summary());
    println!("   {}", inventory_processor.get_event_summary());

    // Application 3: Generic Result Handling System
    println!("\n3️⃣ Generic Result Handling - Universal Error Management:");

    #[derive(Debug)]
    enum BusinessError {
        ValidationError { field: String, message: String },
        NotFound { resource: String, id: String },
        BusinessRule { rule: String, context: String },
        ExternalService { service: String, error: String },
    }

    impl Display for BusinessError {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            match self {
                BusinessError::ValidationError { field, message } =>
                    write!(f, "Validation error in {}: {}", field, message),
                BusinessError::NotFound { resource, id } =>
                    write!(f, "{} with ID {} not found", resource, id),
                BusinessError::BusinessRule { rule, context } =>
                    write!(f, "Business rule '{}' violated: {}", rule, context),
                BusinessError::ExternalService { service, error } =>
                    write!(f, "External service {} error: {}", service, error),
            }
        }
    }

    type BusinessResult<T> = Result<T, BusinessError>;

    struct OrderService;

    impl OrderService {
        fn validate_order(order: &Order) -> BusinessResult<()> {
            if order.items.is_empty() {
                return Err(BusinessError::ValidationError {
                    field: "items".to_string(),
                    message: "Order must contain at least one item".to_string(),
                });
            }

            if order.total <= 0.0 {
                return Err(BusinessError::ValidationError {
                    field: "total".to_string(),
                    message: "Order total must be positive".to_string(),
                });
            }

            Ok(())
        }

        fn process_payment(order: &Order) -> BusinessResult<String> {
            // Simulate payment processing
            if order.total > 100.0 {
                Err(BusinessError::BusinessRule {
                    rule: "Maximum order amount".to_string(),
                    context: format!("Order total ${:.2} exceeds limit of $100.00", order.total),
                })
            } else {
                Ok(format!("Payment processed: ${:.2}", order.total))
            }
        }

        fn create_order(customer_id: u32, items: Vec<String>, total: f64) -> BusinessResult<Order> {
            let order = Order {
                id: format!("ORD{:03}", fastrand::u32(100..999)),
                customer_id,
                items,
                total,
                status: "Created".to_string(),
            };

            Self::validate_order(&order)?;
            let _payment_result = Self::process_payment(&order)?;

            Ok(order)
        }
    }

    // Test the business result system
    let test_cases = [
        (1, vec!["ITEM001".to_string()], 15.99),
        (2, vec![], 20.00),  // Invalid: no items
        (3, vec!["ITEM002".to_string()], -5.0),  // Invalid: negative total
        (4, vec!["ITEM003".to_string(), "ITEM004".to_string()], 150.0),  // Invalid: too expensive
    ];

    for (customer_id, items, total) in test_cases {
        match OrderService::create_order(customer_id, items.clone(), total) {
            Ok(order) => {
                println!("   ✅ Order created successfully: {}", order.id);
                println!("      Customer: {}, Items: {:?}, Total: ${:.2}",
                         order.customer_id, order.items, order.total);
            }
            Err(error) => {
                println!("   ❌ Order creation failed: {}", error);
                println!("      Attempted: Customer={}, Items={:?}, Total=${:.2}",
                         customer_id, items, total);
            }
        }
    }

    println!("\n🎯 Real-World Application Benefits:");
    println!("   📊 Generic repositories provide consistent data access patterns");
    println!("   🔔 Type-safe event systems ensure correct data handling");
    println!("   ⚠️ Structured error handling improves debugging and user experience");
    println!("   🏗️ Generic patterns enable code reuse across different domains");
    println!("   🛡️ Compile-time type checking prevents runtime errors");

    println!("\n💡 Professional Guidelines:");
    println!("   🎯 Start with concrete implementations, then generalize");
    println!("   📝 Use meaningful names for generic type parameters");
    println!("   🔍 Apply constraints judiciously - only what you actually need");
    println!("   ⚖️ Balance flexibility with complexity");
    println!("   📊 Profile performance impact of generic abstractions");

    println!("\n🚀 Scaling Guidelines:");
    println!("   🏢 Use generic patterns for cross-cutting concerns");
    println!("   📦 Create domain-specific generic types for business logic");
    println!("   🔄 Design for extensibility without over-engineering");
    println!("   🛠️ Build generic utilities that solve multiple problems");
    println!("   📈 Monitor compile times and binary size with heavy generic usage");
}

fn main() {
    demonstrate_real_world_applications();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant System Architecture**

Remember our comprehensive professional restaurant system analogy:

- 📦 **Generic structs** = **Universal storage containers** - Flexible data structures for any ingredient type
- 🔄 **Generic enums** = **Standardized process workflows** - Consistent handling of different outcomes
- 🛡️ **Type parameters `<T>`** = **Equipment specifications** - Placeholder for what types the system handles
- ⚙️ **Constraints** = **Quality standards** - Requirements that types must meet to work with the system
- 🏗️ **Advanced patterns** = **Professional architecture** - Sophisticated systems for complex operations


### **Essential Generic Structs and Enums Patterns**

**Generic Struct Syntax:**

```rust
// Basic generic struct
struct Container<T> {
    item: T,
    capacity: usize,
}

// Multiple generic parameters
struct Pair<T, U> {
    first: T,
    second: U,
}

// With constraints
struct ProcessedData<T>
where T: Clone + Display {
    data: T,
    metadata: String,
}

// Implementation for all types
impl<T> Container<T> {
    fn new(item: T, capacity: usize) -> Self {
        Container { item, capacity }
    }
}

// Implementation for specific types
impl Container<String> {
    fn get_length(&self) -> usize {
        self.item.len()
    }
}
```

**Generic Enum Syntax:**

```rust
// Basic generic enum
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Multiple variants with generics
enum ProcessingState<T, E> {
    NotStarted,
    InProgress(T),
    Completed(T),
    Failed(E),
}

// Complex generic enum
enum ApiResponse<T, E> {
    Success { data: T, metadata: HashMap<String, String> },
    Error { error: E, retry_after: Option<u64> },
    Partial { data: Vec<T>, errors: Vec<E> },
}
```


### **Generic Structs vs Enums Decision Matrix**

| **Use Case** | **Generic Struct** | **Generic Enum** | **Reasoning** |
| :-- | :-- | :-- | :-- |
| **Data containers** | ✅ Perfect fit | ❌ Not suitable | Structs group related data |
| **Multiple outcomes** | ❌ Not suitable | ✅ Perfect fit | Enums represent alternatives |
| **State machines** | ✅ With PhantomData | ✅ Natural fit | Both can model states |
| **Error handling** | ❌ Not ideal | ✅ Result<T,E> | Enums model success/failure |
| **Optional values** | ❌ Not suitable | ✅ Option<T> | Enums model presence/absence |
| **Complex data** | ✅ Best choice | ❌ Awkward | Structs excel at complex data |

### **Best Practices Checklist**

**✅ Design Guidelines:**

- Use meaningful generic parameter names (not just `T`)
- Apply minimal necessary constraints
- Consider both current and future use cases
- Design for composability and reusability
- Document generic behavior and constraints clearly

**✅ Implementation Guidelines:**

- Implement methods for generic types when they make sense
- Provide specific implementations for concrete types when needed
- Use associated types in traits for flexible but constrained generics
- Leverage standard library patterns (Option, Result, Vec, HashMap)

**✅ Performance Guidelines:**

- Understand monomorphization impact on binary size
- Profile compilation times with heavy generic usage
- Use generic constraints to enable compiler optimizations
- Consider the trade-off between flexibility and compile-time cost

**❌ Common Pitfalls:**

- Over-generalizing simple types unnecessarily
- Creating overly complex constraint combinations
- Ignoring the compile-time cost of generics
- Not considering ergonomics from the caller's perspective
- Mixing concerns in single generic types


### **Standard Library Patterns to Master**

**Essential Generic Types:**

- `Option<T>` - Represents optional values[^1][^2]
- `Result<T, E>` - Represents success/failure outcomes[^2][^1]
- `Vec<T>` - Dynamic arrays of any type[^1]
- `HashMap<K, V>` - Key-value mappings[^1]
- `Box<T>` - Heap-allocated values[^1]


### **Advanced Professional Patterns**

**Repository Pattern:**

```rust
trait Repository<T: Entity> {
    type Error;
    fn create(&mut self, entity: T) -> Result<(), Self::Error>;
    fn find_by_id(&self, id: &T::Id) -> Option<&T>;
    // ... other CRUD operations
}
```

**Builder Pattern:**

```rust
struct Builder<T> {
    data: Option<T>,
    // ... other fields
}

impl<T> Builder<T> {
    fn new() -> Self { /* ... */ }
    fn build(self) -> Result<T, Error> { /* ... */ }
}
```

**State Machine Pattern:**

```rust
struct StateMachine<T, S: State> {
    data: T,
    _state: PhantomData<S>,
}

impl<T> StateMachine<T, InitialState> {
    fn transition(self) -> StateMachine<T, NextState> {
        // Safe state transitions
    }
}
```


### **The Professional Advantage**

**Mastering generic structs and enums in Rust is like having a complete professional restaurant system architecture** that adapts to any cuisine, scale, or operational requirement:

- 📦 **Flexible infrastructure** - Data structures that work with any type
- 🔄 **Consistent patterns** - Standardized approaches to common problems
- 🛡️ **Type safety** - Compile-time guarantees prevent runtime errors
- ⚡ **Performance** - Zero-cost abstractions with monomorphization
- 🎯 **Maintainability** - Single implementations serve multiple use cases

**Understanding generic structs and enums transforms you from a programmer who duplicates code for different types to an architect** who builds flexible, reusable, and type-safe systems. Just as a master restaurant architect can design systems that work efficiently whether serving 50 or 500 customers, with any cuisine type, while maintaining safety and quality standards, a skilled Rust programmer leverages generic structs and enums to create powerful abstractions that scale seamlessly across different data types and use cases.

This comprehensive understanding of generic structs and enums - from basic syntax through advanced patterns and real-world applications - makes your Rust programs more flexible, your architecture more scalable, and your development process more efficient, whether you're building simple data containers or complex enterprise systems that need to handle diverse data types with compile-time safety guarantees!

1. https://www.programiz.com/rust/generics
2. https://doc.rust-lang.org/book/ch10-01-syntax.html
3. https://doc.rust-lang.org/rust-by-example/generics.html
4. https://www.risein.com/courses/rust-programming/implementation-of-generics-using-structs-and-enums-in-rust
5. https://www.linkedin.com/pulse/mastering-generics-rust-hands-onguide-luis-soares-m-sc-
6. https://academy.patika.dev/courses/rust-programming/implementation-of-generics-using-structs-and-enums-in-rust
7. https://earthly.dev/blog/rust-generics/
8. https://serokell.io/blog/rust-generics
9. https://www.linkedin.com/pulse/rust-random-topic-009-exploring-enums-generics-abhishek-kumar-zjn1f
10. https://fitech101.aalto.fi/en/courses/modern-and-emerging-programming-languages/part-5/8-structs-traits-enums-and-generics
11. https://www.youtube.com/watch?v=sLjOV8kYxfE
12. https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html
13. https://www.reddit.com/r/rust/comments/138e76r/generic_struct_in_enum/
14. https://stackoverflow.com/questions/54504026/how-do-i-provide-an-implementation-of-a-generic-struct-in-rust
15. https://users.rust-lang.org/t/can-enum-case-parameter-be-generic/65228
16. https://users.rust-lang.org/t/how-do-i-create-an-instance-of-a-generic-type-on-a-struct/86359
17. https://doc.rust-lang.org/rust-by-example/custom_types/enum.html
18. https://stackoverflow.com/questions/72438594/how-can-i-use-enum-variants-as-generic-type
19. https://users.rust-lang.org/t/code-architecture-for-struct-of-structs-enum-vs-generic-traits-vs/71435
20. https://users.rust-lang.org/t/enum-vs-generic-with-trait-pros-cons/50069
21. https://stackoverflow.com/questions/62947622/rusts-enums-vs-generics
22. https://users.rust-lang.org/t/noob-question-again-enum-and-struct/100707
23. https://users.rust-lang.org/t/binding-struct-generic-type-parameter-by-enum/84639
24. https://stackoverflow.com/questions/26437043/why-does-rust-have-struct-and-enum
25. https://users.rust-lang.org/t/generic-function-over-enum-with-different-types/121196
26. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch10-01-syntax.html
27. https://rust-book.cs.brown.edu/ch10-01-syntax.html
28. https://www.youtube.com/watch?v=90jSoJkBasA
29. https://www.andy-pearce.com/blog/posts/2023/Apr/uncovering-rust-traits-and-generics/
30. https://stevedonovan.github.io/rust-gentle-intro/2-structs-enums-lifetimes.html
31. https://leapcell.io/blog/understanding-generics-in-rust
32. https://users.rust-lang.org/t/implement-generic-struct-with-different-method-variants/112678
33. https://dev.to/ajtech0001/mastering-generic-types-and-traits-in-rust-a-complete-guide-4e2n
34. https://leapcell.io/blog/generic-associated-types-in-rust
35. https://blog.logrocket.com/understanding-rust-generics/
36. https://users.rust-lang.org/t/optional-generic-type-to-build-new-struct/103453
37. https://john-cd.com/rust_howto/language/enums.html
38. https://moldstud.com/articles/p-deep-dive-into-rust-traits-and-generics-enhancing-type-flexibility
39. https://stackoverflow.com/questions/30399953/what-is-the-proper-way-to-create-a-new-generic-struct
40. https://www.reddit.com/r/rust/comments/n7uhwk/how_can_i_create_a_generic_type_for_structs_that/
41. https://users.rust-lang.org/t/initializing-a-generic-types-field-in-a-struct/104030
42. https://www.youtube.com/watch?v=XKbOVFt3UNY
43. https://users.rust-lang.org/t/use-enum-variant-for-generic/119834
