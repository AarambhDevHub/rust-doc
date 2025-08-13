+++
title = "Performance Considerations"
description = "Understand the performance implications of different Rust programming techniques, including memory allocation, borrowing, cloning, and iteration."
date = 2025-08-13
weight = 5
keywords = ["Rust", "performance", "optimization", "memory", "cloning", "iteration", "borrowing"]
+++

# Performance Considerations for Vectors of Enums in Rust: Comprehensive Documentation for Beginners

Understanding performance considerations when working with vectors of enums is like learning to **optimize the operations of a high-volume vegetarian restaurant during peak hours** - you need to understand how your unified order management system performs under pressure, where bottlenecks occur, and how to design your processes for maximum efficiency. Just as a professional restaurant must balance menu variety with kitchen efficiency, Rust vectors of enums require careful consideration of memory layout, access patterns, and optimization strategies to maintain peak performance while handling diverse data types.

## The High-Performance Restaurant Operations Analogy 🏃♂️

### Imagine You're Optimizing a Busy Vegetarian Restaurant for Peak Efficiency

**Performance Challenges with Diverse Menu Systems:**

```rust
// ❌ Inefficient - like having wildly different prep times for each dish
enum SlowRestaurantData {
    QuickSnack(u8),                           // 1 byte data, 5 min prep
    ComplexMeal(Vec<String>),                 // Large data, 45 min prep
    AverageOrder(String),                     // Medium data, 20 min prep
    GiantFeastData([u8; 1000]),              // Huge data, 60 min prep
}
// Problem: Enum size = largest variant = 1000+ bytes!
// Even storing a QuickSnack takes 1000+ bytes!
```

**Optimized Performance Approach:**

```rust
// ✅ Efficient - like organizing kitchen for optimal flow
enum OptimizedRestaurantData {
    QuickSnack(u8),                          // Keep small data inline
    AverageOrder(String),                    // Medium data acceptable
    ComplexMeal(Box<Vec<String>>),           // Box large data
    GiantFeast(Box<[u8; 1000]>),            // Box huge data
}
// Now enum size = ~24 bytes (size of largest non-boxed variant + pointer)
// Small data stays fast, large data goes to heap efficiently
```

**Why Performance Optimization Matters:**

- ⚡ **Memory efficiency** - Minimize wasted space like optimizing kitchen storage
- 🎯 **Cache performance** - Keep hot data close like organizing prep stations
- 📊 **Predictable costs** - Know the performance characteristics of operations
- 🔄 **Scalable operations** - Handle growing data volumes efficiently
- 🛠️ **Resource optimization** - Make the best use of available system resources


## Understanding Enum Memory Layout and Performance Impact

### Memory Layout Fundamentals

**How Rust arranges enum data in memory:**

```rust
use std::mem;

fn demonstrate_memory_layout_analysis() {
    println!("💾 Restaurant Data Memory Layout Analysis");
    println!("{:=<55}", "");

    // Different enum designs and their memory implications

    // Design 1: Small variants only
    #[derive(Debug)]
    enum SmallData {
        ItemCount(u8),     // 1 byte
        Rating(u8),        // 1 byte
        IsAvailable(bool), // 1 byte
        TableNumber(u16),  // 2 bytes
    }

    // Design 2: Mixed sizes without optimization
    #[derive(Debug)]
    enum MixedSizesNaive {
        Small(u8),                    // 1 byte data
        Medium(String),               // 24 bytes data (on 64-bit)
        Large(Vec<u8>),              // 24 bytes data
        Huge([u8; 1000]),            // 1000 bytes data!
    }

    // Design 3: Optimized mixed sizes
    #[derive(Debug)]
    enum MixedSizesOptimized {
        Small(u8),                    // 1 byte data
        Medium(String),               // 24 bytes data
        Large(Box<Vec<u8>>),         // 8 bytes pointer
        Huge(Box<[u8; 1000]>),       // 8 bytes pointer
    }

    println!("📊 Memory Size Comparison:");
    println!("   SmallData enum: {} bytes", mem::size_of::<SmallData>());
    println!("   MixedSizesNaive enum: {} bytes", mem::size_of::<MixedSizesNaive>());
    println!("   MixedSizesOptimized enum: {} bytes", mem::size_of::<MixedSizesOptimized>());

    println!("\n🔍 Memory Layout Breakdown:");

    // Analyze alignment and padding
    println!("   SmallData alignment: {} bytes", mem::align_of::<SmallData>());
    println!("   MixedSizesNaive alignment: {} bytes", mem::align_of::<MixedSizesNaive>());
    println!("   MixedSizesOptimized alignment: {} bytes", mem::align_of::<MixedSizesOptimized>());

    // Calculate memory efficiency for vectors
    let vector_sizes = [100, 1000, 10000];

    for size in vector_sizes {
        println!("\n📦 Vector of {} items memory usage:");

        let small_total = size * mem::size_of::<SmallData>();
        let naive_total = size * mem::size_of::<MixedSizesNaive>();
        let optimized_total = size * mem::size_of::<MixedSizesOptimized>();

        println!("   Small-only vector: {} KB", small_total / 1024);
        println!("   Naive mixed vector: {} KB", naive_total / 1024);
        println!("   Optimized mixed vector: {} KB", optimized_total / 1024);

        if naive_total > optimized_total {
            let savings = ((naive_total - optimized_total) as f64 / naive_total as f64) * 100.0;
            println!("   💰 Memory savings: {:.1}%", savings);
        }
    }

    // Demonstrate discriminant overhead
    println!("\n🏷️ Enum Discriminant Analysis:");
    println!("   Every enum needs a 'tag' to identify which variant it contains");
    println!("   SmallData discriminant space: ~{} bytes",
             mem::size_of::<SmallData>() - 2); // Largest variant is u16
    println!("   This is the 'menu category' overhead for each order");
}

fn main() {
    demonstrate_memory_layout_analysis();
}
```


### Cache Performance Implications

**How memory layout affects performance:**

```rust
use std::time::Instant;

fn demonstrate_cache_performance() {
    println!("🚀 Cache Performance Analysis");
    println!("{:=<35}", "");

    #[derive(Clone)]
    enum CacheFriendlyData {
        Counter(u32),
        Flag(bool),
        Rating(u8),
        SmallText([u8; 16]), // Fixed-size, cache-friendly
    }

    #[derive(Clone)]
    enum CacheUnfriendlyData {
        Counter(u32),
        Flag(bool),
        Rating(u8),
        LargeText(String), // Variable size, potential cache misses
        HugeArray([u8; 512]), // Large inline data
    }

    // Create test data
    let cache_friendly: Vec<CacheFriendlyData> = (0..100_000)
        .map(|i| match i % 4 {
            0 => CacheFriendlyData::Counter(i as u32),
            1 => CacheFriendlyData::Flag(i % 2 == 0),
            2 => CacheFriendlyData::Rating((i % 5) as u8),
            _ => CacheFriendlyData::SmallText([b'A'; 16]),
        })
        .collect();

    let cache_unfriendly: Vec<CacheUnfriendlyData> = (0..100_000)
        .map(|i| match i % 4 {
            0 => CacheUnfriendlyData::Counter(i as u32),
            1 => CacheUnfriendlyData::Flag(i % 2 == 0),
            2 => CacheUnfriendlyData::Rating((i % 5) as u8),
            _ => CacheUnfriendlyData::HugeArray([b'A'; 512]),
        })
        .collect();

    println!("📊 Performance Test Setup:");
    println!("   Cache-friendly enum size: {} bytes",
             std::mem::size_of::<CacheFriendlyData>());
    println!("   Cache-unfriendly enum size: {} bytes",
             std::mem::size_of::<CacheUnfriendlyData>());
    println!("   Test data size: {} items each", cache_friendly.len());

    // Test iteration performance
    println!("\n⏱️ Iteration Performance Test:");

    // Cache-friendly iteration
    let start = Instant::now();
    let mut count_friendly = 0u64;
    for item in &cache_friendly {
        match item {
            CacheFriendlyData::Counter(n) => count_friendly += *n as u64,
            CacheFriendlyData::Flag(true) => count_friendly += 1,
            CacheFriendlyData::Rating(r) => count_friendly += *r as u64,
            CacheFriendlyData::SmallText(_) => count_friendly += 10,
        }
    }
    let friendly_time = start.elapsed();

    // Cache-unfriendly iteration
    let start = Instant::now();
    let mut count_unfriendly = 0u64;
    for item in &cache_unfriendly {
        match item {
            CacheUnfriendlyData::Counter(n) => count_unfriendly += *n as u64,
            CacheUnfriendlyData::Flag(true) => count_unfriendly += 1,
            CacheUnfriendlyData::Rating(r) => count_unfriendly += *r as u64,
            CacheUnfriendlyData::HugeArray(_) => count_unfriendly += 10,
        }
    }
    let unfriendly_time = start.elapsed();

    println!("   Cache-friendly iteration: {:?}", friendly_time);
    println!("   Cache-unfriendly iteration: {:?}", unfriendly_time);

    if unfriendly_time > friendly_time {
        let slowdown = unfriendly_time.as_nanos() as f64 / friendly_time.as_nanos() as f64;
        println!("   📈 Cache-unfriendly is {:.2}x slower", slowdown);
    }

    // Test memory access patterns
    println!("\n🔍 Memory Access Pattern Analysis:");

    let friendly_memory = cache_friendly.len() * std::mem::size_of::<CacheFriendlyData>();
    let unfriendly_memory = cache_unfriendly.len() * std::mem::size_of::<CacheUnfriendlyData>();

    println!("   Friendly total memory: {} KB", friendly_memory / 1024);
    println!("   Unfriendly total memory: {} KB", unfriendly_memory / 1024);
    println!("   Memory overhead: {}x", unfriendly_memory / friendly_memory);

    // Estimate cache performance
    let typical_cache_line = 64; // bytes
    let friendly_items_per_line = typical_cache_line / std::mem::size_of::<CacheFriendlyData>();
    let unfriendly_items_per_line = typical_cache_line / std::mem::size_of::<CacheUnfriendlyData>();

    println!("\n🏪 Cache Line Efficiency:");
    println!("   Friendly items per cache line: {}", friendly_items_per_line);
    println!("   Unfriendly items per cache line: {}", unfriendly_items_per_line);

    if friendly_items_per_line > unfriendly_items_per_line {
        println!("   ✅ Cache-friendly design fits {}x more items per cache line",
                 friendly_items_per_line / unfriendly_items_per_line.max(1));
    }
}

fn main() {
    demonstrate_cache_performance();
}
```


## Optimization Strategies for High-Performance Enums

### 1. Boxing Large Variants

**Managing large data efficiently:**

```rust
use std::time::Instant;

fn demonstrate_boxing_optimization() {
    println!("📦 Boxing Optimization for Large Data");
    println!("{:=<45}", "");

    // Restaurant order data with different approaches
    #[derive(Debug, Clone)]
    enum UnoptimizedOrder {
        SimpleItem(String),                    // ~24 bytes
        ComplexMeal(Vec<String>),             // ~24 bytes
        CateringOrder([String; 100]),         // ~2400 bytes!
        SpecialEvent {                        // Large struct
            name: String,
            guest_count: u32,
            menu_items: Vec<String>,
            dietary_restrictions: Vec<String>,
            special_instructions: String,
            contact_info: String,
            budget: f64,
            date: String,
            venue_requirements: [String; 50], // ~1200 bytes
        },
    }

    #[derive(Debug, Clone)]
    enum OptimizedOrder {
        SimpleItem(String),                           // ~24 bytes
        ComplexMeal(Vec<String>),                    // ~24 bytes
        CateringOrder(Box<[String; 100]>),           // 8 bytes pointer
        SpecialEvent(Box<SpecialEventData>),         // 8 bytes pointer
    }

    #[derive(Debug, Clone)]
    struct SpecialEventData {
        name: String,
        guest_count: u32,
        menu_items: Vec<String>,
        dietary_restrictions: Vec<String>,
        special_instructions: String,
        contact_info: String,
        budget: f64,
        date: String,
        venue_requirements: Vec<String>, // Use Vec instead of array
    }

    println!("📊 Memory Layout Comparison:");
    println!("   Unoptimized enum size: {} bytes",
             std::mem::size_of::<UnoptimizedOrder>());
    println!("   Optimized enum size: {} bytes",
             std::mem::size_of::<OptimizedOrder>());

    let size_reduction = std::mem::size_of::<UnoptimizedOrder>() as f64 /
                        std::mem::size_of::<OptimizedOrder>() as f64;
    println!("   Size reduction: {:.1}x smaller", size_reduction);

    // Performance test with vectors
    println!("\n⚡ Performance Impact Test:");

    let test_size = 10_000;

    // Create test data
    let unoptimized_orders: Vec<UnoptimizedOrder> = (0..test_size)
        .map(|i| {
            if i % 100 == 0 {
                // 1% large orders
                UnoptimizedOrder::CateringOrder(
                    std::array::from_fn(|j| format!("Item {}", j))
                )
            } else {
                // 99% simple orders
                UnoptimizedOrder::SimpleItem(format!("Order {}", i))
            }
        })
        .collect();

    let optimized_orders: Vec<OptimizedOrder> = (0..test_size)
        .map(|i| {
            if i % 100 == 0 {
                // 1% large orders (boxed)
                OptimizedOrder::CateringOrder(
                    Box::new(std::array::from_fn(|j| format!("Item {}", j)))
                )
            } else {
                // 99% simple orders
                OptimizedOrder::SimpleItem(format!("Order {}", i))
            }
        })
        .collect();

    // Memory usage comparison
    let unoptimized_memory = unoptimized_orders.len() * std::mem::size_of::<UnoptimizedOrder>();
    let optimized_memory = optimized_orders.len() * std::mem::size_of::<OptimizedOrder>();

    println!("   Unoptimized vector memory: {} MB", unoptimized_memory / 1_048_576);
    println!("   Optimized vector memory: {} MB", optimized_memory / 1_048_576);
    println!("   Memory savings: {} MB", (unoptimized_memory - optimized_memory) / 1_048_576);

    // Iteration performance test
    let start = Instant::now();
    let mut unoptimized_count = 0;
    for order in &unoptimized_orders {
        match order {
            UnoptimizedOrder::SimpleItem(_) => unoptimized_count += 1,
            UnoptimizedOrder::CateringOrder(_) => unoptimized_count += 100,
            _ => unoptimized_count += 10,
        }
    }
    let unoptimized_time = start.elapsed();

    let start = Instant::now();
    let mut optimized_count = 0;
    for order in &optimized_orders {
        match order {
            OptimizedOrder::SimpleItem(_) => optimized_count += 1,
            OptimizedOrder::CateringOrder(_) => optimized_count += 100,
            _ => optimized_count += 10,
        }
    }
    let optimized_time = start.elapsed();

    println!("\n🏃‍♂️ Iteration Performance:");
    println!("   Unoptimized iteration: {:?}", unoptimized_time);
    println!("   Optimized iteration: {:?}", optimized_time);

    if unoptimized_time > optimized_time {
        let speedup = unoptimized_time.as_nanos() as f64 / optimized_time.as_nanos() as f64;
        println!("   🚀 Optimized version is {:.2}x faster", speedup);
    }

    println!("\n💡 Boxing Guidelines:");
    println!("   ✅ Box variants larger than ~100 bytes");
    println!("   ✅ Box rarely used large variants");
    println!("   ✅ Keep frequently accessed data unboxed");
    println!("   ⚠️  Consider access patterns when deciding to box");
}

fn main() {
    demonstrate_boxing_optimization();
}
```


### 2. Data Structure Optimization

**Optimizing the internal structure of enum variants:**

```rust
fn demonstrate_data_structure_optimization() {
    println!("🔧 Data Structure Optimization Strategies");
    println!("{:=<50}", "");

    // Inefficient data structures
    #[derive(Debug, Clone)]
    enum InefficientMenuData {
        MenuItem {
            id: u64,                    // 8 bytes
            name: String,               // 24 bytes
            description: String,        // 24 bytes
            price: f64,                 // 8 bytes
            ingredients: Vec<String>,   // 24 bytes
            allergens: Vec<String>,     // 24 bytes
            is_vegan: bool,            // 1 byte
            is_gluten_free: bool,      // 1 byte
            is_available: bool,        // 1 byte
            // Total: ~115 bytes + heap allocations
        },
        DailySpecial {
            base_item: String,          // 24 bytes
            modifications: Vec<String>, // 24 bytes
            discount_percent: f32,      // 4 bytes
            available_quantity: u32,    // 4 bytes
            // Total: ~56 bytes + heap allocations
        },
    }

    // Optimized data structures
    #[derive(Debug, Clone)]
    enum OptimizedMenuData {
        MenuItem {
            id: u32,                    // 4 bytes (sufficient for menu items)
            name: String,               // 24 bytes
            description: String,        // 24 bytes
            price_cents: u32,          // 4 bytes (price in cents, no float)
            ingredients: Box<[String]>, // 8 bytes pointer (boxed slice)
            allergens: u8,             // 1 byte (bitflags for common allergens)
            dietary_flags: u8,         // 1 byte (vegan, gluten-free, etc.)
            available_quantity: u16,    // 2 bytes (sufficient for quantities)
            // Total: ~68 bytes + one heap allocation
        },
        DailySpecial {
            base_item_id: u32,         // 4 bytes (reference to MenuItem)
            modifications: u8,          // 1 byte (bitflags for common mods)
            discount_percent: u8,       // 1 byte (0-100 percent)
            available_quantity: u16,    // 2 bytes
            // Total: 8 bytes (no heap allocations!)
        },
    }

    println!("📊 Structure Size Comparison:");
    println!("   Inefficient enum size: {} bytes",
             std::mem::size_of::<InefficientMenuData>());
    println!("   Optimized enum size: {} bytes",
             std::mem::size_of::<OptimizedMenuData>());

    // Demonstrate bitflag optimization
    println!("\n🏳️ Bitflag Optimization Example:");

    const ALLERGEN_NUTS: u8 = 1 << 0;      // Bit 0
    const ALLERGEN_DAIRY: u8 = 1 << 1;     // Bit 1
    const ALLERGEN_SOY: u8 = 1 << 2;       // Bit 2
    const ALLERGEN_GLUTEN: u8 = 1 << 3;    // Bit 3

    const DIETARY_VEGAN: u8 = 1 << 0;      // Bit 0
    const DIETARY_VEGETARIAN: u8 = 1 << 1; // Bit 1
    const DIETARY_GLUTEN_FREE: u8 = 1 << 2; // Bit 2
    const DIETARY_KETO: u8 = 1 << 3;       // Bit 3

    // Example optimized menu item
    let optimized_item = OptimizedMenuData::MenuItem {
        id: 1001,
        name: "Quinoa Power Bowl".to_string(),
        description: "Nutritious quinoa with fresh vegetables".to_string(),
        price_cents: 1599, // $15.99
        ingredients: vec!["quinoa".to_string(), "vegetables".to_string()].into_boxed_slice(),
        allergens: ALLERGEN_NUTS, // Only nuts
        dietary_flags: DIETARY_VEGAN | DIETARY_GLUTEN_FREE, // Vegan and gluten-free
        available_quantity: 25,
    };

    // Function to check dietary restrictions efficiently
    fn is_vegan(item: &OptimizedMenuData) -> bool {
        if let OptimizedMenuData::MenuItem { dietary_flags, .. } = item {
            dietary_flags & DIETARY_VEGAN != 0
        } else {
            false
        }
    }

    fn has_allergen(item: &OptimizedMenuData, allergen: u8) -> bool {
        if let OptimizedMenuData::MenuItem { allergens, .. } = item {
            allergens & allergen != 0
        } else {
            false
        }
    }

    println!("   Item is vegan: {}", is_vegan(&optimized_item));
    println!("   Item has nuts: {}", has_allergen(&optimized_item, ALLERGEN_NUTS));
    println!("   Item has dairy: {}", has_allergen(&optimized_item, ALLERGEN_DAIRY));

    // Demonstrate string interning for repeated data
    println!("\n📝 String Optimization Strategies:");

    use std::collections::HashMap;

    struct StringInterner {
        strings: Vec<String>,
        indices: HashMap<String, u16>,
    }

    impl StringInterner {
        fn new() -> Self {
            StringInterner {
                strings: Vec::new(),
                indices: HashMap::new(),
            }
        }

        fn intern(&mut self, s: &str) -> u16 {
            if let Some(&index) = self.indices.get(s) {
                index
            } else {
                let index = self.strings.len() as u16;
                self.strings.push(s.to_string());
                self.indices.insert(s.to_string(), index);
                index
            }
        }

        fn get(&self, index: u16) -> Option<&str> {
            self.strings.get(index as usize).map(|s| s.as_str())
        }
    }

    #[derive(Debug)]
    enum StringOptimizedMenu {
        MenuItem {
            name_id: u16,        // Index into string table
            description_id: u16, // Index into string table
            price_cents: u32,
            flags: u16,
        }
    }

    println!("   String-optimized enum size: {} bytes",
             std::mem::size_of::<StringOptimizedMenu>());
    println!("   💡 Share common strings between menu items");
    println!("   💡 Use indices instead of full strings for repeated data");

    // Performance implications
    println!("\n⚡ Performance Implications:");
    println!("   ✅ Smaller enums = better cache performance");
    println!("   ✅ Fewer heap allocations = faster operations");
    println!("   ✅ Bitflags = faster boolean operations");
    println!("   ✅ Integer operations = faster than string comparisons");
    println!("   ⚠️  String interning = upfront cost, long-term savings");
}

fn main() {
    demonstrate_data_structure_optimization();
}
```


### 3. Access Pattern Optimization

**Optimizing for how data is actually used:**

```rust
use std::time::Instant;

fn demonstrate_access_pattern_optimization() {
    println!("🎯 Access Pattern Optimization");
    println!("{:=<40}", "");

    // Analyze different access patterns
    #[derive(Debug, Clone)]
    enum RestaurantEvent {
        OrderReceived { id: u32, items: Vec<String>, timestamp: u64 },
        OrderPreparing { id: u32, chef: String, start_time: u64 },
        OrderReady { id: u32, prep_time_minutes: u16 },
        OrderServed { id: u32, server: String, table: u16 },
        PaymentProcessed { id: u32, amount_cents: u32, method: String },
        // Less common events
        CustomerComplaint { id: u32, issue: String, severity: u8 },
        StaffBreak { staff_id: u32, duration_minutes: u16 },
        InventoryUpdate { item: String, quantity_change: i32 },
    }

    // Optimized for common access patterns
    #[derive(Debug, Clone)]
    enum OptimizedRestaurantEvent {
        // Most common events first (better branch prediction)
        OrderReceived(OrderData),
        OrderStatusUpdate(StatusUpdate),
        PaymentProcessed(PaymentData),

        // Less common events grouped
        MiscEvent(Box<MiscEventData>),
    }

    #[derive(Debug, Clone)]
    struct OrderData {
        id: u32,
        items: Box<[String]>, // Boxed slice for efficiency
        timestamp: u64,
    }

    #[derive(Debug, Clone)]
    struct StatusUpdate {
        id: u32,
        status: OrderStatus,
        timestamp: u64,
    }

    #[derive(Debug, Clone)]
    enum OrderStatus {
        Preparing { chef_id: u16, start_time: u64 },
        Ready { prep_time_minutes: u16 },
        Served { server_id: u16, table: u16 },
    }

    #[derive(Debug, Clone)]
    struct PaymentData {
        id: u32,
        amount_cents: u32,
        method: PaymentMethod,
    }

    #[derive(Debug, Clone)]
    enum PaymentMethod {
        Cash,
        Card,
        DigitalWallet,
    }

    #[derive(Debug, Clone)]
    enum MiscEventData {
        CustomerComplaint { id: u32, issue: String, severity: u8 },
        StaffBreak { staff_id: u32, duration_minutes: u16 },
        InventoryUpdate { item: String, quantity_change: i32 },
    }

    println!("📊 Event Structure Comparison:");
    println!("   Original enum size: {} bytes", std::mem::size_of::<RestaurantEvent>());
    println!("   Optimized enum size: {} bytes", std::mem::size_of::<OptimizedRestaurantEvent>());

    // Simulate realistic access patterns
    println!("\n🔄 Simulating Realistic Access Patterns:");

    let mut events = Vec::new();

    // Generate events based on realistic frequency
    for i in 0..10_000 {
        let event = match i % 100 {
            0..=60 => RestaurantEvent::OrderReceived {
                id: i,
                items: vec![format!("Item {}", i % 10)],
                timestamp: i as u64,
            },
            61..=80 => RestaurantEvent::OrderPreparing {
                id: i,
                chef: format!("Chef {}", i % 3),
                start_time: i as u64,
            },
            81..=90 => RestaurantEvent::OrderReady {
                id: i,
                prep_time_minutes: (15 + i % 20) as u16,
            },
            91..=95 => RestaurantEvent::PaymentProcessed {
                id: i,
                amount_cents: 1500 + (i % 1000) as u32,
                method: format!("Card"),
            },
            96..=97 => RestaurantEvent::CustomerComplaint {
                id: i,
                issue: "Food was cold".to_string(),
                severity: 3,
            },
            98 => RestaurantEvent::StaffBreak {
                staff_id: (i % 10) as u32,
                duration_minutes: 15,
            },
            _ => RestaurantEvent::InventoryUpdate {
                item: format!("Ingredient {}", i % 5),
                quantity_change: -5,
            },
        };
        events.push(event);
    }

    // Test access pattern performance
    println!("\n⏱️ Access Pattern Performance Test:");

    // Pattern 1: Count order events (most common operation)
    let start = Instant::now();
    let mut order_count = 0;
    for event in &events {
        match event {
            RestaurantEvent::OrderReceived { .. } => order_count += 1,
            RestaurantEvent::OrderPreparing { .. } => order_count += 1,
            RestaurantEvent::OrderReady { .. } => order_count += 1,
            RestaurantEvent::OrderServed { .. } => order_count += 1,
            _ => {}
        }
    }
    let order_counting_time = start.elapsed();

    // Pattern 2: Find payment events (less common operation)
    let start = Instant::now();
    let mut payment_total = 0u64;
    for event in &events {
        if let RestaurantEvent::PaymentProcessed { amount_cents, .. } = event {
            payment_total += *amount_cents as u64;
        }
    }
    let payment_processing_time = start.elapsed();

    // Pattern 3: Handle rare events (least common operation)
    let start = Instant::now();
    let mut complaint_count = 0;
    for event in &events {
        match event {
            RestaurantEvent::CustomerComplaint { .. } => complaint_count += 1,
            RestaurantEvent::StaffBreak { .. } => {},
            RestaurantEvent::InventoryUpdate { .. } => {},
            _ => {}
        }
    }
    let rare_events_time = start.elapsed();

    println!("   Order counting (80% of operations): {:?}", order_counting_time);
    println!("   Payment processing (15% of operations): {:?}", payment_processing_time);
    println!("   Rare event handling (5% of operations): {:?}", rare_events_time);

    // Cache-friendly grouping demonstration
    println!("\n🏪 Cache-Friendly Event Grouping:");

    #[derive(Debug)]
    struct EventBatch {
        orders: Vec<OrderData>,
        payments: Vec<PaymentData>,
        misc: Vec<MiscEventData>,
    }

    impl EventBatch {
        fn new() -> Self {
            EventBatch {
                orders: Vec::new(),
                payments: Vec::new(),
                misc: Vec::new(),
            }
        }

        fn process_orders(&self) -> usize {
            // Process all orders together (better cache locality)
            self.orders.len()
        }

        fn process_payments(&self) -> u64 {
            // Process all payments together
            self.payments.iter().map(|p| p.amount_cents as u64).sum()
        }
    }

    println!("   💡 Group similar operations for better cache performance");
    println!("   💡 Process events in batches when possible");
    println!("   💡 Separate hot and cold data paths");

    println!("\n🎯 Access Pattern Optimization Guidelines:");
    println!("   ✅ Put most frequently accessed variants first");
    println!("   ✅ Group related data together");
    println!("   ✅ Separate hot and cold code paths");
    println!("   ✅ Consider batch processing for similar operations");
    println!("   ✅ Profile actual usage patterns in production");
}

fn main() {
    demonstrate_access_pattern_optimization();
}
```


## Performance Profiling and Benchmarking

### Measuring Real-World Performance

```rust
use std::time::{Instant, Duration};
use std::collections::HashMap;

fn demonstrate_performance_profiling() {
    println!("📊 Performance Profiling and Benchmarking");
    println!("{:=<50}", "");

    #[derive(Debug, Clone)]
    enum ProfiledData {
        SmallData(u32),
        MediumData(String),
        LargeData(Vec<u8>),
        HugeData(Box<[u8; 4096]>),
    }

    struct PerformanceProfiler {
        operation_times: HashMap<String, Vec<Duration>>,
    }

    impl PerformanceProfiler {
        fn new() -> Self {
            PerformanceProfiler {
                operation_times: HashMap::new(),
            }
        }

        fn time_operation<F, R>(&mut self, name: &str, operation: F) -> R
        where
            F: FnOnce() -> R,
        {
            let start = Instant::now();
            let result = operation();
            let duration = start.elapsed();

            self.operation_times
                .entry(name.to_string())
                .or_insert_with(Vec::new)
                .push(duration);

            result
        }

        fn report_statistics(&self) {
            println!("\n📈 Performance Statistics:");

            for (operation, times) in &self.operation_times {
                let total_time: Duration = times.iter().sum();
                let avg_time = total_time / times.len() as u32;
                let min_time = times.iter().min().unwrap();
                let max_time = times.iter().max().unwrap();

                println!("   Operation: {}", operation);
                println!("     Runs: {}", times.len());
                println!("     Average: {:?}", avg_time);
                println!("     Min: {:?}", min_time);
                println!("     Max: {:?}", max_time);
                println!("     Total: {:?}", total_time);
                println!();
            }
        }
    }

    let mut profiler = PerformanceProfiler::new();

    // Create test data with different characteristics
    let test_data: Vec<ProfiledData> = (0..50_000)
        .map(|i| match i % 4 {
            0 => ProfiledData::SmallData(i as u32),
            1 => ProfiledData::MediumData(format!("Item {}", i)),
            2 => ProfiledData::LargeData(vec![i as u8; 100]),
            _ => ProfiledData::HugeData(Box::new([i as u8; 4096])),
        })
        .collect();

    println!("🧪 Running Performance Tests:");
    println!("   Test data size: {} items", test_data.len());
    println!("   Data distribution: 25% each of 4 variant types");

    // Benchmark different operations
    for run in 0..10 {
        println!("   Run {} of 10...", run + 1);

        // Test 1: Full iteration and pattern matching
        profiler.time_operation("full_iteration", || {
            let mut count = 0u64;
            for item in &test_data {
                match item {
                    ProfiledData::SmallData(n) => count += *n as u64,
                    ProfiledData::MediumData(s) => count += s.len() as u64,
                    ProfiledData::LargeData(v) => count += v.len() as u64,
                    ProfiledData::HugeData(_) => count += 4096,
                }
            }
            count
        });

        // Test 2: Filtering specific variants
        profiler.time_operation("filter_small_data", || {
            test_data
                .iter()
                .filter(|item| matches!(item, ProfiledData::SmallData(_)))
                .count()
        });

        // Test 3: Extracting and processing specific data
        profiler.time_operation("extract_strings", || {
            test_data
                .iter()
                .filter_map(|item| {
                    if let ProfiledData::MediumData(s) = item {
                        Some(s.len())
                    } else {
                        None
                    }
                })
                .sum::<usize>()
        });

        // Test 4: Memory-intensive operations
        profiler.time_operation("process_large_data", || {
            let mut total = 0usize;
            for item in &test_data {
                match item {
                    ProfiledData::LargeData(v) => {
                        total += v.iter().map(|&b| b as usize).sum::<usize>();
                    }
                    ProfiledData::HugeData(arr) => {
                        total += arr.iter().map(|&b| b as usize).sum::<usize>();
                    }
                    _ => {}
                }
            }
            total
        });

        // Test 5: Vector operations
        profiler.time_operation("vector_push_pop", || {
            let mut temp_vec = Vec::new();

            for item in test_data.iter().take(1000) {
                temp_vec.push(item.clone());
            }

            while temp_vec.len() > 500 {
                temp_vec.pop();
            }

            temp_vec.len()
        });
    }

    profiler.report_statistics();

    // Memory usage analysis
    println!("💾 Memory Usage Analysis:");

    let enum_size = std::mem::size_of::<ProfiledData>();
    let vector_overhead = std::mem::size_of::<Vec<ProfiledData>>();
    let total_vector_memory = test_data.len() * enum_size + vector_overhead;

    println!("   Enum size: {} bytes", enum_size);
    println!("   Vector overhead: {} bytes", vector_overhead);
    println!("   Vector memory usage: {} KB", total_vector_memory / 1024);

    // Estimate heap allocations
    let string_count = test_data.iter()
        .filter(|item| matches!(item, ProfiledData::MediumData(_)))
        .count();
    let large_data_count = test_data.iter()
        .filter(|item| matches!(item, ProfiledData::LargeData(_)))
        .count();
    let huge_data_count = test_data.iter()
        .filter(|item| matches!(item, ProfiledData::HugeData(_)))
        .count();

    println!("\n🏗️ Heap Allocation Estimates:");
    println!("   String allocations: ~{} items", string_count);
    println!("   Large data allocations: ~{} items", large_data_count);
    println!("   Huge data (boxed) allocations: ~{} items", huge_data_count);

    let estimated_heap_usage =
        string_count * 20 +      // Estimated string heap usage
        large_data_count * 100 + // Large data heap usage
        huge_data_count * 4096;  // Huge data heap usage

    println!("   Estimated heap usage: {} KB", estimated_heap_usage / 1024);
    println!("   Total memory (stack + heap): {} KB",
             (total_vector_memory + estimated_heap_usage) / 1024);
}

// Benchmark helper for more detailed analysis
struct DetailedBenchmark {
    name: String,
    measurements: Vec<Duration>,
}

impl DetailedBenchmark {
    fn new(name: &str) -> Self {
        DetailedBenchmark {
            name: name.to_string(),
            measurements: Vec::new(),
        }
    }

    fn measure<F, R>(&mut self, iterations: usize, operation: F) -> Vec<R>
    where
        F: Fn() -> R,
    {
        let mut results = Vec::new();

        for _ in 0..iterations {
            let start = Instant::now();
            let result = operation();
            let duration = start.elapsed();

            self.measurements.push(duration);
            results.push(result);
        }

        results
    }

    fn report(&self) {
        if self.measurements.is_empty() {
            return;
        }

        let mut sorted_measurements = self.measurements.clone();
        sorted_measurements.sort();

        let total: Duration = self.measurements.iter().sum();
        let average = total / self.measurements.len() as u32;
        let median = sorted_measurements[sorted_measurements.len() / 2];
        let p95_index = (sorted_measurements.len() as f64 * 0.95) as usize;
        let p95 = sorted_measurements[p95_index.min(sorted_measurements.len() - 1)];

        println!("📊 Detailed Benchmark: {}", self.name);
        println!("   Iterations: {}", self.measurements.len());
        println!("   Average: {:?}", average);
        println!("   Median: {:?}", median);
        println!("   95th percentile: {:?}", p95);
        println!("   Min: {:?}", sorted_measurements[0]);
        println!("   Max: {:?}", sorted_measurements[sorted_measurements.len() - 1]);
    }
}

fn demonstrate_detailed_benchmarking() {
    println!("\n🎯 Detailed Benchmarking Example:");

    let test_data: Vec<ProfiledData> = (0..1000)
        .map(|i| ProfiledData::SmallData(i as u32))
        .collect();

    let mut benchmark = DetailedBenchmark::new("Small Data Processing");

    benchmark.measure(100, || {
        test_data.iter()
            .filter_map(|item| {
                if let ProfiledData::SmallData(n) = item {
                    Some(*n)
                } else {
                    None
                }
            })
            .sum::<u32>()
    });

    benchmark.report();
}

fn main() {
    demonstrate_performance_profiling();
    demonstrate_detailed_benchmarking();
}
```


## Best Practices and Performance Guidelines

### Comprehensive Performance Optimization Checklist

```rust
fn demonstrate_comprehensive_best_practices() {
    println!("✅ Comprehensive Performance Best Practices");
    println!("{:=<50}", "");

    // 1. Enum Design Guidelines
    println!("🎯 Enum Design Guidelines:");
    println!("   ✅ Keep frequently used variants small");
    println!("   ✅ Box large or rarely used variants");
    println!("   ✅ Use bitflags for boolean combinations");
    println!("   ✅ Order variants by frequency of use");
    println!("   ✅ Consider alignment and padding");

    #[derive(Debug)]
    enum WellDesignedEnum {
        // Most common variants first
        Common(u32),                    // 4 bytes
        Frequent(String),               // 24 bytes

        // Less common variants
        Uncommon(Box<LargeData>),       // 8 bytes pointer
        Rare(Box<HugeData>),           // 8 bytes pointer
    }

    #[derive(Debug)]
    struct LargeData {
        data: Vec<u8>,
        metadata: String,
    }

    #[derive(Debug)]
    struct HugeData {
        massive_array: [u8; 2048],
        extra_info: String,
    }

    println!("   Well-designed enum size: {} bytes",
             std::mem::size_of::<WellDesignedEnum>());

    // 2. Memory Management Best Practices
    println!("\n💾 Memory Management Best Practices:");
    println!("   ✅ Pre-allocate vectors when size is known");
    println!("   ✅ Use Box<[T]> instead of Vec<T> for fixed data");
    println!("   ✅ Consider string interning for repeated strings");
    println!("   ✅ Use appropriate integer sizes (u8, u16, u32)");
    println!("   ✅ Pack related data in structs");

    // Example of good memory management
    #[derive(Debug)]
    struct MemoryEfficientOrder {
        id: u32,                        // 4 bytes
        items: Box<[u16]>,             // 8 bytes pointer to fixed array
        flags: u8,                      // 1 byte for boolean flags
        priority: u8,                   // 1 byte priority level
        timestamp: u64,                 // 8 bytes
        // Total: 24 bytes + heap allocation for items
    }

    // 3. Access Pattern Optimization
    println!("\n🎯 Access Pattern Optimization:");
    println!("   ✅ Group similar operations together");
    println!("   ✅ Minimize random memory access");
    println!("   ✅ Use iterators for cache-friendly traversal");
    println!("   ✅ Consider SIMD operations for bulk processing");
    println!("   ✅ Profile hot paths in production code");

    // Example of cache-friendly processing
    fn process_orders_efficiently(orders: &[MemoryEfficientOrder]) -> (u64, u32) {
        // Process all data in a single pass (better cache usage)
        let mut total_items = 0u64;
        let mut high_priority_count = 0u32;

        for order in orders {
            total_items += order.items.len() as u64;
            if order.priority >= 8 {
                high_priority_count += 1;
            }
        }

        (total_items, high_priority_count)
    }

    // 4. Performance Monitoring
    println!("\n📊 Performance Monitoring:");
    println!("   ✅ Benchmark critical operations regularly");
    println!("   ✅ Monitor memory allocation patterns");
    println!("   ✅ Track performance regressions");
    println!("   ✅ Use profiling tools (perf, flamegraph)");
    println!("   ✅ Set up automated performance tests");

    // Simple performance monitor
    struct PerformanceMonitor {
        operation_counts: std::collections::HashMap<String, usize>,
        total_time: std::time::Duration,
    }

    impl PerformanceMonitor {
        fn new() -> Self {
            PerformanceMonitor {
                operation_counts: std::collections::HashMap::new(),
                total_time: std::time::Duration::new(0, 0),
            }
        }

        fn record_operation(&mut self, operation: &str) {
            *self.operation_counts.entry(operation.to_string()).or_insert(0) += 1;
        }

        fn get_stats(&self) -> String {
            format!("Operations: {:?}, Total time: {:?}",
                   self.operation_counts, self.total_time)
        }
    }

    // 5. Common Performance Antipatterns to Avoid
    println!("\n❌ Performance Antipatterns to Avoid:");
    println!("   ❌ Large inline enum variants");
    println!("   ❌ Unnecessary string allocations in hot paths");
    println!("   ❌ Deep enum nesting without boxing");
    println!("   ❌ Ignoring cache locality in data structures");
    println!("   ❌ Using Vec<T> when Box<[T]> would suffice");
    println!("   ❌ Not profiling before optimizing");

    // Example of what NOT to do
    #[derive(Debug)]
    enum AntipatternEnum {
        Small(u8),
        TooLarge([u8; 4096]),           // ❌ Makes entire enum huge
        DeepNesting(Vec<Vec<String>>),   // ❌ Complex nested allocation
        WastedSpace {                    // ❌ Poor field ordering
            flag: bool,                  // 1 byte
            big_number: u64,            // 8 bytes (padding before this)
            another_flag: bool,          // 1 byte
            // Padding at end
        },
    }

    println!("   Antipattern enum size: {} bytes",
             std::mem::size_of::<AntipatternEnum>());

    // 6. Platform-Specific Considerations
    println!("\n🖥️ Platform-Specific Considerations:");
    println!("   ✅ Consider target architecture (32-bit vs 64-bit)");
    println!("   ✅ Account for different cache line sizes");
    println!("   ✅ Test on representative hardware");
    println!("   ✅ Be aware of NUMA topology effects");
    println!("   ✅ Consider mobile device constraints");

    println!("\n🎓 Performance Optimization Workflow:");
    println!("   1. 📊 Profile to identify bottlenecks");
    println!("   2. 🎯 Focus on hot paths first");
    println!("   3. 🔧 Apply targeted optimizations");
    println!("   4. 📈 Measure improvement");
    println!("   5. 🔄 Repeat until satisfied");
    println!("   6. 🛡️ Add performance regression tests");
}

// Performance testing framework example
struct PerformanceTestSuite {
    tests: Vec<Box<dyn Fn() -> std::time::Duration>>,
    test_names: Vec<String>,
}

impl PerformanceTestSuite {
    fn new() -> Self {
        PerformanceTestSuite {
            tests: Vec::new(),
            test_names: Vec::new(),
        }
    }

    fn add_test<F>(&mut self, name: &str, test: F)
    where
        F: Fn() -> std::time::Duration + 'static
    {
        self.test_names.push(name.to_string());
        self.tests.push(Box::new(test));
    }

    fn run_all(&self) {
        println!("\n🧪 Running Performance Test Suite:");

        for (name, test) in self.test_names.iter().zip(self.tests.iter()) {
            let duration = test();
            println!("   {}: {:?}", name, duration);
        }
    }
}

fn demonstrate_performance_testing_framework() {
    let mut suite = PerformanceTestSuite::new();

    // Add sample tests
    suite.add_test("vector_creation", || {
        let start = std::time::Instant::now();
        let _vec: Vec<u32> = (0..10000).collect();
        start.elapsed()
    });

    suite.add_test("enum_processing", || {
        let data: Vec<WellDesignedEnum> = (0..1000)
            .map(|i| WellDesignedEnum::Common(i))
            .collect();

        let start = std::time::Instant::now();
        let _count = data.iter().filter(|item| {
            matches!(item, WellDesignedEnum::Common(_))
        }).count();
        start.elapsed()
    });

    suite.run_all();
}

fn main() {
    demonstrate_comprehensive_best_practices();
    demonstrate_performance_testing_framework();
}
```


## Summary and Key Takeaways

### **Mental Model: The High-Performance Restaurant Operations**

Remember our high-performance restaurant analogy:

- ⚡ **Memory layout** = **Kitchen organization** - Efficient space usage and workflow
- 🏪 **Cache performance** = **Station proximity** - Keep related tools and ingredients close
- 📦 **Boxing large variants** = **Storage optimization** - Put bulky items in dedicated storage
- 🎯 **Access patterns** = **Order processing flow** - Optimize for common operations
- 📊 **Profiling** = **Performance monitoring** - Track and improve efficiency metrics


### **Essential Performance Principles**

1. **Memory is precious** - Minimize enum size to improve cache performance
2. **Access patterns matter** - Optimize for how data is actually used
3. **Profile before optimizing** - Measure to identify real bottlenecks
4. **Box large variants** - Keep enum size manageable with strategic boxing
5. **Consider the whole system** - Performance is about the entire data flow

### **Performance Optimization Quick Reference**

| **Optimization** | **Technique** | **Impact** | **When to Use** |
| :-- | :-- | :-- | :-- |
| **Boxing** | `Box<LargeData>` | High memory savings | Variants >100 bytes |
| **Bitflags** | `u8` instead of `Vec<bool>` | Medium space/speed | Multiple booleans |
| **String interning** | Index instead of `String` | Medium memory savings | Repeated strings |
| **Fixed arrays** | `[T; N]` instead of `Vec<T>` | Small memory savings | Known-size data |
| **Struct packing** | Reorder fields | Small memory savings | Memory-critical code |

### **Performance Measurement Tools**

```rust
// Quick performance measurement
use std::time::Instant;

fn measure_operation<F, R>(name: &str, operation: F) -> R
where F: FnOnce() -> R
{
    let start = Instant::now();
    let result = operation();
    let duration = start.elapsed();
    println!("{}: {:?}", name, duration);
    result
}

// Usage example
let result = measure_operation("enum processing", || {
    // Your operation here
    42
});
```


### **Best Practices Checklist**

**✅ DO:**

- Profile your actual usage patterns before optimizing
- Box enum variants larger than ~100 bytes
- Order enum variants by frequency of use
- Use appropriate integer sizes (u8, u16, u32)
- Consider cache locality in data structure design
- Set up automated performance regression tests

**❌ DON'T:**

- Optimize prematurely without measurements
- Ignore the cost of indirection when boxing
- Use large inline arrays in enum variants
- Forget about alignment and padding issues
- Assume micro-optimizations will have macro impact
- Sacrifice code clarity for minimal performance gains


### **Real-World Performance Applications**

**Performance-optimized vectors of enums are crucial for:**

- 🎮 **Game engines** - Handling thousands of entities efficiently
- 📊 **Data processing** - Processing large datasets with mixed types
- 🌐 **Web servers** - Managing request/response data efficiently
- 📱 **Mobile apps** - Minimizing memory usage on constrained devices
- 🚀 **High-frequency systems** - Trading, real-time analytics
- 🔬 **Scientific computing** - Processing heterogeneous experimental data


### **The Professional Performance Advantage**

**High-performance vectors of enums are like having a world-class restaurant that serves thousands of customers efficiently** while maintaining quality:

- ⚡ **Predictable performance** - Know exactly how your code will behave under load
- 💾 **Memory efficiency** - Use minimal resources for maximum functionality
- 🎯 **Targeted optimization** - Focus effort where it has the most impact
- 📊 **Measurable improvements** - Track and verify performance gains
- 🔄 **Sustainable performance** - Maintain efficiency as code evolves

**Mastering performance considerations for vectors of enums transforms you from a functional programmer to a systems-level expert** who can build Rust applications that scale efficiently and compete with the fastest systems in any domain. Just as a master chef can serve a complex, diverse menu to hundreds of customers without compromising quality or speed, a performance-conscious Rust programmer can handle complex, heterogeneous data at scale while maintaining the safety and elegance that makes Rust exceptional!
