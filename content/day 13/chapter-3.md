+++
title = "Builder Pattern Implementation"
description = "Implement the Builder Pattern in Rust to construct complex structs step-by-step with method chaining and improved readability."
date = 2025-08-04
draft = false
weight = 3
+++

# Builder Pattern Implementation in Rust: Comprehensive Documentation for Beginners

Understanding the Builder pattern is like learning how to be a **master chef in a vegetarian restaurant kitchen** - you create complex, delicious meals step by step, customizing each dish according to your customer's preferences. Just as a chef assembles ingredients methodically to create the perfect dish, the Builder pattern helps you construct complex objects piece by piece in a clean, readable way.

## The Vegetarian Restaurant Kitchen Analogy

### Imagine You're Running a Vegetarian Restaurant 🥬

**Traditional Constructor Problems:**

```rust
// ❌ The "overwhelming menu" problem
struct VeggieBurger {
    bun_type: String,
    patty: String,
    cheese: Option<String>,
    lettuce: bool,
    tomato: bool,
    onion: bool,
    pickle: bool,
    sauce1: Option<String>,
    sauce2: Option<String>,
    side_dish: Option<String>,
    drink: Option<String>,
}

// Customer gets confused with too many options at once
let burger = VeggieBurger {
    bun_type: "Whole wheat".to_string(),
    patty: "Black bean".to_string(),
    cheese: Some("Vegan cheese".to_string()),
    lettuce: true,
    tomato: true,
    onion: false,
    pickle: true,
    sauce1: Some("Avocado mayo".to_string()),
    sauce2: None,
    side_dish: Some("Sweet potato fries".to_string()),
    drink: Some("Green smoothie".to_string()),
};
```

**Builder Pattern Solution - Step-by-Step Ordering:**

```rust
// ✅ Like a friendly waiter guiding the customer
let burger = VeggieBurger::new()
    .bun_type("Whole wheat")           // "What kind of bun would you like?"
    .patty("Black bean")               // "Which patty sounds good?"
    .add_cheese("Vegan cheese")        // "Would you like cheese?"
    .add_lettuce()                     // "Lettuce?"
    .add_tomato()                      // "Tomato?"
    .skip_onion()                      // "No onion? Got it!"
    .add_pickle()                      // "Pickle?"
    .sauce("Avocado mayo")             // "What sauce?"
    .side_dish("Sweet potato fries")   // "What side?"
    .drink("Green smoothie")           // "What to drink?"
    .prepare();                        // "Perfect! We'll prepare that for you!"
```

**Why This Works Better:**

- 📋 **Guided experience** - Customer makes decisions step by step
- 🔄 **Flexible ordering** - Can skip or add items as desired
- 📖 **Readable recipe** - Easy to understand what's being built
- ✅ **Error prevention** - Can't forget required ingredients


## What is the Builder Pattern?

### Core Concept

The **Builder pattern** is a creational design pattern that lets you construct complex objects step by step. Instead of creating objects with a single constructor call that takes many parameters, you use a series of method calls to build the object gradually.

**Key Benefits:**

- **Readable code** - Looks like natural language
- **Flexible construction** - Only specify what you need
- **Parameter validation** - Check values as you set them
- **Immutable results** - Final object can't be accidentally modified
- **Method chaining** - Fluent, elegant API


### The Kitchen Brigade System

Think of the Builder pattern like a professional kitchen brigade:

```rust
// Each station (method) adds something to the dish
let dish = VeganPasta::start_cooking()     // 👨‍🍳 Head chef starts
    .boil_pasta("Penne")                   // 🍝 Pasta station
    .prepare_sauce("Marinara")             // 🍅 Sauce station
    .add_vegetables(&["Bell peppers", "Mushrooms"])  // 🥬 Veggie station
    .sprinkle_herbs(&["Basil", "Oregano"]) // 🌿 Herb station
    .plate_presentation("Rustic")          // 🍽️ Plating station
    .serve();                              // ✨ Final service
```

Each station (method) knows its job and passes the dish to the next station!

## Basic Builder Implementation

### Simple Vegetarian Sandwich Builder

Let's start with a basic example:

```rust
#[derive(Debug, Clone)]
struct VegetarianSandwich {
    bread: String,
    protein: String,
    vegetables: Vec<String>,
    spread: String,
    toasted: bool,
}

struct SandwichBuilder {
    bread: Option<String>,
    protein: Option<String>,
    vegetables: Vec<String>,
    spread: Option<String>,
    toasted: bool,
}

impl SandwichBuilder {
    // Start building - like greeting the customer
    fn new() -> Self {
        SandwichBuilder {
            bread: None,
            protein: None,
            vegetables: Vec::new(),
            spread: None,
            toasted: false,
        }
    }

    // Each method returns Self for chaining
    fn bread(mut self, bread_type: &str) -> Self {
        self.bread = Some(bread_type.to_string());
        self
    }

    fn protein(mut self, protein_type: &str) -> Self {
        self.protein = Some(protein_type.to_string());
        self
    }

    fn add_vegetable(mut self, veggie: &str) -> Self {
        self.vegetables.push(veggie.to_string());
        self
    }

    fn vegetables(mut self, veggies: &[&str]) -> Self {
        for veggie in veggies {
            self.vegetables.push(veggie.to_string());
        }
        self
    }

    fn spread(mut self, spread_type: &str) -> Self {
        self.spread = Some(spread_type.to_string());
        self
    }

    fn toasted(mut self) -> Self {
        self.toasted = true;
        self
    }

    // Build the final sandwich - like the chef preparing the order
    fn build(self) -> Result<VegetarianSandwich, String> {
        let bread = self.bread.ok_or("Bread is required")?;
        let protein = self.protein.ok_or("Protein is required")?;
        let spread = self.spread.ok_or("Spread is required")?;

        if self.vegetables.is_empty() {
            return Err("At least one vegetable is required".to_string());
        }

        Ok(VegetarianSandwich {
            bread,
            protein,
            vegetables: self.vegetables,
            spread,
            toasted: self.toasted,
        })
    }
}

// Convenience methods on the main struct
impl VegetarianSandwich {
    fn builder() -> SandwichBuilder {
        SandwichBuilder::new()
    }
}

fn main() -> Result<(), String> {
    // Build different sandwiches like taking orders
    let classic_veggie = VegetarianSandwich::builder()
        .bread("Whole grain")
        .protein("Hummus")
        .vegetables(&["Lettuce", "Tomato", "Cucumber"])
        .spread("Avocado mayo")
        .toasted()
        .build()?;

    let hearty_sandwich = VegetarianSandwich::builder()
        .bread("Sourdough")
        .protein("Grilled tofu")
        .add_vegetable("Spinach")
        .add_vegetable("Bell pepper")
        .add_vegetable("Red onion")
        .spread("Pesto")
        .build()?;

    println!("Classic veggie: {:#?}", classic_veggie);
    println!("Hearty sandwich: {:#?}", hearty_sandwich);

    Ok(())
}
```

**Key Points:**

- **`mut self`** - Each method takes ownership and returns it (enables chaining)
- **Validation in `build()`** - Check required fields before creating final object
- **Flexible construction** - Can add vegetables one by one or in batches
- **Error handling** - Returns `Result` if construction fails


## Advanced Vegetarian Restaurant System

### Complete Restaurant Menu Builder

Let's create a more sophisticated system for a vegetarian restaurant:

```rust
use std::collections::HashMap;

#[derive(Debug, Clone)]
enum DietaryRestriction {
    Vegan,
    GlutenFree,
    NutFree,
    SoyFree,
    LowCarb,
}

#[derive(Debug, Clone)]
enum CourseType {
    Appetizer,
    Salad,
    Soup,
    MainCourse,
    Dessert,
    Beverage,
}

#[derive(Debug, Clone)]
struct Ingredient {
    name: String,
    quantity: String,
    is_organic: bool,
}

#[derive(Debug)]
struct VegetarianDish {
    name: String,
    course_type: CourseType,
    ingredients: Vec<Ingredient>,
    cooking_method: String,
    prep_time_minutes: u32,
    serving_size: u32,
    price: f64,
    dietary_restrictions: Vec<DietaryRestriction>,
    description: String,
    chef_notes: Option<String>,
}

struct VegetarianDishBuilder {
    name: Option<String>,
    course_type: Option<CourseType>,
    ingredients: Vec<Ingredient>,
    cooking_method: String,
    prep_time_minutes: u32,
    serving_size: u32,
    price: Option<f64>,
    dietary_restrictions: Vec<DietaryRestriction>,
    description: String,
    chef_notes: Option<String>,
}

impl VegetarianDishBuilder {
    fn new() -> Self {
        VegetarianDishBuilder {
            name: None,
            course_type: None,
            ingredients: Vec::new(),
            cooking_method: "Mixed".to_string(),
            prep_time_minutes: 15, // Default prep time
            serving_size: 1,
            price: None,
            dietary_restrictions: Vec::new(),
            description: String::new(),
            chef_notes: None,
        }
    }

    fn name(mut self, name: &str) -> Self {
        self.name = Some(name.to_string());
        self
    }

    fn course(mut self, course_type: CourseType) -> Self {
        self.course_type = Some(course_type);
        self
    }

    fn add_ingredient(mut self, name: &str, quantity: &str) -> Self {
        self.ingredients.push(Ingredient {
            name: name.to_string(),
            quantity: quantity.to_string(),
            is_organic: false,
        });
        self
    }

    fn add_organic_ingredient(mut self, name: &str, quantity: &str) -> Self {
        self.ingredients.push(Ingredient {
            name: name.to_string(),
            quantity: quantity.to_string(),
            is_organic: true,
        });
        self
    }

    fn cooking_method(mut self, method: &str) -> Self {
        self.cooking_method = method.to_string();
        self
    }

    fn prep_time(mut self, minutes: u32) -> Self {
        self.prep_time_minutes = minutes;
        self
    }

    fn serves(mut self, people: u32) -> Self {
        self.serving_size = people;
        self
    }

    fn price(mut self, amount: f64) -> Self {
        self.price = Some(amount);
        self
    }

    fn suitable_for(mut self, restriction: DietaryRestriction) -> Self {
        if !self.dietary_restrictions.contains(&restriction) {
            self.dietary_restrictions.push(restriction);
        }
        self
    }

    fn description(mut self, desc: &str) -> Self {
        self.description = desc.to_string();
        self
    }

    fn chef_notes(mut self, notes: &str) -> Self {
        self.chef_notes = Some(notes.to_string());
        self
    }

    // Preset methods for common dish types
    fn vegan_friendly(mut self) -> Self {
        self.dietary_restrictions.push(DietaryRestriction::Vegan);
        // Remove any dairy ingredients that might have been added
        self.ingredients.retain(|ingredient| {
            !["cheese", "butter", "cream", "milk", "yogurt"]
                .iter()
                .any(|dairy| ingredient.name.to_lowercase().contains(dairy))
        });
        self
    }

    fn gluten_free(mut self) -> Self {
        self.dietary_restrictions.push(DietaryRestriction::GlutenFree);
        self
    }

    fn quick_prep(mut self) -> Self {
        self.prep_time_minutes = 10;
        self
    }

    fn build(self) -> Result<VegetarianDish, String> {
        let name = self.name.ok_or("Dish name is required")?;
        let course_type = self.course_type.ok_or("Course type is required")?;
        let price = self.price.ok_or("Price is required")?;

        if self.ingredients.is_empty() {
            return Err("At least one ingredient is required".to_string());
        }

        if price <= 0.0 {
            return Err("Price must be positive".to_string());
        }

        if self.description.is_empty() {
            return Err("Description is required".to_string());
        }

        Ok(VegetarianDish {
            name,
            course_type,
            ingredients: self.ingredients,
            cooking_method: self.cooking_method,
            prep_time_minutes: self.prep_time_minutes,
            serving_size: self.serving_size,
            price,
            dietary_restrictions: self.dietary_restrictions,
            description: self.description,
            chef_notes: self.chef_notes,
        })
    }
}

impl VegetarianDish {
    fn builder() -> VegetarianDishBuilder {
        VegetarianDishBuilder::new()
    }
}

fn main() -> Result<(), String> {
    // Create different dishes like a chef designing the menu

    // Appetizer - Stuffed Mushrooms
    let stuffed_mushrooms = VegetarianDish::builder()
        .name("Mediterranean Stuffed Mushrooms")
        .course(CourseType::Appetizer)
        .add_organic_ingredient("Large portobello mushrooms", "4 pieces")
        .add_ingredient("Sun-dried tomatoes", "1/2 cup")
        .add_ingredient("Fresh mozzarella", "4 oz")
        .add_ingredient("Fresh basil", "2 tbsp")
        .add_ingredient("Garlic", "2 cloves")
        .cooking_method("Baked")
        .prep_time(25)
        .serves(2)
        .price(12.95)
        .suitable_for(DietaryRestriction::GlutenFree)
        .description("Juicy portobello mushrooms stuffed with sun-dried tomatoes, fresh mozzarella, and basil")
        .chef_notes("Best served immediately while cheese is still melting")
        .build()?;

    // Main Course - Vegan Buddha Bowl
    let buddha_bowl = VegetarianDish::builder()
        .name("Rainbow Buddha Bowl")
        .course(CourseType::MainCourse)
        .add_organic_ingredient("Quinoa", "1 cup cooked")
        .add_organic_ingredient("Roasted chickpeas", "1/2 cup")
        .add_ingredient("Avocado", "1/2 medium")
        .add_ingredient("Red cabbage", "1/2 cup shredded")
        .add_ingredient("Carrots", "1/2 cup julienned")
        .add_ingredient("Spinach", "2 cups fresh")
        .add_ingredient("Tahini", "2 tbsp")
        .cooking_method("Raw and roasted")
        .quick_prep()
        .serves(1)
        .price(16.50)
        .vegan_friendly()
        .gluten_free()
        .suitable_for(DietaryRestriction::NutFree)
        .description("A colorful, nutritious bowl packed with plant-based protein and fresh vegetables")
        .chef_notes("Drizzle with lemon-tahini dressing just before serving")
        .build()?;

    // Dessert - Chocolate Avocado Mousse
    let chocolate_mousse = VegetarianDish::builder()
        .name("Vegan Chocolate Avocado Mousse")
        .course(CourseType::Dessert)
        .add_organic_ingredient("Ripe avocados", "2 large")
        .add_ingredient("Cocoa powder", "1/4 cup")
        .add_ingredient("Maple syrup", "3 tbsp")
        .add_ingredient("Vanilla extract", "1 tsp")
        .add_ingredient("Coconut cream", "2 tbsp")
        .cooking_method("Blended")
        .prep_time(15)
        .serves(4)
        .price(8.95)
        .vegan_friendly()
        .gluten_free()
        .suitable_for(DietaryRestriction::SoyFree)
        .description("Rich, creamy chocolate mousse made with wholesome avocados")
        .chef_notes("Chill for at least 2 hours before serving for best texture")
        .build()?;

    // Display the menu
    println!("🥬 VEGETARIAN RESTAURANT MENU 🥬\n");

    let dishes = vec![stuffed_mushrooms, buddha_bowl, chocolate_mousse];

    for dish in dishes {
        println!("📖 {}", dish.name);
        println!("   Course: {:?}", dish.course_type);
        println!("   Price: ${:.2}", dish.price);
        println!("   Prep Time: {} minutes", dish.prep_time_minutes);
        println!("   Serves: {}", dish.serving_size);

        if !dish.dietary_restrictions.is_empty() {
            println!("   Dietary: {:?}", dish.dietary_restrictions);
        }

        println!("   Description: {}", dish.description);

        if let Some(notes) = &dish.chef_notes {
            println!("   Chef's Note: {}", notes);
        }

        println!("   Ingredients:");
        for ingredient in &dish.ingredients {
            let organic_label = if ingredient.is_organic { " (Organic)" } else { "" };
            println!("     - {} - {}{}", ingredient.name, ingredient.quantity, organic_label);
        }

        println!(); // Empty line between dishes
    }

    Ok(())
}
```


## Builder Pattern Variations

### 1. Non-Consuming Builder (Reusable Templates)

Sometimes you want to reuse a builder as a template:

```rust
struct MenuTemplateBuilder {
    base_price: f64,
    cooking_method: String,
    dietary_restrictions: Vec<DietaryRestriction>,
}

impl MenuTemplateBuilder {
    fn new() -> Self {
        MenuTemplateBuilder {
            base_price: 10.0,
            cooking_method: "Mixed".to_string(),
            dietary_restrictions: Vec::new(),
        }
    }

    // Non-consuming methods - return &mut self
    fn set_base_price(&mut self, price: f64) -> &mut Self {
        self.base_price = price;
        self
    }

    fn set_cooking_method(&mut self, method: &str) -> &mut Self {
        self.cooking_method = method.to_string();
        self
    }

    fn add_dietary_restriction(&mut self, restriction: DietaryRestriction) -> &mut Self {
        self.dietary_restrictions.push(restriction);
        self
    }

    // Create a dish builder from this template
    fn create_dish_builder(&self, name: &str) -> VegetarianDishBuilder {
        let mut builder = VegetarianDish::builder()
            .name(name)
            .price(self.base_price)
            .cooking_method(&self.cooking_method);

        for restriction in &self.dietary_restrictions {
            builder = builder.suitable_for(restriction.clone());
        }

        builder
    }
}

fn main() -> Result<(), String> {
    // Create reusable templates
    let mut vegan_template = MenuTemplateBuilder::new();
    vegan_template
        .set_base_price(14.00)
        .set_cooking_method("Plant-based")
        .add_dietary_restriction(DietaryRestriction::Vegan)
        .add_dietary_restriction(DietaryRestriction::SoyFree);

    // Use template to create multiple dishes
    let vegan_salad = vegan_template
        .create_dish_builder("Garden Fresh Salad")
        .course(CourseType::Salad)
        .add_ingredient("Mixed greens", "3 cups")
        .add_ingredient("Cherry tomatoes", "1 cup")
        .description("Fresh, crisp salad with seasonal vegetables")
        .build()?;

    let vegan_soup = vegan_template
        .create_dish_builder("Hearty Lentil Soup")
        .course(CourseType::Soup)
        .add_ingredient("Red lentils", "1 cup")
        .add_ingredient("Vegetable broth", "4 cups")
        .description("Warming soup perfect for cold days")
        .build()?;

    println!("Vegan dishes created from template:");
    println!("1. {:?}", vegan_salad.name);
    println!("2. {:?}", vegan_soup.name);

    Ok(())
}
```


### 2. Typed Builder (Compile-Time Safety)

Ensure required fields are set at compile time:

```rust
// State markers
struct NoName;
struct HasName;
struct NoCourse;
struct HasCourse;
struct NoPrice;
struct HasPrice;

struct TypedDishBuilder<N, C, P> {
    name: Option<String>,
    course_type: Option<CourseType>,
    price: Option<f64>,
    ingredients: Vec<Ingredient>,
    description: String,
    _name: std::marker::PhantomData<N>,
    _course: std::marker::PhantomData<C>,
    _price: std::marker::PhantomData<P>,
}

impl TypedDishBuilder<NoName, NoCourse, NoPrice> {
    fn new() -> Self {
        TypedDishBuilder {
            name: None,
            course_type: None,
            price: None,
            ingredients: Vec::new(),
            description: String::new(),
            _name: std::marker::PhantomData,
            _course: std::marker::PhantomData,
            _price: std::marker::PhantomData,
        }
    }
}

impl<C, P> TypedDishBuilder<NoName, C, P> {
    fn name(self, name: &str) -> TypedDishBuilder<HasName, C, P> {
        TypedDishBuilder {
            name: Some(name.to_string()),
            course_type: self.course_type,
            price: self.price,
            ingredients: self.ingredients,
            description: self.description,
            _name: std::marker::PhantomData,
            _course: std::marker::PhantomData,
            _price: std::marker::PhantomData,
        }
    }
}

impl<N, P> TypedDishBuilder<N, NoCourse, P> {
    fn course(self, course_type: CourseType) -> TypedDishBuilder<N, HasCourse, P> {
        TypedDishBuilder {
            name: self.name,
            course_type: Some(course_type),
            price: self.price,
            ingredients: self.ingredients,
            description: self.description,
            _name: std::marker::PhantomData,
            _course: std::marker::PhantomData,
            _price: std::marker::PhantomData,
        }
    }
}

impl<N, C> TypedDishBuilder<N, C, NoPrice> {
    fn price(self, amount: f64) -> TypedDishBuilder<N, C, HasPrice> {
        TypedDishBuilder {
            name: self.name,
            course_type: self.course_type,
            price: Some(amount),
            ingredients: self.ingredients,
            description: self.description,
            _name: std::marker::PhantomData,
            _course: std::marker::PhantomData,
            _price: std::marker::PhantomData,
        }
    }
}

// Methods available at any stage
impl<N, C, P> TypedDishBuilder<N, C, P> {
    fn add_ingredient(mut self, name: &str, quantity: &str) -> Self {
        self.ingredients.push(Ingredient {
            name: name.to_string(),
            quantity: quantity.to_string(),
            is_organic: false,
        });
        self
    }

    fn description(mut self, desc: &str) -> Self {
        self.description = desc.to_string();
        self
    }
}

// Only allow build when all required fields are set
impl TypedDishBuilder<HasName, HasCourse, HasPrice> {
    fn build(self) -> VegetarianDish {
        VegetarianDish {
            name: self.name.unwrap(),
            course_type: self.course_type.unwrap(),
            price: self.price.unwrap(),
            ingredients: self.ingredients,
            cooking_method: "Mixed".to_string(),
            prep_time_minutes: 15,
            serving_size: 1,
            dietary_restrictions: Vec::new(),
            description: self.description,
            chef_notes: None,
        }
    }
}
```


## Real-World Patterns

### Configuration Builder

```rust
#[derive(Debug)]
struct RestaurantConfig {
    name: String,
    address: String,
    phone: String,
    email: Option<String>,
    website: Option<String>,
    opening_hours: HashMap<String, String>,
    max_capacity: u32,
    takeout_available: bool,
    delivery_available: bool,
    payment_methods: Vec<String>,
}

struct RestaurantConfigBuilder {
    name: Option<String>,
    address: Option<String>,
    phone: Option<String>,
    email: Option<String>,
    website: Option<String>,
    opening_hours: HashMap<String, String>,
    max_capacity: u32,
    takeout_available: bool,
    delivery_available: bool,
    payment_methods: Vec<String>,
}

impl RestaurantConfigBuilder {
    fn new() -> Self {
        RestaurantConfigBuilder {
            name: None,
            address: None,
            phone: None,
            email: None,
            website: None,
            opening_hours: HashMap::new(),
            max_capacity: 50,
            takeout_available: true,
            delivery_available: false,
            payment_methods: vec!["Cash".to_string(), "Card".to_string()],
        }
    }

    fn name(mut self, name: &str) -> Self {
        self.name = Some(name.to_string());
        self
    }

    fn address(mut self, address: &str) -> Self {
        self.address = Some(address.to_string());
        self
    }

    fn phone(mut self, phone: &str) -> Self {
        self.phone = Some(phone.to_string());
        self
    }

    fn email(mut self, email: &str) -> Self {
        self.email = Some(email.to_string());
        self
    }

    fn website(mut self, website: &str) -> Self {
        self.website = Some(website.to_string());
        self
    }

    fn hours(mut self, day: &str, hours: &str) -> Self {
        self.opening_hours.insert(day.to_string(), hours.to_string());
        self
    }

    fn capacity(mut self, max_people: u32) -> Self {
        self.max_capacity = max_people;
        self
    }

    fn enable_takeout(mut self) -> Self {
        self.takeout_available = true;
        self
    }

    fn enable_delivery(mut self) -> Self {
        self.delivery_available = true;
        self
    }

    fn add_payment_method(mut self, method: &str) -> Self {
        if !self.payment_methods.contains(&method.to_string()) {
            self.payment_methods.push(method.to_string());
        }
        self
    }

    fn build(self) -> Result<RestaurantConfig, String> {
        let name = self.name.ok_or("Restaurant name is required")?;
        let address = self.address.ok_or("Address is required")?;
        let phone = self.phone.ok_or("Phone number is required")?;

        Ok(RestaurantConfig {
            name,
            address,
            phone,
            email: self.email,
            website: self.website,
            opening_hours: self.opening_hours,
            max_capacity: self.max_capacity,
            takeout_available: self.takeout_available,
            delivery_available: self.delivery_available,
            payment_methods: self.payment_methods,
        })
    }
}

impl RestaurantConfig {
    fn builder() -> RestaurantConfigBuilder {
        RestaurantConfigBuilder::new()
    }
}

fn main() -> Result<(), String> {
    let restaurant = RestaurantConfig::builder()
        .name("Green Garden Bistro")
        .address("123 Veggie Lane, Plant City, PC 12345")
        .phone("(555) 123-VEGGIE")
        .email("info@greengardenrestro.com")
        .website("www.greengardenbistro.com")
        .hours("Monday", "11:00 AM - 9:00 PM")
        .hours("Tuesday", "11:00 AM - 9:00 PM")
        .hours("Wednesday", "11:00 AM - 9:00 PM")
        .hours("Thursday", "11:00 AM - 10:00 PM")
        .hours("Friday", "11:00 AM - 10:00 PM")
        .hours("Saturday", "10:00 AM - 10:00 PM")
        .hours("Sunday", "10:00 AM - 8:00 PM")
        .capacity(75)
        .enable_takeout()
        .enable_delivery()
        .add_payment_method("Mobile Pay")
        .add_payment_method("Crypto")
        .build()?;

    println!("Restaurant Configuration:\n{:#?}", restaurant);

    Ok(())
}
```


## Best Practices and Common Patterns

### 1. Default Values with Override

```rust
impl VegetarianDishBuilder {
    fn with_defaults() -> Self {
        Self::new()
            .cooking_method("Sautéed")
            .prep_time(20)
            .serves(2)
    }

    fn healthy_option(mut self) -> Self {
        self.dietary_restrictions.extend(vec![
            DietaryRestriction::Vegan,
            DietaryRestriction::GlutenFree,
            DietaryRestriction::LowCarb,
        ]);
        self
    }

    fn quick_meal(mut self) -> Self {
        self.prep_time_minutes = 10;
        self.cooking_method = "Raw or minimal cooking".to_string();
        self
    }
}
```


### 2. Validation Strategies

```rust
impl VegetarianDishBuilder {
    fn validate(&self) -> Result<(), String> {
        if self.name.is_none() {
            return Err("Dish name is required".to_string());
        }

        if let Some(price) = self.price {
            if price < 0.0 {
                return Err("Price cannot be negative".to_string());
            }
            if price > 100.0 {
                return Err("Price seems too high, please verify".to_string());
            }
        }

        if self.prep_time_minutes > 120 {
            return Err("Prep time over 2 hours might not be practical".to_string());
        }

        // Check for ingredient combinations that don't make sense
        let ingredient_names: Vec<String> = self.ingredients
            .iter()
            .map(|i| i.name.to_lowercase())
            .collect();

        if ingredient_names.contains(&"meat".to_string()) {
            return Err("This is a vegetarian restaurant - no meat allowed!".to_string());
        }

        Ok(())
    }

    fn build(self) -> Result<VegetarianDish, String> {
        self.validate()?;

        let name = self.name.ok_or("Dish name is required")?;
        let course_type = self.course_type.ok_or("Course type is required")?;
        let price = self.price.ok_or("Price is required")?;

        // Rest of build logic...
        Ok(VegetarianDish {
            name,
            course_type,
            ingredients: self.ingredients,
            cooking_method: self.cooking_method,
            prep_time_minutes: self.prep_time_minutes,
            serving_size: self.serving_size,
            price,
            dietary_restrictions: self.dietary_restrictions,
            description: self.description,
            chef_notes: self.chef_notes,
        })
    }
}
```


### 3. Builder with Conditional Logic

```rust
impl VegetarianDishBuilder {
    fn if_condition<F>(mut self, condition: bool, f: F) -> Self
    where
        F: FnOnce(Self) -> Self,
    {
        if condition {
            f(self)
        } else {
            self
        }
    }

    fn maybe_add_ingredient(mut self, ingredient: Option<(&str, &str)>) -> Self {
        if let Some((name, quantity)) = ingredient {
            self.add_ingredient(name, quantity)
        } else {
            self
        }
    }
}

fn main() -> Result<(), String> {
    let is_vegan_customer = true;
    let has_nut_allergy = true;
    let preferred_cheese = if is_vegan_customer { None } else { Some(("Mozzarella", "2 oz")) };

    let customized_dish = VegetarianDish::builder()
        .name("Customized Pasta")
        .course(CourseType::MainCourse)
        .add_ingredient("Pasta", "1 cup")
        .add_ingredient("Tomato sauce", "1/2 cup")
        .if_condition(is_vegan_customer, |builder| {
            builder.vegan_friendly()
                .add_ingredient("Nutritional yeast", "2 tbsp")
        })
        .if_condition(has_nut_allergy, |builder| {
            builder.suitable_for(DietaryRestriction::NutFree)
        })
        .maybe_add_ingredient(preferred_cheese)
        .price(13.95)
        .description("Pasta dish customized to dietary preferences")
        .build()?;

    println!("Customized dish: {:#?}", customized_dish);

    Ok(())
}
```


## Common Mistakes and Solutions

### Mistake 1: Not Returning Self

```rust
// ❌ Wrong - breaks method chaining
impl SandwichBuilder {
    fn add_vegetable(mut self, veggie: &str) {
        self.vegetables.push(veggie.to_string());
        // Missing: self
    }
}

// ✅ Correct - enables chaining
impl SandwichBuilder {
    fn add_vegetable(mut self, veggie: &str) -> Self {
        self.vegetables.push(veggie.to_string());
        self
    }
}
```


### Mistake 2: Inconsistent Parameter Types

```rust
// ❌ Inconsistent - some methods take &str, others take String
impl DishBuilder {
    fn name(mut self, name: String) -> Self { /* ... */ }
    fn description(mut self, desc: &str) -> Self { /* ... */ }
}

// ✅ Consistent - all methods accept the same parameter type
impl DishBuilder {
    fn name(mut self, name: &str) -> Self {
        self.name = Some(name.to_string());
        self
    }

    fn description(mut self, desc: &str) -> Self {
        self.description = desc.to_string();
        self
    }
}
```


### Mistake 3: Poor Error Messages

```rust
// ❌ Vague error messages
fn build(self) -> Result<VegetarianDish, String> {
    let name = self.name.ok_or("Missing field")?;
    let price = self.price.ok_or("Invalid value")?;
    // ...
}

// ✅ Helpful error messages
fn build(self) -> Result<VegetarianDish, String> {
    let name = self.name.ok_or("Dish name is required. Use .name(\"Dish Name\")")?;
    let price = self.price.ok_or("Price is required. Use .price(amount)")?;

    if price <= 0.0 {
        return Err(format!("Price must be positive, got {}", price));
    }

    // ...
}
```


## Summary and Key Takeaways

### **Mental Model: The Restaurant Kitchen**

Remember the kitchen analogy:

- 🏪 **Restaurant** = Your application
- 👨🍳 **Chef** = Builder pattern
- 📋 **Order** = Method calls
- 🍽️ **Finished dish** = Built object
- 🔄 **Preparation steps** = Chained methods


### **Core Principles**

1. **Step-by-step construction** - Build complex objects gradually
2. **Fluent interface** - Method chaining creates readable code
3. **Validation** - Check constraints before building final object
4. **Flexibility** - Only set the fields you need
5. **Immutability** - Final object can't be accidentally modified

### **When to Use Builder Pattern**

| **Use When** | **Example** | **Benefit** |
| :-- | :-- | :-- |
| **Many optional parameters** | Configuration objects | Avoid telescoping constructors |
| **Complex object creation** | Database queries | Step-by-step construction |
| **Validation needed** | User input forms | Validate during construction |
| **Multiple variations** | API request builders | Flexible object creation |
| **Readable APIs** | DSL-like interfaces | Natural language feel |

### **Quick Implementation Checklist**

```rust
// ✅ Essential Builder Implementation Steps

// 1. Create builder struct with Option/default fields
struct MyBuilder {
    required_field: Option<String>,
    optional_field: Option<String>,
    default_field: i32,
}

// 2. Constructor method
impl MyBuilder {
    fn new() -> Self { /* ... */ }
}

// 3. Builder methods (return Self)
impl MyBuilder {
    fn field(mut self, value: Type) -> Self {
        self.field = Some(value);
        self
    }
}

// 4. Build method with validation
impl MyBuilder {
    fn build(self) -> Result<MyStruct, String> {
        // Validate required fields
        // Create and return final object
    }
}

// 5. Convenience method on main struct
impl MyStruct {
    fn builder() -> MyBuilder {
        MyBuilder::new()
    }
}
```


### **Best Practices Summary**

- **Return `Self`** from all builder methods to enable chaining
- **Use `Option<T>`** for required fields, direct values for optional ones
- **Provide clear error messages** with guidance on how to fix issues
- **Validate in `build()`** method before creating final object
- **Consider providing preset methods** for common configurations
- **Use consistent parameter types** across all methods
- **Document your builder methods** with examples

**The Builder pattern is like having a patient, helpful waiter** who guides customers through creating their perfect meal step by step. It transforms the overwhelming experience of choosing from dozens of options into a pleasant, guided journey that results in exactly what the customer wanted!

1. https://rust-unofficial.github.io/patterns/patterns/creational/builder.html
2. https://refactoring.guru/design-patterns/builder/rust/example
3. https://www.reddit.com/r/rust/comments/10nm7mz/til_all_about_the_builder_pattern/
4. https://blog.coolhead.in/builder-patter-in-rust-explained-with-example
5. https://dev-state.com/posts/builder_pattern/
6. https://refactoring.guru/design-patterns/builder
7. https://www.hackingwithrust.net/2023/03/24/design-patterns-in-rust-builder-pattern/
8. https://zerotomastery.io/blog/rust-struct-guide/
9. https://en.wikipedia.org/wiki/Builder_pattern
10. https://www.reddit.com/r/rust/comments/qb5v0i/builder_pattern_in_rust_blog_article_for_beginners/
11. https://www.youtube.com/watch?v=Z_3WOSiYYFY
12. https://www.geeksforgeeks.org/system-design/builder-design-pattern/
13. https://dev.to/mindflavor/rust-builder-pattern-with-types-3chf
14. https://users.rust-lang.org/t/builder-pattern-implementation/101959
15. https://blogs.oracle.com/javamagazine/post/exploring-joshua-blochs-builder-design-pattern-in-java
16. https://dhghomon.github.io/easy_rust/Chapter_55.html
17. https://stackoverflow.com/questions/76585928/implementing-builder-pattern-in-rust
18. https://www.digitalocean.com/community/tutorials/builder-design-pattern-in-java
19. https://blog.ediri.io/design-patterns-in-rust-upgrading-the-builder-pattern-using-the-typestate-pattern
20. https://stackoverflow.com/questions/328496/when-would-you-use-the-builder-pattern
21. https://softwarepatterns.com/rust/builder-software-pattern-rust-example
22. https://www.youtube.com/watch?v=5DWU-56mjmg
23. http://juggernaut.github.io/rust/2017/11/02/rust-builder-pattern.html
24. https://www.shuttle.dev/blog/2022/06/09/the-builder-pattern
