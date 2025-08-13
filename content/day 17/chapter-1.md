+++
title = "HashMap Creation and Manipulation"
description = "Learn how to create, insert, update, and retrieve values from a HashMap in Rust, including handling missing keys and using entry API."
date = 2025-08-13
weight = 1
keywords = ["Rust", "HashMap", "collections", "key-value", "entry API", "insert", "update", "retrieve"]
+++

# HashMap Creation and Manipulation in Rust: Comprehensive Documentation for Beginners

Understanding HashMap creation and manipulation in Rust is like learning to **manage a sophisticated vegetarian restaurant's reservation and customer management system** - you need a way to efficiently store, retrieve, and update customer information using unique identifiers (like table numbers or customer IDs) rather than searching through long lists. Just as a professional restaurant uses a reservation system where each customer name maps to their table assignment, dietary preferences, and order history, Rust's HashMap provides lightning-fast key-value storage that lets you organize and access related data instantly.

## The Restaurant Customer Management System Analogy 🏪

### Imagine You're Managing a High-End Vegetarian Restaurant's Customer Database

**Linear Search Approach (Inefficient):**

```rust
// ❌ Searching through a vector every time - like checking every table manually
let customers = vec![
    ("Alice Johnson", "Table 5, Vegan, Quinoa Bowl"),
    ("Bob Smith", "Table 3, Gluten-Free, Lentil Soup"),
    ("Carol Davis", "Table 8, Vegetarian, Pasta Primavera"),
];

// To find Alice's info, you have to check every entry!
for (name, info) in &customers {
    if name == &"Alice Johnson" {
        println!("Found Alice: {}", info);
        break;
    }
}
// Slow! O(n) time complexity
```

**HashMap Approach (Efficient Restaurant System):**

```rust
// ✅ Instant lookup system - like having a smart reservation computer
use std::collections::HashMap;

let mut customer_system = HashMap::new();
customer_system.insert("Alice Johnson", "Table 5, Vegan, Quinoa Bowl");
customer_system.insert("Bob Smith", "Table 3, Gluten-Free, Lentil Soup");
customer_system.insert("Carol Davis", "Table 8, Vegetarian, Pasta Primavera");

// Instant access to any customer's information!
if let Some(info) = customer_system.get("Alice Johnson") {
    println!("Found Alice instantly: {}", info);
}
// Fast! O(1) average time complexity
```

**Why HashMap is Superior:**

- ⚡ **Instant access** - Find any customer immediately by name
- 🎯 **Unique keys** - Each customer name maps to exactly one record
- 🔄 **Easy updates** - Change customer information without searching
- 📈 **Scalable** - Performance stays consistent even with thousands of customers
- 🔗 **Flexible relationships** - Link any type of key to any type of value


## Understanding HashMap Fundamentals

### What is a HashMap?

A **HashMap** is Rust's primary key-value data structure that provides:

```rust
fn demonstrate_hashmap_basics() {
    println!("🗝️ HashMap Fundamentals - Restaurant Customer System");
    println!("{:=<55}", "");

    use std::collections::HashMap;

    // Think of HashMap as a sophisticated filing system
    // Key = Customer ID/Name (unique identifier)
    // Value = Customer information (what we want to store)

    let mut restaurant_customers: HashMap<String, String> = HashMap::new();

    println!("📊 HashMap Characteristics:");
    println!("   • Key-Value pairs: Each key maps to exactly one value");
    println!("   • Unique keys: No duplicate keys allowed");
    println!("   • Fast access: Average O(1) lookup time");
    println!("   • Hash-based: Uses hashing for efficient storage");
    println!("   • Mutable: Can add, remove, and update entries");

    // Add some customer records
    restaurant_customers.insert(
        "CUST001".to_string(),
        "Alice Johnson - Vegan preferences, Regular customer".to_string()
    );
    restaurant_customers.insert(
        "CUST002".to_string(),
        "Bob Smith - Gluten-free diet, First visit".to_string()
    );
    restaurant_customers.insert(
        "CUST003".to_string(),
        "Carol Davis - Vegetarian, Anniversary dinner".to_string()
    );

    println!("\n🗃️ Customer Database:");
    println!("   Database size: {} customers", restaurant_customers.len());
    println!("   Memory efficient: Only stores what you need");
    println!("   Type safe: Keys and values must match declared types");

    // Demonstrate instant access
    println!("\n⚡ Instant Customer Lookup:");
    if let Some(customer_info) = restaurant_customers.get("CUST001") {
        println!("   ✅ Found CUST001: {}", customer_info);
    } else {
        println!("   ❌ Customer not found");
    }

    println!("\n🏗️ HashMap Memory Layout (Conceptual):");
    println!("   ┌───────────────────────────────────────────┐");
    println!("   │ HashMap<String, String>                   │");
    println!("   ├───────────────────────────────────────────┤");
    println!("   │ Hash Table (Internal)                     │");
    println!("   │ ┌─────────┬─────────────────────────────┐ │");
    println!("   │ │ Hash    │ Key → Value                 │ │");
    println!("   │ ├─────────┼─────────────────────────────┤ │");
    println!("   │ │ 1234... │ CUST001 → Alice Johnson...  │ │");
    println!("   │ │ 5678... │ CUST002 → Bob Smith...      │ │");
    println!("   │ │ 9012... │ CUST003 → Carol Davis...    │ │");
    println!("   │ └─────────┴─────────────────────────────┘ │");
    println!("   └───────────────────────────────────────────┘");
}

fn main() {
    demonstrate_hashmap_basics();
}
```


### How HashMap Hashing Works

**The magic behind instant access:**

```rust
fn demonstrate_hashing_concept() {
    println!("🔮 How HashMap Hashing Works - The Magic Behind Speed");
    println!("{:=<55}", "");

    use std::collections::HashMap;
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{Hash, Hasher};

    // Function to show hash values (for educational purposes)
    fn calculate_hash<T: Hash>(t: &T) -> u64 {
        let mut s = DefaultHasher::new();
        t.hash(&mut s);
        s.finish()
    }

    println!("🏪 Restaurant Table Assignment System:");
    println!("   When a customer arrives, we need to instantly find their reservation");
    println!("   HashMap uses 'hashing' to convert names into memory addresses");

    let customer_names = ["Alice Johnson", "Bob Smith", "Carol Davis", "Diana Wilson"];

    println!("\n🔢 Hash Function Magic:");
    for name in &customer_names {
        let hash_value = calculate_hash(name);
        let table_slot = hash_value % 8; // Simulate hash table with 8 slots
        println!("   '{}' → Hash: {} → Table Slot: {}",
                 name, hash_value, table_slot);
    }

    println!("\n⚡ Why This Is Fast:");
    println!("   1. 🔄 Convert key to hash number (fast math operation)");
    println!("   2. 📍 Use hash to find exact memory location");
    println!("   3. 🎯 Go directly to that location (no searching!)");
    println!("   4. ✅ Retrieve value instantly");

    // Demonstrate with actual HashMap
    let mut table_assignments = HashMap::new();
    table_assignments.insert("Alice Johnson", 5);
    table_assignments.insert("Bob Smith", 3);
    table_assignments.insert("Carol Davis", 8);
    table_assignments.insert("Diana Wilson", 12);

    println!("\n🍽️ Actual Table Assignment Results:");
    for name in &customer_names {
        if let Some(table) = table_assignments.get(name) {
            println!("   {} → Table {}", name, table);
        }
    }

    println!("\n📊 Performance Comparison:");
    println!("   📋 Vector Search: O(n) - Gets slower with more customers");
    println!("   🗝️ HashMap Lookup: O(1) - Always fast, regardless of size");
    println!("   💡 With 1000 customers: Vector = 1000x slower than HashMap!");
}

fn main() {
    demonstrate_hashing_concept();
}
```


## Creating HashMaps - Setting Up Your Customer System

### 1. Basic HashMap Creation

**Different ways to initialize your restaurant management system:**

```rust
fn demonstrate_hashmap_creation() {
    println!("🏗️ HashMap Creation Methods - Setting Up Restaurant Systems");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Method 1: Empty HashMap with new()
    println!("🏪 Method 1: Starting with Empty System");
    let mut customer_preferences: HashMap<String, String> = HashMap::new();
    println!("   ✅ Created empty customer preferences system");

    // Method 2: With type inference
    println!("\n🎯 Method 2: Let Rust Infer Types");
    let mut table_reservations = HashMap::new();
    table_reservations.insert("Alice", 5); // Rust infers HashMap<&str, i32>
    println!("   ✅ Created table reservations (Rust figured out types)");

    // Method 3: With initial capacity
    println!("\n⚡ Method 3: Pre-sized for Performance");
    let mut large_customer_db = HashMap::with_capacity(1000);
    large_customer_db.insert("CUST001".to_string(), "VIP Customer".to_string());
    println!("   ✅ Created high-capacity system for {} customers",
             large_customer_db.capacity());

    // Method 4: From iterator (collected data)
    println!("\n📊 Method 4: From Existing Data");
    let customer_data = vec![
        ("Alice Johnson", "Vegan"),
        ("Bob Smith", "Gluten-Free"),
        ("Carol Davis", "Vegetarian"),
    ];

    let dietary_preferences: HashMap<&str, &str> = customer_data
        .into_iter()
        .collect();

    println!("   ✅ Created dietary preferences from customer survey data");
    println!("   📋 Loaded {} dietary preferences", dietary_preferences.len());

    // Method 5: Using the from() method
    println!("\n🔄 Method 5: From Array Data");
    let menu_prices = HashMap::from([
        ("Quinoa Bowl", 15.99),
        ("Lentil Soup", 8.99),
        ("Veggie Burger", 12.49),
        ("Pasta Primavera", 14.99),
    ]);

    println!("   ✅ Created menu pricing system");
    println!("   💰 Loaded {} menu items with prices", menu_prices.len());

    // Method 6: Structured data approach
    println!("\n🏢 Method 6: Structured Customer Records");

    #[derive(Debug)]
    struct CustomerRecord {
        name: String,
        dietary_preference: String,
        visit_count: u32,
        favorite_dish: Option<String>,
    }

    let mut customer_records: HashMap<String, CustomerRecord> = HashMap::new();

    customer_records.insert("CUST001".to_string(), CustomerRecord {
        name: "Alice Johnson".to_string(),
        dietary_preference: "Vegan".to_string(),
        visit_count: 15,
        favorite_dish: Some("Quinoa Buddha Bowl".to_string()),
    });

    customer_records.insert("CUST002".to_string(), CustomerRecord {
        name: "Bob Smith".to_string(),
        dietary_preference: "Gluten-Free".to_string(),
        visit_count: 3,
        favorite_dish: None,
    });

    println!("   ✅ Created comprehensive customer record system");
    println!("   📊 Tracking detailed information for {} customers",
             customer_records.len());

    // Display all systems created
    println!("\n📈 Summary of Restaurant Systems Created:");
    println!("   🎯 Customer Preferences: {} entries", customer_preferences.len());
    println!("   🪑 Table Reservations: {} entries", table_reservations.len());
    println!("   💾 Large Customer DB: {} capacity", large_customer_db.capacity());
    println!("   🥗 Dietary Preferences: {} entries", dietary_preferences.len());
    println!("   💰 Menu Prices: {} entries", menu_prices.len());
    println!("   📋 Customer Records: {} entries", customer_records.len());

    // Demonstrate accessing data from different systems
    println!("\n🔍 Quick System Tests:");

    // Test menu prices
    if let Some(price) = menu_prices.get("Quinoa Bowl") {
        println!("   💰 Quinoa Bowl price: ${:.2}", price);
    }

    // Test customer records
    if let Some(record) = customer_records.get("CUST001") {
        println!("   👤 Customer CUST001: {}", record.name);
        println!("   🥗 Dietary preference: {}", record.dietary_preference);
        println!("   📅 Visit count: {}", record.visit_count);
    }
}

fn main() {
    demonstrate_hashmap_creation();
}
```


### 2. Working with Different Key and Value Types

**Flexible systems for different restaurant needs:**

```rust
fn demonstrate_hashmap_types() {
    println!("🎨 HashMap Type Flexibility - Different Restaurant Systems");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // System 1: String keys with integer values (Table numbers)
    println!("🪑 Table Assignment System (String → Integer):");
    let mut table_assignments: HashMap<String, u32> = HashMap::new();
    table_assignments.insert("Alice Johnson".to_string(), 5);
    table_assignments.insert("Bob Smith".to_string(), 3);
    table_assignments.insert("Carol Davis".to_string(), 8);

    for (customer, table) in &table_assignments {
        println!("   👤 {} → Table {}", customer, table);
    }

    // System 2: Integer keys with string values (Menu ID to dish name)
    println!("\n🍽️ Menu ID System (Integer → String):");
    let mut menu_items: HashMap<u32, String> = HashMap::new();
    menu_items.insert(101, "Quinoa Buddha Bowl".to_string());
    menu_items.insert(102, "Mediterranean Wrap".to_string());
    menu_items.insert(103, "Lentil Curry".to_string());
    menu_items.insert(104, "Veggie Burger Deluxe".to_string());

    for (id, dish) in &menu_items {
        println!("   🆔 Menu #{} → {}", id, dish);
    }

    // System 3: Complex structures as values
    println!("\n📊 Advanced Customer Analytics (String → Struct):");

    #[derive(Debug)]
    struct CustomerAnalytics {
        total_visits: u32,
        total_spent: f64,
        favorite_items: Vec<String>,
        last_visit: String,
        loyalty_tier: String,
    }

    let mut customer_analytics: HashMap<String, CustomerAnalytics> = HashMap::new();

    customer_analytics.insert("alice@email.com".to_string(), CustomerAnalytics {
        total_visits: 23,
        total_spent: 487.65,
        favorite_items: vec![
            "Quinoa Bowl".to_string(),
            "Green Smoothie".to_string(),
        ],
        last_visit: "2024-01-15".to_string(),
        loyalty_tier: "Gold".to_string(),
    });

    customer_analytics.insert("bob@email.com".to_string(), CustomerAnalytics {
        total_visits: 8,
        total_spent: 156.32,
        favorite_items: vec!["Veggie Burger".to_string()],
        last_visit: "2024-01-10".to_string(),
        loyalty_tier: "Silver".to_string(),
    });

    for (email, analytics) in &customer_analytics {
        println!("   📧 {}: {} visits, ${:.2} spent, {} tier",
                 email, analytics.total_visits, analytics.total_spent, analytics.loyalty_tier);
    }

    // System 4: Tuple keys for complex indexing
    println!("\n📅 Reservation System (Date + Time → Details):");
    let mut reservations: HashMap<(String, String), String> = HashMap::new();

    reservations.insert(
        ("2024-01-15".to_string(), "7:30 PM".to_string()),
        "Johnson Party of 4 - Anniversary celebration".to_string()
    );
    reservations.insert(
        ("2024-01-15".to_string(), "8:00 PM".to_string()),
        "Smith Party of 2 - Business dinner".to_string()
    );
    reservations.insert(
        ("2024-01-16".to_string(), "6:30 PM".to_string()),
        "Davis Family of 6 - Birthday party".to_string()
    );

    for ((date, time), details) in &reservations {
        println!("   📅 {} at {} → {}", date, time, details);
    }

    // System 5: Enum keys for categorization
    println!("\n📋 Inventory by Category (Enum → HashMap):");

    #[derive(Debug, Hash, Eq, PartialEq)]
    enum IngredientCategory {
        Vegetables,
        Grains,
        Proteins,
        Spices,
        Dairy,
    }

    let mut inventory_by_category: HashMap<IngredientCategory, HashMap<String, u32>> = HashMap::new();

    // Vegetables
    let mut vegetables = HashMap::new();
    vegetables.insert("Tomatoes".to_string(), 50);
    vegetables.insert("Lettuce".to_string(), 30);
    vegetables.insert("Onions".to_string(), 25);
    inventory_by_category.insert(IngredientCategory::Vegetables, vegetables);

    // Grains
    let mut grains = HashMap::new();
    grains.insert("Quinoa".to_string(), 40);
    grains.insert("Brown Rice".to_string(), 60);
    grains.insert("Oats".to_string(), 35);
    inventory_by_category.insert(IngredientCategory::Grains, grains);

    for (category, items) in &inventory_by_category {
        println!("   📦 {:?}:", category);
        for (item, quantity) in items {
            println!("      • {} units of {}", quantity, item);
        }
    }

    // System 6: Performance comparison
    println!("\n⚡ Performance Characteristics by Type:");
    println!("   🔢 Integer keys: Fastest (simple hash)");
    println!("   📝 String keys: Fast (optimized string hashing)");
    println!("   🏗️ Struct keys: Good (if Hash trait implemented well)");
    println!("   📊 Complex values: No impact on lookup speed");
    println!("   💡 Rule: Key complexity affects performance, value complexity doesn't");
}

fn main() {
    demonstrate_hashmap_types();
}
```


## HashMap Manipulation - Managing Your Restaurant Data

### 1. Adding and Updating Data

**Core operations for restaurant management:**

```rust
fn demonstrate_hashmap_manipulation() {
    println!("🔄 HashMap Data Manipulation - Restaurant Operations");
    println!("{:=<55}", "");

    use std::collections::HashMap;

    let mut restaurant_orders: HashMap<String, Vec<String>> = HashMap::new();

    // Method 1: insert() - Add or update entries
    println!("📥 Adding Customer Orders:");

    restaurant_orders.insert("Table-5".to_string(), vec![
        "Quinoa Buddha Bowl".to_string(),
        "Green Smoothie".to_string(),
    ]);

    restaurant_orders.insert("Table-3".to_string(), vec![
        "Veggie Burger".to_string(),
        "Sweet Potato Fries".to_string(),
        "Iced Tea".to_string(),
    ]);

    restaurant_orders.insert("Table-8".to_string(), vec![
        "Mediterranean Wrap".to_string(),
    ]);

    println!("   ✅ Added orders for {} tables", restaurant_orders.len());

    // Method 2: Updating existing entries
    println!("\n🔄 Updating Customer Orders:");

    // Add another item to Table-5's order
    if let Some(order) = restaurant_orders.get_mut("Table-5") {
        order.push("Chocolate Avocado Mousse".to_string());
        println!("   ✅ Added dessert to Table-5's order");
    }

    // Replace entire order for Table-3
    restaurant_orders.insert("Table-3".to_string(), vec![
        "Lentil Soup".to_string(),  // Customer changed their mind
        "Artisan Bread".to_string(),
    ]);
    println!("   🔄 Table-3 changed their order completely");

    // Method 3: entry() API for conditional operations
    println!("\n🎯 Smart Order Management with entry() API:");

    // Add order only if table doesn't exist
    restaurant_orders.entry("Table-12".to_string())
        .or_insert(vec!["Pasta Primavera".to_string()]);
    println!("   ✅ Added new order for Table-12");

    // Try to add for existing table (won't overwrite)
    restaurant_orders.entry("Table-5".to_string())
        .or_insert(vec!["Default Order".to_string()]);
    println!("   ℹ️ Table-5 already has order, didn't overwrite");

    // Add item to existing order or create new order
    restaurant_orders.entry("Table-15".to_string())
        .or_insert_with(Vec::new)
        .push("Caesar Salad".to_string());
    println!("   ✅ Added item to Table-15 (created new order if needed)");

    // Method 4: Complex updates based on existing values
    println!("\n🧮 Advanced Order Processing:");

    let mut customer_points: HashMap<String, u32> = HashMap::new();
    customer_points.insert("alice@email.com".to_string(), 150);
    customer_points.insert("bob@email.com".to_string(), 75);
    customer_points.insert("carol@email.com".to_string(), 200);

    // Add bonus points for all customers
    for (customer, points) in customer_points.iter_mut() {
        *points += 25; // Holiday bonus!
        println!("   🎁 Added 25 bonus points to {}", customer);
    }

    // Double points for VIP customers (over 150 points)
    for (customer, points) in customer_points.iter_mut() {
        if *points > 150 {
            let bonus = *points / 2;
            *points += bonus;
            println!("   ⭐ VIP bonus: {} doubled points for {}", bonus, customer);
        }
    }

    // Method 5: Bulk operations
    println!("\n📦 Bulk Order Processing:");

    let new_orders = [
        ("Table-20", vec!["Hummus Platter", "Pita Bread"]),
        ("Table-21", vec!["Veggie Sushi Roll", "Miso Soup"]),
        ("Table-22", vec!["Mushroom Risotto", "Side Salad"]),
    ];

    for (table, items) in new_orders {
        let order: Vec<String> = items.iter().map(|s| s.to_string()).collect();
        restaurant_orders.insert(table.to_string(), order);
        println!("   📋 Processed bulk order for {}", table);
    }

    // Display current state
    println!("\n📊 Current Restaurant Status:");
    println!("   🍽️ Active orders: {} tables", restaurant_orders.len());
    println!("   👥 Customer points tracked: {} customers", customer_points.len());

    // Show sample orders
    println!("\n🔍 Sample Current Orders:");
    for (table, order) in restaurant_orders.iter().take(3) {
        println!("   {} → {} items: {}", table, order.len(), order.join(", "));
    }

    println!("\n💰 Sample Customer Points:");
    for (customer, points) in customer_points.iter().take(3) {
        println!("   {} → {} points", customer, points);
    }
}

fn main() {
    demonstrate_hashmap_manipulation();
}
```


### 2. Accessing and Retrieving Data

**Safe and efficient data access patterns:**

```rust
fn demonstrate_hashmap_access() {
    println!("🔍 HashMap Data Access - Safe Restaurant Queries");
    println!("{:=<50}", "");

    use std::collections::HashMap;

    // Setup restaurant data
    let mut menu_prices = HashMap::new();
    menu_prices.insert("Quinoa Bowl", 15.99);
    menu_prices.insert("Veggie Burger", 12.49);
    menu_prices.insert("Lentil Soup", 8.99);
    menu_prices.insert("Pasta Primavera", 14.99);
    menu_prices.insert("Green Smoothie", 6.99);

    let mut customer_preferences = HashMap::new();
    customer_preferences.insert("alice@email.com", vec!["Vegan", "Gluten-Free"]);
    customer_preferences.insert("bob@email.com", vec!["Vegetarian"]);
    customer_preferences.insert("carol@email.com", vec!["Vegan", "Nut-Free"]);

    // Method 1: get() - Safe access with Option
    println!("🛡️ Safe Menu Price Lookup:");

    let dishes_to_check = ["Quinoa Bowl", "Pizza", "Lentil Soup", "Steak"];

    for dish in &dishes_to_check {
        match menu_prices.get(dish) {
            Some(price) => println!("   ✅ {} → ${:.2}", dish, price),
            None => println!("   ❌ {} → Not available", dish),
        }
    }

    // Method 2: Using if let for cleaner code
    println!("\n💰 Special Price Checks:");

    if let Some(price) = menu_prices.get("Quinoa Bowl") {
        if *price > 15.0 {
            println!("   💎 Quinoa Bowl is premium priced at ${:.2}", price);
        } else {
            println!("   💰 Quinoa Bowl is reasonably priced at ${:.2}", price);
        }
    }

    // Method 3: get_key_value() - Get both key and value
    println!("\n🔍 Finding Menu Items by Price Range:");

    for (dish, price) in &menu_prices {
        if *price <= 10.0 {
            println!("   💰 Budget option: {} at ${:.2}", dish, price);
        }
    }

    // Method 4: contains_key() - Check existence
    println!("\n❓ Menu Availability Check:");

    let customer_requests = ["Quinoa Bowl", "Meat Burger", "Green Smoothie", "Coffee"];

    for request in &customer_requests {
        if menu_prices.contains_key(request) {
            println!("   ✅ We have {}", request);
        } else {
            println!("   ❌ Sorry, {} is not available", request);
        }
    }

    // Method 5: Complex data access
    println!("\n🎯 Customer Preference Analysis:");

    let customers_to_check = ["alice@email.com", "bob@email.com", "unknown@email.com"];

    for customer in &customers_to_check {
        if let Some(preferences) = customer_preferences.get(customer) {
            println!("   👤 {}: {} preferences", customer, preferences.len());
            for preference in preferences {
                println!("      • {}", preference);
            }

            // Check if customer is vegan
            if preferences.contains(&"Vegan") {
                println!("      🌱 Customer is vegan - show vegan menu");
            }
        } else {
            println!("   ❓ {} → New customer, no preferences on file", customer);
        }
    }

    // Method 6: Default values with unwrap_or
    println!("\n🔄 Handling Missing Data with Defaults:");

    let unknown_dishes = ["Mystery Dish", "Secret Recipe"];

    for dish in &unknown_dishes {
        let price = menu_prices.get(dish).unwrap_or(&0.0);
        if *price == 0.0 {
            println!("   🤷 {} → Market price (not in menu)", dish);
        } else {
            println!("   💰 {} → ${:.2}", dish, price);
        }
    }

    // Method 7: Multiple lookups efficiently
    println!("\n📋 Order Price Calculation:");

    let customer_order = ["Quinoa Bowl", "Green Smoothie", "Lentil Soup"];
    let mut total_price = 0.0;
    let mut available_items = Vec::new();
    let mut unavailable_items = Vec::new();

    for item in &customer_order {
        if let Some(price) = menu_prices.get(item) {
            total_price += price;
            available_items.push(*item);
            println!("   ✅ {} → ${:.2}", item, price);
        } else {
            unavailable_items.push(*item);
            println!("   ❌ {} → Not available", item);
        }
    }

    println!("\n📊 Order Summary:");
    println!("   ✅ Available items: {}", available_items.len());
    println!("   ❌ Unavailable items: {}", unavailable_items.len());
    println!("   💰 Total price: ${:.2}", total_price);

    if !unavailable_items.is_empty() {
        println!("   💡 Suggest alternatives for: {}", unavailable_items.join(", "));
    }

    // Method 8: Keys and values iteration
    println!("\n📋 Menu Overview:");

    println!("   🍽️ All menu items:");
    for dish in menu_prices.keys() {
        println!("      • {}", dish);
    }

    println!("   💰 All prices:");
    let mut prices: Vec<f64> = menu_prices.values().copied().collect();
    prices.sort_by(|a, b| a.partial_cmp(b).unwrap());
    println!("      Range: ${:.2} - ${:.2}", prices[^0], prices[prices.len()-1]);

    let average_price: f64 = prices.iter().sum::<f64>() / prices.len() as f64;
    println!("      Average: ${:.2}", average_price);
}

fn main() {
    demonstrate_hashmap_access();
}
```


### 3. Removing and Clearing Data

**Efficient data cleanup and removal operations:**

```rust
fn demonstrate_hashmap_removal() {
    println!("🗑️ HashMap Data Removal - Restaurant Cleanup Operations");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Setup restaurant order system
    let mut active_orders: HashMap<String, Vec<String>> = HashMap::new();
    active_orders.insert("Table-1".to_string(), vec!["Quinoa Bowl".to_string(), "Green Tea".to_string()]);
    active_orders.insert("Table-2".to_string(), vec!["Veggie Burger".to_string(), "Fries".to_string()]);
    active_orders.insert("Table-3".to_string(), vec!["Lentil Soup".to_string()]);
    active_orders.insert("Table-5".to_string(), vec!["Pasta".to_string(), "Salad".to_string(), "Wine".to_string()]);
    active_orders.insert("Table-7".to_string(), vec!["Smoothie".to_string()]);

    let mut customer_waitlist: HashMap<String, u32> = HashMap::new();
    customer_waitlist.insert("Johnson Party".to_string(), 4);
    customer_waitlist.insert("Smith Family".to_string(), 6);
    customer_waitlist.insert("Davis Couple".to_string(), 2);
    customer_waitlist.insert("Wilson Group".to_string(), 8);

    println!("📊 Initial Restaurant Status:");
    println!("   🍽️ Active orders: {} tables", active_orders.len());
    println!("   ⏳ Waitlist: {} parties", customer_waitlist.len());

    // Method 1: remove() - Remove and return value
    println!("\n🍽️ Completing Orders (remove with return value):");

    if let Some(completed_order) = active_orders.remove("Table-1") {
        println!("   ✅ Table-1 completed: {} items served", completed_order.len());
        println!("      Items: {}", completed_order.join(", "));
    } else {
        println!("   ❌ Table-1 not found");
    }

    if let Some(completed_order) = active_orders.remove("Table-3") {
        println!("   ✅ Table-3 completed: {} items served", completed_order.len());
        println!("      Items: {}", completed_order.join(", "));
    }

    // Method 2: Conditional removal
    println!("\n🎯 Smart Order Management:");

    // Remove small orders (1 item or less) that might be quick takeaways
    let mut tables_to_remove = Vec::new();

    for (table, order) in &active_orders {
        if order.len() <= 1 {
            tables_to_remove.push(table.clone());
        }
    }

    for table in tables_to_remove {
        if let Some(order) = active_orders.remove(&table) {
            println!("   ⚡ Quick takeaway completed: {} → {}", table, order.join(", "));
        }
    }

    // Method 3: Seating customers from waitlist
    println!("\n🪑 Seating Customers from Waitlist:");

    // Seat parties of 4 or fewer (we have small tables available)
    let mut parties_to_seat = Vec::new();

    for (party, size) in &customer_waitlist {
        if *size <= 4 {
            parties_to_seat.push(party.clone());
        }
    }

    for party in parties_to_seat {
        if let Some(party_size) = customer_waitlist.remove(&party) {
            println!("   🎉 Seated {} (party of {})", party, party_size);

            // Add them to active orders
            active_orders.insert(format!("Table-{}", party.split_whitespace().next().unwrap_or("New")),
                                vec!["Menu requested".to_string()]);
        }
    }

    // Method 4: Bulk removal operations
    println!("\n📦 End-of-Rush Cleanup:");

    // Remove all orders with specific characteristics
    let mut completed_tables = Vec::new();

    for (table, order) in &active_orders {
        // Tables that have wine are likely finishing up
        if order.iter().any(|item| item.to_lowercase().contains("wine")) {
            completed_tables.push(table.clone());
        }
    }

    for table in completed_tables {
        if let Some(order) = active_orders.remove(&table) {
            println!("   🍷 Wine service completed: {} → {} items", table, order.len());
        }
    }

    // Method 5: retain() - Keep only items that match condition
    println!("\n🧹 Selective Cleanup with retain():");

    let initial_waitlist_size = customer_waitlist.len();

    // Keep only large parties (restaurant can handle big groups now)
    customer_waitlist.retain(|party, size| {
        let keep = *size >= 6;
        if !keep {
            println!("   👋 Sent {} (party of {}) to nearby partner restaurant", party, size);
        }
        keep
    });

    println!("   📊 Waitlist reduced from {} to {} parties",
             initial_waitlist_size, customer_waitlist.len());

    // Method 6: drain() - Remove and iterate over removed items
    println!("\n🔄 Processing Remaining Waitlist:");

    if !customer_waitlist.is_empty() {
        println!("   📞 Calling remaining parties for tomorrow's reservations:");

        let drained_parties: HashMap<String, u32> = customer_waitlist.drain().collect();

        for (party, size) in drained_parties {
            println!("      📅 {} (party of {}) → Reservation for tomorrow", party, size);
        }
    }

    // Method 7: clear() - Remove all entries
    println!("\n🌙 End of Day Cleanup:");

    let remaining_orders = active_orders.len();

    if remaining_orders > 0 {
        println!("   📋 Processing final {} orders before closing:", remaining_orders);

        for (table, order) in &active_orders {
            println!("      {} → {} items (kitchen closing soon)", table, order.len());
        }

        // Clear all orders at end of day
        active_orders.clear();
        println!("   🧹 All orders cleared for end of day");
    }

    // Method 8: Safe removal patterns
    println!("\n🛡️ Safe Removal Patterns Demonstrated:");

    let mut demo_map = HashMap::new();
    demo_map.insert("item1", "value1");
    demo_map.insert("item2", "value2");

    // Safe pattern: Check before removing
    if demo_map.contains_key("item1") {
        demo_map.remove("item1");
        println!("   ✅ Safely removed item1");
    }

    // Safe pattern: Handle Option return
    match demo_map.remove("item2") {
        Some(value) => println!("   ✅ Removed item2 with value: {}", value),
        None => println!("   ℹ️ item2 was already removed"),
    }

    // Safe pattern: Remove non-existent key
    match demo_map.remove("item3") {
        Some(value) => println!("   Removed item3: {}", value),
        None => println!("   ℹ️ item3 didn't exist (no problem)"),
    }

    println!("\n📊 Final Restaurant Status:");
    println!("   🍽️ Active orders: {}", active_orders.len());
    println!("   ⏳ Waitlist: {}", customer_waitlist.len());
    println!("   ✅ Restaurant ready for tomorrow!");
}

fn main() {
    demonstrate_hashmap_removal();
}
```


## Advanced HashMap Patterns and Techniques

### 1. The Entry API - Sophisticated Data Management

**Advanced operations for complex restaurant scenarios:**

```rust
fn demonstrate_entry_api() {
    println!("🎯 HashMap Entry API - Advanced Restaurant Management");
    println!("{:=<55}", "");

    use std::collections::HashMap;

    // Customer order tracking system
    let mut customer_orders: HashMap<String, Vec<String>> = HashMap::new();
    let mut item_popularity: HashMap<String, u32> = HashMap::new();
    let mut customer_visit_count: HashMap<String, u32> = HashMap::new();

    // Simulate incoming orders
    let incoming_orders = [
        ("alice@email.com", "Quinoa Bowl"),
        ("bob@email.com", "Veggie Burger"),
        ("alice@email.com", "Green Smoothie"),
        ("carol@email.com", "Quinoa Bowl"),
        ("bob@email.com", "Lentil Soup"),
        ("alice@email.com", "Pasta Primavera"),
        ("david@email.com", "Quinoa Bowl"),
        ("carol@email.com", "Veggie Burger"),
    ];

    println!("📥 Processing Customer Orders with Entry API:");

    for (customer, item) in incoming_orders {
        // Pattern 1: or_insert() - Add if not present
        customer_orders
            .entry(customer.to_string())
            .or_insert_with(Vec::new)
            .push(item.to_string());

        // Pattern 2: Update counters
        *item_popularity
            .entry(item.to_string())
            .or_insert(0) += 1;

        *customer_visit_count
            .entry(customer.to_string())
            .or_insert(0) += 1;

        println!("   📋 {} ordered {} (visit #{})",
                 customer, item, customer_visit_count[customer]);
    }

    // Pattern 3: or_insert_with() - Expensive computation only when needed
    println!("\n🎯 Smart Default Value Creation:");

    let mut customer_profiles: HashMap<String, CustomerProfile> = HashMap::new();

    #[derive(Debug)]
    struct CustomerProfile {
        name: String,
        preferences: Vec<String>,
        loyalty_points: u32,
        join_date: String,
    }

    impl Default for CustomerProfile {
        fn default() -> Self {
            println!("      🏗️ Creating new customer profile (expensive operation)");
            CustomerProfile {
                name: "New Customer".to_string(),
                preferences: vec!["Unknown".to_string()],
                loyalty_points: 0,
                join_date: "2024-01-15".to_string(),
            }
        }
    }

    // This only creates default profile if customer doesn't exist
    let alice_profile = customer_profiles
        .entry("alice@email.com".to_string())
        .or_insert_with(CustomerProfile::default);

    alice_profile.name = "Alice Johnson".to_string();
    alice_profile.preferences = vec!["Vegan".to_string(), "Gluten-Free".to_string()];
    alice_profile.loyalty_points = 150;

    println!("   ✅ Alice's profile: {} points, {} preferences",
             alice_profile.loyalty_points, alice_profile.preferences.len());

    // Pattern 4: and_modify() - Update existing entry
    println!("\n🔄 Updating Existing Customer Data:");

    customer_profiles
        .entry("alice@email.com".to_string())
        .and_modify(|profile| {
            profile.loyalty_points += 25; // Bonus points!
            profile.preferences.push("Organic".to_string());
            println!("      🎁 Added 25 bonus points to Alice");
        })
        .or_insert_with(CustomerProfile::default);

    // Pattern 5: Complex entry operations
    println!("\n⚗️ Advanced Entry Operations:");

    let mut daily_specials: HashMap<String, DailySpecial> = HashMap::new();

    #[derive(Debug)]
    struct DailySpecial {
        dish: String,
        original_price: f64,
        discount_percent: u32,
        quantity_available: u32,
    }

    // Create or update daily special
    let special_key = "Monday".to_string();

    match daily_specials.entry(special_key.clone()) {
        std::collections::hash_map::Entry::Occupied(mut entry) => {
            // Special already exists - update it
            let special = entry.get_mut();
            special.quantity_available = special.quantity_available.saturating_sub(1);
            println!("   📉 Updated Monday special: {} remaining", special.quantity_available);

            if special.quantity_available == 0 {
                println!("   🔚 Monday special sold out!");
                entry.remove();
            }
        }
        std::collections::hash_map::Entry::Vacant(entry) => {
            // No special for Monday - create one
            entry.insert(DailySpecial {
                dish: "Mediterranean Quinoa Bowl".to_string(),
                original_price: 15.99,
                discount_percent: 20,
                quantity_available: 10,
            });
            println!("   🎉 Created new Monday special: Mediterranean Quinoa Bowl");
        }
    }

    // Pattern 6: Conditional updates based on current value
    println!("\n📊 Smart Inventory Management:");

    let mut ingredient_stock: HashMap<String, u32> = HashMap::new();
    ingredient_stock.insert("Quinoa".to_string(), 50);
    ingredient_stock.insert("Tomatoes".to_string(), 30);
    ingredient_stock.insert("Avocado".to_string(), 15);

    let ingredient_usage = [
        ("Quinoa", 5),
        ("Tomatoes", 8),
        ("Avocado", 12),
        ("Lettuce", 6), // New ingredient
    ];

    for (ingredient, used) in ingredient_usage {
        let stock_entry = ingredient_stock.entry(ingredient.to_string());

        match stock_entry {
            std::collections::hash_map::Entry::Occupied(mut entry) => {
                let current_stock = entry.get_mut();
                if *current_stock >= used {
                    *current_stock -= used;
                    println!("   ✅ Used {} {}, {} remaining", used, ingredient, current_stock);
                } else {
                    println!("   ⚠️ Not enough {} (need {}, have {})", ingredient, used, current_stock);
                    *current_stock = 0;
                }
            }
            std::collections::hash_map::Entry::Vacant(entry) => {
                println!("   📦 {} not in stock, ordering emergency supply", ingredient);
                entry.insert(100); // Emergency order
            }
        }
    }

    // Pattern 7: Entry API for statistics and analytics
    println!("\n📈 Customer Analytics with Entry API:");

    let mut order_analytics: HashMap<String, OrderStats> = HashMap::new();

    #[derive(Debug, Default)]
    struct OrderStats {
        total_orders: u32,
        total_spent: f64,
        favorite_items: HashMap<String, u32>,
    }

    // Process order history for analytics
    for (customer, orders) in &customer_orders {
        let stats = order_analytics.entry(customer.clone()).or_default();

        for item in orders {
            stats.total_orders += 1;
            stats.total_spent += 12.99; // Simplified pricing

            *stats.favorite_items.entry(item.clone()).or_insert(0) += 1;
        }
    }

    // Display analytics
    for (customer, stats) in &order_analytics {
        println!("   👤 {}: {} orders, ${:.2} spent",
                 customer, stats.total_orders, stats.total_spent);

        if let Some((favorite_item, count)) = stats.favorite_items
            .iter()
            .max_by_key(|(_, &count)| count) {
            println!("      ⭐ Favorite: {} (ordered {} times)", favorite_item, count);
        }
    }

    println!("\n🎯 Entry API Benefits Demonstrated:");
    println!("   ✅ Efficient conditional operations");
    println!("   ✅ Avoid unnecessary lookups");
    println!("   ✅ Safe updates without panics");
    println!("   ✅ Clean, readable code");
    println!("   ✅ Optimal performance for complex logic");
}

fn main() {
    demonstrate_entry_api();
}
```


### 2. Custom Key Types and Hashing

**Advanced key management for specialized restaurant systems:**

```rust
fn demonstrate_custom_keys() {
    println!("🔑 Custom HashMap Keys - Specialized Restaurant Systems");
    println!("{:=<60}", "");

    use std::collections::HashMap;
    use std::hash::{Hash, Hasher};

    // Custom key type for table reservations
    #[derive(Debug, Clone, PartialEq, Eq, Hash)]
    struct ReservationKey {
        date: String,
        time_slot: u8, // Hour in 24-hour format
        party_size: u8,
    }

    impl ReservationKey {
        fn new(date: &str, hour: u8, party_size: u8) -> Self {
            ReservationKey {
                date: date.to_string(),
                time_slot: hour,
                party_size,
            }
        }
    }

    // Custom key type for menu items with variants
    #[derive(Debug, Clone, PartialEq, Eq)]
    struct MenuItemKey {
        base_dish: String,
        size: MenuSize,
        dietary_mods: Vec<String>,
    }

    #[derive(Debug, Clone, PartialEq, Eq, Hash)]
    enum MenuSize {
        Small,
        Medium,
        Large,
    }

    // Custom Hash implementation for MenuItemKey
    impl Hash for MenuItemKey {
        fn hash<H: Hasher>(&self, state: &mut H) {
            self.base_dish.hash(state);
            self.size.hash(state);

            // Sort dietary modifications for consistent hashing
            let mut sorted_mods = self.dietary_mods.clone();
            sorted_mods.sort();
            for mod_item in sorted_mods {
                mod_item.hash(state);
            }
        }
    }

    println!("🏗️ Advanced Reservation System:");

    let mut reservations: HashMap<ReservationKey, String> = HashMap::new();

    // Add various reservations
    reservations.insert(
        ReservationKey::new("2024-01-15", 19, 4),
        "Johnson Anniversary - Window table requested".to_string()
    );

    reservations.insert(
        ReservationKey::new("2024-01-15", 20, 2),
        "Smith Business Dinner - Quiet corner preferred".to_string()
    );

    reservations.insert(
        ReservationKey::new("2024-01-16", 18, 6),
        "Davis Family Birthday - High chair needed".to_string()
    );

    // Query reservations
    let lookup_key = ReservationKey::new("2024-01-15", 19, 4);
    if let Some(details) = reservations.get(&lookup_key) {
        println!("   ✅ Found reservation: {}", details);
    }

    println!("\n📋 All reservations:");
    for (key, details) in &reservations {
        println!("   📅 {} at {}:00 for {} people → {}",
                 key.date, key.time_slot, key.party_size, details);
    }

    println!("\n🍽️ Advanced Menu Pricing System:");

    let mut menu_pricing: HashMap<MenuItemKey, f64> = HashMap::new();

    // Add menu items with variants
    menu_pricing.insert(MenuItemKey {
        base_dish: "Quinoa Bowl".to_string(),
        size: MenuSize::Medium,
        dietary_mods: vec!["Extra Avocado".to_string()],
    }, 17.99);

    menu_pricing.insert(MenuItemKey {
        base_dish: "Quinoa Bowl".to_string(),
        size: MenuSize::Large,
        dietary_mods: vec!["Extra Avocado".to_string(), "No Nuts".to_string()],
    }, 19.99);

    menu_pricing.insert(MenuItemKey {
        base_dish: "Veggie Burger".to_string(),
        size: MenuSize::Medium,
        dietary_mods: vec!["Gluten-Free Bun".to_string()],
    }, 14.49);

    // Test different orderings of dietary modifications
    let key1 = MenuItemKey {
        base_dish: "Quinoa Bowl".to_string(),
        size: MenuSize::Large,
        dietary_mods: vec!["Extra Avocado".to_string(), "No Nuts".to_string()],
    };

    let key2 = MenuItemKey {
        base_dish: "Quinoa Bowl".to_string(),
        size: MenuSize::Large,
        dietary_mods: vec!["No Nuts".to_string(), "Extra Avocado".to_string()],
    };

    // These should be treated as the same due to custom Hash implementation
    println!("   🔍 Testing hash consistency:");
    println!("   Key1 == Key2: {}", key1 == key2);

    if let Some(price) = menu_pricing.get(&key1) {
        println!("   💰 Price with key1: ${:.2}", price);
    }

    if let Some(price) = menu_pricing.get(&key2) {
        println!("   💰 Price with key2: ${:.2}", price);
    }

    // Complex key for employee scheduling
    #[derive(Debug, Clone, PartialEq, Eq, Hash)]
    struct ScheduleKey {
        employee_id: u32,
        date: String,
        shift: ShiftType,
    }

    #[derive(Debug, Clone, PartialEq, Eq, Hash)]
    enum ShiftType {
        Morning,
        Afternoon,
        Evening,
        Double,
    }

    println!("\n👥 Employee Scheduling System:");

    let mut employee_schedule: HashMap<ScheduleKey, ScheduleDetails> = HashMap::new();

    #[derive(Debug)]
    struct ScheduleDetails {
        hours: f32,
        role: String,
        notes: Option<String>,
    }

    // Schedule employees
    employee_schedule.insert(
        ScheduleKey {
            employee_id: 101,
            date: "2024-01-15".to_string(),
            shift: ShiftType::Morning,
        },
        ScheduleDetails {
            hours: 8.0,
            role: "Head Chef".to_string(),
            notes: Some("Menu planning meeting at 10 AM".to_string()),
        }
    );

    employee_schedule.insert(
        ScheduleKey {
            employee_id: 102,
            date: "2024-01-15".to_string(),
            shift: ShiftType::Evening,
        },
        ScheduleDetails {
            hours: 6.0,
            role: "Server".to_string(),
            notes: None,
        }
    );

    // Query schedule
    let check_key = ScheduleKey {
        employee_id: 101,
        date: "2024-01-15".to_string(),
        shift: ShiftType::Morning,
    };

    if let Some(schedule) = employee_schedule.get(&check_key) {
        println!("   ✅ Employee 101 morning shift: {} hours as {}",
                 schedule.hours, schedule.role);
        if let Some(notes) = &schedule.notes {
            println!("      📝 Notes: {}", notes);
        }
    }

    // Tuple keys for multi-dimensional indexing
    println!("\n📊 Sales Analytics with Tuple Keys:");

    let mut sales_data: HashMap<(String, String, u8), f64> = HashMap::new();
    // (date, dish_category, hour) -> revenue

    sales_data.insert(("2024-01-15".to_string(), "Bowls".to_string(), 12), 247.50);
    sales_data.insert(("2024-01-15".to_string(), "Bowls".to_string(), 13), 189.75);
    sales_data.insert(("2024-01-15".to_string(), "Burgers".to_string(), 12), 156.80);
    sales_data.insert(("2024-01-15".to_string(), "Smoothies".to_string(), 14), 98.25);

    // Analyze lunch hour sales
    let lunch_hours = [12, 13, 14];
    println!("   🍽️ Lunch hour performance:");

    for hour in lunch_hours {
        let mut hour_total = 0.0;
        for ((date, category, sale_hour), revenue) in &sales_data {
            if *sale_hour == hour && date == "2024-01-15" {
                hour_total += revenue;
                println!("      {}:00 {} → ${:.2}", hour, category, revenue);
            }
        }
        println!("      Total for {}:00 → ${:.2}", hour, hour_total);
    }

    println!("\n🎯 Custom Key Benefits:");
    println!("   ✅ Precise data modeling");
    println!("   ✅ Complex multi-field lookups");
    println!("   ✅ Type safety for domain concepts");
    println!("   ✅ Efficient querying of related data");
    println!("   ✅ Consistent hashing for similar data");
}

fn main() {
    demonstrate_custom_keys();
}
```


## Best Practices and Performance Considerations

### HashMap Optimization and Professional Usage

```rust
fn demonstrate_hashmap_best_practices() {
    println!("🎯 HashMap Best Practices - Professional Restaurant Systems");
    println!("{:=<65}", "");

    use std::collections::HashMap;
    use std::time::Instant;

    // Best Practice 1: Pre-size HashMaps when possible
    println!("⚡ Performance Optimization:");

    let expected_customers = 1000;

    // ✅ Good: Pre-sized HashMap
    let start = Instant::now();
    let mut optimized_customers = HashMap::with_capacity(expected_customers);
    for i in 0..expected_customers {
        optimized_customers.insert(format!("customer_{}", i), format!("Customer {}", i));
    }
    let optimized_time = start.elapsed();

    // ❌ Less efficient: Default size that needs to grow
    let start = Instant::now();
    let mut growing_customers = HashMap::new();
    for i in 0..expected_customers {
        growing_customers.insert(format!("customer_{}", i), format!("Customer {}", i));
    }
    let growing_time = start.elapsed();

    println!("   Pre-sized HashMap: {:?}", optimized_time);
    println!("   Growing HashMap: {:?}", growing_time);
    println!("   💡 Pre-sizing can be {}x faster",
             growing_time.as_nanos() / optimized_time.as_nanos().max(1));

    // Best Practice 2: Choose appropriate key types
    println!("\n🔑 Key Type Selection:");

    // ✅ Good: Use integer keys when possible (faster hashing)
    let mut menu_by_id: HashMap<u32, String> = HashMap::new();
    menu_by_id.insert(101, "Quinoa Bowl".to_string());
    menu_by_id.insert(102, "Veggie Burger".to_string());

    // ✅ Also good: String keys when necessary
    let mut menu_by_name: HashMap<String, f64> = HashMap::new();
    menu_by_name.insert("Quinoa Bowl".to_string(), 15.99);
    menu_by_name.insert("Veggie Burger".to_string(), 12.49);

    // ❌ Avoid: Complex keys that are expensive to hash
    #[derive(Hash, PartialEq, Eq)]
    struct ExpensiveKey {
        data: Vec<String>, // Expensive to hash
        metadata: HashMap<String, String>, // Very expensive to hash
    }

    println!("   ✅ Integer keys: Fastest hashing");
    println!("   ✅ String keys: Good for most cases");
    println!("   ⚠️ Complex struct keys: Use carefully");
    println!("   ❌ Nested collection keys: Generally avoid");

    // Best Practice 3: Memory-efficient value storage
    println!("\n💾 Memory Optimization:");

    // ✅ Good: Use references when data is owned elsewhere
    let menu_items = vec!["Quinoa Bowl", "Veggie Burger", "Lentil Soup"];
    let mut item_ratings: HashMap<&str, f32> = HashMap::new();

    for item in &menu_items {
        item_ratings.insert(item, 4.5);
    }

    // ✅ Good: Use Box for large values to avoid stack overflow
    #[derive(Debug)]
    struct LargeCustomerData {
        order_history: Vec<String>,  // Could be hundreds of orders
        preferences: Vec<String>,
        reviews: Vec<String>,
        // ... many more fields
    }

    let mut customer_data: HashMap<String, Box<LargeCustomerData>> = HashMap::new();
    customer_data.insert("alice@email.com".to_string(), Box::new(LargeCustomerData {
        order_history: vec!["Quinoa Bowl".to_string(); 100],
        preferences: vec!["Vegan".to_string(), "Organic".to_string()],
        reviews: vec!["Great food!".to_string()],
    }));

    println!("   ✅ Reference values when possible");
    println!("   ✅ Box large values to keep HashMap lean");
    println!("   ✅ Consider Arc<T> for shared ownership");

    // Best Practice 4: Safe HashMap operations
    println!("\n🛡️ Safe Operations:");

    let mut order_status: HashMap<String, String> = HashMap::new();
    order_status.insert("ORDER_001".to_string(), "Preparing".to_string());
    order_status.insert("ORDER_002".to_string(), "Ready".to_string());

    // ✅ Good: Always handle the Option
    match order_status.get("ORDER_001") {
        Some(status) => println!("   Order status: {}", status),
        None => println!("   Order not found"),
    }

    // ✅ Good: Use entry API for conditional operations
    order_status.entry("ORDER_003".to_string())
        .or_insert("New Order".to_string());

    // ❌ Dangerous: Direct indexing can panic
    // let status = order_status["ORDER_999"]; // Would panic!

    println!("   ✅ Always use get() or entry API");
    println!("   ✅ Handle Option return values");
    println!("   ❌ Avoid direct indexing");

    // Best Practice 5: Iteration patterns
    println!("\n🔄 Efficient Iteration:");

    let mut daily_sales: HashMap<String, f64> = HashMap::new();
    daily_sales.insert("Quinoa Bowl".to_string(), 159.99);
    daily_sales.insert("Veggie Burger".to_string(), 98.75);
    daily_sales.insert("Lentil Soup".to_string(), 67.25);

    // ✅ Good: Borrow for read-only operations
    let total_sales: f64 = daily_sales.values().sum();
    println!("   Total daily sales: ${:.2}", total_sales);

    // ✅ Good: Use iter() for key-value pairs
    for (item, sales) in &daily_sales {
        if *sales > 100.0 {
            println!("   🏆 Top seller: {} (${:.2})", item, sales);
        }
    }

    // ✅ Good: Use into_iter() when consuming
    let sales_vec: Vec<(String, f64)> = daily_sales.into_iter().collect();
    println!("   Converted to vector: {} items", sales_vec.len());

    // Best Practice 6: Error handling patterns
    println!("\n🚨 Error Handling Patterns:");

    let mut inventory: HashMap<String, u32> = HashMap::new();
    inventory.insert("Quinoa".to_string(), 50);
    inventory.insert("Tomatoes".to_string(), 30);

    // Pattern 1: Graceful degradation
    fn get_inventory_safe(inventory: &HashMap<String, u32>, item: &str) -> u32 {
        inventory.get(item).copied().unwrap_or(0)
    }

    let quinoa_stock = get_inventory_safe(&inventory, "Quinoa");
    let rice_stock = get_inventory_safe(&inventory, "Rice"); // Not in inventory

    println!("   Quinoa stock: {} (found)", quinoa_stock);
    println!("   Rice stock: {} (default)", rice_stock);

    // Pattern 2: Result-based operations
    fn update_inventory(
        inventory: &mut HashMap<String, u32>,
        item: &str,
        amount: u32
    ) -> Result<u32, String> {
        match inventory.get_mut(item) {
            Some(stock) => {
                if *stock >= amount {
                    *stock -= amount;
                    Ok(*stock)
                } else {
                    Err(format!("Insufficient stock: need {}, have {}", amount, stock))
                }
            }
            None => Err(format!("Item '{}' not found in inventory", item))
        }
    }

    match update_inventory(&mut inventory, "Quinoa", 10) {
        Ok(remaining) => println!("   ✅ Used quinoa, {} remaining", remaining),
        Err(error) => println!("   ❌ Error: {}", error),
    }

    match update_inventory(&mut inventory, "Rice", 5) {
        Ok(remaining) => println!("   ✅ Used rice, {} remaining", remaining),
        Err(error) => println!("   ❌ Error: {}", error),
    }

    println!("\n📋 Best Practices Summary:");
    println!("   ✅ Pre-size HashMaps when capacity is known");
    println!("   ✅ Choose efficient key types (prefer integers)");
    println!("   ✅ Use references or Box for large values");
    println!("   ✅ Always handle Option return values safely");
    println!("   ✅ Use entry API for conditional operations");
    println!("   ✅ Implement proper error handling");
    println!("   ✅ Profile performance in production use cases");
}

fn main() {
    demonstrate_hashmap_best_practices();
}
```


## Common Mistakes and How to Avoid Them

### Learning from Restaurant Management Disasters

```rust
fn demonstrate_common_mistakes() {
    println!("⚠️ Common HashMap Mistakes - Learning from Restaurant Disasters");
    println!("{:=<65}", "");

    use std::collections::HashMap;

    println!("❌ MISTAKE 1: Direct indexing without checking");
    let mut orders: HashMap<String, String> = HashMap::new();
    orders.insert("ORDER_001".to_string(), "Quinoa Bowl".to_string());

    // ❌ This will panic if key doesn't exist!
    // let order = orders["ORDER_999"]; // PANIC!

    println!("   ✅ FIX: Always use safe access methods");
    match orders.get("ORDER_999") {
        Some(order) => println!("   Found order: {}", order),
        None => println!("   Order not found - handled safely"),
    }

    println!("\n❌ MISTAKE 2: Forgetting HashMap ownership rules");
    let customer_names = vec!["Alice".to_string(), "Bob".to_string()];
    let mut customer_ages: HashMap<String, u32> = HashMap::new();

    // ❌ This moves the strings out of the vector
    // for name in customer_names {
    //     customer_ages.insert(name, 25);
    // }
    // println!("{:?}", customer_names); // Error: value used after move

    println!("   ✅ FIX: Use references or clone when needed");
    for name in &customer_names {
        customer_ages.insert(name.clone(), 25);
    }
    println!("   Original vector still usable: {:?}", customer_names);

    println!("\n❌ MISTAKE 3: Modifying HashMap while iterating");
    let mut menu_items: HashMap<String, f64> = HashMap::new();
    menu_items.insert("Quinoa Bowl".to_string(), 15.99);
    menu_items.insert("Veggie Burger".to_string(), 12.49);
    menu_items.insert("Old Dish".to_string(), 8.99);

    // ❌ This won't compile - can't modify while borrowing for iteration
    // for (name, price) in &menu_items {
    //     if *price < 10.0 {
    //         menu_items.remove(name); // Error: cannot borrow as mutable
    //     }
    // }

    println!("   ✅ FIX: Collect keys to remove first, then remove");
    let mut items_to_remove = Vec::new();
    for (name, price) in &menu_items {
        if *price < 10.0 {
            items_to_remove.push(name.clone());
        }
    }

    for item in items_to_remove {
        menu_items.remove(&item);
        println!("   Removed old item: {}", item);
    }

    println!("\n❌ MISTAKE 4: Not handling the Option in remove()");
    println!("   ✅ FIX: Always handle the Option from remove()");

    match menu_items.remove("Nonexistent Item") {
        Some(price) => println!("   Removed item with price: ${:.2}", price),
        None => println!("   Item didn't exist (no problem)"),
    }

    println!("\n❌ MISTAKE 5: Inefficient key types");
    // ❌ Using String when &str would work
    fn bad_lookup_function(items: &HashMap<String, f64>, key: &str) -> Option<f64> {
        items.get(&key.to_string()).copied() // Unnecessary allocation!
    }

    // ✅ Better: Use consistent key types
    fn good_lookup_function(items: &HashMap<String, f64>, key: &str) -> Option<f64> {
        items.get(key).copied() // Direct lookup
    }

    let test_map: HashMap<String, f64> = HashMap::new();
    let _ = good_lookup_function(&test_map, "test");
    println!("   ✅ Used efficient key lookup");

    println!("\n❌ MISTAKE 6: Not considering key mutability");
    use std::collections::HashMap;

    // ❌ Problematic: Using mutable types as keys
    #[derive(Hash, PartialEq, Eq)]
    struct MutableKey {
        data: Vec<String>, // Mutable!
    }

    println!("   ⚠️ Avoid mutable types as HashMap keys");
    println!("   ✅ Use immutable types for keys when possible");

    println!("\n❌ MISTAKE 7: Ignoring HashMap capacity management");
    println!("   ❌ Creating HashMap without considering final size");
    let mut growing_map = HashMap::new(); // Will resize multiple times

    println!("   ✅ Pre-size when you know approximate capacity");
    let mut efficient_map = HashMap::with_capacity(1000); // One allocation

    efficient_map.insert("example".to_string(), "value".to_string());
    println!("   Efficient map capacity: {}", efficient_map.capacity());

    println!("\n❌ MISTAKE 8: Using HashMap for small, fixed collections");
    // ❌ Overkill for small, known sets
    let mut small_config = HashMap::new();
    small_config.insert("timeout", 30);
    small_config.insert("retries", 3);
    small_config.insert("port", 8080);

    // ✅ Better: Use match or simple struct for small, fixed data
    struct Config {
        timeout: u32,
        retries: u32,
        port: u32,
    }

    let efficient_config = Config {
        timeout: 30,
        retries: 3,
        port: 8080,
    };

    println!("   ✅ Used struct instead of HashMap for small fixed data");

    println!("\n❌ MISTAKE 9: Not handling hash collisions gracefully");
    println!("   💡 Rust's HashMap handles collisions automatically");
    println!("   💡 But choose good key types that distribute well");

    // ❌ Poor key distribution
    let mut bad_keys: HashMap<u8, String> = HashMap::new(); // Only 256 possible keys

    // ✅ Better key distribution
    let mut good_keys: HashMap<String, String> = HashMap::new(); // Unlimited keys

    good_keys.insert("unique_customer_id_12345".to_string(), "data".to_string());
    println!("   ✅ Used well-distributed key type");

    println!("\n📋 Mistake Prevention Checklist:");
    println!("   ✅ Always use get() instead of direct indexing");
    println!("   ✅ Handle ownership correctly (clone vs reference)");
    println!("   ✅ Don't modify HashMap while iterating");
    println!("   ✅ Handle Option return values from remove()");
    println!("   ✅ Use efficient key types and avoid unnecessary allocations");
    println!("   ✅ Consider mutability implications for keys");
    println!("   ✅ Pre-size HashMap when capacity is known");
    println!("   ✅ Use appropriate data structures for the use case");
    println!("   ✅ Choose keys that distribute well across hash space");
}

fn main() {
    demonstrate_common_mistakes();
}
```


## Summary and Key Takeaways

### **Mental Model: The Professional Restaurant Management System**

Remember our restaurant management analogy:

- 🗝️ **HashMap** = **Advanced reservation and customer system** - Instant access to any information
- 📋 **Keys** = **Customer IDs, table numbers, order IDs** - Unique identifiers for quick lookup
- 💾 **Values** = **Customer details, orders, preferences** - The information you want to store and retrieve
- ⚡ **Hashing** = **Smart filing system** - Converts names/IDs into exact storage locations
- 🎯 **Entry API** = **Sophisticated management tools** - Handle complex operations efficiently


### **Essential HashMap Operations Quick Reference**

| **Operation** | **Method** | **Use Case** | **Example** |
| :-- | :-- | :-- | :-- |
| **Create** | `HashMap::new()` | Empty map | `let mut customers = HashMap::new();` |
| **Add/Update** | `insert(key, value)` | Store data | `customers.insert("alice", "VIP");` |
| **Get** | `get(key)` | Safe access | `customers.get("alice")` |
| **Remove** | `remove(key)` | Delete entry | `customers.remove("alice")` |
| **Check exists** | `contains_key(key)` | Verify presence | `customers.contains_key("alice")` |
| **Conditional** | `entry(key).or_insert()` | Smart updates | `stats.entry("count").or_insert(0) += 1;` |

### **HashMap Creation Patterns**

```rust
use std::collections::HashMap;

// Empty HashMap
let mut customers = HashMap::new();

// With initial capacity
let mut large_db = HashMap::with_capacity(1000);

// From array data
let prices = HashMap::from([
    ("Quinoa Bowl", 15.99),
    ("Veggie Burger", 12.49),
]);

// From iterator
let ratings: HashMap<&str, f32> = dishes
    .iter()
    .map(|dish| (*dish, 4.5))
    .collect();
```


### **Safe Access Patterns**

```rust
// ✅ Safe access patterns
match customers.get("alice") {
    Some(info) => println!("Customer: {}", info),
    None => println!("Customer not found"),
}

// ✅ Safe with defaults
let info = customers.get("alice").unwrap_or(&"New Customer");

// ✅ Safe conditional updates
customers.entry("alice".to_string())
    .or_insert("Default Info".to_string());

// ❌ Dangerous pattern
// let info = customers["alice"]; // Can panic!
```


### **Best Practices Checklist**

**✅ DO:**

- Use `get()` for safe access, never direct indexing
- Pre-size HashMaps with `with_capacity()` when size is known
- Handle `Option` return values properly
- Use the Entry API for conditional operations
- Choose efficient key types (prefer integers over complex types)
- Use references when possible to avoid unnecessary clones

**❌ DON'T:**

- Use direct indexing `map[key]` without being certain key exists
- Modify HashMap while iterating over it
- Ignore the `Option` return from `remove()` and `get()`
- Use HashMap for small, fixed collections (use structs instead)
- Choose mutable types as keys
- Create unnecessary string allocations during lookups


### **Real-World Applications**

**HashMap is perfect for:**

- 🏪 **Customer databases** - Quick customer information lookup
- 📊 **Caching systems** - Store computed results for fast retrieval
- 🎮 **Game state management** - Player stats, inventory, settings
- 🌐 **Web applications** - Session data, user preferences, configurations
- 📈 **Analytics** - Counting occurrences, aggregating data
- 🔄 **Translation systems** - Language dictionaries, code mappings


### **Performance Characteristics**

- ⚡ **Average case:** O(1) for insert, get, remove
- 📊 **Space complexity:** O(n) where n is number of key-value pairs
- 🎯 **When to use:** When you need fast lookups by key
- 🔄 **When not to use:** Small fixed collections, ordered data requirements


### **The Professional Advantage**

**HashMap in Rust is like having a world-class restaurant management system** that instantly handles any customer request:

- ⚡ **Lightning-fast access** - Find any information immediately
- 🛡️ **Memory safety** - Rust prevents common pointer errors
- 🎯 **Type safety** - Keys and values are strongly typed
- 📈 **Excellent performance** - Scales to millions of entries efficiently
- 🔄 **Flexible operations** - Rich API for complex data management

**Mastering HashMap transforms you from a beginner who searches through lists to a professional who builds efficient, scalable Rust applications.** Just as a sophisticated restaurant can serve hundreds of customers simultaneously by having the right information systems, a well-designed Rust program uses HashMap to handle complex data relationships efficiently and safely.

This fundamental data structure is the backbone of modern software development - learn it well, and you'll build faster, more reliable programs that scale beautifully from small projects to enterprise systems![^1][^2][^3][^4][^5][^6]

1. https://www.w3schools.com/rust/rust_hashmap.php
2. https://www.programiz.com/rust/hashmap
3. https://doc.rust-lang.org/book/ch08-03-hash-maps.html
4. https://doc.rust-lang.org/std/collections/struct.HashMap.html
5. https://rustbyexample.io/hash-maps
6. https://www.geeksforgeeks.org/rust/rust-hashmaps/
7. https://www.geeksforgeeks.org/java/java-util-hashmap-in-java-with-examples/
8. https://www.w3schools.com/java/java_hashmap.asp
9. https://www.programiz.com/java-programming/hashmap
10. https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html
11. https://stackoverflow.com/questions/6802483/how-to-directly-initialize-a-hashmap-in-a-literal-way
12. https://www.baeldung.com/java-hashmap-advanced
13. https://java-programming.mooc.fi/part-8/2-hash-map/
14. https://www.baeldung.com/java-initialize-hashmap
15. https://www.w3schools.com/java/java_ref_hashmap.asp
16. https://www.youtube.com/watch?v=-yXcVNqo0DQ
17. https://javaconceptoftheday.com/java-hashmap-programs-and-examples/
18. https://www.geeksforgeeks.org/java/initialize-hashmap-in-java/
19. https://www.geeksforgeeks.org/java/internal-working-of-hashmap-java/
20. https://www.baeldung.com/java-hashmap
21. https://leetcode.com/problems/design-hashmap/
22. https://codesignal.com/learn/courses/mastering-algorithms-hashmaps-two-pointers-and-beyond-in-java/lessons/using-hashmap-for-optimal-list-manipulation-and-analysis-in-java
23. https://dev.to/iamdipankarpaul/comprehensive-guide-to-hashmaps-in-rust-mci
24. https://www.youtube.com/watch?v=H62Jfv1DJlU
25. https://builtin.com/articles/hashmap-in-java
26. https://notes.kodekloud.com/docs/Rust-Programming/Collections-Error-Handling/Hashmaps
27. https://www.dotnetperls.com/hashmap-rust
28. https://www.youtube.com/watch?v=QYesA996W1U
29. https://doc.rust-lang.org/rust-by-example/std/hash.html
30. https://dev.to/fadygrab/learning-rust-18-rust-collections-hashmaps-accessing-values-with-keys-instead-of-indices-4kg2
31. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch08-03-hash-maps.html
32. https://www.koderhq.com/tutorial/rust/hashmap/
33. https://www.reddit.com/r/rust/comments/2sjayc/building_a_hashmap_in_rust_part_1_whats_a_hashmap/
34. https://updraft.cyfrin.io/courses/rust-programming-basics/data-types/hash-map-exercises
35. https://stackoverflow.com/questions/27582739/how-do-i-create-a-hashmap-literal
36. https://www.linkedin.com/learning/practice-it-rust-file-manipulation/overview-of-hashmap
37. https://exercism.org/tracks/rust/concepts/hashmap
38. https://techwasti.com/day-21learning-strings-hash-maps-and-vectors-in-rust
39. https://labex.io/tutorials/rust-rust-hashmap-data-storage-tutorial-99260
