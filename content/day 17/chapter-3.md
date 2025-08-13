+++
title = "HashMap Methods and Iteration"
description = "Learn how to use common HashMap methods in Rust and iterate over keys, values, and key-value pairs efficiently."
date = 2025-08-13
weight = 3
keywords = ["Rust", "HashMap", "methods", "iteration", "keys", "values", "key-value pairs"]
+++

# HashMap Methods and Iteration in Rust: Comprehensive Documentation for Beginners

Understanding HashMap methods and iteration in Rust is like learning to **operate a sophisticated vegetarian restaurant's comprehensive management system** - you need to master all the tools for storing customer information, processing orders, managing inventory, and analyzing data efficiently. Just as a professional restaurant manager must know when to add new customer records, update existing information, search through databases, and generate reports, a skilled Rust programmer must understand the rich ecosystem of HashMap methods and iteration patterns to build robust, efficient applications.

## The Complete Restaurant Management System Analogy 🏪

### Imagine You're Operating a Full-Service Vegetarian Restaurant Management Platform

**Basic Operations (Core Methods):**

```rust
let mut restaurant_system = HashMap::new();

// 📥 ADDING - New customer registration
restaurant_system.insert("alice@email.com", "VIP Customer");

// 🔍 SEARCHING - Find customer information
if let Some(info) = restaurant_system.get("alice@email.com") {
    println!("Customer found: {}", info);
}

// 🗑️ REMOVING - Customer cancellation
restaurant_system.remove("alice@email.com");

// ❓ CHECKING - Verify customer exists
if restaurant_system.contains_key("bob@email.com") {
    println!("Bob is in our system");
}
```

**Advanced Operations (Iteration \& Analysis):**

```rust
// 📊 ANALYTICS - Review all customers
for (email, status) in restaurant_system.iter() {
    println!("Customer: {} → Status: {}", email, status);
}

// 🔄 UPDATES - Batch modify customer data
for (_, status) in restaurant_system.iter_mut() {
    *status = format!("Updated: {}", status);
}

// 📋 REPORTING - Extract specific information
let all_emails: Vec<String> = restaurant_system.keys().cloned().collect();
let all_statuses: Vec<String> = restaurant_system.values().cloned().collect();
```

**Why This Comprehensive Approach Works:**

- 🎯 **Complete coverage** - Handle every restaurant operation efficiently
- 📊 **Data insights** - Analyze customer patterns and trends
- 🔄 **Flexible processing** - Transform data in multiple ways
- ⚡ **Performance optimization** - Choose the right method for each task
- 🛡️ **Safe operations** - Handle missing data gracefully


## Core HashMap Methods - Essential Restaurant Operations

### 1. Data Insertion and Modification Methods

**Managing your restaurant's customer and order data:**

```rust
fn demonstrate_insertion_methods() {
    println!("📥 Restaurant Data Management - Core Insertion Methods");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    let mut customer_database: HashMap<String, CustomerInfo> = HashMap::new();

    #[derive(Debug, Clone)]
    struct CustomerInfo {
        name: String,
        email: String,
        visit_count: u32,
        total_spent: f64,
        preferences: Vec<String>,
    }

    // Method 1: insert() - Add or replace data
    println!("🏪 Method 1: Basic Customer Registration (insert)");

    let alice_info = CustomerInfo {
        name: "Alice Johnson".to_string(),
        email: "alice@email.com".to_string(),
        visit_count: 1,
        total_spent: 25.99,
        preferences: vec!["Vegan".to_string(), "Gluten-Free".to_string()],
    };

    // Insert returns Option<V> - None if key was new, Some(old_value) if key existed
    match customer_database.insert("CUST_001".to_string(), alice_info) {
        None => println!("   ✅ New customer Alice registered successfully"),
        Some(old_info) => println!("   🔄 Updated existing customer: {:?}", old_info),
    }

    // Method 2: insert() for updates
    println!("\n🔄 Method 2: Customer Information Updates");

    if let Some(alice) = customer_database.get_mut("CUST_001") {
        alice.visit_count += 1;
        alice.total_spent += 18.50;
        alice.preferences.push("Organic".to_string());
        println!("   ✅ Alice updated: {} visits, ${:.2} total",
                 alice.visit_count, alice.total_spent);
    }

    // Method 3: try_insert() - Insert only if key doesn't exist (unstable feature)
    println!("\n🎯 Method 3: Conditional Registration");

    // Using entry API as stable alternative to try_insert
    let bob_info = CustomerInfo {
        name: "Bob Smith".to_string(),
        email: "bob@email.com".to_string(),
        visit_count: 1,
        total_spent: 15.75,
        preferences: vec!["Vegetarian".to_string()],
    };

    match customer_database.entry("CUST_002".to_string()) {
        std::collections::hash_map::Entry::Vacant(entry) => {
            entry.insert(bob_info);
            println!("   ✅ Bob registered as new customer");
        }
        std::collections::hash_map::Entry::Occupied(_) => {
            println!("   ℹ️ Customer CUST_002 already exists - no update");
        }
    }

    // Method 4: Bulk insertion with extend()
    println!("\n📦 Method 4: Bulk Customer Import");

    let new_customers = vec![
        ("CUST_003".to_string(), CustomerInfo {
            name: "Carol Davis".to_string(),
            email: "carol@email.com".to_string(),
            visit_count: 3,
            total_spent: 67.25,
            preferences: vec!["Vegan".to_string(), "Nut-Free".to_string()],
        }),
        ("CUST_004".to_string(), CustomerInfo {
            name: "Diana Wilson".to_string(),
            email: "diana@email.com".to_string(),
            visit_count: 2,
            total_spent: 42.80,
            preferences: vec!["Gluten-Free".to_string()],
        }),
    ];

    customer_database.extend(new_customers);
    println!("   ✅ Bulk imported {} customers", 2);

    // Method 5: Entry API for complex conditional logic
    println!("\n⚗️ Method 5: Advanced Entry API Operations");

    // Update existing customer or create with default values
    let updated_customer = customer_database
        .entry("CUST_005".to_string())
        .or_insert_with(|| {
            println!("   🆕 Creating new customer CUST_005 with defaults");
            CustomerInfo {
                name: "New Customer".to_string(),
                email: "unknown@email.com".to_string(),
                visit_count: 0,
                total_spent: 0.0,
                preferences: vec!["No preferences set".to_string()],
            }
        });

    // Update the customer we just inserted or retrieved
    updated_customer.name = "Emma Thompson".to_string();
    updated_customer.email = "emma@email.com".to_string();
    updated_customer.visit_count = 1;
    updated_customer.total_spent = 22.50;

    println!("   ✅ Customer CUST_005 processed: {}", updated_customer.name);

    // Display final database state
    println!("\n📊 Final Customer Database Summary:");
    println!("   Total customers: {}", customer_database.len());

    for (id, info) in customer_database.iter().take(3) {
        println!("   {} → {} ({} visits, ${:.2})",
                 id, info.name, info.visit_count, info.total_spent);
    }
}

fn main() {
    demonstrate_insertion_methods();
}
```


### 2. Data Retrieval and Query Methods

**Accessing your restaurant's information efficiently:**

```rust
fn demonstrate_retrieval_methods() {
    println!("🔍 Restaurant Information Retrieval - Query Methods");
    println!("{:=<55}", "");

    use std::collections::HashMap;

    // Setup restaurant menu with pricing and details
    let mut menu_database = HashMap::new();

    #[derive(Debug, Clone)]
    struct MenuItem {
        name: String,
        price: f64,
        category: String,
        ingredients: Vec<String>,
        available: bool,
        rating: f32,
    }

    // Populate menu database
    menu_database.insert("ITEM_001", MenuItem {
        name: "Quinoa Buddha Bowl".to_string(),
        price: 15.99,
        category: "Bowls".to_string(),
        ingredients: vec!["Quinoa".to_string(), "Avocado".to_string(), "Vegetables".to_string()],
        available: true,
        rating: 4.8,
    });

    menu_database.insert("ITEM_002", MenuItem {
        name: "Mediterranean Wrap".to_string(),
        price: 12.49,
        category: "Wraps".to_string(),
        ingredients: vec!["Hummus".to_string(), "Vegetables".to_string(), "Pita".to_string()],
        available: true,
        rating: 4.5,
    });

    menu_database.insert("ITEM_003", MenuItem {
        name: "Lentil Curry".to_string(),
        price: 13.75,
        category: "Curries".to_string(),
        ingredients: vec!["Lentils".to_string(), "Coconut Milk".to_string(), "Spices".to_string()],
        available: false,
        rating: 4.9,
    });

    // Method 1: get() - Safe single item retrieval
    println!("🎯 Method 1: Safe Menu Item Lookup (get)");

    let items_to_check = ["ITEM_001", "ITEM_999", "ITEM_002"];

    for item_id in items_to_check {
        match menu_database.get(item_id) {
            Some(item) => {
                let status = if item.available { "✅ Available" } else { "❌ Unavailable" };
                println!("   {} → {} - ${:.2} {}", item_id, item.name, item.price, status);
            }
            None => {
                println!("   {} → ❌ Item not found in menu", item_id);
            }
        }
    }

    // Method 2: contains_key() - Existence checking
    println!("\n❓ Method 2: Menu Item Availability Check (contains_key)");

    let customer_requests = ["ITEM_001", "ITEM_004", "ITEM_003"];

    for request in customer_requests {
        if menu_database.contains_key(request) {
            println!("   {} → ✅ We have this item", request);
        } else {
            println!("   {} → ❌ Not on our menu", request);
        }
    }

    // Method 3: get_key_value() - Retrieve both key and value
    println!("\n🔗 Method 3: Complete Item Information (get_key_value)");

    if let Some((key, item)) = menu_database.get_key_value("ITEM_001") {
        println!("   Key: {} → Item: {} (Rating: {:.1}⭐)",
                 key, item.name, item.rating);
        println!("   Ingredients: {}", item.ingredients.join(", "));
    }

    // Method 4: Multiple safe retrievals
    println!("\n📋 Method 4: Order Processing - Multiple Item Lookup");

    let customer_order = ["ITEM_001", "ITEM_002", "ITEM_003"];
    let mut order_total = 0.0;
    let mut available_items = Vec::new();
    let mut unavailable_items = Vec::new();

    for item_id in customer_order {
        match menu_database.get(item_id) {
            Some(item) if item.available => {
                order_total += item.price;
                available_items.push((item_id, &item.name, item.price));
                println!("   ✅ {} - {} (${:.2})", item_id, item.name, item.price);
            }
            Some(item) => {
                unavailable_items.push((item_id, &item.name));
                println!("   ⚠️ {} - {} (Currently unavailable)", item_id, item.name);
            }
            None => {
                unavailable_items.push((item_id, "Unknown item"));
                println!("   ❌ {} - Not found in menu", item_id);
            }
        }
    }

    println!("\n📊 Order Summary:");
    println!("   Available items: {}", available_items.len());
    println!("   Unavailable/missing: {}", unavailable_items.len());
    println!("   Order total: ${:.2}", order_total);

    // Method 5: Advanced queries with filtering
    println!("\n🔍 Method 5: Advanced Menu Queries");

    // Find items by category
    let category_filter = "Bowls";
    let bowls: Vec<&MenuItem> = menu_database
        .values()
        .filter(|item| item.category == category_filter)
        .collect();

    println!("   {} items found:", category_filter);
    for item in bowls {
        println!("     • {} - ${:.2}", item.name, item.price);
    }

    // Find highly rated available items
    let premium_items: Vec<(&str, &MenuItem)> = menu_database
        .iter()
        .filter(|(_, item)| item.rating >= 4.7 && item.available)
        .collect();

    println!("   Premium items (4.7⭐+, available):");
    for (id, item) in premium_items {
        println!("     • {} - {} ({:.1}⭐)", id, item.name, item.rating);
    }

    // Price range analysis
    let budget_items: Vec<&MenuItem> = menu_database
        .values()
        .filter(|item| item.price <= 13.00 && item.available)
        .collect();

    println!("   Budget-friendly options (≤$13.00):");
    for item in budget_items {
        println!("     • {} - ${:.2}", item.name, item.price);
    }
}

fn main() {
    demonstrate_retrieval_methods();
}
```


### 3. Data Removal and Cleanup Methods

**Managing restaurant data lifecycle efficiently:**

```rust
fn demonstrate_removal_methods() {
    println!("🗑️ Restaurant Data Cleanup - Removal Methods");
    println!("{:=<50}", "");

    use std::collections::HashMap;

    // Setup order tracking system
    let mut active_orders: HashMap<String, OrderDetails> = HashMap::new();

    #[derive(Debug, Clone)]
    struct OrderDetails {
        customer_name: String,
        items: Vec<String>,
        total_amount: f64,
        order_time: String,
        table_number: Option<u32>,
        special_instructions: String,
    }

    // Populate with active orders
    active_orders.insert("ORD_001".to_string(), OrderDetails {
        customer_name: "Alice Johnson".to_string(),
        items: vec!["Quinoa Bowl".to_string(), "Green Tea".to_string()],
        total_amount: 18.98,
        order_time: "12:30 PM".to_string(),
        table_number: Some(5),
        special_instructions: "Extra avocado".to_string(),
    });

    active_orders.insert("ORD_002".to_string(), OrderDetails {
        customer_name: "Bob Smith".to_string(),
        items: vec!["Veggie Burger".to_string(), "Fries".to_string()],
        total_amount: 16.48,
        order_time: "12:35 PM".to_string(),
        table_number: Some(3),
        special_instructions: "No onions".to_string(),
    });

    active_orders.insert("ORD_003".to_string(), OrderDetails {
        customer_name: "Carol Davis".to_string(),
        items: vec!["Lentil Soup".to_string()],
        total_amount: 8.99,
        order_time: "12:40 PM".to_string(),
        table_number: None, // Takeaway
        special_instructions: "".to_string(),
    });

    println!("📊 Initial active orders: {}", active_orders.len());

    // Method 1: remove() - Remove and return value
    println!("\n✅ Method 1: Order Completion (remove)");

    match active_orders.remove("ORD_001") {
        Some(completed_order) => {
            println!("   🎉 Order ORD_001 completed!");
            println!("     Customer: {}", completed_order.customer_name);
            println!("     Items: {}", completed_order.items.join(", "));
            println!("     Total: ${:.2}", completed_order.total_amount);

            // Process payment, update customer loyalty points, etc.
            println!("     💳 Payment processed: ${:.2}", completed_order.total_amount);
        }
        None => {
            println!("   ❌ Order ORD_001 not found");
        }
    }

    // Method 2: remove_entry() - Remove and return both key and value
    println!("\n📋 Method 2: Complete Order Processing (remove_entry)");

    match active_orders.remove_entry("ORD_002") {
        Some((order_id, order_details)) => {
            println!("   ✅ Processed order: {}", order_id);
            println!("     Customer: {}", order_details.customer_name);
            println!("     Table: {:?}", order_details.table_number);

            // Log the completed order for analytics
            println!("     📊 Logged for daily sales report");
        }
        None => {
            println!("   ❌ Order not found for removal");
        }
    }

    // Method 3: Conditional removal with retain()
    println!("\n🎯 Method 3: Conditional Cleanup (retain)");

    let initial_count = active_orders.len();

    // Remove takeaway orders (no table number) that are ready
    active_orders.retain(|order_id, order_details| {
        let keep_order = order_details.table_number.is_some(); // Keep dine-in orders

        if !keep_order {
            println!("   📦 Takeaway order {} ready for pickup: {}",
                     order_id, order_details.customer_name);
        }

        keep_order
    });

    println!("   🧹 Cleaned up {} takeaway orders", initial_count - active_orders.len());

    // Method 4: Bulk removal with drain()
    println!("\n🔄 Method 4: End-of-Shift Processing (drain)");

    // Process all remaining orders
    let remaining_orders: HashMap<String, OrderDetails> = active_orders.drain().collect();

    println!("   📊 End-of-shift processing:");
    for (order_id, order_details) in remaining_orders {
        println!("     {} → {} (${:.2}) - Moved to completed orders",
                 order_id, order_details.customer_name, order_details.total_amount);
    }

    println!("   ✅ All active orders processed: {} remaining", active_orders.len());

    // Method 5: Selective removal with extract_if (Rust 1.88+)
    println!("\n⚗️ Method 5: Advanced Conditional Removal");

    // Setup a new set of orders for demonstration
    let mut order_queue: HashMap<String, OrderPriority> = HashMap::new();

    #[derive(Debug)]
    enum OrderPriority {
        Low,
        Normal,
        High,
        Rush,
    }

    order_queue.insert("ORD_101".to_string(), OrderPriority::Low);
    order_queue.insert("ORD_102".to_string(), OrderPriority::Rush);
    order_queue.insert("ORD_103".to_string(), OrderPriority::Normal);
    order_queue.insert("ORD_104".to_string(), OrderPriority::High);
    order_queue.insert("ORD_105".to_string(), OrderPriority::Rush);

    println!("   📋 Order queue before priority processing: {} orders", order_queue.len());

    // Note: extract_if is available in Rust 1.88+, using retain for compatibility
    let mut rush_orders = HashMap::new();
    let mut orders_to_remove = Vec::new();

    for (order_id, priority) in &order_queue {
        if matches!(priority, OrderPriority::Rush) {
            rush_orders.insert(order_id.clone(), priority);
            orders_to_remove.push(order_id.clone());
        }
    }

    for order_id in orders_to_remove {
        order_queue.remove(&order_id);
    }

    println!("   🚨 Extracted {} rush orders for immediate processing", rush_orders.len());
    println!("   📋 Remaining in queue: {} orders", order_queue.len());

    // Method 6: Complete cleanup with clear()
    println!("\n🧹 Method 6: Complete System Reset (clear)");

    let items_before = order_queue.len();
    order_queue.clear();

    println!("   🗑️ Cleared {} orders from queue", items_before);
    println!("   📊 Queue status: {} orders (capacity preserved: {})",
             order_queue.len(), order_queue.capacity() > 0);

    println!("\n🎯 Removal Methods Summary:");
    println!("   ✅ remove() - Remove and get value back");
    println!("   ✅ remove_entry() - Remove and get both key and value");
    println!("   ✅ retain() - Keep only items matching condition");
    println!("   ✅ drain() - Remove all items and iterate through them");
    println!("   ✅ extract_if() - Remove items matching condition (Rust 1.88+)");
    println!("   ✅ clear() - Remove everything, keep capacity");
}

fn main() {
    demonstrate_removal_methods();
}
```


## HashMap Iteration Patterns - Restaurant Data Analysis

### 1. Basic Iteration Methods

**Exploring your restaurant data in different ways:**

```rust
fn demonstrate_basic_iteration() {
    println!("🔄 Restaurant Data Analysis - Basic Iteration Patterns");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Setup comprehensive restaurant data
    let mut restaurant_analytics: HashMap<String, RestaurantMetrics> = HashMap::new();

    #[derive(Debug, Clone)]
    struct RestaurantMetrics {
        daily_revenue: f64,
        customer_count: u32,
        popular_dish: String,
        avg_rating: f32,
        staff_count: u8,
    }

    // Add analytics for different days
    restaurant_analytics.insert("Monday".to_string(), RestaurantMetrics {
        daily_revenue: 1247.50,
        customer_count: 89,
        popular_dish: "Quinoa Bowl".to_string(),
        avg_rating: 4.6,
        staff_count: 8,
    });

    restaurant_analytics.insert("Tuesday".to_string(), RestaurantMetrics {
        daily_revenue: 1156.25,
        customer_count: 78,
        popular_dish: "Veggie Burger".to_string(),
        avg_rating: 4.4,
        staff_count: 7,
    });

    restaurant_analytics.insert("Wednesday".to_string(), RestaurantMetrics {
        daily_revenue: 1389.75,
        customer_count: 95,
        popular_dish: "Mediterranean Wrap".to_string(),
        avg_rating: 4.8,
        staff_count: 9,
    });

    restaurant_analytics.insert("Thursday".to_string(), RestaurantMetrics {
        daily_revenue: 1534.80,
        customer_count: 112,
        popular_dish: "Lentil Curry".to_string(),
        avg_rating: 4.7,
        staff_count: 10,
    });

    // Pattern 1: iter() - Read-only iteration over key-value pairs
    println!("📊 Pattern 1: Daily Performance Review (iter)");

    for (day, metrics) in restaurant_analytics.iter() {
        let revenue_per_customer = metrics.daily_revenue / metrics.customer_count as f64;
        println!("   {} → ${:.2} revenue, {} customers (${:.2}/customer)",
                 day, metrics.daily_revenue, metrics.customer_count, revenue_per_customer);
    }

    // Pattern 2: keys() - Iterate over keys only
    println!("\n📅 Pattern 2: Operating Days Analysis (keys)");

    let operating_days: Vec<String> = restaurant_analytics.keys().cloned().collect();
    println!("   Days of operation: {}", operating_days.join(", "));

    // Find alphabetically first and last operating days
    let mut sorted_days = operating_days.clone();
    sorted_days.sort();
    if let (Some(first), Some(last)) = (sorted_days.first(), sorted_days.last()) {
        println!("   Week span: {} to {}", first, last);
    }

    // Pattern 3: values() - Iterate over values only
    println!("\n💰 Pattern 3: Revenue and Performance Analysis (values)");

    let total_revenue: f64 = restaurant_analytics.values()
        .map(|metrics| metrics.daily_revenue)
        .sum();

    let total_customers: u32 = restaurant_analytics.values()
        .map(|metrics| metrics.customer_count)
        .sum();

    let avg_daily_rating: f32 = restaurant_analytics.values()
        .map(|metrics| metrics.avg_rating)
        .sum::<f32>() / restaurant_analytics.len() as f32;

    println!("   📈 Weekly totals:");
    println!("     Revenue: ${:.2}", total_revenue);
    println!("     Customers: {}", total_customers);
    println!("     Average rating: {:.1}⭐", avg_daily_rating);
    println!("     Average per customer: ${:.2}", total_revenue / total_customers as f64);

    // Pattern 4: enumerate() with iter() - Index-aware iteration
    println!("\n🔢 Pattern 4: Day-by-Day Ranking (enumerate)");

    let mut daily_performance: Vec<(String, f64)> = restaurant_analytics
        .iter()
        .map(|(day, metrics)| (day.clone(), metrics.daily_revenue))
        .collect();

    // Sort by revenue (highest first)
    daily_performance.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());

    for (rank, (day, revenue)) in daily_performance.iter().enumerate() {
        let rank_emoji = match rank {
            0 => "🥇",
            1 => "🥈",
            2 => "🥉",
            _ => "📊",
        };
        println!("   {} Rank {}: {} (${:.2})", rank_emoji, rank + 1, day, revenue);
    }

    // Pattern 5: Filtering during iteration
    println!("\n🎯 Pattern 5: High-Performance Days (filter)");

    let high_revenue_days: Vec<&String> = restaurant_analytics
        .iter()
        .filter(|(_, metrics)| metrics.daily_revenue > 1300.0)
        .map(|(day, _)| day)
        .collect();

    println!("   High revenue days (>$1300): {}", high_revenue_days.join(", "));

    let high_customer_days: Vec<&String> = restaurant_analytics
        .iter()
        .filter(|(_, metrics)| metrics.customer_count > 90)
        .map(|(day, _)| day)
        .collect();

    println!("   Busy days (>90 customers): {}", high_customer_days.join(", "));

    // Pattern 6: Complex analysis with iteration
    println!("\n⚗️ Pattern 6: Advanced Performance Insights");

    // Find best and worst performing days
    let best_day = restaurant_analytics
        .iter()
        .max_by(|a, b| a.1.daily_revenue.partial_cmp(&b.1.daily_revenue).unwrap());

    let worst_day = restaurant_analytics
        .iter()
        .min_by(|a, b| a.1.daily_revenue.partial_cmp(&b.1.daily_revenue).unwrap());

    if let (Some((best_day, best_metrics)), Some((worst_day, worst_metrics))) = (best_day, worst_day) {
        println!("   🏆 Best day: {} (${:.2}, {} customers)",
                 best_day, best_metrics.daily_revenue, best_metrics.customer_count);
        println!("   📉 Lowest day: {} (${:.2}, {} customers)",
                 worst_day, worst_metrics.daily_revenue, worst_metrics.customer_count);

        let performance_gap = best_metrics.daily_revenue - worst_metrics.daily_revenue;
        println!("   📊 Performance gap: ${:.2} ({:.1}% difference)",
                 performance_gap,
                 (performance_gap / worst_metrics.daily_revenue) * 100.0);
    }

    // Popular dishes analysis
    let mut dish_popularity: HashMap<String, u32> = HashMap::new();
    for metrics in restaurant_analytics.values() {
        *dish_popularity.entry(metrics.popular_dish.clone()).or_insert(0) += 1;
    }

    println!("\n🍽️ Most popular dishes across the week:");
    for (dish, days) in dish_popularity.iter() {
        println!("     • {} → {} days", dish, days);
    }
}

fn main() {
    demonstrate_basic_iteration();
}
```


### 2. Mutable Iteration and Data Transformation

**Updating restaurant data through iteration:**

```rust
fn demonstrate_mutable_iteration() {
    println!("✏️ Restaurant Data Updates - Mutable Iteration Patterns");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Setup inventory management system
    let mut inventory_system: HashMap<String, InventoryItem> = HashMap::new();

    #[derive(Debug, Clone)]
    struct InventoryItem {
        name: String,
        current_stock: u32,
        min_threshold: u32,
        max_capacity: u32,
        unit_cost: f64,
        supplier: String,
        last_restocked: String,
    }

    // Initialize inventory
    inventory_system.insert("ING_001".to_string(), InventoryItem {
        name: "Organic Quinoa".to_string(),
        current_stock: 25,
        min_threshold: 10,
        max_capacity: 100,
        unit_cost: 8.50,
        supplier: "Organic Farms Co".to_string(),
        last_restocked: "2024-01-10".to_string(),
    });

    inventory_system.insert("ING_002".to_string(), InventoryItem {
        name: "Fresh Avocados".to_string(),
        current_stock: 8,
        min_threshold: 15,
        max_capacity: 50,
        unit_cost: 2.25,
        supplier: "Local Produce".to_string(),
        last_restocked: "2024-01-12".to_string(),
    });

    inventory_system.insert("ING_003".to_string(), InventoryItem {
        name: "Organic Tomatoes".to_string(),
        current_stock: 45,
        min_threshold: 20,
        max_capacity: 80,
        unit_cost: 3.75,
        supplier: "Greenhouse Gardens".to_string(),
        last_restocked: "2024-01-11".to_string(),
    });

    println!("📦 Initial Inventory Status:");
    for (id, item) in inventory_system.iter() {
        let status = if item.current_stock <= item.min_threshold {
            "🚨 LOW"
        } else if item.current_stock >= item.max_capacity * 8 / 10 {
            "✅ HIGH"
        } else {
            "📊 OK"
        };
        println!("   {} → {} units {} [{}]", id, item.current_stock, status, item.name);
    }

    // Pattern 1: iter_mut() - Mutable iteration over values
    println!("\n🔄 Pattern 1: Daily Usage Updates (iter_mut)");

    // Simulate daily ingredient usage
    let daily_usage = [
        ("ING_001", 5),  // Used 5 units of quinoa
        ("ING_002", 12), // Used 12 avocados
        ("ING_003", 15), // Used 15 tomatoes
    ];

    // Update stock levels for daily usage
    for (item_id, used_amount) in daily_usage {
        if let Some(item) = inventory_system.get_mut(item_id) {
            let previous_stock = item.current_stock;
            item.current_stock = item.current_stock.saturating_sub(used_amount);

            println!("   {} → Used {} units: {} → {} remaining",
                     item_id, used_amount, previous_stock, item.current_stock);
        }
    }

    // Pattern 2: Bulk updates with iter_mut()
    println!("\n📈 Pattern 2: Price Adjustment (bulk iter_mut)");

    // Apply 5% price increase to all items
    for (item_id, item) in inventory_system.iter_mut() {
        let old_cost = item.unit_cost;
        item.unit_cost *= 1.05; // 5% increase

        println!("   {} → Price updated: ${:.2} → ${:.2}",
                 item_id, old_cost, item.unit_cost);
    }

    // Pattern 3: Conditional updates based on status
    println!("\n🎯 Pattern 3: Automatic Restock Logic (conditional iter_mut)");

    let mut restock_orders = Vec::new();

    for (item_id, item) in inventory_system.iter_mut() {
        if item.current_stock <= item.min_threshold {
            // Calculate restock amount
            let restock_amount = item.max_capacity - item.current_stock;
            let old_stock = item.current_stock;

            // Update stock (simulate restock)
            item.current_stock = item.max_capacity;
            item.last_restocked = "2024-01-15".to_string(); // Today

            restock_orders.push((item_id.clone(), item.name.clone(), restock_amount, item.unit_cost));

            println!("   🔄 {} → Restocked: {} → {} units (ordered {} units)",
                     item_id, old_stock, item.current_stock, restock_amount);
        }
    }

    // Pattern 4: values_mut() - Iterate over mutable values only
    println!("\n⚗️ Pattern 4: Supplier Information Update (values_mut)");

    // Update all suppliers with new contact information
    for item in inventory_system.values_mut() {
        // Add contact info to supplier names
        if !item.supplier.contains("Contact:") {
            item.supplier = format!("{} (Contact: supplier@email.com)", item.supplier);
        }
    }

    println!("   ✅ Updated supplier contact information for all items");

    // Pattern 5: Entry API with mutable iteration
    println!("\n🔗 Pattern 5: Advanced Inventory Management (entry + iter_mut)");

    // Add new items or update existing ones
    let new_inventory_data = [
        ("ING_004", "Bell Peppers", 30, 12, 60, 1.85),
        ("ING_002", "Fresh Avocados", 0, 15, 50, 2.25), // Update existing
    ];

    for (item_id, name, stock, min_thresh, max_cap, cost) in new_inventory_data {
        let item = inventory_system.entry(item_id.to_string()).or_insert_with(|| {
            println!("   🆕 Adding new inventory item: {}", item_id);
            InventoryItem {
                name: name.to_string(),
                current_stock: 0,
                min_threshold: min_thresh,
                max_capacity: max_cap,
                unit_cost: cost,
                supplier: "New Supplier".to_string(),
                last_restocked: "2024-01-15".to_string(),
            }
        });

        // Update or set values
        if stock > 0 {
            item.current_stock = stock;
            println!("   📦 {} → Stock updated to {} units", item_id, stock);
        }
    }

    // Pattern 6: Complex data transformation
    println!("\n🧮 Pattern 6: Inventory Value Calculation and Analysis");

    let mut total_inventory_value = 0.0;
    let mut low_stock_alerts = Vec::new();
    let mut high_value_items = Vec::new();

    for (item_id, item) in inventory_system.iter_mut() {
        // Calculate item value
        let item_value = item.current_stock as f64 * item.unit_cost;
        total_inventory_value += item_value;

        // Check for alerts
        if item.current_stock <= item.min_threshold {
            low_stock_alerts.push((item_id.clone(), item.name.clone(), item.current_stock));
        }

        if item_value > 200.0 {
            high_value_items.push((item_id.clone(), item.name.clone(), item_value));
        }

        // Update internal tracking (simulate)
        // item.some_calculated_field = calculate_something();
    }

    println!("   💰 Total inventory value: ${:.2}", total_inventory_value);

    if !low_stock_alerts.is_empty() {
        println!("   🚨 Low stock alerts:");
        for (id, name, stock) in low_stock_alerts {
            println!("     • {} → {} ({} units)", id, name, stock);
        }
    }

    if !high_value_items.is_empty() {
        println!("   💎 High-value items:");
        for (id, name, value) in high_value_items {
            println!("     • {} → {} (${:.2})", id, name, value);
        }
    }

    // Display final inventory state
    println!("\n📊 Final Inventory Summary:");
    for (id, item) in inventory_system.iter() {
        let value = item.current_stock as f64 * item.unit_cost;
        println!("   {} → {} units @ ${:.2} each = ${:.2} total",
                 id, item.current_stock, item.unit_cost, value);
    }

    // Process restock orders
    if !restock_orders.is_empty() {
        println!("\n📋 Restock Orders Generated:");
        let mut total_restock_cost = 0.0;

        for (id, name, amount, cost) in restock_orders {
            let order_cost = amount as f64 * cost;
            total_restock_cost += order_cost;
            println!("   📦 {} → {} units of {} @ ${:.2} = ${:.2}",
                     id, amount, name, cost, order_cost);
        }

        println!("   💰 Total restock cost: ${:.2}", total_restock_cost);
    }
}

fn main() {
    demonstrate_mutable_iteration();
}
```


### 3. Consuming Iteration and Data Transformation

**Advanced iteration patterns for restaurant data processing:**

```rust
fn demonstrate_consuming_iteration() {
    println!("🔄 Restaurant Data Transformation - Consuming Iteration");
    println!("{:=<60}", "");

    use std::collections::HashMap;

    // Setup customer feedback system
    let customer_feedback: HashMap<String, CustomerFeedback> = {
        let mut feedback = HashMap::new();

        #[derive(Debug, Clone)]
        struct CustomerFeedback {
            customer_name: String,
            email: String,
            rating: u8,
            review_text: String,
            visit_date: String,
            dish_ordered: String,
            would_recommend: bool,
        }

        feedback.insert("FB_001".to_string(), CustomerFeedback {
            customer_name: "Alice Johnson".to_string(),
            email: "alice@email.com".to_string(),
            rating: 5,
            review_text: "Amazing quinoa bowl! Fresh ingredients and perfect seasoning.".to_string(),
            visit_date: "2024-01-10".to_string(),
            dish_ordered: "Quinoa Buddha Bowl".to_string(),
            would_recommend: true,
        });

        feedback.insert("FB_002".to_string(), CustomerFeedback {
            customer_name: "Bob Smith".to_string(),
            email: "bob@email.com".to_string(),
            rating: 4,
            review_text: "Good veggie burger, though the wait was a bit long.".to_string(),
            visit_date: "2024-01-11".to_string(),
            dish_ordered: "Veggie Burger".to_string(),
            would_recommend: true,
        });

        feedback.insert("FB_003".to_string(), CustomerFeedback {
            customer_name: "Carol Davis".to_string(),
            email: "carol@email.com".to_string(),
            rating: 3,
            review_text: "The lentil curry was okay, but not exceptional.".to_string(),
            visit_date: "2024-01-12".to_string(),
            dish_ordered: "Lentil Curry".to_string(),
            would_recommend: false,
        });

        feedback.insert("FB_004".to_string(), CustomerFeedback {
            customer_name: "Diana Wilson".to_string(),
            email: "diana@email.com".to_string(),
            rating: 5,
            review_text: "Excellent Mediterranean wrap! Will definitely be back.".to_string(),
            visit_date: "2024-01-13".to_string(),
            dish_ordered: "Mediterranean Wrap".to_string(),
            would_recommend: true,
        });

        feedback
    };

    println!("📝 Initial Feedback Database: {} reviews", customer_feedback.len());

    // Pattern 1: into_iter() - Consuming iteration
    println!("\n🔄 Pattern 1: Customer Data Migration (into_iter)");

    // Convert feedback into customer contact list
    let customer_contacts: Vec<(String, String)> = customer_feedback
        .clone() // Clone so we can reuse later
        .into_iter()
        .map(|(_, feedback)| (feedback.customer_name, feedback.email))
        .collect();

    println!("   📧 Customer contact list created:");
    for (name, email) in &customer_contacts {
        println!("     • {} → {}", name, email);
    }

    // Pattern 2: into_keys() - Extract all keys
    println!("\n🔑 Pattern 2: Feedback ID Analysis (into_keys)");

    let feedback_ids: Vec<String> = customer_feedback
        .clone()
        .into_keys()
        .collect();

    println!("   🆔 Feedback IDs: {}", feedback_ids.join(", "));

    // Analyze ID pattern
    let id_numbers: Vec<u32> = feedback_ids
        .iter()
        .filter_map(|id| id.strip_prefix("FB_"))
        .filter_map(|num_str| num_str.parse().ok())
        .collect();

    if let (Some(&min_id), Some(&max_id)) = (id_numbers.iter().min(), id_numbers.iter().max()) {
        println!("   📊 ID range: {} to {} (total span: {})", min_id, max_id, max_id - min_id + 1);
    }

    // Pattern 3: into_values() - Extract all values
    println!("\n💬 Pattern 3: Review Analysis (into_values)");

    let all_reviews: Vec<CustomerFeedback> = customer_feedback
        .clone()
        .into_values()
        .collect();

    // Calculate analytics
    let total_reviews = all_reviews.len();
    let average_rating = all_reviews.iter()
        .map(|r| r.rating as f64)
        .sum::<f64>() / total_reviews as f64;

    let recommendation_rate = all_reviews.iter()
        .filter(|r| r.would_recommend)
        .count() as f64 / total_reviews as f64 * 100.0;

    println!("   📊 Review Analytics:");
    println!("     Average rating: {:.1}/5 stars", average_rating);
    println!("     Recommendation rate: {:.1}%", recommendation_rate);

    // Pattern 4: Complex transformation with into_iter()
    println!("\n⚗️ Pattern 4: Dish Performance Analysis (complex transformation)");

    // Transform feedback into dish performance metrics
    let dish_performance: HashMap<String, DishMetrics> = {
        let mut performance = HashMap::new();

        #[derive(Debug)]
        struct DishMetrics {
            total_orders: u32,
            total_rating: u32,
            average_rating: f64,
            recommendations: u32,
            recommendation_rate: f64,
            recent_reviews: Vec<String>,
        }

        // Group feedback by dish
        for (_, feedback) in customer_feedback.clone().into_iter() {
            let dish = feedback.dish_ordered.clone();
            let entry = performance.entry(dish).or_insert_with(|| DishMetrics {
                total_orders: 0,
                total_rating: 0,
                average_rating: 0.0,
                recommendations: 0,
                recommendation_rate: 0.0,
                recent_reviews: Vec::new(),
            });

            entry.total_orders += 1;
            entry.total_rating += feedback.rating as u32;
            if feedback.would_recommend {
                entry.recommendations += 1;
            }
            entry.recent_reviews.push(feedback.review_text);
        }

        // Calculate averages
        for metrics in performance.values_mut() {
            metrics.average_rating = metrics.total_rating as f64 / metrics.total_orders as f64;
            metrics.recommendation_rate = metrics.recommendations as f64 / metrics.total_orders as f64 * 100.0;
        }

        performance
    };

    println!("   📈 Dish Performance Metrics:");
    for (dish, metrics) in &dish_performance {
        println!("     🍽️ {}:", dish);
        println!("       Orders: {}", metrics.total_orders);
        println!("       Avg Rating: {:.1}/5", metrics.average_rating);
        println!("       Recommendation Rate: {:.1}%", metrics.recommendation_rate);
    }

    // Pattern 5: Data aggregation and reporting
    println!("\n📊 Pattern 5: Comprehensive Reporting (aggregation)");

    // Create different reports from the same data
    let reports = {
        let mut reports = HashMap::new();

        // High-rated dishes report
        let high_rated_dishes: Vec<String> = dish_performance
            .iter()
            .filter(|(_, metrics)| metrics.average_rating >= 4.5)
            .map(|(dish, _)| dish.clone())
            .collect();

        reports.insert("high_rated_dishes".to_string(),
                      format!("Excellent dishes: {}", high_rated_dishes.join(", ")));

        // Customer satisfaction report
        let satisfied_customers = all_reviews.iter()
            .filter(|r| r.rating >= 4)
            .count();

        reports.insert("customer_satisfaction".to_string(),
                      format!("{}/{} customers satisfied ({}%)",
                             satisfied_customers, total_reviews,
                             (satisfied_customers as f64 / total_reviews as f64 * 100.0) as u32));

        // Improvement opportunities
        let low_rated_feedback: Vec<String> = all_reviews
            .iter()
            .filter(|r| r.rating <= 3)
            .map(|r| format!("{}: {}", r.dish_ordered, r.review_text))
            .collect();

        reports.insert("improvement_opportunities".to_string(),
                      format!("Areas for improvement: {}",
                             if low_rated_feedback.is_empty() {
                                 "None identified".to_string()
                             } else {
                                 low_rated_feedback.join("; ")
                             }));

        reports
    };

    println!("   📋 Generated Reports:");
    for (report_type, content) in reports {
        println!("     📄 {}: {}", report_type, content);
    }

    // Pattern 6: Data export and transformation
    println!("\n📤 Pattern 6: Data Export Preparation (final transformation)");

    // Transform feedback into CSV-ready format
    let csv_data: Vec<String> = customer_feedback
        .into_iter()
        .map(|(id, feedback)| {
            format!("{},{},{},{},{},{},{}",
                   id,
                   feedback.customer_name,
                   feedback.email,
                   feedback.rating,
                   feedback.dish_ordered,
                   feedback.would_recommend,
                   feedback.visit_date)
        })
        .collect();

    println!("   📊 CSV Export Data Prepared:");
    println!("     Header: ID,Name,Email,Rating,Dish,Recommend,Date");
    for (i, row) in csv_data.iter().take(3).enumerate() {
        println!("     Row {}: {}", i + 1, row);
    }
    if csv_data.len() > 3 {
        println!("     ... and {} more rows", csv_data.len() - 3);
    }

    // Summary statistics
    println!("\n🎯 Processing Summary:");
    println!("   ✅ Processed {} customer feedback entries", csv_data.len());
    println!("   ✅ Generated {} performance metrics", dish_performance.len());
    println!("   ✅ Created {} customer contacts", customer_contacts.len());
    println!("   ✅ Prepared data export with {} rows", csv_data.len());
}

fn main() {
    demonstrate_consuming_iteration();
}
```


## Advanced HashMap Methods and Entry API

### The Entry API - Sophisticated Data Management

**Professional-level restaurant data operations:**

```rust
fn demonstrate_entry_api() {
    println!("🎯 Advanced Restaurant Management - Entry API Operations");
    println!("{:=<60}", "");

    use std::collections::HashMap;
    use std::collections::hash_map::Entry;

    // Setup loyalty program system
    let mut loyalty_program: HashMap<String, LoyaltyAccount> = HashMap::new();

    #[derive(Debug, Clone)]
    struct LoyaltyAccount {
        customer_name: String,
        points_balance: u32,
        tier: String,
        last_visit: String,
        total_spent: f64,
        favorite_dishes: Vec<String>,
    }

    // Pattern 1: or_insert() - Create account if doesn't exist
    println!("🆕 Pattern 1: Customer Account Creation (or_insert)");

    let new_customers = [
        ("alice@email.com", "Alice Johnson", 25.50, "Quinoa Bowl"),
        ("bob@email.com", "Bob Smith", 18.75, "Veggie Burger"),
        ("carol@email.com", "Carol Davis", 32.25, "Mediterranean Wrap"),
    ];

    for (email, name, amount_spent, dish) in new_customers {
        let account = loyalty_program
            .entry(email.to_string())
            .or_insert(LoyaltyAccount {
                customer_name: name.to_string(),
                points_balance: 0,
                tier: "Bronze".to_string(),
                last_visit: "2024-01-15".to_string(),
                total_spent: 0.0,
                favorite_dishes: Vec::new(),
            });

        // Update account for this visit
        account.points_balance += (amount_spent as u32) * 2; // 2 points per dollar
        account.total_spent += amount_spent;
        account.favorite_dishes.push(dish.to_string());

        println!("   ✅ {} → {} points, ${:.2} spent", email, account.points_balance, account.total_spent);
    }

    // Pattern 2: or_insert_with() - Expensive default creation
    println!("\n💰 Pattern 2: Premium Account Setup (or_insert_with)");

    fn create_premium_account(name: &str) -> LoyaltyAccount {
        println!("     🏗️ Creating premium account for {}", name);
        LoyaltyAccount {
            customer_name: name.to_string(),
            points_balance: 500, // Welcome bonus
            tier: "Gold".to_string(),
            last_visit: "2024-01-15".to_string(),
            total_spent: 0.0,
            favorite_dishes: vec!["Premium Selection".to_string()],
        }
    }

    let premium_customer = "diana@email.com";
    let account = loyalty_program
        .entry(premium_customer.to_string())
        .or_insert_with(|| create_premium_account("Diana Wilson"));

    println!("   🌟 Premium account: {} points, {} tier",
             account.points_balance, account.tier);

    // Pattern 3: and_modify() - Update existing accounts
    println!("\n🔄 Pattern 3: Account Updates (and_modify)");

    // Process daily visits and point earnings
    let daily_visits = [
        ("alice@email.com", 45.25, vec!["Green Smoothie", "Salad"]),
        ("unknown@email.com", 22.50, vec!["Pasta"]), // New customer
        ("bob@email.com", 38.75, vec!["Burger", "Fries"]),
    ];

    for (email, amount, dishes) in daily_visits {
        loyalty_program
            .entry(email.to_string())
            .and_modify(|account| {
                // Update existing account
                let points_earned = (amount as u32) * 2;
                account.points_balance += points_earned;
                account.total_spent += amount;
                account.last_visit = "2024-01-16".to_string();
                account.favorite_dishes.extend(dishes.iter().map(|s| s.to_string()));

                // Check for tier upgrade
                if account.total_spent > 100.0 && account.tier == "Bronze" {
                    account.tier = "Silver".to_string();
                    account.points_balance += 100; // Tier upgrade bonus
                    println!("     🎉 {} upgraded to Silver tier!", email);
                }

                println!("     ✅ {} → +{} points, ${:.2} total spent",
                         email, points_earned, account.total_spent);
            })
            .or_insert_with(|| {
                // Create new account for unknown customers
                println!("     🆕 New customer account created for {}", email);
                LoyaltyAccount {
                    customer_name: "New Customer".to_string(),
                    points_balance: (amount as u32) * 2,
                    tier: "Bronze".to_string(),
                    last_visit: "2024-01-16".to_string(),
                    total_spent: amount,
                    favorite_dishes: dishes.iter().map(|s| s.to_string()).collect(),
                }
            });
    }

    // Pattern 4: Complex Entry API operations
    println!("\n⚗️ Pattern 4: Advanced Loyalty Management (complex operations)");

    // Implement point redemption system
    let redemption_requests = [
        ("alice@email.com", 50, "Free Appetizer"),
        ("bob@email.com", 200, "Free Main Course"),
        ("carol@email.com", 1000, "VIP Experience"), // Insufficient points
    ];

    for (email, points_needed, reward) in redemption_requests {
        match loyalty_program.entry(email.to_string()) {
            Entry::Occupied(mut entry) => {
                let account = entry.get_mut();
                if account.points_balance >= points_needed {
                    account.points_balance -= points_needed;
                    println!("   ✅ {} redeemed {} for {} ({} points remaining)",
                             email, reward, points_needed, account.points_balance);
                } else {
                    let points_short = points_needed - account.points_balance;
                    println!("   ❌ {} needs {} more points for {} (has {})",
                             email, points_short, reward, account.points_balance);
                }
            }
            Entry::Vacant(_) => {
                println!("   ❌ {} → No loyalty account found", email);
            }
        }
    }

    // Pattern 5: Batch processing with Entry API
    println!("\n📦 Pattern 5: Batch Loyalty Updates (batch processing)");

    // End-of-month tier adjustments
    let mut tier_adjustments = 0;

    for (email, account) in loyalty_program.iter_mut() {
        let new_tier = match account.total_spent {
            spent if spent >= 200.0 => "Platinum",
            spent if spent >= 100.0 => "Gold",
            spent if spent >= 50.0 => "Silver",
            _ => "Bronze",
        };

        if new_tier != account.tier {
            let old_tier = account.tier.clone();
            account.tier = new_tier.to_string();

            // Tier upgrade bonus
            let bonus_points = match new_tier {
                "Platinum" => 500,
                "Gold" => 200,
                "Silver" => 100,
                _ => 0,
            };

            account.points_balance += bonus_points;
            tier_adjustments += 1;

            println!("   📈 {} → {} to {} tier (+{} bonus points)",
                     email, old_tier, new_tier, bonus_points);
        }
    }

    if tier_adjustments == 0 {
        println!("   ℹ️ No tier adjustments needed this month");
    }

    // Pattern 6: Entry API for analytics and reporting
    println!("\n📊 Pattern 6: Loyalty Program Analytics");

    // Analyze loyalty program performance
    let mut tier_distribution: HashMap<String, u32> = HashMap::new();
    let mut total_points_issued = 0u32;
    let mut total_customer_value = 0.0;

    for account in loyalty_program.values() {
        *tier_distribution.entry(account.tier.clone()).or_insert(0) += 1;
        total_points_issued += account.points_balance;
        total_customer_value += account.total_spent;
    }

    println!("   🏆 Tier Distribution:");
    for (tier, count) in &tier_distribution {
        let percentage = (*count as f64 / loyalty_program.len() as f64) * 100.0;
        println!("     {} → {} customers ({:.1}%)", tier, count, percentage);
    }

    println!("   💎 Program Metrics:");
    println!("     Total customers: {}", loyalty_program.len());
    println!("     Total points issued: {}", total_points_issued);
    println!("     Total customer value: ${:.2}", total_customer_value);
    println!("     Average customer value: ${:.2}",
             total_customer_value / loyalty_program.len() as f64);

    // Find top customers
    let mut top_customers: Vec<_> = loyalty_program
        .iter()
        .map(|(email, account)| (email, account.total_spent, account.points_balance))
        .collect();

    top_customers.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());

    println!("   🥇 Top Customers by Spending:");
    for (i, (email, spent, points)) in top_customers.iter().take(3).enumerate() {
        let rank_emoji = match i {
            0 => "🥇",
            1 => "🥈",
            2 => "🥉",
            _ => "📊",
        };
        println!("     {} {} → ${:.2} spent, {} points", rank_emoji, email, spent, points);
    }
}

fn main() {
    demonstrate_entry_api();
}
```


## Best Practices and Performance Considerations

### Professional HashMap Usage Guidelines

```rust
fn demonstrate_best_practices() {
    println!("✅ HashMap Methods & Iteration Best Practices");
    println!("{:=<55}", "");

    use std::collections::HashMap;
    use std::time::Instant;

    // Best Practice 1: Choose the right iteration method
    println!("🎯 Best Practice 1: Optimal Iteration Method Selection");

    let mut restaurant_data: HashMap<String, f64> = HashMap::new();
    for i in 0..1000 {
        restaurant_data.insert(format!("item_{}", i), i as f64 * 1.5);
    }

    // ✅ Use iter() for read-only operations
    let start = Instant::now();
    let sum_iter: f64 = restaurant_data.iter().map(|(_, &value)| value).sum();
    let iter_time = start.elapsed();

    // ✅ Use values() when you only need values
    let start = Instant::now();
    let sum_values: f64 = restaurant_data.values().sum();
    let values_time = start.elapsed();

    println!("   📊 Performance comparison:");
    println!("     iter() + map: {:?} → sum = {:.1}", iter_time, sum_iter);
    println!("     values(): {:?} → sum = {:.1}", values_time, sum_values);
    println!("     ✅ values() is more efficient for value-only operations");

    // Best Practice 2: Safe HashMap operations
    println!("\n🛡️ Best Practice 2: Safe Access Patterns");

    let menu_prices: HashMap<&str, f64> = [
        ("Quinoa Bowl", 15.99),
        ("Veggie Burger", 12.49),
        ("Lentil Soup", 9.75),
    ].into();

    // ✅ Safe access with pattern matching
    match menu_prices.get("Quinoa Bowl") {
        Some(price) => println!("   ✅ Safe access: Quinoa Bowl costs ${:.2}", price),
        None => println!("   ❌ Item not found"),
    }

    // ✅ Safe access with unwrap_or for defaults
    let unknown_price = menu_prices.get("Pizza").unwrap_or(&0.0);
    println!("   ✅ Safe default: Unknown item costs ${:.2}", unknown_price);

    // ❌ Dangerous: Direct indexing (can panic)
    // let price = menu_prices["Pizza"]; // Would panic!

    println!("   💡 Always use get() instead of direct indexing");

    // Best Practice 3: Efficient data updates
    println!("\n⚡ Best Practice 3: Efficient Update Patterns");

    let mut order_counts: HashMap<String, u32> = HashMap::new();
    let orders = ["Pizza", "Burger", "Pizza", "Salad", "Burger", "Pizza"];

    // ✅ Efficient: Use entry API for conditional updates
    let start = Instant::now();
    for &order in &orders {
        *order_counts.entry(order.to_string()).or_insert(0) += 1;
    }
    let entry_api_time = start.elapsed();

    // ❌ Less efficient: Check then update pattern
    let mut order_counts_slow: HashMap<String, u32> = HashMap::new();
    let start = Instant::now();
    for &order in &orders {
        if order_counts_slow.contains_key(order) {
            *order_counts_slow.get_mut(order).unwrap() += 1;
        } else {
            order_counts_slow.insert(order.to_string(), 1);
        }
    }
    let check_update_time = start.elapsed();

    println!("   📊 Update performance:");
    println!("     Entry API: {:?}", entry_api_time);
    println!("     Check-then-update: {:?}", check_update_time);
    println!("     ✅ Entry API is more efficient and cleaner");

    // Best Practice 4: Memory-efficient iteration
    println!("\n💾 Best Practice 4: Memory-Efficient Patterns");

    let large_dataset: HashMap<String, Vec<u32>> = (0..100)
        .map(|i| (format!("key_{}", i), vec![i; 100]))
        .collect();

    // ✅ Efficient: Process without cloning
    let total_elements: usize = large_dataset.values()
        .map(|v| v.len())
        .sum();

    println!("   ✅ Efficient counting: {} total elements", total_elements);

    // ❌ Inefficient: Unnecessary cloning
    // let cloned_data: Vec<Vec<u32>> = large_dataset.values().cloned().collect();

    println!("   💡 Use references when possible, avoid unnecessary cloning");

    // Best Practice 5: Iterator chaining for complex operations
    println!("\n🔗 Best Practice 5: Effective Iterator Chaining");

    let customer_ratings: HashMap<String, (f32, u32)> = [
        ("Alice".to_string(), (4.8, 15)),  // (rating, visit_count)
        ("Bob".to_string(), (4.2, 8)),
        ("Carol".to_string(), (4.9, 22)),
        ("Diana".to_string(), (3.8, 5)),
        ("Emma".to_string(), (4.6, 12)),
    ].into();

    // ✅ Efficient: Chain operations without intermediate collections
    let loyal_customers: Vec<String> = customer_ratings
        .iter()
        .filter(|(_, (rating, visits))| *rating >= 4.5 && *visits >= 10)
        .map(|(name, _)| name.clone())
        .collect();

    println!("   🏆 Loyal customers (4.5+ rating, 10+ visits): {}",
             loyal_customers.join(", "));

    // Calculate statistics in one pass
    let (total_rating, total_visits, customer_count) = customer_ratings
        .values()
        .fold((0.0, 0u32, 0u32), |(sum_rating, sum_visits, count), (rating, visits)| {
            (sum_rating + rating, sum_visits + visits, count + 1)
        });

    let avg_rating = total_rating / customer_count as f32;
    let avg_visits = total_visits as f32 / customer_count as f32;

    println!("   📊 Statistics: {:.1} avg rating, {:.1} avg visits per customer",
             avg_rating, avg_visits);

    // Best Practice 6: Appropriate data structures for different use cases
    println!("\n🏗️ Best Practice 6: Right Tool for the Job");

    println!("   ✅ Use HashMap when:");
    println!("     • You need O(1) lookups by key");
    println!("     • Key-value associations are primary concern");
    println!("     • Order doesn't matter");

    println!("   ⚠️ Consider alternatives when:");
    println!("     • You need ordered iteration → BTreeMap");
    println!("     • You have small, fixed datasets → arrays or small structs");
    println!("     • You need multi-dimensional lookups → nested maps or specialized structures");

    // Best Practice 7: Error handling patterns
    println!("\n🚨 Best Practice 7: Robust Error Handling");

    fn safe_menu_lookup(menu: &HashMap<&str, f64>, item: &str) -> Result<f64, String> {
        menu.get(item)
            .copied()
            .ok_or_else(|| format!("Menu item '{}' not found", item))
    }

    fn calculate_order_total(menu: &HashMap<&str, f64>, items: &[&str]) -> Result<f64, String> {
        items.iter()
            .map(|&item| safe_menu_lookup(menu, item))
            .collect::<Result<Vec<_>, _>>()
            .map(|prices| prices.iter().sum())
    }

    let menu = [("Salad", 12.99), ("Soup", 8.99)].into();
    let order = ["Salad", "Soup", "Pizza"];

    match calculate_order_total(&menu, &order) {
        Ok(total) => println!("   ✅ Order total: ${:.2}", total),
        Err(error) => println!("   ❌ Order error: {}", error),
    }

    println!("\n🎯 HashMap Best Practices Summary:");
    println!("   ✅ Use appropriate iteration methods (iter, keys, values)");
    println!("   ✅ Always use safe access patterns (get, not direct indexing)");
    println!("   ✅ Leverage Entry API for conditional operations");
    println!("   ✅ Chain iterators efficiently without intermediate collections");
    println!("   ✅ Handle errors gracefully with Result types");
    println!("   ✅ Choose the right data structure for your use case");
    println!("   ✅ Measure performance and optimize bottlenecks");
}

fn main() {
    demonstrate_best_practices();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Restaurant Management System**

Remember our comprehensive restaurant analogy:

- 🎯 **Core methods** = **Essential operations** - Add, find, update, remove customer data
- 🔄 **Iteration patterns** = **Data analysis tools** - Review, transform, and report on information
- 🏗️ **Entry API** = **Advanced management features** - Sophisticated conditional operations
- ⚡ **Performance optimization** = **Efficient workflows** - Choose the right tool for each task
- 🛡️ **Best practices** = **Professional standards** - Safe, reliable, maintainable operations


### **Essential HashMap Methods Quick Reference**

| **Category** | **Method** | **Purpose** | **Returns** | **Use Case** |
| :-- | :-- | :-- | :-- | :-- |
| **Insertion** | `insert(k, v)` | Add/update entry | `Option<V>` | Store customer data |
| **Retrieval** | `get(&k)` | Safe access | `Option<&V>` | Look up information |
| **Existence** | `contains_key(&k)` | Check presence | `bool` | Verify customer exists |
| **Removal** | `remove(&k)` | Delete entry | `Option<V>` | Remove customer record |
| **Conditional** | `entry(k).or_insert(v)` | Insert if absent | `&mut V` | Create default records |

### **Iteration Patterns Quick Reference**

| **Method** | **Returns** | **Use Case** | **Performance** |
| :-- | :-- | :-- | :-- |
| **`iter()`** | `(&K, &V)` pairs | Read-only analysis | O(capacity) |
| **`iter_mut()`** | `(&K, &mut V)` pairs | Update values | O(capacity) |
| **`into_iter()`** | `(K, V)` pairs | Consume HashMap | O(capacity) |
| **`keys()`** | `&K` iterator | Extract keys only | O(capacity) |
| **`values()`** | `&V` iterator | Extract values only | O(capacity) |
| **`values_mut()`** | `&mut V` iterator | Modify values only | O(capacity) |

### **Entry API Patterns**

```rust
// Essential Entry API patterns
use std::collections::HashMap;

let mut map = HashMap::new();

// Create if absent
map.entry("customer").or_insert("default_info");

// Update existing or create new
map.entry("customer")
   .and_modify(|info| *info = "updated")
   .or_insert("new_info");

// Complex conditional logic
match map.entry("customer") {
    Entry::Occupied(mut e) => {
        // Update existing
        e.insert("new_value");
    }
    Entry::Vacant(e) => {
        // Create new
        e.insert("initial_value");
    }
}
```


### **Best Practices Checklist**

**✅ DO:**

- Use `get()` for safe access, never direct indexing `map[key]`
- Choose the right iteration method for your use case
- Use Entry API for conditional insert/update operations
- Chain iterators efficiently without intermediate collections
- Handle `Option` and `Result` types properly
- Use `values()` when you only need values, not keys
- Pre-size HashMap with `with_capacity()` when size is known

**❌ DON'T:**

- Use direct indexing `map[key]` - it can panic
- Clone large data unnecessarily during iteration
- Use `unwrap()` on `get()` results without checking
- Create intermediate collections when iterator chaining works
- Ignore the performance characteristics of different methods
- Mix up owned and borrowed iteration when ownership matters


### **Real-World Applications**

**HashMap methods and iteration are perfect for:**

- 🏪 **Customer management** - Registration, updates, loyalty programs
- 📊 **Data analytics** - Aggregation, reporting, trend analysis
- 🎮 **Game development** - Player stats, inventory, leaderboards
- 🌐 **Web applications** - Session management, caching, user preferences
- 📈 **Business intelligence** - KPI tracking, performance metrics
- 🔄 **Data processing pipelines** - Transform, filter, and aggregate data


### **Performance Guidelines**

**Method Selection:**

- Use `get()` for single lookups: O(1) average
- Use `values()` for value-only operations: More efficient than `iter().map()`
- Use Entry API for conditional operations: Avoids double lookup
- Use `retain()` for bulk conditional removal: More efficient than manual filtering
- Use appropriate iteration method: `iter()` vs `into_iter()` vs `iter_mut()`

**Memory Efficiency:**

- Avoid cloning when borrowing works
- Use `&` for read-only access to values
- Pre-allocate HashMap capacity when size is predictable
- Consider using `Box<T>` for large values to keep HashMap compact


### **The Professional Advantage**

**Mastering HashMap methods and iteration in Rust is like having a complete restaurant management system** that handles every operational need efficiently:

- ⚡ **Instant operations** - O(1) average performance for core operations
- 🔄 **Flexible analysis** - Multiple ways to process and transform data
- 🛡️ **Memory safety** - No crashes from invalid access or iterator invalidation
- 📊 **Rich functionality** - Comprehensive method ecosystem for all use cases
- 🎯 **Performance optimization** - Choose optimal patterns for each scenario

**Understanding these methods and patterns transforms you from a functional programmer to a systems expert** who can build efficient, reliable Rust applications that scale from simple scripts to enterprise systems. Just as a master restaurateur uses every tool and technique to create exceptional dining experiences, a skilled Rust programmer leverages the full HashMap API to build robust, performant software that handles complex data relationships with elegance and efficiency!

1. https://www.w3schools.com/rust/rust_hashmap.php
2. https://doc.rust-lang.org/std/collections/struct.HashMap.html
3. https://www.programiz.com/rust/hashmap
4. https://dev.to/iamdipankarpaul/comprehensive-guide-to-hashmaps-in-rust-mci
5. https://www.geeksforgeeks.org/rust/rust-hashmaps/
6. https://www.w3resource.com/rust/collections_and_data_structures/rust-hashmaps-exercise-2.php
7. https://www.youtube.com/watch?v=-yXcVNqo0DQ
8. https://doc.rust-lang.org/rust-by-example/std/hash.html
9. https://www.dotnetperls.com/hashmap-rust
10. https://dev.to/fadygrab/learning-rust-18-rust-collections-hashmaps-accessing-values-with-keys-instead-of-indices-4kg2
11. https://www.koderhq.com/tutorial/rust/hashmap/
12. https://lakum.in/blog/iterate-over-a-hashmap-in-rust/
13. https://www.youtube.com/watch?v=QYesA996W1U
14. https://stackoverflow.com/questions/45724517/how-to-iterate-through-a-hashmap-print-the-key-value-and-remove-the-value-in-ru
15. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch08-03-hash-maps.html
16. https://www.reddit.com/r/rust/comments/afcyb0/how_do_you_iterate_over_a_hash_table_in_rust/
17. https://doc.rust-lang.org/std/collections/hash_map/struct.Iter.html
18. https://users.rust-lang.org/t/hashmap-values-to-slice-or-iterator/103278
19. https://betterprogramming.pub/implementing-a-hashmap-in-rust-35d055b5ac2b
20. https://www.reddit.com/r/rust/comments/16irvqw/iterating_over_a_hashmap_and_filtering/
21. https://www.reddit.com/r/rust/comments/5tzinx/hashmapiter_entries_and_hashmapiter_entries_mut/
22. https://www.w3schools.com/java/java_howto_loop_through_hashmap.asp
23. https://users.rust-lang.org/t/iterate-through-hashmap-while-mutating-it/17100
24. https://users.rust-lang.org/t/recursive-iteration-while-caching-into-hashmap/115018
25. https://notes.kodekloud.com/docs/Rust-Programming/Collections-Error-Handling/Hashmaps
26. https://www.reddit.com/r/rust/comments/1f9qc6u/loop_through_hashmap_and_mutate_its_values_during/
27. https://users.rust-lang.org/t/iterating-hashmap-vs-iterating-vec/92590
28. https://users.rust-lang.org/t/mutating-values-while-iterating-a-map-based-on-keys/91814
29. https://users.rust-lang.org/t/mapping-hashmap-values-to-a-different-type/113731
30. https://stackoverflow.com/questions/51542024/how-do-i-use-the-entry-api-with-an-expensive-key-that-is-only-constructed-if-the
31. https://stackoverflow.com/questions/56724014/how-do-i-collect-the-values-of-a-hashmap-into-a-vector
32. https://www.knowbe4.com/careers/blogs/engineering/on-rusts-map-entry-pattern
33. https://www.reddit.com/r/rust/comments/2hneka/help_filtermap_items_of_hashmap/
34. https://users.rust-lang.org/t/understanding-the-hashmap-entry-method/88765
35. https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html
36. https://exercism.org/tracks/rust/concepts/entry-api
37. https://users.rust-lang.org/t/hashmap-entry-api-and-ownership/81368
