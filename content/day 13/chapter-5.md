+++
title = "Best Practices for Struct Design"
description = "Learn the best practices for designing structs in Rust, including naming conventions, encapsulation, modularity, and maintainability."
date = 2025-08-04
draft = false
weight = 5
+++

# Best Practices for Struct Design in Rust: Comprehensive Documentation for Beginners

Understanding how to design structs well is like learning to **organize a professional vegetarian kitchen** for maximum efficiency and deliciousness. Just as a master chef arranges ingredients, tools, and workspace to create amazing plant-based dishes seamlessly, a skilled Rust programmer designs structs to make code clean, maintainable, and powerful.

## The Professional Vegetarian Kitchen Analogy 🍃

### Imagine You're Designing the Ultimate Veggie Kitchen

**Poor Kitchen Design (Bad Struct):**

```rust
// ❌ Everything jumbled together - hard to find anything!
struct MessyKitchen {
    onion_location: String,
    knife_sharpness: u8,
    tomato_ripeness: f32,
    stove_temperature: u32,
    recipe_difficulty: u8,
    carrot_color: String,
    pan_material: String,
    cooking_time: u32,
    // ... 50+ more random fields mixed together!
}
```

**Professional Kitchen Design (Good Struct):**

```rust
// ✅ Everything organized in logical sections
struct Ingredients {
    vegetables: Vec<Vegetable>,
    seasonings: Vec<Seasoning>,
    oils: Vec<CookingOil>,
}

struct Equipment {
    knives: Vec<Knife>,
    cookware: Vec<Cookware>,
    appliances: Vec<Appliance>,
}

struct Recipe {
    name: String,
    steps: Vec<CookingStep>,
    estimated_time: Duration,
    difficulty: SkillLevel,
}

struct VegeterianKitchen {
    ingredients: Ingredients,    // 🥬 Ingredient station
    equipment: Equipment,        // 🔪 Tool station
    active_recipes: Vec<Recipe>, // 📖 Recipe station
    chef_notes: String,         // 📝 Personal notes
}
```

**Why the Professional Design Works:**

- 🗂️ **Logical organization** - Each section has a clear purpose
- 🔍 **Easy to find things** - Know exactly where to look
- 🧹 **Easy to maintain** - Update one section without affecting others
- 📈 **Scalable** - Add new equipment without reorganizing everything
- 👥 **Team-friendly** - Other chefs can understand the layout


## Core Design Principles

### 1. Use Clear, Descriptive Names (The Recipe Card Principle)

**Good struct names should read like recipe titles - clear and appetizing!**

```rust
// ❌ Vague names - what do these actually represent?
struct Thing {
    stuff: String,
    num: u32,
    flag: bool,
}

struct Data {
    x: f32,
    y: f32,
    z: bool,
}

// ✅ Clear, descriptive names - immediately understandable
struct VegetableSoup {
    name: String,
    serving_count: u32,
    is_vegan_friendly: bool,
}

struct GardenCoordinate {
    latitude: f64,
    longitude: f64,
    is_suitable_for_tomatoes: bool,
}

// ✅ Domain-specific vocabulary makes intent clear
struct PlantNutrition {
    calories_per_100g: u32,
    protein_content_grams: f32,
    fiber_content_grams: f32,
    vitamin_c_mg: f32,
}
```

**Naming Guidelines:**

- Use **full words** instead of abbreviations (`temperature` not `temp`)
- Include **units** when relevant (`cooking_time_minutes`, `weight_grams`)
- Be **specific** about what the field represents (`is_organic` not just `organic`)
- Use **domain language** that makes sense to anyone reading your code


### 2. Single Responsibility Principle (One Job Per Kitchen Station)

**Each struct should be like a specialized kitchen station - focused on doing one thing extremely well.**

```rust
// ❌ The "everything station" - trying to do too much
struct OverloadedVegetable {
    // Botanical info
    name: String,
    scientific_name: String,
    family: String,

    // Growing info
    planting_season: String,
    harvest_time: u32,
    soil_ph: f32,

    // Nutritional info
    calories: u32,
    vitamins: Vec<String>,

    // Culinary info
    cooking_methods: Vec<String>,
    flavor_profile: String,

    // Storage info
    storage_temperature: f32,
    shelf_life_days: u32,

    // Market info
    price_per_pound: f64,
    availability: String,

    // Personal notes
    my_rating: u8,
    last_used_date: String,
    // ... this keeps growing!
}

// ✅ Focused structs - each has one clear responsibility
#[derive(Debug, Clone)]
struct BotanicalInfo {
    common_name: String,
    scientific_name: String,
    plant_family: String,
}

#[derive(Debug, Clone)]
struct GrowingRequirements {
    planting_season: Season,
    days_to_harvest: u32,
    ideal_soil_ph_range: (f32, f32),
    sunlight_hours_needed: u32,
}

#[derive(Debug, Clone)]
struct NutritionalProfile {
    calories_per_100g: u32,
    protein_grams: f32,
    fiber_grams: f32,
    vitamins: Vec<Vitamin>,
    minerals: Vec<Mineral>,
}

#[derive(Debug, Clone)]
struct CulinaryUses {
    best_cooking_methods: Vec<CookingMethod>,
    flavor_notes: Vec<String>,
    pairs_well_with: Vec<String>,
    preparation_tips: Vec<String>,
}

// The main struct composes focused components
#[derive(Debug)]
struct Vegetable {
    botanical: BotanicalInfo,
    growing: Option<GrowingRequirements>,  // Optional - might not grow it
    nutrition: NutritionalProfile,
    culinary: CulinaryUses,
}

#[derive(Debug, Clone)]
enum Season { Spring, Summer, Fall, Winter }

#[derive(Debug, Clone)]
enum CookingMethod { Raw, Steamed, Roasted, Grilled, Sauteed }

#[derive(Debug, Clone)]
struct Vitamin { name: String, amount_mg: f32 }

#[derive(Debug, Clone)]
struct Mineral { name: String, amount_mg: f32 }
```

**Benefits of Single Responsibility:**

- 🎯 **Clear purpose** - Each struct has one job
- 🔧 **Easy to modify** - Change nutrition info without affecting growing info
- 🧪 **Easy to test** - Test each component independently
- ♻️ **Reusable** - Use `NutritionalProfile` for fruits, nuts, grains, etc.
- 📖 **Easy to understand** - New team members can quickly grasp each piece


### 3. Composition Over Inheritance (The Modular Kitchen)

**Build complexity by combining simple, well-designed components - like assembling a modular kitchen.**

```rust
// Rust doesn't have inheritance, so we use composition
// Think of this like modular kitchen appliances that can be combined

#[derive(Debug)]
struct BaseAppliance {
    brand: String,
    model: String,
    power_watts: u32,
    warranty_years: u32,
}

#[derive(Debug)]
struct BlenderFeatures {
    max_speed_rpm: u32,
    jar_capacity_ml: u32,
    has_pulse_mode: bool,
    blade_count: u32,
}

#[derive(Debug)]
struct FoodProcessorFeatures {
    bowl_capacity_cups: u32,
    attachments: Vec<String>,
    speed_settings: u32,
}

#[derive(Debug)]
struct JuicerFeatures {
    juice_extraction_method: JuiceMethod,
    pulp_container_size: u32,
    produces_smooth_juice: bool,
}

// Compose different appliances from base + specific features
#[derive(Debug)]
struct VitamixBlender {
    base: BaseAppliance,
    blender_specs: BlenderFeatures,
}

#[derive(Debug)]
struct CuisinartFoodProcessor {
    base: BaseAppliance,
    processor_specs: FoodProcessorFeatures,
}

#[derive(Debug)]
struct BrevilleJuicer {
    base: BaseAppliance,
    juicer_specs: JuicerFeatures,
}

#[derive(Debug)]
enum JuiceMethod { Centrifugal, Masticating, Citrus }

impl VitamixBlender {
    fn new(model: &str) -> Self {
        VitamixBlender {
            base: BaseAppliance {
                brand: "Vitamix".to_string(),
                model: model.to_string(),
                power_watts: 1200,
                warranty_years: 7,
            },
            blender_specs: BlenderFeatures {
                max_speed_rpm: 22000,
                jar_capacity_ml: 2000,
                has_pulse_mode: true,
                blade_count: 4,
            },
        }
    }

    fn can_make_smoothie(&self) -> bool {
        self.blender_specs.jar_capacity_ml >= 500 && self.base.power_watts >= 1000
    }

    fn get_warranty_info(&self) -> String {
        format!("{} {} - {} year warranty",
                self.base.brand, self.base.model, self.base.warranty_years)
    }
}

fn main() {
    let my_blender = VitamixBlender::new("A3500");

    println!("Appliance: {:?}", my_blender);
    println!("Can make smoothies: {}", my_blender.can_make_smoothie());
    println!("Warranty: {}", my_blender.get_warranty_info());
}
```


### 4. Smart Visibility Control (The Professional Kitchen Access)

**Control who can access what, like a professional kitchen with different access levels.**

```rust
// Public module - the "front of house" that customers see
pub mod restaurant {
    // ✅ Customer-facing information is public
    #[derive(Debug)]
    pub struct MenuItem {
        pub name: String,
        pub description: String,
        pub price: f64,
        pub allergen_info: Vec<String>,

        // Private internal details customers don't need
        cost_to_make: f64,        // Keep profit margins private
        prep_complexity: u8,      // Internal kitchen info
        supplier_info: String,    // Business relationships
    }

    impl MenuItem {
        // ✅ Public constructor with validation
        pub fn new(name: String, description: String, price: f64) -> Result<Self, String> {
            if price <= 0.0 {
                return Err("Price must be positive".to_string());
            }

            if name.trim().is_empty() {
                return Err("Menu item must have a name".to_string());
            }

            Ok(MenuItem {
                name,
                description,
                price,
                allergen_info: Vec::new(),
                cost_to_make: 0.0,        // Will be calculated internally
                prep_complexity: 1,       // Default to simple
                supplier_info: "TBD".to_string(),
            })
        }

        // ✅ Public method to add allergen info
        pub fn add_allergen(&mut self, allergen: String) {
            if !self.allergen_info.contains(&allergen) {
                self.allergen_info.push(allergen);
            }
        }

        // ✅ Public getter for profit margin (calculated, not raw data)
        pub fn is_profitable(&self) -> bool {
            self.price > self.cost_to_make * 1.5  // Need 50% markup
        }

        // ❌ Private method - only for internal kitchen use
        fn set_cost_breakdown(&mut self, ingredient_cost: f64, labor_cost: f64) {
            self.cost_to_make = ingredient_cost + labor_cost;
        }

        // ❌ Private method - kitchen operations
        fn update_complexity_rating(&mut self, complexity: u8) {
            self.prep_complexity = complexity.min(10);  // Cap at 10
        }
    }
}

// Usage example
fn main() -> Result<(), String> {
    use restaurant::MenuItem;

    let mut quinoa_bowl = MenuItem::new(
        "Rainbow Quinoa Bowl".to_string(),
        "Colorful quinoa with roasted vegetables and tahini dressing".to_string(),
        14.95
    )?;

    // ✅ Customers can see public info
    println!("Menu item: {}", quinoa_bowl.name);
    println!("Price: ${:.2}", quinoa_bowl.price);
    println!("Description: {}", quinoa_bowl.description);

    // ✅ Can add allergen information
    quinoa_bowl.add_allergen("Sesame (tahini)".to_string());
    println!("Allergens: {:?}", quinoa_bowl.allergen_info);

    // ✅ Can check if profitable (but not see raw costs)
    println!("Is profitable: {}", quinoa_bowl.is_profitable());

    // ❌ Cannot access private fields directly
    // println!("Cost: {}", quinoa_bowl.cost_to_make);  // Compiler error!

    Ok(())
}
```

**Visibility Guidelines:**

- 🏢 **Public (`pub`)**: Customer-facing information, stable APIs
- 🔒 **Private (default)**: Internal implementation details, business logic
- 📦 **Module-level**: Organize related functionality
- 🛡️ **Getters/Setters**: Control how data is accessed and modified


### 5. Constructor Patterns (The Prep Station Setup)

**Provide convenient ways to create instances, like having different prep station setups for different types of cooking.**

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct VegetarianRecipe {
    name: String,
    ingredients: Vec<Ingredient>,
    instructions: Vec<String>,
    prep_time_minutes: u32,
    cook_time_minutes: u32,
    servings: u32,
    difficulty: DifficultyLevel,
    dietary_tags: Vec<DietaryTag>,
}

#[derive(Debug, Clone)]
struct Ingredient {
    name: String,
    amount: String,
    notes: Option<String>,
}

#[derive(Debug, Clone)]
enum DifficultyLevel { Beginner, Intermediate, Advanced, ChefLevel }

#[derive(Debug, Clone)]
enum DietaryTag { Vegan, GlutenFree, NutFree, SoyFree, RawFood, LowCarb }

impl VegetarianRecipe {
    // ✅ Basic constructor - most common case
    pub fn new(name: String, servings: u32) -> Self {
        VegetarianRecipe {
            name,
            servings,
            ingredients: Vec::new(),
            instructions: Vec::new(),
            prep_time_minutes: 0,
            cook_time_minutes: 0,
            difficulty: DifficultyLevel::Beginner,
            dietary_tags: Vec::new(),
        }
    }

    // ✅ Constructor with validation
    pub fn new_validated(name: String, servings: u32) -> Result<Self, String> {
        if name.trim().is_empty() {
            return Err("Recipe name cannot be empty".to_string());
        }

        if servings == 0 || servings > 20 {
            return Err("Servings must be between 1 and 20".to_string());
        }

        Ok(Self::new(name, servings))
    }

    // ✅ Quick setup constructors for common patterns
    pub fn quick_salad(name: String) -> Self {
        let mut recipe = Self::new(name, 2);
        recipe.prep_time_minutes = 15;
        recipe.cook_time_minutes = 0;
        recipe.difficulty = DifficultyLevel::Beginner;
        recipe.dietary_tags = vec![DietaryTag::Vegan, DietaryTag::GlutenFree];
        recipe
    }

    pub fn hearty_soup(name: String) -> Self {
        let mut recipe = Self::new(name, 4);
        recipe.prep_time_minutes = 20;
        recipe.cook_time_minutes = 45;
        recipe.difficulty = DifficultyLevel::Intermediate;
        recipe.dietary_tags = vec![DietaryTag::Vegan];
        recipe
    }

    pub fn advanced_entree(name: String) -> Self {
        let mut recipe = Self::new(name, 4);
        recipe.prep_time_minutes = 45;
        recipe.cook_time_minutes = 60;
        recipe.difficulty = DifficultyLevel::Advanced;
        recipe
    }

    // ✅ Factory method from external data
    pub fn from_recipe_card(card_data: &str) -> Result<Self, String> {
        // Parse recipe card format (simplified example)
        let lines: Vec<&str> = card_data.lines().collect();
        if lines.len() < 2 {
            return Err("Invalid recipe card format".to_string());
        }

        let name = lines[^0].trim().to_string();
        let servings = lines[^1]
            .split_whitespace()
            .find(|word| word.parse::<u32>().is_ok())
            .and_then(|s| s.parse().ok())
            .ok_or("Could not find servings count")?;

        Self::new_validated(name, servings)
    }

    // Builder-style methods for fluent construction
    pub fn add_ingredient(mut self, name: &str, amount: &str) -> Self {
        self.ingredients.push(Ingredient {
            name: name.to_string(),
            amount: amount.to_string(),
            notes: None,
        });
        self
    }

    pub fn add_instruction(mut self, instruction: &str) -> Self {
        self.instructions.push(instruction.to_string());
        self
    }

    pub fn set_difficulty(mut self, level: DifficultyLevel) -> Self {
        self.difficulty = level;
        self
    }

    pub fn add_dietary_tag(mut self, tag: DietaryTag) -> Self {
        if !self.dietary_tags.contains(&tag) {
            self.dietary_tags.push(tag);
        }
        self
    }
}

fn main() -> Result<(), String> {
    // ✅ Simple constructor
    let basic_recipe = VegetarianRecipe::new("Simple Salad".to_string(), 2);

    // ✅ Quick setup constructor
    let soup = VegetarianRecipe::hearty_soup("Lentil Vegetable Soup".to_string());

    // ✅ Builder-style construction
    let pasta_recipe = VegetarianRecipe::new("Veggie Pasta".to_string(), 4)
        .add_ingredient("Pasta", "1 lb")
        .add_ingredient("Marinara sauce", "2 cups")
        .add_ingredient("Zucchini", "2 medium, diced")
        .add_instruction("Cook pasta according to package directions")
        .add_instruction("Sauté zucchini until tender")
        .add_instruction("Combine pasta with sauce and vegetables")
        .set_difficulty(DifficultyLevel::Beginner)
        .add_dietary_tag(DietaryTag::Vegan);

    // ✅ Parse from external data
    let recipe_data = "Mediterranean Bowl\nServes 3\nIngredients: quinoa, olives, tomatoes";
    let parsed_recipe = VegetarianRecipe::from_recipe_card(recipe_data)?;

    println!("Basic: {:?}", basic_recipe.name);
    println!("Soup: {:?}", soup.name);
    println!("Pasta: {:?}", pasta_recipe.name);
    println!("Parsed: {:?}", parsed_recipe.name);

    Ok(())
}
```


### 6. Derive Common Traits (The Standard Kitchen Equipment)

**Just like every professional kitchen needs standard equipment, every struct should have standard capabilities.**

```rust
// ✅ Most structs should derive these essential traits
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct VegetableVariety {
    name: String,
    color: String,
    shape: String,
    season: Season,
}

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum Season {
    Spring,
    Summer,
    Fall,
    Winter,
}

// ✅ For structs with floating point fields, use PartialEq instead of Eq
#[derive(Debug, Clone, PartialEq)]
struct NutritionalData {
    calories_per_100g: f32,
    protein_grams: f32,
    fat_grams: f32,
    carbs_grams: f32,
}

// ✅ For ordering/sorting capabilities
#[derive(Debug, Clone, PartialEq, Eq, PartialOrd, Ord)]
enum SpiceIntensity {
    Mild,      // Lowest
    Medium,
    Hot,
    ExtraHot,  // Highest
}

// ✅ For default values
#[derive(Debug, Clone, Default)]
struct CookingSettings {
    temperature_fahrenheit: u32,   // Default: 0
    timer_minutes: u32,           // Default: 0
    use_convection: bool,         // Default: false
}

// ✅ Custom Default implementation when automatic doesn't work
impl Default for NutritionalData {
    fn default() -> Self {
        NutritionalData {
            calories_per_100g: 0.0,
            protein_grams: 0.0,
            fat_grams: 0.0,
            carbs_grams: 0.0,
        }
    }
}

use std::collections::HashMap;
use std::collections::HashSet;

fn main() {
    // Debug - easy printing for development
    let tomato = VegetableVariety {
        name: "Cherry Tomato".to_string(),
        color: "Red".to_string(),
        shape: "Round".to_string(),
        season: Season::Summer,
    };
    println!("Tomato variety: {:?}", tomato);

    // Clone - create independent copies
    let heirloom_tomato = VegetableVariety {
        name: "Heirloom Tomato".to_string(),
        ..tomato.clone()  // Clone and modify
    };

    // PartialEq - compare for equality
    let another_cherry = tomato.clone();
    println!("Same variety? {}", tomato == another_cherry);
    println!("Different variety? {}", tomato == heirloom_tomato);

    // Hash - use in HashSet/HashMap
    let mut vegetable_inventory: HashSet<VegetableVariety> = HashSet::new();
    vegetable_inventory.insert(tomato.clone());
    vegetable_inventory.insert(heirloom_tomato.clone());

    let mut nutrition_database: HashMap<String, NutritionalData> = HashMap::new();
    nutrition_database.insert("Spinach".to_string(), NutritionalData {
        calories_per_100g: 23.0,
        protein_grams: 2.9,
        fat_grams: 0.4,
        carbs_grams: 3.6,
    });

    // Ord - sorting
    let mut spice_levels = vec![
        SpiceIntensity::Hot,
        SpiceIntensity::Mild,
        SpiceIntensity::ExtraHot,
        SpiceIntensity::Medium,
    ];
    spice_levels.sort();
    println!("Spice levels (sorted): {:?}", spice_levels);

    // Default - easy initialization
    let default_settings = CookingSettings::default();
    let custom_settings = CookingSettings {
        temperature_fahrenheit: 350,
        ..CookingSettings::default()  // Use defaults for other fields
    };

    println!("Default settings: {:?}", default_settings);
    println!("Custom settings: {:?}", custom_settings);
}
```

**Common Trait Guidelines:**


| Trait | When to Use | What It Enables |
| :-- | :-- | :-- |
| **`Debug`** | Almost always | Easy printing with `{:?}` for development |
| **`Clone`** | When you need to duplicate | Create independent copies |
| **`PartialEq`** | For equality comparisons | Use `==` and `!=` operators |
| **`Eq`** | When all values can be compared | Use in `HashMap` and `HashSet` |
| **`Hash`** | For use in hash-based collections | Keys in `HashMap`, items in `HashSet` |
| **`PartialOrd`** | For some ordering | Use `<`, `>`, etc. (works with floats) |
| **`Ord`** | For total ordering | Full sorting capability |
| **`Default`** | For sensible default values | Easy initialization and builder patterns |

### 7. Memory Layout Optimization (The Efficient Kitchen Layout)

**Arrange fields thoughtfully to minimize wasted space, like optimizing counter space in a compact kitchen.**

```rust
use std::mem;

// ❌ Poor memory layout - lots of wasted space
#[derive(Debug)]
struct IneffientVeggie {
    is_organic: bool,          // 1 byte
    // 7 bytes padding here!
    price_per_pound: f64,      // 8 bytes
    quantity_available: u16,   // 2 bytes
    // 6 bytes padding here!
    name: String,              // 24 bytes (3 pointers)
    is_local: bool,           // 1 byte
    // 7 bytes padding here!
}

// ✅ Optimized memory layout - minimal padding
#[derive(Debug)]
struct EfficientVeggie {
    // Group larger fields first
    name: String,              // 24 bytes (3 pointers)
    price_per_pound: f64,      // 8 bytes

    // Group medium fields
    quantity_available: u16,   // 2 bytes

    // Group smaller fields together at the end
    is_organic: bool,          // 1 byte
    is_local: bool,           // 1 byte
    // Only 4 bytes padding after the bools
}

fn demonstrate_memory_layout() {
    println!("Inefficient struct size: {} bytes", mem::size_of::<IneffientVeggie>());
    println!("Efficient struct size: {} bytes", mem::size_of::<EfficientVeggie>());

    // You'll see the efficient version uses significantly less memory!
}

// ✅ Even better - use #[repr(C)] for predictable layout when needed
#[repr(C)]
#[derive(Debug)]
struct PredictableVeggie {
    name: String,
    price: f64,
    quantity: u16,
    is_organic: bool,
    is_local: bool,
}

// ✅ For performance-critical structs, consider alignment
#[repr(align(64))]  // Align to cache line
#[derive(Debug)]
struct CacheOptimizedVeggie {
    // Hot data - accessed frequently
    name: String,
    price: f64,
    quantity: u16,

    // Less frequently accessed data
    supplier: String,
    last_restocked: String,
}

fn main() {
    demonstrate_memory_layout();

    let efficient = EfficientVeggie {
        name: "Organic Carrots".to_string(),
        price_per_pound: 2.99,
        quantity_available: 50,
        is_organic: true,
        is_local: false,
    };

    println!("Efficient veggie: {:?}", efficient);
}
```

**Memory Layout Guidelines:**

- 📏 **Order by size**: Put larger fields first, smaller ones last
- 📦 **Group related data**: Keep frequently accessed fields together
- ⚡ **Consider cache alignment**: Use `#[repr(align(N))]` for hot paths
- 🎯 **Use `#[repr(C)]`**: When you need predictable layout (FFI, serialization)
- 📊 **Check sizes**: Use `std::mem::size_of()` to verify optimizations


### 8. Documentation and Examples (The Recipe Documentation)

**Document your structs like writing clear, helpful recipes that anyone can follow.**

```rust
use std::collections::HashMap;

/// Represents a complete vegetarian recipe with all necessary information
/// for preparation, cooking, and nutritional analysis.
///
/// # Examples
///
/// ```
/// use crate::VegetarianRecipe;
/// use crate::DietaryTag;
///
/// let recipe = VegetarianRecipe::new("Simple Garden Salad", 4)
///     .add_ingredient("Mixed greens", "6 cups")
///     .add_ingredient("Cherry tomatoes", "1 cup")
///     .set_prep_time(10)
///     .add_dietary_tag(DietaryTag::Vegan);
///
/// assert_eq!(recipe.get_servings(), 4);
/// assert!(recipe.is_quick_prep());
/// ```
///
/// # Common Usage Patterns
///
/// - Use `VegetarianRecipe::quick_salad()` for simple raw dishes
/// - Use `VegetarianRecipe::hearty_soup()` for longer-cooking dishes
/// - Chain methods for fluent recipe building
#[derive(Debug, Clone)]
pub struct VegetarianRecipe {
    /// The display name of the recipe (e.g., "Mediterranean Quinoa Bowl")
    name: String,

    /// List of ingredients with measurements
    ///
    /// Each ingredient should include specific amounts and preparation notes
    /// Example: "2 cups quinoa, rinsed" or "1 large onion, diced"
    ingredients: Vec<RecipeIngredient>,

    /// Step-by-step cooking instructions
    ///
    /// Instructions should be clear, specific, and in chronological order.
    /// Assume the reader is a beginner cook.
    instructions: Vec<String>,

    /// Active preparation time in minutes (not including passive cooking time)
    prep_time_minutes: u32,

    /// Active cooking/baking time in minutes
    cook_time_minutes: u32,

    /// Number of servings this recipe produces
    ///
    /// Should be realistic serving sizes for the intended audience
    servings: u32,

    /// Skill level required to successfully make this recipe
    difficulty: SkillLevel,

    /// Dietary restrictions this recipe accommodates
    ///
    /// Only include tags that are definitively true - when in doubt, omit
    dietary_tags: Vec<DietaryTag>,

    /// Optional notes from the chef or recipe developer
    ///
    /// Use for variations, storage tips, or personal recommendations
    chef_notes: Option<String>,
}

/// A single ingredient with amount and optional preparation notes
#[derive(Debug, Clone)]
pub struct RecipeIngredient {
    /// Ingredient name (e.g., "quinoa", "red bell pepper")
    pub name: String,

    /// Amount needed with units (e.g., "2 cups", "1 large", "3 tablespoons")
    pub amount: String,

    /// Optional preparation instructions (e.g., "diced", "rinsed", "at room temperature")
    pub prep_notes: Option<String>,

    /// Whether this ingredient is optional for the recipe
    pub is_optional: bool,
}

/// Skill level required for a recipe
#[derive(Debug, Clone, PartialEq)]
pub enum SkillLevel {
    /// No cooking required, simple assembly (salads, smoothies)
    Beginner,

    /// Basic cooking techniques, timing not critical (soups, stir-fries)
    Intermediate,

    /// Advanced techniques or precise timing required (bread, complex sauces)
    Advanced,

    /// Professional-level techniques (sous vide, complex plating)
    Professional,
}

/// Dietary accommodations this recipe supports
#[derive(Debug, Clone, PartialEq)]
pub enum DietaryTag {
    /// Contains no animal products whatsoever
    Vegan,

    /// Contains no gluten-containing ingredients
    GlutenFree,

    /// Contains no tree nuts or peanuts
    NutFree,

    /// Contains no soy products
    SoyFree,

    /// No cooking required - all raw ingredients
    Raw,

    /// Lower in carbohydrates (subjective, but generally < 30g per serving)
    LowCarb,

    /// Higher in protein content (subjective, but generally > 15g per serving)
    HighProtein,

    /// Can be prepared in 30 minutes or less total time
    QuickMeal,
}

impl VegetarianRecipe {
    /// Creates a new recipe with the given name and serving count.
    ///
    /// # Arguments
    ///
    /// * `name` - A descriptive name for the recipe
    /// * `servings` - Number of servings (must be between 1 and 20)
    ///
    /// # Returns
    ///
    /// Returns a new `VegetarianRecipe` with default values for other fields.
    /// Use builder methods to customize further.
    ///
    /// # Examples
    ///
    /// ```
    /// let recipe = VegetarianRecipe::new("Veggie Stir Fry", 4);
    /// assert_eq!(recipe.get_name(), "Veggie Stir Fry");
    /// assert_eq!(recipe.get_servings(), 4);
    /// ```
    pub fn new(name: &str, servings: u32) -> Self {
        VegetarianRecipe {
            name: name.to_string(),
            servings,
            ingredients: Vec::new(),
            instructions: Vec::new(),
            prep_time_minutes: 0,
            cook_time_minutes: 0,
            difficulty: SkillLevel::Beginner,
            dietary_tags: Vec::new(),
            chef_notes: None,
        }
    }

    /// Returns the recipe name
    pub fn get_name(&self) -> &str {
        &self.name
    }

    /// Returns the number of servings
    pub fn get_servings(&self) -> u32 {
        self.servings
    }

    /// Returns total time (prep + cook) in minutes
    pub fn get_total_time(&self) -> u32 {
        self.prep_time_minutes + self.cook_time_minutes
    }

    /// Checks if this is considered a quick recipe (≤ 30 minutes total)
    ///
    /// # Examples
    ///
    /// ```
    /// let quick_salad = VegetarianRecipe::new("Garden Salad", 2)
    ///     .set_prep_time(15);
    /// assert!(quick_salad.is_quick_prep());
    ///
    /// let slow_soup = VegetarianRecipe::new("Slow-Cooked Lentil Soup", 6)
    ///     .set_prep_time(20)
    ///     .set_cook_time(90);
    /// assert!(!slow_soup.is_quick_prep());
    /// ```
    pub fn is_quick_prep(&self) -> bool {
        self.get_total_time() <= 30
    }

    /// Adds an ingredient to the recipe
    ///
    /// # Arguments
    ///
    /// * `name` - The ingredient name
    /// * `amount` - Amount with units (e.g., "2 cups", "1 tablespoon")
    ///
    /// # Examples
    ///
    /// ```
    /// let recipe = VegetarianRecipe::new("Pasta", 4)
    ///     .add_ingredient("pasta", "1 pound")
    ///     .add_ingredient("olive oil", "2 tablespoons");
    /// ```
    pub fn add_ingredient(mut self, name: &str, amount: &str) -> Self {
        self.ingredients.push(RecipeIngredient {
            name: name.to_string(),
            amount: amount.to_string(),
            prep_notes: None,
            is_optional: false,
        });
        self
    }

    /// Sets the preparation time in minutes
    pub fn set_prep_time(mut self, minutes: u32) -> Self {
        self.prep_time_minutes = minutes;
        self
    }

    /// Sets the cooking time in minutes
    pub fn set_cook_time(mut self, minutes: u32) -> Self {
        self.cook_time_minutes = minutes;
        self
    }

    /// Adds a dietary tag if it doesn't already exist
    pub fn add_dietary_tag(mut self, tag: DietaryTag) -> Self {
        if !self.dietary_tags.contains(&tag) {
            self.dietary_tags.push(tag);
        }
        self
    }
}

fn main() {
    let recipe = VegetarianRecipe::new("Mediterranean Bowl", 3)
        .add_ingredient("quinoa", "1 cup")
        .add_ingredient("cucumber", "1 large, diced")
        .add_ingredient("cherry tomatoes", "2 cups")
        .set_prep_time(20)
        .add_dietary_tag(DietaryTag::Vegan)
        .add_dietary_tag(DietaryTag::GlutenFree);

    println!("Recipe: {}", recipe.get_name());
    println!("Serves: {}", recipe.get_servings());
    println!("Total time: {} minutes", recipe.get_total_time());
    println!("Quick meal: {}", recipe.is_quick_prep());
}
```

**Documentation Guidelines:**

- 📖 **Struct-level docs**: Explain what it represents and common usage
- 📝 **Field-level docs**: Clarify non-obvious fields and their constraints
- 💡 **Include examples**: Show how to create and use the struct
- ⚠️ **Document invariants**: Explain any rules or assumptions
- 🔗 **Link related items**: Reference related structs, enums, functions
- 📚 **Provide context**: Explain when to use this struct vs alternatives


## Anti-Patterns to Avoid

### ❌ The "God Struct" Anti-Pattern

```rust
// ❌ Don't create monolithic structs that do everything
struct EverythingRestaurant {
    // Menu items
    appetizers: Vec<String>,
    entrees: Vec<String>,
    desserts: Vec<String>,
    drinks: Vec<String>,

    // Staff
    chefs: Vec<String>,
    servers: Vec<String>,
    managers: Vec<String>,

    // Inventory
    ingredients: HashMap<String, u32>,
    equipment: Vec<String>,

    // Finances
    daily_revenue: f64,
    expenses: Vec<f64>,

    // Customer data
    reservations: Vec<String>,
    reviews: Vec<String>,

    // Building info
    address: String,
    square_footage: u32,

    // ... this keeps growing forever!
}

// ✅ Break into focused, composable structs instead
struct Menu {
    appetizers: Vec<MenuItem>,
    entrees: Vec<MenuItem>,
    desserts: Vec<MenuItem>,
    beverages: Vec<MenuItem>,
}

struct Staff {
    kitchen_staff: Vec<Employee>,
    service_staff: Vec<Employee>,
    management: Vec<Employee>,
}

struct Inventory {
    ingredients: HashMap<String, IngredientStock>,
    equipment: Vec<KitchenEquipment>,
}

struct Restaurant {
    name: String,
    menu: Menu,           // Composed
    staff: Staff,         // Composed
    inventory: Inventory, // Composed
    location: Address,    // Composed
}
```


### ❌ The "Stringly Typed" Anti-Pattern

```rust
// ❌ Don't use strings for everything - use proper types
struct BadRecipe {
    name: String,
    difficulty: String,      // Should be an enum!
    dietary_tags: String,    // Should be Vec<DietaryTag>!
    cook_time: String,       // Should be numeric!
    temperature: String,     // Should be numeric!
}

// ✅ Use appropriate types
struct GoodRecipe {
    name: String,
    difficulty: DifficultyLevel,    // Type-safe enum
    dietary_tags: Vec<DietaryTag>,  // Structured data
    cook_time_minutes: u32,         // Numeric with clear units
    temperature_fahrenheit: u32,    // Numeric with clear units
}
```


### ❌ The "Public Everything" Anti-Pattern

```rust
// ❌ Don't make everything public without thinking
pub struct BadAccount {
    pub name: String,
    pub balance: f64,            // Should be controlled!
    pub pin: String,             // Should be private!
    pub internal_id: u64,       // Implementation detail!
    pub temp_calculations: Vec<f64>, // Internal state!
}

// ✅ Thoughtful visibility control
pub struct GoodAccount {
    pub name: String,            // Safe to expose
    balance: f64,               // Controlled through methods
    pin_hash: String,           // Private and hashed
    internal_id: u64,          // Private implementation detail
    temp_calculations: Vec<f64>, // Private internal state
}

impl GoodAccount {
    pub fn get_balance(&self) -> f64 {
        self.balance
    }

    pub fn verify_pin(&self, pin: &str) -> bool {
        // Proper pin verification logic
        hash_pin(pin) == self.pin_hash
    }
}
```


## Real-World Example: Complete Vegetarian Restaurant System

Let's put all these principles together in a comprehensive example:

```rust
use std::collections::HashMap;
use std::fmt;

/// Represents a vegetarian restaurant with all its components
#[derive(Debug)]
pub struct VegetarianRestaurant {
    pub name: String,
    pub location: Address,
    menu: Menu,
    kitchen: Kitchen,
    staff: RestaurantStaff,
    business_info: BusinessInfo,
}

/// Physical address information
#[derive(Debug, Clone)]
pub struct Address {
    pub street: String,
    pub city: String,
    pub state: String,
    pub zip_code: String,
}

/// Complete menu with different course categories
#[derive(Debug)]
struct Menu {
    appetizers: Vec<MenuItem>,
    salads: Vec<MenuItem>,
    entrees: Vec<MenuItem>,
    desserts: Vec<MenuItem>,
    beverages: Vec<MenuItem>,
    daily_specials: Vec<MenuItem>,
}

/// Individual menu item with complete information
#[derive(Debug, Clone)]
pub struct MenuItem {
    pub name: String,
    pub description: String,
    pub price: f64,
    pub dietary_tags: Vec<DietaryTag>,
    recipe: Recipe,
    nutritional_info: NutritionalInfo,
}

/// Complete recipe with ingredients and instructions
#[derive(Debug, Clone)]
struct Recipe {
    ingredients: Vec<RecipeIngredient>,
    instructions: Vec<CookingInstruction>,
    prep_time_minutes: u32,
    cook_time_minutes: u32,
    skill_level: SkillLevel,
    equipment_needed: Vec<String>,
}

#[derive(Debug, Clone)]
struct RecipeIngredient {
    name: String,
    amount: String,
    preparation: Option<String>,
    is_optional: bool,
}

#[derive(Debug, Clone)]
struct CookingInstruction {
    step_number: u32,
    instruction: String,
    estimated_time_minutes: Option<u32>,
    temperature: Option<u32>,
}

/// Nutritional information per serving
#[derive(Debug, Clone)]
pub struct NutritionalInfo {
    pub calories: u32,
    pub protein_grams: f32,
    pub carbs_grams: f32,
    pub fat_grams: f32,
    pub fiber_grams: f32,
    pub sodium_mg: u32,
    pub vitamins: Vec<VitaminContent>,
}

#[derive(Debug, Clone)]
pub struct VitaminContent {
    pub vitamin: String,
    pub amount: String,
    pub daily_value_percentage: Option<f32>,
}

/// Kitchen operations and inventory
#[derive(Debug)]
struct Kitchen {
    inventory: IngredientInventory,
    equipment: Vec<KitchenEquipment>,
    active_orders: Vec<Order>,
    prep_stations: Vec<PrepStation>,
}

#[derive(Debug)]
struct IngredientInventory {
    ingredients: HashMap<String, IngredientStock>,
    last_updated: String, // In a real app, use proper date types
}

#[derive(Debug, Clone)]
struct IngredientStock {
    ingredient: Ingredient,
    quantity_available: f32,
    unit: String,
    expiration_date: Option<String>,
    supplier: String,
    cost_per_unit: f64,
}

#[derive(Debug, Clone)]
pub struct Ingredient {
    pub name: String,
    pub category: IngredientCategory,
    pub is_organic: bool,
    pub allergen_info: Vec<AllergenType>,
    pub storage_requirements: StorageRequirements,
}

#[derive(Debug, Clone)]
pub enum IngredientCategory {
    Vegetable,
    Fruit,
    Grain,
    Legume,
    Nut,
    Seed,
    Herb,
    Spice,
    Dairy,
    Oil,
    Condiment,
}

#[derive(Debug, Clone)]
pub enum AllergenType {
    Nuts,
    Soy,
    Gluten,
    Dairy,
    Eggs,
    Sesame,
}

#[derive(Debug, Clone)]
struct StorageRequirements {
    temperature_range: (i32, i32), // Fahrenheit
    humidity_requirements: Option<String>,
    storage_type: StorageType,
}

#[derive(Debug, Clone)]
enum StorageType {
    Refrigerated,
    Frozen,
    Pantry,
    Root cellar,
}

/// Restaurant staffing information
#[derive(Debug)]
struct RestaurantStaff {
    kitchen_staff: Vec<Employee>,
    service_staff: Vec<Employee>,
    management: Vec<Employee>,
}

#[derive(Debug, Clone)]
pub struct Employee {
    pub name: String,
    pub role: JobRole,
    pub experience_years: u32,
    pub certifications: Vec<String>,
    schedule: WorkSchedule,
    compensation: CompensationInfo,
}

#[derive(Debug, Clone)]
pub enum JobRole {
    HeadChef,
    LineCook,
    PrepCook,
    Server,
    Host,
    Manager,
    Owner,
}

#[derive(Debug, Clone)]
struct WorkSchedule {
    weekly_hours: u32,
    available_days: Vec<DayOfWeek>,
    shift_preference: ShiftType,
}

#[derive(Debug, Clone)]
enum DayOfWeek {
    Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday,
}

#[derive(Debug, Clone)]
enum ShiftType {
    Morning,
    Afternoon,
    Evening,
    Split,
    Flexible,
}

#[derive(Debug, Clone)]
struct CompensationInfo {
    hourly_rate: f64,
    tips_eligible: bool,
    benefits: Vec<String>,
}

/// Business operations information
#[derive(Debug)]
struct BusinessInfo {
    hours_of_operation: OperatingHours,
    contact_info: ContactInfo,
    seating_capacity: u32,
    reservation_system: ReservationSystem,
    payment_methods: Vec<PaymentMethod>,
}

#[derive(Debug)]
struct OperatingHours {
    daily_hours: HashMap<DayOfWeek, (String, String)>, // (open_time, close_time)
    special_hours: HashMap<String, (String, String)>,  // holidays, events
}

#[derive(Debug)]
struct ContactInfo {
    phone: String,
    email: String,
    website: Option<String>,
    social_media: HashMap<String, String>,
}

#[derive(Debug)]
enum ReservationSystem {
    CallOnly,
    Online(String), // URL
    WalkInOnly,
    Both(String),   // URL
}

#[derive(Debug)]
enum PaymentMethod {
    Cash,
    Credit,
    Debit,
    DigitalWallet(String), // Apple Pay, Google Pay, etc.
    Cryptocurrency,
}

// Additional supporting types
#[derive(Debug, Clone, PartialEq)]
pub enum DietaryTag {
    Vegan,
    Vegetarian,
    GlutenFree,
    NutFree,
    SoyFree,
    DairyFree,
    Raw,
    LowCarb,
    HighProtein,
    Keto,
    Paleo,
}

#[derive(Debug, Clone)]
enum SkillLevel {
    Beginner,
    Intermediate,
    Advanced,
    Professional,
}

#[derive(Debug)]
struct KitchenEquipment {
    name: String,
    equipment_type: EquipmentType,
    status: EquipmentStatus,
    maintenance_schedule: MaintenanceInfo,
}

#[derive(Debug)]
enum EquipmentType {
    Stove,
    Oven,
    Refrigerator,
    Freezer,
    Mixer,
    FoodProcessor,
    Blender,
    Grill,
}

#[derive(Debug)]
enum EquipmentStatus {
    Working,
    Maintenance,
    Broken,
    Replaced,
}

#[derive(Debug)]
struct MaintenanceInfo {
    last_serviced: String,
    next_service_due: String,
    service_provider: String,
}

#[derive(Debug)]
struct PrepStation {
    station_id: String,
    assigned_cook: Option<String>,
    current_tasks: Vec<PrepTask>,
    equipment_available: Vec<String>,
}

#[derive(Debug)]
struct PrepTask {
    ingredient: String,
    preparation_type: String,
    quantity_needed: f32,
    estimated_time_minutes: u32,
    priority: TaskPriority,
}

#[derive(Debug)]
enum TaskPriority {
    Low,
    Medium,
    High,
    Urgent,
}

#[derive(Debug)]
struct Order {
    order_id: String,
    items: Vec<OrderItem>,
    customer_info: Option<CustomerInfo>,
    order_time: String,
    special_requests: Vec<String>,
    status: OrderStatus,
}

#[derive(Debug)]
struct OrderItem {
    menu_item: String,
    quantity: u32,
    modifications: Vec<String>,
}

#[derive(Debug)]
struct CustomerInfo {
    name: String,
    phone: Option<String>,
    dietary_restrictions: Vec<DietaryTag>,
    preferences: Vec<String>,
}

#[derive(Debug)]
enum OrderStatus {
    Received,
    InPreparation,
    Ready,
    Served,
    Cancelled,
}

// Implementation with real-world methods
impl VegetarianRestaurant {
    /// Creates a new vegetarian restaurant
    pub fn new(name: String, address: Address) -> Self {
        VegetarianRestaurant {
            name,
            location: address,
            menu: Menu {
                appetizers: Vec::new(),
                salads: Vec::new(),
                entrees: Vec::new(),
                desserts: Vec::new(),
                beverages: Vec::new(),
                daily_specials: Vec::new(),
            },
            kitchen: Kitchen {
                inventory: IngredientInventory {
                    ingredients: HashMap::new(),
                    last_updated: "Never".to_string(),
                },
                equipment: Vec::new(),
                active_orders: Vec::new(),
                prep_stations: Vec::new(),
            },
            staff: RestaurantStaff {
                kitchen_staff: Vec::new(),
                service_staff: Vec::new(),
                management: Vec::new(),
            },
            business_info: BusinessInfo {
                hours_of_operation: OperatingHours {
                    daily_hours: HashMap::new(),
                    special_hours: HashMap::new(),
                },
                contact_info: ContactInfo {
                    phone: String::new(),
                    email: String::new(),
                    website: None,
                    social_media: HashMap::new(),
                },
                seating_capacity: 0,
                reservation_system: ReservationSystem::CallOnly,
                payment_methods: vec![PaymentMethod::Cash, PaymentMethod::Credit],
            },
        }
    }

    /// Adds a menu item to the appropriate category
    pub fn add_menu_item(&mut self, item: MenuItem, category: MenuCategory) {
        match category {
            MenuCategory::Appetizer => self.menu.appetizers.push(item),
            MenuCategory::Salad => self.menu.salads.push(item),
            MenuCategory::Entree => self.menu.entrees.push(item),
            MenuCategory::Dessert => self.menu.desserts.push(item),
            MenuCategory::Beverage => self.menu.beverages.push(item),
            MenuCategory::Special => self.menu.daily_specials.push(item),
        }
    }

    /// Checks if a dish can be prepared with current inventory
    pub fn can_prepare_dish(&self, menu_item_name: &str) -> bool {
        // Search all menu categories for the item
        let all_items = [
            &self.menu.appetizers,
            &self.menu.salads,
            &self.menu.entrees,
            &self.menu.desserts,
            &self.menu.beverages,
            &self.menu.daily_specials,
        ].concat();

        if let Some(item) = all_items.iter().find(|item| item.name == menu_item_name) {
            // Check if all ingredients are available in sufficient quantity
            for ingredient in &item.recipe.ingredients {
                if let Some(stock) = self.kitchen.inventory.ingredients.get(&ingredient.name) {
                    // In a real implementation, you'd parse the amounts and compare
                    if stock.quantity_available <= 0.0 {
                        return false;
                    }
                } else {
                    return false; // Ingredient not in inventory
                }
            }
            true
        } else {
            false // Menu item not found
        }
    }

    /// Gets menu items that accommodate specific dietary restrictions
    pub fn get_dietary_friendly_items(&self, dietary_tags: &[DietaryTag]) -> Vec<&MenuItem> {
        let all_items = [
            &self.menu.appetizers,
            &self.menu.salads,
            &self.menu.entrees,
            &self.menu.desserts,
            &self.menu.beverages,
        ].concat();

        all_items.into_iter()
            .filter(|item| {
                dietary_tags.iter().all(|tag| item.dietary_tags.contains(tag))
            })
            .collect()
    }

    /// Gets the restaurant's operating status
    pub fn get_status_summary(&self) -> String {
        format!(
            "{} - {} staff members, {} menu items, capacity for {} guests",
            self.name,
            self.staff.kitchen_staff.len() + self.staff.service_staff.len(),
            self.menu.appetizers.len() + self.menu.entrees.len() + self.menu.desserts.len(),
            self.business_info.seating_capacity
        )
    }
}

#[derive(Debug)]
pub enum MenuCategory {
    Appetizer,
    Salad,
    Entree,
    Dessert,
    Beverage,
    Special,
}

// Helper implementations
impl fmt::Display for Address {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "{}, {}, {} {}", self.street, self.city, self.state, self.zip_code)
    }
}

impl MenuItem {
    pub fn new(name: String, description: String, price: f64) -> Self {
        MenuItem {
            name,
            description,
            price,
            dietary_tags: Vec::new(),
            recipe: Recipe {
                ingredients: Vec::new(),
                instructions: Vec::new(),
                prep_time_minutes: 0,
                cook_time_minutes: 0,
                skill_level: SkillLevel::Beginner,
                equipment_needed: Vec::new(),
            },
            nutritional_info: NutritionalInfo {
                calories: 0,
                protein_grams: 0.0,
                carbs_grams: 0.0,
                fat_grams: 0.0,
                fiber_grams: 0.0,
                sodium_mg: 0,
                vitamins: Vec::new(),
            },
        }
    }

    pub fn add_dietary_tag(mut self, tag: DietaryTag) -> Self {
        if !self.dietary_tags.contains(&tag) {
            self.dietary_tags.push(tag);
        }
        self
    }
}

fn main() {
    // Create a vegetarian restaurant using our well-designed structs
    let address = Address {
        street: "123 Garden Street".to_string(),
        city: "Green Valley".to_string(),
        state: "CA".to_string(),
        zip_code: "90210".to_string(),
    };

    let mut restaurant = VegetarianRestaurant::new(
        "The Fresh Garden Bistro".to_string(),
        address
    );

    // Add some menu items
    let quinoa_bowl = MenuItem::new(
        "Rainbow Quinoa Bowl".to_string(),
        "Colorful quinoa with roasted vegetables and tahini dressing".to_string(),
        16.95
    )
    .add_dietary_tag(DietaryTag::Vegan)
    .add_dietary_tag(DietaryTag::GlutenFree);

    restaurant.add_menu_item(quinoa_bowl, MenuCategory::Entree);

    // Display restaurant info
    println!("Restaurant: {}", restaurant.name);
    println!("Location: {}", restaurant.location);
    println!("Status: {}", restaurant.get_status_summary());

    // Check dietary options
    let vegan_items = restaurant.get_dietary_friendly_items(&[DietaryTag::Vegan]);
    println!("Vegan options available: {}", vegan_items.len());
}
```


## Summary and Key Takeaways

### **Mental Model: The Professional Vegetarian Kitchen**

Remember our kitchen analogy:

- 🏪 **Well-organized kitchen** = Well-designed structs
- 🗂️ **Ingredient stations** = Focused, single-responsibility structs
- 🔧 **Professional equipment** = Proper traits and methods
- 📋 **Recipe cards** = Clear documentation
- 👥 **Kitchen brigade** = Composition and modular design


### **Essential Design Principles**

1. **Clear, descriptive naming** - Make intent obvious
2. **Single responsibility** - Each struct does one thing well
3. **Smart composition** - Build complexity from simple parts
4. **Controlled visibility** - Expose what's needed, hide implementation
5. **Convenient constructors** - Make creation easy and safe
6. **Standard traits** - Derive common behaviors
7. **Memory efficiency** - Order fields thoughtfully
8. **Comprehensive documentation** - Explain purpose and usage

### **Design Decision Framework**

| **Question** | **Good Practice** | **Avoid** |
| :-- | :-- | :-- |
| **What should this struct represent?** | Single, clear concept | Multiple unrelated concepts |
| **How big should it be?** | Small, focused structs | Giant "god structs" |
| **What should be public?** | Essential API only | Everything by default |
| **How should users create it?** | Safe constructors with validation | Raw struct literals |
| **What traits does it need?** | Derive common ones | Implement everything manually |
| **How should I document it?** | Clear examples and usage | Implementation details only |

### **Quick Quality Checklist**

**✅ Well-Designed Struct:**

- Has a clear, single purpose
- Uses descriptive names for struct and fields
- Controls visibility appropriately
- Provides convenient constructors
- Derives appropriate traits
- Is well-documented with examples
- Composes smaller structs when complex
- Handles errors gracefully

**❌ Poorly-Designed Struct:**

- Tries to do too many things
- Uses vague or abbreviated names
- Makes everything public
- Forces users to set all fields manually
- Missing essential traits
- No documentation or examples
- One giant monolithic structure
- Panics or allows invalid states


### **The Ultimate Goal**

**Design structs like you're creating the perfect kitchen for a master chef** - everything should be:

- **Where you expect it** (clear organization)
- **Easy to find** (descriptive names)
- **Safe to use** (controlled access)
- **Professional quality** (proper traits and methods)
- **Well-documented** (clear instructions)
- **Efficient** (optimal layout)
- **Modular** (composable components)

Great struct design makes your Rust programs feel like working in a perfectly organized professional kitchen - every tool is in its place, every ingredient is clearly labeled, and creating amazing dishes (programs) becomes a joy rather than a struggle!

1. https://users.rust-lang.org/t/rust-best-practices/40436
2. https://users.rust-lang.org/t/best-practices-for-lifetimes-within-hierarchies-of-structs/77209
3. https://www.reddit.com/r/rust/comments/xd0p0q/i_am_still_trying_the_understand_rust_what_are/
4. https://dev.to/ajtech0001/mastering-structs-in-rust-from-basic-data-organization-to-method-implementation-11ol
5. https://users.rust-lang.org/t/best-practices-to-write-rust-code/110040
6. https://doc.rust-lang.org/book/ch05-01-defining-structs.html
7. https://serokell.io/blog/structs-in-rust
8. https://users.rust-lang.org/t/best-practices-for-struct-members-needing-initialization/119178
9. https://users.rust-lang.org/t/designing-structs/68773
10. https://www.w3schools.com/rust/rust_structs.php
11. https://stackoverflow.com/questions/25274493/whats-the-best-practice-for-creating-instances-of-a-struct
12. https://dev.to/francescoxx/structs-in-rust-2a9a
13. https://www.youtube.com/watch?v=PCjuO-Bv5FI
14. https://blog.logrocket.com/fundamentals-for-using-structs-in-rust/
15. https://www.reddit.com/r/rust/comments/uf7yoy/design_patternguidelines_to_architecture_rust_code/
16. https://doc.rust-lang.org/book/ch05-00-structs.html
17. https://rust-unofficial.github.io/patterns/
18. https://www.youtube.com/watch?v=3Ly25IYHIMc
19. https://users.rust-lang.org/t/struct-padding-rules-in-rust/69812
