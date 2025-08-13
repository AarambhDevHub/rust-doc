+++
title = "BTreeMap Introduction"
description = "Learn about the BTreeMap collection in Rust, how it stores sorted key-value pairs, and when to use it over HashMap."
date = 2025-08-13
weight = 5
keywords = ["Rust", "BTreeMap", "collections", "key-value", "sorted map", "binary tree"]
+++

# BTreeMap Introduction in Rust: Comprehensive Documentation for Beginners

Understanding BTreeMap in Rust is like learning to **manage a sophisticated vegetarian restaurant's reservation system that automatically keeps all bookings in perfect chronological order** - you need a data structure that not only stores customer information efficiently but also maintains everything in sorted order for easy browsing, range queries, and systematic processing. Just as a professional restaurant keeps reservations organized by time so staff can see what's coming next and plan accordingly, Rust's BTreeMap maintains your key-value pairs in sorted order, enabling powerful operations like "find all orders between 6 PM and 8 PM" or "get the next three reservations after this time."

## The Organized Restaurant Reservation System Analogy 📅

### Imagine You're Managing a High-End Vegetarian Restaurant's Perfect Reservation System

**The Problem with Unorganized Storage:**

```rust
// ❌ HashMap - like keeping reservations in random order
use std::collections::HashMap;

let mut reservations = HashMap::new();
reservations.insert("19:30", "Johnson Anniversary Dinner");
reservations.insert("18:00", "Smith Business Meeting");
reservations.insert("20:15", "Davis Family Celebration");

// Problem: When you iterate, you get random order!
for (time, details) in &reservations {
    println!("{} → {}", time, details); // Random order: 20:15, 18:00, 19:30
}
```

**The BTreeMap Solution - Perfect Organization:**

```rust
// ✅ BTreeMap - like a perfectly organized reservation book
use std::collections::BTreeMap;

let mut reservations = BTreeMap::new();
reservations.insert("19:30", "Johnson Anniversary Dinner");
reservations.insert("18:00", "Smith Business Meeting");
reservations.insert("20:15", "Davis Family Celebration");

// Magic: Always get reservations in time order!
for (time, details) in &reservations {
    println!("{} → {}", time, details); // Perfect order: 18:00, 19:30, 20:15
}
```

**Why BTreeMap is Superior for Ordered Data:**

- 📅 **Automatic sorting** - Keys are always in order, no manual sorting needed
- 🔍 **Range queries** - Find all reservations between two times instantly
- 📊 **Predictable iteration** - Always iterate in sorted order
- ⚡ **Efficient operations** - O(log n) for insert, search, and delete
- 🎯 **Perfect for time-series data** - Timestamps, dates, sequential IDs


## Understanding BTreeMap Fundamentals

### What is a BTreeMap?

A **BTreeMap** is Rust's ordered key-value collection that uses a B-tree data structure:

```rust
fn demonstrate_btreemap_basics() {
    println!("🌳 BTreeMap Fundamentals - Organized Restaurant Management");
    println!("{:=<60}", "");

    use std::collections::BTreeMap;

    // Think of BTreeMap as a sophisticated filing system that keeps
    // everything automatically sorted by key

    let mut daily_schedule: BTreeMap<String, String> = BTreeMap::new();

    println!("📊 BTreeMap Characteristics:");
    println!("   • Ordered storage: Keys are kept in sorted order");
    println!("   • Tree structure: Uses B-tree for efficient operations");
    println!("   • O(log n) operations: Insert, search, delete all O(log n)");
    println!("   • Range queries: Can efficiently query ranges of keys");
    println!("   • Deterministic: Always iterates in the same order");

    // Add restaurant events (notice we add them out of order)
    daily_schedule.insert("09:00".to_string(), "Staff meeting".to_string());
    daily_schedule.insert("14:30".to_string(), "Lunch rush ends".to_string());
    daily_schedule.insert("11:00".to_string(), "Restaurant opens".to_string());
    daily_schedule.insert("18:00".to_string(), "Dinner service begins".to_string());
    daily_schedule.insert("22:00".to_string(), "Kitchen closes".to_string());

    println!("\n📅 Daily Schedule (automatically sorted):");
    for (time, event) in &daily_schedule {
        println!("   {} → {}", time, event);
    }

    println!("\n🎯 Key Benefits Demonstrated:");
    println!("   ✅ Added events in random order");
    println!("   ✅ BTreeMap automatically sorted them by time");
    println!("   ✅ Perfect for chronological data!");
}

fn main() {
    demonstrate_btreemap_basics();
}
```


### How BTreeMap Works Internally

**The B-tree structure explained:**

```rust
use std::collections::BTreeMap;
use ordered_float::OrderedFloat;

fn demonstrate_btree_concept() {
    println!("🌳 How BTreeMap Works - The Tree Structure");
    println!("{:=<55}", "");

    // BTreeMap organizes data like a family tree with special rules:
    // 1. Keys in each node are sorted
    // 2. All data flows from root to leaves in order
    // 3. Tree stays balanced for consistent performance

    let mut restaurant_ratings: BTreeMap<OrderedFloat<f64>, &str> = BTreeMap::new();

    // Add restaurant ratings (quality scores)
    restaurant_ratings.insert(OrderedFloat(4.2), "Good Italian Place");
    restaurant_ratings.insert(OrderedFloat(4.8), "Amazing Vegan Bistro");
    restaurant_ratings.insert(OrderedFloat(3.9), "Decent Coffee Shop");
    restaurant_ratings.insert(OrderedFloat(4.5), "Great Thai Restaurant");
    restaurant_ratings.insert(OrderedFloat(4.9), "Exceptional French Cuisine");
    restaurant_ratings.insert(OrderedFloat(4.1), "Nice Breakfast Spot");

    println!("🏆 Restaurant Ratings (automatically sorted by score):");
    for (rating, name) in &restaurant_ratings {
        let stars = "⭐".repeat(rating.0.floor() as usize);
        println!("   {:.1} {} → {}", rating.0, stars, name);
    }

    // Conceptual tree structure visualization
    println!("\n🌳 Conceptual Tree Structure:");
    println!("   Each 'branch' keeps keys sorted");
    println!("   Tree automatically balances itself");
    println!("   All operations follow tree paths = O(log n)");

    println!("\n📐 Visual Tree Concept:");
    println!("            [4.5]");
    println!("           /     \\");
    println!("      [4.1,4.2]  [4.8,4.9]");
    println!("      /   |   |    |   |   \\");
    println!("    3.9  4.1 4.2  4.8 4.9  ...");
    println!("   (Each level keeps everything sorted)");

    // Demonstrate tree properties
    println!("\n⚡ Tree Properties in Action:");

    // Find all highly-rated restaurants (>= 4.5)
    let high_rated: Vec<_> = restaurant_ratings
        .range(OrderedFloat(4.5)..)
        .collect();

    println!("   🌟 High-rated restaurants (>= 4.5 stars):");
    for (rating, name) in high_rated {
        println!("     {:.1}⭐ {}", rating.0, name);
    }

    // Find restaurants in specific range
    let good_range: Vec<_> = restaurant_ratings
        .range(OrderedFloat(4.0)..OrderedFloat(4.5))
        .collect();

    println!("   👍 Good restaurants (4.0-4.5 stars):");
    for (rating, name) in good_range {
        println!("     {:.1}⭐ {}", rating.0, name);
    }
}

fn main() {
    demonstrate_btree_concept();
}
```


## Core BTreeMap Operations - Restaurant Management

### 1. Creating and Populating BTreeMaps

**Different ways to create and fill your organized restaurant data:**

```rust
fn demonstrate_btreemap_creation() {
    println!("🏗️ BTreeMap Creation - Setting Up Restaurant Systems");
    println!("{:=<55}", "");

    use std::collections::BTreeMap;

    // Method 1: Empty BTreeMap with new()
    println!("📋 Method 1: Empty Reservation System");
    let mut reservations: BTreeMap<String, String> = BTreeMap::new();
    println!("   ✅ Created empty reservation system");

    // Method 2: From array data (Rust 1.56+)
    println!("\n🍽️ Method 2: Menu Pricing from Array");
    let menu_prices = BTreeMap::from([
        ("Appetizer", 8.99),
        ("Main Course", 15.99),
        ("Dessert", 6.99),
        ("Beverage", 3.99),
    ]);

    println!("   📊 Menu prices in alphabetical order:");
    for (category, price) in &menu_prices {
        println!("     {} → ${:.2}", category, price);
    }

    // Method 3: From iterator (converting from other collections)
    println!("\n📅 Method 3: Time Slots from Vector");
    let time_slots = vec![
        ("20:00", "Late dinner"),
        ("18:00", "Early dinner"),
        ("19:00", "Prime time"),
        ("21:00", "Last seating"),
    ];

    let organized_slots: BTreeMap<&str, &str> = time_slots
        .into_iter()
        .collect();

    println!("   ⏰ Time slots in chronological order:");
    for (time, description) in &organized_slots {
        println!("     {} → {}", time, description);
    }

    // Method 4: Building with insert operations
    println!("\n🎯 Method 4: Customer Loyalty Tiers");
    let mut loyalty_tiers = BTreeMap::new();

    // Insert in random order - BTreeMap will sort automatically
    loyalty_tiers.insert(1000, "Platinum Member");
    loyalty_tiers.insert(100, "Bronze Member");
    loyalty_tiers.insert(500, "Gold Member");
    loyalty_tiers.insert(250, "Silver Member");

    println!("   🏆 Loyalty tiers by points required:");
    for (points, tier) in &loyalty_tiers {
        println!("     {} points → {}", points, tier);
    }

    // Method 5: Complex structured data
    println!("\n🏢 Method 5: Comprehensive Restaurant Analytics");

    #[derive(Debug)]
    struct DailyMetrics {
        revenue: f64,
        customer_count: u32,
        avg_satisfaction: f32,
    }

    let mut daily_analytics = BTreeMap::new();

    daily_analytics.insert("2024-01-10", DailyMetrics {
        revenue: 2847.50,
        customer_count: 89,
        avg_satisfaction: 4.6,
    });

    daily_analytics.insert("2024-01-08", DailyMetrics {
        revenue: 2234.75,
        customer_count: 67,
        avg_satisfaction: 4.4,
    });

    daily_analytics.insert("2024-01-09", DailyMetrics {
        revenue: 2653.20,
        customer_count: 78,
        avg_satisfaction: 4.7,
    });

    println!("   📈 Daily analytics in chronological order:");
    for (date, metrics) in &daily_analytics {
        println!("     {} → ${:.2} revenue, {} customers, {:.1}⭐",
                 date, metrics.revenue, metrics.customer_count, metrics.avg_satisfaction);
    }

    println!("\n✨ Creation Methods Summary:");
    println!("   ✅ BTreeMap::new() - Start empty");
    println!("   ✅ BTreeMap::from([...]) - From array data");
    println!("   ✅ collect() from iterator - Convert collections");
    println!("   ✅ Manual insert() - Build step by step");
    println!("   ✅ All methods automatically maintain sorted order!");
}

fn main() {
    demonstrate_btreemap_creation();
}
```


### 2. Accessing and Querying Data

**Efficient data retrieval in your organized system:**

```rust
fn demonstrate_btreemap_access() {
    println!("🔍 BTreeMap Data Access - Restaurant Information Queries");
    println!("{:=<60}", "");

    use std::collections::BTreeMap;

    // Setup restaurant event schedule
    let mut event_schedule = BTreeMap::new();
    event_schedule.insert("08:00", "Staff arrival");
    event_schedule.insert("10:00", "Prep kitchen");
    event_schedule.insert("11:30", "Restaurant opens");
    event_schedule.insert("12:00", "Lunch service");
    event_schedule.insert("14:30", "Lunch service ends");
    event_schedule.insert("17:00", "Dinner prep");
    event_schedule.insert("18:00", "Dinner service");
    event_schedule.insert("22:00", "Last orders");
    event_schedule.insert("23:00", "Restaurant closes");

    // Method 1: get() - Safe single item access
    println!("🎯 Method 1: Single Event Lookup (get)");

    let check_times = ["12:00", "15:00", "18:00"];
    for time in check_times {
        match event_schedule.get(time) {
            Some(event) => println!("   {} → ✅ {}", time, event),
            None => println!("   {} → ❌ No scheduled event", time),
        }
    }

    // Method 2: contains_key() - Check existence
    println!("\n❓ Method 2: Event Existence Check");

    let important_times = ["11:30", "16:00", "22:00"];
    for time in important_times {
        if event_schedule.contains_key(time) {
            println!("   {} → ✅ Event scheduled", time);
        } else {
            println!("   {} → ⏰ Free time slot", time);
        }
    }

    // Method 3: first_key_value() and last_key_value() - Boundaries
    println!("\n📊 Method 3: Day Boundaries");

    if let Some((first_time, first_event)) = event_schedule.first_key_value() {
        println!("   🌅 Day starts: {} with '{}'", first_time, first_event);
    }

    if let Some((last_time, last_event)) = event_schedule.last_key_value() {
        println!("   🌙 Day ends: {} with '{}'", last_time, last_event);
    }

    // Method 4: range() - The killer feature!
    println!("\n🔍 Method 4: Range Queries (BTreeMap's Superpower!)");

    // Find all events during lunch period
    println!("   🥗 Lunch period events (12:00 - 15:00):");
    for (time, event) in event_schedule.range("12:00".."15:00") {
        println!("     {} → {}", time, event);
    }

    // Find all events after 18:00
    println!("   🌆 Evening events (18:00 onwards):");
    for (time, event) in event_schedule.range("18:00"..) {
        println!("     {} → {}", time, event);
    }

    // Find all events before noon
    println!("   🌅 Morning events (before 12:00):");
    for (time, event) in event_schedule.range(.."12:00") {
        println!("     {} → {}", time, event);
    }

    // Method 5: Advanced range queries with complex data
    println!("\n📈 Method 5: Advanced Range Queries");

    let mut order_queue = BTreeMap::new();
    order_queue.insert(1, "Order #001 - Quinoa Bowl");
    order_queue.insert(3, "Order #003 - Veggie Burger");
    order_queue.insert(5, "Order #005 - Lentil Soup");
    order_queue.insert(7, "Order #007 - Mediterranean Wrap");
    order_queue.insert(9, "Order #009 - Green Salad");
    order_queue.insert(12, "Order #012 - Pasta Primavera");

    // Process next 3 orders (range query + take)
    println!("   👨‍🍳 Next 3 orders to prepare:");
    for (order_num, details) in order_queue.range(1..).take(3) {
        println!("     Order {} → {}", order_num, details);
    }

    // Find orders in middle range
    println!("   📋 Orders 5-10:");
    for (order_num, details) in order_queue.range(5..=10) {
        println!("     Order {} → {}", order_num, details);
    }

    // Method 6: Getting adjacent items
    println!("\n🔗 Method 6: Finding Adjacent Events");

    let current_time = "16:00";

    // Find next event after current time
    if let Some((next_time, next_event)) = event_schedule.range(current_time..).next() {
        println!("   ⏭️ Next event after {}: {} → {}", current_time, next_time, next_event);
    }

    // Find previous event before current time
    if let Some((prev_time, prev_event)) = event_schedule.range(..current_time).last() {
        println!("   ⏮️ Previous event before {}: {} → {}", current_time, prev_time, prev_event);
    }

    println!("\n🎯 BTreeMap Access Superpowers:");
    println!("   ✅ O(log n) single item access");
    println!("   ✅ Efficient range queries (impossible with HashMap!)");
    println!("   ✅ Easy boundary access (first/last)");
    println!("   ✅ Natural adjacent item finding");
    println!("   ✅ Perfect for time-series and ordered data");
}

fn main() {
    demonstrate_btreemap_access();
}
```


### 3. Modification and Updates

**Safely updating your organized restaurant data:**

```rust
fn demonstrate_btreemap_modification() {
    println!("✏️ BTreeMap Modifications - Restaurant System Updates");
    println!("{:=<55}", "");

    use std::collections::BTreeMap;

    // Setup table capacity management
    let mut table_capacity = BTreeMap::new();
    table_capacity.insert(2, 8);   // 8 tables for 2 people
    table_capacity.insert(4, 12);  // 12 tables for 4 people
    table_capacity.insert(6, 6);   // 6 tables for 6 people
    table_capacity.insert(8, 3);   // 3 tables for 8 people
    table_capacity.insert(10, 2);  // 2 tables for 10 people

    println!("📊 Initial Table Capacity:");
    for (size, count) in &table_capacity {
        println!("   {}-person tables: {} available", size, count);
    }

    // Method 1: insert() - Add or update
    println!("\n🔄 Method 1: Table Updates (insert)");

    // Add new table size
    table_capacity.insert(3, 4);  // 4 tables for 3 people
    println!("   ➕ Added 3-person tables: 4 available");

    // Update existing table count
    let old_count = table_capacity.insert(4, 15);  // Update 4-person tables
    if let Some(previous) = old_count {
        println!("   🔄 Updated 4-person tables: {} → 15", previous);
    }

    // Method 2: get_mut() - Modify existing values
    println!("\n✏️ Method 2: In-Place Modifications (get_mut)");

    // Restaurant gets busy - some tables become unavailable
    if let Some(count) = table_capacity.get_mut(&6) {
        *count -= 2;  // 2 tables are now occupied
        println!("   📉 6-person tables: 2 tables now occupied, {} available", count);
    }

    if let Some(count) = table_capacity.get_mut(&8) {
        *count -= 1;  // 1 large table occupied
        println!("   📉 8-person tables: 1 table occupied, {} available", count);
    }

    // Method 3: remove() - Remove entries
    println!("\n🗑️ Method 3: Removing Table Types (remove)");

    // Remove 10-person tables (converted to storage)
    if let Some(removed_count) = table_capacity.remove(&10) {
        println!("   ➖ Removed 10-person tables: {} tables converted to storage", removed_count);
    }

    // Method 4: Entry API for conditional operations
    println!("\n🎯 Method 4: Smart Table Management (Entry API)");

    // Add tables or update existing counts
    *table_capacity.entry(12).or_insert(0) += 1;  // Add 1 private dining room
    println!("   🆕 Added private dining room (12-person): 1 available");

    // Conditional update based on current value
    table_capacity.entry(2)
        .and_modify(|count| *count += 3)  // Add 3 more 2-person tables
        .or_insert(3);  // Or start with 3 if none existed

    println!("   ➕ Added 3 more 2-person tables");

    // Method 5: Bulk modifications
    println!("\n📦 Method 5: Bulk Table Adjustments");

    // Weekend rush - temporarily reduce available tables
    let weekend_reservations = [(2, 2), (4, 4), (6, 1)];  // (table_size, reserved_count)

    for (table_size, reserved) in weekend_reservations {
        if let Some(available) = table_capacity.get_mut(&table_size) {
            *available = available.saturating_sub(reserved);
            println!("   📅 Weekend: {}-person tables, {} reserved, {} available",
                     table_size, reserved, available);
        }
    }

    // Method 6: Complex data structure updates
    println!("\n🏢 Method 6: Complex Restaurant Analytics Updates");

    #[derive(Debug)]
    struct RestaurantStats {
        daily_revenue: f64,
        customer_count: u32,
        popular_dish: String,
        avg_wait_time: u32, // minutes
    }

    let mut weekly_stats = BTreeMap::new();

    // Add initial data
    weekly_stats.insert("Monday", RestaurantStats {
        daily_revenue: 2450.75,
        customer_count: 87,
        popular_dish: "Quinoa Bowl".to_string(),
        avg_wait_time: 15,
    });

    weekly_stats.insert("Tuesday", RestaurantStats {
        daily_revenue: 2156.40,
        customer_count: 76,
        popular_dish: "Veggie Burger".to_string(),
        avg_wait_time: 12,
    });

    // Update Tuesday's stats (end of day correction)
    if let Some(tuesday_stats) = weekly_stats.get_mut("Tuesday") {
        tuesday_stats.daily_revenue += 234.50;  // Late orders came in
        tuesday_stats.customer_count += 8;       // Additional customers
        tuesday_stats.avg_wait_time = 10;        // Improved efficiency

        println!("   📊 Tuesday stats updated: ${:.2} revenue, {} customers",
                 tuesday_stats.daily_revenue, tuesday_stats.customer_count);
    }

    // Add Wednesday data
    weekly_stats.insert("Wednesday", RestaurantStats {
        daily_revenue: 2789.60,
        customer_count: 94,
        popular_dish: "Mediterranean Wrap".to_string(),
        avg_wait_time: 8,
    });

    println!("\n📈 Current Table Availability (sorted by table size):");
    for (size, count) in &table_capacity {
        let status = if *count > 5 {
            "✅ Good availability"
        } else if *count > 0 {
            "⚠️ Limited availability"
        } else {
            "❌ Fully booked"
        };

        println!("   {}-person tables: {} available {}", size, count, status);
    }

    println!("\n📊 Weekly Performance (chronological order):");
    for (day, stats) in &weekly_stats {
        println!("   {} → ${:.2}, {} customers, avg wait: {}min",
                 day, stats.daily_revenue, stats.customer_count, stats.avg_wait_time);
    }
}

fn main() {
    demonstrate_btreemap_modification();
}
```


## BTreeMap vs HashMap - When to Choose What

### Understanding the Trade-offs

```rust
fn demonstrate_btreemap_vs_hashmap() {
    println!("⚖️ BTreeMap vs HashMap - Choosing the Right Tool");
    println!("{:=<55}", "");

    use std::collections::{BTreeMap, HashMap};
    use std::time::Instant;

    // Scenario comparison
    println!("🎯 Scenario-Based Comparison:");

    // Scenario 1: Random access performance
    println!("\n⚡ Scenario 1: Random Access Performance");

    let mut hashmap = HashMap::new();
    let mut btreemap = BTreeMap::new();

    // Add the same data to both
    for i in 0..1000 {
        let key = format!("customer_{:04}", i);
        let value = format!("Customer data {}", i);
        hashmap.insert(key.clone(), value.clone());
        btreemap.insert(key, value);
    }

    // Test random access
    let test_key = "customer_0500";

    let start = Instant::now();
    let _result = hashmap.get(test_key);
    let hashmap_time = start.elapsed();

    let start = Instant::now();
    let _result = btreemap.get(test_key);
    let btreemap_time = start.elapsed();

    println!("   HashMap lookup: {:?}", hashmap_time);
    println!("   BTreeMap lookup: {:?}", btreemap_time);
    println!("   💡 HashMap is typically faster for single lookups");

    // Scenario 2: Ordered iteration
    println!("\n📊 Scenario 2: Ordered Data Processing");

    let mut order_times_hash = HashMap::new();
    let mut order_times_btree = BTreeMap::new();

    let times = ["14:30", "12:15", "18:45", "11:00", "19:30"];
    for time in times {
        order_times_hash.insert(time, format!("Order at {}", time));
        order_times_btree.insert(time, format!("Order at {}", time));
    }

    println!("   HashMap iteration (random order):");
    for (time, order) in order_times_hash.iter().take(3) {
        println!("     {} → {}", time, order);
    }

    println!("   BTreeMap iteration (sorted order):");
    for (time, order) in order_times_btree.iter().take(3) {
        println!("     {} → {}", time, order);
    }

    println!("   ✅ BTreeMap provides guaranteed order");

    // Scenario 3: Range queries
    println!("\n🔍 Scenario 3: Range Queries");

    let mut reservation_times = BTreeMap::new();
    reservation_times.insert("17:00", "Early dinner");
    reservation_times.insert("18:30", "Prime time");
    reservation_times.insert("19:15", "Peak hours");
    reservation_times.insert("20:45", "Late dinner");
    reservation_times.insert("21:30", "Last seating");

    // Range query: evening prime time (18:00 - 20:00)
    println!("   Prime time reservations (18:00-20:00):");
    for (time, description) in reservation_times.range("18:00".."20:00") {
        println!("     {} → {}", time, description);
    }

    println!("   💡 HashMap cannot do range queries efficiently!");

    // Decision matrix
    println!("\n🎯 Decision Matrix:");

    println!("\n✅ Use HashMap when:");
    println!("   • You need fastest possible single-item access");
    println!("   • Order doesn't matter");
    println!("   • You're doing lots of lookups/inserts");
    println!("   • Memory usage is a concern (slightly less overhead)");

    println!("\n✅ Use BTreeMap when:");
    println!("   • You need data in sorted order");
    println!("   • You need range queries");
    println!("   • You want deterministic iteration order");
    println!("   • You're working with time-series data");
    println!("   • You need to find min/max keys efficiently");

    println!("\n📊 Performance Characteristics:");
    println!("┌─────────────┬──────────────┬──────────────┐");
    println!("│ Operation   │ HashMap      │ BTreeMap     │");
    println!("├─────────────┼──────────────┼──────────────┤");
    println!("│ Insert      │ O(1) avg     │ O(log n)     │");
    println!("│ Lookup      │ O(1) avg     │ O(log n)     │");
    println!("│ Delete      │ O(1) avg     │ O(log n)     │");
    println!("│ Iteration   │ O(n) random  │ O(n) sorted  │");
    println!("│ Range Query │ Not possible │ O(log n + k) │");
    println!("│ Min/Max     │ O(n)         │ O(log n)     │");
    println!("└─────────────┴──────────────┴──────────────┘");

    println!("\n🎯 Real-World Guidelines:");
    println!("   🏪 Customer database → HashMap (fast lookups)");
    println!("   📅 Reservation system → BTreeMap (chronological order)");
    println!("   💰 Price catalog → HashMap (quick price checks)");
    println!("   📊 Daily reports → BTreeMap (date-ordered)");
    println!("   🎮 Game leaderboard → BTreeMap (score-ordered)");
    println!("   ⚙️ Configuration settings → HashMap (key-value access)");
}

fn main() {
    demonstrate_btreemap_vs_hashmap();
}
```


## Advanced BTreeMap Techniques

### Powerful Operations for Restaurant Management

```rust
fn demonstrate_advanced_btreemap() {
    println!("🚀 Advanced BTreeMap Techniques - Pro Restaurant Management");
    println!("{:=<65}", "");

    use std::collections::BTreeMap;

    // Advanced Technique 1: Time-based analytics with ranges
    println!("📊 Technique 1: Time-Based Analytics");

    let mut hourly_sales = BTreeMap::new();

    // Sales data throughout the day (hour -> revenue)
    hourly_sales.insert(11, 145.50);  // 11 AM
    hourly_sales.insert(12, 267.75);  // 12 PM
    hourly_sales.insert(13, 298.25);  // 1 PM
    hourly_sales.insert(14, 189.50);  // 2 PM
    hourly_sales.insert(17, 234.75);  // 5 PM
    hourly_sales.insert(18, 456.25);  // 6 PM
    hourly_sales.insert(19, 523.80);  // 7 PM
    hourly_sales.insert(20, 387.40);  // 8 PM
    hourly_sales.insert(21, 245.30);  // 9 PM

    // Analyze lunch period (11 AM - 3 PM)
    let lunch_revenue: f64 = hourly_sales
        .range(11..=14)
        .map(|(_, revenue)| revenue)
        .sum();

    println!("   🥗 Lunch period revenue (11 AM - 2 PM): ${:.2}", lunch_revenue);

    // Analyze dinner period (5 PM - 9 PM)
    let dinner_revenue: f64 = hourly_sales
        .range(17..=21)
        .map(|(_, revenue)| revenue)
        .sum();

    println!("   🍽️ Dinner period revenue (5 PM - 9 PM): ${:.2}", dinner_revenue);

    // Find peak hour
    if let Some((peak_hour, peak_revenue)) = hourly_sales
        .iter()
        .max_by(|(_, a), (_, b)| a.partial_cmp(b).unwrap()) {

        let time_display = if *peak_hour <= 12 {
            format!("{} AM", peak_hour)
        } else {
            format!("{} PM", peak_hour - 12)
        };

        println!("   🏆 Peak hour: {} with ${:.2} revenue", time_display, peak_revenue);
    }

    // Advanced Technique 2: Sliding window analysis
    println!("\n📈 Technique 2: Sliding Window Analysis");

    // Calculate 3-hour rolling averages
    let mut rolling_averages = BTreeMap::new();

    for (&hour, &revenue) in &hourly_sales {
        // Get 3-hour window centered on current hour
        let window_start = hour.saturating_sub(1);
        let window_end = hour + 2;

        let window_sales: Vec<f64> = hourly_sales
            .range(window_start..window_end)
            .map(|(_, revenue)| *revenue)
            .collect();

        if window_sales.len() >= 2 {  // Need at least 2 data points
            let average = window_sales.iter().sum::<f64>() / window_sales.len() as f64;
            rolling_averages.insert(hour, average);
        }
    }

    println!("   📊 3-hour rolling averages:");
    for (hour, avg) in rolling_averages {
        let time_display = if hour <= 12 {
            format!("{} AM", hour)
        } else {
            format!("{} PM", hour - 12)
        };
        println!("     {} → ${:.2} average", time_display, avg);
    }

    // Advanced Technique 3: Capacity planning with ranges
    println!("\n🎯 Technique 3: Smart Capacity Planning");

    let mut table_bookings = BTreeMap::new();

    // Time slots with booking counts (30-minute intervals)
    table_bookings.insert("17:00", 12);  // 12 tables booked
    table_bookings.insert("17:30", 18);
    table_bookings.insert("18:00", 25);  // Peak starts
    table_bookings.insert("18:30", 28);
    table_bookings.insert("19:00", 30);  // Peak
    table_bookings.insert("19:30", 27);
    table_bookings.insert("20:00", 22);
    table_bookings.insert("20:30", 15);
    table_bookings.insert("21:00", 8);

    let total_tables = 35;

    // Find peak period (when capacity > 80%)
    println!("   🚨 High-capacity periods (>80% full):");
    for (time, bookings) in &table_bookings {
        let capacity_percent = (*bookings as f64 / total_tables as f64) * 100.0;
        if capacity_percent > 80.0 {
            println!("     {} → {} tables ({:.1}% capacity)",
                     time, bookings, capacity_percent);
        }
    }

    // Find available time slots
    println!("   ✅ Available time slots (<70% capacity):");
    for (time, bookings) in &table_bookings {
        let capacity_percent = (*bookings as f64 / total_tables as f64) * 100.0;
        if capacity_percent < 70.0 {
            let available = total_tables - bookings;
            println!("     {} → {} tables available ({:.1}% capacity)",
                     time, available, capacity_percent);
        }
    }

    // Advanced Technique 4: Hierarchical data organization
    println!("\n🏗️ Technique 4: Hierarchical Menu Organization");

    // Use composite keys for hierarchical data
    let mut menu_hierarchy = BTreeMap::new();

    // Key format: "category:subcategory:item"
    menu_hierarchy.insert("1:Appetizers:Hummus Platter".to_string(), 8.99);
    menu_hierarchy.insert("1:Appetizers:Stuffed Mushrooms".to_string(), 9.99);
    menu_hierarchy.insert("2:Main:Quinoa Bowl".to_string(), 15.99);
    menu_hierarchy.insert("2:Main:Veggie Burger".to_string(), 12.99);
    menu_hierarchy.insert("2:Main:Pasta Primavera".to_string(), 14.99);
    menu_hierarchy.insert("3:Desserts:Chocolate Mousse".to_string(), 6.99);
    menu_hierarchy.insert("3:Desserts:Fruit Tart".to_string(), 5.99);

    // Display menu by category (automatic grouping due to sorted keys)
    println!("   📋 Hierarchical menu display:");
    let mut current_category = "";

    for (key, price) in &menu_hierarchy {
        let parts: Vec<&str> = key.split(':').collect();
        if parts.len() >= 3 {
            let category = parts[^1];
            let item = parts[^2];

            if category != current_category {
                println!("     📂 {}", category);
                current_category = category;
            }

            println!("       • {} → ${:.2}", item, price);
        }
    }

    // Advanced Technique 5: Range-based inventory management
    println!("\n📦 Technique 5: Range-Based Inventory Alerts");

    let mut inventory_levels = BTreeMap::new();

    // Item ID ranges: 1000-1999 = Vegetables, 2000-2999 = Grains, etc.
    inventory_levels.insert(1001, ("Tomatoes", 45));
    inventory_levels.insert(1002, ("Lettuce", 23));
    inventory_levels.insert(1003, ("Onions", 67));
    inventory_levels.insert(2001, ("Quinoa", 89));
    inventory_levels.insert(2002, ("Brown Rice", 34));
    inventory_levels.insert(3001, ("Olive Oil", 12));
    inventory_levels.insert(3002, ("Vinegar", 8));

    // Check vegetable inventory (1000-1999)
    println!("   🥬 Vegetable inventory status:");
    for (id, (name, quantity)) in inventory_levels.range(1000..2000) {
        let status = if *quantity < 30 {
            "🚨 LOW"
        } else if *quantity < 50 {
            "⚠️ MEDIUM"
        } else {
            "✅ GOOD"
        };

        println!("     {} ({}) → {} units {}", name, id, quantity, status);
    }

    // Check condiment inventory (3000-3999)
    println!("   🧂 Condiment inventory status:");
    for (id, (name, quantity)) in inventory_levels.range(3000..4000) {
        let status = if *quantity < 15 {
            "🚨 LOW"
        } else {
            "✅ GOOD"
        };

        println!("     {} ({}) → {} units {}", name, id, quantity, status);
    }

    println!("\n🎓 Advanced BTreeMap Benefits:");
    println!("   ✅ Time-series analysis with range queries");
    println!("   ✅ Rolling window calculations");
    println!("   ✅ Capacity planning with sorted data");
    println!("   ✅ Hierarchical organization with composite keys");
    println!("   ✅ Range-based categorization and filtering");
    println!("   ✅ Natural ordering enables sophisticated algorithms");
}

fn main() {
    demonstrate_advanced_btreemap();
}
```


## Best Practices and Common Patterns

### Professional BTreeMap Usage

```rust
fn demonstrate_btreemap_best_practices() {
    println!("✅ BTreeMap Best Practices - Professional Restaurant Management");
    println!("{:=<70}", "");

    use std::collections::BTreeMap;

    // Best Practice 1: Choose appropriate key types
    println!("🔑 Best Practice 1: Key Type Selection");

    // ✅ Good: Use types that have natural ordering
    let mut daily_revenue: BTreeMap<String, f64> = BTreeMap::new();  // Dates as strings
    daily_revenue.insert("2024-01-15".to_string(), 2847.50);
    daily_revenue.insert("2024-01-14".to_string(), 2234.75);
    daily_revenue.insert("2024-01-16".to_string(), 3102.25);

    println!("   ✅ Date strings sort chronologically:");
    for (date, revenue) in daily_revenue.iter().take(3) {
        println!("     {} → ${:.2}", date, revenue);
    }

    // ✅ Better: Use types designed for ordering
    use std::time::SystemTime;

    let mut time_based_events: BTreeMap<u64, String> = BTreeMap::new();  // Unix timestamps
    time_based_events.insert(1705123200, "Morning prep".to_string());   // 09:00
    time_based_events.insert(1705129200, "Lunch service".to_string());  // 11:30
    time_based_events.insert(1705143600, "Dinner service".to_string()); // 18:00

    println!("   ✅ Timestamps provide precise ordering");

    // Best Practice 2: Efficient range operations
    println!("\n📊 Best Practice 2: Efficient Range Operations");

    let mut order_priorities = BTreeMap::new();

    // Use range-friendly keys for efficient queries
    order_priorities.insert(1, "Low priority");
    order_priorities.insert(5, "Normal priority");
    order_priorities.insert(8, "High priority");
    order_priorities.insert(9, "Urgent");
    order_priorities.insert(10, "Emergency");

    // Efficient high-priority filtering
    let high_priority_orders: Vec<_> = order_priorities
        .range(8..)  // O(log n + k) where k is result size
        .collect();

    println!("   ⚡ High priority orders (efficient range query):");
    for (priority, description) in high_priority_orders {
        println!("     Priority {}: {}", priority, description);
    }

    // Best Practice 3: Memory-efficient operations
    println!("\n💾 Best Practice 3: Memory Efficiency");

    // ✅ Use references when possible to avoid cloning
    let menu_items = BTreeMap::from([
        ("Appetizer", vec!["Hummus", "Bruschetta", "Salad"]),
        ("Main", vec!["Quinoa Bowl", "Burger", "Pasta"]),
        ("Dessert", vec!["Mousse", "Tart", "Ice Cream"]),
    ]);

    // Efficient iteration without cloning
    for (category, items) in &menu_items {
        println!("   📋 {}: {} items", category, items.len());
        for item in items {
            println!("     • {}", item);
        }
    }

    // Best Practice 4: Error handling patterns
    println!("\n🚨 Best Practice 4: Robust Error Handling");

    let mut table_reservations = BTreeMap::new();
    table_reservations.insert("18:00", "Johnson party of 4");
    table_reservations.insert("19:30", "Smith anniversary dinner");
    table_reservations.insert("20:15", "Davis business meeting");

    // Safe reservation lookup
    fn find_reservation(
        reservations: &BTreeMap<&str, &str>,
        time: &str
    ) -> Result<&str, String> {
        reservations.get(time)
            .ok_or_else(|| format!("No reservation found for {}", time))
    }

    // Safe range queries with validation
    fn get_evening_reservations(
        reservations: &BTreeMap<&str, &str>
    ) -> Vec<(&str, &str)> {
        reservations
            .range("18:00".."22:00")
            .map(|(&time, &details)| (time, details))
            .collect()
    }

    match find_reservation(&table_reservations, "19:30") {
        Ok(details) => println!("   ✅ Found reservation: {}", details),
        Err(error) => println!("   ❌ {}", error),
    }

    let evening_reservations = get_evening_reservations(&table_reservations);
    println!("   📅 Evening reservations: {} found", evening_reservations.len());

    // Best Practice 5: Performance optimization patterns
    println!("\n⚡ Best Practice 5: Performance Optimization");

    // Pre-allocate when you know the approximate size (not available for BTreeMap directly)
    // But you can batch insert for better performance
    let mut large_dataset = BTreeMap::new();

    // Batch insert sorted data for optimal performance
    let sorted_data = [
        (1, "First"),
        (2, "Second"),
        (3, "Third"),
        (4, "Fourth"),
        (5, "Fifth"),
    ];

    // ✅ Inserting pre-sorted data is more efficient
    for (key, value) in sorted_data {
        large_dataset.insert(key, value);
    }

    println!("   ✅ Inserted {} items efficiently", large_dataset.len());

    // Best Practice 6: Type-safe key design
    println!("\n🔒 Best Practice 6: Type-Safe Key Design");

    // Define custom key types for better type safety
    #[derive(Debug, PartialEq, Eq, PartialOrd, Ord)]
    struct TimeSlot {
        hour: u8,
        minute: u8,
    }

    impl TimeSlot {
        fn new(hour: u8, minute: u8) -> Result<Self, String> {
            if hour > 23 {
                return Err("Hour must be 0-23".to_string());
            }
            if minute > 59 {
                return Err("Minute must be 0-59".to_string());
            }
            Ok(TimeSlot { hour, minute })
        }
    }

    impl std::fmt::Display for TimeSlot {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            write!(f, "{:02}:{:02}", self.hour, self.minute)
        }
    }

    let mut schedule = BTreeMap::new();

    if let Ok(time1) = TimeSlot::new(18, 0) {
        schedule.insert(time1, "Dinner service starts");
    }

    if let Ok(time2) = TimeSlot::new(19, 30) {
        schedule.insert(time2, "Peak dinner time");
    }

    if let Ok(time3) = TimeSlot::new(21, 0) {
        schedule.insert(time3, "Last orders");
    }

    println!("   ✅ Type-safe time-based schedule:");
    for (time, event) in &schedule {
        println!("     {} → {}", time, event);
    }

    println!("\n📋 BTreeMap Best Practices Summary:");
    println!("   ✅ Choose keys with natural, meaningful ordering");
    println!("   ✅ Use range queries instead of filtering when possible");
    println!("   ✅ Prefer references over cloning for read operations");
    println!("   ✅ Implement proper error handling for missing keys");
    println!("   ✅ Insert pre-sorted data when possible for performance");
    println!("   ✅ Use custom key types for better type safety");
    println!("   ✅ Consider BTreeMap for any ordered, searchable data");

    println!("\n🎯 When to Use BTreeMap:");
    println!("   📅 Time-series data (logs, events, schedules)");
    println!("   📊 Ordered statistics (rankings, scores, metrics)");
    println!("   🔍 Range-based queries (time periods, score ranges)");
    println!("   📋 Hierarchical data (categories, levels, priorities)");
    println!("   🎮 Leaderboards and competitive rankings");
    println!("   📈 Financial data (prices, transactions, reports)");
}

fn main() {
    demonstrate_btreemap_best_practices();
}
```


## Summary and Key Takeaways

### **Mental Model: The Perfect Restaurant Reservation System**

Remember our restaurant reservation system analogy:

- 🌳 **BTreeMap** = **Automatic chronological organizer** - Keeps everything perfectly sorted
- 📅 **Ordered storage** = **Time-based reservation book** - Always see what's coming next
- 🔍 **Range queries** = **Time period analysis** - "Show me all 6-8 PM reservations"
- 📊 **Sorted iteration** = **Systematic processing** - Handle events in logical order
- ⚡ **O(log n) operations** = **Efficient tree navigation** - Fast even with thousands of entries


### **Essential BTreeMap Operations Quick Reference**

| **Operation** | **Syntax** | **Use Case** | **Performance** |
| :-- | :-- | :-- | :-- |
| **Create** | `BTreeMap::new()` | Empty ordered map | O(1) |
| **Insert** | `map.insert(key, value)` | Add/update entry | O(log n) |
| **Get** | `map.get(&key)` | Safe access | O(log n) |
| **Range** | `map.range(start..end)` | Query ranges | O(log n + k) |
| **First/Last** | `map.first_key_value()` | Boundary access | O(log n) |
| **Remove** | `map.remove(&key)` | Delete entry | O(log n) |

### **BTreeMap's Killer Features**

```rust
use std::collections::BTreeMap;

// Automatic ordering
let mut events = BTreeMap::new();
events.insert("19:30", "Dinner");
events.insert("11:00", "Lunch");
events.insert("08:00", "Breakfast");
// Always iterates: 08:00, 11:00, 19:30

// Range queries (impossible with HashMap!)
for (time, event) in events.range("12:00".."18:00") {
    println!("{}: {}", time, event); // Only afternoon events
}

// Boundary access
let (first_time, _) = events.first_key_value().unwrap();
let (last_time, _) = events.last_key_value().unwrap();
```


### **Decision Guide: BTreeMap vs HashMap**

**✅ Choose BTreeMap when:**

- You need data in sorted order
- You want range queries ("all items between X and Y")
- You're working with time-series data
- You need deterministic iteration order
- You want to find min/max efficiently
- You're building leaderboards or rankings

**✅ Choose HashMap when:**

- You need fastest possible lookups
- Order doesn't matter
- You're doing lots of random access
- Memory efficiency is critical


### **Common BTreeMap Patterns**

**Time-Series Analysis:**

```rust
let mut daily_sales = BTreeMap::new();
daily_sales.insert("2024-01-15", 2847.50);
daily_sales.insert("2024-01-14", 2234.75);

// Get week's data
let week_total: f64 = daily_sales
    .range("2024-01-10".."2024-01-17")
    .map(|(_, sales)| sales)
    .sum();
```

**Priority Queues:**

```rust
let mut task_priority = BTreeMap::new();
task_priority.insert(1, "Low priority task");
task_priority.insert(9, "High priority task");

// Process highest priority first
while let Some((priority, task)) = task_priority.pop_last() {
    println!("Processing priority {}: {}", priority, task);
}
```

**Hierarchical Data:**

```rust
let mut menu = BTreeMap::new();
menu.insert("1:Appetizers:Hummus", 8.99);
menu.insert("2:Mains:Quinoa Bowl", 15.99);
menu.insert("3:Desserts:Mousse", 6.99);
// Automatic grouping by category prefix
```


### **Performance Characteristics**

- **Insert/Delete/Search:** O(log n) - Consistently fast even with millions of items
- **Iteration:** O(n) - Always in sorted order
- **Range queries:** O(log n + k) - where k is the number of results
- **Memory:** Slightly higher overhead than HashMap, but reasonable
- **Cache performance:** Good locality for range operations


### **Best Practices Checklist**

**✅ DO:**

- Use BTreeMap for any naturally ordered data
- Leverage range queries instead of filtering
- Use custom key types for type safety
- Handle `Option` returns from `get()` properly
- Take advantage of first/last methods for boundaries
- Use `&` for read-only access to avoid cloning

**❌ DON'T:**

- Use BTreeMap if you only need random access (use HashMap)
- Ignore the ordering - if you don't need it, don't pay for it
- Use complex keys without considering performance
- Forget that keys must implement `Ord` trait
- Use unwrap() on get() without checking


### **Real-World Applications**

**BTreeMap excels for:**

- 📅 **Calendar and scheduling systems** - Natural time ordering
- 📊 **Analytics and reporting** - Ordered metrics and time-series
- 🎮 **Game leaderboards** - Score-based rankings
- 💰 **Financial applications** - Transaction histories, price data
- 📈 **Monitoring systems** - Time-stamped logs and metrics
- 🏪 **Inventory management** - Product codes, priority levels


### **The Professional Advantage**

**Mastering BTreeMap in Rust is like having a world-class restaurant reservation system** that automatically organizes everything perfectly:

- 📅 **Natural organization** - Your data stays perfectly sorted without effort
- 🔍 **Powerful queries** - Find exactly what you need with range operations
- ⚡ **Predictable performance** - O(log n) operations scale beautifully
- 🎯 **Type safety** - Rust ensures your ordering logic is correct
- 📊 **Iterator efficiency** - Sorted iteration enables advanced algorithms

**Understanding BTreeMap transforms you from a programmer who struggles with ordered data to a systems expert** who chooses the perfect data structure for each use case. Just as a master restaurateur uses sophisticated reservation systems to handle complex scheduling efficiently, a skilled Rust programmer leverages BTreeMap's ordered nature to build elegant solutions for time-series data, rankings, analytics, and any scenario where order matters.

This knowledge of BTreeMap, combined with understanding when to choose it over HashMap, makes you a more versatile and effective Rust programmer who can tackle complex data organization challenges with confidence and efficiency![^1][^2][^3]

1. https://doc.rust-lang.org/std/collections/struct.BTreeMap.html
2. https://www.linkedin.com/pulse/hashmap-btreemap-rust-collections-gabriel-puiggrós
3. https://www.w3resource.com/rust-tutorial/rust-maps-hashmap-btreemap.php
4. https://faultlore.com/blah/rust-btree-case/
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/std/collections/struct.BTreeMap.html
6. https://docs.rs/btree-slab/latest/btree_slab/type.BTreeMap.html
7. https://stackoverflow.com/questions/58357421/difference-between-iterating-over-btreemap-and-btreemap
8. https://www.reddit.com/r/rust/comments/zsws3k/introducing_rangeboundsmap_a_btreemaplike/
9. https://www.youtube.com/watch?v=DBdbe2QUlf8
10. https://doc.rust-lang.org/std/collections/index.html
11. https://stackoverflow.com/questions/75566040/iterating-through-btreemap-using-range-question-on-the-type-being-iterated-o
12. https://users.rust-lang.org/t/interface-the-btreemap-and-hashmap/97937
13. https://rust-exercises.com/100-exercises/06_ticket_management/16_btreemap.html
14. https://cglab.ca/~abeinges/blah/rust-btree-case/
15. https://www.reddit.com/r/rust/comments/xbkuc7/btreemap_vs_hashmap/
16. https://docs.rs/ic-stable-structures/latest/ic_stable_structures/btreemap/struct.BTreeMap.html
17. https://www.dotnetperls.com/btreemap-rust
18. https://www.thecodedmessage.com/posts/prefix-ranges/
19. https://docs.rs/ex3-ic-stable-structures/latest/ex3_ic_stable_structures/btreemap/struct.BTreeMap.html
20. https://www.thecodedmessage.com/rust-c-book/entries.html
21. https://stackoverflow.com/questions/76825499/interface-the-btreemap-and-hashmap-in-rust
22. https://users.rust-lang.org/t/how-to-construct-a-btreemap-of-fixed-values-in-rust/115576
23. https://stackoverflow.com/questions/70635142/btreemap-of-f64s
24. https://stackoverflow.com/questions/74473058/how-to-use-common-btreemap-variable-in-rustsingle-thread
25. https://users.rust-lang.org/t/iterating-over-my-btreemap/108764
