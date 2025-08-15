+++
title = "Thread Safety Concepts"
description = "A guide to key thread safety concepts, data races, and how Rust prevents them at compile time."
date = "2025-08-13"
weight = 3
+++

# Thread Safety Concepts in Rust: Comprehensive Documentation for Beginners

Understanding thread safety in Rust is like learning to **design comprehensive coordination systems for your professional restaurant chain** - you need systems that allow multiple teams (kitchen staff, servers, managers) to work simultaneously on shared resources (ingredients, equipment, customer orders) without conflicts, mix-ups, or dangerous situations. Just as a professional restaurant manager must ensure that multiple chefs can safely use the same kitchen equipment without interfering with each other's work, Rust's thread safety system ensures that multiple threads can safely access shared data without causing data races, corruption, or unpredictable behavior.

## The Professional Restaurant Coordination System Analogy 🏪

### Imagine You're Designing Safe Multi-Team Operations for Your Restaurant Chain

**The Problem without Proper Coordination:**

```rust
// ❌ Dangerous approach - like having no coordination between teams
// Multiple chefs try to use the same equipment simultaneously
// Multiple servers try to modify the same order at once
// Results in chaos, mistakes, and potentially dangerous situations
```

**The Power of Thread Safety - Professional Team Coordination:**

```rust
// ✅ Professional approach - systematic coordination and safety protocols
use std::sync::{Arc, Mutex};
use std::thread;

fn professional_coordination_demo() {
    // Shared equipment that needs coordination (like shared data)
    let kitchen_equipment = Arc::new(Mutex::new("Professional Oven"));
    let mut team_handles = vec![];

    // Multiple teams working safely with shared resources
    for team_id in 0..3 {
        let equipment = Arc::clone(&kitchen_equipment);
        let handle = thread::spawn(move || {
            // Each team safely accesses shared equipment
            let equipment_access = equipment.lock().unwrap();
            println!("Team {} safely using: {}", team_id, *equipment_access);
            // Equipment automatically released when done
        });
        team_handles.push(handle);
    }

    // Wait for all teams to complete their work
    for handle in team_handles {
        handle.join().unwrap();
    }
}
```

**Why Thread Safety Is Essential for Professional Operations:**

- 🛡️ **Prevents conflicts** - Multiple teams can work simultaneously without interference
- 🎯 **Guarantees consistency** - Shared resources remain in predictable states
- ⚡ **Enables efficiency** - Maximum utilization of resources without safety compromises
- 🔒 **Maintains data integrity** - Information remains accurate across all team interactions
- 📈 **Scales operations** - Systems work reliably as teams and complexity grow


## Understanding Thread Safety Fundamentals

### The Professional Team Coordination Framework

**Thread safety ensures multiple threads can safely work with shared data concurrently:**

```rust
fn demonstrate_thread_safety_fundamentals() {
    println!("🛡️ Thread Safety Fundamentals - Professional Team Coordination");
    println!("{:=<75}", "");

    use std::sync::{Arc, Mutex, RwLock};
    use std::thread;
    use std::time::Duration;

    // Thread safety is like having professional protocols for team coordination
    println!("📋 Thread Safety Core Concepts:");
    println!("   🔐 Mutual Exclusion - Only one team uses critical resources at a time");
    println!("   🤝 Safe Sharing - Multiple teams can safely share read-only information");
    println!("   📡 Message Passing - Teams coordinate through structured communication");
    println!("   ⚛️ Atomic Operations - Indivisible actions that can't be interrupted");
    println!("   🎯 Compiler Guarantees - Safety rules enforced before operations begin");

    // Example 1: Basic Thread Safety with Mutex
    println!("\n1️⃣ Basic Thread Safety - Shared Resource Protection:");

    #[derive(Debug)]
    struct SharedKitchenData {
        orders_completed: u32,
        current_temperature: i32,
        active_recipes: Vec<String>,
    }

    impl SharedKitchenData {
        fn new() -> Self {
            SharedKitchenData {
                orders_completed: 0,
                current_temperature: 350,
                active_recipes: Vec::new(),
            }
        }

        fn complete_order(&mut self, recipe: String) {
            self.orders_completed += 1;
            self.active_recipes.push(recipe);
            println!("     ✅ Order completed: {} total orders", self.orders_completed);
        }

        fn adjust_temperature(&mut self, new_temp: i32) {
            self.current_temperature = new_temp;
            println!("     🌡️ Temperature adjusted to: {}°F", self.current_temperature);
        }
    }

    // Shared kitchen data protected by Mutex
    let kitchen_data = Arc::new(Mutex::new(SharedKitchenData::new()));
    let mut chef_handles = vec![];

    // Multiple chefs working with shared kitchen data
    for chef_id in 0..3 {
        let data = Arc::clone(&kitchen_data);
        let handle = thread::spawn(move || {
            // Each chef safely accesses shared data
            let mut kitchen = data.lock().unwrap();

            match chef_id {
                0 => {
                    kitchen.complete_order("Pizza Margherita".to_string());
                    kitchen.adjust_temperature(375);
                }
                1 => {
                    kitchen.complete_order("Caesar Salad".to_string());
                }
                2 => {
                    kitchen.complete_order("Pasta Carbonara".to_string());
                    kitchen.adjust_temperature(325);
                }
                _ => {}
            }

            println!("     👨‍🍳 Chef {} finished working", chef_id);
            // Mutex automatically unlocked when kitchen goes out of scope
        });
        chef_handles.push(handle);
    }

    // Wait for all chefs to complete
    for handle in chef_handles {
        handle.join().unwrap();
    }

    // Check final state
    let final_state = kitchen_data.lock().unwrap();
    println!("     📊 Final kitchen state:");
    println!("       Orders completed: {}", final_state.orders_completed);
    println!("       Final temperature: {}°F", final_state.current_temperature);
    println!("       Active recipes: {:?}", final_state.active_recipes);

    // Example 2: Read-Write Lock for Efficient Sharing
    println!("\n2️⃣ Read-Write Lock - Efficient Information Sharing:");

    #[derive(Debug, Clone)]
    struct MenuInformation {
        daily_specials: Vec<String>,
        prices: std::collections::HashMap<String, f64>,
        availability: std::collections::HashMap<String, bool>,
    }

    impl MenuInformation {
        fn new() -> Self {
            let mut prices = std::collections::HashMap::new();
            prices.insert("Pizza".to_string(), 15.99);
            prices.insert("Salad".to_string(), 8.50);
            prices.insert("Pasta".to_string(), 12.75);

            let mut availability = std::collections::HashMap::new();
            availability.insert("Pizza".to_string(), true);
            availability.insert("Salad".to_string(), true);
            availability.insert("Pasta".to_string(), false);

            MenuInformation {
                daily_specials: vec!["Grilled Salmon".to_string()],
                prices,
                availability,
            }
        }

        fn check_price(&self, item: &str) -> Option<f64> {
            self.prices.get(item).copied()
        }

        fn is_available(&self, item: &str) -> bool {
            self.availability.get(item).unwrap_or(&false).clone()
        }

        fn update_availability(&mut self, item: &str, available: bool) {
            self.availability.insert(item.to_string(), available);
        }
    }

    // Menu information that can be read by many, written by few
    let menu_info = Arc::new(RwLock::new(MenuInformation::new()));
    let mut worker_handles = vec![];

    // Multiple readers (servers checking menu)
    for server_id in 0..5 {
        let menu = Arc::clone(&menu_info);
        let handle = thread::spawn(move || {
            // Multiple servers can read simultaneously
            let menu_reader = menu.read().unwrap();

            if let Some(price) = menu_reader.check_price("Pizza") {
                println!("     👩‍💼 Server {}: Pizza costs ${:.2}", server_id, price);
            }

            let pasta_available = menu_reader.is_available("Pasta");
            println!("     👩‍💼 Server {}: Pasta available: {}", server_id, pasta_available);

            // Read lock automatically released
        });
        worker_handles.push(handle);
    }

    // One writer (manager updating availability)
    {
        let menu = Arc::clone(&menu_info);
        let handle = thread::spawn(move || {
            thread::sleep(Duration::from_millis(50)); // Let readers start first

            // Exclusive write access
            let mut menu_writer = menu.write().unwrap();
            println!("     👨‍💼 Manager: Updating pasta availability");
            menu_writer.update_availability("Pasta", true);

            // Write lock automatically released
        });
        worker_handles.push(handle);
    }

    // Wait for all workers
    for handle in worker_handles {
        handle.join().unwrap();
    }

    println!("     ✅ Read-write lock allows efficient concurrent reading with safe writing");

    // Example 3: The Send and Sync Traits
    println!("\n3️⃣ Send and Sync Traits - Team Member Capabilities:");

    println!("   🏷️ Send Trait - Team members who can move between locations:");
    println!("     ✅ Most types are Send: String, Vec<T>, i32, f64, etc.");
    println!("     ❌ Some types are !Send: Rc<T> (not thread-safe reference counting)");

    println!("   🏷️ Sync Trait - Team members who can be safely referenced by multiple teams:");
    println!("     ✅ Most types are Sync: i32, String, &T where T: Sync");
    println!("     ✅ Mutex<T> is Sync (provides safe shared access)");
    println!("     ❌ Some types are !Sync: Cell<T>, RefCell<T> (not thread-safe)");

    // Demonstrate Send constraint
    fn demonstrate_send_constraint() {
        let data = String::from("Mobile team member"); // String is Send

        let handle = thread::spawn(move || {
            println!("     🚚 Data moved to new thread: {}", data);
            // data ownership transferred to this thread
        });

        handle.join().unwrap();
        // data is no longer accessible in original thread
    }

    demonstrate_send_constraint();

    // Demonstrate Sync with Arc
    fn demonstrate_sync_with_arc() {
        let shared_data = Arc::new("Shared team resource".to_string()); // Arc<String> is Sync
        let mut handles = vec![];

        for worker_id in 0..3 {
            let data = Arc::clone(&shared_data); // Clone the Arc, not the data
            let handle = thread::spawn(move || {
                println!("     👥 Worker {} accessing shared data: {}", worker_id, data);
                // Multiple threads can safely share this data
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }
    }

    demonstrate_sync_with_arc();

    println!("\n🎯 Thread Safety Key Benefits:");
    println!("   🔐 Mutex provides exclusive access to prevent data races");
    println!("   📚 RwLock allows efficient concurrent reading with exclusive writing");
    println!("   🏷️ Send/Sync traits ensure only safe types cross thread boundaries");
    println!("   ⚡ Compiler enforces thread safety rules at compile time");
    println!("   🛡️ Zero runtime overhead for safety - all checks done during compilation");
}

fn main() {
    demonstrate_thread_safety_fundamentals();
}
```


## Send and Sync Traits - The Team Capability System

### Understanding Thread Boundary Safety

**Send and Sync traits define what types can safely cross thread boundaries:**

```rust
fn demonstrate_send_sync_traits() {
    println!("🏷️ Send and Sync Traits - Team Capability Classification System");
    println!("{:=<75}", "");

    use std::sync::{Arc, Mutex};
    use std::rc::Rc;
    use std::cell::{Cell, RefCell};
    use std::thread;

    // Send and Sync are like job classifications for team members
    println!("📋 Team Member Capability Classifications:");
    println!("   🚚 Send: Can be transferred to work at different locations");
    println!("   👥 Sync: Can be safely referenced by multiple teams simultaneously");
    println!("   🎯 Most types are both Send + Sync (safe for all team operations)");
    println!("   ⚠️ Some types have restrictions for safety reasons");

    // Example 1: Send Trait - Transferable Types
    println!("\n1️⃣ Send Trait - Transferable Team Members:");

    fn demonstrate_send_types() {
        println!("   ✅ Types that implement Send (can move between threads):");

        // Basic types are Send
        let number_data = 42i32;
        let text_data = String::from("Transferable information");
        let list_data = vec![1, 2, 3, 4, 5];

        let handle = thread::spawn(move || {
            println!("     📊 Received number: {}", number_data);
            println!("     📝 Received text: {}", text_data);
            println!("     📋 Received list: {:?}", list_data);
            println!("     ✅ All data successfully transferred to new thread");
        });

        handle.join().unwrap();
        // Original variables are no longer accessible (moved)

        println!("
   📚 Send Type Examples:
   • i32, u64, f64 - All primitive types
   • String, Vec<T> - Owned collections
   • Box<T> - Owned heap data
   • Most user-defined structs
   • Arc<T> - Thread-safe shared ownership");
    }

    demonstrate_send_types();

    // Example 2: Non-Send Types and Why
    println!("\n2️⃣ Non-Send Types - Location-Restricted Team Members:");

    fn demonstrate_non_send_types() {
        println!("   ❌ Types that do NOT implement Send (cannot move between threads):");

        // Rc is not Send - not thread-safe reference counting
        let local_resource = Rc::new("Local-only resource".to_string());
        println!("     📍 Rc<T>: {}", local_resource);

        // This would NOT compile:
        /*
        let handle = thread::spawn(move || {
            println!("Cannot move Rc: {}", local_resource); // ERROR!
        });
        */

        println!("
   🚫 Non-Send Type Examples:
   • Rc<T> - Not thread-safe (use Arc<T> instead)
   • Raw pointers (*const T, *mut T) - Unsafe
   • Types containing non-Send fields

   💡 Why Rc<T> is not Send:
   • Uses non-atomic reference counting
   • Multiple threads could corrupt the count
   • Could lead to use-after-free or double-free");

        // Safe alternative: Arc is Send
        let thread_safe_resource = Arc::new("Thread-safe resource".to_string());
        let resource_clone = Arc::clone(&thread_safe_resource);

        let handle = thread::spawn(move || {
            println!("     ✅ Arc<T> can be moved: {}", resource_clone);
        });

        handle.join().unwrap();
        println!("     💡 Use Arc<T> instead of Rc<T> for thread-safe sharing");
    }

    demonstrate_non_send_types();

    // Example 3: Sync Trait - Shareable References
    println!("\n3️⃣ Sync Trait - Simultaneously Referenceable Types:");

    fn demonstrate_sync_types() {
        println!("   👥 Types that implement Sync (safe to reference from multiple threads):");

        // Basic immutable data is Sync
        let shared_number = 42i32;
        let shared_text = "Shared information";

        // Arc allows sharing Sync types
        let shared_data = Arc::new((shared_number, shared_text));
        let mut handles = vec![];

        for worker_id in 0..3 {
            let data_ref = Arc::clone(&shared_data);
            let handle = thread::spawn(move || {
                println!("     Worker {}: Number={}, Text={}",
                         worker_id, data_ref.0, data_ref.1);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        println!("
   ✅ Sync Type Examples:
   • i32, u64, f64 - Immutable primitive types
   • &T where T: Sync - Shared references
   • Arc<T> where T: Send + Sync - Shared ownership
   • Mutex<T> where T: Send - Protected data
   • String - Immutable when shared via &str");
    }

    demonstrate_sync_types();

    // Example 4: Non-Sync Types and Alternatives
    println!("\n4️⃣ Non-Sync Types - Single-Team Access Required:");

    fn demonstrate_non_sync_types() {
        println!("   ❌ Types that do NOT implement Sync (unsafe to share references):");

        // Cell and RefCell are not Sync
        let local_cell = Cell::new(42);
        let local_refcell = RefCell::new("Local data".to_string());

        println!("     📱 Cell<T>: {}", local_cell.get());
        println!("     📱 RefCell<T>: {}", local_refcell.borrow());

        // This would NOT compile:
        /*
        let cell_ref = &local_cell;
        let handle = thread::spawn(move || {
            println!("Cannot share Cell: {}", cell_ref.get()); // ERROR!
        });
        */

        println!("
   🚫 Non-Sync Type Examples:
   • Cell<T> - Interior mutability without runtime checks
   • RefCell<T> - Runtime-checked interior mutability
   • Rc<T> - Non-atomic reference counting
   • Raw pointers - Unmanaged memory access

   💡 Why Cell<T>/RefCell<T> are not Sync:
   • Provide interior mutability without thread safety
   • Multiple threads could race on mutations
   • Could violate Rust's borrowing rules");

        // Safe alternative: Mutex for shared mutable data
        let thread_safe_data = Arc::new(Mutex::new("Thread-safe mutable data".to_string()));
        let mut handles = vec![];

        for worker_id in 0..2 {
            let data = Arc::clone(&thread_safe_data);
            let handle = thread::spawn(move || {
                let mut data_guard = data.lock().unwrap();
                data_guard.push_str(&format!(" - Worker {}", worker_id));
                println!("     Worker {} updated data", worker_id);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        let final_data = thread_safe_data.lock().unwrap();
        println!("     ✅ Final shared data: {}", *final_data);
        println!("     💡 Use Mutex<T> for thread-safe interior mutability");
    }

    demonstrate_non_sync_types();

    // Example 5: Custom Types and Send/Sync
    println!("\n5️⃣ Custom Types - Automatic and Manual Implementation:");

    #[derive(Debug)]
    struct TeamMember {
        name: String,
        role: String,
        employee_id: u32,
    }

    // TeamMember is automatically Send + Sync because all fields are Send + Sync

    #[derive(Debug)]
    struct LocalTeamManager {
        team_members: Vec<TeamMember>,
        local_resource: Rc<String>, // This makes the struct !Send + !Sync
    }

    #[derive(Debug)]
    struct ThreadSafeTeamManager {
        team_members: Vec<TeamMember>,
        shared_resource: Arc<String>, // This keeps the struct Send + Sync
    }

    fn demonstrate_custom_send_sync() {
        println!("   🏗️ Custom types inherit Send/Sync from their fields:");

        let team_member = TeamMember {
            name: "Alice Johnson".to_string(),
            role: "Chef".to_string(),
            employee_id: 12345,
        };

        // TeamMember can be moved to another thread
        let handle = thread::spawn(move || {
            println!("     ✅ Team member transferred: {:?}", team_member);
        });
        handle.join().unwrap();

        // Thread-safe team manager
        let safe_manager = Arc::new(ThreadSafeTeamManager {
            team_members: vec![
                TeamMember {
                    name: "Bob Smith".to_string(),
                    role: "Server".to_string(),
                    employee_id: 12346,
                }
            ],
            shared_resource: Arc::new("Shared resource".to_string()),
        });

        let manager_ref = Arc::clone(&safe_manager);
        let handle = thread::spawn(move || {
            println!("     ✅ Shared manager access: {} team members",
                     manager_ref.team_members.len());
        });
        handle.join().unwrap();

        println!("
   📝 Send/Sync Rules for Custom Types:
   • Struct is Send if ALL fields are Send
   • Struct is Sync if ALL fields are Sync
   • Compiler automatically implements when possible
   • Can manually implement (unsafe) when needed
   • One non-Send/Sync field makes entire struct non-Send/Sync");
    }

    demonstrate_custom_send_sync();

    println!("\n🎯 Send/Sync Guidelines:");
    println!("   🚚 Send: Type can be moved between threads (transfer ownership)");
    println!("   👥 Sync: Type can be safely referenced from multiple threads");
    println!("   ✅ Most types are automatically Send + Sync");
    println!("   🔄 Use Arc<T> instead of Rc<T> for thread-safe sharing");
    println!("   🔒 Use Mutex<T> instead of RefCell<T> for thread-safe mutation");

    println!("\n💡 Professional Guidelines:");
    println!("   📋 Design custom types with thread safety in mind");
    println!("   🔍 Check compiler errors for Send/Sync violations");
    println!("   🎯 Choose appropriate synchronization primitives");
    println!("   ⚖️ Balance safety with performance in concurrent designs");
    println!("   📚 Understand the capabilities and limitations of each type");
}

fn main() {
    demonstrate_send_sync_traits();
}
```


## Synchronization Primitives - The Professional Coordination Tools

![Common Thread Safety Challenges in Rust and How Rust Prevents Them](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAACWAAAAZACAYAAAD9qXmxAAAgAElEQVR4XuzdBZRV1fv/8UdQQBoUBKRDulNAQEoBpbsbBAHp7u7uRjoFFJAS6ZQUVFJCuhkQFP2v5/7+M9+JU/fOvTMDvvda3+Va3L332ed1zo3vOp959huPH9/7V2gIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAJuC7xBAMttMwYggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAi4BAljcCAgggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIICAhwIEsDyEYxgCCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAgggQACLewABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQ8FCAAJaHcAxDAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBAhgcQ8ggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAh4KEMDyEI5hCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAABLO4BBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQMBDAQJYHsIxDAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBAggMU9gAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAgh4KEAAy0M4hiGAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACBLC4BxBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABDwUIYHkIxzAEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAgAAW9wACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggg4KEAASwP4RiGAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCBDA4h5AAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBDwUIIDlIRzDEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAECWNwDCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggICHAgSwPIRjGAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCBAAIt7AAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBDwUIAAlodwDEMAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEECGBxDyCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACHgoQwPIQjmEIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAEs7gEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAwEMBAlgewjEMAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEECCAxT2AAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCHgoQADLQziGIYAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIEsLgHEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEPBQhgeQjHMAQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEECAABb3AAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCDgoQABLA/hGIYAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIEMDiHkAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEPBQggOUhHMMQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQJY3AMIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAgIcCBLA8hGMYAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIEAAi3sAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEPBQgACWh3AMQwABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQIYHEPIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIeChDA8hCOYQgggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAASzuAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEDAQwECWB7CMQwBBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQIIDFPYAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIeChAAMtDOIYhgAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAgSwuAcQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQ8FCGB5CMcwBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQIAAFvcAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIOChAAEsD+EYhgACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggQwOIeQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQ8FCCA5SEcwxBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABAljcAwgggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIICAhwIEsDyEYxgCCCCAAAIIIIAAAggggAACCCCAAAII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CBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBBIRUAAKxVGgxAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkEUBAawsdl3NBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAikIiCAlQqjQQgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyKKAAFYWu65mAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRSERDASoXRIAQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIZFFAACuLXVczAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKpCAhgpcJoEAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEsigggJXFrquZAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFUBASwUmE0CAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECWRQQwMpi19VMgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAqAgJYqTAahAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBLAoIYGWx62omQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAVAQGsVBgNQoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAFgUEsLLYdTUTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJCKgABWKowGIUCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgiwICWFnsupoJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEEhFQAArFUaDECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQRQEBrCx2Xc0ECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECKQiIICVCqNBCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIooAAVha7rmYCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBFIREMBKhdEgBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhkUUAAK4tdVzMBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAqkICGClwmgQAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgSyKCCAlcWuq5kAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVQEBLBSYTQIAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJZFBDAymLX1UyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCoCAlipMBqEAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIEsCghgZbHraiZAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBUBAaxUGA1CgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAWBQSwsth1NRMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkIqAAFYqjAYhQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCLAgJYWey6mgkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQSEVAACsVRoMQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBFAQGsLHZdzQQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIpCIggJUKo0EIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMiigABWFruuZgIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEUhEQwEqF0SAECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECGRRQAAri11XMwECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECqQgIYKXCaBACBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLIoIICVxa6rmQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBVAQEsFJhNAgBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlkUEMDKYtfVTIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAKgICWKkwGoQAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgSwKCGBlsetqJkCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFQEBrFQYDUKAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQBYFBLCy2HU1EyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQioAAViqMBiFAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIIsCAlhZ7LqaCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBBIRUAAKxVGgxAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkEUBAawsdl3NBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAikIiCAlQqjQQgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyKKAAFYWu65mAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRSERDASoXRIAQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIZFFAACuLXVczAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKpCAhgpcJoEAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEsigggJXFrquZAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFUBASwUmE0CAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECWRQQwMpi19VMgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAqAgJYqTAahAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBLAoIYGWx62omQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAVAQGsVBgNQoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAFgUEsLLYdTUTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJCKgABWKowGIUCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgiwICWFnsupoJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEEhFQAArFUaDECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQRQEBrCx2Xc0ECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECKQiIICVCqNBCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIooAAVha7rmYCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBFIREMBKhdEgBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhkUUAAK4tdVzMBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAqkICGClwmgQAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgSyKCCAlcWuq5kAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVQEBLBSYTQIAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJZFBDAymLX1UyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCoCAlipMBqEAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIEsCghgZbHraiZAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBUBAaxUGA1CgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAWBQSwsth1NRMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkIqAAFYqjAYhQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCLAgJYWey6mgkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQSEVAACsVRoMQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBFAQGsLHZdzQQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIpCIggJUKo0EIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMiigABWFruuZgIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEUhEQwEqF0SAECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECGRRQAAri11XMwECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECqQgIYKXCaBACBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLIoIICVxa6rmQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBVAQEsFJhNAgBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlkUEMDKYtfVTIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAKgICWKkwGoQAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgSwKCGBlsetqJkCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFQEBrFQYDUKAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQBYFBLCy2HU1EyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQioAAViqMBiFAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIIsCAlhZ7LqaCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBBIRUAAKxVGgxAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkEUBAawsdl3NBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAikIiCAlQqjQQgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyKKAAFYWu65mAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRSERDASoXRIAQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIZFFAACuLXVczAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKpCAhgpcJoEAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEsigggJXFrquZAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFUBASwUmE0CAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECWRQQwMpi19VMgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAqAgJYqTAahAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBLAoIYGWx62omQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAVAQGsVBgNQoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAFgUEsLLYdTUTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJCKgABWKowGIUCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgiwICWFnsupoJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEEhFQAArFUaDECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQRQEBrCx2Xc0ECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECKQiIICVCqNBCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIooAAVha7rmYCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBFIREMBKhdEgBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhkUUAAK4tdVzMBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAqkICGClwmgQAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgSyKCCAlcWuq5kAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVQEBLBSYTQIAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJZFBDAymLX1UyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCoCAlipMBqEAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIEsCghgZbHraiZAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBUBAaxUGA1CgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAWBQSwsth1NRMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkIqAAFYqjAYhQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCLAgJYWey6mgkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQSEVAACsVRoMQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBFAQGsLHZdzQQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIpCIggJUKo0EIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMiigABWFruuZgIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEUhEQwEqF0SAECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECGRRQAAri11XMwECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECqQgIYKXCaBACBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLIoIICVxa6rmQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBVAQEsFJhNAgBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlkUEMDKYtfVTIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAKgICWKkwGoQAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgSwKCGBlsetqJkCAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFQEBrFQYDUKAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQBYFBLCy2HU1EyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQioAAViqMBiFAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIIsCAlhZ7LqaCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBBIRUAAKxVGgxAgQIAAAQIECBAgQIAAAQIECBAgQJWaGUwAACAASURBVIAAAQIECBAgQIAAAQIECBAgkEUBAawsdl3NBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAikIiCAlQqjQQgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyKKAAFYWu65mAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRSERDASoXRIAQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIZFFAACuLXVczAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKpCAhgpcJoEAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEsigggJXFrquZAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFUBASwUmE0CAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECWRQQwMpi19VMgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAqAgJYqTAa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vacAqx6s4NI2UIDlUsOz2yRAAiRAAiRAAiTQRAQYqGoiY7IrJEACJEACJEACJOBiAvRrXWx8dp0ESIAESIAESIAEmoQABVhNYsgG7gYFWA1svEZvOgVYjW5Btp8ESIAESIAESIAESICBKo4BEiABEiABEiABEiCBZiBAv7YZrMg+kAAJkAAJkAAJkIC7CVCA5W7710PvKcCqByu4tA0UYLnU8Ow2CZAACZAACZAACTQRAQaqmsiY7AoJkAAJkAAJkAAJuJgA/VoXG59dJwESIAESIAESIIEmIUABVpMYsoG7QQFWAxuv0ZtOAVajW5DtJwESIAESIAESIAESYKCKY4AESIAESIAESIAESKAZCNCvbQYrsg8kQAIkQAIkQAIk4G4CFGC52/710HsKsOrBCi5tAwVYLjU8u00CJEACJEACJEACTUSAgaomMia7QgIkQAIkQAIkQAIuJkC/1sXGZ9dJgARIgARIgARIoEkIUIDVJIZs4G5QgNXAxmv0plOA1egWZPtJgARIgARIgARIgAQYqOIYIAESIAESIAESIAESaAYC9GubwYrsAwmQAAmQAAmQAAm4mwAFWO62fz30ngKserCCS9tAAZZLDc9ukwAJkAAJkAAJkEATEWCgqomMya6QAAmQAAmQAAmQgIsJ0K91sfHZdRIgARIgARIgARJoEgIUYDWJIRu4GxRgNbDxGr3pFGA1ugXZfhIgARIgARIgARIgAQaqOAZIgARIgARIgARIgASagQD92mawIvtAAiRAAiRAAiRAAu4mQAGWu+1fD72nAKserODSNlCA5VLDs9skQAIkQAIkQAIk0EQEGKhqImOyKyRAAiRAAiRAAiTgYgL0a11sfHadBEiABEiABEiABJqEAAVYTWLIBu4GBVgNbLxGbzoFWI1uQbafBEiABEiABEiABEiAgSqOARIgARIgARIgARIggWYgQL+2GazIPpAACZAACZAACZCAuwlQgOVu+9dD7ynAqgcruLQNFGC51PDsNgmQAAmQAAmQAAk0EQEGqprImOwKCZAACZAACZAACbiYAP1aFxufXScBEiABEiABEiCBJiFAAVaTGLKBu0EBVgMbr9GbTgFWo1uQ7ScBEiABEiABEiABEmCgimOABEiABEiABEiABEigGQjQr20GK7IPJEACJEACJEACJOBuAhRgudv+9dB7CrDqwQoubYMjwHJp99ltEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiCBGhIYN+6fNbwbb0UC5ROgAKt8VjyzxgQowKoxUN6OBEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABEiABFxMgAIsFxu/l7tOAVYvG8DNj3cEWGPefNPNGNh3EiABEiABEiABEiCBBiZw14gRqvX0aRvYiGw6CZAACZAACZAACZAA6NdyEJAACZAACZAACZAACTQ6AcenpQCr0S3ZuO2nAKtxbdfwLacAq+FNyA6QAAmQAAmQAAmQgOsJMFDl+iFAACRAAiRAAiRAAiTQFATo1zaFGdkJEiABEiABEiABEnA1AQqwXG3+uug8BVh1YQZ3NoICLHfanb0mARIgARIgARIggWYiwEBVM1mTfSEBEiABEiABEiAB9xKgX+te27PnJEACJEACJEACJNAsBCjAahZLNm4/KMBqXNs1fMspwGp4E7IDJEACJEACJEACJOB6AgxUuX4IEAAJkAAJkAAJkAAJNAUB+rVNYUZ2ggRIgARIgARIgARcTYACLFebvy46TwFWXZjBnY2gAMuddmevSYAESIAESIAESKCZCDBQ1UzWZF9IgARIgARIgARIwL0E6Ne61/bsOQmQAAmQAAmQAAk0CwEKsJrFko3bDwqwGtd2Dd9yCrAa3oTsAAmQAAmQAAmQAAm4ngADVa4fAgRAAiRAAiRAAiRAAk1BgH5tU5iRnSABEiABEiABEiABVxOgAMvV5q+LzlOAVRdmcGcjKMByp93ZaxIgARIgARIgARJoJgIMVDWTNdkXEiABEiABEiABEnAvAfq17rU9e04CJEACJEACJEACzUKAAqxmsWTj9oMCrMa1XcO3nAKshjchO0ACJEACJEACJEACrifAQJXrhwABkAAJkAAJkAAJkEBTEKBf2xRmZCdIgARIgARIgARIwNUEKMBytfnrovMUYNWFGdzZCAqw3Gl39poESIAESIAESIAEmokAA1XNZE32hQRIgARIgARIgATcS4B+rXttz56TAAmQAAmQAAmQQLMQoACrWSzZuP2gAKtxbdfwLacAq+FNyA6QAAmQAAmQAAmQgOsJMFDl+iFAACRAAiRAAiRAAiTQFATo1zaFGdkJEiABEiABEiABEnA1AQqwXG3+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QIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJRXQACrvLXTcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEChYQwCq4AJonQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKC8AgJY5a2dnhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgULCAAFbBBdA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQLlFRDAKm/t9JwAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgYIFBLAKLoDmCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAor4AAVnlrp+cECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBQsIIBVcAE0T4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAeQUEsMpbOz0nQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKBgAQGsggugeQIECBAgQIAAAQIECBAgQIDA/2PXDkoAAAAQiPVvbQw5WAJlfiVAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIEBr0Kc9AAAgAElEQVSAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAPFLL7AAACAASURBVAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEDgLOGCdBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBXwAGru53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAicBRywzgOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECgK+CA1d1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEzgIOWOcBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg0BVwwOpupzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAmcBB6zzAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEOgKOGB1t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGzgAPWeQDxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAh0BRywuttpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAWcAB6zyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEugIOWN3tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4CzggHUeQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAl0BB6zudpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIHAWcMA6DyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGugANWdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBA4CzhgnQcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBAV8ABq7ud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQInAUcsM4DiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoCvggNXdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBM4CDljnAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQINAVcMDqeAUdGAAAIABJREFUbqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJnAQes8wDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDoCjhgdbfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBs4AD1nkA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIdAUcsLrbaU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwFnAAes8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBLoCDljd7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOAs4IB1HkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJdAQes7naaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBwFnDAOg8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBroADVnc7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQOAs4YJ0HEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQFfAAau7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECJwFHLDOA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQKAr4IDV3U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgTOAg5Y5wHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDQFXDA6m6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECZwEHrPMA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ6Ao4YHW305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgbOAA9Z5APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECHQFHLC622lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMBZwAHrPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgS6Ag5Y3e00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgLOCAdR5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECXQEHrO52mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgcBZwwDoPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAga6AA1Z3O80JECBAgAABAgQIECBAgAABAgQIECBAYO3aQQkAAAACsf6tjXEIS6DMrwQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD/79LiAAAgAElEQVT4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXwAHrdzvNCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBCIBRyw4gHEEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECDwK+CA9bud5gQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIxAIOWPEA4gkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQ+BVwwPrdTnMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBGIBB6x4APEECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECPwKOGD9bqc5AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKxgANWPIB4AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgR+BRywfrfTnAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBWMABKx5APAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECvwIOWL/baU6AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQCzggBUPIJ4AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgV8BB6zf7TQnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCAWcMCKBxBPgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMCvgAPW73aaEyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAQCzhgxQOIJ0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEDgV8AB63c7zQkQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQiAUcsOIBxBMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAg8CvggPW7neYECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECMQCDljxAOIJECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEPgVcMD63U5zAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRiAQeseADxBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAj8Cjhg/W6nOQECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECsYADVjyAeAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEfgUcsH6305wAAQIECIkQJMgAAAQ8SURBVBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgVjAASseQDwBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAr8CDli/22lOgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEAs4IAVDyCeAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFfAQes3+00J0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgFnDAigcQT4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAr4AD1u92mhMgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgEAs4YMUDiCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBA4FfAAet3O80JECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEIgFHLDiAcQTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIPAr4ID1u53mBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjEAg5Y8QDiCRAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBD4FXDA+t1OcwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEYgEHrHgA8QQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI/Ao4YP1upzkBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABArGAA1Y8gHgCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBH4FHLB+t9OcAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIFYwAErHkA8AQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQK/Ag5Yv9tpToAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBALOCAFQ8gngABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBXwEHrN/tNCdAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIBZwwIoHEE+AAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwK+AA9bvdpoTIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIBALOGDFA4gnQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQOBXYDWJL+S1H0V8AAAAAElFTkSuQmCC)

Common Thread Safety Challenges in Rust and How Rust Prevents Them

**Comprehensive guide to Rust's synchronization tools for safe concurrent programming:**

```rust
fn demonstrate_synchronization_primitives() {
    println!("🔧 Synchronization Primitives - Professional Coordination Tools");
    println!("{:=<75}", "");

    use std::sync::{Arc, Mutex, RwLock, Condvar, Barrier};
    use std::sync::atomic::{AtomicUsize, AtomicBool, Ordering};
    use std::sync::mpsc;
    use std::thread;
    use std::time::Duration;
    use std::collections::HashMap;

    // Synchronization primitives are like professional coordination tools
    println!("🛠️ Professional Coordination Tool Arsenal:");
    println!("   🔒 Mutex - Exclusive access control (one team at a time)");
    println!("   📚 RwLock - Read/write access control (many readers, one writer)");
    println!("   📡 Channels - Message passing between teams");
    println!("   ⚛️ Atomic Types - Lock-free operations for simple data");
    println!("   🚦 Condvar - Wait/signal coordination between teams");
    println!("   🚧 Barrier - Synchronization checkpoints for team coordination");

    // Tool 1: Mutex - Exclusive Resource Access
    println!("\n1️⃣ Mutex - Exclusive Resource Access Control:");

    #[derive(Debug)]
    struct Restaurant {
        name: String,
        daily_revenue: f64,
        orders_processed: u32,
        current_status: String,
    }

    impl Restaurant {
        fn new(name: String) -> Self {
            Restaurant {
                name,
                daily_revenue: 0.0,
                orders_processed: 0,
                current_status: "Open".to_string(),
            }
        }

        fn process_order(&mut self, order_value: f64) {
            self.daily_revenue += order_value;
            self.orders_processed += 1;
            println!("     💰 Order processed: ${:.2} (Total: ${:.2})",
                     order_value, self.daily_revenue);
        }

        fn update_status(&mut self, new_status: String) {
            self.current_status = new_status;
            println!("     🏪 Status updated to: {}", self.current_status);
        }
    }

    fn demonstrate_mutex() {
        println!("   🔒 Mutex ensures exclusive access to shared restaurant data:");

        let restaurant = Arc::new(Mutex::new(Restaurant::new("Ocean View Bistro".to_string())));
        let mut worker_handles = vec![];

        // Multiple workers (servers, cashiers) processing orders
        for worker_id in 0..5 {
            let restaurant_ref = Arc::clone(&restaurant);
            let handle = thread::spawn(move || {
                // Each worker gets exclusive access when needed
                let mut restaurant_guard = restaurant_ref.lock().unwrap();

                match worker_id {
                    0 => restaurant_guard.process_order(25.99),
                    1 => restaurant_guard.process_order(15.50),
                    2 => restaurant_guard.update_status("Busy".to_string()),
                    3 => restaurant_guard.process_order(32.75),
                    4 => restaurant_guard.update_status("Normal".to_string()),
                    _ => {}
                }

                println!("     👷 Worker {} completed task", worker_id);
                // Mutex automatically unlocked when guard goes out of scope
            });
            worker_handles.push(handle);
        }

        for handle in worker_handles {
            handle.join().unwrap();
        }

        let final_state = restaurant.lock().unwrap();
        println!("     📊 Final state: {} orders, ${:.2} revenue, Status: {}",
                 final_state.orders_processed,
                 final_state.daily_revenue,
                 final_state.current_status);
    }

    demonstrate_mutex();

    // Tool 2: RwLock - Efficient Read/Write Access
    println!("\n2️⃣ RwLock - Efficient Read/Write Access Control:");

    #[derive(Debug, Clone)]
    struct MenuDatabase {
        items: HashMap<String, f64>,
        daily_specials: Vec<String>,
        last_updated: String,
    }

    impl MenuDatabase {
        fn new() -> Self {
            let mut items = HashMap::new();
            items.insert("Pizza Margherita".to_string(), 15.99);
            items.insert("Caesar Salad".to_string(), 8.50);
            items.insert("Pasta Carbonara".to_string(), 12.75);

            MenuDatabase {
                items,
                daily_specials: vec!["Grilled Salmon".to_string()],
                last_updated: "Morning".to_string(),
            }
        }

        fn get_price(&self, item: &str) -> Option<f64> {
            self.items.get(item).copied()
        }

        fn get_specials(&self) -> Vec<String> {
            self.daily_specials.clone()
        }

        fn update_price(&mut self, item: String, price: f64) {
            self.items.insert(item, price);
            self.last_updated = "Just now".to_string();
        }
    }

    fn demonstrate_rwlock() {
        println!("   📚 RwLock allows multiple readers, exclusive writer:");

        let menu_db = Arc::new(RwLock::new(MenuDatabase::new()));
        let mut handles = vec![];

        // Multiple readers (servers checking prices)
        for server_id in 0..4 {
            let menu = Arc::clone(&menu_db);
            let handle = thread::spawn(move || {
                // Multiple servers can read simultaneously
                let menu_reader = menu.read().unwrap();

                if let Some(price) = menu_reader.get_price("Pizza Margherita") {
                    println!("     👩‍💼 Server {}: Pizza price is ${:.2}", server_id, price);
                }

                let specials = menu_reader.get_specials();
                println!("     👩‍💼 Server {}: Today's specials: {:?}", server_id, specials);

                thread::sleep(Duration::from_millis(10)); // Simulate work
            });
            handles.push(handle);
        }

        // One writer (manager updating prices)
        {
            let menu = Arc::clone(&menu_db);
            let handle = thread::spawn(move || {
                thread::sleep(Duration::from_millis(20)); // Let readers start

                // Exclusive write access
                let mut menu_writer = menu.write().unwrap();
                println!("     👨‍💼 Manager: Updating pizza price");
                menu_writer.update_price("Pizza Margherita".to_string(), 16.99);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        let final_menu = menu_db.read().unwrap();
        println!("     📊 Final pizza price: ${:.2}",
                 final_menu.get_price("Pizza Margherita").unwrap());
    }

    demonstrate_rwlock();

    // Tool 3: Channels - Message Passing
    println!("\n3️⃣ Channels - Message Passing Communication:");

    #[derive(Debug, Clone)]
    enum RestaurantMessage {
        NewOrder { table: u32, items: Vec<String>, total: f64 },
        OrderComplete { table: u32 },
        StatusUpdate { message: String },
        Shutdown,
    }

    fn demonstrate_channels() {
        println!("   📡 Channels enable safe message passing between threads:");

        let (sender, receiver) = mpsc::channel();

        // Kitchen thread receiving messages
        let kitchen_handle = thread::spawn(move || {
            let mut orders_handled = 0;

            loop {
                match receiver.recv() {
                    Ok(RestaurantMessage::NewOrder { table, items, total }) => {
                        println!("     👨‍🍳 Kitchen: Received order for table {} - {:?} (${:.2})",
                                 table, items, total);
                        orders_handled += 1;

                        // Simulate cooking time
                        thread::sleep(Duration::from_millis(50));
                        println!("     ✅ Kitchen: Order for table {} completed", table);
                    }
                    Ok(RestaurantMessage::OrderComplete { table }) => {
                        println!("     📋 Kitchen: Table {} order marked complete", table);
                    }
                    Ok(RestaurantMessage::StatusUpdate { message }) => {
                        println!("     📢 Kitchen received status: {}", message);
                    }
                    Ok(RestaurantMessage::Shutdown) => {
                        println!("     🔒 Kitchen shutting down. Handled {} orders", orders_handled);
                        break;
                    }
                    Err(_) => {
                        println!("     ❌ Kitchen: Channel disconnected");
                        break;
                    }
                }
            }
        });

        // Front-of-house sending messages
        let sender_clone = sender.clone();
        let front_house_handle = thread::spawn(move || {
            // Send various messages
            sender_clone.send(RestaurantMessage::NewOrder {
                table: 1,
                items: vec!["Pizza".to_string(), "Salad".to_string()],
                total: 24.49,
            }).unwrap();

            sender_clone.send(RestaurantMessage::NewOrder {
                table: 2,
                items: vec!["Pasta".to_string()],
                total: 12.75,
            }).unwrap();

            sender_clone.send(RestaurantMessage::StatusUpdate {
                message: "Rush hour starting".to_string(),
            }).unwrap();

            sender_clone.send(RestaurantMessage::OrderComplete {
                table: 1,
            }).unwrap();

            thread::sleep(Duration::from_millis(100));

            sender_clone.send(RestaurantMessage::Shutdown).unwrap();
            println!("     👥 Front-of-house: Sent all messages");
        });

        front_house_handle.join().unwrap();
        kitchen_handle.join().unwrap();

        println!("     ✅ Channel communication completed successfully");
    }

    demonstrate_channels();

    // Tool 4: Atomic Types - Lock-free Operations
    println!("\n4️⃣ Atomic Types - Lock-free High-Performance Operations:");

    fn demonstrate_atomics() {
        println!("   ⚛️ Atomic types provide lock-free thread-safe operations:");

        let customer_count = Arc::new(AtomicUsize::new(0));
        let restaurant_open = Arc::new(AtomicBool::new(true));
        let mut handles = vec![];

        // Multiple threads handling customers
        for worker_id in 0..6 {
            let count = Arc::clone(&customer_count);
            let open_flag = Arc::clone(&restaurant_open);

            let handle = thread::spawn(move || {
                // Check if restaurant is open (atomic read)
                if open_flag.load(Ordering::SeqCst) {
                    // Atomically increment customer count
                    let current_count = count.fetch_add(1, Ordering::SeqCst);
                    println!("     👤 Worker {}: Customer #{} served", worker_id, current_count + 1);

                    // Simulate service time
                    thread::sleep(Duration::from_millis(20));

                    // Check if we should close (every 3rd worker checks)
                    if worker_id % 3 == 0 && current_count > 2 {
                        println!("     🔒 Worker {}: Suggesting to close", worker_id);
                        open_flag.store(false, Ordering::SeqCst);
                    }
                } else {
                    println!("     🚫 Worker {}: Restaurant closed, turning away customer", worker_id);
                }
            });

            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        let final_count = customer_count.load(Ordering::SeqCst);
        let is_open = restaurant_open.load(Ordering::SeqCst);

        println!("     📊 Final results: {} customers served, Open: {}", final_count, is_open);
        println!("     ⚡ Atomic operations provide thread-safe counters without locks");
    }

    demonstrate_atomics();

    // Tool 5: Condvar and Barrier - Advanced Coordination
    println!("\n5️⃣ Condvar and Barrier - Advanced Team Coordination:");

    fn demonstrate_condvar_barrier() {
        println!("   🚦 Condvar for wait/signal coordination:");

        let data = Arc::new(Mutex::new(0));
        let condvar = Arc::new(Condvar::new());

        let data_clone = Arc::clone(&data);
        let condvar_clone = Arc::clone(&condvar);

        // Worker thread waiting for signal
        let worker = thread::spawn(move || {
            let mut value = data_clone.lock().unwrap();

            while *value < 10 {
                println!("     ⏳ Worker: Waiting for value to reach 10 (current: {})", *value);
                value = condvar_clone.wait(value).unwrap();
            }

            println!("     ✅ Worker: Value reached {}! Proceeding with work.", *value);
        });

        // Main thread updating value and signaling
        thread::sleep(Duration::from_millis(50));

        for i in 1..=10 {
            {
                let mut value = data.lock().unwrap();
                *value = i;
                println!("     📈 Main: Updated value to {}", i);
            }
            condvar.notify_one();
            thread::sleep(Duration::from_millis(10));
        }

        worker.join().unwrap();

        println!("   🚧 Barrier for team synchronization:");

        let barrier = Arc::new(Barrier::new(3));
        let mut handles = vec![];

        for worker_id in 0..3 {
            let barrier_clone = Arc::clone(&barrier);
            let handle = thread::spawn(move || {
                println!("     🏃 Worker {} starting preparation phase", worker_id);
                thread::sleep(Duration::from_millis(worker_id * 20));

                println!("     ⏰ Worker {} finished preparation, waiting for others", worker_id);
                barrier_clone.wait();

                println!("     🚀 Worker {} starting service phase (all ready!)", worker_id);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        println!("     ✅ All workers synchronized and completed their phases");
    }

    demonstrate_condvar_barrier();

    println!("\n🎯 Synchronization Primitive Guidelines:");
    println!("   🔒 Use Mutex for exclusive access to shared mutable data");
    println!("   📚 Use RwLock when you have many readers, few writers");
    println!("   📡 Use channels for message passing between threads");
    println!("   ⚛️ Use atomic types for simple lock-free operations");
    println!("   🚦 Use Condvar for complex wait/signal patterns");
    println!("   🚧 Use Barrier for synchronized group operations");

    println!("\n💡 Professional Selection Criteria:");
    println!("   📊 Choose based on access patterns (read-heavy vs write-heavy)");
    println!("   ⚡ Consider performance requirements (lock-free vs locked)");
    println!("   🔄 Evaluate complexity (simple counters vs complex coordination)");
    println!("   📋 Plan for deadlock prevention in lock-based solutions");
    println!("   🎯 Test concurrent code thoroughly under load");
}

fn main() {
    demonstrate_synchronization_primitives();
}
```


## Real-World Thread Safety Applications

**Complete restaurant management system demonstrating professional thread safety patterns:**

```rust
fn demonstrate_real_world_thread_safety() {
    println!("🏢 Real-World Thread Safety - Complete Restaurant Management System");
    println!("{:=<75}", "");

    use std::sync::{Arc, Mutex, RwLock, mpsc};
    use std::sync::atomic::{AtomicUsize, AtomicBool, Ordering};
    use std::thread;
    use std::time::{Duration, Instant};
    use std::collections::{HashMap, VecDeque};

    // Complete thread-safe restaurant system
    println!("💼 Professional Thread-Safe Restaurant Management:");

    // Application 1: Multi-threaded Order Processing System
    println!("\n1️⃣ Multi-threaded Order Processing System:");

    #[derive(Debug, Clone)]
    struct Order {
        id: u32,
        table_number: u32,
        items: Vec<String>,
        total: f64,
        timestamp: Instant,
        status: OrderStatus,
    }

    #[derive(Debug, Clone)]
    enum OrderStatus {
        Received,
        InProgress,
        Ready,
        Served,
    }

    struct OrderManager {
        pending_orders: Arc<Mutex<VecDeque<Order>>>,
        active_orders: Arc<RwLock<HashMap<u32, Order>>>,
        completed_orders: Arc<Mutex<Vec<Order>>>,
        order_counter: Arc<AtomicUsize>,
        total_revenue: Arc<Mutex<f64>>,
    }

    impl OrderManager {
        fn new() -> Self {
            OrderManager {
                pending_orders: Arc::new(Mutex::new(VecDeque::new())),
                active_orders: Arc::new(RwLock::new(HashMap::new())),
                completed_orders: Arc::new(Mutex::new(Vec::new())),
                order_counter: Arc::new(AtomicUsize::new(0)),
                total_revenue: Arc::new(Mutex::new(0.0)),
            }
        }

        fn submit_order(&self, table_number: u32, items: Vec<String>, total: f64) -> u32 {
            let order_id = self.order_counter.fetch_add(1, Ordering::SeqCst) as u32;

            let order = Order {
                id: order_id,
                table_number,
                items,
                total,
                timestamp: Instant::now(),
                status: OrderStatus::Received,
            };

            // Thread-safe addition to pending queue
            self.pending_orders.lock().unwrap().push_back(order.clone());

            println!("     📝 Order {} submitted for table {}: ${:.2}",
                     order_id, table_number, total);
            order_id
        }

        fn process_next_order(&self) -> Option<Order> {
            let mut pending = self.pending_orders.lock().unwrap();
            if let Some(mut order) = pending.pop_front() {
                order.status = OrderStatus::InProgress;

                // Move to active orders (thread-safe)
                self.active_orders.write().unwrap().insert(order.id, order.clone());

                println!("     👨‍🍳 Kitchen: Started processing order {}", order.id);
                Some(order)
            } else {
                None
            }
        }

        fn complete_order(&self, order_id: u32) -> Result<(), String> {
            let mut active = self.active_orders.write().unwrap();

            if let Some(mut order) = active.remove(&order_id) {
                order.status = OrderStatus::Ready;

                // Add to revenue
                *self.total_revenue.lock().unwrap() += order.total;

                // Move to completed
                self.completed_orders.lock().unwrap().push(order);

                println!("     ✅ Order {} completed and ready", order_id);
                Ok(())
            } else {
                Err(format!("Order {} not found in active orders", order_id))
            }
        }

        fn get_stats(&self) -> (usize, usize, usize, f64) {
            let pending_count = self.pending_orders.lock().unwrap().len();
            let active_count = self.active_orders.read().unwrap().len();
            let completed_count = self.completed_orders.lock().unwrap().len();
            let revenue = *self.total_revenue.lock().unwrap();

            (pending_count, active_count, completed_count, revenue)
        }
    }

    fn demonstrate_order_processing() {
        println!("   🏗️ Multi-threaded order processing with thread-safe coordination:");

        let order_manager = Arc::new(OrderManager::new());

        // Customer threads submitting orders
        let mut customer_handles = vec![];
        for table in 1..=5 {
            let manager = Arc::clone(&order_manager);
            let handle = thread::spawn(move || {
                let items = match table {
                    1 => vec!["Pizza".to_string(), "Salad".to_string()],
                    2 => vec!["Pasta".to_string(), "Wine".to_string()],
                    3 => vec!["Burger".to_string(), "Fries".to_string()],
                    4 => vec!["Fish".to_string(), "Vegetables".to_string()],
                    5 => vec!["Soup".to_string(), "Bread".to_string()],
                    _ => vec!["Default".to_string()],
                };
                let total = table as f64 * 5.0 + 10.0;

                manager.submit_order(table, items, total);
                thread::sleep(Duration::from_millis(table * 10));
            });
            customer_handles.push(handle);
        }

        // Kitchen threads processing orders
        let mut kitchen_handles = vec![];
        for chef_id in 0..2 {
            let manager = Arc::clone(&order_manager);
            let handle = thread::spawn(move || {
                for _ in 0..3 {  // Each chef processes up to 3 orders
                    if let Some(order) = manager.process_next_order() {
                        // Simulate cooking time
                        thread::sleep(Duration::from_millis(50));
                        manager.complete_order(order.id).ok();
                    }
                    thread::sleep(Duration::from_millis(20));
                }
                println!("     👨‍🍳 Chef {} finished shift", chef_id);
            });
            kitchen_handles.push(handle);
        }

        // Wait for all customers to submit orders
        for handle in customer_handles {
            handle.join().unwrap();
        }

        // Wait for kitchen to process orders
        for handle in kitchen_handles {
            handle.join().unwrap();
        }

        let (pending, active, completed, revenue) = order_manager.get_stats();
        println!("     📊 Final stats: {} pending, {} active, {} completed, ${:.2} revenue",
                 pending, active, completed, revenue);
    }

    demonstrate_order_processing();

    // Application 2: Real-time Inventory Management
    println!("\n2️⃣ Real-time Inventory Management System:");

    #[derive(Debug, Clone)]
    struct InventoryItem {
        name: String,
        quantity: u32,
        unit_cost: f64,
        reorder_level: u32,
    }

    struct InventoryManager {
        items: Arc<RwLock<HashMap<String, InventoryItem>>>,
        low_stock_alerts: Arc<Mutex<Vec<String>>>,
        usage_stats: Arc<RwLock<HashMap<String, u32>>>,
        reorder_requests: Arc<Mutex<Vec<String>>>,
    }

    impl InventoryManager {
        fn new() -> Self {
            let mut items = HashMap::new();

            items.insert("Flour".to_string(), InventoryItem {
                name: "Flour".to_string(),
                quantity: 50,
                unit_cost: 2.50,
                reorder_level: 10,
            });

            items.insert("Tomatoes".to_string(), InventoryItem {
                name: "Tomatoes".to_string(),
                quantity: 30,
                unit_cost: 3.00,
                reorder_level: 5,
            });

            items.insert("Cheese".to_string(), InventoryItem {
                name: "Cheese".to_string(),
                quantity: 20,
                unit_cost: 5.00,
                reorder_level: 8,
            });

            InventoryManager {
                items: Arc::new(RwLock::new(items)),
                low_stock_alerts: Arc::new(Mutex::new(Vec::new())),
                usage_stats: Arc::new(RwLock::new(HashMap::new())),
                reorder_requests: Arc::new(Mutex::new(Vec::new())),
            }
        }

        fn use_ingredient(&self, name: &str, quantity: u32) -> Result<(), String> {
            let mut items = self.items.write().unwrap();

            if let Some(item) = items.get_mut(name) {
                if item.quantity >= quantity {
                    item.quantity -= quantity;

                    // Update usage stats
                    *self.usage_stats.write().unwrap()
                        .entry(name.to_string()).or_insert(0) += quantity;

                    println!("     📦 Used {} units of {} (remaining: {})",
                             quantity, name, item.quantity);

                    // Check for low stock
                    if item.quantity <= item.reorder_level {
                        self.low_stock_alerts.lock().unwrap().push(name.to_string());
                        self.reorder_requests.lock().unwrap().push(name.to_string());
                        println!("     ⚠️ Low stock alert: {} (only {} left)", name, item.quantity);
                    }

                    Ok(())
                } else {
                    Err(format!("Insufficient {} (need {}, have {})", name, quantity, item.quantity))
                }
            } else {
                Err(format!("Ingredient {} not found", name))
            }
        }

        fn restock_ingredient(&self, name: &str, quantity: u32) {
            let mut items = self.items.write().unwrap();

            if let Some(item) = items.get_mut(name) {
                item.quantity += quantity;
                println!("     📈 Restocked {} units of {} (total: {})",
                         quantity, name, item.quantity);

                // Remove from alerts if no longer low
                if item.quantity > item.reorder_level {
                    let mut alerts = self.low_stock_alerts.lock().unwrap();
                    alerts.retain(|x| x != name);
                }
            }
        }

        fn get_inventory_status(&self) -> (usize, usize, usize) {
            let items_count = self.items.read().unwrap().len();
            let alerts_count = self.low_stock_alerts.lock().unwrap().len();
            let usage_tracked = self.usage_stats.read().unwrap().len();

            (items_count, alerts_count, usage_tracked)
        }
    }

    fn demonstrate_inventory_management() {
        println!("   📦 Thread-safe inventory management with concurrent access:");

        let inventory = Arc::new(InventoryManager::new());
        let mut handles = vec![];

        // Kitchen threads using ingredients
        for chef_id in 0..3 {
            let inv = Arc::clone(&inventory);
            let handle = thread::spawn(move || {
                let ingredients = match chef_id {
                    0 => vec![("Flour", 5), ("Cheese", 3)],
                    1 => vec![("Tomatoes", 8), ("Flour", 3)],
                    2 => vec![("Cheese", 4), ("Tomatoes", 2)],
                    _ => vec![],
                };

                for (ingredient, amount) in ingredients {
                    match inv.use_ingredient(ingredient, amount) {
                        Ok(_) => println!("     👨‍🍳 Chef {} used {} {}", chef_id, amount, ingredient),
                        Err(e) => println!("     ❌ Chef {}: {}", chef_id, e),
                    }
                    thread::sleep(Duration::from_millis(30));
                }
            });
            handles.push(handle);
        }

        // Supplier thread restocking
        {
            let inv = Arc::clone(&inventory);
            let handle = thread::spawn(move || {
                thread::sleep(Duration::from_millis(80)); // Let some usage happen first

                println!("     🚚 Supplier: Restocking low items");
                inv.restock_ingredient("Flour", 25);
                inv.restock_ingredient("Tomatoes", 15);
                inv.restock_ingredient("Cheese", 20);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        let (items, alerts, usage_tracked) = inventory.get_inventory_status();
        println!("     📊 Inventory status: {} items, {} alerts, {} usage tracked",
                 items, alerts, usage_tracked);
    }

    demonstrate_inventory_management();

    // Application 3: Customer Service Coordination
    println!("\n3️⃣ Customer Service Coordination System:");

    #[derive(Debug, Clone)]
    enum ServiceRequest {
        TableService { table: u32, request: String },
        Complaint { table: u32, issue: String, priority: u32 },
        BillRequest { table: u32 },
        Reservation { name: String, party_size: u32, time: String },
    }

    struct ServiceCoordinator {
        request_queue: Arc<Mutex<VecDeque<ServiceRequest>>>,
        active_staff: Arc<AtomicUsize>,
        requests_processed: Arc<AtomicUsize>,
        high_priority_queue: Arc<Mutex<VecDeque<ServiceRequest>>>,
        is_busy_period: Arc<AtomicBool>,
    }

    impl ServiceCoordinator {
        fn new() -> Self {
            ServiceCoordinator {
                request_queue: Arc::new(Mutex::new(VecDeque::new())),
                active_staff: Arc::new(AtomicUsize::new(0)),
                requests_processed: Arc::new(AtomicUsize::new(0)),
                high_priority_queue: Arc::new(Mutex::new(VecDeque::new())),
                is_busy_period: Arc::new(AtomicBool::new(false)),
            }
        }

        fn submit_request(&self, request: ServiceRequest) {
            match &request {
                ServiceRequest::Complaint { priority, .. } if *priority > 7 => {
                    self.high_priority_queue.lock().unwrap().push_back(request.clone());
                    println!("     🚨 High priority request queued: {:?}", request);
                }
                _ => {
                    self.request_queue.lock().unwrap().push_back(request.clone());
                    println!("     📝 Service request queued: {:?}", request);
                }
            }

            // Update busy status based on queue length
            let queue_len = self.request_queue.lock().unwrap().len() +
                           self.high_priority_queue.lock().unwrap().len();

            self.is_busy_period.store(queue_len > 5, Ordering::SeqCst);
        }

        fn process_requests(&self, staff_id: u32) {
            self.active_staff.fetch_add(1, Ordering::SeqCst);
            println!("     👩‍💼 Staff member {} started shift", staff_id);

            for _ in 0..3 { // Each staff member handles up to 3 requests
                // Check high priority queue first
                let request = {
                    let mut high_priority = self.high_priority_queue.lock().unwrap();
                    if let Some(req) = high_priority.pop_front() {
                        Some(req)
                    } else {
                        self.request_queue.lock().unwrap().pop_front()
                    }
                };

                if let Some(req) = request {
                    println!("     🔧 Staff {} handling: {:?}", staff_id, req);

                    // Simulate service time
                    let service_time = match req {
                        ServiceRequest::Complaint { .. } => 100,
                        ServiceRequest::BillRequest { .. } => 30,
                        _ => 50,
                    };

                    thread::sleep(Duration::from_millis(service_time));
                    self.requests_processed.fetch_add(1, Ordering::SeqCst);

                    println!("     ✅ Staff {} completed request", staff_id);
                } else {
                    thread::sleep(Duration::from_millis(20)); // Wait for more requests
                }
            }

            self.active_staff.fetch_sub(1, Ordering::SeqCst);
            println!("     👋 Staff member {} finished shift", staff_id);
        }

        fn get_status(&self) -> (usize, usize, usize, bool) {
            let regular_queue = self.request_queue.lock().unwrap().len();
            let priority_queue = self.high_priority_queue.lock().unwrap().len();
            let active = self.active_staff.load(Ordering::SeqCst);
            let processed = self.requests_processed.load(Ordering::SeqCst);
            let busy = self.is_busy_period.load(Ordering::SeqCst);

            (regular_queue, priority_queue, active, busy)
        }
    }

    fn demonstrate_service_coordination() {
        println!("   👥 Thread-safe customer service coordination:");

        let coordinator = Arc::new(ServiceCoordinator::new());
        let mut handles = vec![];

        // Customer threads submitting requests
        {
            let coord = Arc::clone(&coordinator);
            let handle = thread::spawn(move || {
                let requests = vec![
                    ServiceRequest::TableService { table: 1, request: "More water".to_string() },
                    ServiceRequest::Complaint { table: 2, issue: "Cold food".to_string(), priority: 8 },
                    ServiceRequest::BillRequest { table: 3 },
                    ServiceRequest::Reservation {
                        name: "Johnson".to_string(),
                        party_size: 4,
                        time: "7:00 PM".to_string()
                    },
                    ServiceRequest::TableService { table: 4, request: "Menu change".to_string() },
                ];

                for req in requests {
                    coord.submit_request(req);
                    thread::sleep(Duration::from_millis(20));
                }
            });
            handles.push(handle);
        }

        // Staff threads processing requests
        for staff_id in 0..3 {
            let coord = Arc::clone(&coordinator);
            let handle = thread::spawn(move || {
                coord.process_requests(staff_id);
            });
            handles.push(handle);
        }

        for handle in handles {
            handle.join().unwrap();
        }

        let (regular, priority, active, busy) = coordinator.get_status();
        println!("     📊 Service status: {} regular, {} priority, {} active, busy: {}",
                 regular, priority, active, busy);
    }

    demonstrate_service_coordination();

    println!("\n🎯 Real-World Thread Safety Benefits:");
    println!("   🏗️ Multi-threaded systems handle concurrent operations safely");
    println!("   📊 Shared state remains consistent across all threads");
    println!("   ⚡ High performance through parallel processing");
    println!("   🛡️ Data integrity maintained under heavy load");
    println!("   📈 Scalable architecture supporting growth");

    println!("\n💡 Professional Implementation Guidelines:");
    println!("   🎯 Choose appropriate synchronization primitives for each use case");
    println!("   📋 Design thread-safe APIs from the ground up");
    println!("   🔍 Test concurrent code thoroughly under realistic loads");
    println!("   ⚖️ Balance thread safety with performance requirements");
    println!("   🏢 Build systems that scale gracefully with demand");
}

fn main() {
    demonstrate_real_world_thread_safety();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Team Coordination System**

Remember our comprehensive professional restaurant team coordination analogy:

- 🛡️ **Thread safety** = **Professional coordination protocols** - Systems that allow multiple teams to work simultaneously without conflicts
- 🏷️ **Send/Sync traits** = **Team capability classification** - Clear understanding of which team members can work where and how
- 🔧 **Synchronization primitives** = **Coordination tools** - Professional equipment for managing team interactions safely
- 📡 **Message passing** = **Communication systems** - Structured ways for teams to coordinate without direct conflicts
- 💼 **Real-world applications** = **Complete operational systems** - Professional implementations handling complex multi-team scenarios


### **Thread Safety Fundamentals**

**Core Principles:**

- **Data Race Prevention** - Ensuring no two threads access mutable data simultaneously without synchronization
- **Memory Safety** - Maintaining Rust's ownership guarantees across thread boundaries
- **Deadlock Prevention** - Designing coordination systems that don't create circular waiting
- **Performance Optimization** - Achieving thread safety without unnecessary overhead
- **Scalability** - Building systems that work reliably as concurrency increases


### **Send and Sync Traits Matrix**

| **Type Category** | **Send** | **Sync** | **Use Case** | **Thread Safety Level** |
| :-- | :-- | :-- | :-- | :-- |
| **Basic Types** | ✅ | ✅ | `i32`, `String`, `Vec<T>` | Full thread safety |
| **Smart Pointers** | `Arc<T>`: ✅ | `Arc<T>`: ✅ | Shared ownership | Thread-safe sharing |
|  | `Rc<T>`: ❌ | `Rc<T>`: ❌ | Single-threaded only | Not thread-safe |
| **Synchronization** | `Mutex<T>`: ✅ | `Mutex<T>`: ✅ | Exclusive access | Protected data |
|  | `RwLock<T>`: ✅ | `RwLock<T>`: ✅ | Read/write access | Efficient sharing |
| **Interior Mutability** | `Cell<T>`: ✅ | `Cell<T>`: ❌ | Single-threaded mutation | Local use only |
|  | `RefCell<T>`: ✅ | `RefCell<T>`: ❌ | Runtime borrow checking | Local use only |

### **Synchronization Primitive Selection Guide**

**🔒 Mutex - When to Use:**

- Exclusive access to mutable data
- Simple read/write operations
- Protecting critical sections
- Single-writer scenarios

**📚 RwLock - When to Use:**

- Many readers, few writers
- Read-heavy workloads
- Data that's frequently accessed but rarely modified
- Performance-critical read operations

**📡 Channels - When to Use:**

- Message passing between threads
- Producer-consumer patterns
- Decoupling thread communication
- Complex coordination sequences

**⚛️ Atomic Types - When to Use:**

- Simple counters and flags
- Lock-free programming
- High-performance scenarios
- Basic coordination primitives


### **Best Practices Checklist**

**✅ Design Guidelines:**

- Choose the right synchronization primitive for each use case
- Minimize shared mutable state when possible
- Design APIs to be thread-safe by default
- Use message passing to reduce synchronization complexity
- Plan for deadlock prevention from the start

**✅ Implementation Guidelines:**

- Always handle lock acquisition failures appropriately
- Keep critical sections as small as possible
- Use RAII for automatic resource cleanup
- Test concurrent code under realistic loads
- Document thread safety requirements clearly

**✅ Performance Guidelines:**

- Profile concurrent code to identify bottlenecks
- Consider lock-free alternatives for hot paths
- Use atomic operations for simple shared state
- Balance safety with performance requirements
- Design for scalability from the beginning

**❌ Common Thread Safety Pitfalls:**

- Sharing non-thread-safe types (`Rc`, `RefCell`) between threads
- Creating deadlocks through improper lock ordering
- Using too much synchronization and causing contention
- Not handling lock poisoning appropriately
- Assuming single-threaded code will work in multi-threaded contexts


### **The Professional Advantage**

**Mastering thread safety concepts in Rust is like having a complete professional restaurant team coordination system** that enables multiple teams to work simultaneously at maximum efficiency while maintaining safety and consistency:

- 🛡️ **Compile-time guarantees** - Thread safety verified before deployment, preventing runtime failures
- ⚡ **Zero-cost abstractions** - Safety achieved without performance penalties
- 📊 **Scalable architecture** - Systems that perform well from small to enterprise scale
- 🔧 **Powerful tools** - Rich ecosystem of synchronization primitives for any scenario
- 💼 **Professional reliability** - Production-ready systems with predictable, safe behavior

**Understanding thread safety in Rust transforms you from a programmer who avoids concurrency to an architect** who builds efficient, reliable multi-threaded systems. Just as a master restaurant manager can coordinate complex operations involving dozens of team members working simultaneously without conflicts or safety issues, a skilled Rust programmer can leverage thread safety concepts to create high-performance concurrent applications that maintain data integrity and system stability under any load.

This comprehensive understanding of thread safety concepts - from fundamental Send/Sync traits through advanced synchronization patterns and real-world implementations - enables you to build Rust applications that fully utilize modern multi-core systems while maintaining the safety and reliability that make Rust exceptional for systems programming, whether you're building simple concurrent utilities or complex enterprise systems requiring sophisticated thread coordination!

1. https://dzone.com/articles/rust-threading-concurrent-programming
2. https://onesignal.com/blog/thread-safety-rust/
3. https://www.rustfinity.com/blog/what-is-thread-safety
4. https://blog.reverberate.org/2021/12/18/thread-safety-cpp-rust.html
5. https://www.ardanlabs.com/blog/2024/09/fearless-concurrency-ep1-rusts-approach-to-safe-and-manageable-multithreading.html
6. https://www.ardanlabs.com/blog/2024/10/fearless-concurrency-ep4-understanding-mutexes-and-thread-safety-in-rust.html
7. https://www.pullrequest.com/blog/rust-safety-writing-secure-concurrency-without-fear/
8. https://cratecode.com/info/thread-safety-in-rust
9. https://akilmohideen.github.io/java-rust-bindings-manual/cha03-04.html
10. https://doc.rust-lang.org/book/ch16-00-concurrency.html
11. https://doc.rust-lang.org/book/ch16-01-threads.html
12. https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html
13. https://earthly.dev/blog/rust-concurrency-patterns-parallel-programming/
14. https://www.youtube.com/watch?v=6Qg84mkSQv8
15. https://www.reddit.com/r/rust/comments/18faxjg/understanding_threadsafety_vs_race_conditions/
16. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/concurrency.html
17. https://doc.rust-lang.org/book/ch16-03-shared-state.html
