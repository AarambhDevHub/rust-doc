+++
title = "Struct Composition"
description = "Explore struct composition in Rust as a way to build complex data types by combining simpler structs, promoting modularity and reuse."
date = 2025-08-04
draft = false
weight = 4
+++

# Struct Composition in Rust: Comprehensive Documentation for Beginners

Understanding struct composition is like learning to **create the perfect vegetarian salad bowl** - you combine different fresh ingredients (structs) together to build something more complex and delicious than any single ingredient could be on its own. Just as a great salad brings together lettuce, tomatoes, dressing, and toppings into one harmonious dish, struct composition lets you build complex data types by assembling smaller, focused structs.

## The Vegetarian Salad Bowl Analogy 🥗

### Imagine You're Creating the Ultimate Veggie Salad

**Traditional Approach (Flat Structure):**

```rust
// ❌ Everything mixed together - hard to organize
struct MessySalad {
    lettuce_type: String,
    lettuce_fresh: bool,
    tomato_count: u32,
    tomato_color: String,
    cucumber_sliced: bool,
    dressing_type: String,
    dressing_vegan: bool,
    dressing_calories: u32,
    // ... 20 more fields mixed together!
}
```

**Composition Approach (Organized Ingredients):**

```rust
// ✅ Each ingredient is its own component
struct Greens {
    lettuce_type: String,
    spinach_amount: u32,
    freshness_level: u8,
}

struct Vegetables {
    tomatoes: u32,
    cucumbers: u32,
    bell_peppers: u32,
    carrots: bool,
}

struct Dressing {
    name: String,
    is_vegan: bool,
    calories_per_serving: u32,
}

struct VeggieSalad {
    name: String,
    greens: Greens,           // 🥬 Green component
    vegetables: Vegetables,   // 🥕 Veggie component
    dressing: Dressing,       // 🥄 Dressing component
    serving_size: String,
}
```

**Why This Works Better:**

- 🧩 **Modular ingredients** - Each component has a clear purpose
- 🔄 **Reusable components** - Use `Dressing` in multiple salad types
- 📖 **Easy to understand** - Clear organization like a recipe
- 🛠️ **Easy to maintain** - Change one ingredient without affecting others


## What is Struct Composition?

### Core Concept

**Struct composition** is the practice of building complex data types by including other structs as fields. Instead of creating one massive struct with dozens of fields, you create smaller, focused structs and combine them like ingredients in a recipe.

**Key Benefits:**

- **Modularity** - Each struct has a single responsibility
- **Reusability** - Components can be used in multiple contexts
- **Maintainability** - Easy to update individual components
- **Readability** - Clear organization and relationships
- **Rust-friendly** - Works perfectly with ownership and borrowing


### The Recipe Card Metaphor

Think of struct composition like organizing recipe cards:

```rust
// Each "recipe card" is a focused struct
struct Ingredient {
    name: String,
    quantity: String,
    is_organic: bool,
}

struct NutritionalInfo {
    calories_per_serving: u32,
    protein_grams: f32,
    fiber_grams: f32,
    vitamins: Vec<String>,
}

struct CookingInstructions {
    prep_time_minutes: u32,
    difficulty_level: String,
    steps: Vec<String>,
}

// The complete "recipe book" combines all cards
struct VegetarianRecipe {
    name: String,
    ingredients: Vec<Ingredient>,     // 📝 Ingredient cards
    nutrition: NutritionalInfo,       // 📊 Nutrition card
    instructions: CookingInstructions, // 👨‍🍳 Cooking card
    servings: u32,
    cuisine_type: String,
}
```

Each "card" (struct) can be understood, updated, and reused independently!

## Basic Composition Examples

### Simple Vegetarian Meal Planning

```rust
#[derive(Debug)]
struct Vegetables {
    primary: String,
    secondary: Vec<String>,
    is_seasonal: bool,
}

#[derive(Debug)]
struct Seasoning {
    herbs: Vec<String>,
    spices: Vec<String>,
    salt_level: u8, // 1-10 scale
}

#[derive(Debug)]
struct CookingMethod {
    technique: String,
    temperature: Option<u32>, // None for raw dishes
    time_minutes: u32,
}

#[derive(Debug)]
struct VegetarianDish {
    name: String,
    vegetables: Vegetables,
    seasoning: Seasoning,
    cooking: CookingMethod,
    is_vegan: bool,
}

fn main() {
    let stir_fry = VegetarianDish {
        name: "Rainbow Veggie Stir-Fry".to_string(),
        vegetables: Vegetables {
            primary: "Bell peppers".to_string(),
            secondary: vec![
                "Broccoli".to_string(),
                "Carrots".to_string(),
                "Snow peas".to_string(),
            ],
            is_seasonal: true,
        },
        seasoning: Seasoning {
            herbs: vec!["Basil".to_string(), "Cilantro".to_string()],
            spices: vec!["Ginger".to_string(), "Garlic".to_string()],
            salt_level: 6,
        },
        cooking: CookingMethod {
            technique: "Stir-frying".to_string(),
            temperature: Some(375),
            time_minutes: 12,
        },
        is_vegan: true,
    };

    println!("🍽️ Dish: {}", stir_fry.name);
    println!("🥬 Main veggie: {}", stir_fry.vegetables.primary);
    println!("🌿 Herbs: {:?}", stir_fry.seasoning.herbs);
    println!("👨‍🍳 Method: {} for {} minutes",
             stir_fry.cooking.technique,
             stir_fry.cooking.time_minutes);
    println!("🌱 Vegan: {}", stir_fry.is_vegan);
}
```


### Garden Planning System

```rust
#[derive(Debug)]
struct PlantInfo {
    name: String,
    variety: String,
    days_to_harvest: u32,
    spacing_inches: u32,
}

#[derive(Debug)]
struct GrowingConditions {
    sunlight_hours: u32,
    water_frequency_days: u32,
    soil_ph_min: f32,
    soil_ph_max: f32,
    companion_plants: Vec<String>,
}

#[derive(Debug)]
struct PlantingSchedule {
    seed_start_date: String,
    transplant_date: Option<String>,
    expected_harvest: String,
}

#[derive(Debug)]
struct GardenPlant {
    plant: PlantInfo,
    conditions: GrowingConditions,
    schedule: PlantingSchedule,
    plot_location: String,
}

impl GardenPlant {
    fn new(
        name: &str,
        variety: &str,
        plot_location: &str
    ) -> Self {
        GardenPlant {
            plant: PlantInfo {
                name: name.to_string(),
                variety: variety.to_string(),
                days_to_harvest: 0, // Will be set later
                spacing_inches: 12,
            },
            conditions: GrowingConditions {
                sunlight_hours: 6,
                water_frequency_days: 2,
                soil_ph_min: 6.0,
                soil_ph_max: 7.0,
                companion_plants: Vec::new(),
            },
            schedule: PlantingSchedule {
                seed_start_date: "TBD".to_string(),
                transplant_date: None,
                expected_harvest: "TBD".to_string(),
            },
            plot_location: plot_location.to_string(),
        }
    }

    fn days_to_harvest(mut self, days: u32) -> Self {
        self.plant.days_to_harvest = days;
        self
    }

    fn sunlight_needs(mut self, hours: u32) -> Self {
        self.conditions.sunlight_hours = hours;
        self
    }

    fn add_companion(mut self, companion: &str) -> Self {
        self.conditions.companion_plants.push(companion.to_string());
        self
    }

    fn plant_summary(&self) -> String {
        format!(
            "🌱 {} {} in plot '{}' - {} days to harvest, needs {}h sun",
            self.plant.name,
            self.plant.variety,
            self.plot_location,
            self.plant.days_to_harvest,
            self.conditions.sunlight_hours
        )
    }
}

fn main() {
    let tomato_plant = GardenPlant::new("Tomato", "Cherry", "Plot A")
        .days_to_harvest(75)
        .sunlight_needs(8)
        .add_companion("Basil")
        .add_companion("Marigold");

    let lettuce_plant = GardenPlant::new("Lettuce", "Buttercrunch", "Plot B")
        .days_to_harvest(45)
        .sunlight_needs(4)
        .add_companion("Radish");

    println!("{}", tomato_plant.plant_summary());
    println!("{}", lettuce_plant.plant_summary());

    println!("\nTomato companions: {:?}", tomato_plant.conditions.companion_plants);
}
```


## Advanced Composition Patterns

### Restaurant Management System

Let's build a comprehensive vegetarian restaurant system using composition:

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
struct Ingredient {
    name: String,
    quantity_available: u32,
    unit: String, // "cups", "lbs", "pieces", etc.
    price_per_unit: f64,
    is_organic: bool,
    allergens: Vec<String>,
}

#[derive(Debug)]
struct Recipe {
    name: String,
    ingredients: Vec<(String, u32)>, // ingredient name, quantity needed
    prep_time_minutes: u32,
    cooking_time_minutes: u32,
    difficulty: u8, // 1-5 scale
    instructions: Vec<String>,
}

#[derive(Debug)]
struct NutritionalInfo {
    calories_per_serving: u32,
    protein_grams: f32,
    carbs_grams: f32,
    fat_grams: f32,
    fiber_grams: f32,
    vitamins: HashMap<String, String>,
}

#[derive(Debug)]
struct MenuItem {
    name: String,
    description: String,
    recipe: Recipe,
    nutrition: NutritionalInfo,
    price: f64,
    category: String, // "Appetizer", "Main", "Dessert", etc.
    dietary_tags: Vec<String>, // "Vegan", "Gluten-Free", etc.
}

#[derive(Debug)]
struct Kitchen {
    ingredients: HashMap<String, Ingredient>,
    menu_items: Vec<MenuItem>,
    daily_specials: Vec<String>,
}

impl Kitchen {
    fn new() -> Self {
        Kitchen {
            ingredients: HashMap::new(),
            menu_items: Vec::new(),
            daily_specials: Vec::new(),
        }
    }

    fn add_ingredient(&mut self, ingredient: Ingredient) {
        self.ingredients.insert(ingredient.name.clone(), ingredient);
    }

    fn add_menu_item(&mut self, item: MenuItem) {
        self.menu_items.push(item);
    }

    fn can_make_dish(&self, dish_name: &str) -> bool {
        if let Some(menu_item) = self.menu_items.iter().find(|item| item.name == dish_name) {
            // Check if we have all required ingredients
            for (ingredient_name, quantity_needed) in &menu_item.recipe.ingredients {
                if let Some(available_ingredient) = self.ingredients.get(ingredient_name) {
                    if available_ingredient.quantity_available < *quantity_needed {
                        return false;
                    }
                } else {
                    return false;
                }
            }
            true
        } else {
            false
        }
    }

    fn calculate_dish_cost(&self, dish_name: &str) -> Option<f64> {
        if let Some(menu_item) = self.menu_items.iter().find(|item| item.name == dish_name) {
            let mut total_cost = 0.0;
            for (ingredient_name, quantity_needed) in &menu_item.recipe.ingredients {
                if let Some(ingredient) = self.ingredients.get(ingredient_name) {
                    total_cost += ingredient.price_per_unit * (*quantity_needed as f64);
                }
            }
            Some(total_cost)
        } else {
            None
        }
    }

    fn get_dishes_by_dietary_requirement(&self, requirement: &str) -> Vec<&MenuItem> {
        self.menu_items
            .iter()
            .filter(|item| item.dietary_tags.contains(&requirement.to_string()))
            .collect()
    }
}

fn main() {
    let mut kitchen = Kitchen::new();

    // Add ingredients to inventory
    kitchen.add_ingredient(Ingredient {
        name: "Quinoa".to_string(),
        quantity_available: 50,
        unit: "cups".to_string(),
        price_per_unit: 0.75,
        is_organic: true,
        allergens: vec![],
    });

    kitchen.add_ingredient(Ingredient {
        name: "Black beans".to_string(),
        quantity_available: 30,
        unit: "cups".to_string(),
        price_per_unit: 0.50,
        is_organic: true,
        allergens: vec![],
    });

    kitchen.add_ingredient(Ingredient {
        name: "Avocado".to_string(),
        quantity_available: 20,
        unit: "pieces".to_string(),
        price_per_unit: 1.25,
        is_organic: true,
        allergens: vec![],
    });

    // Create a menu item using composition
    let quinoa_bowl = MenuItem {
        name: "Superfood Quinoa Bowl".to_string(),
        description: "Nutritious bowl with quinoa, black beans, and fresh avocado".to_string(),
        recipe: Recipe {
            name: "Quinoa Bowl".to_string(),
            ingredients: vec![
                ("Quinoa".to_string(), 2),
                ("Black beans".to_string(), 1),
                ("Avocado".to_string(), 1),
            ],
            prep_time_minutes: 10,
            cooking_time_minutes: 20,
            difficulty: 2,
            instructions: vec![
                "Rinse quinoa and cook according to package directions".to_string(),
                "Heat black beans in a pan".to_string(),
                "Slice avocado".to_string(),
                "Combine all ingredients in a bowl".to_string(),
            ],
        },
        nutrition: NutritionalInfo {
            calories_per_serving: 450,
            protein_grams: 18.0,
            carbs_grams: 65.0,
            fat_grams: 15.0,
            fiber_grams: 12.0,
            vitamins: {
                let mut vitamins = HashMap::new();
                vitamins.insert("Vitamin K".to_string(), "High".to_string());
                vitamins.insert("Folate".to_string(), "High".to_string());
                vitamins
            },
        },
        price: 14.50,
        category: "Main Course".to_string(),
        dietary_tags: vec!["Vegan".to_string(), "Gluten-Free".to_string(), "High-Protein".to_string()],
    };

    kitchen.add_menu_item(quinoa_bowl);

    // Test kitchen functionality
    let dish_name = "Superfood Quinoa Bowl";

    println!("🍽️ Kitchen Menu Analysis");
    println!("Can make {}? {}", dish_name, kitchen.can_make_dish(dish_name));

    if let Some(cost) = kitchen.calculate_dish_cost(dish_name) {
        println!("💰 Cost to make: ${:.2}", cost);
    }

    let vegan_dishes = kitchen.get_dishes_by_dietary_requirement("Vegan");
    println!("🌱 Vegan dishes available: {}", vegan_dishes.len());

    for dish in vegan_dishes {
        println!("   - {} (${:.2})", dish.name, dish.price);
        println!("     Calories: {}, Protein: {}g",
                 dish.nutrition.calories_per_serving,
                 dish.nutrition.protein_grams);
    }
}
```


## Composition vs Inheritance

### Why Composition Works Better in Rust

**Traditional OOP Inheritance Issues:**

```rust
// ❌ Rust doesn't support this - inheritance doesn't exist
/*
class Vegetable {
    name: String,
    color: String,
}

class Tomato extends Vegetable {
    // Inherits name and color
    acidity_level: f32,
}

class Carrot extends Vegetable {
    // Inherits name and color
    sweetness_level: f32,
}
*/
```

**Rust Composition Solution:**

```rust
// ✅ This works great in Rust
#[derive(Debug)]
struct BasicVegetableInfo {
    name: String,
    color: String,
    growing_season: String,
}

#[derive(Debug)]
struct Tomato {
    basic_info: BasicVegetableInfo,  // Composition instead of inheritance
    acidity_level: f32,
    variety: String,
}

#[derive(Debug)]
struct Carrot {
    basic_info: BasicVegetableInfo,  // Same basic info, different specialization
    sweetness_level: f32,
    length_inches: f32,
}

// Use traits for shared behavior
trait VegetableInfo {
    fn get_name(&self) -> &str;
    fn get_color(&self) -> &str;
    fn is_in_season(&self, current_season: &str) -> bool;
}

impl VegetableInfo for Tomato {
    fn get_name(&self) -> &str {
        &self.basic_info.name
    }

    fn get_color(&self) -> &str {
        &self.basic_info.color
    }

    fn is_in_season(&self, current_season: &str) -> bool {
        self.basic_info.growing_season == current_season
    }
}

impl VegetableInfo for Carrot {
    fn get_name(&self) -> &str {
        &self.basic_info.name
    }

    fn get_color(&self) -> &str {
        &self.basic_info.color
    }

    fn is_in_season(&self, current_season: &str) -> bool {
        self.basic_info.growing_season == current_season
    }
}

impl Tomato {
    fn new(name: String, color: String, variety: String, acidity: f32) -> Self {
        Tomato {
            basic_info: BasicVegetableInfo {
                name,
                color,
                growing_season: "Summer".to_string(),
            },
            acidity_level: acidity,
            variety,
        }
    }

    fn taste_profile(&self) -> String {
        match self.acidity_level {
            0.0..=3.0 => "Mild and sweet".to_string(),
            3.1..=6.0 => "Balanced".to_string(),
            _ => "Tangy and acidic".to_string(),
        }
    }
}

impl Carrot {
    fn new(name: String, color: String, sweetness: f32, length: f32) -> Self {
        Carrot {
            basic_info: BasicVegetableInfo {
                name,
                color,
                growing_season: "Fall".to_string(),
            },
            sweetness_level: sweetness,
            length_inches: length,
        }
    }

    fn size_category(&self) -> &str {
        match self.length_inches {
            0.0..=4.0 => "Baby carrot",
            4.1..=8.0 => "Medium carrot",
            _ => "Large carrot",
        }
    }
}

fn main() {
    let cherry_tomato = Tomato::new(
        "Cherry Tomato".to_string(),
        "Red".to_string(),
        "Sweet 100".to_string(),
        4.2
    );

    let purple_carrot = Carrot::new(
        "Purple Haze Carrot".to_string(),
        "Purple".to_string(),
        7.5,
        6.0
    );

    // Use shared behavior through traits
    println!("🍅 {} is {} and {}",
             cherry_tomato.get_name(),
             cherry_tomato.get_color(),
             cherry_tomato.taste_profile());

    println!("🥕 {} is {} and classified as {}",
             purple_carrot.get_name(),
             purple_carrot.get_color(),
             purple_carrot.size_category());

    // Check seasons
    println!("Is {} in season during summer? {}",
             cherry_tomato.get_name(),
             cherry_tomato.is_in_season("Summer"));

    println!("Is {} in season during fall? {}",
             purple_carrot.get_name(),
             purple_carrot.is_in_season("Fall"));
}
```

**Advantages of Composition:**

- 🔧 **Flexible design** - Mix and match components as needed
- 🧩 **Clear ownership** - No complex inheritance chains
- 🔄 **Easy to test** - Test each component independently
- 📦 **Reusable components** - Use `BasicVegetableInfo` everywhere
- 🦀 **Rust-friendly** - Works perfectly with ownership rules


## Advanced Patterns

### Optional Components Pattern

Sometimes you want optional components in your composition:

```rust
#[derive(Debug)]
struct BaseVegetable {
    name: String,
    scientific_name: String,
    color: String,
}

#[derive(Debug)]
struct NutritionalFacts {
    calories_per_100g: u32,
    vitamin_c_mg: f32,
    fiber_g: f32,
    antioxidants: Vec<String>,
}

#[derive(Debug)]
struct GrowingInfo {
    climate_zones: Vec<u8>,
    soil_ph_range: (f32, f32),
    days_to_maturity: u32,
    companion_plants: Vec<String>,
}

#[derive(Debug)]
struct CulinaryUses {
    cooking_methods: Vec<String>,
    flavor_profile: String,
    pairs_well_with: Vec<String>,
    storage_tips: String,
}

#[derive(Debug)]
struct VegetableProfile {
    base: BaseVegetable,
    nutrition: Option<NutritionalFacts>,    // Optional - might not have data
    growing: Option<GrowingInfo>,           // Optional - might not grow it
    culinary: Option<CulinaryUses>,         // Optional - might just grow for sale
}

impl VegetableProfile {
    fn new(name: &str, scientific_name: &str, color: &str) -> Self {
        VegetableProfile {
            base: BaseVegetable {
                name: name.to_string(),
                scientific_name: scientific_name.to_string(),
                color: color.to_string(),
            },
            nutrition: None,
            growing: None,
            culinary: None,
        }
    }

    fn with_nutrition(mut self, nutrition: NutritionalFacts) -> Self {
        self.nutrition = Some(nutrition);
        self
    }

    fn with_growing_info(mut self, growing: GrowingInfo) -> Self {
        self.growing = Some(growing);
        self
    }

    fn with_culinary_info(mut self, culinary: CulinaryUses) -> Self {
        self.culinary = Some(culinary);
        self
    }

    fn is_high_vitamin_c(&self) -> bool {
        self.nutrition
            .as_ref()
            .map(|n| n.vitamin_c_mg > 50.0)
            .unwrap_or(false)
    }

    fn can_grow_in_zone(&self, zone: u8) -> bool {
        self.growing
            .as_ref()
            .map(|g| g.climate_zones.contains(&zone))
            .unwrap_or(false)
    }

    fn get_summary(&self) -> String {
        let mut summary = format!("🥬 {} ({})", self.base.name, self.base.color);

        if let Some(ref nutrition) = self.nutrition {
            summary.push_str(&format!(" - {} cal/100g", nutrition.calories_per_100g));
        }

        if let Some(ref growing) = self.growing {
            summary.push_str(&format!(" - {} days to maturity", growing.days_to_maturity));
        }

        if let Some(ref culinary) = self.culinary {
            summary.push_str(&format!(" - {}", culinary.flavor_profile));
        }

        summary
    }
}

fn main() {
    // Create vegetables with different levels of information
    let basic_spinach = VegetableProfile::new(
        "Spinach",
        "Spinacia oleracea",
        "Dark Green"
    );

    let detailed_tomato = VegetableProfile::new(
        "Tomato",
        "Solanum lycopersicum",
        "Red"
    )
    .with_nutrition(NutritionalFacts {
        calories_per_100g: 18,
        vitamin_c_mg: 14.0,
        fiber_g: 1.2,
        antioxidants: vec!["Lycopene".to_string(), "Beta-carotene".to_string()],
    })
    .with_growing_info(GrowingInfo {
        climate_zones: vec![3, 4, 5, 6, 7, 8, 9],
        soil_ph_range: (6.0, 6.8),
        days_to_maturity: 75,
        companion_plants: vec!["Basil".to_string(), "Marigold".to_string()],
    })
    .with_culinary_info(CulinaryUses {
        cooking_methods: vec!["Raw".to_string(), "Grilled".to_string(), "Roasted".to_string()],
        flavor_profile: "Sweet and slightly acidic".to_string(),
        pairs_well_with: vec!["Basil".to_string(), "Mozzarella".to_string(), "Olive oil".to_string()],
        storage_tips: "Store at room temperature until ripe, then refrigerate".to_string(),
    });

    println!("Basic profile: {}", basic_spinach.get_summary());
    println!("Detailed profile: {}", detailed_tomato.get_summary());

    println!("Can tomato grow in zone 6? {}", detailed_tomato.can_grow_in_zone(6));
    println!("Is tomato high in vitamin C? {}", detailed_tomato.is_high_vitamin_c());
}
```


### Component-Based Entity System

For more complex scenarios, you can create a flexible component system:

```rust
use std::collections::HashMap;

// Define different component types
#[derive(Debug, Clone)]
struct Transform {
    x: f32,
    y: f32,
    rotation: f32,
}

#[derive(Debug, Clone)]
struct PlantGrowth {
    growth_stage: String,
    days_growing: u32,
    height_cm: f32,
    health: u8, // 0-100
}

#[derive(Debug, Clone)]
struct WaterNeeds {
    days_since_watered: u32,
    frequency_days: u32,
    amount_ml: u32,
}

#[derive(Debug, Clone)]
struct Harvestable {
    is_ready: bool,
    harvest_yield: u32,
    quality_grade: String,
}

// Entity that can have multiple components
#[derive(Debug)]
struct GardenPlant {
    id: u32,
    name: String,
    // Optional components - a plant doesn't need all of them
    transform: Option<Transform>,
    growth: Option<PlantGrowth>,
    water_needs: Option<WaterNeeds>,
    harvestable: Option<Harvestable>,
}

impl GardenPlant {
    fn new(id: u32, name: String) -> Self {
        GardenPlant {
            id,
            name,
            transform: None,
            growth: None,
            water_needs: None,
            harvestable: None,
        }
    }

    fn add_transform(mut self, x: f32, y: f32) -> Self {
        self.transform = Some(Transform { x, y, rotation: 0.0 });
        self
    }

    fn add_growth(mut self, stage: &str) -> Self {
        self.growth = Some(PlantGrowth {
            growth_stage: stage.to_string(),
            days_growing: 0,
            height_cm: 1.0,
            health: 100,
        });
        self
    }

    fn add_water_needs(mut self, frequency: u32, amount: u32) -> Self {
        self.water_needs = Some(WaterNeeds {
            days_since_watered: 0,
            frequency_days: frequency,
            amount_ml: amount,
        });
        self
    }

    fn make_harvestable(mut self) -> Self {
        self.harvestable = Some(Harvestable {
            is_ready: false,
            harvest_yield: 0,
            quality_grade: "Developing".to_string(),
        });
        self
    }

    fn advance_day(&mut self) {
        // Update growth component
        if let Some(ref mut growth) = self.growth {
            growth.days_growing += 1;
            growth.height_cm += 0.5; // Grow a bit each day

            // Update growth stage
            match growth.days_growing {
                1..=14 => growth.growth_stage = "Seedling".to_string(),
                15..=45 => growth.growth_stage = "Vegetative".to_string(),
                46..=75 => growth.growth_stage = "Flowering".to_string(),
                _ => growth.growth_stage = "Mature".to_string(),
            }
        }

        // Update water needs
        if let Some(ref mut water) = self.water_needs {
            water.days_since_watered += 1;
        }

        // Update harvestable status
        if let (Some(ref growth), Some(ref mut harvestable)) = (&self.growth, &mut self.harvestable) {
            if growth.days_growing >= 60 && growth.health > 70 {
                harvestable.is_ready = true;
                harvestable.harvest_yield = (growth.height_cm * 0.1) as u32;
                harvestable.quality_grade = match growth.health {
                    90..=100 => "Premium".to_string(),
                    75..=89 => "Grade A".to_string(),
                    60..=74 => "Grade B".to_string(),
                    _ => "Grade C".to_string(),
                };
            }
        }
    }

    fn water(&mut self) {
        if let Some(ref mut water) = self.water_needs {
            water.days_since_watered = 0;
        }

        if let Some(ref mut growth) = self.growth {
            if growth.health < 100 {
                growth.health = (growth.health + 10).min(100);
            }
        }
    }

    fn needs_water(&self) -> bool {
        self.water_needs
            .as_ref()
            .map(|w| w.days_since_watered >= w.frequency_days)
            .unwrap_or(false)
    }

    fn is_ready_to_harvest(&self) -> bool {
        self.harvestable
            .as_ref()
            .map(|h| h.is_ready)
            .unwrap_or(false)
    }

    fn get_status(&self) -> String {
        let mut status = format!("🌱 {}", self.name);

        if let Some(ref growth) = self.growth {
            status.push_str(&format!(" - {} stage, {}cm tall",
                                   growth.growth_stage, growth.height_cm));
        }

        if self.needs_water() {
            status.push_str(" - 💧 Needs water!");
        }

        if self.is_ready_to_harvest() {
            if let Some(ref harvestable) = self.harvestable {
                status.push_str(&format!(" - ✅ Ready to harvest! ({} yield, {} quality)",
                                       harvestable.harvest_yield, harvestable.quality_grade));
            }
        }

        status
    }
}

struct Garden {
    plants: HashMap<u32, GardenPlant>,
    day_counter: u32,
}

impl Garden {
    fn new() -> Self {
        Garden {
            plants: HashMap::new(),
            day_counter: 0,
        }
    }

    fn add_plant(&mut self, plant: GardenPlant) {
        self.plants.insert(plant.id, plant);
    }

    fn advance_day(&mut self) {
        self.day_counter += 1;
        println!("\n🌅 Day {} in the garden", self.day_counter);

        for plant in self.plants.values_mut() {
            plant.advance_day();
        }
    }

    fn water_all_plants(&mut self) {
        for plant in self.plants.values_mut() {
            if plant.needs_water() {
                plant.water();
                println!("💧 Watered {}", plant.name);
            }
        }
    }

    fn get_garden_status(&self) -> String {
        let mut status = format!("🏡 Garden Status (Day {})\n", self.day_counter);

        for plant in self.plants.values() {
            status.push_str(&format!("  {}\n", plant.get_status()));
        }

        status
    }
}

fn main() {
    let mut garden = Garden::new();

    // Create plants with different component combinations
    let tomato = GardenPlant::new(1, "Cherry Tomato".to_string())
        .add_transform(2.0, 3.0)
        .add_growth("Seed")
        .add_water_needs(3, 250)
        .make_harvestable();

    let lettuce = GardenPlant::new(2, "Buttercrunch Lettuce".to_string())
        .add_transform(1.0, 1.0)
        .add_growth("Seed")
        .add_water_needs(2, 150)
        .make_harvestable();

    let basil = GardenPlant::new(3, "Sweet Basil".to_string())
        .add_growth("Seedling")
        .add_water_needs(2, 100);

    garden.add_plant(tomato);
    garden.add_plant(lettuce);
    garden.add_plant(basil);

    // Simulate garden over time
    for day in 1..=70 {
        garden.advance_day();

        if day % 7 == 0 {  // Weekly garden check
            println!("{}", garden.get_garden_status());
            garden.water_all_plants();
        }
    }

    println!("{}", garden.get_garden_status());
}
```


## Best Practices and Common Patterns

### 1. Single Responsibility Principle

```rust
// ✅ Good - each struct has one clear purpose
struct PersonalInfo {
    first_name: String,
    last_name: String,
    birth_date: String,
}

struct ContactInfo {
    email: String,
    phone: String,
    address: String,
}

struct DietaryPreferences {
    is_vegetarian: bool,
    is_vegan: bool,
    allergies: Vec<String>,
    preferred_cuisines: Vec<String>,
}

struct Customer {
    personal: PersonalInfo,
    contact: ContactInfo,
    dietary: DietaryPreferences,
}

// ❌ Bad - everything mixed together
struct BadCustomer {
    first_name: String,
    last_name: String,
    birth_date: String,
    email: String,
    phone: String,
    address: String,
    is_vegetarian: bool,
    is_vegan: bool,
    allergies: Vec<String>,
    preferred_cuisines: Vec<String>,
    // ... many more fields mixed together
}
```


### 2. Builder Pattern with Composition

```rust
struct RecipeBuilder {
    name: Option<String>,
    ingredients: Vec<Ingredient>,
    instructions: Vec<String>,
    nutrition: Option<NutritionalInfo>,
    timing: Option<CookingTime>,
}

struct CookingTime {
    prep_minutes: u32,
    cook_minutes: u32,
    total_minutes: u32,
}

impl RecipeBuilder {
    fn new() -> Self {
        RecipeBuilder {
            name: None,
            ingredients: Vec::new(),
            instructions: Vec::new(),
            nutrition: None,
            timing: None,
        }
    }

    fn name(mut self, name: &str) -> Self {
        self.name = Some(name.to_string());
        self
    }

    fn add_ingredient(mut self, name: &str, amount: &str) -> Self {
        self.ingredients.push(Ingredient {
            name: name.to_string(),
            quantity: amount.to_string(),
            is_organic: false,
        });
        self
    }

    fn timing(mut self, prep: u32, cook: u32) -> Self {
        self.timing = Some(CookingTime {
            prep_minutes: prep,
            cook_minutes: cook,
            total_minutes: prep + cook,
        });
        self
    }

    fn build(self) -> Result<Recipe, String> {
        let name = self.name.ok_or("Recipe name is required")?;

        Ok(Recipe {
            name,
            ingredients: self.ingredients,
            prep_time_minutes: self.timing.as_ref().map(|t| t.prep_minutes).unwrap_or(0),
            cooking_time_minutes: self.timing.as_ref().map(|t| t.cook_minutes).unwrap_or(0),
            difficulty: 1,
            instructions: self.instructions,
        })
    }
}
```


### 3. Trait-Based Composition

```rust
// Define traits for different capabilities
trait Growable {
    fn grow(&mut self, days: u32);
    fn is_mature(&self) -> bool;
}

trait Harvestable {
    fn can_harvest(&self) -> bool;
    fn harvest(&mut self) -> Option<u32>; // Returns yield
}

trait Watered {
    fn water(&mut self);
    fn needs_water(&self) -> bool;
}

// Implement traits for composed structs
impl Growable for GardenPlant {
    fn grow(&mut self, days: u32) {
        for _ in 0..days {
            self.advance_day();
        }
    }

    fn is_mature(&self) -> bool {
        self.growth
            .as_ref()
            .map(|g| g.days_growing >= 60)
            .unwrap_or(false)
    }
}

impl Harvestable for GardenPlant {
    fn can_harvest(&self) -> bool {
        self.is_ready_to_harvest()
    }

    fn harvest(&mut self) -> Option<u32> {
        if let Some(ref mut harvestable) = self.harvestable {
            if harvestable.is_ready {
                let yield_amount = harvestable.harvest_yield;
                harvestable.is_ready = false;
                harvestable.harvest_yield = 0;
                return Some(yield_amount);
            }
        }
        None
    }
}

impl Watered for GardenPlant {
    fn water(&mut self) {
        self.water();
    }

    fn needs_water(&self) -> bool {
        self.needs_water()
    }
}
```


## Common Mistakes and Solutions

### Mistake 1: Over-nesting Structures

```rust
// ❌ Too deeply nested - hard to use
struct OverNested {
    info: PersonInfo {
        basic: BasicInfo {
            name: NameInfo {
                first: String,
                last: String,
            },
        },
        contact: ContactInfo {
            address: AddressInfo {
                street: StreetInfo {
                    number: u32,
                    name: String,
                },
            },
        },
    },
}

// ✅ Flatter structure - easier to work with
struct BetterStructured {
    name: String,
    contact_info: ContactInfo,
    personal_info: PersonalInfo,
}
```


### Mistake 2: Not Using Option for Optional Components

```rust
// ❌ Forces all fields to be present
struct IncompleteVegetable {
    name: String,
    nutrition: NutritionalInfo,    // What if we don't have this data?
    growing_info: GrowingInfo,     // What if we don't grow this?
}

// ✅ Optional components allow flexible usage
struct FlexibleVegetable {
    name: String,
    nutrition: Option<NutritionalInfo>,    // Optional
    growing_info: Option<GrowingInfo>,     // Optional
}
```


### Mistake 3: Ignoring Borrowing in Composed Structs

```rust
// ❌ This causes borrowing issues
impl Kitchen {
    fn get_ingredient_and_modify_menu(&mut self) -> &Ingredient {
        // This won't work - can't borrow ingredient while modifying menu
        self.menu_items.push(/* new item */);
        self.ingredients.get("tomato").unwrap()
    }
}

// ✅ Separate the operations
impl Kitchen {
    fn get_ingredient(&self, name: &str) -> Option<&Ingredient> {
        self.ingredients.get(name)
    }

    fn add_menu_item(&mut self, item: MenuItem) {
        self.menu_items.push(item);
    }
}
```


## Summary and Key Takeaways

### **Mental Model: The Vegetarian Salad Bowl**

Remember the salad analogy:

- 🥬 **Individual ingredients** = Focused structs (greens, vegetables, dressing)
- 🥗 **Complete salad** = Composed struct that combines ingredients
- 👨🍳 **Recipe** = Implementation that brings components together
- 🍽️ **Serving** = Using the composed struct in your program


### **Essential Principles**

1. **Break down complexity** - Use small, focused structs instead of large monolithic ones
2. **Compose, don't inherit** - Rust favors composition over inheritance
3. **Use Option for flexibility** - Not every component needs to be present
4. **Single responsibility** - Each struct should have one clear purpose
5. **Traits for behavior** - Use traits to share behavior across composed types

### **When to Use Composition**

| **Use Case** | **Example** | **Benefit** |
| :-- | :-- | :-- |
| **Complex entities** | Restaurant with menu, kitchen, staff | Modular organization |
| **Optional features** | Vegetable with optional growing info | Flexible data modeling |
| **Code reuse** | Address used in multiple structs | Avoid duplication |
| **Clear separation** | Recipe ingredients vs cooking instructions | Better maintainability |
| **Domain modeling** | Garden with plants, soil, tools | Matches real-world structure |

### **Composition Patterns Summary**

```rust
// Basic composition
struct Composed {
    component1: ComponentA,
    component2: ComponentB,
}

// Optional composition
struct Flexible {
    required: ComponentA,
    optional: Option<ComponentB>,
}

// Collection composition
struct Container {
    items: Vec<Component>,
    metadata: Metadata,
}

// Trait-based composition
impl SharedBehavior for Composed {
    fn common_method(&self) {
        // Use composed components
    }
}
```


### **Best Practices Checklist**

- ✅ **Keep structs focused** - Each struct should have a single, clear purpose
- ✅ **Use descriptive names** - Make the composition relationship obvious
- ✅ **Leverage Option** - Make optional components actually optional
- ✅ **Implement helpful methods** - Add convenience methods for common operations
- ✅ **Use traits for behavior** - Share behavior across different composed types
- ✅ **Document relationships** - Explain how components work together
- ✅ **Test components independently** - Each component should be testable on its own

**Struct composition is like being a master chef** who creates amazing vegetarian dishes by thoughtfully combining fresh, high-quality ingredients. Each ingredient (struct) brings its own unique properties and flavors (data and behavior), and when combined skillfully through composition, they create something far more delicious and valuable than any single ingredient could provide on its own!

Just as a great chef understands how different vegetables, herbs, and cooking techniques work together to create harmony on the plate, a great Rust programmer understands how to compose structs to create clean, maintainable, and powerful software architectures.

1. https://doc.rust-lang.org/book/ch05-00-structs.html
2. https://www.reddit.com/r/learnrust/comments/vppaw9/implementing_composition_structures_in_rust/
3. https://rust-unofficial.github.io/patterns/patterns/structural/compose-structs.html
4. https://doc.rust-lang.org/book/ch05-01-defining-structs.html
5. https://www.w3schools.com/rust/rust_structs.php
6. https://doc.rust-lang.org/book/ch05-02-example-structs.html
7. https://www.reddit.com/r/rust/comments/1fcad10/understanding_struct_organization_in_rust/
8. https://www.programiz.com/rust/struct
9. https://www.youtube.com/watch?v=mgK5LezkHl8
10. https://users.rust-lang.org/t/complex-composition-implementation/104367
11. https://stackoverflow.com/questions/74191402/composition-and-reusability-in-rust-compiled-code
12. https://stackoverflow.com/questions/32552593/is-it-possible-for-one-struct-to-extend-an-existing-struct-keeping-all-the-fiel
13. https://users.rust-lang.org/t/struct-field-behavior-composition-staying-dry-in-rust/54444
14. https://dev.to/antonov_mike/composition-in-rust-and-python-21c3
15. https://www.linkedin.com/pulse/struct-rust-amit-nadiger
16. https://users.rust-lang.org/t/with-composition-over-inheritance-in-rust-how-does-one-implement-shared-state-to-go-with-shared-behaviour/47357
17. https://docs.rs/structstruck/latest/structstruck/
18. https://blog.logrocket.com/fundamentals-for-using-structs-in-rust/
19. https://www.youtube.com/watch?v=PCjuO-Bv5FI
20. https://stackoverflow.com/questions/23629201/are-nested-structs-supported-in-rust
21. https://doc.rust-lang.org/rust-by-example/custom_types/structs.html
22. https://www.reddit.com/r/rust/comments/1grt9f0/define_nested_struct_in_rust/
23. https://internals.rust-lang.org/t/nested-struct-declaration/13314
