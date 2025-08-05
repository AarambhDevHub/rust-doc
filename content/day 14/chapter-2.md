+++
title = "Enum Variants with Data"
description = "Explore how Rust enums can store data in each variant, making them powerful tools for building expressive and type-safe applications."
date = 2025-08-05
draft = false
weight = 2
+++

# Enum Variants with Data in Rust: Comprehensive Documentation for Beginners

Understanding enum variants with data is like learning to **create specialized order tickets in a vegetarian restaurant** - each type of order (enum variant) needs to carry different information with it. Just as a "Take-out Order" needs an address and phone number, while a "Dine-in Order" needs a table number and dietary preferences, enum variants can carry exactly the data they need to be useful.

## The Restaurant Order System Analogy 🍽️

### Imagine You're Managing Different Types of Restaurant Orders

**Traditional Approach (Separate Structs):**

```rust
// ❌ Separate types - hard to handle uniformly
struct DineInOrder {
    table_number: u32,
    guest_count: u32,
}

struct TakeoutOrder {
    customer_phone: String,
    pickup_time: String,
}

struct DeliveryOrder {
    address: String,
    delivery_fee: f64,
}

// How do you handle these together? Very difficult!
```

**Enum with Data Approach (Unified Order System):**

```rust
// ✅ One enum that handles all order types with their specific data
#[derive(Debug)]
enum RestaurantOrder {
    DineIn {
        table_number: u32,
        guest_count: u32
    },
    Takeout {
        customer_phone: String,
        pickup_time: String
    },
    Delivery {
        address: String,
        delivery_fee: f64
    },
    Catering(String, u32, f64), // Event name, guests, total cost
}

fn process_order(order: RestaurantOrder) {
    match order {
        RestaurantOrder::DineIn { table_number, guest_count } => {
            println!("🍽️ Prepare table {} for {} guests", table_number, guest_count);
        }
        RestaurantOrder::Takeout { customer_phone, pickup_time } => {
            println!("📞 Call {} when order ready at {}", customer_phone, pickup_time);
        }
        RestaurantOrder::Delivery { address, delivery_fee } => {
            println!("🚗 Deliver to {} (${:.2} delivery fee)", address, delivery_fee);
        }
        RestaurantOrder::Catering(event, guests, cost) => {
            println!("🎉 Catering for '{}' - {} guests, total: ${:.2}", event, guests, cost);
        }
    }
}

fn main() {
    let orders = vec![
        RestaurantOrder::DineIn { table_number: 5, guest_count: 4 },
        RestaurantOrder::Takeout {
            customer_phone: "555-0123".to_string(),
            pickup_time: "6:30 PM".to_string()
        },
        RestaurantOrder::Delivery {
            address: "123 Veggie Lane".to_string(),
            delivery_fee: 3.99
        },
        RestaurantOrder::Catering("Office Party".to_string(), 25, 450.00),
    ];

    for order in orders {
        process_order(order);
    }
}
```

**Why This Works Better:**

- 🎯 **Type safety** - Each order type carries exactly the data it needs
- 🔄 **Uniform handling** - Process all order types with one function
- 🚫 **No invalid states** - Can't have a dine-in order with a delivery address
- 📋 **Clear organization** - All order types are related but distinct


## What are Enum Variants with Data?

### Core Concept

**Enum variants with data** allow each variant to carry additional information relevant to that specific case. This lets you create powerful, type-safe data structures that can represent different scenarios while maintaining related behavior.


| Topic | Explanation |
| :-- | :-- |
| **Basic Concept** | Enum variants in Rust can carry additional data, enabling each variant to store relevant information alongside its name. |
| **Tuple Variants** | Variants can have unnamed data fields, similar to tuple structs. For example: enum Message { Quit, Move(i32, i32), Write(String) } |
| **Struct Variants** | Variants can also contain named fields, like structs. For example: enum Message { ChangeColor { r: u8, g: u8, b: u8 } } |
| **Usage** | When matching against an enum, destructuring lets you access the data inside the variant. |
| **Design Tip** | Use enums to encode different types of related data under a single type safety umbrella. |
| **Common Use Cases** | Event handling, state machines, network protocols, parsing complex data. |
| **Advantages** | Enums with associated data eliminate invalid states and enforce safe handling via pattern matching. |

### The Recipe Instruction System

Think of enum variants with data like detailed cooking instructions:

```rust
#[derive(Debug)]
enum CookingInstruction {
    // Unit variant - no extra data needed
    Preheat,

    // Tuple variant - single piece of data
    SetTimer(u32), // minutes

    // Tuple variant - multiple pieces of data
    SetTemperature(u32, String), // degrees, unit (F/C)

    // Struct variant - named fields for clarity
    AddIngredient {
        name: String,
        amount: String,
        preparation: Option<String>
    },

    // Complex tuple variant
    CookFor(u32, String, u32), // time, method, temperature
}

fn execute_instruction(instruction: CookingInstruction) {
    match instruction {
        CookingInstruction::Preheat => {
            println!("🔥 Preheating oven to default temperature");
        }
        CookingInstruction::SetTimer(minutes) => {
            println!("⏰ Setting timer for {} minutes", minutes);
        }
        CookingInstruction::SetTemperature(degrees, unit) => {
            println!("🌡️ Setting temperature to {}°{}", degrees, unit);
        }
        CookingInstruction::AddIngredient { name, amount, preparation } => {
            let prep_note = preparation
                .map(|p| format!(" ({})", p))
                .unwrap_or_default();
            println!("🥕 Add {} {}{}", amount, name, prep_note);
        }
        CookingInstruction::CookFor(time, method, temp) => {
            println!("👨‍🍳 {} for {} minutes at {}°F", method, time, temp);
        }
    }
}

fn main() {
    let recipe_steps = vec![
        CookingInstruction::Preheat,
        CookingInstruction::SetTemperature(375, "F".to_string()),
        CookingInstruction::AddIngredient {
            name: "quinoa".to_string(),
            amount: "2 cups".to_string(),
            preparation: Some("rinsed".to_string()),
        },
        CookingInstruction::SetTimer(15),
        CookingInstruction::CookFor(20, "Simmer".to_string(), 350),
    ];

    println!("🍳 Executing Recipe:");
    println!("{:=<50}", "");

    for (step, instruction) in recipe_steps.into_iter().enumerate() {
        print!("Step {}: ", step + 1);
        execute_instruction(instruction);
    }
}
```

Each instruction carries exactly the data it needs - no more, no less!

## Different Types of Data Variants

### 1. Tuple Variants (Unnamed Fields)

**Tuple variants** store data in unnamed fields, like tuples:

```rust
#[derive(Debug)]
enum VegetablePrep {
    Wash,                           // No data
    Chop(String),                   // Single field: cutting style
    Season(String, String),         // Two fields: spice, amount
    Cook(u32, String, u32),         // Three fields: time, method, temperature
}

impl VegetablePrep {
    fn get_description(&self) -> String {
        match self {
            VegetablePrep::Wash => {
                "Rinse thoroughly under cold water".to_string()
            }
            VegetablePrep::Chop(style) => {
                format!("Chop into {} pieces", style)
            }
            VegetablePrep::Season(spice, amount) => {
                format!("Add {} of {}", amount, spice)
            }
            VegetablePrep::Cook(time, method, temp) => {
                format!("{} for {} minutes at {}°F", method, time, temp)
            }
        }
    }

    fn estimated_time(&self) -> u32 {
        match self {
            VegetablePrep::Wash => 2,
            VegetablePrep::Chop(_) => 5,
            VegetablePrep::Season(_, _) => 1,
            VegetablePrep::Cook(time, _, _) => *time,
        }
    }
}

fn main() {
    let prep_steps = vec![
        VegetablePrep::Wash,
        VegetablePrep::Chop("small dice".to_string()),
        VegetablePrep::Season("salt".to_string(), "1 tsp".to_string()),
        VegetablePrep::Cook(15, "Sauté".to_string(), 350),
    ];

    let mut total_time = 0;

    for step in &prep_steps {
        let time = step.estimated_time();
        total_time += time;
        println!("{} ({}min)", step.get_description(), time);
    }

    println!("\nTotal prep time: {} minutes", total_time);
}
```


### 2. Struct Variants (Named Fields)

**Struct variants** use named fields for better clarity:

```rust
#[derive(Debug, Clone)]
enum MenuItemDetails {
    Appetizer {
        name: String,
        price: f64,
        serving_size: String,
        is_shareable: bool,
    },
    MainCourse {
        name: String,
        price: f64,
        protein_source: String,
        calories: u32,
        is_gluten_free: bool,
    },
    Dessert {
        name: String,
        price: f64,
        sweetness_level: u8, // 1-10 scale
        contains_nuts: bool,
    },
    Beverage {
        name: String,
        price: f64,
        size_oz: u32,
        is_hot: bool,
        caffeine_mg: Option<u32>,
    },
}

impl MenuItemDetails {
    fn get_name(&self) -> &str {
        match self {
            MenuItemDetails::Appetizer { name, .. } => name,
            MenuItemDetails::MainCourse { name, .. } => name,
            MenuItemDetails::Dessert { name, .. } => name,
            MenuItemDetails::Beverage { name, .. } => name,
        }
    }

    fn get_price(&self) -> f64 {
        match self {
            MenuItemDetails::Appetizer { price, .. } => *price,
            MenuItemDetails::MainCourse { price, .. } => *price,
            MenuItemDetails::Dessert { price, .. } => *price,
            MenuItemDetails::Beverage { price, .. } => *price,
        }
    }

    fn get_category(&self) -> &str {
        match self {
            MenuItemDetails::Appetizer { .. } => "Appetizer",
            MenuItemDetails::MainCourse { .. } => "Main Course",
            MenuItemDetails::Dessert { .. } => "Dessert",
            MenuItemDetails::Beverage { .. } => "Beverage",
        }
    }

    fn is_suitable_for_dietary_restriction(&self, restriction: &str) -> bool {
        match (self, restriction) {
            (MenuItemDetails::MainCourse { is_gluten_free: true, .. }, "gluten-free") => true,
            (MenuItemDetails::Dessert { contains_nuts: false, .. }, "nut-free") => true,
            (MenuItemDetails::Beverage { caffeine_mg: None, .. }, "caffeine-free") => true,
            (MenuItemDetails::Beverage { caffeine_mg: Some(0), .. }, "caffeine-free") => true,
            _ => false,
        }
    }

    fn format_for_menu(&self) -> String {
        match self {
            MenuItemDetails::Appetizer { name, price, serving_size, is_shareable } => {
                let share_note = if *is_shareable { " (perfect for sharing)" } else { "" };
                format!("{} - ${:.2}\n  Serving: {}{}", name, price, serving_size, share_note)
            }
            MenuItemDetails::MainCourse { name, price, protein_source, calories, is_gluten_free } => {
                let gf_note = if *is_gluten_free { " 🌾 Gluten-Free" } else { "" };
                format!("{} - ${:.2}\n  Protein: {} | {} calories{}", name, price, protein_source, calories, gf_note)
            }
            MenuItemDetails::Dessert { name, price, sweetness_level, contains_nuts } => {
                let sweetness = "🍯".repeat(*sweetness_level as usize / 2);
                let nut_warning = if *contains_nuts { " ⚠️ Contains nuts" } else { "" };
                format!("{} - ${:.2}\n  Sweetness: {}{}", name, price, sweetness, nut_warning)
            }
            MenuItemDetails::Beverage { name, price, size_oz, is_hot, caffeine_mg } => {
                let temp = if *is_hot { "☕ Hot" } else { "🧊 Cold" };
                let caffeine = caffeine_mg
                    .map(|mg| format!(" | {}mg caffeine", mg))
                    .unwrap_or(" | Caffeine-free".to_string());
                format!("{} - ${:.2}\n  {}oz {}{}", name, price, size_oz, temp, caffeine)
            }
        }
    }
}

fn main() {
    let menu_items = vec![
        MenuItemDetails::Appetizer {
            name: "Mediterranean Hummus Plate".to_string(),
            price: 9.95,
            serving_size: "generous portion".to_string(),
            is_shareable: true,
        },
        MenuItemDetails::MainCourse {
            name: "Quinoa Power Bowl".to_string(),
            price: 16.95,
            protein_source: "quinoa and black beans".to_string(),
            calories: 450,
            is_gluten_free: true,
        },
        MenuItemDetails::Dessert {
            name: "Chocolate Avocado Mousse".to_string(),
            price: 7.95,
            sweetness_level: 8,
            contains_nuts: false,
        },
        MenuItemDetails::Beverage {
            name: "Green Goddess Smoothie".to_string(),
            price: 5.95,
            size_oz: 16,
            is_hot: false,
            caffeine_mg: None,
        },
    ];

    println!("🌱 VEGETARIAN MENU 🌱");
    println!("{:=<50}", "");

    for item in &menu_items {
        println!("\n{}", item.format_for_menu());

        // Check dietary restrictions
        let mut notes = Vec::new();
        if item.is_suitable_for_dietary_restriction("gluten-free") {
            notes.push("Gluten-Free Friendly");
        }
        if item.is_suitable_for_dietary_restriction("nut-free") {
            notes.push("Nut-Free");
        }
        if item.is_suitable_for_dietary_restriction("caffeine-free") {
            notes.push("Caffeine-Free");
        }

        if !notes.is_empty() {
            println!("  ✨ {}", notes.join(" | "));
        }
    }

    println!("\n{:=<50}", "");

    // Calculate menu statistics
    let total_items = menu_items.len();
    let avg_price = menu_items.iter()
        .map(|item| item.get_price())
        .sum::<f64>() / total_items as f64;

    println!("Menu Summary: {} items, Average price: ${:.2}", total_items, avg_price);
}
```


### 3. Mixed Variants (Different Data Types)

**You can mix different types of variants in the same enum:**

```rust
#[derive(Debug)]
enum KitchenEvent {
    // Unit variant
    ShiftStart,

    // Tuple variants with different data
    OrderReceived(u32), // order ID
    IngredientLow(String, u32), // ingredient name, quantity remaining
    EquipmentMaintenance(String, String), // equipment, issue description

    // Struct variants
    CustomerComplaint {
        order_id: u32,
        issue: String,
        resolution_needed: bool,
    },

    StaffUpdate {
        employee_name: String,
        action: String, // "arrived", "break", "departed"
        timestamp: String,
    },

    // Complex tuple variant
    DeliveryArrival(String, Vec<String>, f64), // supplier, items, total cost
}

struct KitchenManager {
    events_log: Vec<KitchenEvent>,
}

impl KitchenManager {
    fn new() -> Self {
        KitchenManager {
            events_log: Vec::new(),
        }
    }

    fn log_event(&mut self, event: KitchenEvent) {
        println!("{}", self.format_event(&event));
        self.events_log.push(event);
    }

    fn format_event(&self, event: &KitchenEvent) -> String {
        match event {
            KitchenEvent::ShiftStart => {
                "🌅 Kitchen shift started - all stations ready!".to_string()
            }
            KitchenEvent::OrderReceived(order_id) => {
                format!("📋 New order received: #{}", order_id)
            }
            KitchenEvent::IngredientLow(ingredient, quantity) => {
                let urgency = if *quantity == 0 { "🚨 URGENT" } else { "⚠️ Warning" };
                format!("{} - {} running low: {} remaining", urgency, ingredient, quantity)
            }
            KitchenEvent::EquipmentMaintenance(equipment, issue) => {
                format!("🔧 Equipment issue: {} - {}", equipment, issue)
            }
            KitchenEvent::CustomerComplaint { order_id, issue, resolution_needed } => {
                let priority = if *resolution_needed { "HIGH PRIORITY" } else { "Standard" };
                format!("📞 Customer complaint for order #{} ({}): {}", order_id, priority, issue)
            }
            KitchenEvent::StaffUpdate { employee_name, action, timestamp } => {
                format!("👤 Staff update: {} {} at {}", employee_name, action, timestamp)
            }
            KitchenEvent::DeliveryArrival(supplier, items, cost) => {
                format!("🚚 Delivery from {}: {} items, ${:.2} total",
                       supplier, items.len(), cost)
            }
        }
    }

    fn get_urgent_events(&self) -> Vec<&KitchenEvent> {
        self.events_log.iter().filter(|event| {
            matches!(event,
                KitchenEvent::IngredientLow(_, 0) |
                KitchenEvent::EquipmentMaintenance(_, _) |
                KitchenEvent::CustomerComplaint { resolution_needed: true, .. }
            )
        }).collect()
    }

    fn count_orders_today(&self) -> usize {
        self.events_log.iter().filter(|event| {
            matches!(event, KitchenEvent::OrderReceived(_))
        }).count()
    }
}

fn main() {
    let mut kitchen = KitchenManager::new();

    // Simulate a day in the kitchen
    kitchen.log_event(KitchenEvent::ShiftStart);

    kitchen.log_event(KitchenEvent::StaffUpdate {
        employee_name: "Alice".to_string(),
        action: "arrived".to_string(),
        timestamp: "7:00 AM".to_string(),
    });

    kitchen.log_event(KitchenEvent::OrderReceived(1001));
    kitchen.log_event(KitchenEvent::OrderReceived(1002));

    kitchen.log_event(KitchenEvent::IngredientLow("quinoa".to_string(), 2));

    kitchen.log_event(KitchenEvent::DeliveryArrival(
        "Fresh Farms Co".to_string(),
        vec!["tomatoes".to_string(), "spinach".to_string(), "bell peppers".to_string()],
        89.50
    ));

    kitchen.log_event(KitchenEvent::EquipmentMaintenance(
        "Main Oven".to_string(),
        "Temperature not reaching set point".to_string()
    ));

    kitchen.log_event(KitchenEvent::CustomerComplaint {
        order_id: 1001,
        issue: "Food arrived cold".to_string(),
        resolution_needed: true,
    });

    // Check urgent events
    println!("\n🚨 URGENT ITEMS REQUIRING ATTENTION:");
    println!("{:-<50}", "");
    let urgent_events = kitchen.get_urgent_events();

    if urgent_events.is_empty() {
        println!("✅ No urgent items - kitchen running smoothly!");
    } else {
        for event in urgent_events {
            println!("• {}", kitchen.format_event(event));
        }
    }

    println!("\n📊 Day Summary: {} orders processed", kitchen.count_orders_today());
}
```


## Advanced Patterns with Data Variants

### 1. Result-Like Error Handling

```rust
#[derive(Debug)]
enum CookingResult<T> {
    Success(T),
    RecipeError {
        missing_ingredients: Vec<String>,
        suggested_substitutions: Vec<(String, String)>, // (missing, substitute)
    },
    EquipmentError {
        equipment: String,
        issue: String,
        workaround: Option<String>,
    },
    TimingError {
        step: String,
        expected_time: u32,
        actual_time: u32,
        salvageable: bool,
    },
    TemperatureError(u32, u32), // expected, actual
}

struct Recipe {
    name: String,
    ingredients: Vec<String>,
    steps: Vec<String>,
}

impl<T> CookingResult<T> {
    fn is_success(&self) -> bool {
        matches!(self, CookingResult::Success(_))
    }

    fn get_error_message(&self) -> Option<String> {
        match self {
            CookingResult::Success(_) => None,
            CookingResult::RecipeError { missing_ingredients, suggested_substitutions } => {
                let mut msg = format!("Missing ingredients: {}", missing_ingredients.join(", "));
                if !suggested_substitutions.is_empty() {
                    msg.push_str("\nSuggested substitutions:");
                    for (missing, substitute) in suggested_substitutions {
                        msg.push_str(&format!("\n  {} → {}", missing, substitute));
                    }
                }
                Some(msg)
            }
            CookingResult::EquipmentError { equipment, issue, workaround } => {
                let mut msg = format!("Equipment problem with {}: {}", equipment, issue);
                if let Some(workaround) = workaround {
                    msg.push_str(&format!("\nWorkaround: {}", workaround));
                }
                Some(msg)
            }
            CookingResult::TimingError { step, expected_time, actual_time, salvageable } => {
                let mut msg = format!("Timing issue at '{}': expected {}min, took {}min",
                                      step, expected_time, actual_time);
                if *salvageable {
                    msg.push_str(" (still salvageable)");
                } else {
                    msg.push_str(" (may be overcooked)");
                }
                Some(msg)
            }
            CookingResult::TemperatureError(expected, actual) => {
                Some(format!("Temperature error: expected {}°F, got {}°F", expected, actual))
            }
        }
    }
}

fn cook_recipe(recipe: Recipe, available_ingredients: &[String]) -> CookingResult<String> {
    // Check ingredients
    let missing: Vec<String> = recipe.ingredients.iter()
        .filter(|ingredient| !available_ingredients.contains(ingredient))
        .cloned()
        .collect();

    if !missing.is_empty() {
        let substitutions = vec![
            ("quinoa".to_string(), "brown rice".to_string()),
            ("cashews".to_string(), "sunflower seeds".to_string()),
        ];

        return CookingResult::RecipeError {
            missing_ingredients: missing,
            suggested_substitutions: substitutions,
        };
    }

    // Simulate cooking process with various possible errors
    use rand::Rng;
    let mut rng = rand::thread_rng();

    match rng.gen_range(0..5) {
        0 => CookingResult::Success(format!("{} cooked perfectly! 🎉", recipe.name)),
        1 => CookingResult::EquipmentError {
            equipment: "Oven".to_string(),
            issue: "Not heating properly".to_string(),
            workaround: Some("Use stovetop method instead".to_string()),
        },
        2 => CookingResult::TimingError {
            step: "Sautéing vegetables".to_string(),
            expected_time: 8,
            actual_time: 12,
            salvageable: true,
        },
        3 => CookingResult::TemperatureError(375, 325),
        _ => CookingResult::Success(format!("{} completed successfully!", recipe.name)),
    }
}

fn main() {
    let recipe = Recipe {
        name: "Mediterranean Quinoa Bowl".to_string(),
        ingredients: vec![
            "quinoa".to_string(),
            "olive oil".to_string(),
            "bell peppers".to_string(),
            "cherry tomatoes".to_string(),
            "feta cheese".to_string(),
        ],
        steps: vec![
            "Cook quinoa".to_string(),
            "Chop vegetables".to_string(),
            "Sauté vegetables".to_string(),
            "Combine and serve".to_string(),
        ],
    };

    let available_ingredients = vec![
        "quinoa".to_string(),
        "olive oil".to_string(),
        "bell peppers".to_string(),
        "cherry tomatoes".to_string(),
        // Missing feta cheese
    ];

    println!("🍳 Attempting to cook: {}", recipe.name);
    println!("{:=<50}", "");

    let result = cook_recipe(recipe, &available_ingredients);

    match result {
        CookingResult::Success(message) => {
            println!("✅ {}", message);
        }
        error => {
            println!("❌ Cooking failed!");
            if let Some(error_msg) = error.get_error_message() {
                println!("{}", error_msg);
            }

            // Provide specific advice based on error type
            match error {
                CookingResult::RecipeError { .. } => {
                    println!("\n💡 Try shopping for missing ingredients or use substitutions");
                }
                CookingResult::EquipmentError { .. } => {
                    println!("\n💡 Contact maintenance or try alternative cooking methods");
                }
                CookingResult::TimingError { salvageable: true, .. } => {
                    println!("\n💡 Dish is still good! Next time watch the timing more carefully");
                }
                CookingResult::TimingError { salvageable: false, .. } => {
                    println!("\n💡 Consider starting over with fresh ingredients");
                }
                CookingResult::TemperatureError(_, _) => {
                    println!("\n💡 Check oven calibration and adjust accordingly");
                }
                _ => {}
            }
        }
    }
}
```


### 2. State Machine with Complex Data

```rust
#[derive(Debug, Clone)]
enum OrderState {
    Draft {
        items: Vec<String>,
        notes: Vec<String>,
    },
    Submitted {
        order_id: u32,
        items: Vec<String>,
        total_price: f64,
        customer_info: CustomerInfo,
    },
    Confirmed {
        order_id: u32,
        estimated_prep_time: u32,
        assigned_chef: String,
    },
    InProgress {
        order_id: u32,
        current_step: String,
        progress_percentage: u8,
        start_time: String,
    },
    QualityCheck {
        order_id: u32,
        checker: String,
        issues_found: Vec<String>,
    },
    Ready {
        order_id: u32,
        completion_time: String,
        pickup_code: String,
    },
    Completed {
        order_id: u32,
        customer_rating: Option<u8>, // 1-5 stars
        feedback: Option<String>,
    },
    Cancelled {
        reason: String,
        refund_amount: f64,
        cancelled_at: String,
    },
}

#[derive(Debug, Clone)]
struct CustomerInfo {
    name: String,
    phone: String,
    dietary_restrictions: Vec<String>,
}

struct RestaurantOrderSystem {
    current_state: OrderState,
}

impl RestaurantOrderSystem {
    fn new() -> Self {
        RestaurantOrderSystem {
            current_state: OrderState::Draft {
                items: Vec::new(),
                notes: Vec::new(),
            },
        }
    }

    fn add_item(&mut self, item: String) -> Result<(), String> {
        match &mut self.current_state {
            OrderState::Draft { items, .. } => {
                items.push(item);
                Ok(())
            }
            _ => Err("Can only add items to draft orders".to_string()),
        }
    }

    fn add_note(&mut self, note: String) -> Result<(), String> {
        match &mut self.current_state {
            OrderState::Draft { notes, .. } => {
                notes.push(note);
                Ok(())
            }
            _ => Err("Can only add notes to draft orders".to_string()),
        }
    }

    fn submit_order(self, customer_info: CustomerInfo) -> Result<RestaurantOrderSystem, String> {
        match self.current_state {
            OrderState::Draft { items, .. } => {
                if items.is_empty() {
                    return Err("Cannot submit empty order".to_string());
                }

                let order_id = 12345; // In reality, generate unique ID
                let total_price = items.len() as f64 * 12.50; // Simplified pricing

                Ok(RestaurantOrderSystem {
                    current_state: OrderState::Submitted {
                        order_id,
                        items,
                        total_price,
                        customer_info,
                    },
                })
            }
            _ => Err("Can only submit draft orders".to_string()),
        }
    }

    fn confirm_order(self, chef: String) -> Result<RestaurantOrderSystem, String> {
        match self.current_state {
            OrderState::Submitted { order_id, items, .. } => {
                let estimated_prep_time = items.len() as u32 * 8; // 8 minutes per item

                Ok(RestaurantOrderSystem {
                    current_state: OrderState::Confirmed {
                        order_id,
                        estimated_prep_time,
                        assigned_chef: chef,
                    },
                })
            }
            _ => Err("Can only confirm submitted orders".to_string()),
        }
    }

    fn start_preparation(self) -> Result<RestaurantOrderSystem, String> {
        match self.current_state {
            OrderState::Confirmed { order_id, .. } => {
                Ok(RestaurantOrderSystem {
                    current_state: OrderState::InProgress {
                        order_id,
                        current_step: "Gathering ingredients".to_string(),
                        progress_percentage: 10,
                        start_time: "2:30 PM".to_string(), // Simplified timestamp
                    },
                })
            }
            _ => Err("Can only start preparation on confirmed orders".to_string()),
        }
    }

    fn update_progress(&mut self, step: String, percentage: u8) -> Result<(), String> {
        match &mut self.current_state {
            OrderState::InProgress { current_step, progress_percentage, .. } => {
                *current_step = step;
                *progress_percentage = percentage;
                Ok(())
            }
            _ => Err("Can only update progress on in-progress orders".to_string()),
        }
    }

    fn get_status_description(&self) -> String {
        match &self.current_state {
            OrderState::Draft { items, notes } => {
                format!("Draft order: {} items, {} notes", items.len(), notes.len())
            }
            OrderState::Submitted { order_id, total_price, customer_info, .. } => {
                format!("Order #{} submitted by {} - ${:.2}",
                       order_id, customer_info.name, total_price)
            }
            OrderState::Confirmed { order_id, estimated_prep_time, assigned_chef } => {
                format!("Order #{} confirmed - Chef: {}, ETA: {}min",
                       order_id, assigned_chef, estimated_prep_time)
            }
            OrderState::InProgress { order_id, current_step, progress_percentage, .. } => {
                format!("Order #{} in progress: {} ({}% complete)",
                       order_id, current_step, progress_percentage)
            }
            OrderState::QualityCheck { order_id, checker, issues_found } => {
                let issue_status = if issues_found.is_empty() {
                    "passed".to_string()
                } else {
                    format!("{} issues found", issues_found.len())
                };
                format!("Order #{} in quality check by {} - {}",
                       order_id, checker, issue_status)
            }
            OrderState::Ready { order_id, pickup_code, .. } => {
                format!("Order #{} ready for pickup! Code: {}", order_id, pickup_code)
            }
            OrderState::Completed { order_id, customer_rating, .. } => {
                let rating_text = customer_rating
                    .map(|r| format!("{} stars", r))
                    .unwrap_or("No rating yet".to_string());
                format!("Order #{} completed - {}", order_id, rating_text)
            }
            OrderState::Cancelled { reason, refund_amount, .. } => {
                format!("Order cancelled: {} (${:.2} refunded)", reason, refund_amount)
            }
        }
    }
}

fn main() -> Result<(), String> {
    println!("🍽️ Restaurant Order System Simulation");
    println!("{:=<60}", "");

    let mut order = RestaurantOrderSystem::new();
    println!("✅ Created new order: {}", order.get_status_description());

    // Add items to draft
    order.add_item("Quinoa Power Bowl".to_string())?;
    order.add_item("Green Goddess Smoothie".to_string())?;
    order.add_note("Extra avocado please".to_string())?;
    println!("✅ Added items: {}", order.get_status_description());

    // Submit order
    let customer = CustomerInfo {
        name: "Alice Johnson".to_string(),
        phone: "555-0123".to_string(),
        dietary_restrictions: vec!["gluten-free".to_string()],
    };
    let order = order.submit_order(customer)?;
    println!("✅ Submitted: {}", order.get_status_description());

    // Confirm order
    let order = order.confirm_order("Chef Mario".to_string())?;
    println!("✅ Confirmed: {}", order.get_status_description());

    // Start preparation
    let mut order = order.start_preparation()?;
    println!("✅ Started: {}", order.get_status_description());

    // Update progress
    order.update_progress("Cooking quinoa".to_string(), 30)?;
    println!("✅ Progress: {}", order.get_status_description());

    order.update_progress("Preparing smoothie".to_string(), 60)?;
    println!("✅ Progress: {}", order.get_status_description());

    order.update_progress("Final plating".to_string(), 90)?;
    println!("✅ Progress: {}", order.get_status_description());

    println!("\n{:=<60}", "");
    println!("🎉 Order workflow completed successfully!");

    Ok(())
}
```


## Best Practices for Enum Variants with Data

### 1. Choose Appropriate Data Structures

```rust
// ✅ Good - use struct variants for complex data with multiple fields
enum PaymentMethod {
    Cash,
    CreditCard {
        card_number: String,    // Last 4 digits only
        expiry_month: u8,
        expiry_year: u16,
        cardholder_name: String,
    },
    DigitalWallet {
        provider: String,       // "Apple Pay", "Google Pay", etc.
        account_email: String,
    },
    BankTransfer {
        routing_number: String,
        account_number: String, // Masked for security
        bank_name: String,
    },
}

// ❌ Avoid - tuple variants with too many unnamed fields
enum BadPaymentMethod {
    CreditCard(String, u8, u16, String), // Hard to remember what each field is
    BankTransfer(String, String, String), // Very confusing
}
```


### 2. Use Meaningful Field Names

```rust
// ✅ Good - descriptive field names
enum NetworkEvent {
    DataReceived {
        bytes_count: usize,
        source_address: String,
        timestamp: u64,
    },
    ConnectionLost {
        peer_id: String,
        error_code: u32,
        last_seen: u64,
        auto_reconnect: bool,
    },
}

// ❌ Bad - generic or unclear names
enum BadNetworkEvent {
    DataReceived {
        data: usize,    // Data what? Size? Content?
        addr: String,   // Abbreviations are unclear
        time: u64,      // Time of what?
    },
}
```


### 3. Handle All Cases in Pattern Matching

```rust
fn process_cooking_instruction(instruction: CookingInstruction) {
    match instruction {
        CookingInstruction::Preheat => {
            println!("Starting preheat sequence");
        }
        CookingInstruction::SetTimer(minutes) => {
            println!("Timer set for {} minutes", minutes);
        }
        CookingInstruction::SetTemperature(degrees, unit) => {
            println!("Temperature set to {}°{}", degrees, unit);
        }
        CookingInstruction::AddIngredient { name, amount, preparation } => {
            let prep_note = preparation
                .as_ref()
                .map(|p| format!(" ({})", p))
                .unwrap_or_default();
            println!("Adding {} {}{}", amount, name, prep_note);
        }
        CookingInstruction::CookFor(time, method, temp) => {
            println!("Cooking: {} for {}min at {}°F", method, time, temp);
        }
        // ✅ All variants handled - compiler ensures this
    }
}
```


### 4. Provide Helpful Methods

```rust
impl CookingInstruction {
    fn estimated_time_minutes(&self) -> u32 {
        match self {
            CookingInstruction::Preheat => 10,
            CookingInstruction::SetTimer(minutes) => *minutes,
            CookingInstruction::SetTemperature(_, _) => 2,
            CookingInstruction::AddIngredient { .. } => 3,
            CookingInstruction::CookFor(time, _, _) => *time,
        }
    }

    fn requires_active_monitoring(&self) -> bool {
        match self {
            CookingInstruction::Preheat => false,
            CookingInstruction::SetTimer(_) => false,
            CookingInstruction::SetTemperature(_, _) => false,
            CookingInstruction::AddIngredient { .. } => true,
            CookingInstruction::CookFor(_, method, _) => {
                method.to_lowercase().contains("sauté") ||
                method.to_lowercase().contains("stir")
            }
        }
    }

    fn get_equipment_needed(&self) -> Vec<&str> {
        match self {
            CookingInstruction::Preheat => vec!["oven"],
            CookingInstruction::SetTimer(_) => vec!["timer"],
            CookingInstruction::SetTemperature(_, _) => vec!["thermometer"],
            CookingInstruction::AddIngredient { .. } => vec!["measuring cups", "mixing bowl"],
            CookingInstruction::CookFor(_, method, _) => {
                match method.to_lowercase().as_str() {
                    s if s.contains("sauté") => vec!["pan", "spatula"],
                    s if s.contains("bake") => vec!["oven", "baking dish"],
                    s if s.contains("grill") => vec!["grill", "tongs"],
                    _ => vec!["cookware"],
                }
            }
        }
    }
}
```


## Common Mistakes to Avoid

### Mistake 1: Too Much Data in Variants

```rust
// ❌ Bad - cramming too much unrelated data
enum OverloadedOrder {
    DineIn(u32, u32, String, String, bool, f64, Vec<String>, String, u32),
    // What do all these fields mean? Impossible to remember!
}

// ✅ Good - organized, meaningful data
enum ClearOrder {
    DineIn {
        table_number: u32,
        guest_count: u32,
        server_name: String,
        special_requests: Vec<String>,
    },
}
```


### Mistake 2: Not Using Option for Optional Data

```rust
// ❌ Bad - using empty strings for missing data
enum BadEvent {
    UserAction {
        user_id: String,
        action: String,
        details: String, // Empty string when no details
    },
}

// ✅ Good - use Option for optional data
enum GoodEvent {
    UserAction {
        user_id: String,
        action: String,
        details: Option<String>, // None when no details
    },
}
```


### Mistake 3: Inconsistent Data Patterns

```rust
// ❌ Bad - inconsistent patterns across variants
enum InconsistentMessage {
    Success(String),              // Tuple variant
    Warning { message: String },  // Struct variant with one field
    Error(u32, String),          // Tuple variant with two fields
}

// ✅ Good - consistent patterns
enum ConsistentMessage {
    Success(String),
    Warning(String),
    Error { code: u32, message: String }, // Struct when multiple fields
}
```


## Summary and Key Takeaways

### **Mental Model: The Specialized Order Tickets**

Remember the restaurant order system analogy:

- 🎫 **Each order type** = Enum variant (dine-in, takeout, delivery)
- 📝 **Order details** = Associated data (table number, address, phone)
- 👨💼 **Order processor** = Pattern matching (handles each type appropriately)
- 🍽️ **Kitchen workflow** = Methods on enums (process, estimate time, etc.)


### **Essential Principles**

1. **Each variant carries relevant data** - Only include data that makes sense for that variant
2. **Use appropriate data structures** - Struct variants for complex data, tuple variants for simple data
3. **Pattern matching extracts data** - Use `match` to access and use the associated data
4. **Type safety prevents errors** - Impossible to mix up data from different variants
5. **Methods provide behavior** - Add functionality through `impl` blocks

### **Variant Data Types Quick Reference**

| Pattern | When to Use | Example |
| :-- | :-- | :-- |
| **Unit variant** | No extra data needed | `enum Status { Ready, Busy }` |
| **Tuple variant** | Simple data, few fields | `enum Size(u32, u32)` |
| **Struct variant** | Complex data, many fields | `enum Event { Click { x: i32, y: i32, button: String } }` |
| **Mixed variants** | Different variants need different data | Combine unit, tuple, and struct variants as needed |

### **Pattern Matching with Data**

```rust
// Extract data from variants
match cooking_instruction {
    CookingInstruction::SetTimer(minutes) => {
        // Use minutes directly
        println!("Timer: {} min", minutes);
    }
    CookingInstruction::AddIngredient { name, amount, preparation } => {
        // Use named fields
        println!("Add {} {}", amount, name);
        if let Some(prep) = preparation {
            println!("Preparation: {}", prep);
        }
    }
    CookingInstruction::CookFor(time, method, temp) => {
        // Use multiple tuple fields
        println!("{} for {}min at {}°F", method, time, temp);
    }
    _ => {} // Handle other variants
}
```


### **Design Guidelines Checklist**

**✅ Well-Designed Enum with Data:**

- Each variant has a clear, specific purpose
- Associated data is relevant to the variant
- Uses appropriate data structures (struct vs tuple)
- Field names are descriptive and clear
- Provides helpful methods via `impl` blocks
- Handles all cases in pattern matching

**❌ Poorly-Designed Enum with Data:**

- Variants carry unrelated or excessive data
- Uses tuple variants for complex data
- Field names are unclear or abbreviated
- Missing essential functionality
- Incomplete pattern matching


### **Real-World Benefits**

**Enum variants with data excel at:**

- 🎯 **Type-safe state management** - Each state carries its own data
- 🔄 **Event handling systems** - Different events with different payloads
- 📨 **Message passing** - Different message types with specific data
- 🎮 **Game programming** - Different game states with state-specific data
- 🌐 **API responses** - Success/error cases with appropriate data
- 📊 **Data parsing** - Different data formats with structure-specific fields

**Enum variants with data are like having a smart filing system** in your vegetarian restaurant - each type of document (order, invoice, recipe) goes in the right folder with exactly the information it needs. This keeps everything organized, prevents mix-ups, and makes it easy to find and use the right information when you need it. The Rust compiler acts like a meticulous office manager, ensuring you never put the wrong paperwork in the wrong folder!

1. https://www.programiz.com/rust/enum
2. https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html
3. https://doc.rust-lang.org/rust-by-example/custom_types/enum.html
4. https://www.reddit.com/r/rust/comments/11fi2cb/enum_variants/
5. https://www.w3schools.com/rust/rust_enums.php
6. https://www.risein.com/courses/rust-programming/rust-programming-enum
7. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch06-01-defining-an-enum.html
8. https://stackoverflow.com/questions/51429501/how-do-i-conditionally-check-if-an-enum-is-one-variant-or-another
9. https://www.codecademy.com/resources/docs/rust/enums
10. https://stackoverflow.com/questions/78335374/how-to-write-doc-comments-for-enum-variants-with-multiple-data-holders-in-rust
11. https://stackoverflow.com/questions/36928569/how-can-i-create-enums-with-constant-values-in-rust
12. https://stackoverflow.com/questions/68540653/assign-associated-non-mutable-constant-data-to-enum-variants
13. https://users.rust-lang.org/t/checking-for-an-enum-variant/51558
14. https://doc.rust-lang.org/reference/items/enumerations.html
15. https://www.reddit.com/r/rust/comments/k5nlqa/why_would_anyone_ever_put_data_inside_an_enum/
16. https://serokell.io/blog/enums-and-pattern-matching
17. https://github.com/rustwasm/wasm-bindgen/issues/2407
18. https://users.rust-lang.org/t/enum-with-values-with-associated-enum/91632
19. https://stackoverflow.com/questions/75448510/associated-types-on-enum-variants-in-rust
20. https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations/
