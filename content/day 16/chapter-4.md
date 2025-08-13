+++
title = "Vector of Different Types with Enums"
description = "Learn how to store multiple data types in a single Rust vector using enums for flexible and type-safe collections."
date = 2025-08-13
weight = 4
keywords = ["Rust", "vectors", "enums", "multiple types", "collections", "type safety"]
+++

# Vectors of Different Types with Enums in Rust: Comprehensive Documentation for Beginners

Understanding how to store different types in vectors using enums is like learning to **manage a diverse vegetarian restaurant menu system where each dish can be completely different** - you need a unified way to handle salads, soups, main courses, and desserts all in the same ordering system, even though each item has different properties, ingredients, and preparation methods. Just as a professional restaurant uses a standardized order ticket system that can accommodate any type of dish, Rust's enums let you create vectors that safely store different types of data under one unified interface.

## The Diverse Restaurant Menu System Analogy 🍽️

### Imagine You're Managing a Vegetarian Restaurant with Diverse Menu Items

**The Problem - Different Types of Menu Items:**

```rust
// ❌ This won't work - vectors need the same type
let menu_items = vec![
    "Caesar Salad",           // &str
    42.5,                     // f64 (price)
    true,                     // bool (available)
    vec!["lettuce", "cheese"] // Vec<&str> (ingredients)
];
// Error: cannot mix different types in a vector!
```

**The Solution - Unified Menu Item System:**

```rust
// ✅ Use enums to create a unified type system
#[derive(Debug)]
enum MenuItem {
    Salad(String),                    // Text-based dish name
    Price(f64),                       // Numerical price
    Available(bool),                  // Boolean availability
    Ingredients(Vec<String>),         // List of ingredients
}

let unified_menu = vec![
    MenuItem::Salad("Caesar Salad".to_string()),
    MenuItem::Price(12.99),
    MenuItem::Available(true),
    MenuItem::Ingredients(vec!["lettuce".to_string(), "parmesan".to_string()]),
];
// All items are now MenuItem type - vector is happy!
```

**Why This Works Better:**

- 🎯 **Type unification** - All items are technically the same type (MenuItem)
- 🔄 **Flexible storage** - Each variant can hold completely different data
- 🛡️ **Type safety** - Rust ensures you handle all possible cases
- 📋 **Organized system** - Clear categories for different kinds of data
- ⚡ **Performance** - Efficient memory layout with no heap allocations for enum itself


## Understanding the Core Problem and Solution

### Why Vectors Need Same Types

Rust vectors require all elements to be the same type because the vector needs to know:

```rust
fn demonstrate_the_problem() {
    println!("🚫 The Vector Type Problem");
    println!("{:=<40}", "");

    // Each type has different memory requirements
    let string_data = "Hello World";     // String slice - pointer + length
    let number_data = 42;                // i32 - 4 bytes
    let boolean_data = true;             // bool - 1 byte
    let vector_data = vec![1, 2, 3];     // Vec<i32> - pointer + length + capacity

    println!("Memory sizes:");
    println!("   String slice: {} bytes", std::mem::size_of_val(&string_data));
    println!("   Integer: {} bytes", std::mem::size_of_val(&number_data));
    println!("   Boolean: {} bytes", std::mem::size_of_val(&boolean_data));
    println!("   Vector: {} bytes", std::mem::size_of_val(&vector_data));

    // ❌ This won't compile - different types
    // let mixed_items = vec![string_data, number_data, boolean_data];

    println!("\n❌ Problem: Vector needs to know:");
    println!("   • How much memory each element takes");
    println!("   • How to properly drop each element");
    println!("   • Memory alignment requirements");
    println!("   • Type-specific operations");
}

fn main() {
    demonstrate_the_problem();
}
```


### How Enums Solve This Problem

**Enums create a unified type that can hold different data:**

```rust
// The enum solution - all variants are the same type
#[derive(Debug)]
enum RestaurantData {
    DishName(String),
    Price(f64),
    IsVegan(bool),
    Ingredients(Vec<String>),
    ServingSize(u32),
    Rating(f32),
}

fn demonstrate_enum_solution() {
    println!("✅ The Enum Solution");
    println!("{:=<30}", "");

    // All items are RestaurantData type
    let restaurant_info = vec![
        RestaurantData::DishName("Quinoa Buddha Bowl".to_string()),
        RestaurantData::Price(15.99),
        RestaurantData::IsVegan(true),
        RestaurantData::Ingredients(vec![
            "quinoa".to_string(),
            "chickpeas".to_string(),
            "avocado".to_string(),
        ]),
        RestaurantData::ServingSize(450), // grams
        RestaurantData::Rating(4.8),
    ];

    println!("🍽️ Restaurant data (all same type: RestaurantData):");

    // All items can be stored together
    for (index, item) in restaurant_info.iter().enumerate() {
        println!("   Item {}: {:?}", index + 1, item);
    }

    // Each enum variant takes the same amount of space
    println!("\n📊 Memory efficiency:");
    println!("   Enum size: {} bytes", std::mem::size_of::<RestaurantData>());
    println!("   Vector capacity: {} items", restaurant_info.capacity());
    println!("   Total vector memory: {} bytes",
             restaurant_info.capacity() * std::mem::size_of::<RestaurantData>());
}

fn main() {
    demonstrate_enum_solution();
}
```


## Creating Enums for Different Data Types

### 1. Basic Enum Variants with Different Types

**Like having different categories of restaurant information:**

```rust
#[derive(Debug, Clone)]
enum MenuInformation {
    // Simple types
    Name(String),
    Price(f64),
    IsAvailable(bool),
    PrepTimeMinutes(u32),

    // Complex types
    Ingredients(Vec<String>),
    NutritionalInfo { calories: u32, protein: f32, carbs: f32 },
    CustomerReviews(Vec<(String, u8)>), // (review, rating)

    // Nested structures
    ChefSpecial {
        dish: String,
        special_price: f64,
        limited_quantity: Option<u32>,
    },
}

fn demonstrate_basic_enum_variants() {
    println!("🍽️ Menu Information Management System");
    println!("{:=<45}", "");

    let mut menu_database = Vec::new();

    // Add different types of information
    menu_database.push(MenuInformation::Name("Mediterranean Quinoa Bowl".to_string()));
    menu_database.push(MenuInformation::Price(16.99));
    menu_database.push(MenuInformation::IsAvailable(true));
    menu_database.push(MenuInformation::PrepTimeMinutes(15));

    menu_database.push(MenuInformation::Ingredients(vec![
        "quinoa".to_string(),
        "cucumber".to_string(),
        "tomatoes".to_string(),
        "olives".to_string(),
        "feta cheese".to_string(),
    ]));

    menu_database.push(MenuInformation::NutritionalInfo {
        calories: 450,
        protein: 18.5,
        carbs: 52.0,
    });

    menu_database.push(MenuInformation::CustomerReviews(vec![
        ("Amazing flavors!".to_string(), 5),
        ("Fresh and healthy".to_string(), 4),
        ("Perfect portion size".to_string(), 5),
    ]));

    menu_database.push(MenuInformation::ChefSpecial {
        dish: "Mediterranean Quinoa Bowl".to_string(),
        special_price: 13.99,
        limited_quantity: Some(20),
    });

    println!("📊 Menu Database Contents:");
    for (index, info) in menu_database.iter().enumerate() {
        println!("   Entry {}: {:?}", index + 1, info);
    }

    println!("\n📈 Database Statistics:");
    println!("   Total entries: {}", menu_database.len());
    println!("   Memory per entry: {} bytes", std::mem::size_of::<MenuInformation>());
    println!("   Total memory used: {} bytes",
             menu_database.len() * std::mem::size_of::<MenuInformation>());
}

fn main() {
    demonstrate_basic_enum_variants();
}
```


### 2. Practical Restaurant Order System

**A complete ordering system using enums to handle different types of order data:**

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
enum OrderData {
    // Customer information
    CustomerName(String),
    TableNumber(u32),
    PhoneNumber(String),

    // Order details
    OrderId(u64),
    Items(Vec<String>),
    Quantities(Vec<u32>),
    Prices(Vec<f64>),

    // Special requirements
    DietaryRestrictions(Vec<String>),
    Allergies(Vec<String>),
    SpecialInstructions(String),

    // Order status
    IsRushOrder(bool),
    EstimatedPrepTime(u32),
    PaymentMethod(String),

    // Complex data
    Discounts {
        discount_type: String,
        percentage: f64,
        amount: f64,
    },

    // Timestamps
    OrderTime(String),
    PromisedTime(String),
}

fn demonstrate_restaurant_order_system() {
    println!("📋 Restaurant Order Management System");
    println!("{:=<45}", "");

    // Create a complete order using different data types
    let order_data = vec![
        OrderData::OrderId(10001),
        OrderData::CustomerName("Alice Johnson".to_string()),
        OrderData::TableNumber(7),
        OrderData::PhoneNumber("+1-555-0123".to_string()),

        OrderData::Items(vec![
            "Quinoa Buddha Bowl".to_string(),
            "Green Goddess Smoothie".to_string(),
            "Avocado Toast".to_string(),
        ]),

        OrderData::Quantities(vec![1, 2, 1]),
        OrderData::Prices(vec![15.99, 8.99, 12.99]),

        OrderData::DietaryRestrictions(vec![
            "vegetarian".to_string(),
            "gluten-free".to_string(),
        ]),

        OrderData::Allergies(vec!["nuts".to_string()]),
        OrderData::SpecialInstructions("Extra avocado on the toast please".to_string()),

        OrderData::IsRushOrder(false),
        OrderData::EstimatedPrepTime(25),
        OrderData::PaymentMethod("credit card".to_string()),

        OrderData::Discounts {
            discount_type: "first-time customer".to_string(),
            percentage: 10.0,
            amount: 4.70,
        },

        OrderData::OrderTime("2024-01-15 12:30:00".to_string()),
        OrderData::PromisedTime("2024-01-15 12:55:00".to_string()),
    ];

    println!("🍽️ Complete Order Information:");

    // Process the order data
    for (index, data) in order_data.iter().enumerate() {
        match data {
            OrderData::OrderId(id) => {
                println!("   🆔 Order ID: #{}", id);
            }
            OrderData::CustomerName(name) => {
                println!("   👤 Customer: {}", name);
            }
            OrderData::TableNumber(table) => {
                println!("   🪑 Table: {}", table);
            }
            OrderData::Items(items) => {
                println!("   🍽️ Items ordered:");
                for (i, item) in items.iter().enumerate() {
                    println!("      {}. {}", i + 1, item);
                }
            }
            OrderData::Prices(prices) => {
                let total: f64 = prices.iter().sum();
                println!("   💰 Individual prices: {:?}", prices);
                println!("   💰 Subtotal: ${:.2}", total);
            }
            OrderData::DietaryRestrictions(restrictions) => {
                println!("   🌱 Dietary restrictions: {}", restrictions.join(", "));
            }
            OrderData::Discounts { discount_type, percentage, amount } => {
                println!("   🎟️ Discount: {} - {}% off (${:.2})",
                        discount_type, percentage, amount);
            }
            OrderData::EstimatedPrepTime(minutes) => {
                println!("   ⏰ Estimated prep time: {} minutes", minutes);
            }
            _ => {
                // Handle other cases if needed
                println!("   📄 Data entry {}: {:?}", index + 1, data);
            }
        }
    }

    println!("\n📊 Order Summary:");
    println!("   Total data entries: {}", order_data.len());
    println!("   Vector type: Vec<OrderData>");
    println!("   All entries safely stored in single vector!");
}

fn main() {
    demonstrate_restaurant_order_system();
}
```


### 3. Advanced Enum Patterns for Complex Data

**Handling sophisticated restaurant management data:**

```rust
use std::collections::HashMap;

#[derive(Debug)]
enum RestaurantAnalytics {
    // Numeric data
    DailySales(f64),
    CustomerCount(u32),
    AverageOrderValue(f64),

    // Time series data
    HourlySales(Vec<f64>),
    WeeklyTrends(HashMap<String, f64>),

    // Text analytics
    PopularDishes(Vec<(String, u32)>), // (dish name, order count)
    CustomerFeedback(Vec<String>),
    StaffNotes(String),

    // Complex structured data
    InventoryReport {
        item: String,
        current_stock: u32,
        min_threshold: u32,
        cost_per_unit: f64,
        supplier: String,
    },

    // Nested enums
    OrderStatus(OrderProcessingStage),

    // Result-like data
    KitchenOperation {
        operation: String,
        success: bool,
        details: Option<String>,
    },

    // Mixed data types in tuples
    MenuPerformance(String, f64, u32, bool), // (dish, rating, orders, still_available)
}

#[derive(Debug)]
enum OrderProcessingStage {
    Received,
    InPreparation { chef: String, started_at: String },
    QualityCheck { checker: String, passed: bool },
    ReadyToServe,
    Served { server: String, table: u32 },
}

fn demonstrate_advanced_enum_patterns() {
    println!("📊 Advanced Restaurant Analytics System");
    println!("{:=<50}", "");

    let mut analytics_data = Vec::new();

    // Add various types of analytics data
    analytics_data.push(RestaurantAnalytics::DailySales(2847.50));
    analytics_data.push(RestaurantAnalytics::CustomerCount(127));
    analytics_data.push(RestaurantAnalytics::AverageOrderValue(22.42));

    analytics_data.push(RestaurantAnalytics::HourlySales(vec![
        120.50, 89.25, 156.75, 234.80, 298.60, 187.45, 156.90, 203.25
    ]));

    let mut weekly_trends = HashMap::new();
    weekly_trends.insert("Monday".to_string(), 1850.75);
    weekly_trends.insert("Tuesday".to_string(), 1654.20);
    weekly_trends.insert("Wednesday".to_string(), 2123.45);
    weekly_trends.insert("Thursday".to_string(), 2456.80);
    weekly_trends.insert("Friday".to_string(), 3102.60);
    analytics_data.push(RestaurantAnalytics::WeeklyTrends(weekly_trends));

    analytics_data.push(RestaurantAnalytics::PopularDishes(vec![
        ("Quinoa Buddha Bowl".to_string(), 23),
        ("Mediterranean Wrap".to_string(), 19),
        ("Green Goddess Smoothie".to_string(), 31),
    ]));

    analytics_data.push(RestaurantAnalytics::CustomerFeedback(vec![
        "Amazing fresh ingredients!".to_string(),
        "Perfect portion sizes".to_string(),
        "Love the variety of options".to_string(),
    ]));

    analytics_data.push(RestaurantAnalytics::InventoryReport {
        item: "Organic Quinoa".to_string(),
        current_stock: 45,
        min_threshold: 20,
        cost_per_unit: 3.25,
        supplier: "Organic Farms Co".to_string(),
    });

    analytics_data.push(RestaurantAnalytics::OrderStatus(
        OrderProcessingStage::InPreparation {
            chef: "Marco".to_string(),
            started_at: "12:45 PM".to_string(),
        }
    ));

    analytics_data.push(RestaurantAnalytics::KitchenOperation {
        operation: "Deep clean prep stations".to_string(),
        success: true,
        details: Some("All stations sanitized and restocked".to_string()),
    });

    analytics_data.push(RestaurantAnalytics::MenuPerformance(
        "Avocado Toast".to_string(), 4.6, 18, true
    ));

    println!("📈 Analytics Data Processing:");

    for (index, data) in analytics_data.iter().enumerate() {
        match data {
            RestaurantAnalytics::DailySales(amount) => {
                println!("   💰 Daily Sales: ${:.2}", amount);
            }
            RestaurantAnalytics::CustomerCount(count) => {
                println!("   👥 Customer Count: {}", count);
            }
            RestaurantAnalytics::HourlySales(sales) => {
                let peak_hour = sales.iter().enumerate()
                    .max_by(|(_, a), (_, b)| a.partial_cmp(b).unwrap())
                    .map(|(hour, amount)| (hour, amount));

                if let Some((hour, amount)) = peak_hour {
                    println!("   📊 Peak Sales Hour: {}:00 (${:.2})", hour + 9, amount);
                }
            }
            RestaurantAnalytics::WeeklyTrends(trends) => {
                if let Some(best_day) = trends.iter()
                    .max_by(|(_, a), (_, b)| a.partial_cmp(b).unwrap()) {
                    println!("   📅 Best Day: {} (${:.2})", best_day.0, best_day.1);
                }
            }
            RestaurantAnalytics::PopularDishes(dishes) => {
                if let Some((dish, count)) = dishes.iter()
                    .max_by(|(_, a), (_, b)| a.cmp(b)) {
                    println!("   🏆 Most Popular: {} ({} orders)", dish, count);
                }
            }
            RestaurantAnalytics::InventoryReport { item, current_stock, min_threshold, .. } => {
                let status = if current_stock <= min_threshold {
                    "🚨 LOW STOCK"
                } else {
                    "✅ SUFFICIENT"
                };
                println!("   📦 Inventory: {} - {} units {}", item, current_stock, status);
            }
            RestaurantAnalytics::OrderStatus(stage) => {
                match stage {
                    OrderProcessingStage::InPreparation { chef, started_at } => {
                        println!("   👨‍🍳 Order in prep: Chef {} started at {}", chef, started_at);
                    }
                    _ => println!("   📋 Order Status: {:?}", stage),
                }
            }
            RestaurantAnalytics::MenuPerformance(dish, rating, orders, available) => {
                let status = if *available { "Available" } else { "Sold Out" };
                println!("   🍽️ {}: {:.1}⭐ ({} orders) - {}", dish, rating, orders, status);
            }
            _ => {
                println!("   📄 Entry {}: {:?}", index + 1, data);
            }
        }
    }

    println!("\n🎯 System Summary:");
    println!("   Total analytics entries: {}", analytics_data.len());
    println!("   Memory per entry: {} bytes", std::mem::size_of::<RestaurantAnalytics>());
    println!("   ✅ All different data types unified in one vector!");
}

fn main() {
    demonstrate_advanced_enum_patterns();
}
```


## Working with Vectors of Enums - Practical Operations

### 1. Adding and Removing Items

**Managing your heterogeneous data collection:**

```rust
#[derive(Debug, Clone)]
enum InventoryItem {
    Ingredient(String),
    Quantity(u32),
    ExpirationDate(String),
    Price(f64),
    Supplier(String),
    IsOrganic(bool),
    Notes(String),
}

fn demonstrate_vector_operations() {
    println!("🔄 Vector Operations with Heterogeneous Data");
    println!("{:=<50}", "");

    let mut inventory = Vec::new();

    // Adding different types of data
    println!("📥 Adding inventory data:");

    inventory.push(InventoryItem::Ingredient("Organic Tomatoes".to_string()));
    inventory.push(InventoryItem::Quantity(50));
    inventory.push(InventoryItem::ExpirationDate("2024-01-20".to_string()));
    inventory.push(InventoryItem::Price(4.99));
    inventory.push(InventoryItem::Supplier("Fresh Farms Co".to_string()));
    inventory.push(InventoryItem::IsOrganic(true));
    inventory.push(InventoryItem::Notes("Premium quality, locally sourced".to_string()));

    println!("   Added {} items to inventory", inventory.len());

    // Batch adding with extend
    let additional_data = vec![
        InventoryItem::Ingredient("Bell Peppers".to_string()),
        InventoryItem::Quantity(30),
        InventoryItem::Price(3.49),
        InventoryItem::IsOrganic(false),
    ];

    inventory.extend(additional_data);
    println!("   Extended with {} more items", 4);

    // Display current inventory
    println!("\n📦 Current inventory contents:");
    for (index, item) in inventory.iter().enumerate() {
        println!("   {}: {:?}", index + 1, item);
    }

    // Removing items
    println!("\n📤 Removing items:");

    if let Some(last_item) = inventory.pop() {
        println!("   Removed last item: {:?}", last_item);
    }

    // Remove specific item by pattern
    let removed_index = inventory.iter().position(|item| {
        matches!(item, InventoryItem::Notes(_))
    });

    if let Some(index) = removed_index {
        let removed_item = inventory.remove(index);
        println!("   Removed notes: {:?}", removed_item);
    }

    println!("\n📊 Final inventory: {} items", inventory.len());
}

fn main() {
    demonstrate_vector_operations();
}
```


### 2. Filtering and Searching

**Finding specific types of data in your heterogeneous collection:**[^1]

```rust
#[derive(Debug, Clone)]
enum CustomerData {
    Name(String),
    Age(u8),
    Email(String),
    OrderHistory(Vec<String>),
    LoyaltyPoints(u32),
    IsMember(bool),
    Preferences(Vec<String>),
    LastVisit(String),
}

fn demonstrate_filtering_and_searching() {
    println!("🔍 Filtering and Searching Heterogeneous Data");
    println!("{:=<50}", "");

    let customer_database = vec![
        CustomerData::Name("Alice Johnson".to_string()),
        CustomerData::Age(28),
        CustomerData::Email("alice@email.com".to_string()),
        CustomerData::OrderHistory(vec![
            "Quinoa Bowl".to_string(),
            "Green Smoothie".to_string(),
            "Avocado Toast".to_string(),
        ]),
        CustomerData::LoyaltyPoints(450),
        CustomerData::IsMember(true),
        CustomerData::Preferences(vec![
            "vegan".to_string(),
            "gluten-free".to_string(),
        ]),
        CustomerData::LastVisit("2024-01-15".to_string()),

        // Add another customer's data
        CustomerData::Name("Bob Smith".to_string()),
        CustomerData::Age(35),
        CustomerData::Email("bob@email.com".to_string()),
        CustomerData::LoyaltyPoints(120),
        CustomerData::IsMember(false),
    ];

    println!("📊 Customer Database Analysis:");

    // Filter by enum variant type
    println!("\n👤 Finding all customer names:");
    let names: Vec<&CustomerData> = customer_database
        .iter()
        .filter(|item| matches!(item, CustomerData::Name(_)))
        .collect();

    for name_data in names {
        if let CustomerData::Name(name) = name_data {
            println!("   • {}", name);
        }
    }

    // Filter by data content
    println!("\n🎯 Finding customers with high loyalty points:");
    let high_loyalty: Vec<&CustomerData> = customer_database
        .iter()
        .filter(|item| {
            if let CustomerData::LoyaltyPoints(points) = item {
                *points > 200
            } else {
                false
            }
        })
        .collect();

    for loyalty_data in high_loyalty {
        if let CustomerData::LoyaltyPoints(points) = loyalty_data {
            println!("   🏆 Customer with {} loyalty points", points);
        }
    }

    // Find specific data by pattern matching
    println!("\n📧 Finding email addresses:");
    for data in &customer_database {
        if let CustomerData::Email(email) = data {
            println!("   📬 {}", email);
        }
    }

    // Complex filtering with multiple conditions
    println!("\n🔍 Advanced search - members under 30:");
    let mut found_young_members = false;
    let mut temp_age = None;
    let mut temp_member_status = None;

    for data in &customer_database {
        match data {
            CustomerData::Age(age) => temp_age = Some(*age),
            CustomerData::IsMember(is_member) => temp_member_status = Some(*is_member),
            CustomerData::Name(name) => {
                // Check if we have age and membership data
                if let (Some(age), Some(is_member)) = (temp_age, temp_member_status) {
                    if age < 30 && is_member {
                        println!("   🌟 Young member found: {} (age {})", name, age);
                        found_young_members = true;
                    }
                }
                // Reset for next customer
                temp_age = None;
                temp_member_status = None;
            }
            _ => {}
        }
    }

    if !found_young_members {
        println!("   No young members found in this dataset");
    }

    // Count items by type
    println!("\n📈 Data type distribution:");
    let name_count = customer_database.iter().filter(|item| matches!(item, CustomerData::Name(_))).count();
    let age_count = customer_database.iter().filter(|item| matches!(item, CustomerData::Age(_))).count();
    let email_count = customer_database.iter().filter(|item| matches!(item, CustomerData::Email(_))).count();
    let points_count = customer_database.iter().filter(|item| matches!(item, CustomerData::LoyaltyPoints(_))).count();

    println!("   Names: {}", name_count);
    println!("   Ages: {}", age_count);
    println!("   Emails: {}", email_count);
    println!("   Loyalty Points: {}", points_count);
}

fn main() {
    demonstrate_filtering_and_searching();
}
```


### 3. Transforming and Processing Heterogeneous Data

**Converting and manipulating different types of data:**[^2]

```rust
#[derive(Debug, Clone)]
enum OrderProcessingData {
    RawOrder(String),           // Unparsed order string
    ParsedItems(Vec<String>),   // Extracted menu items
    Quantities(Vec<u32>),       // Item quantities
    UnitPrices(Vec<f64>),      // Price per item
    Subtotal(f64),             // Calculated subtotal
    Tax(f64),                  // Tax amount
    Tip(f64),                  // Tip amount
    Total(f64),                // Final total
    PaymentStatus(bool),       // Payment completed
}

fn demonstrate_data_transformation() {
    println!("⚗️ Transforming Heterogeneous Order Data");
    println!("{:=<45}", "");

    let mut order_pipeline = vec![
        OrderProcessingData::RawOrder("Quinoa Bowl x2, Green Smoothie x1, Avocado Toast x1".to_string()),
    ];

    println!("🔄 Order Processing Pipeline:");

    // Step 1: Parse the raw order
    if let Some(OrderProcessingData::RawOrder(raw)) = order_pipeline.get(0) {
        println!("   📝 Raw order: {}", raw);

        // Extract items and quantities
        let items = vec!["Quinoa Bowl", "Green Smoothie", "Avocado Toast"];
        let quantities = vec![2, 1, 1];

        order_pipeline.push(OrderProcessingData::ParsedItems(
            items.iter().map(|s| s.to_string()).collect()
        ));
        order_pipeline.push(OrderProcessingData::Quantities(quantities));

        println!("   ✅ Parsed into items and quantities");
    }

    // Step 2: Add pricing information
    let unit_prices = vec![15.99, 8.99, 12.99];
    order_pipeline.push(OrderProcessingData::UnitPrices(unit_prices.clone()));

    // Step 3: Calculate subtotal
    let subtotal = order_pipeline.iter()
        .find_map(|data| {
            if let OrderProcessingData::Quantities(qty) = data {
                Some(qty.iter().zip(unit_prices.iter())
                    .map(|(q, p)| (*q as f64) * p)
                    .sum())
            } else {
                None
            }
        })
        .unwrap_or(0.0);

    order_pipeline.push(OrderProcessingData::Subtotal(subtotal));

    // Step 4: Calculate tax and tip
    let tax = subtotal * 0.08; // 8% tax
    let tip = subtotal * 0.18; // 18% tip

    order_pipeline.push(OrderProcessingData::Tax(tax));
    order_pipeline.push(OrderProcessingData::Tip(tip));

    // Step 5: Calculate total
    let total = subtotal + tax + tip;
    order_pipeline.push(OrderProcessingData::Total(total));

    // Step 6: Set payment status
    order_pipeline.push(OrderProcessingData::PaymentStatus(true));

    println!("\n📊 Processing Results:");

    for data in &order_pipeline {
        match data {
            OrderProcessingData::RawOrder(raw) => {
                println!("   📝 Original: {}", raw);
            }
            OrderProcessingData::ParsedItems(items) => {
                println!("   🍽️ Items: {}", items.join(", "));
            }
            OrderProcessingData::Quantities(quantities) => {
                println!("   🔢 Quantities: {:?}", quantities);
            }
            OrderProcessingData::UnitPrices(prices) => {
                println!("   💵 Unit Prices: {:?}", prices);
            }
            OrderProcessingData::Subtotal(amount) => {
                println!("   💰 Subtotal: ${:.2}", amount);
            }
            OrderProcessingData::Tax(amount) => {
                println!("   🏛️ Tax (8%): ${:.2}", amount);
            }
            OrderProcessingData::Tip(amount) => {
                println!("   💸 Tip (18%): ${:.2}", amount);
            }
            OrderProcessingData::Total(amount) => {
                println!("   🎯 Total: ${:.2}", amount);
            }
            OrderProcessingData::PaymentStatus(paid) => {
                let status = if *paid { "✅ PAID" } else { "❌ PENDING" };
                println!("   💳 Payment: {}", status);
            }
        }
    }

    // Transform data based on business rules
    println!("\n🔄 Business Rule Transformations:");

    let transformed_data: Vec<String> = order_pipeline
        .iter()
        .filter_map(|data| {
            match data {
                OrderProcessingData::ParsedItems(items) => {
                    Some(format!("Receipt Items: {}", items.join(", ")))
                }
                OrderProcessingData::Total(amount) => {
                    if *amount > 50.0 {
                        Some(format!("🎁 Eligible for loyalty bonus! Total: ${:.2}", amount))
                    } else {
                        Some(format!("Order Total: ${:.2}", amount))
                    }
                }
                OrderProcessingData::PaymentStatus(true) => {
                    Some("✅ Payment Confirmed - Order Processing".to_string())
                }
                _ => None,
            }
        })
        .collect();

    for transformation in transformed_data {
        println!("   {}", transformation);
    }
}

fn main() {
    demonstrate_data_transformation();
}
```


## Advanced Patterns and Best Practices

### 1. Creating Type-Safe Heterogeneous Collections

**Building robust systems with proper error handling:**

```rust
#[derive(Debug, Clone)]
enum ConfigValue {
    String(String),
    Integer(i64),
    Float(f64),
    Boolean(bool),
    List(Vec<String>),
    Object(std::collections::HashMap<String, String>),
}

#[derive(Debug)]
enum ConfigError {
    TypeMismatch { expected: String, found: String },
    KeyNotFound(String),
    InvalidFormat(String),
}

struct RestaurantConfig {
    settings: Vec<(String, ConfigValue)>,
}

impl RestaurantConfig {
    fn new() -> Self {
        RestaurantConfig {
            settings: Vec::new(),
        }
    }

    fn add_setting(&mut self, key: String, value: ConfigValue) {
        self.settings.push((key, value));
    }

    fn get_string(&self, key: &str) -> Result<String, ConfigError> {
        self.find_setting(key)
            .and_then(|value| {
                if let ConfigValue::String(s) = value {
                    Ok(s.clone())
                } else {
                    Err(ConfigError::TypeMismatch {
                        expected: "String".to_string(),
                        found: format!("{:?}", value),
                    })
                }
            })
    }

    fn get_integer(&self, key: &str) -> Result<i64, ConfigError> {
        self.find_setting(key)
            .and_then(|value| {
                if let ConfigValue::Integer(i) = value {
                    Ok(*i)
                } else {
                    Err(ConfigError::TypeMismatch {
                        expected: "Integer".to_string(),
                        found: format!("{:?}", value),
                    })
                }
            })
    }

    fn get_boolean(&self, key: &str) -> Result<bool, ConfigError> {
        self.find_setting(key)
            .and_then(|value| {
                if let ConfigValue::Boolean(b) = value {
                    Ok(*b)
                } else {
                    Err(ConfigError::TypeMismatch {
                        expected: "Boolean".to_string(),
                        found: format!("{:?}", value),
                    })
                }
            })
    }

    fn find_setting(&self, key: &str) -> Result<&ConfigValue, ConfigError> {
        self.settings
            .iter()
            .find(|(k, _)| k == key)
            .map(|(_, v)| v)
            .ok_or_else(|| ConfigError::KeyNotFound(key.to_string()))
    }

    fn validate_all(&self) -> Vec<ConfigError> {
        let mut errors = Vec::new();

        // Check for required settings
        let required_keys = ["restaurant_name", "max_tables", "is_open"];

        for key in required_keys {
            if self.find_setting(key).is_err() {
                errors.push(ConfigError::KeyNotFound(key.to_string()));
            }
        }

        errors
    }
}

fn demonstrate_type_safe_collections() {
    println!("🔒 Type-Safe Heterogeneous Configuration System");
    println!("{:=<55}", "");

    let mut config = RestaurantConfig::new();

    // Add various configuration settings
    config.add_setting("restaurant_name".to_string(),
                      ConfigValue::String("Green Garden Bistro".to_string()));
    config.add_setting("max_tables".to_string(),
                      ConfigValue::Integer(25));
    config.add_setting("average_meal_price".to_string(),
                      ConfigValue::Float(18.99));
    config.add_setting("is_open".to_string(),
                      ConfigValue::Boolean(true));
    config.add_setting("cuisine_types".to_string(),
                      ConfigValue::List(vec![
                          "Mediterranean".to_string(),
                          "Modern Vegetarian".to_string(),
                          "Organic".to_string(),
                      ]));

    let mut contact_info = std::collections::HashMap::new();
    contact_info.insert("phone".to_string(), "+1-555-0123".to_string());
    contact_info.insert("email".to_string(), "info@greengarden.com".to_string());
    config.add_setting("contact_info".to_string(),
                      ConfigValue::Object(contact_info));

    println!("🔧 Configuration Settings:");

    // Safe access with proper error handling
    match config.get_string("restaurant_name") {
        Ok(name) => println!("   🏪 Restaurant: {}", name),
        Err(e) => println!("   ❌ Error getting name: {:?}", e),
    }

    match config.get_integer("max_tables") {
        Ok(tables) => println!("   🪑 Max Tables: {}", tables),
        Err(e) => println!("   ❌ Error getting tables: {:?}", e),
    }

    match config.get_boolean("is_open") {
        Ok(open) => {
            let status = if open { "🟢 OPEN" } else { "🔴 CLOSED" };
            println!("   📊 Status: {}", status);
        }
        Err(e) => println!("   ❌ Error getting status: {:?}", e),
    }

    // Demonstrate type safety
    println!("\n🛡️ Type Safety Demonstration:");

    match config.get_integer("restaurant_name") {
        Ok(value) => println!("   Unexpected success: {}", value),
        Err(ConfigError::TypeMismatch { expected, found }) => {
            println!("   ✅ Type safety works! Expected {}, found {}", expected, found);
        }
        Err(e) => println!("   Other error: {:?}", e),
    }

    // Validate configuration
    println!("\n✅ Configuration Validation:");
    let validation_errors = config.validate_all();

    if validation_errors.is_empty() {
        println!("   🎉 All required settings present!");
    } else {
        println!("   ⚠️ Validation errors:");
        for error in validation_errors {
            println!("     • {:?}", error);
        }
    }

    println!("\n📊 System Summary:");
    println!("   Total settings: {}", config.settings.len());
    println!("   ✅ Type-safe access to heterogeneous data");
    println!("   ✅ Proper error handling for mismatched types");
    println!("   ✅ Validation for required configuration");
}

fn main() {
    demonstrate_type_safe_collections();
}
```


### 2. Performance Optimization Patterns

**Optimizing vectors of enums for better performance:**

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
enum OptimizedData {
    // Small data variants (fit in enum discriminant + small data)
    SmallText([u8; 15]),        // Fixed-size array for short strings
    Counter(u32),
    Flag(bool),
    Rating(u8),                 // 0-255, perfect for ratings

    // Medium data variants
    StandardText(String),       // Only when text is longer
    FloatValue(f64),

    // Large data variants (use sparingly)
    LargeDataSet(Vec<f64>),
    ComplexObject(HashMap<String, String>),
}

impl OptimizedData {
    // Helper constructor for text that chooses optimal variant
    fn new_text(text: &str) -> Self {
        if text.len() <= 15 {
            let mut array = [0u8; 15];
            let bytes = text.as_bytes();
            array[..bytes.len()].copy_from_slice(bytes);
            OptimizedData::SmallText(array)
        } else {
            OptimizedData::StandardText(text.to_string())
        }
    }

    // Extract text regardless of variant
    fn as_text(&self) -> Option<String> {
        match self {
            OptimizedData::SmallText(array) => {
                // Find the null terminator or use full array
                let end = array.iter().position(|&b| b == 0).unwrap_or(15);
                let text = std::str::from_utf8(&array[..end]).ok()?;
                Some(text.to_string())
            }
            OptimizedData::StandardText(s) => Some(s.clone()),
            _ => None,
        }
    }
}

fn demonstrate_performance_optimization() {
    println!("⚡ Performance-Optimized Heterogeneous Data");
    println!("{:=<50}", "");

    let mut data_collection = Vec::with_capacity(1000); // Pre-allocate

    // Add various types of optimized data
    println!("📊 Memory optimization strategies:");

    // Use small variants when possible
    data_collection.push(OptimizedData::new_text("Pizza"));      // SmallText
    data_collection.push(OptimizedData::new_text("Quinoa"));     // SmallText
    data_collection.push(OptimizedData::new_text("This is a very long dish name that exceeds the limit")); // StandardText

    data_collection.push(OptimizedData::Counter(42));
    data_collection.push(OptimizedData::Flag(true));
    data_collection.push(OptimizedData::Rating(5)); // Rating out of 5
    data_collection.push(OptimizedData::FloatValue(15.99));

    // Analyze memory usage
    println!("   Enum size: {} bytes", std::mem::size_of::<OptimizedData>());
    println!("   Vector capacity: {} items", data_collection.capacity());
    println!("   Current items: {}", data_collection.len());
    println!("   Total allocated: {} bytes",
             data_collection.capacity() * std::mem::size_of::<OptimizedData>());

    // Demonstrate efficient iteration
    println!("\n🔄 Efficient Processing:");

    let start = std::time::Instant::now();

    let mut text_count = 0;
    let mut number_count = 0;
    let mut flag_count = 0;

    for item in &data_collection {
        match item {
            OptimizedData::SmallText(_) | OptimizedData::StandardText(_) => {
                text_count += 1;
            }
            OptimizedData::Counter(_) | OptimizedData::FloatValue(_) | OptimizedData::Rating(_) => {
                number_count += 1;
            }
            OptimizedData::Flag(_) => {
                flag_count += 1;
            }
            _ => {}
        }
    }

    let processing_time = start.elapsed();

    println!("   📝 Text items: {}", text_count);
    println!("   🔢 Numeric items: {}", number_count);
    println!("   🚩 Boolean items: {}", flag_count);
    println!("   ⏱️ Processing time: {:?}", processing_time);

    // Demonstrate batch operations
    println!("\n📦 Batch Operations:");

    // Extract all text values efficiently
    let text_values: Vec<String> = data_collection
        .iter()
        .filter_map(|item| item.as_text())
        .collect();

    println!("   Extracted {} text values:", text_values.len());
    for text in text_values {
        println!("     • {}", text);
    }

    // Memory comparison demonstration
    println!("\n💾 Memory Efficiency Comparison:");

    // Less efficient: storing everything as strings
    let inefficient_data: Vec<String> = vec![
        "Pizza".to_string(),
        "42".to_string(),
        "true".to_string(),
        "15.99".to_string(),
    ];

    let efficient_data: Vec<OptimizedData> = vec![
        OptimizedData::new_text("Pizza"),
        OptimizedData::Counter(42),
        OptimizedData::Flag(true),
        OptimizedData::FloatValue(15.99),
    ];

    println!("   Inefficient (all strings): {} bytes per item",
             std::mem::size_of::<String>());
    println!("   Efficient (optimized enum): {} bytes per item",
             std::mem::size_of::<OptimizedData>());

    let inefficient_total = inefficient_data.len() * std::mem::size_of::<String>() +
                           inefficient_data.iter().map(|s| s.capacity()).sum::<usize>();
    let efficient_total = efficient_data.len() * std::mem::size_of::<OptimizedData>();

    println!("   Total memory - Inefficient: ~{} bytes", inefficient_total);
    println!("   Total memory - Efficient: {} bytes", efficient_total);

    if inefficient_total > efficient_total {
        let savings = ((inefficient_total - efficient_total) as f64 / inefficient_total as f64) * 100.0;
        println!("   💰 Memory savings: {:.1}%", savings);
    }
}

fn main() {
    demonstrate_performance_optimization();
}
```


## Common Mistakes and How to Avoid Them

### Learning from Data Management Disasters

```rust
fn demonstrate_common_mistakes() {
    println!("⚠️ Common Mistakes with Heterogeneous Vectors");
    println!("{:=<50}", "");

    // Mistake 1: Forgetting that enum variants are NOT separate types
    println!("❌ MISTAKE 1: Treating enum variants as separate types");

    #[derive(Debug)]
    enum BadExample {
        Text(String),
        Number(i32),
    }

    let mixed_data = vec![
        BadExample::Text("Hello".to_string()),
        BadExample::Number(42),
    ];

    // ❌ This doesn't work - you can't filter by "type" directly
    // let strings_only: Vec<String> = mixed_data.into_iter()
    //     .filter(|item| matches!(item, BadExample::Text(_)))
    //     .collect(); // Error: trying to collect different types

    println!("   ✅ FIX: Use filter_map to extract specific types:");

    let mixed_data = vec![
        BadExample::Text("Hello".to_string()),
        BadExample::Number(42),
        BadExample::Text("World".to_string()),
    ];

    let strings_only: Vec<String> = mixed_data
        .into_iter()
        .filter_map(|item| {
            if let BadExample::Text(s) = item {
                Some(s)
            } else {
                None
            }
        })
        .collect();

    println!("     Extracted strings: {:?}", strings_only);

    // Mistake 2: Not handling all enum variants in pattern matching
    println!("\n❌ MISTAKE 2: Non-exhaustive pattern matching");

    #[derive(Debug)]
    enum ProcessingData {
        Input(String),
        Progress(f64),
        Complete(bool),
        Error(String),
    }

    let data = ProcessingData::Error("Something went wrong".to_string());

    // ❌ This is incomplete and won't compile in some contexts
    /*
    match data {
        ProcessingData::Input(s) => println!("Input: {}", s),
        ProcessingData::Progress(p) => println!("Progress: {}%", p * 100.0),
        // Missing Complete and Error cases!
    }
    */

    println!("   ✅ FIX: Handle all variants or use wildcard:");

    match data {
        ProcessingData::Input(s) => println!("     📝 Input: {}", s),
        ProcessingData::Progress(p) => println!("     📊 Progress: {:.1}%", p * 100.0),
        ProcessingData::Complete(done) => {
            let status = if done { "✅ DONE" } else { "⏳ PENDING" };
            println!("     🎯 Status: {}", status);
        }
        ProcessingData::Error(err) => println!("     ❌ Error: {}", err),
    }

    // Mistake 3: Using enums when simpler solutions exist
    println!("\n❌ MISTAKE 3: Over-engineering with enums");

    // If all your data is actually the same type, don't use enums
    #[derive(Debug)]
    enum OverEngineered {
        Name1(String),
        Name2(String),
        Name3(String),
        Name4(String),
    }

    println!("   ❌ BAD: Using enum for same-type variants");
    println!("   ✅ FIX: Just use Vec<String> if all data is strings");

    let simple_solution = vec![
        "Name 1".to_string(),
        "Name 2".to_string(),
        "Name 3".to_string(),
        "Name 4".to_string(),
    ];

    println!("     Simple vector: {:?}", simple_solution);

    // Mistake 4: Not considering memory layout
    println!("\n❌ MISTAKE 4: Inefficient enum design");

    #[derive(Debug)]
    enum InefficientEnum {
        SmallData(u8),           // 1 byte of data
        LargeData(Vec<String>),  // 24+ bytes of data
    }

    println!("   ⚠️ Problem: Enum size is determined by largest variant");
    println!("   Enum size: {} bytes", std::mem::size_of::<InefficientEnum>());
    println!("   Even SmallData(1) takes {} bytes!", std::mem::size_of::<InefficientEnum>());

    println!("   ✅ FIX: Consider boxing large variants:");

    #[derive(Debug)]
    enum EfficientEnum {
        SmallData(u8),
        LargeData(Box<Vec<String>>), // Boxed large data
    }

    println!("   Optimized enum size: {} bytes", std::mem::size_of::<EfficientEnum>());

    // Mistake 5: Not providing helper methods
    println!("\n❌ MISTAKE 5: No helper methods for common operations");

    #[derive(Debug)]
    enum ImprovedData {
        Text(String),
        Number(i32),
        Flag(bool),
    }

    impl ImprovedData {
        // Helper method to check type
        fn is_text(&self) -> bool {
            matches!(self, ImprovedData::Text(_))
        }

        // Helper method to extract value safely
        fn as_string(&self) -> Option<String> {
            match self {
                ImprovedData::Text(s) => Some(s.clone()),
                ImprovedData::Number(n) => Some(n.to_string()),
                ImprovedData::Flag(b) => Some(b.to_string()),
            }
        }

        // Helper method for type-safe conversion
        fn to_display_format(&self) -> String {
            match self {
                ImprovedData::Text(s) => format!("📝 {}", s),
                ImprovedData::Number(n) => format!("🔢 {}", n),
                ImprovedData::Flag(b) => format!("🚩 {}", b),
            }
        }
    }

    println!("   ✅ FIX: Add helpful methods to your enums:");

    let improved_data = vec![
        ImprovedData::Text("Hello".to_string()),
        ImprovedData::Number(42),
        ImprovedData::Flag(true),
    ];

    for item in improved_data {
        println!("     {}", item.to_display_format());

        if item.is_text() {
            println!("       This is text data!");
        }
    }
}

fn main() {
    demonstrate_common_mistakes();
}
```


## Summary and Key Takeaways

### **Mental Model: The Unified Restaurant Management System**

Remember our restaurant system analogy:

- 🎯 **Enums** = **Unified order ticket system** - Handle any type of order with one system
- 📦 **Vectors of enums** = **Organized filing system** - Store all different order types together
- 🔍 **Pattern matching** = **Processing protocols** - Handle each order type appropriately
- 🛡️ **Type safety** = **Quality control** - Ensure all order types are handled correctly
- ⚡ **Performance** = **Efficient operations** - Fast access to any type of data


### **Essential Principles**

1. **Type unification** - Enums make different types look the same to vectors
2. **Explicit handling** - Pattern matching forces you to handle all cases
3. **Memory efficiency** - All enum variants take the same space
4. **Safe extraction** - Use pattern matching to safely get data out
5. **Flexible design** - Add new data types by adding enum variants

### **Quick Usage Reference**

```rust
// Creating heterogeneous vector
#[derive(Debug)]
enum MyData {
    Text(String),
    Number(i32),
    Flag(bool),
    List(Vec<String>),
}

let mixed_data = vec![
    MyData::Text("Hello".to_string()),
    MyData::Number(42),
    MyData::Flag(true),
    MyData::List(vec!["item1".to_string(), "item2".to_string()]),
];

// Accessing data safely
for item in &mixed_data {
    match item {
        MyData::Text(s) => println!("Text: {}", s),
        MyData::Number(n) => println!("Number: {}", n),
        MyData::Flag(b) => println!("Flag: {}", b),
        MyData::List(v) => println!("List: {:?}", v),
    }
}

// Filtering specific types
let numbers_only: Vec<i32> = mixed_data
    .iter()
    .filter_map(|item| {
        if let MyData::Number(n) = item {
            Some(*n)
        } else {
            None
        }
    })
    .collect();
```


### **Best Practices Checklist**

**✅ DO:**

- Use enums to unify different types in vectors
- Handle all enum variants in pattern matching
- Provide helper methods for common operations
- Consider memory layout when designing large enums
- Use `Box<T>` for large enum variants when appropriate
- Validate data when converting between types

**❌ DON'T:**

- Use enums when all data is actually the same type
- Ignore non-exhaustive pattern matching warnings
- Try to treat enum variants as separate types
- Create enums with many similar variants
- Forget to handle the `None` case in pattern matching
- Use unwrap() without checking enum variants first


### **Real-World Applications**

**Vectors of enums are perfect for:**

- 🍽️ **Configuration systems** - Different types of settings in one collection
- 📊 **Data processing pipelines** - Mixed data flowing through processing stages
- 🎮 **Game development** - Different types of entities in game world
- 📱 **User interfaces** - Different types of UI elements in lists
- 🌐 **API responses** - Variable response data from external services
- 📈 **Analytics data** - Different metrics and measurements in one dataset


### **The Professional Advantage**

**Vectors of enums are like having a master filing system** for your vegetarian restaurant that can handle any type of document - orders, invoices, recipes, customer feedback, staff schedules - all organized in one unified system:

- 🎯 **Unified interface** - Handle all data types with the same vector operations[^1]
- 🛡️ **Type safety** - Rust ensures you handle every possible data type
- ⚡ **Performance** - Efficient memory layout with predictable access patterns
- 📋 **Extensibility** - Add new data types by adding enum variants
- 🔍 **Clear organization** - Each piece of data has a clear category and type

**Mastering vectors of enums transforms your ability to handle complex, mixed data safely and efficiently.** Just as a professional restaurant can handle any type of order through one streamlined system, vectors of enums let you build robust Rust programs that gracefully manage diverse data types while maintaining the safety and performance that makes Rust exceptional!

1. https://doc.rust-lang.org/book/ch08-01-vectors.html
2. https://dev.to/fadygrab/learning-rust-16-collections-vectors-43nm
3. https://www.reddit.com/r/rust/comments/11t5zxg/how_to_make_a_vector_of_enum_variants/
4. https://doc.rust-lang.org/rust-by-example/custom_types/enum.html
5. https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html
6. https://stackoverflow.com/questions/62714678/how-to-convert-a-vector-of-enums-into-a-vector-of-inner-values-of-a-specific-var
7. https://stackoverflow.com/questions/68427115/how-to-generate-a-vec-of-enum-options-in-rust
8. https://bandarra.me/posts/Heterogeneous-collections-in-Rust
9. https://users.rust-lang.org/t/vec-enum-to-vec-my-type/88735
10. https://rust-book.cs.brown.edu/ch08-01-vectors.html
11. https://www.reddit.com/r/rust/comments/1df4zgy/what_am_i_missing_handling_heterogeneous_data_in/
12. https://www.youtube.com/watch?v=f4a8rWkSl8U
13. https://users.rust-lang.org/t/how-to-create-a-heterogenous-collection-without-heap-allocation/55721
14. https://www.programiz.com/rust/enum
15. https://stackoverflow.com/questions/53382985/how-is-memory-allocation-for-vectors-efficient-when-storing-multiple-types-using
16. https://stackoverflow.com/questions/27957103/how-do-i-create-a-heterogeneous-collection-of-objects
17. https://www.youtube.com/watch?v=XCUOvO1sqIE
18. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch08-01-vectors.html
19. https://users.rust-lang.org/t/need-heterogeneous-list-thats-not-statically-typed/68961
20. https://users.rust-lang.org/t/vec-with-different-types/123535
21. https://users.rust-lang.org/t/a-vec-of-enum-s-representing-struct-s-how-to-add-and-access-the-elements/11638
22. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/enums.html
23. https://www.w3schools.com/rust/rust_enums.php
24. https://github.com/Badel2/enum_vec
25. https://users.rust-lang.org/t/check-if-a-vector-of-enums-contains-any-enum-of-a-type-regardless-the-value/32859
26. https://stackoverflow.com/questions/52344295/how-to-implement-an-enum-containing-a-cow-of-a-vector-of-itself
27. https://www.reddit.com/r/rust/comments/x875v9/use_slices_instead_of_vectors_in_enums_when_enum/
28. https://doc.rust-lang.org/book/ch18-02-trait-objects.html
29. https://www.risein.com/courses/rust-programming/rust-programming-enum
30. https://users.rust-lang.org/t/constant-heterogeneous-vector/76124
31. https://users.rust-lang.org/t/vector-with-generic-types-heterogeneous-container/46644
32. https://updraft.cyfrin.io/courses/rust-programming-basics/data-types/enum
33. https://doc.rust-lang.org/reference/items/enumerations.html
34. https://www.geeksforgeeks.org/rust/rust-vectors/
35. https://stackoverflow.com/questions/40411045/is-it-possible-to-have-a-heterogeneous-vector-of-types-that-implement-eq
36. https://www.reddit.com/r/rust/comments/v8zcu0/how_to_handle_enum_variants_that_have_different/
37. https://users.rust-lang.org/t/typesafe-heterogeneous-vec-container/18003
38. https://stackoverflow.com/questions/72438594/how-can-i-use-enum-variants-as-generic-type
39. https://awjunaid.com/rust/using-heterogeneous-data-structures-in-rust/
40. https://users.rust-lang.org/t/extracting-a-value-from-an-enum-variant/116677
41. https://frontendmasters.com/courses/typescript-go-rust/enums/
42. https://stackoverflow.com/questions/67714697/rust-enums-how-to-get-data-value-from-mixed-type-enum-in-rust
43. https://users.rust-lang.org/t/mixed-types-in-vec-t/34153
44. https://stackoverflow.com/questions/74527327/cleanly-iterating-over-a-vector-of-mixed-enum-variants
