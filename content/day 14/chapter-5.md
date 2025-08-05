+++
title = "Advanced Match Patterns in Rust"
description = "Explore advanced match patterns in Rust, including destructuring, guards, bindings, and matching multiple patterns for powerful control flow."
date = 2025-08-05
draft = false
weight = 5
+++

# Advanced Match Patterns in Rust: Comprehensive Documentation for Beginners

Understanding advanced match patterns in Rust is like learning to be a **master sommelier in a vegetarian restaurant** - you need to expertly categorize, analyze, and respond to an incredible variety of dishes, ingredients, dietary restrictions, and customer preferences with precision and elegance. Just as a sommelier can instantly identify the perfect wine pairing by examining subtle characteristics, Rust's advanced match patterns let you precisely identify and handle complex data structures with surgical accuracy.

## The Vegetarian Restaurant Sommelier Analogy 🍷

### Imagine You're the Head Sommelier at an Upscale Vegetarian Restaurant

**Basic Matching (Beginner Sommelier):**

```rust
// ❌ Simple but limited - like only knowing "red" or "white" wine
match dish {
    "salad" => "light wine",
    "pasta" => "medium wine",
    _ => "house wine",
}
```

**Advanced Pattern Matching (Master Sommelier):**

```rust
// ✅ Sophisticated analysis - like a master sommelier considering every detail
match customer_order {
    VegetarianDish::Salad {
        greens: Greens::Arugula | Greens::Spinach,
        dressing: Dressing::Vinaigrette { acidity: 7..=9 },
        toppings: ref nuts @ Some(nuts_list) if nuts_list.contains(&"walnuts"),
        ..
    } => recommend_crisp_sauvignon_blanc(),

    VegetarianDish::Pasta {
        sauce: Sauce::Cream(_),
        protein: Some(protein),
        spice_level: level @ 1..=3
    } if protein.is_mushroom_based() => recommend_chardonnay(level),

    _ => recommend_house_selection(),
}
```

**Why Advanced Patterns Are Better:**

- 🎯 **Precise categorization** - Consider multiple factors simultaneously
- 🔍 **Deep inspection** - Look inside complex data structures
- 🎨 **Elegant handling** - Express complex logic clearly
- ✅ **Exhaustive coverage** - Compiler ensures you handle all cases
- 🚀 **Performance** - Zero-cost abstractions with compile-time optimization


## Understanding Advanced Match Patterns

### Core Concepts

**Advanced match patterns** go far beyond simple value comparison. They allow you to:

- **Destructure complex data** - Break apart structs, enums, tuples
- **Combine multiple conditions** - Use OR, AND, and range logic
- **Add conditional guards** - Include runtime checks
- **Bind and capture values** - Extract and name parts of matched data
- **Ignore irrelevant parts** - Focus only on what matters


## Pattern Types and Techniques

### 1. Multiple Patterns with OR (`|`)

**Like a sommelier who knows multiple dishes pair with the same wine:**

```rust
#[derive(Debug)]
enum DishCategory {
    LightSalad,
    HeavySalad,
    CreamyPasta,
    TomatoPasta,
    SpicyCurry,
    MildCurry,
    Dessert,
}

fn recommend_wine(dish: DishCategory) -> &'static str {
    match dish {
        // Multiple patterns for light, refreshing dishes
        DishCategory::LightSalad | DishCategory::TomatoPasta => {
            "🍷 Sauvignon Blanc - crisp and acidic to complement fresh flavors"
        }

        // Creamy, rich dishes
        DishCategory::HeavySalad | DishCategory::CreamyPasta => {
            "🍷 Chardonnay - rich and buttery to match creamy textures"
        }

        // Spicy dishes
        DishCategory::SpicyCurry | DishCategory::MildCurry => {
            "🍷 Riesling - sweetness balances heat and spice"
        }

        DishCategory::Dessert => {
            "🍷 Port - sweet dessert wine for sweet endings"
        }
    }
}

fn main() {
    let dishes = [
        DishCategory::LightSalad,
        DishCategory::CreamyPasta,
        DishCategory::SpicyCurry,
        DishCategory::Dessert,
    ];

    println!("🍽️ Wine Pairing Recommendations:");
    for dish in dishes {
        println!("   {:?}: {}", dish, recommend_wine(dish));
    }
}
```


### 2. Range Patterns with `..=`

**Like a sommelier categorizing by intensity, price, or age:**

```rust
fn categorize_by_characteristics(
    spice_level: u8,
    price: f64,
    prep_time: u32,
    customer_age: u8,
) -> String {
    match (spice_level, price, prep_time, customer_age) {
        // Young customers, budget-conscious, want quick mild food
        (1..=3, 0.0..=15.0, 1..=20, 18..=25) => {
            "🍕 Student Special: Mild dishes, quick service, great value"
        }

        // Adult customers, mid-range, moderate spice
        (4..=6, 15.01..=30.0, 20..=45, 26..=50) => {
            "🍽️ Executive Lunch: Balanced flavors, quality ingredients"
        }

        // Mature customers, premium, complex flavors, longer prep
        (7..=10, 30.01..=100.0, 45..=90, 51..=80) => {
            "👑 Gourmet Experience: Complex spices, artisanal preparation"
        }

        // High spice tolerance, any price, any time
        (8..=10, _, _, _) => {
            "🔥 Spice Enthusiast: Bring on the heat!"
        }

        // Premium pricing regardless of other factors
        (_, 100.01..=f64::INFINITY, _, _) => {
            "💎 Luxury Dining: Only the finest ingredients and preparation"
        }

        _ => {
            "🍴 House Recommendation: Something delicious for everyone"
        }
    }
}

fn main() {
    let customers = [
        (2, 12.50, 15, 22),   // Young, budget, quick, mild
        (5, 25.00, 35, 35),   // Adult, moderate everything
        (8, 50.00, 60, 55),   // Mature, premium, complex
        (9, 15.00, 30, 28),   // Spice lover, moderate budget
        (3, 150.00, 90, 45),  // Luxury customer
        (6, 8.00, 10, 30),    // Doesn't fit main categories
    ];

    println!("🎯 Customer Categorization:");
    for (spice, price, time, age) in customers {
        let category = categorize_by_characteristics(spice, price, time, age);
        println!("   Customer (🌶️{}, ${:.2}, {}min, {}y): {}",
                 spice, price, time, age, category);
    }
}
```


### 3. Destructuring Structs and Enums

**Like a sommelier analyzing the complete composition of a dish:**

```rust
#[derive(Debug)]
struct Ingredient {
    name: String,
    quantity: u32,
    quality: Quality,
    origin: String,
}

#[derive(Debug)]
enum Quality {
    Organic,
    Premium,
    Standard,
    Budget,
}

#[derive(Debug)]
struct VegetarianDish {
    name: String,
    main_ingredient: Ingredient,
    secondary_ingredients: Vec<Ingredient>,
    cooking_method: CookingMethod,
    presentation: Presentation,
}

#[derive(Debug)]
enum CookingMethod {
    Raw,
    Steamed { temperature: u32, time_minutes: u32 },
    Roasted { temperature: u32 },
    Sauteed { oil_type: String },
    Grilled,
}

#[derive(Debug)]
struct Presentation {
    plating_style: String,
    garnish: Option<String>,
    temperature_served: TempServed,
}

#[derive(Debug)]
enum TempServed {
    Hot,
    Cold,
    RoomTemperature,
}

fn analyze_dish_for_wine_pairing(dish: &VegetarianDish) -> String {
    match dish {
        // Destructure the entire dish to analyze every component
        VegetarianDish {
            name,
            main_ingredient: Ingredient {
                name: main_name,
                quality: Quality::Organic | Quality::Premium,
                origin,
                ..
            },
            cooking_method: CookingMethod::Raw,
            presentation: Presentation {
                temperature_served: TempServed::Cold,
                garnish: Some(garnish),
                ..
            },
            ..
        } => {
            format!("🥗 {} with premium {} from {}: Pair with crisp Sancerre. The {} garnish adds elegance!",
                   name, main_name, origin, garnish)
        }

        // Hot, roasted dishes with high-quality ingredients
        VegetarianDish {
            main_ingredient: Ingredient { quality: Quality::Premium, .. },
            cooking_method: CookingMethod::Roasted { temperature: 375..=450 },
            presentation: Presentation { temperature_served: TempServed::Hot, .. },
            ..
        } => {
            format!("🔥 {}: High-heat roasting develops complex flavors. Pair with full-bodied Côtes du Rhône.",
                   dish.name)
        }

        // Sautéed dishes - oil type matters for pairing
        VegetarianDish {
            cooking_method: CookingMethod::Sauteed { oil_type },
            secondary_ingredients,
            ..
        } if secondary_ingredients.len() >= 3 => {
            format!("🍳 {}: Sautéed in {} with {} ingredients creates complexity. Try Burgundian Chardonnay.",
                   dish.name, oil_type, secondary_ingredients.len())
        }

        // Simple dishes with standard ingredients
        VegetarianDish {
            main_ingredient: Ingredient { quality: Quality::Standard | Quality::Budget, .. },
            secondary_ingredients,
            ..
        } if secondary_ingredients.len() <= 2 => {
            format!("🍽️ {}: Simple, honest cooking deserves an honest wine. House Côtes du Ventoux.",
                   dish.name)
        }

        // Catch-all for complex cases
        _ => {
            format!("🍷 {}: Complex dish requiring individual analysis. Consult sommelier.",
                   dish.name)
        }
    }
}

fn main() {
    let dishes = vec![
        VegetarianDish {
            name: "Heirloom Tomato Carpaccio".to_string(),
            main_ingredient: Ingredient {
                name: "Heirloom Tomatoes".to_string(),
                quantity: 3,
                quality: Quality::Premium,
                origin: "Tuscany".to_string(),
            },
            secondary_ingredients: vec![
                Ingredient {
                    name: "Basil".to_string(),
                    quantity: 10,
                    quality: Quality::Organic,
                    origin: "Local".to_string(),
                }
            ],
            cooking_method: CookingMethod::Raw,
            presentation: Presentation {
                plating_style: "Artistic arrangement".to_string(),
                garnish: Some("Microgreens".to_string()),
                temperature_served: TempServed::Cold,
            },
        },

        VegetarianDish {
            name: "Roasted Root Vegetable Medley".to_string(),
            main_ingredient: Ingredient {
                name: "Purple Carrots".to_string(),
                quantity: 500,
                quality: Quality::Premium,
                origin: "Organic Farm".to_string(),
            },
            secondary_ingredients: vec![
                Ingredient {
                    name: "Parsnips".to_string(),
                    quantity: 300,
                    quality: Quality::Organic,
                    origin: "Local".to_string(),
                },
                Ingredient {
                    name: "Sweet Potatoes".to_string(),
                    quantity: 400,
                    quality: Quality::Premium,
                    origin: "North Carolina".to_string(),
                }
            ],
            cooking_method: CookingMethod::Roasted { temperature: 425 },
            presentation: Presentation {
                plating_style: "Rustic".to_string(),
                garnish: None,
                temperature_served: TempServed::Hot,
            },
        },
    ];

    println!("🍷 Advanced Wine Pairing Analysis:");
    println!("{:=<80}", "");

    for dish in &dishes {
        let pairing = analyze_dish_for_wine_pairing(dish);
        println!("{}\n", pairing);
    }
}
```


### 4. Pattern Guards with `if`

**Like a sommelier making final adjustments based on subtle conditions:**

```rust
#[derive(Debug)]
struct Customer {
    name: String,
    age: u32,
    dietary_restrictions: Vec<String>,
    spice_tolerance: u8,
    budget_per_person: f64,
    occasion: String,
}

#[derive(Debug)]
enum WineRecommendation {
    Specific { name: String, reason: String, price: f64 },
    Category { category: String, reason: String },
    Consultation { message: String },
}

fn recommend_wine_with_guards(customer: &Customer, dish_spice_level: u8) -> WineRecommendation {
    match customer {
        // High-end customer with specific preferences
        Customer {
            budget_per_person,
            spice_tolerance,
            age,
            occasion,
            dietary_restrictions,
            ..
        } if *budget_per_person > 100.0 &&
             *spice_tolerance >= dish_spice_level &&
             *age >= 30 &&
             occasion.contains("anniversary") => {
            WineRecommendation::Specific {
                name: "Dom Pérignon Vintage Champagne".to_string(),
                reason: "Perfect for your anniversary celebration - elegant bubbles complement any spice level".to_string(),
                price: 200.0,
            }
        }

        // Vegan customer with moderate budget
        Customer {
            dietary_restrictions,
            budget_per_person: 25.0..=60.0,
            spice_tolerance,
            ..
        } if dietary_restrictions.contains(&"vegan".to_string()) &&
             *spice_tolerance >= dish_spice_level => {
            WineRecommendation::Category {
                category: "Certified Vegan Wines".to_string(),
                reason: "Filtered without animal products, perfectly suited to your dietary needs".to_string(),
            }
        }

        // Budget-conscious customer
        Customer {
            budget_per_person: budget,
            spice_tolerance,
            age,
            ..
        } if *budget <= 25.0 && *spice_tolerance >= dish_spice_level && *age >= 21 => {
            WineRecommendation::Category {
                category: "House Selection".to_string(),
                reason: format!("Excellent value wines under ${:.0} that pair beautifully", budget),
            }
        }

        // Customer can't handle the spice level
        Customer { spice_tolerance, .. } if *spice_tolerance < dish_spice_level => {
            WineRecommendation::Consultation {
                message: format!("This dish (spice level {}) might be too spicy for your tolerance ({}). Let's find a milder option or a wine that cools the palate!",
                                dish_spice_level, spice_tolerance),
            }
        }

        // Young customer
        Customer { age, .. } if *age < 21 => {
            WineRecommendation::Consultation {
                message: "We have an excellent selection of artisanal mocktails and fresh juices!".to_string(),
            }
        }

        // Default consultation for complex cases
        _ => {
            WineRecommendation::Consultation {
                message: "Let me personally help you find the perfect pairing for your unique preferences".to_string(),
            }
        }
    }
}

fn main() {
    let customers = vec![
        Customer {
            name: "Eleanor".to_string(),
            age: 45,
            dietary_restrictions: vec![],
            spice_tolerance: 8,
            budget_per_person: 150.0,
            occasion: "25th wedding anniversary".to_string(),
        },
        Customer {
            name: "Marcus".to_string(),
            age: 28,
            dietary_restrictions: vec!["vegan".to_string()],
            spice_tolerance: 6,
            budget_per_person: 40.0,
            occasion: "casual dinner".to_string(),
        },
        Customer {
            name: "Sarah".to_string(),
            age: 24,
            dietary_restrictions: vec![],
            spice_tolerance: 3,
            budget_per_person: 20.0,
            occasion: "student meal".to_string(),
        },
        Customer {
            name: "James".to_string(),
            age: 19,
            dietary_restrictions: vec![],
            spice_tolerance: 7,
            budget_per_person: 30.0,
            occasion: "date night".to_string(),
        },
    ];

    let dish_spice_level = 5;

    println!("🍷 Personalized Wine Recommendations (Dish spice level: {})", dish_spice_level);
    println!("{:=<80}", "");

    for customer in customers {
        let recommendation = recommend_wine_with_guards(&customer, dish_spice_level);
        println!("👤 {} ({}y, tolerance: {}, budget: ${:.0}):",
                 customer.name, customer.age, customer.spice_tolerance, customer.budget_per_person);

        match recommendation {
            WineRecommendation::Specific { name, reason, price } => {
                println!("   🍾 Recommendation: {} (${:.2})", name, price);
                println!("   💡 Reason: {}", reason);
            }
            WineRecommendation::Category { category, reason } => {
                println!("   🍷 Category: {}", category);
                println!("   💡 Reason: {}", reason);
            }
            WineRecommendation::Consultation { message } => {
                println!("   🤝 Personal consultation needed: {}", message);
            }
        }
        println!();
    }
}
```


### 5. Reference Patterns and Borrowing

**Like a sommelier examining a wine list without taking ownership:**

```rust
#[derive(Debug)]
struct WineInventory {
    red_wines: Vec<String>,
    white_wines: Vec<String>,
    sparkling_wines: Vec<String>,
    dessert_wines: Vec<String>,
}

fn check_availability_and_recommend(
    inventory: &WineInventory,
    customer_preference: &str,
) -> String {
    match inventory {
        // Match with references - we're borrowing, not taking ownership
        WineInventory {
            ref red_wines,
            ref white_wines,
            ref sparkling_wines,
            ref dessert_wines,
        } => {
            match customer_preference {
                "bold" if !red_wines.is_empty() => {
                    format!("🍷 Bold red recommendation: {} ({} options available)",
                           red_wines[^0], red_wines.len())
                }
                "light" | "crisp" if !white_wines.is_empty() => {
                    format!("🥂 Light white recommendation: {} ({} options available)",
                           white_wines[^0], white_wines.len())
                }
                "celebration" if !sparkling_wines.is_empty() => {
                    format!("🍾 Celebration sparkling: {} ({} options available)",
                           sparkling_wines[^0], sparkling_wines.len())
                }
                "sweet" if !dessert_wines.is_empty() => {
                    format!("🍯 Sweet dessert wine: {} ({} options available)",
                           dessert_wines[^0], dessert_wines.len())
                }
                _ => "🤔 Let me check our special reserves for you".to_string(),
            }
        }
    }
}

// Alternative pattern matching with references
fn analyze_inventory_depth(inventory: &WineInventory) -> String {
    match inventory {
        // Using & in patterns to match references directly
        &WineInventory {
            red_wines: ref reds,
            white_wines: ref whites,
            sparkling_wines: ref sparklings,
            dessert_wines: ref desserts,
        } => {
            let total = reds.len() + whites.len() + sparklings.len() + desserts.len();

            match (reds.len(), whites.len(), sparklings.len(), desserts.len()) {
                (r, w, s, d) if r > 10 && w > 10 && s > 5 && d > 3 => {
                    format!("🏆 Exceptional inventory: {} total wines with strong selection in all categories", total)
                }
                (r, w, _, _) if r > 5 && w > 5 => {
                    format!("✅ Good basic selection: {} wines, strong in reds and whites", total)
                }
                (0, 0, 0, 0) => {
                    "❌ No wines in inventory!".to_string()
                }
                _ => {
                    format!("⚠️ Limited selection: {} wines total, may need restocking", total)
                }
            }
        }
    }
}

fn main() {
    let inventory = WineInventory {
        red_wines: vec![
            "Cabernet Sauvignon Reserve".to_string(),
            "Pinot Noir Burgundy".to_string(),
            "Merlot Estate".to_string(),
        ],
        white_wines: vec![
            "Chardonnay Chablis".to_string(),
            "Sauvignon Blanc Loire".to_string(),
        ],
        sparkling_wines: vec![
            "Champagne Brut".to_string(),
        ],
        dessert_wines: vec![
            "Port Vintage".to_string(),
            "Ice Wine".to_string(),
        ],
    };

    let preferences = ["bold", "light", "celebration", "sweet", "unknown"];

    println!("🍷 Wine Availability Check:");
    println!("{}", analyze_inventory_depth(&inventory));
    println!();

    for preference in preferences {
        let recommendation = check_availability_and_recommend(&inventory, preference);
        println!("   Preference '{}': {}", preference, recommendation);
    }

    // Inventory is still available after borrowing
    println!("\n📊 Final inventory count: {} total wines",
             inventory.red_wines.len() + inventory.white_wines.len() +
             inventory.sparkling_wines.len() + inventory.dessert_wines.len());
}
```


### 6. Binding with `@` Patterns

**Like a sommelier noting specific details while categorizing:**

```rust
#[derive(Debug)]
enum WineRegion {
    Bordeaux,
    Burgundy,
    Champagne,
    Loire,
    Rhone,
    Alsace,
    California,
    Oregon,
    Australia,
}

#[derive(Debug)]
struct Wine {
    name: String,
    region: WineRegion,
    vintage: u32,
    price: f64,
    alcohol_content: f32,
}

fn categorize_wine_with_binding(wine: &Wine) -> String {
    match wine {
        // Bind the entire price range to a variable while matching
        Wine {
            price: expensive_price @ 100.0..=500.0,
            vintage: classic_vintage @ 1990..=2010,
            region: WineRegion::Bordeaux | WineRegion::Burgundy,
            alcohol_content: alcohol @ 12.5..=14.5,
            ..
        } => {
            format!("🏆 Premium Classic: ${:.2} from vintage {} with {:.1}% alcohol - excellent investment wine!",
                   expensive_price, classic_vintage, alcohol)
        }

        // Bind vintage while checking range
        Wine {
            vintage: recent_vintage @ 2015..=2023,
            price: moderate_price @ 25.0..=75.0,
            region: region @ (WineRegion::California | WineRegion::Oregon),
            ..
        } => {
            format!("🌟 New World Excellence: {} {} vintage {} at ${:.2} - fresh and approachable!",
                   format!("{:?}", region), wine.name, recent_vintage, moderate_price)
        }

        // Bind alcohol content for specific analysis
        Wine {
            alcohol_content: high_alcohol @ 14.5..=16.0,
            region: WineRegion::Rhone | WineRegion::Australia,
            price: value_price @ 15.0..=40.0,
            ..
        } => {
            format!("🔥 Bold & Powerful: {:.1}% alcohol gives this wine serious body at ${:.2} - perfect with rich dishes!",
                   high_alcohol, value_price)
        }

        // Bind and use vintage for age calculation
        Wine {
            vintage: old_vintage @ 1970..=1989,
            region: french_region @ (WineRegion::Bordeaux | WineRegion::Burgundy | WineRegion::Champagne),
            ..
        } => {
            let age = 2024 - old_vintage;
            format!("🕰️ Mature Classic: This {:?} from {} is {} years old - a piece of wine history!",
                   french_region, old_vintage, age)
        }

        // Bind multiple values for budget analysis
        Wine {
            price: budget_price @ 10.0..=25.0,
            vintage: recent @ 2018..=2023,
            alcohol_content: moderate_alcohol @ 11.0..=13.5,
            ..
        } => {
            format!("💰 Great Value: Recent {} vintage at ${:.2} with moderate {:.1}% alcohol - everyday drinking perfection!",
                   recent, budget_price, moderate_alcohol)
        }

        // Catch remaining cases
        _ => {
            format!("🍷 Specialty Wine: {} - requires individual consultation", wine.name)
        }
    }
}

fn main() {
    let wine_collection = vec![
        Wine {
            name: "Château Margaux".to_string(),
            region: WineRegion::Bordeaux,
            vintage: 2005,
            price: 350.0,
            alcohol_content: 13.5,
        },
        Wine {
            name: "Willamette Valley Pinot".to_string(),
            region: WineRegion::Oregon,
            vintage: 2020,
            price: 45.0,
            alcohol_content: 13.8,
        },
        Wine {
            name: "Châteauneuf-du-Pape".to_string(),
            region: WineRegion::Rhone,
            vintage: 2019,
            price: 32.0,
            alcohol_content: 15.2,
        },
        Wine {
            name: "Dom Pérignon".to_string(),
            region: WineRegion::Champagne,
            vintage: 1985,
            price: 400.0,
            alcohol_content: 12.5,
        },
        Wine {
            name: "Everyday Chardonnay".to_string(),
            region: WineRegion::California,
            vintage: 2022,
            price: 18.0,
            alcohol_content: 12.8,
        },
    ];

    println!("🍷 Wine Collection Analysis with Binding Patterns:");
    println!("{:=<80}", "");

    for wine in &wine_collection {
        let analysis = categorize_wine_with_binding(wine);
        println!("🍾 {}: {}\n", wine.name, analysis);
    }
}
```


### 7. Slice Patterns

**Like a sommelier analyzing a flight of wines as a group:**

```rust
fn analyze_wine_flight(wines: &[String]) -> String {
    match wines {
        // Empty flight
        [] => "❌ No wines selected for tasting flight".to_string(),

        // Single wine
        [single] => format!("🍷 Solo tasting: {} - let's focus on this wine's complexity", single),

        // Perfect pair
        [first, second] => {
            format!("👥 Classic pairing: {} and {} - ideal for comparing styles", first, second)
        }

        // Traditional flight of three
        [light, medium, bold] => {
            format!("✨ Perfect flight progression: {} (light) → {} (medium) → {} (bold)",
                   light, medium, bold)
        }

        // Four wines - can do two pairs
        [w1, w2, w3, w4] => {
            format!("🎯 Double comparison: First pair ({}, {}) vs Second pair ({}, {})",
                   w1, w2, w3, w4)
        }

        // Many wines - focus on first and last with count
        [first, .., last] => {
            format!("🌟 Grand tasting: Starting with {} and ending with {} ({} wines total)",
                   first, last, wines.len())
        }
    }
}

fn analyze_flavor_profile_sequence(flavor_notes: &[&str]) -> String {
    match flavor_notes {
        [] => "No flavor notes recorded".to_string(),

        // Single dominant note
        [dominant] => format!("🎯 Focused flavor: Pure {} character", dominant),

        // Simple profile
        [primary, secondary] => {
            format!("⚖️ Balanced: {} dominant with {} undertones", primary, secondary)
        }

        // Complex but manageable
        [primary, secondary, tertiary] => {
            format!("🎨 Complex: {} leads, {} in the middle, {} finish",
                   primary, secondary, tertiary)
        }

        // Rich complexity - highlight key elements
        [primary, secondary, rest @ ..] if rest.len() <= 3 => {
            format!("🌈 Rich complexity: {} and {} lead, with hints of {}",
                   primary, secondary, rest.join(", "))
        }

        // Overwhelming complexity - simplify
        [primary, secondary, ..] => {
            format!("🎪 Extraordinarily complex: {} and {} are most prominent among {} total notes",
                   primary, secondary, flavor_notes.len())
        }
    }
}

fn recommend_tasting_order(wine_strengths: &[u8]) -> String {
    match wine_strengths {
        [] => "No wines to arrange".to_string(),

        [single] => format!("Single wine (strength {})", single),

        // Check if already in ascending order (lightest to strongest)
        strengths if strengths.windows(2).all(|pair| pair[^0] <= pair[^1]) => {
            format!("✅ Perfect order: {} wines arranged light to bold ({})",
                   strengths.len(),
                   strengths.iter().map(|s| s.to_string()).collect::<Vec<_>>().join(" → "))
        }

        // Check if in descending order (needs reversal)
        strengths if strengths.windows(2).all(|pair| pair[^0] >= pair[^1]) => {
            format!("🔄 Reverse order needed: Currently bold to light ({})",
                   strengths.iter().map(|s| s.to_string()).collect::<Vec<_>>().join(" → "))
        }

        // Mixed order - needs sorting
        _ => {
            let mut sorted = wine_strengths.to_vec();
            sorted.sort();
            format!("🎲 Random order detected: Suggest rearranging to {}",
                   sorted.iter().map(|s| s.to_string()).collect::<Vec<_>>().join(" → "))
        }
    }
}

fn main() {
    // Test wine flight analysis
    let flights = [
        vec![],
        vec!["Pinot Grigio".to_string()],
        vec!["Chardonnay".to_string(), "Pinot Noir".to_string()],
        vec!["Sauvignon Blanc".to_string(), "Chardonnay".to_string(), "Cabernet".to_string()],
        vec!["Riesling".to_string(), "Gewürztraminer".to_string(), "Pinot Noir".to_string(), "Malbec".to_string()],
        vec!["Champagne".to_string(), "Chablis".to_string(), "Sancerre".to_string(),
             "Burgundy".to_string(), "Bordeaux".to_string(), "Port".to_string()],
    ];

    println!("🍷 Wine Flight Analysis:");
    println!("{:=<60}", "");
    for (i, flight) in flights.iter().enumerate() {
        println!("Flight {}: {}", i + 1, analyze_wine_flight(flight));
    }

    // Test flavor profile analysis
    let flavor_profiles = [
        vec![],
        vec!["cherry"],
        vec!["vanilla", "oak"],
        vec!["blackberry", "chocolate", "spice"],
        vec!["apple", "citrus", "mineral", "honey"],
        vec!["plum", "leather", "tobacco", "cedar", "coffee", "dark chocolate", "herbs"],
    ];

    println!("\n🎨 Flavor Profile Analysis:");
    println!("{:=<60}", "");
    for (i, profile) in flavor_profiles.iter().enumerate() {
        println!("Profile {}: {}", i + 1, analyze_flavor_profile_sequence(profile));
    }

    // Test tasting order recommendations
    let tasting_orders = [
        vec![],
        vec![^5],
        vec![3, 7],          // Light to bold ✓
        vec![8, 5, 2],       // Bold to light - needs reversal
        vec![4, 2, 8, 1, 6], // Random order
        vec![1, 3, 5, 7, 9], // Perfect ascending order
    ];

    println!("\n📊 Tasting Order Analysis:");
    println!("{:=<60}", "");
    for (i, order) in tasting_orders.iter().enumerate() {
        println!("Order {}: {}", i + 1, recommend_tasting_order(order));
    }
}
```


### 8. Nested Destructuring

**Like a sommelier analyzing the complete terroir and production details:**

```rust
#[derive(Debug)]
struct Vineyard {
    name: String,
    location: Location,
    soil_type: SoilType,
}

#[derive(Debug)]
struct Location {
    region: String,
    country: String,
    coordinates: (f64, f64), // latitude, longitude
    elevation_meters: u32,
}

#[derive(Debug)]
enum SoilType {
    Limestone,
    Clay,
    Sand,
    Volcanic,
    Mixed { primary: Box<SoilType>, secondary: Box<SoilType> },
}

#[derive(Debug)]
struct WineProduction {
    vineyard: Vineyard,
    harvest_year: u32,
    fermentation: FermentationDetails,
    aging: AgingProcess,
}

#[derive(Debug)]
struct FermentationDetails {
    vessel_type: VesselType,
    temperature_celsius: u8,
    duration_days: u32,
    yeast_type: YeastType,
}

#[derive(Debug)]
enum VesselType {
    StainlessSteel,
    Oak { toast_level: ToastLevel, origin: String },
    Concrete,
    Clay,
}

#[derive(Debug)]
enum ToastLevel {
    Light,
    Medium,
    Heavy,
}

#[derive(Debug)]
enum YeastType {
    Wild,
    Cultured(String), // strain name
}

#[derive(Debug)]
struct AgingProcess {
    method: AgingMethod,
    duration_months: u32,
}

#[derive(Debug)]
enum AgingMethod {
    StainlessSteel,
    Oak { barrel_type: BarrelType, previous_use: PreviousUse },
    Bottle,
}

#[derive(Debug)]
enum BarrelType {
    French,
    American,
    Hungarian,
}

#[derive(Debug)]
enum PreviousUse {
    New,
    OnceUsed,
    Neutral, // used many times
}

fn analyze_terroir_and_production(wine: &WineProduction) -> String {
    match wine {
        // French limestone terroir with traditional oak aging
        WineProduction {
            vineyard: Vineyard {
                location: Location {
                    country: country,
                    region: region,
                    elevation_meters: elevation @ 200..=500,
                    ..
                },
                soil_type: SoilType::Limestone,
                ..
            },
            fermentation: FermentationDetails {
                vessel_type: VesselType::Oak { toast_level: ToastLevel::Medium, .. },
                yeast_type: YeastType::Wild,
                ..
            },
            aging: AgingProcess {
                method: AgingMethod::Oak {
                    barrel_type: BarrelType::French,
                    previous_use: PreviousUse::New | PreviousUse::OnceUsed
                },
                duration_months: aging_months @ 12..=24,
            },
            harvest_year: vintage @ 2015..=2020,
        } if country == "France" => {
            format!("🇫🇷 Classic French Terroir: {} {} from {} vintage, limestone soil at {}m elevation. \
                     Wild fermentation in medium-toast oak, aged {} months in French barrels. Traditional excellence!",
                    region, country, vintage, elevation, aging_months)
        }

        // New World stainless steel style
        WineProduction {
            vineyard: Vineyard {
                location: Location {
                    country: country @ ("United States" | "Australia" | "New Zealand"),
                    region,
                    elevation_meters: elevation @ 50..=300,
                    ..
                },
                soil_type: SoilType::Mixed { primary, secondary },
                ..
            },
            fermentation: FermentationDetails {
                vessel_type: VesselType::StainlessSteel,
                temperature_celsius: temp @ 15..=18,
                yeast_type: YeastType::Cultured(strain),
                ..
            },
            aging: AgingProcess {
                method: AgingMethod::StainlessSteel,
                duration_months: short_aging @ 3..=6,
            },
            ..
        } => {
            format!("🌍 New World Precision: {} {} style with mixed {:?}/{:?} soil at {}m. \
                     Controlled {}°C fermentation with {} yeast, {} months stainless aging. Pure fruit expression!",
                    region, country, primary, secondary, elevation, temp, strain, short_aging)
        }

        // Volcanic terroir - unique characteristics
        WineProduction {
            vineyard: Vineyard {
                soil_type: SoilType::Volcanic,
                location: Location { region, elevation_meters: high_elevation @ 500..=1200, .. },
                ..
            },
            fermentation: FermentationDetails {
                vessel_type: vessel_type,
                temperature_celsius: temp,
                ..
            },
            aging: AgingProcess { duration_months: aging, .. },
            harvest_year: vintage,
        } => {
            format!("🌋 Volcanic Terroir: {} at {}m elevation from {} harvest. \
                     Volcanic soil imparts unique minerality. {:?} fermentation at {}°C, {} months aging. \
                     Distinctive mineral-driven profile!",
                    region, high_elevation, vintage, vessel_type, temp, aging)
        }

        // Experimental/modern techniques
        WineProduction {
            fermentation: FermentationDetails {
                vessel_type: VesselType::Concrete | VesselType::Clay,
                yeast_type: YeastType::Wild,
                duration_days: long_fermentation @ 30..=60,
                ..
            },
            aging: AgingProcess {
                method: AgingMethod::Oak {
                    barrel_type: BarrelType::Hungarian,
                    previous_use: PreviousUse::Neutral
                },
                duration_months: extended_aging @ 18..=36,
            },
            ..
        } => {
            format!("🔬 Artisanal Innovation: Extended {}-day wild fermentation in {:?} vessels, \
                     followed by {} months in neutral Hungarian oak. Modern techniques with traditional soul!",
                    long_fermentation, wine.fermentation.vessel_type, extended_aging)
        }

        // Catch-all for other combinations
        _ => {
            format!("🍷 Unique Production: {} {} - distinctive winemaking approach requiring individual analysis",
                    wine.vineyard.location.region, wine.vineyard.location.country)
        }
    }
}

fn main() {
    let wines = vec![
        WineProduction {
            vineyard: Vineyard {
                name: "Domaine de la Côte".to_string(),
                location: Location {
                    region: "Burgundy".to_string(),
                    country: "France".to_string(),
                    coordinates: (47.0, 4.0),
                    elevation_meters: 350,
                },
                soil_type: SoilType::Limestone,
            },
            harvest_year: 2018,
            fermentation: FermentationDetails {
                vessel_type: VesselType::Oak {
                    toast_level: ToastLevel::Medium,
                    origin: "French".to_string()
                },
                temperature_celsius: 28,
                duration_days: 21,
                yeast_type: YeastType::Wild,
            },
            aging: AgingProcess {
                method: AgingMethod::Oak {
                    barrel_type: BarrelType::French,
                    previous_use: PreviousUse::OnceUsed
                },
                duration_months: 18,
            },
        },

        WineProduction {
            vineyard: Vineyard {
                name: "Willamette Valley Estate".to_string(),
                location: Location {
                    region: "Oregon".to_string(),
                    country: "United States".to_string(),
                    coordinates: (45.0, -123.0),
                    elevation_meters: 180,
                },
                soil_type: SoilType::Mixed {
                    primary: Box::new(SoilType::Clay),
                    secondary: Box::new(SoilType::Sand)
                },
            },
            harvest_year: 2021,
            fermentation: FermentationDetails {
                vessel_type: VesselType::StainlessSteel,
                temperature_celsius: 16,
                duration_days: 18,
                yeast_type: YeastType::Cultured("EC-1118".to_string()),
            },
            aging: AgingProcess {
                method: AgingMethod::StainlessSteel,
                duration_months: 4,
            },
        },

        WineProduction {
            vineyard: Vineyard {
                name: "Volcanic Ridge".to_string(),
                location: Location {
                    region: "Sicily".to_string(),
                    country: "Italy".to_string(),
                    coordinates: (37.5, 15.0),
                    elevation_meters: 800,
                },
                soil_type: SoilType::Volcanic,
            },
            harvest_year: 2019,
            fermentation: FermentationDetails {
                vessel_type: VesselType::Concrete,
                temperature_celsius: 24,
                duration_days: 25,
                yeast_type: YeastType::Wild,
            },
            aging: AgingProcess {
                method: AgingMethod::Oak {
                    barrel_type: BarrelType::French,
                    previous_use: PreviousUse::New
                },
                duration_months: 12,
            },
        },
    ];

    println!("🍇 Terroir and Production Analysis:");
    println!("{:=<100}", "");

    for (i, wine) in wines.iter().enumerate() {
        println!("Wine {}:", i + 1);
        println!("{}\n", analyze_terroir_and_production(wine));
    }
}
```


## Advanced Pattern Matching Techniques

### 1. The `matches!` Macro

**Quick boolean checks without full pattern matching:**

```rust
#[derive(Debug)]
enum OrderStatus {
    Pending,
    Confirmed,
    InProgress,
    ReadyToServe,
    Served,
    Cancelled,
}

#[derive(Debug)]
struct Order {
    id: u32,
    dish: String,
    status: OrderStatus,
    priority: u8,
}

fn filter_urgent_orders(orders: &[Order]) -> Vec<&Order> {
    orders.iter().filter(|order| {
        // Use matches! for quick boolean checks
        matches!(order.status, OrderStatus::Pending | OrderStatus::Confirmed) &&
        matches!(order.priority, 4..=5)
    }).collect()
}

fn categorize_orders(orders: &[Order]) -> (usize, usize, usize, usize) {
    let pending = orders.iter().filter(|o| matches!(o.status, OrderStatus::Pending)).count();
    let active = orders.iter().filter(|o| matches!(o.status, OrderStatus::InProgress)).count();
    let ready = orders.iter().filter(|o| matches!(o.status, OrderStatus::ReadyToServe)).count();
    let completed = orders.iter().filter(|o| matches!(o.status, OrderStatus::Served)).count();

    (pending, active, ready, completed)
}

fn main() {
    let orders = vec![
        Order { id: 1, dish: "Quinoa Bowl".to_string(), status: OrderStatus::Pending, priority: 5 },
        Order { id: 2, dish: "Pasta Primavera".to_string(), status: OrderStatus::Confirmed, priority: 3 },
        Order { id: 3, dish: "Lentil Soup".to_string(), status: OrderStatus::InProgress, priority: 4 },
        Order { id: 4, dish: "Veggie Burger".to_string(), status: OrderStatus::ReadyToServe, priority: 2 },
        Order { id: 5, dish: "Caesar Salad".to_string(), status: OrderStatus::Pending, priority: 4 },
    ];

    println!("📊 Order Analysis:");

    let urgent = filter_urgent_orders(&orders);
    println!("🚨 Urgent orders (pending/confirmed, priority 4-5): {}", urgent.len());
    for order in urgent {
        println!("   Order #{}: {} (priority {}, {:?})",
                 order.id, order.dish, order.priority, order.status);
    }

    let (pending, active, ready, completed) = categorize_orders(&orders);
    println!("\n📈 Order Status Summary:");
    println!("   Pending: {}", pending);
    println!("   Active: {}", active);
    println!("   Ready: {}", ready);
    println!("   Completed: {}", completed);
}
```


### 2. `if let` and `while let` Patterns

**Simplified pattern matching for specific cases:**

```rust
use std::collections::VecDeque;

#[derive(Debug)]
enum KitchenTask {
    PrepIngredients { dish: String, time_estimate: u32 },
    CookDish { dish: String, cooking_time: u32 },
    PlateAndServe { dish: String, table: u32 },
    CleanEquipment { equipment: String },
}

struct KitchenQueue {
    tasks: VecDeque<KitchenTask>,
}

impl KitchenQueue {
    fn new() -> Self {
        KitchenQueue {
            tasks: VecDeque::new(),
        }
    }

    fn add_task(&mut self, task: KitchenTask) {
        self.tasks.push_back(task);
    }

    // Process only cooking tasks with if let
    fn process_next_cooking_task(&mut self) -> Option<String> {
        // Look for the first cooking task without removing others
        for i in 0..self.tasks.len() {
            if let Some(KitchenTask::CookDish { dish, cooking_time }) = self.tasks.get(i) {
                let result = format!("🍳 Cooking {} for {} minutes", dish, cooking_time);
                self.tasks.remove(i);
                return Some(result);
            }
        }
        None
    }

    // Process prep tasks that are quick
    fn process_quick_prep_tasks(&mut self) -> Vec<String> {
        let mut completed = Vec::new();

        // while let to keep processing until no more quick prep tasks
        while let Some(task) = self.tasks.pop_front() {
            if let KitchenTask::PrepIngredients { dish, time_estimate } = task {
                if time_estimate <= 10 {
                    completed.push(format!("✅ Quick prep: {} ({}min)", dish, time_estimate));
                    continue;
                }
                // Put back if not quick
                self.tasks.push_front(KitchenTask::PrepIngredients { dish, time_estimate });
                break;
            } else {
                // Put back non-prep tasks
                self.tasks.push_front(task);
                break;
            }
        }

        completed
    }

    // Process all plating tasks for a specific table
    fn process_table_orders(&mut self, target_table: u32) -> Vec<String> {
        let mut plated_dishes = Vec::new();
        let mut remaining_tasks = VecDeque::new();

        while let Some(task) = self.tasks.pop_front() {
            if let KitchenTask::PlateAndServe { dish, table } = task {
                if table == target_table {
                    plated_dishes.push(format!("🍽️ Serving {} to table {}", dish, table));
                } else {
                    remaining_tasks.push_back(KitchenTask::PlateAndServe { dish, table });
                }
            } else {
                remaining_tasks.push_back(task);
            }
        }

        self.tasks = remaining_tasks;
        plated_dishes
    }
}

fn main() {
    let mut kitchen = KitchenQueue::new();

    // Add various tasks
    kitchen.add_task(KitchenTask::PrepIngredients {
        dish: "Salad".to_string(),
        time_estimate: 5
    });
    kitchen.add_task(KitchenTask::CookDish {
        dish: "Pasta".to_string(),
        cooking_time: 12
    });
    kitchen.add_task(KitchenTask::PrepIngredients {
        dish: "Soup".to_string(),
        time_estimate: 15
    });
    kitchen.add_task(KitchenTask::PlateAndServe {
        dish: "Quinoa Bowl".to_string(),
        table: 5
    });
    kitchen.add_task(KitchenTask::CookDish {
        dish: "Stir Fry".to_string(),
        cooking_time: 8
    });
    kitchen.add_task(KitchenTask::PlateAndServe {
        dish: "Veggie Burger".to_string(),
        table: 5
    });
    kitchen.add_task(KitchenTask::CleanEquipment {
        equipment: "Blender".to_string()
    });

    println!("🏃‍♂️ Kitchen Task Processing:");
    println!("Initial tasks: {}", kitchen.tasks.len());

    // Process quick prep tasks
    println!("\n⚡ Processing quick prep tasks:");
    let quick_preps = kitchen.process_quick_prep_tasks();
    for prep in quick_preps {
        println!("   {}", prep);
    }

    // Process cooking tasks one by one
    println!("\n🍳 Processing cooking tasks:");
    while let Some(cooking_result) = kitchen.process_next_cooking_task() {
        println!("   {}", cooking_result);
    }

    // Process all orders for table 5
    println!("\n🍽️ Processing table 5 orders:");
    let table_orders = kitchen.process_table_orders(5);
    for order in table_orders {
        println!("   {}", order);
    }

    println!("\n📋 Remaining tasks: {}", kitchen.tasks.len());
    for task in &kitchen.tasks {
        println!("   {:?}", task);
    }
}
```


## Best Practices and Common Patterns

### 1. Prefer Pattern Matching Over Conditional Chains

```rust
// ❌ Verbose conditional chain
fn bad_wine_recommendation(age: u32, budget: f64, occasion: &str) -> String {
    if age < 21 {
        "Sparkling grape juice".to_string()
    } else if age >= 21 && age <= 30 && budget <= 25.0 {
        if occasion == "date" {
            "Prosecco".to_string()
        } else {
            "House wine".to_string()
        }
    } else if age > 30 && budget > 50.0 && (occasion == "anniversary" || occasion == "celebration") {
        "Premium Champagne".to_string()
    } else {
        "Wine by the glass".to_string()
    }
}

// ✅ Clean pattern matching
fn good_wine_recommendation(customer: (u32, f64, &str)) -> String {
    match customer {
        (age, _, _) if age < 21 => "Sparkling grape juice".to_string(),
        (21..=30, budget, "date") if budget <= 25.0 => "Prosecco".to_string(),
        (21..=30, budget, _) if budget <= 25.0 => "House wine".to_string(),
        (age, budget, "anniversary" | "celebration") if age > 30 && budget > 50.0 => {
            "Premium Champagne".to_string()
        }
        _ => "Wine by the glass".to_string(),
    }
}
```


### 2. Use Exhaustive Matching

```rust
#[derive(Debug)]
enum MenuSection {
    Appetizers,
    Salads,
    MainCourses,
    Desserts,
    Beverages,
}

// ✅ Exhaustive - compiler ensures all cases are handled
fn get_section_description(section: MenuSection) -> &'static str {
    match section {
        MenuSection::Appetizers => "Light starters to awaken your palate",
        MenuSection::Salads => "Fresh, seasonal salads with house-made dressings",
        MenuSection::MainCourses => "Hearty, satisfying vegetarian entrees",
        MenuSection::Desserts => "Sweet endings to your meal",
        MenuSection::Beverages => "Curated wines, craft cocktails, and artisanal drinks",
        // Compiler ensures we handle all variants
    }
}

// ❌ Non-exhaustive - dangerous if new variants are added
fn bad_section_description(section: MenuSection) -> &'static str {
    match section {
        MenuSection::Appetizers => "Starters",
        MenuSection::MainCourses => "Mains",
        _ => "Other", // Loses specific information
    }
}
```


### 3. Order Patterns by Specificity

```rust
fn analyze_customer_order(
    dish_type: &str,
    dietary_needs: &[&str],
    urgency: u8,
    budget: f64,
) -> String {
    match (dish_type, dietary_needs, urgency, budget) {
        // Most specific cases first
        ("pasta", dietary, 9..=10, budget) if dietary.contains(&"gluten-free") && budget > 20.0 => {
            "🍝 Premium gluten-free pasta - made fresh with rice flour"
        }

        ("pasta", dietary, urgency, _) if dietary.contains(&"gluten-free") => {
            "🍝 Gluten-free pasta option available"
        }

        // Medium specificity
        ("salad", _, 8..=10, _) => {
            "🥗 Express salad - ready in 5 minutes"
        }

        ("salad", dietary, _, budget) if dietary.contains(&"vegan") && budget > 15.0 => {
            "🥗 Premium vegan salad with artisanal ingredients"
        }

        // Less specific
        (_, _, 9..=10, _) => {
            "⚡ Rush order - whatever's ready fastest"
        }

        (_, dietary, _, _) if dietary.contains(&"vegan") => {
            "🌱 Vegan options available"
        }

        // Most general case last
        _ => {
            "🍽️ Regular menu recommendations"
        }
    }
}
```


## Common Mistakes to Avoid

### Mistake 1: Unreachable Patterns

```rust
// ❌ Bad - second pattern will never match
fn bad_priority_matching(priority: u8) -> &'static str {
    match priority {
        1..=10 => "Valid priority",
        5 => "Medium priority", // ❌ Unreachable! Already covered by 1..=10
        _ => "Invalid priority",
    }
}

// ✅ Good - specific patterns before general ones
fn good_priority_matching(priority: u8) -> &'static str {
    match priority {
        5 => "Medium priority - right in the middle",
        1..=4 => "Low priority",
        6..=10 => "High priority",
        _ => "Invalid priority",
    }
}
```


### Mistake 2: Not Using Guards When Needed

```rust
// ❌ Bad - can't express complex conditions
fn bad_wine_selection(wine: (&str, u32, f64)) -> &'static str {
    match wine {
        ("red", _, _) => "Red wine selection", // Too general
        ("white", _, _) => "White wine selection", // Too general
        _ => "Other wines",
    }
}

// ✅ Good - guards allow complex conditions
fn good_wine_selection(wine: (&str, u32, f64)) -> &'static str {
    match wine {
        ("red", vintage, price) if vintage >= 2015 && price > 50.0 => {
            "Premium recent red wine"
        }
        ("red", vintage, _) if vintage < 2010 => {
            "Mature red wine - aged complexity"
        }
        ("red", _, _) => {
            "Standard red wine selection"
        }
        ("white", _, price) if price > 30.0 => {
            "Premium white wine"
        }
        ("white", _, _) => {
            "Standard white wine selection"
        }
        _ => "Specialty wine - consult sommelier",
    }
}
```


### Mistake 3: Overcomplicating Simple Cases

```rust
// ❌ Overcomplicated for simple boolean logic
fn bad_seating_check(party_size: u32, has_reservation: bool) -> String {
    match (party_size, has_reservation) {
        (size, reservation) if size <= 4 && reservation => "Table ready".to_string(),
        (size, reservation) if size <= 4 && !reservation => "Please wait".to_string(),
        (size, reservation) if size > 4 && reservation => "Large table ready".to_string(),
        (size, reservation) if size > 4 && !reservation => "Large party wait".to_string(),
        _ => unreachable!(),
    }
}

// ✅ Simple and clear
fn good_seating_check(party_size: u32, has_reservation: bool) -> String {
    match (party_size, has_reservation) {
        (1..=4, true) => "Table ready".to_string(),
        (1..=4, false) => "Please wait for next available table".to_string(),
        (_, true) => "Large table ready".to_string(),
        (_, false) => "Large party - please wait for availability".to_string(),
    }
}
```


## Summary and Key Takeaways

### **Mental Model: The Master Sommelier**

Remember the sommelier analogy:

- 🍷 **Pattern matching** = Expertly analyzing wine characteristics
- 🎯 **Destructuring** = Breaking down complex flavor profiles
- 🔍 **Guards** = Final taste test for perfect pairing
- 📋 **Exhaustive matching** = Considering every possible wine type
- ✨ **Elegant handling** = Sophisticated recommendations


### **Essential Principles**

1. **Patterns are powerful** - Go beyond simple equality checks
2. **Order matters** - Most specific patterns first
3. **Use guards wisely** - Add conditions when needed
4. **Destructure deeply** - Extract exactly what you need
5. **Be exhaustive** - Handle all possible cases
6. **Stay readable** - Complex patterns should still be clear

### **Pattern Matching Toolkit**

| **Pattern Type** | **Use Case** | **Example** |
| :-- | :-- | :-- |
| **Literal** | Exact values | `"vegetarian"` |
| **Range** | Value ranges | `1..=5`, `'a'..='z'` |
| **OR (`\|`)** | Multiple options | `Red \| Green \| Blue` |
| **Destructuring** | Extract data | `Some(value)`, `Point { x, y }` |
| **Guards** | Conditions | `x if x > 0` |
| **Binding (`@`)** | Capture matched values | `x @ 1..=5` |
| **Wildcards** | Ignore parts | `_`, `Some(_)` |
| **References** | Borrow patterns | `&value`, `ref x` |

### **Quick Decision Guide**

```rust
// Choose the right pattern for your needs:

match value {
    // Specific cases first
    42 => "The answer",

    // Ranges for numeric/char data
    1..=10 => "Small number",

    // Multiple patterns with |
    "red" | "green" | "blue" => "Primary color",

    // Destructuring for complex data
    Point { x: 0, y } => format!("On Y-axis at {}", y),

    // Guards for conditions
    x if x > 100 => "Large number",

    // Binding to capture and use
    x @ 50..=99 => format!("Medium number: {}", x),

    // Catch-all
    _ => "Something else",
}
```


### **Best Practices Checklist**

**✅ DO:**

- Order patterns from specific to general
- Use guards for complex conditions
- Destructure to access inner data
- Handle all cases exhaustively
- Use meaningful variable names in patterns
- Combine patterns with `|` when appropriate

**❌ DON'T:**

- Put general patterns before specific ones
- Create unreachable patterns
- Overcomplicate simple boolean logic
- Ignore the wildcard pattern when needed
- Use pattern matching for simple equality checks
- Forget to handle edge cases

**Advanced pattern matching is like having a master sommelier's palate** - you can detect subtle differences, categorize complex combinations, and make sophisticated decisions based on multiple factors simultaneously. It transforms your Rust code from simple if-else chains into elegant detection systems that handle real-world complexity with grace and precision.

Just as a master sommelier can instantly identify the perfect wine pairing by analyzing grape variety, region, vintage, production method, and customer preferences all at once, advanced Rust patterns let you write code that elegantly handles complex data structures by examining their shape, content, and relationships in a single, comprehensive expression!

1. https://doc.rust-lang.org/book/ch19-00-patterns.html
2. https://doc.rust-lang.org/book/ch19-03-pattern-syntax.html
3. https://www.freecodecamp.org/news/rust-tutorial-build-a-json-parser/
4. https://www.youtube.com/watch?v=FiBO0tXbxXY
5. https://dev.to/lymah/week-3-of-learning-rust-match-patterns-and-the-power-of-methods-3ed4
6. https://dev.to/ajtech0001/enums-and-pattern-matching-in-rust-a-deep-dive-into-type-safety-and-control-flow-1gjp
7. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/match.html
8. https://doc.rust-lang.org/book/ch06-02-match.html
9. https://notes.iveselov.info/programming/refs-and-pattern-matching-in-rust
10. https://www.rustfinity.com/learn/rust/the-programming-basics/control-flow/pattern-matching
11. https://masteringbackend.com/posts/how-to-use-the-matches-macro-pattern-matching
