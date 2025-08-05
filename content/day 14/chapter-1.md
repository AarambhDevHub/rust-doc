+++
title = "Defining Enums"
description = "Learn how to define enums in Rust, enabling expressive and type-safe representation of multiple possible values in your applications."
date = 2025-08-05
draft = false
weight = 1
+++

# Defining Enums in Rust: Comprehensive Documentation for Beginners

Understanding enums is like learning to **create a vegetarian restaurant menu with different meal categories** - each category (enum variant) represents a distinct type of dish you can offer, but each menu item can only be one specific type at a time. Just as a "Mediterranean Bowl" can be an appetizer OR a main course but never both simultaneously, an enum value can only be one variant at a specific moment.

## The Vegetarian Restaurant Menu Analogy 🍽️

### Imagine You're Designing a Restaurant Menu System

**Traditional Approach (Using Multiple Variables):**

```rust
// ❌ Confusing - multiple variables that could conflict
struct MenuItem {
    name: String,
    is_appetizer: bool,
    is_main_course: bool,
    is_dessert: bool,
    is_beverage: bool,
    // What if multiple booleans are true? Chaos!
}
```

**Enum Approach (Clean Menu Categories):**

```rust
// ✅ Crystal clear - each item has exactly one category
#[derive(Debug)]
enum MenuCategory {
    Appetizer,
    MainCourse,
    Dessert,
    Beverage,
    DailySpecial,
}

struct MenuItem {
    name: String,
    category: MenuCategory,  // Can only be ONE category
    price: f64,
}

fn main() {
    let hummus_plate = MenuItem {
        name: "Mediterranean Hummus Plate".to_string(),
        category: MenuCategory::Appetizer,  // Clearly an appetizer
        price: 8.95,
    };

    println!("Item: {} is a {:?}", hummus_plate.name, hummus_plate.category);
}
```

**Why This Works Better:**

- 📋 **Clear categorization** - Each item has exactly one category
- 🚫 **Impossible invalid states** - Can't be both appetizer AND dessert
- 🧠 **Easy to understand** - Mirrors real-world menu organization
- ✅ **Compiler-enforced** - Rust prevents inconsistent states


## What are Enums?

### Core Concept

**Enums** (short for "enumerations") define a type that can be one of several distinct variants. Each variant represents a possible value the enum can hold, but an enum instance can only be one variant at any given time.


| Topic | Explanation |
| :-- | :-- |
| **Basic Definition** | An enum (short for enumeration) is a type that can be one of several variants. Each variant represents a distinct possibility of the type, and the enum value can only be one variant at a time. |
| **Enums vs Structs** | Structs group related data, while enums represent one value out of several possible variants. |
| **Matching on Enums** | Rust's match statement allows you to branch behavior depending on the enum variant. |
| **Enums with Data** | Variants can also hold associated data, like enum Message { Quit, Move { x: i32, y: i32 }, Write(String) } |
| **Use Cases** | Enums are perfect for representing states, commands, events, options, results (like success / error), and more. |
| **Advantages** | Enums enable safe, clear representation of variant data, avoiding unsafe nulls and enhancing compiler guarantees. |

### The Cooking Method Analogy

Think of enums like different cooking methods for vegetables:

```rust
#[derive(Debug)]
enum CookingMethod {
    Raw,           // No cooking at all
    Steamed,       // Steam cooking
    Roasted,       // Oven roasted
    Grilled,       // Grilled on BBQ
    Sauteed,       // Pan sautéed
}

fn describe_cooking(method: CookingMethod) {
    match method {
        CookingMethod::Raw => println!("🥗 Fresh and crisp - perfect for salads!"),
        CookingMethod::Steamed => println!("♨️ Gentle and healthy - preserves nutrients!"),
        CookingMethod::Roasted => println!("🔥 Caramelized and flavorful!"),
        CookingMethod::Grilled => println!("🍖 Smoky BBQ taste!"),
        CookingMethod::Sauteed => println!("🍳 Quick and delicious!"),
    }
}

fn main() {
    let broccoli_prep = CookingMethod::Steamed;
    let carrot_prep = CookingMethod::Roasted;

    describe_cooking(broccoli_prep);
    describe_cooking(carrot_prep);
}
```

A vegetable can be prepared using ONE cooking method at a time - it can't be simultaneously raw AND roasted!

## Basic Enum Definition and Usage

### Simple Enums (Unit Variants)

Let's start with the simplest type of enum:

```rust
// Define different types of vegetarian diets
#[derive(Debug, Clone, PartialEq)]
enum VegetarianType {
    Lacto,          // Dairy allowed
    Ovo,            // Eggs allowed
    LactoOvo,       // Both dairy and eggs allowed
    Vegan,          // No animal products
    Pescatarian,    // Fish allowed
    Flexitarian,    // Mostly vegetarian
}

fn get_diet_description(diet_type: VegetarianType) -> &'static str {
    match diet_type {
        VegetarianType::Lacto => "Includes dairy products but no eggs",
        VegetarianType::Ovo => "Includes eggs but no dairy products",
        VegetarianType::LactoOvo => "Includes both dairy and eggs",
        VegetarianType::Vegan => "No animal products whatsoever",
        VegetarianType::Pescatarian => "Includes fish and seafood",
        VegetarianType::Flexitarian => "Mostly vegetarian with occasional meat",
    }
}

fn can_eat_cheese(diet_type: &VegetarianType) -> bool {
    match diet_type {
        VegetarianType::Lacto | VegetarianType::LactoOvo | VegetarianType::Flexitarian => true,
        _ => false,
    }
}

fn main() {
    let my_diet = VegetarianType::Vegan;
    let friend_diet = VegetarianType::LactoOvo;

    println!("My diet: {}", get_diet_description(my_diet.clone()));
    println!("Friend's diet: {}", get_diet_description(friend_diet.clone()));

    println!("Can I eat cheese? {}", can_eat_cheese(&my_diet));
    println!("Can my friend eat cheese? {}", can_eat_cheese(&friend_diet));

    // Compare diets
    if my_diet == VegetarianType::Vegan {
        println!("I'm following a strict vegan diet! 🌱");
    }
}
```


### Restaurant Order Status System

```rust
#[derive(Debug, Clone)]
enum OrderStatus {
    Pending,      // Order just placed
    Confirmed,    // Kitchen acknowledged
    Preparing,    // Currently cooking
    Ready,        // Ready for pickup/serving
    Delivered,    // Served to customer
    Cancelled,    // Order cancelled
}

struct RestaurantOrder {
    order_id: u32,
    dish_name: String,
    status: OrderStatus,
}

impl RestaurantOrder {
    fn new(order_id: u32, dish_name: String) -> Self {
        RestaurantOrder {
            order_id,
            dish_name,
            status: OrderStatus::Pending,
        }
    }

    fn update_status(&mut self, new_status: OrderStatus) {
        let old_status = self.status.clone();
        self.status = new_status;

        println!("Order #{}: {} - Status changed from {:?} to {:?}",
                 self.order_id, self.dish_name, old_status, self.status);
    }

    fn get_estimated_wait_time(&self) -> &str {
        match self.status {
            OrderStatus::Pending => "Processing your order...",
            OrderStatus::Confirmed => "15-20 minutes",
            OrderStatus::Preparing => "10-15 minutes",
            OrderStatus::Ready => "Ready now!",
            OrderStatus::Delivered => "Enjoy your meal!",
            OrderStatus::Cancelled => "Order was cancelled",
        }
    }

    fn can_be_cancelled(&self) -> bool {
        match self.status {
            OrderStatus::Pending | OrderStatus::Confirmed => true,
            _ => false,
        }
    }
}

fn main() {
    let mut order = RestaurantOrder::new(
        101,
        "Quinoa Buddha Bowl".to_string()
    );

    println!("Wait time: {}", order.get_estimated_wait_time());

    // Simulate order progression
    order.update_status(OrderStatus::Confirmed);
    println!("Wait time: {}", order.get_estimated_wait_time());

    order.update_status(OrderStatus::Preparing);
    println!("Wait time: {}", order.get_estimated_wait_time());

    order.update_status(OrderStatus::Ready);
    println!("Wait time: {}", order.get_estimated_wait_time());

    println!("Can cancel order: {}", order.can_be_cancelled());
}
```


## Enums with Data (Associated Data)

### The Recipe Instruction System

Sometimes enum variants need to carry additional information:

```rust
#[derive(Debug, Clone)]
enum CookingInstruction {
    Preheat(u32),                    // Temperature in Fahrenheit
    Chop(String),                    // What to chop
    Mix(Vec<String>),                // Ingredients to mix
    Cook { time_minutes: u32, method: String }, // Cooking time and method
    Season(String, String),          // Ingredient and amount
    Rest(u32),                       // Rest time in minutes
    Serve(u32),                      // Number of servings
}

struct Recipe {
    name: String,
    instructions: Vec<CookingInstruction>,
}

impl Recipe {
    fn new(name: String) -> Self {
        Recipe {
            name,
            instructions: Vec::new(),
        }
    }

    fn add_instruction(mut self, instruction: CookingInstruction) -> Self {
        self.instructions.push(instruction);
        self
    }

    fn execute_recipe(&self) {
        println!("\n🍳 Cooking: {}", self.name);
        println!("{:=<50}", "");

        for (step, instruction) in self.instructions.iter().enumerate() {
            print!("Step {}: ", step + 1);

            match instruction {
                CookingInstruction::Preheat(temp) => {
                    println!("🔥 Preheat oven to {}°F", temp);
                }
                CookingInstruction::Chop(ingredient) => {
                    println!("🔪 Chop the {}", ingredient);
                }
                CookingInstruction::Mix(ingredients) => {
                    println!("🥄 Mix together: {}", ingredients.join(", "));
                }
                CookingInstruction::Cook { time_minutes, method } => {
                    println!("👨‍🍳 {} for {} minutes", method, time_minutes);
                }
                CookingInstruction::Season(ingredient, amount) => {
                    println!("🧂 Add {} of {}", amount, ingredient);
                }
                CookingInstruction::Rest(minutes) => {
                    println!("⏰ Let rest for {} minutes", minutes);
                }
                CookingInstruction::Serve(servings) => {
                    println!("🍽️ Serve immediately to {} people", servings);
                }
            }
        }
        println!("{:=<50}", "");
    }

    fn total_cooking_time(&self) -> u32 {
        let mut total_time = 0;

        for instruction in &self.instructions {
            match instruction {
                CookingInstruction::Cook { time_minutes, .. } => {
                    total_time += time_minutes;
                }
                CookingInstruction::Rest(minutes) => {
                    total_time += minutes;
                }
                _ => {
                    total_time += 2; // Assume 2 minutes for each other step
                }
            }
        }

        total_time
    }
}

fn main() {
    let roasted_veggie_bowl = Recipe::new("Mediterranean Roasted Vegetable Bowl".to_string())
        .add_instruction(CookingInstruction::Preheat(425))
        .add_instruction(CookingInstruction::Chop("zucchini, bell peppers, and red onion".to_string()))
        .add_instruction(CookingInstruction::Season("olive oil".to_string(), "3 tablespoons".to_string()))
        .add_instruction(CookingInstruction::Season("salt and pepper".to_string(), "to taste".to_string()))
        .add_instruction(CookingInstruction::Mix(vec![
            "chopped vegetables".to_string(),
            "olive oil".to_string(),
            "seasonings".to_string()
        ]))
        .add_instruction(CookingInstruction::Cook {
            time_minutes: 25,
            method: "Roast in oven".to_string()
        })
        .add_instruction(CookingInstruction::Rest(5))
        .add_instruction(CookingInstruction::Serve(4));

    roasted_veggie_bowl.execute_recipe();
    println!("\n⏱️ Total time needed: {} minutes", roasted_veggie_bowl.total_cooking_time());
}
```


### User Input and Commands

```rust
#[derive(Debug)]
enum UserCommand {
    AddItem(String, f64),           // name, price
    RemoveItem(String),             // name
    UpdatePrice(String, f64),       // name, new_price
    ViewMenu,
    ViewItem(String),               // name
    Exit,
}

struct RestaurantMenu {
    items: std::collections::HashMap<String, f64>,
}

impl RestaurantMenu {
    fn new() -> Self {
        RestaurantMenu {
            items: std::collections::HashMap::new(),
        }
    }

    fn execute_command(&mut self, command: UserCommand) {
        match command {
            UserCommand::AddItem(name, price) => {
                self.items.insert(name.clone(), price);
                println!("✅ Added '{}' for ${:.2}", name, price);
            }
            UserCommand::RemoveItem(name) => {
                if self.items.remove(&name).is_some() {
                    println!("🗑️ Removed '{}'", name);
                } else {
                    println!("❌ '{}' not found in menu", name);
                }
            }
            UserCommand::UpdatePrice(name, new_price) => {
                if let Some(price) = self.items.get_mut(&name) {
                    let old_price = *price;
                    *price = new_price;
                    println!("💰 Updated '{}' price from ${:.2} to ${:.2}",
                             name, old_price, new_price);
                } else {
                    println!("❌ '{}' not found in menu", name);
                }
            }
            UserCommand::ViewMenu => {
                println!("\n📋 Current Menu:");
                println!("{:-<40}", "");
                if self.items.is_empty() {
                    println!("Menu is empty");
                } else {
                    for (name, price) in &self.items {
                        println!("{:<25} ${:>8.2}", name, price);
                    }
                }
                println!("{:-<40}", "");
            }
            UserCommand::ViewItem(name) => {
                if let Some(price) = self.items.get(&name) {
                    println!("🍽️ {}: ${:.2}", name, price);
                } else {
                    println!("❌ '{}' not found in menu", name);
                }
            }
            UserCommand::Exit => {
                println!("👋 Thanks for using the menu system!");
            }
        }
    }
}

fn main() {
    let mut menu = RestaurantMenu::new();

    // Simulate user commands
    let commands = vec![
        UserCommand::AddItem("Quinoa Salad".to_string(), 12.95),
        UserCommand::AddItem("Veggie Burger".to_string(), 14.50),
        UserCommand::AddItem("Mushroom Risotto".to_string(), 16.95),
        UserCommand::ViewMenu,
        UserCommand::UpdatePrice("Veggie Burger".to_string(), 13.95),
        UserCommand::ViewItem("Quinoa Salad".to_string()),
        UserCommand::RemoveItem("Mushroom Risotto".to_string()),
        UserCommand::ViewMenu,
        UserCommand::Exit,
    ];

    for command in commands {
        println!("\nExecuting: {:?}", command);
        menu.execute_command(command);
    }
}
```


## Advanced Enum Patterns

### Result-Like Error Handling

```rust
#[derive(Debug)]
enum CookingResult {
    Success(String),                    // Success message
    BurnedFood(String),                // What burned
    MissingIngredient(String),         // What's missing
    EquipmentFailure(String),          // What equipment failed
    TimeOut,                           // Took too long
}

struct VegetarianKitchen {
    ingredients: Vec<String>,
    equipment_working: bool,
}

impl VegetarianKitchen {
    fn new() -> Self {
        VegetarianKitchen {
            ingredients: vec![
                "quinoa".to_string(),
                "bell peppers".to_string(),
                "olive oil".to_string(),
                "salt".to_string(),
            ],
            equipment_working: true,
        }
    }

    fn cook_dish(&self, dish: &str, required_ingredients: Vec<&str>, cook_time: u32) -> CookingResult {
        // Check if equipment is working
        if !self.equipment_working {
            return CookingResult::EquipmentFailure("Oven is not working".to_string());
        }

        // Check if we have all ingredients
        for ingredient in &required_ingredients {
            if !self.ingredients.contains(&ingredient.to_string()) {
                return CookingResult::MissingIngredient(ingredient.to_string());
            }
        }

        // Simulate cooking based on time
        match cook_time {
            0..=30 => CookingResult::Success(format!("{} cooked perfectly!", dish)),
            31..=60 => {
                if cook_time > 45 {
                    CookingResult::BurnedFood(dish.to_string())
                } else {
                    CookingResult::Success(format!("{} cooked well!", dish))
                }
            },
            _ => CookingResult::TimeOut,
        }
    }
}

fn handle_cooking_result(result: CookingResult) {
    match result {
        CookingResult::Success(message) => {
            println!("🎉 {}", message);
        }
        CookingResult::BurnedFood(dish) => {
            println!("🔥 Oh no! The {} burned! Try lower heat next time.", dish);
        }
        CookingResult::MissingIngredient(ingredient) => {
            println!("🛒 We need to buy {} before we can cook this dish.", ingredient);
        }
        CookingResult::EquipmentFailure(problem) => {
            println!("🔧 Equipment problem: {}", problem);
        }
        CookingResult::TimeOut => {
            println!("⏰ This dish is taking too long - something went wrong!");
        }
    }
}

fn main() {
    let kitchen = VegetarianKitchen::new();

    // Try cooking different dishes with different outcomes
    let dishes = vec![
        ("Quinoa Salad", vec!["quinoa", "bell peppers", "olive oil"], 15),
        ("Pasta Primavera", vec!["pasta", "bell peppers"], 25), // Missing pasta
        ("Roasted Vegetables", vec!["bell peppers", "olive oil"], 50), // Will burn
        ("Complex Stew", vec!["quinoa", "bell peppers"], 90), // Too long
    ];

    for (dish, ingredients, time) in dishes {
        println!("\n👨‍🍳 Attempting to cook: {}", dish);
        let result = kitchen.cook_dish(dish, ingredients, time);
        handle_cooking_result(result);
    }
}
```


### State Machine with Enums

```rust
#[derive(Debug, Clone)]
enum OrderState {
    Draft { items: Vec<String> },
    Submitted { order_id: u32, items: Vec<String> },
    InKitchen { order_id: u32, estimated_time: u32 },
    Ready { order_id: u32, pickup_code: String },
    Completed { order_id: u32 },
    Cancelled { reason: String },
}

struct Order {
    state: OrderState,
}

impl Order {
    fn new() -> Self {
        Order {
            state: OrderState::Draft { items: Vec::new() },
        }
    }

    fn add_item(&mut self, item: String) -> Result<(), String> {
        match &mut self.state {
            OrderState::Draft { items } => {
                items.push(item);
                Ok(())
            }
            _ => Err("Can only add items to draft orders".to_string()),
        }
    }

    fn submit(self) -> Result<Order, String> {
        match self.state {
            OrderState::Draft { items } => {
                if items.is_empty() {
                    return Err("Cannot submit empty order".to_string());
                }

                let order_id = 12345; // In reality, this would be generated
                Ok(Order {
                    state: OrderState::Submitted { order_id, items },
                })
            }
            _ => Err("Can only submit draft orders".to_string()),
        }
    }

    fn start_cooking(self) -> Result<Order, String> {
        match self.state {
            OrderState::Submitted { order_id, items } => {
                let estimated_time = items.len() as u32 * 10; // 10 minutes per item
                Ok(Order {
                    state: OrderState::InKitchen { order_id, estimated_time },
                })
            }
            _ => Err("Can only start cooking submitted orders".to_string()),
        }
    }

    fn mark_ready(self) -> Result<Order, String> {
        match self.state {
            OrderState::InKitchen { order_id, .. } => {
                let pickup_code = format!("PICKUP{}", order_id);
                Ok(Order {
                    state: OrderState::Ready { order_id, pickup_code },
                })
            }
            _ => Err("Can only mark kitchen orders as ready".to_string()),
        }
    }

    fn complete(self) -> Result<Order, String> {
        match self.state {
            OrderState::Ready { order_id, .. } => {
                Ok(Order {
                    state: OrderState::Completed { order_id },
                })
            }
            _ => Err("Can only complete ready orders".to_string()),
        }
    }

    fn cancel(self, reason: String) -> Order {
        Order {
            state: OrderState::Cancelled { reason },
        }
    }

    fn get_status(&self) -> String {
        match &self.state {
            OrderState::Draft { items } => {
                format!("Draft order with {} items", items.len())
            }
            OrderState::Submitted { order_id, .. } => {
                format!("Order #{} submitted, waiting for kitchen", order_id)
            }
            OrderState::InKitchen { order_id, estimated_time } => {
                format!("Order #{} in kitchen, ready in {} minutes", order_id, estimated_time)
            }
            OrderState::Ready { order_id, pickup_code } => {
                format!("Order #{} ready! Pickup code: {}", order_id, pickup_code)
            }
            OrderState::Completed { order_id } => {
                format!("Order #{} completed", order_id)
            }
            OrderState::Cancelled { reason } => {
                format!("Order cancelled: {}", reason)
            }
        }
    }
}

fn main() -> Result<(), String> {
    let mut order = Order::new();
    println!("Status: {}", order.get_status());

    // Add items to draft
    order.add_item("Quinoa Bowl".to_string())?;
    order.add_item("Green Smoothie".to_string())?;
    println!("Status: {}", order.get_status());

    // Submit the order
    let order = order.submit()?;
    println!("Status: {}", order.get_status());

    // Start cooking
    let order = order.start_cooking()?;
    println!("Status: {}", order.get_status());

    // Mark as ready
    let order = order.mark_ready()?;
    println!("Status: {}", order.get_status());

    // Complete the order
    let order = order.complete()?;
    println!("Status: {}", order.get_status());

    Ok(())
}
```


## Implementing Methods on Enums

### Enums with Associated Methods

```rust
#[derive(Debug, Clone)]
enum VegetablePrepMethod {
    Raw,
    Blanched(u32),              // time in seconds
    Steamed(u32),               // time in minutes
    Roasted { temp: u32, time: u32 },  // temperature and time
    Grilled(String),            // marinade type
    Fermented(u32),             // days of fermentation
}

impl VegetablePrepMethod {
    // Constructor methods
    fn quick_blanch() -> Self {
        VegetablePrepMethod::Blanched(30)
    }

    fn standard_steam() -> Self {
        VegetablePrepMethod::Steamed(8)
    }

    fn high_heat_roast() -> Self {
        VegetablePrepMethod::Roasted { temp: 425, time: 25 }
    }

    fn teriyaki_grill() -> Self {
        VegetablePrepMethod::Grilled("teriyaki".to_string())
    }

    // Query methods
    fn requires_heat(&self) -> bool {
        match self {
            VegetablePrepMethod::Raw => false,
            VegetablePrepMethod::Fermented(_) => false,
            _ => true,
        }
    }

    fn estimated_time_minutes(&self) -> u32 {
        match self {
            VegetablePrepMethod::Raw => 0,
            VegetablePrepMethod::Blanched(seconds) => (seconds + 59) / 60, // Round up
            VegetablePrepMethod::Steamed(minutes) => *minutes,
            VegetablePrepMethod::Roasted { time, .. } => *time,
            VegetablePrepMethod::Grilled(_) => 15, // Default grill time
            VegetablePrepMethod::Fermented(days) => days * 24 * 60, // Convert days to minutes
        }
    }

    fn complexity_level(&self) -> &str {
        match self {
            VegetablePrepMethod::Raw => "Beginner",
            VegetablePrepMethod::Blanched(_) | VegetablePrepMethod::Steamed(_) => "Easy",
            VegetablePrepMethod::Roasted { .. } | VegetablePrepMethod::Grilled(_) => "Intermediate",
            VegetablePrepMethod::Fermented(_) => "Advanced",
        }
    }

    fn get_instructions(&self) -> String {
        match self {
            VegetablePrepMethod::Raw => {
                "Wash thoroughly and serve fresh".to_string()
            }
            VegetablePrepMethod::Blanched(seconds) => {
                format!("Boil water, add vegetables for {} seconds, then ice bath", seconds)
            }
            VegetablePrepMethod::Steamed(minutes) => {
                format!("Steam vegetables for {} minutes until tender", minutes)
            }
            VegetablePrepMethod::Roasted { temp, time } => {
                format!("Preheat oven to {}°F, roast for {} minutes", temp, time)
            }
            VegetablePrepMethod::Grilled(marinade) => {
                format!("Marinate with {} sauce, grill over medium heat", marinade)
            }
            VegetablePrepMethod::Fermented(days) => {
                format!("Ferment vegetables for {} days in salt brine", days)
            }
        }
    }

    fn modify_time(&mut self, new_time: u32) {
        match self {
            VegetablePrepMethod::Blanched(ref mut seconds) => *seconds = new_time,
            VegetablePrepMethod::Steamed(ref mut minutes) => *minutes = new_time,
            VegetablePrepMethod::Roasted { ref mut time, .. } => *time = new_time,
            VegetablePrepMethod::Fermented(ref mut days) => *days = new_time,
            _ => println!("Cannot modify time for this preparation method"),
        }
    }
}

struct VegetableDish {
    name: String,
    vegetables: Vec<String>,
    prep_method: VegetablePrepMethod,
}

impl VegetableDish {
    fn new(name: String, vegetables: Vec<String>, prep_method: VegetablePrepMethod) -> Self {
        VegetableDish {
            name,
            vegetables,
            prep_method,
        }
    }

    fn get_cooking_summary(&self) -> String {
        format!(
            "🥗 {}\n🥕 Vegetables: {}\n🔥 Prep: {:?}\n⏰ Time: {} minutes\n📚 Difficulty: {}\n📝 Instructions: {}",
            self.name,
            self.vegetables.join(", "),
            self.prep_method,
            self.prep_method.estimated_time_minutes(),
            self.prep_method.complexity_level(),
            self.prep_method.get_instructions()
        )
    }
}

fn main() {
    let mut dishes = vec![
        VegetableDish::new(
            "Fresh Garden Salad".to_string(),
            vec!["lettuce".to_string(), "tomatoes".to_string(), "cucumbers".to_string()],
            VegetablePrepMethod::Raw,
        ),
        VegetableDish::new(
            "Perfect Broccoli".to_string(),
            vec!["broccoli".to_string()],
            VegetablePrepMethod::quick_blanch(),
        ),
        VegetableDish::new(
            "Roasted Root Vegetables".to_string(),
            vec!["carrots".to_string(), "parsnips".to_string(), "sweet potatoes".to_string()],
            VegetablePrepMethod::high_heat_roast(),
        ),
        VegetableDish::new(
            "Grilled Zucchini".to_string(),
            vec!["zucchini".to_string(), "yellow squash".to_string()],
            VegetablePrepMethod::teriyaki_grill(),
        ),
    ];

    println!("🍽️ Vegetarian Dish Preparation Guide");
    println!("{:=<60}", "");

    for (i, dish) in dishes.iter_mut().enumerate() {
        println!("\nDish #{}: ", i + 1);
        println!("{}", dish.get_cooking_summary());

        // Demonstrate method modification
        if i == 1 {
            println!("\n🔄 Adjusting blanching time...");
            dish.prep_method.modify_time(60);
            println!("Updated time: {} minutes", dish.prep_method.estimated_time_minutes());
        }

        println!("{:-<60}", "");
    }
}
```


## Best Practices for Enum Design

### 1. Use Descriptive Names

```rust
// ❌ Poor naming - unclear what these represent
enum Status {
    A, B, C, D,
}

enum Thing {
    Good, Bad, Maybe,
}

// ✅ Clear, descriptive naming
enum OrderStatus {
    Pending, Processing, Shipped, Delivered, Cancelled,
}

enum PaymentMethod {
    Cash, CreditCard, DebitCard, DigitalWallet, BankTransfer,
}
```


### 2. Group Related Variants Logically

```rust
// ✅ Well-organized dietary preferences
#[derive(Debug, Clone)]
enum DietaryRestriction {
    // Animal products
    Vegetarian,
    Vegan,

    // Allergens
    NutFree,
    GlutenFree,
    DairyFree,
    SoyFree,

    // Health-focused
    LowSodium,
    LowCarb,
    Keto,

    // Religious/Cultural
    Halal,
    Kosher,
    Jain,
}
```


### 3. Use Data Appropriately

```rust
// ✅ Good - data is relevant to the variant
enum NotificationLevel {
    Info(String),                           // Message to display
    Warning { message: String, code: u32 }, // Message and error code
    Error { message: String, code: u32, details: Vec<String> }, // Full error info
}

// ❌ Bad - irrelevant data
enum BadNotification {
    Info(String, u32, bool, f64), // What do these numbers mean?
}
```


### 4. Implement Common Traits

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum VegetableCategory {
    Leafy,      // Spinach, kale, lettuce
    Cruciferous, // Broccoli, cauliflower, cabbage
    Root,       // Carrots, beets, radishes
    Nightshade, // Tomatoes, peppers, eggplant
    Legume,     // Beans, peas, lentils
    Allium,     // Onions, garlic, leeks
}

// This allows us to:
// - Print with {:?}
// - Clone values
// - Compare with ==
// - Use in HashMap/HashSet
```


### 5. Provide Convenient Methods

```rust
impl VegetableCategory {
    // Constructor methods
    fn all_categories() -> Vec<VegetableCategory> {
        vec![
            VegetableCategory::Leafy,
            VegetableCategory::Cruciferous,
            VegetableCategory::Root,
            VegetableCategory::Nightshade,
            VegetableCategory::Legume,
            VegetableCategory::Allium,
        ]
    }

    // Query methods
    fn is_high_in_fiber(&self) -> bool {
        matches!(self, VegetableCategory::Legume | VegetableCategory::Cruciferous)
    }

    fn typical_cooking_time_minutes(&self) -> u32 {
        match self {
            VegetableCategory::Leafy => 3,
            VegetableCategory::Cruciferous => 8,
            VegetableCategory::Root => 25,
            VegetableCategory::Nightshade => 15,
            VegetableCategory::Legume => 45, // Dried legumes
            VegetableCategory::Allium => 10,
        }
    }

    fn get_examples(&self) -> Vec<&'static str> {
        match self {
            VegetableCategory::Leafy => vec!["spinach", "kale", "lettuce", "arugula"],
            VegetableCategory::Cruciferous => vec!["broccoli", "cauliflower", "cabbage", "Brussels sprouts"],
            VegetableCategory::Root => vec!["carrots", "beets", "radishes", "turnips"],
            VegetableCategory::Nightshade => vec!["tomatoes", "bell peppers", "eggplant", "potatoes"],
            VegetableCategory::Legume => vec!["black beans", "chickpeas", "lentils", "peas"],
            VegetableCategory::Allium => vec!["onions", "garlic", "leeks", "shallots"],
        }
    }
}
```


## Common Mistakes to Avoid

### Mistake 1: Using Enums Like C-Style Constants

```rust
// ❌ Don't use enums just for constants
enum BadColors {
    Red = 1,
    Green = 2,
    Blue = 3,
}

// ✅ Use enums for variants of a type
#[derive(Debug)]
enum VegetableColor {
    Green,
    Red,
    Orange,
    Purple,
    Yellow,
    White,
}

impl VegetableColor {
    fn hex_code(&self) -> &str {
        match self {
            VegetableColor::Green => "#228B22",
            VegetableColor::Red => "#DC143C",
            VegetableColor::Orange => "#FF8C00",
            VegetableColor::Purple => "#9370DB",
            VegetableColor::Yellow => "#FFD700",
            VegetableColor::White => "#FFFFFF",
        }
    }
}
```


### Mistake 2: Not Handling All Cases

```rust
enum OrderStatus {
    Pending, Confirmed, Preparing, Ready, Delivered,
}

fn handle_order(status: OrderStatus) {
    // ❌ Missing cases - compiler will warn
    match status {
        OrderStatus::Pending => println!("Order received"),
        OrderStatus::Ready => println!("Order ready"),
        // Missing other cases!
    }
}

// ✅ Handle all cases
fn handle_order_correctly(status: OrderStatus) {
    match status {
        OrderStatus::Pending => println!("Order received"),
        OrderStatus::Confirmed => println!("Order confirmed"),
        OrderStatus::Preparing => println!("Being prepared"),
        OrderStatus::Ready => println!("Order ready"),
        OrderStatus::Delivered => println!("Order delivered"),
    }
}
```


### Mistake 3: Overcomplicating Simple Cases

```rust
// ❌ Don't overcomplicate simple boolean-like cases
enum OverComplicated {
    IsVegetarian { vegetarian: bool, details: String },
    IsNotVegetarian { reason: String, alternatives: Vec<String> },
}

// ✅ Keep it simple when possible
#[derive(Debug)]
enum DietType {
    Vegetarian,
    NonVegetarian,
}

// ✅ Add complexity only when needed
#[derive(Debug)]
struct DishInfo {
    name: String,
    diet_type: DietType,
    vegetarian_alternatives: Vec<String>, // Only when needed
}
```


## Summary and Key Takeaways

### **Mental Model: The Restaurant Menu System**

Remember the restaurant menu analogy:

- 🍽️ **Menu categories** = Enum variants (appetizer, main course, dessert)
- 📋 **Each dish** = Enum value (can only be in one category)
- 🧾 **Menu item details** = Associated data (price, ingredients, etc.)
- 👨🍳 **Kitchen operations** = Methods on enums (prepare, price, categorize)


### **Essential Principles**

1. **One variant at a time** - An enum value can only be one variant
2. **Type safety** - Compiler ensures you handle all cases
3. **Express intent clearly** - Use descriptive variant names
4. **Associated data** - Variants can carry relevant information
5. **Pattern matching** - Use `match` to handle different variants
6. **Methods on enums** - Add behavior through `impl` blocks

### **When to Use Enums**

| **Use Case** | **Example** | **Why Enums Work Well** |
| :-- | :-- | :-- |
| **States** | Order status, game state | Clear transitions, impossible invalid states |
| **Categories** | Menu categories, user types | Mutually exclusive options |
| **Commands** | User actions, API operations | Different actions with different data |
| **Results** | Success/Error, Some/None | Safe error handling without nulls |
| **Configuration** | Cooking methods, themes | Clear choices with associated behavior |

### **Enum Design Checklist**

**✅ Well-Designed Enum:**

- Has clear, descriptive variant names
- Represents mutually exclusive possibilities
- Includes relevant associated data
- Implements appropriate traits (`Debug`, `Clone`, etc.)
- Provides helpful methods via `impl` blocks
- Handles all cases in match statements

**❌ Poorly-Designed Enum:**

- Uses vague names (A, B, C)
- Represents unrelated concepts
- Has unnecessary or confusing data
- Missing essential traits
- No helpful methods
- Incomplete match handling


### **Quick Pattern Reference**

```rust
// Basic enum
enum MenuCategory { Appetizer, MainCourse, Dessert }

// Enum with data
enum CookingInstruction {
    Preheat(u32),
    Cook { time: u32, temp: u32 },
    Season(String, String),
}

// Pattern matching
match instruction {
    CookingInstruction::Preheat(temp) => println!("Heat to {}", temp),
    CookingInstruction::Cook { time, temp } => println!("Cook {}min at {}", time, temp),
    CookingInstruction::Season(spice, amount) => println!("Add {} {}", amount, spice),
}

// Methods on enums
impl CookingInstruction {
    fn estimated_time(&self) -> u32 {
        match self {
            CookingInstruction::Preheat(_) => 10,
            CookingInstruction::Cook { time, .. } => *time,
            CookingInstruction::Season(_, _) => 2,
        }
    }
}
```

**Enums are like having a perfectly organized restaurant menu** - each dish clearly belongs to exactly one category, the categories cover all possibilities, and both staff and customers know exactly what to expect. They bring type safety, clarity, and expressiveness to your Rust programs, helping you model the real world accurately and safely!

1. https://www.w3schools.com/rust/rust_enums.php
2. https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html
3. https://rust-book.cs.brown.edu/ch06-01-defining-an-enum.html
4. https://doc.rust-lang.org/rust-by-example/custom_types/enum.html
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/enums.html
6. https://www.geeksforgeeks.org/rust/rust-enum/
7. https://www.reddit.com/r/rust/comments/ob3vvf/can_someone_explain_enum/
8. https://dev.to/sonubardai/structs-and-enums-in-rust-hme
9. https://dev.to/fadygrab/learning-rust-14-option-enum-an-enum-and-pattern-matching-use-case-1dgf
10. https://serokell.io/blog/enums-and-pattern-matching
11. https://2coffee.dev/en/articles/one-month-learning-rust-enums-and-pattern-matching
12. https://www.alexdwilson.dev/learning-in-public/programming-with-rust-enums
13. https://www.youtube.com/watch?v=XQWX7dMqY3U
14. https://www.programiz.com/rust/enum
