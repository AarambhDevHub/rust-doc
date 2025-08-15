+++
title = "Memory Leak Prevention"
description = "Best practices for preventing memory leaks and managing resources safely in Rust."
date = "2025-08-13"
weight = 5
+++

# Memory Leak Prevention in Rust: Comprehensive Documentation for Beginners

Understanding memory leak prevention in Rust is like learning to **design comprehensive resource management systems for your professional restaurant chain** - you need systems that automatically track, use, and return all resources (ingredients, equipment, staff time) without anything being forgotten or wasted indefinitely. Just as a professional restaurant manager must ensure that every ingredient is either used in dishes or properly disposed of, and every piece of equipment is returned to its proper place after use, Rust's ownership system ensures that every piece of memory is either actively used or automatically cleaned up, preventing the accumulation of unused resources that could eventually overwhelm your system.

## The Professional Restaurant Resource Management System Analogy 🏪

### Imagine You're Designing Complete Resource Tracking for Your Restaurant Chain

**The Problem without Systematic Resource Management:**

```rust
// ❌ Poor resource management - like losing track of expensive ingredients
// Resources get allocated but never properly returned or cleaned up
// Over time, this leads to waste, inefficiency, and eventual system failure
```

**The Power of Rust's Ownership System - Professional Resource Management:**

```rust
// ✅ Systematic resource management - every resource has a clear owner
// When the owner is done, resources are automatically returned/cleaned up
// Built-in safeguards prevent resource accumulation and waste

fn professional_resource_management() {
    let expensive_ingredient = acquire_resource("Wagyu Beef"); // Resource acquired
    use_in_recipe(expensive_ingredient); // Resource used
    // Resource automatically cleaned up when owner goes out of scope
} // ✅ No resource left behind!
```

**Why Comprehensive Memory Leak Prevention Is Critical:**

- 🛡️ **System stability** - Prevents gradual resource exhaustion over time
- ⚡ **Performance maintenance** - Keeps systems running efficiently long-term
- 💰 **Cost control** - Prevents waste of valuable computational resources
- 🔄 **Sustainable operations** - Enables systems to run indefinitely without degradation
- 🎯 **Predictable behavior** - Eliminates mysterious slowdowns and crashes


## Understanding Rust's Memory Leak Prevention Fundamentals

### The Automatic Resource Return System

**Rust's ownership system prevents most memory leaks through automatic resource management:**

```rust
fn demonstrate_memory_leak_prevention_fundamentals() {
    println!("🛡️ Memory Leak Prevention Fundamentals - Automatic Resource Management");
    println!("{:=<75}", "");

    // Rust's memory management is like having an automated resource tracking system
    println!("📋 Rust's Memory Leak Prevention Mechanisms:");
    println!("   🏠 Ownership System - Every value has exactly one responsible owner");
    println!("   📏 Automatic Cleanup - Resources freed when owner goes out of scope");
    println!("   🔒 RAII Pattern - Resource Acquisition Is Initialization");
    println!("   🎯 Drop Trait - Custom cleanup for complex resources");
    println!("   ⚡ Zero Runtime Cost - All management happens at compile time");

    // Example 1: Basic Automatic Memory Management
    println!("\n1️⃣ Basic Automatic Memory Management:");

    fn demonstrate_automatic_cleanup() {
        println!("   🔄 Automatic Resource Lifecycle:");

        {
            let restaurant_data = String::from("Expensive menu analysis data");
            println!("     📊 Resource acquired: {} bytes", restaurant_data.len());

            // Resource is actively used
            let processed_data = format!("Processed: {}", restaurant_data);
            println!("     ⚙️ Resource in use: {}", &processed_data[0..20]);

        } // ← Both restaurant_data and processed_data automatically cleaned up here

        println!("     ✅ Resources automatically returned to system");
        println!("     💡 No manual cleanup required - zero memory leak risk!");
    }

    demonstrate_automatic_cleanup();

    // Example 2: Heap Memory Management
    println!("\n2️⃣ Heap Memory Automatic Management:");

    fn demonstrate_heap_management() {
        println!("   📦 Heap allocation and automatic cleanup:");

        {
            // Allocate expensive resources on heap
            let menu_items = vec![
                "Wagyu Steak".to_string(),
                "Lobster Thermidor".to_string(),
                "Truffle Pasta".to_string(),
            ];

            let customer_data = Box::new(format!("Customer preferences: {:?}", menu_items));

            println!("     💰 Expensive heap resources allocated");
            println!("     📊 Menu items: {} entries", menu_items.len());
            println!("     👤 Customer data: {} chars", customer_data.len());

            // Resources actively used for business logic
            for (i, item) in menu_items.iter().enumerate() {
                println!("       {}: {}", i + 1, item);
            }

        } // ← All heap memory automatically freed here

        println!("     ✅ All heap memory returned to system automatically");
        println!("     🛡️ Zero risk of heap memory leaks in normal operation");
    }

    demonstrate_heap_management();

    // Example 3: Resource Transfer and Ownership
    println!("\n3️⃣ Resource Transfer and Ownership:");

    #[derive(Debug)]
    struct ExpensiveResource {
        resource_id: String,
        data: Vec<String>,
    }

    impl ExpensiveResource {
        fn new(id: &str) -> Self {
            ExpensiveResource {
                resource_id: id.to_string(),
                data: vec!["Important".to_string(), "Business".to_string(), "Data".to_string()],
            }
        }

        fn process(&self) -> String {
            format!("Processing {} with {} items", self.resource_id, self.data.len())
        }
    }

    // Function that takes ownership and automatically cleans up
    fn use_resource(resource: ExpensiveResource) -> String {
        let result = resource.process();
        println!("     🔄 Using resource: {}", result);
        result
        // resource automatically cleaned up when function ends
    }

    // Function that borrows without taking ownership
    fn borrow_resource(resource: &ExpensiveResource) -> String {
        resource.process()
        // resource remains owned by caller
    }

    {
        let expensive_resource = ExpensiveResource::new("MENU_ANALYSIS_001");
        println!("     📊 Resource created: {}", expensive_resource.resource_id);

        // Borrow the resource (no ownership transfer)
        let analysis_result = borrow_resource(&expensive_resource);
        println!("     📋 Analysis result: {}", analysis_result);

        // Transfer ownership (resource will be cleaned up in function)
        let final_result = use_resource(expensive_resource);
        // expensive_resource is no longer accessible here - moved to function

        println!("     ✅ Final result: {}", final_result);
    }

    println!("     🛡️ Resource lifecycle completely managed by ownership system");

    // Example 4: Custom Drop Implementation
    println!("\n4️⃣ Custom Resource Cleanup with Drop Trait:");

    struct DatabaseConnection {
        connection_id: String,
        is_connected: bool,
    }

    impl DatabaseConnection {
        fn new(id: &str) -> Self {
            println!("     🔌 Opening database connection: {}", id);
            DatabaseConnection {
                connection_id: id.to_string(),
                is_connected: true,
            }
        }

        fn query(&self, sql: &str) -> String {
            if self.is_connected {
                format!("Executing '{}' on connection {}", sql, self.connection_id)
            } else {
                "Connection closed".to_string()
            }
        }
    }

    // Custom cleanup when resource goes out of scope
    impl Drop for DatabaseConnection {
        fn drop(&mut self) {
            if self.is_connected {
                println!("     🔒 Closing database connection: {}", self.connection_id);
                self.is_connected = false;
            }
        }
    }

    {
        let db_connection = DatabaseConnection::new("RESTAURANT_DB_001");
        let query_result = db_connection.query("SELECT * FROM menu_items");
        println!("     📊 Query result: {}", query_result);

    } // ← Custom Drop automatically called here

    println!("     ✅ Database connection automatically closed by Drop trait");

    println!("\n🎯 Rust's Memory Leak Prevention Benefits:");
    println!("   🏠 Ownership ensures every resource has exactly one responsible manager");
    println!("   ⚡ Automatic cleanup happens at compile-time determined points");
    println!("   🔒 RAII pattern guarantees resource cleanup even during errors");
    println!("   🎯 Custom Drop trait enables complex resource management");
    println!("   🛡️ Zero runtime overhead for memory safety guarantees");
}

fn main() {
    demonstrate_memory_leak_prevention_fundamentals();
}
```


## Common Memory Leak Scenarios and Prevention

![Common Causes of Memory Leaks in Rust with Examples and Prevention](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAACWAAAAZACAYAAAD9qXmxAAAgAElEQVR4XuzdB5QUVdrG8VdBVHIGFZAokgUkCAgiQREJkiRJjiM5g+ScM0OOEh3JAkoQlCQqGclBQJScg6i433l7v2En3Kqu7umZnoH/PWfPnrN9761bv6qunj318N5n7ty5/h+hIYAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIeCzwDAEsj80YgAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAgi4BAhgcSMggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAl4KEMDyEo5hCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAABLO4BBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQMBLAQJYXsIxDAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBAggMU9gAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAgh4KUAAy0s4hiGAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACBLC4BxBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABLwUIYHkJxzAEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAgAAW9wACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggg4KUAASwv4RiGAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCBDA4h5AAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBLwUIIDlJRzDEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAECWNwDCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggICXAgSwvIRjGAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCBAAIt7AAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBDwUoAAlpdwDEMAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEECGBxDyCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACXgoQwPISjmEIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAEs7gEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAwEsBAlhewjEMAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEECCAxT2AAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCHgpQADLSziGIYAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIEsLgHEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEvBQhgeQnHMAQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEECAABb3AAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCDgpQABLC/hGIYAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIEMDiHkAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEvBQggOUlHMMQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQJY3AMIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAgJcCBLC8hGMYAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIEAAi3sAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEPBSgACWl3AMQwABBBBAAAEEEEAA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QAABBBBAAAEEEEAAAQQQQAABBBBAAAEEECCAxT2AAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCHgpQADLSziGIYAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIEsLgHEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEvBQhgeQnHMAQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEEAAAQQQQAABBBBAAAEEEECAABb3AAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCCAAAIIIIAAAggggAACCCDgpQABLC/hGIYAAggggAACCCCAAAIIIIAAAggggMDTJHDj1h25dO2a3Ll3X27fvSexnn1WXnghjiRPnEheSplC4r344tPEwbkigAACCCCAAAIIIIAAAggggAACjwUIYHEzIIAAAggggAACCCDwlAv8+fAv+f6n3T5TePbZWBIv7guSNFEiSftSKon7wgs+m5uJ/COwbfdeuf/gT9uDF8mbR+LHixvpCzx+5pz8euGC7XEypn1FMr+aLtLXwgEQ8LfALydOy4VLl4zLiBc3rhTNl8ffS3xij3/vwQPZvnuf5fmVequgxIoVK8af/6Wr1+TrrTvlux93y+FTZ+TilWu256S/+3mzZZWShQtI6SIFJEG8eDHegBNAIFjg1wt/yPEzv0YrkCL58kj8uJH/91fwSbt79hUvkF9eeD5OlBgdOHZCLl65ajzWK6lSSY4sGaNkHRwEAQQQQAABBBBAAAEEEAgWIIDFvYAAAggggAACCCCAwFMu8MflK/LWx40iTSFj2jRSKE8OKVe8iCsM8CS8kI40rGg6cYk6TeXs7xdtV9enVVNpWLVipJ9B+WZtRUMndu3TujWkc+NPIn0tHAABfwt0GT5evli3wbiMLOnTyYbZk/y9xCf2+Gd++11KftLc8vyOfbNUno8TNSGEyEDesfeATJr/hWzfs9/r6TWA/VGZktKydjVJkzql1/MwEIHoIjAzaKUMCJwRXZbjWsemuYGSKV3aKFuTu2ffnhXzXf8IIypau0GjZMXGLcZD1ShXRoZ3aRMVy+AYCCCAAAIIIIAAAggggMBjAQJY3AwIIIAAAggggAACCDzlApEdwArJ+0qq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gAABAgQIECBAgAABAgQyAQEsDwIBAgQIECBAgAABAgQIECBQ1QKtCWDV1NTE6667Mj794fdFj27dqtpL4wkQIECAAAECBAgQIECAAAECBAgQOFRAAMsTQYAAAQIECBAgQIAAAQIECFS1wOEBrNra2ujds0f079snJo0ZFaedcnK89porY8xJw6vaSeMJECBAgAABAgQIECBAgAABAgQIEGheQADLk0GAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGcAgJYOeFUI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgACWZ4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQI5BQSwcsKpRoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQEszwABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRyCghg5YRTjQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYngECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjkFBDAygmnGgECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBASwPAMECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIKSCAlRNONQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAhgeQYIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQU0AAKyecagQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEBDA8gwQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgp4AAVk441QgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCA5RkgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBATgEBrJxwqhEgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQEAAyzNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBnAICWDnhVCNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAlmeAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECOQUEsHLCqUaAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAEBLM8AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEcgoIYOWEU40AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWJ4BAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI5BQQwMoJpxoBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQEsDwDBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyCkggJUTTjUCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIYHkGCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkFNAACsnnGoECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAQwPIMECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIKeAAFZOONUIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggOUZIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQE4BAayccKoRIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEBAAMszQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgZwCAlg54VQjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAJZngAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAjkFBLBywqlGgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABASzPAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBHIKCGDlhFONAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlieAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECOQUEMDKCacaAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEBLA8AwQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMgpIICVE041AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICGB5BggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBTQAArJ5xqBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQEMDyDBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCngABWTjjVCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIDlGSBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBOAQGsnHCqESBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAQADLM0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGcAgJYOeFUI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgACWZ4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQI5BQSwcsKpRoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQEszwABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRyCghg5YRTjQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYngECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjkFBDAygmnGgECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBASwPAMECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIKSCAlRNONQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAhgeQYIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQU0AAKyecagQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEBDA8gwQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgp4AAVk441QgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCA5RkgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBATgEBrJxwqhEgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQEAAyzNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBnAICWDnhVCNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAlmeAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECOQUEsHLCqUaAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAEBLM8AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEcgoIYOWEU40AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWJ4BAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI5BQQwMoJpxoBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQEsDwDBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyCkggJUTTjUCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIYHkGCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkFNAACsnnGoECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAQwPIMECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIKeAAFZOONUIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggOUZIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQE4BAayccKoRIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEBAAMszQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgZwCAlg54VQjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAJZngAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAjkFBLBywqlGgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABASzPAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBHIKCGDlhFONAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlieAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECOQUEMDKCacaAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEBLA8AwQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMgpIICVE041AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICGB5BggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBTQAArJ5xqBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQEMDyDBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCngABWTjjVCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIDlGSBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBOAQGsnHCqESBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAQADLM0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGcAgJYOeFUI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgACWZ4AAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQI5BQSwcsKpRoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQEszwABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRyCghg5YRTjQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYngECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjkFBDAygmnGgECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBASwPAMECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIKSCAlRNONQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAhgeQYIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQU0AAKyecagQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEBDA8gwQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgp4AAVk441QgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCA5RkgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBATgEBrJxwqhEgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQEAAyzNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBnAICWDnhVCNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAlmeAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECOQUEsHLCqUaAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAEBLM8AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEcgoIYOWEU40AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWJ4BAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQI5BQQwMoJpxoBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQEsDwDBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQyCkggJUTTjUCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIYHkGCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkFNAACsnnGoECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAQwPIMECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAIKeAAFZOONUIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggOUZIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAQE4BAayccKoRIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEBAAMszQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAgZwCAlg54VQjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAJZngAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAjkFBLBywqlGgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABASzPAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBHIKCGDlhFONAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlieAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECOQUEMDKCacaAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEBLA8AwQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEMgpIICVE041AgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICGB5BggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJBTQAArJ5xqBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQEMDyDBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQCCngABWTjjVCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIDlGSBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgEBOAQGsnHCqESBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAQADLM0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAIGcAgJYOeFUI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgACWZxrymTsAACAASURBVIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQI5BQSwcsKpRoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQEszwABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgRyCghg5YRTjQABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYngECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAjkFBDAygmnGgECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBASwPAMECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBDIKSCAlRNONQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAhgeQYIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQU0AAKyecagQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIEBDA8gwQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIEAgp4AAVk441QgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCA5RkgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIBATgEBrJxwqhEgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQEAAyzNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBnAICWDnhVCNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAlmeAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECOQUEsHLCqUaAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAEBLM8AAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIEcgoIYOWEU639Al/4wlfbfxJnIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhExA03fJADgQ4REMDqEHYXTQICWJ4DAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAoFwCAljlknSetgoIYLVVzPFlEygFsL5z/ivLdk4nIkCAAAECBAgQIHAiBd7x4M3Z5YxpT6S6axEgQIAAAQIECJRbwLi23KLOR4AAAQIECBAgcKIFSmNaAawTLe96JQEBLM9ChwkIYHUYvQsTIECAAAECBAiUScBEVZkgnYYAAQIECBAgQKBDBYxrO5TfxQkQIECAAAECBMogIIBVBkSnaJeAAFa7+FRuj4AAVnv01CVAgAABAgQIEOgMAiaqOkMvuAcCBAgQIECAAIH2ChjXtldQfQIECBAgQIAAgY4WEMDq6B5wfQEsz0CHCQhgdRi9CxMgQIAAAQIECJRJwERVmSCdhgABAgQIECBAoEMFjGs7lN/FCRAgQIAAAQIEyiAggFUGRKdol4AAVrv4VG6PgABWe/TUJUCAAAECBAgQ6AwCJqo6Qy+4BwIECBAgQIAAgfYKGNe2V1B9AgQIECBAgACBjhYQwOroHnB9ASzPQIcJCGB1GL0LEyBAgAABAgQI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Vl34+HA9BCjAqocenyUBEiABEiABEiABEmgGAnRUNcMpcA0kQAIkQAIkQAIkQAL1EqBdWy9BPk8CJEACJEACJEACJNBoAhRgNfoEOD8FWHwHGkaAAqyGoefEJEACJEACJEACJEACARGgoyogkByGBEiABEiABEiABEigoQRo1zYUPycnARIgARIgARIgARIIgAAFWAFA5BB1EaAAqy58fLgeAhRg1UOPz5IACZAACZAACZAACTQDATqqmuEUuAYSIAESIAESIAESIIF6CdCurZcgnycBEiABEiABEiABEmg0AQqwGn0CnJ8CLL4DDSNAAVbD0HNiEiABEiABEiABEiCBgAjQURUQSA5DAiRAAiRAAiRAAiTQUAK0axuKn5OTAAmQAAmQAAmQAAkEQIACrAAgcoi6CFCAVRc+PlwPAQqw6qHHZ0mABEiABEiABEiABJqBAB1VzXAKXAMJkAAJkAAJkAAJkEC9BGjX1kuQz5MACZAACZAACZAACTSaAAVYjT4Bzk8BFt+BhhGgAKth6DkxCZAACZAACZAACZBAQAToqAoIJIchARIgARIgARIgARJoKAHatQ3Fz8lJgARIgARIgARIgAQCIEABVgAQOURdBCjAqgsfH66HAAVY9dDjsyRAAiRAAiRAAiRAAs1AgI6qZjgFroEESIAESIAESIAESKBeArRr6yXI50mABEiABEiABEiABBpNgAKsRp8A56cAi+9AwwhQgNUw9JyYBEiABEiABEiABEggIAJ0VAUEksOQAAmQAAmQAAmQAAk0lADt2obi5+QkQAIkQAIkQAIkQAIBEKAAKwCIHKIuAhRg1YWPD9dDgAKseujxWRIgARIgARIgARIggWYgQEdVM5wC10ACJEACJEACJEACJFAvAdq19RLk8yRAAiRAAiRAAiRAAo0mQAFWo0+A81OAxXegYQQowGoYek5MAiRAAiRAAiRAAiQQEAE6qgICyWFIgARIgARIgARIgAQaSoB2bUPxc3ISIF5vNmUAACAASURBVAESIAESIAESIIEACFCAFQBEDlEXAQqw6sLHh+shQAFWPfT4LAmQAAmQAAmQAAmQQDMQoKOqGU6BayABEiABEiABEiABEqiXAO3aegnyeRIgARIgARIgARIggUYToACr0SfA+SnA4jvQMAIUYDUMPScmARIgARIgARIgARIIiAAdVQGB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L4E///M/y9/BcWTzmgAFWPN6efN7chRg5ff6cHQkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkQAIkMJcIUIA1l1Zrfo2VAqz5tZ5zajaaAOu5rWxOjZuDJQESIAESIAESIAESIAGNwDLvkPonbVq+EyRAAiRAAiRAAiRAAnOZAO3aubx6HDsJkAAJkAAJkAAJkIAQ0GxaCrD4PuSKAAVYuSLP54ICLL4EJEACJEACJEACJEACc50AHVVzfQU5fhIgARIgARIgARIggVhnFQML+D6QAAmQAAmQAAmQAAnMVQIUYM3VlZs/46YAa/6s5ZybCQVYc27JOGASIAESIAESIAESIIE4AhRg8ZUgARIgARIgARIgARKYDwRo186HVeQcSIAESIAESIAESKCwCVCAVdjrnw+zpwArH1ahQMdAAVaBLjynTQIkQAIkQAIkQALziAAdVfNoMTkVEiABEiABEiABEihgArRrC3jxOXUSIAESIAESIAESmCcEKMCaJws5h6dBAdYcXry5PnQKsOb6CnL8JEACJEACJEACJEACdFTxHSABEiABEiABEiABEpgPBGjXzodV5BxIgARIgARIgARIoLAJUIBV2OufD7OnACsfVqFAx0ABVoEuPKdNAiRAAiRAAiRAAvOIAB1V82gxORUSIAESIAESIAESKGACtGsLePE5dRIgARIgARIgARKYJwQowJonCzmHp0EB1hxevLk+dAqw5voKcvwkQAIkQAIkQAIkQAJ0VPEdIAESIAESIAESIAESmA8EaNfOh1XkHEiABEiABEiABEigsAlQgFXY658Ps6cAKx9WoUDHQAFWgS48p00CJEACJEACJEAC84gAHVXzaDE5FRIgARIgARIgARIoYAK0awt48Tl1EiABEiABEiABEpgnBCjAmicLOYenQQHWHF68uT50CrDm+gpy/CRAAiRAAiRAAiRAAnRU8R0gARIgARIgARIgARKYDwRo186HVeQcSIAESIAESIAESKCwCVCAVdjrnw+zpwArH1ahQMdAAVaBLjynTQIkQAIkQAIkQALziAAdVfNoMTkVEiABEiABEiABEihgArRrC3jxOXUSIAESIAESIAESmCcEKMCaJws5h6dBAdYcXry5PnQKsOb6CnL8JEACJEACJEACJEACdFTxHSABEiABEiABEiAB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sHSAyC4yI0ABVmbceBcJkAAJkAAJkAAJkED+EKCjKn/WgiMhARIgARIgARIgARLInADt2szZ8U4SIAESIAESIAESIIH8IEABVn6sQyGPggKsQl79HM+dAqwcLwAfTwIkQAIkQAIkQAIkkDUBOqqyRsgOSIAESIAESIAESIAE8oAA7do8WAQOgQRIgARIgARIgARIICsCFGBlhY8360CAAiwdILKLzAhQgJUZN95FAiRAAiRAAiRAAiSQPwToqMqfteBISIAESIAESIAESIAEMidAuzZzdryTBEiABEiABEiABEggPwhQgJUf61DIo6AAq5BXP8dzpwArxwvAx5MACZAACZAACZAACWRNgI6qrBGyAxIgARIgARIgARIggTwgQLs2DxaBQyABEiABEiABEiABEsiKAAVYWeHjzToQoABLB4jsIjMCFGBlxo13kQAJkAAJkAAJkAAJ5A8BOqryZy04EhIgARIgARIgARIggcwJ0K7NnB3vJAESIAESIAESIAESyA8CFGDlxzoU8igowCrk1c/x3CnAyvEC8PEkQAIkQAIkQAIkQAJZE6CjKmuE7IAESIAESIAESIAESCAPCNCuzYNF4BBIgARIgARIgARIgASyIkABVlb4eLMOBCjA0gEiu8iMAAVYmXHjXSRAAiRAAiRAAiRAAvlDgI6q/FkLjoQESIAESIAESIAESCBzArRrM2fHO0mABEiABEiABEiABPKDAAVY+bEOhTwKCrAKefVzPHcKsHK8AHw8CZAACZAACZAACZBA1gToqMoaITsgARIgARIgARIgARLIAwK0a/NgETgEEiABEiABEiABEiCBrAhQgJUVPt6sAwEKsHSAyC4yI0ABVmbceBcJkAAJkAAJkAAJkED+EKCjKn/WgiMhARIgARIgARIgARLInADt2szZ8U4SIAESIAESIAESIIH8IEABVn6sQyGPggKsQl79HM+dAqwcLwAfTwIkQAIkQAIkQAIkkDUBOqqyRsgOSIAESIAESIAESIAE8oAA7do8WAQOgQRIgARIgARIgARIICsCFGBlhY8360CAAiwdILKLzAhQgJUZN95FAiRAAiRAAiRAAiSQPwToqMqfteBISIAESIAESIAESIAEMidAuzZzdryTBEiABEiABEiABEggPwhQgJUf61DIo6AAq5BXP8dzpwArxwvAx5MACZAACZAA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BKafgADW9Lvmk+2MBbAm2xWZRscjgDV5L7YA1uS9No6MAAECBAgQmFwCpT5RlappAidVbuzrXIuxN/WJBAgQIEBgOgqU+rhWAGv4rjVmnI4/1c6ZAAECBAhMPwEBrOl3zSfbGQtgTbYrMo2OpzeANY1OuWROtbm9Pb53/y/zx/sv7986Vpo9s2SOfagD/cvLf4975/wxqioq4qidP1Ty5+MECBAgQIAAAQITKWBsNZH6i363azF5roUjIUCAAAECBCZO4EcPPhJvNjXH+zZcLzZbZ82JO5BJ+s3GjJP0wjgsAgQIECBAYFwETj75+HH5XB9KYCQBAayRhPz7uAkIYI0b7RJ/sADWEhP6AAIECBAgQIDAlBYwgTN5Lq9rMXmuhSMhQIAAAQIEJk5AAGt4e2PGietN30yAAAECBAgsfQEBrKVv7hsLAgJYOoEAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAK1BRiQAACAASURBVAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAA/uALJgAAIABJREFUAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAggKUHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgkCgggJUIp4wAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICWHqAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECiQICWIlwyggQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICCApQcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECCQKCCAlQinjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgJYeoAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQKJAgJYiXDKCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIIClBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIJAoIICVCKeMAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAlh6gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAokCAliJcMoIECBAgAABAgT+n107pAEAAGAY5t/1PAxXwElzOAIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBBUXifPAAAgAElEQVQgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAbJCuAEAACAASURBVAHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBGrXDmkAAAAYhvl3PQ/DFXDSHI4AAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAg0lF4FgAAIABJREFUQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJi35/cAAAKNUlEQVQECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFBFgTzowAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQICLB8gQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIDAFBBgTTgzAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQICLB8gAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAlNAgDXhzAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQICDA8gECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAhMAQHWhDMjQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQICAAMsHCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgMAUEWBPOjAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgIsHyBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgMAUEGBNODMCBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgIsHyAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECU0CANeHMCBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgIMDyAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECEwBAdaEMyNAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgIAAywcIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAwBQRYE86MAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECAiwfIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAwBQQYE04MwIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECAiwfIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQIECBAgQIAAAQJTQIA14cwIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECAgwPIBAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQIECBAgAABAgQITAEB1oQzI0CAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAgADLBwgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIECAAAECBAgQIDAFAnXVmk8Ga+/QAAAAAElFTkSuQmCC)

Common Causes of Memory Leaks in Rust with Examples and Prevention

**Understanding the specific scenarios where memory leaks can still occur in Rust:**

```rust
fn demonstrate_common_leak_scenarios() {
    println!("⚠️ Common Memory Leak Scenarios - Professional Risk Management");
    println!("{:=<75}", "");

    use std::rc::{Rc, Weak};
    use std::cell::RefCell;
    use std::collections::HashMap;

    // Even with Rust's safeguards, certain patterns can create memory leaks
    println!("🔍 Memory Leak Risk Scenarios:");

    // Scenario 1: Reference Cycles with Rc/Arc
    println!("\n1️⃣ Scenario: Reference Cycles - Circular Dependencies");

    println!("   ⚠️ THE PROBLEM:");
    println!("   Reference cycles prevent reference counts from reaching zero");
    println!("   Resources remain allocated because they reference each other");

    // Problematic pattern (conceptual - this would leak)
    #[derive(Debug)]
    struct Restaurant {
        name: String,
        manager: RefCell<Option<Rc<Employee>>>,
        employees: RefCell<Vec<Rc<Employee>>>,
    }

    #[derive(Debug)]
    struct Employee {
        name: String,
        restaurant: RefCell<Option<Rc<Restaurant>>>,
        subordinates: RefCell<Vec<Rc<Employee>>>,
    }

    // ❌ PROBLEMATIC: Creates reference cycle
    fn create_reference_cycle_problem() {
        println!("   🚫 Problematic pattern (would create cycle):");
        println!("
   let restaurant = Rc::new(Restaurant { ... });
   let manager = Rc::new(Employee { ... });

   // This creates a cycle:
   restaurant.manager = Some(Rc::clone(&manager));     // Restaurant → Manager
   manager.restaurant = Some(Rc::clone(&restaurant));  // Manager → Restaurant

   // Both Rc's have reference count > 0, so neither is ever dropped!
   // RESULT: Memory leak");
    }

    create_reference_cycle_problem();

    // ✅ SOLUTION: Use Weak references to break cycles
    #[derive(Debug)]
    struct SafeRestaurant {
        name: String,
        manager: RefCell<Option<Rc<SafeEmployee>>>,
    }

    #[derive(Debug)]
    struct SafeEmployee {
        name: String,
        restaurant: RefCell<Weak<SafeRestaurant>>, // Weak reference breaks cycle
    }

    fn demonstrate_cycle_prevention() {
        println!("   ✅ SOLUTION: Breaking cycles with Weak references");

        let restaurant = Rc::new(SafeRestaurant {
            name: "Ocean View Bistro".to_string(),
            manager: RefCell::new(None),
        });

        let manager = Rc::new(SafeEmployee {
            name: "Alice Johnson".to_string(),
            restaurant: RefCell::new(Rc::downgrade(&restaurant)), // Weak reference
        });

        // Set up relationship
        *restaurant.manager.borrow_mut() = Some(Rc::clone(&manager));

        println!("     Restaurant: {}", restaurant.name);
        println!("     Manager: {}", manager.name);

        // Check if manager can still access restaurant
        if let Some(rest_ref) = manager.restaurant.borrow().upgrade() {
            println!("     Manager works at: {}", rest_ref.name);
        }

        println!("     ✅ No reference cycle - memory will be properly freed");
    }

    demonstrate_cycle_prevention();

    // Scenario 2: Intentional Memory Leaks
    println!("\n2️⃣ Scenario: Intentional Memory Leaks - When You Want Static Data");

    println!("   📌 LEGITIMATE USE CASES:");
    println!("   Sometimes you actually want data to live for entire program duration");

    fn demonstrate_intentional_leaks() {
        println!("   🎯 Using `Box::leak` for static configuration:");

        // Create configuration that should live for entire program
        let config_data = Box::new(HashMap::from([
            ("restaurant_name".to_string(), "Global Gourmet".to_string()),
            ("max_capacity".to_string(), "200".to_string()),
            ("cuisine_type".to_string(), "International".to_string()),
        ]));

        // Leak it to get a static reference
        let static_config: &'static HashMap<String, String> = Box::leak(config_data);

        println!("     Configuration (static): {:?}", static_config.get("restaurant_name"));

        // This data will never be freed (intentionally)
        println!("     ⚠️ This data is intentionally leaked and will live forever");
        println!("     💡 Use only when you need true static lifetime data");
    }

    demonstrate_intentional_leaks();

    // Scenario 3: Forgotten Drop Implementations
    println!("\n3️⃣ Scenario: Missing Resource Cleanup - Forgotten Drop");

    println!("   ⚠️ THE PROBLEM:");
    println!("   Custom types that manage external resources need explicit cleanup");

    // Problematic pattern - resource not cleaned up
    struct ProblematicFileHandler {
        filename: String,
        // In real code, this might hold file handles, network connections, etc.
        resource_count: usize,
    }

    impl ProblematicFileHandler {
        fn new(filename: &str) -> Self {
            println!("     📂 Opening file resource: {}", filename);
            ProblematicFileHandler {
                filename: filename.to_string(),
                resource_count: 1, // Simulating resource allocation
            }
        }
    }

    // ❌ NO DROP IMPLEMENTATION - resources might not be cleaned up properly

    // ✅ SOLUTION: Implement Drop for proper cleanup
    struct ProperFileHandler {
        filename: String,
        resource_count: usize,
    }

    impl ProperFileHandler {
        fn new(filename: &str) -> Self {
            println!("     📂 Opening file resource: {}", filename);
            ProperFileHandler {
                filename: filename.to_string(),
                resource_count: 1,
            }
        }
    }

    impl Drop for ProperFileHandler {
        fn drop(&mut self) {
            println!("     🔒 Properly closing file resource: {}", self.filename);
            self.resource_count = 0; // Cleanup resource
        }
    }

    fn demonstrate_drop_importance() {
        println!("   ✅ SOLUTION: Implementing Drop for proper cleanup");

        {
            let _file_handler = ProperFileHandler::new("menu_data.txt");
            println!("     📊 Using file handler for business operations");
        } // Drop automatically called here

        println!("     ✅ Resource properly cleaned up by Drop implementation");
    }

    demonstrate_drop_importance();

    // Scenario 4: Global State Accumulation
    println!("\n4️⃣ Scenario: Global State Accumulation - Unbounded Growth");

    println!("   ⚠️ THE PROBLEM:");
    println!("   Global collections that grow indefinitely without cleanup");

    use std::sync::Mutex;
    use std::sync::OnceLock;

    // Problematic pattern - unbounded growth
    static GLOBAL_CACHE: OnceLock<Mutex<HashMap<String, String>>> = OnceLock::new();

    fn problematic_caching(key: String, value: String) {
        let cache = GLOBAL_CACHE.get_or_init(|| Mutex::new(HashMap::new()));
        let mut cache_guard = cache.lock().unwrap();
        cache_guard.insert(key, value);
        // Cache grows forever - no cleanup mechanism!
    }

    // ✅ SOLUTION: Bounded cache with cleanup
    static BOUNDED_CACHE: OnceLock<Mutex<BoundedCache>> = OnceLock::new();

    struct BoundedCache {
        data: HashMap<String, String>,
        max_size: usize,
    }

    impl BoundedCache {
        fn new(max_size: usize) -> Self {
            BoundedCache {
                data: HashMap::new(),
                max_size,
            }
        }

        fn insert(&mut self, key: String, value: String) {
            if self.data.len() >= self.max_size {
                // Remove oldest entries (simplified LRU)
                if let Some(old_key) = self.data.keys().next().cloned() {
                    self.data.remove(&old_key);
                }
            }
            self.data.insert(key, value);
        }
    }

    fn proper_caching(key: String, value: String) {
        let cache = BOUNDED_CACHE.get_or_init(|| Mutex::new(BoundedCache::new(100)));
        let mut cache_guard = cache.lock().unwrap();
        cache_guard.insert(key, value);
    }

    fn demonstrate_bounded_caching() {
        println!("   ✅ SOLUTION: Bounded cache with automatic cleanup");

        for i in 0..5 {
            proper_caching(
                format!("order_{}", i),
                format!("Order data for customer {}", i)
            );
        }

        println!("     📊 Cache properly bounded to prevent unbounded growth");
        println!("     ✅ Automatic cleanup prevents memory accumulation");
    }

    demonstrate_bounded_caching();

    println!("\n🎯 Memory Leak Prevention Strategies:");
    println!("   🔄 Use Weak references to break reference cycles");
    println!("   📌 Use intentional leaks (Box::leak) only when truly needed");
    println!("   🔒 Implement Drop trait for custom resource management");
    println!("   📊 Design bounded data structures to prevent unbounded growth");
    println!("   🛡️ Trust Rust's ownership system for automatic memory management");

    println!("\n💡 Professional Guidelines:");
    println!("   📋 Audit code for potential reference cycles in complex data structures");
    println!("   🔍 Monitor long-running applications for memory usage patterns");
    println!("   ⚖️ Balance static data usage with memory efficiency");
    println!("   🎯 Design cleanup strategies for global state and caches");
    println!("   📚 Document intentional design decisions around memory management");
}

fn main() {
    demonstrate_common_leak_scenarios();
}
```


## Advanced Memory Leak Detection and Prevention

### Professional Memory Management Tools and Techniques

**Advanced strategies for detecting, preventing, and managing memory in complex applications:**

```rust
fn demonstrate_advanced_memory_management() {
    println!("🔬 Advanced Memory Management - Professional Monitoring and Prevention");
    println!("{:=<75}", "");

    use std::sync::{Arc, Weak, Mutex};
    use std::thread;
    use std::collections::HashMap;
    use std::time::Duration;

    // Advanced memory management is like having sophisticated monitoring systems
    println!("🛠️ Professional Memory Management Techniques:");

    // Technique 1: Smart Pointer Pattern for Complex Ownership
    println!("\n1️⃣ Smart Pointer Patterns - Complex Ownership Management:");

    // Restaurant chain with complex relationships
    #[derive(Debug)]
    struct RestaurantChain {
        name: String,
        restaurants: Vec<Arc<Restaurant>>,
        central_management: Arc<Management>,
    }

    #[derive(Debug)]
    struct Restaurant {
        name: String,
        location: String,
        management: Weak<Management>, // Weak to prevent cycles
        staff: Mutex<Vec<Arc<Employee>>>,
    }

    #[derive(Debug)]
    struct Management {
        company_name: String,
        restaurants_managed: Mutex<Vec<Weak<Restaurant>>>, // Weak to prevent cycles
    }

    #[derive(Debug)]
    struct Employee {
        name: String,
        role: String,
        restaurant: Weak<Restaurant>, // Weak to prevent cycles
    }

    impl RestaurantChain {
        fn new(name: String) -> Self {
            let management = Arc::new(Management {
                company_name: format!("{} Management", name),
                restaurants_managed: Mutex::new(Vec::new()),
            });

            RestaurantChain {
                name,
                restaurants: Vec::new(),
                central_management: management,
            }
        }

        fn add_restaurant(&mut self, name: String, location: String) -> Arc<Restaurant> {
            let restaurant = Arc::new(Restaurant {
                name,
                location,
                management: Arc::downgrade(&self.central_management),
                staff: Mutex::new(Vec::new()),
            });

            // Add weak reference to management
            self.central_management
                .restaurants_managed
                .lock()
                .unwrap()
                .push(Arc::downgrade(&restaurant));

            self.restaurants.push(Arc::clone(&restaurant));
            restaurant
        }

        fn add_employee(&self, restaurant: &Arc<Restaurant>, name: String, role: String) {
            let employee = Arc::new(Employee {
                name,
                role,
                restaurant: Arc::downgrade(restaurant),
            });

            restaurant.staff.lock().unwrap().push(employee);
        }

        fn print_structure(&self) {
            println!("     🏢 Chain: {}", self.name);
            println!("     🏬 Restaurants: {}", self.restaurants.len());

            for restaurant in &self.restaurants {
                let staff_count = restaurant.staff.lock().unwrap().len();
                println!("       📍 {}: {} staff members", restaurant.name, staff_count);
            }
        }
    }

    fn demonstrate_complex_ownership() {
        println!("   🏗️ Complex ownership with cycle prevention:");

        let mut chain = RestaurantChain::new("Global Gourmet".to_string());

        let restaurant1 = chain.add_restaurant(
            "Downtown Bistro".to_string(),
            "123 Main St".to_string()
        );

        let restaurant2 = chain.add_restaurant(
            "Seaside Cafe".to_string(),
            "456 Ocean Ave".to_string()
        );

        chain.add_employee(&restaurant1, "Alice Johnson".to_string(), "Manager".to_string());
        chain.add_employee(&restaurant1, "Bob Smith".to_string(), "Chef".to_string());
        chain.add_employee(&restaurant2, "Carol Davis".to_string(), "Server".to_string());

        chain.print_structure();

        println!("     ✅ Complex relationships managed without cycles");
        println!("     🔄 Weak references prevent reference cycles");
        println!("     🛡️ All memory will be properly cleaned up");
    }

    demonstrate_complex_ownership();

    // Technique 2: Resource Pool Management
    println!("\n2️⃣ Resource Pool Management - Preventing Resource Exhaustion:");

    struct ResourcePool<T> {
        available: Mutex<Vec<T>>,
        in_use: Mutex<Vec<T>>,
        factory: Box<dyn Fn() -> T + Send + Sync>,
        max_size: usize,
    }

    impl<T> ResourcePool<T> {
        fn new<F>(factory: F, max_size: usize) -> Self
        where
            F: Fn() -> T + Send + Sync + 'static,
        {
            ResourcePool {
                available: Mutex::new(Vec::new()),
                in_use: Mutex::new(Vec::new()),
                factory: Box::new(factory),
                max_size,
            }
        }

        fn acquire(&self) -> Option<T> {
            let mut available = self.available.lock().unwrap();
            let mut in_use = self.in_use.lock().unwrap();

            if let Some(resource) = available.pop() {
                in_use.push(resource);
                in_use.pop()
            } else if in_use.len() < self.max_size {
                let new_resource = (self.factory)();
                in_use.push(new_resource);
                in_use.pop()
            } else {
                None // Pool exhausted
            }
        }

        fn release(&self, resource: T) {
            let mut available = self.available.lock().unwrap();
            let mut in_use = self.in_use.lock().unwrap();

            // Find and remove from in_use (simplified)
            available.push(resource);
        }

        fn stats(&self) -> (usize, usize) {
            let available = self.available.lock().unwrap();
            let in_use = self.in_use.lock().unwrap();
            (available.len(), in_use.len())
        }
    }

    #[derive(Debug)]
    struct DatabaseConnection {
        id: u32,
        connected: bool,
    }

    impl DatabaseConnection {
        fn new(id: u32) -> Self {
            println!("     🔌 Creating database connection {}", id);
            DatabaseConnection { id, connected: true }
        }

        fn query(&self, _sql: &str) -> String {
            format!("Query result from connection {}", self.id)
        }
    }

    impl Drop for DatabaseConnection {
        fn drop(&mut self) {
            println!("     🔒 Closing database connection {}", self.id);
        }
    }

    fn demonstrate_resource_pooling() {
        println!("   🏊 Resource pooling for connection management:");

        static mut CONNECTION_COUNTER: u32 = 0;
        let pool = Arc::new(ResourcePool::new(
            || {
                unsafe {
                    CONNECTION_COUNTER += 1;
                    DatabaseConnection::new(CONNECTION_COUNTER)
                }
            },
            3, // Max 3 connections
        ));

        // Simulate multiple threads using the pool
        let mut handles = vec![];

        for i in 0..5 {
            let pool_clone = Arc::clone(&pool);
            let handle = thread::spawn(move || {
                if let Some(conn) = pool_clone.acquire() {
                    let result = conn.query("SELECT * FROM menu");
                    println!("     Thread {}: {}", i, result);

                    // Simulate work
                    thread::sleep(Duration::from_millis(100));

                    pool_clone.release(conn);
                } else {
                    println!("     Thread {}: Pool exhausted", i);
                }
            });
            handles.push(handle);
        }

        // Wait for all threads
        for handle in handles {
            handle.join().unwrap();
        }

        let (available, in_use) = pool.stats();
        println!("     📊 Final pool state: {} available, {} in use", available, in_use);
        println!("     ✅ Resource pool prevents connection leaks");
    }

    demonstrate_resource_pooling();

    // Technique 3: Memory Usage Monitoring
    println!("\n3️⃣ Memory Usage Monitoring - Proactive Leak Detection:");

    struct MemoryMonitor {
        allocations: Mutex<HashMap<String, usize>>,
    }

    impl MemoryMonitor {
        fn new() -> Self {
            MemoryMonitor {
                allocations: Mutex::new(HashMap::new()),
            }
        }

        fn track_allocation(&self, category: &str, size: usize) {
            let mut allocations = self.allocations.lock().unwrap();
            *allocations.entry(category.to_string()).or_insert(0) += size;
        }

        fn track_deallocation(&self, category: &str, size: usize) {
            let mut allocations = self.allocations.lock().unwrap();
            if let Some(current) = allocations.get_mut(category) {
                *current = current.saturating_sub(size);
            }
        }

        fn report(&self) -> HashMap<String, usize> {
            self.allocations.lock().unwrap().clone()
        }

        fn detect_leaks(&self, threshold: usize) -> Vec<String> {
            let allocations = self.allocations.lock().unwrap();
            allocations
                .iter()
                .filter(|(_, &size)| size > threshold)
                .map(|(category, _)| category.clone())
                .collect()
        }
    }

    struct MonitoredResource {
        data: Vec<u8>,
        category: String,
        monitor: Arc<MemoryMonitor>,
    }

    impl MonitoredResource {
        fn new(size: usize, category: String, monitor: Arc<MemoryMonitor>) -> Self {
            monitor.track_allocation(&category, size);
            MonitoredResource {
                data: vec![0; size],
                category,
                monitor,
            }
        }
    }

    impl Drop for MonitoredResource {
        fn drop(&mut self) {
            self.monitor.track_deallocation(&self.category, self.data.len());
        }
    }

    fn demonstrate_memory_monitoring() {
        println!("   📊 Memory monitoring for leak detection:");

        let monitor = Arc::new(MemoryMonitor::new());

        {
            let _resource1 = MonitoredResource::new(1024, "menu_data".to_string(), Arc::clone(&monitor));
            let _resource2 = MonitoredResource::new(2048, "customer_data".to_string(), Arc::clone(&monitor));
            let _resource3 = MonitoredResource::new(512, "order_cache".to_string(), Arc::clone(&monitor));

            let report = monitor.report();
            println!("     Memory usage during peak:");
            for (category, size) in report {
                println!("       {}: {} bytes", category, size);
            }

        } // Resources dropped here

        let final_report = monitor.report();
        println!("     Memory usage after cleanup:");
        for (category, size) in final_report {
            println!("       {}: {} bytes", category, size);
        }

        let potential_leaks = monitor.detect_leaks(100);
        if potential_leaks.is_empty() {
            println!("     ✅ No memory leaks detected");
        } else {
            println!("     ⚠️ Potential leaks in: {:?}", potential_leaks);
        }
    }

    demonstrate_memory_monitoring();

    // Technique 4: Graceful Shutdown and Cleanup
    println!("\n4️⃣ Graceful Shutdown - Complete Resource Cleanup:");

    struct ApplicationState {
        running: Arc<Mutex<bool>>,
        resources: Arc<Mutex<Vec<String>>>,
        cleanup_handlers: Arc<Mutex<Vec<Box<dyn Fn() + Send>>>>,
    }

    impl ApplicationState {
        fn new() -> Self {
            ApplicationState {
                running: Arc::new(Mutex::new(true)),
                resources: Arc::new(Mutex::new(Vec::new())),
                cleanup_handlers: Arc::new(Mutex::new(Vec::new())),
            }
        }

        fn add_resource(&self, resource: String) {
            self.resources.lock().unwrap().push(resource);
        }

        fn add_cleanup_handler<F>(&self, handler: F)
        where
            F: Fn() + Send + 'static,
        {
            self.cleanup_handlers.lock().unwrap().push(Box::new(handler));
        }

        fn shutdown(&self) {
            println!("     🔄 Beginning graceful shutdown...");

            // Mark as shutting down
            *self.running.lock().unwrap() = false;

            // Run cleanup handlers
            let handlers = self.cleanup_handlers.lock().unwrap();
            for (i, handler) in handlers.iter().enumerate() {
                println!("     🧹 Running cleanup handler {}", i + 1);
                handler();
            }

            // Clean up resources
            let mut resources = self.resources.lock().unwrap();
            for resource in resources.drain(..) {
                println!("     🔒 Cleaning up resource: {}", resource);
            }

            println!("     ✅ Graceful shutdown complete - all resources cleaned up");
        }
    }

    fn demonstrate_graceful_shutdown() {
        println!("   🔄 Graceful shutdown with complete cleanup:");

        let app_state = ApplicationState::new();

        // Simulate adding resources
        app_state.add_resource("Database Pool".to_string());
        app_state.add_resource("Cache Manager".to_string());
        app_state.add_resource("HTTP Server".to_string());

        // Add cleanup handlers
        app_state.add_cleanup_handler(|| {
            println!("       💾 Flushing database writes");
        });

        app_state.add_cleanup_handler(|| {
            println!("       🌐 Closing network connections");
        });

        app_state.add_cleanup_handler(|| {
            println!("       📁 Saving application state");
        });

        // Simulate application shutdown
        app_state.shutdown();
    }

    demonstrate_graceful_shutdown();

    println!("\n🎯 Advanced Memory Management Benefits:");
    println!("   🏗️ Complex ownership patterns with cycle prevention");
    println!("   🏊 Resource pooling prevents exhaustion and leaks");
    println!("   📊 Memory monitoring enables proactive leak detection");
    println!("   🔄 Graceful shutdown ensures complete resource cleanup");
    println!("   🛡️ Professional-grade memory management for production systems");

    println!("\n💡 Professional Best Practices:");
    println!("   📋 Design ownership hierarchies to prevent cycles from the start");
    println!("   🔍 Implement monitoring for long-running applications");
    println!("   ⚖️ Use resource pools for expensive-to-create resources");
    println!("   🧹 Plan cleanup strategies as part of system architecture");
    println!("   📚 Document memory management patterns for team understanding");
}

fn main() {
    demonstrate_advanced_memory_management();
}
```


## Real-World Memory Management Case Studies

### Complete Restaurant System Implementation

**Practical examples showing memory leak prevention in real applications:**

```rust
fn demonstrate_real_world_memory_management() {
    println!("🏢 Real-World Memory Management - Complete Restaurant System");
    println!("{:=<75}", "");

    use std::sync::{Arc, Weak, Mutex, RwLock};
    use std::collections::{HashMap, VecDeque};
    use std::thread;
    use std::time::{Duration, Instant};

    // Real-world applications demonstrate comprehensive memory management
    println!("💼 Professional Memory Management Case Studies:");

    // Case Study 1: Order Management System with Complex Relationships
    println!("\n1️⃣ Case Study: Order Management System");

    #[derive(Debug, Clone)]
    struct OrderId(u32);

    #[derive(Debug, Clone)]
    struct CustomerId(u32);

    #[derive(Debug)]
    struct Order {
        id: OrderId,
        customer_id: CustomerId,
        items: Vec<String>,
        total: f64,
        status: OrderStatus,
        created_at: Instant,
    }

    #[derive(Debug, Clone)]
    enum OrderStatus {
        Pending,
        Confirmed,
        Preparing,
        Ready,
        Delivered,
        Cancelled,
    }

    struct OrderManager {
        orders: RwLock<HashMap<OrderId, Arc<Order>>>,
        customer_orders: RwLock<HashMap<CustomerId, Vec<OrderId>>>,
        order_history: Mutex<VecDeque<Arc<Order>>>,
        max_history_size: usize,
        cleanup_threshold: Duration,
    }

    impl OrderManager {
        fn new(max_history_size: usize, cleanup_threshold: Duration) -> Self {
            OrderManager {
                orders: RwLock::new(HashMap::new()),
                customer_orders: RwLock::new(HashMap::new()),
                order_history: Mutex::new(VecDeque::new()),
                max_history_size,
                cleanup_threshold,
            }
        }

        fn create_order(&self, customer_id: CustomerId, items: Vec<String>, total: f64) -> OrderId {
            static mut ORDER_COUNTER: u32 = 0;

            let order_id = unsafe {
                ORDER_COUNTER += 1;
                OrderId(ORDER_COUNTER)
            };

            let order = Arc::new(Order {
                id: order_id.clone(),
                customer_id: customer_id.clone(),
                items,
                total,
                status: OrderStatus::Pending,
                created_at: Instant::now(),
            });

            // Store in active orders
            self.orders.write().unwrap().insert(order_id.clone(), Arc::clone(&order));

            // Link to customer
            self.customer_orders
                .write()
                .unwrap()
                .entry(customer_id)
                .or_insert_with(Vec::new)
                .push(order_id.clone());

            println!("     📝 Created order {:?} for customer {:?}", order_id, customer_id);
            order_id
        }

        fn complete_order(&self, order_id: &OrderId) -> Result<(), String> {
            let mut orders = self.orders.write().unwrap();

            if let Some(order) = orders.remove(order_id) {
                // Move to history with size management
                let mut history = self.order_history.lock().unwrap();

                if history.len() >= self.max_history_size {
                    if let Some(old_order) = history.pop_front() {
                        println!("     🗑️ Removing old order from history: {:?}", old_order.id);
                    }
                }

                history.push_back(order);
                println!("     ✅ Completed and archived order {:?}", order_id);
                Ok(())
            } else {
                Err(format!("Order {:?} not found", order_id))
            }
        }

        fn cleanup_old_data(&self) {
            let now = Instant::now();
            let mut history = self.order_history.lock().unwrap();

            let original_len = history.len();
            history.retain(|order| now.duration_since(order.created_at) < self.cleanup_threshold);

            let removed = original_len - history.len();
            if removed > 0 {
                println!("     🧹 Cleaned up {} old orders from history", removed);
            }
        }

        fn get_stats(&self) -> (usize, usize, usize) {
            let active_orders = self.orders.read().unwrap().len();
            let customers = self.customer_orders.read().unwrap().len();
            let history_size = self.order_history.lock().unwrap().len();

            (active_orders, customers, history_size)
        }
    }

    fn demonstrate_order_management() {
        println!("   🏗️ Order management with automatic cleanup:");

        let order_manager = OrderManager::new(5, Duration::from_secs(10)); // Max 5 in history, 10s threshold

        // Create some orders
        let customer1 = CustomerId(101);
        let customer2 = CustomerId(102);

        let order1 = order_manager.create_order(
            customer1.clone(),
            vec!["Pizza".to_string(), "Salad".to_string()],
            28.99
        );

        let order2 = order_manager.create_order(
            customer2.clone(),
            vec!["Pasta".to_string()],
            15.50
        );

        let order3 = order_manager.create_order(
            customer1.clone(),
            vec!["Soup".to_string(), "Bread".to_string()],
            12.75
        );

        let (active, customers, history) = order_manager.get_stats();
        println!("     📊 Initial stats: {} active orders, {} customers, {} in history",
                 active, customers, history);

        // Complete some orders
        order_manager.complete_order(&order1).ok();
        order_manager.complete_order(&order2).ok();

        let (active, customers, history) = order_manager.get_stats();
        println!("     📊 After completion: {} active orders, {} customers, {} in history",
                 active, customers, history);

        // Simulate time-based cleanup
        thread::sleep(Duration::from_millis(100));
        order_manager.cleanup_old_data();

        let (active, customers, history) = order_manager.get_stats();
        println!("     📊 Final stats: {} active orders, {} customers, {} in history",
                 active, customers, history);

        println!("     ✅ Memory management: bounded history, automatic cleanup");
    }

    demonstrate_order_management();

    // Case Study 2: Cache Management with TTL and Memory Limits
    println!("\n2️⃣ Case Study: Intelligent Cache Management");

    #[derive(Debug, Clone)]
    struct CacheEntry<T> {
        value: T,
        created_at: Instant,
        access_count: u32,
        last_accessed: Instant,
    }

    struct IntelligentCache<T> {
        data: Mutex<HashMap<String, CacheEntry<T>>>,
        max_size: usize,
        ttl: Duration,
        access_stats: Mutex<HashMap<String, u32>>,
    }

    impl<T: Clone> IntelligentCache<T> {
        fn new(max_size: usize, ttl: Duration) -> Self {
            IntelligentCache {
                data: Mutex::new(HashMap::new()),
                max_size,
                ttl,
                access_stats: Mutex::new(HashMap::new()),
            }
        }

        fn insert(&self, key: String, value: T) {
            let mut data = self.data.lock().unwrap();

            // Clean up expired entries first
            self.cleanup_expired(&mut data);

            // If at capacity, remove least recently used
            if data.len() >= self.max_size {
                self.evict_lru(&mut data);
            }

            let entry = CacheEntry {
                value,
                created_at: Instant::now(),
                access_count: 0,
                last_accessed: Instant::now(),
            };

            data.insert(key.clone(), entry);

            // Update access stats
            *self.access_stats.lock().unwrap().entry(key).or_insert(0) += 1;

            println!("     💾 Cached entry, cache size: {}", data.len());
        }

        fn get(&self, key: &str) -> Option<T> {
            let mut data = self.data.lock().unwrap();

            if let Some(entry) = data.get_mut(key) {
                // Check if expired
                if entry.created_at.elapsed() > self.ttl {
                    data.remove(key);
                    println!("     ⏰ Cache entry expired and removed: {}", key);
                    return None;
                }

                // Update access info
                entry.access_count += 1;
                entry.last_accessed = Instant::now();

                // Update access stats
                *self.access_stats.lock().unwrap().entry(key.to_string()).or_insert(0) += 1;

                Some(entry.value.clone())
            } else {
                None
            }
        }

        fn cleanup_expired(&self, data: &mut HashMap<String, CacheEntry<T>>) {
            let now = Instant::now();
            let original_len = data.len();

            data.retain(|_, entry| now.duration_since(entry.created_at) < self.ttl);

            let removed = original_len - data.len();
            if removed > 0 {
                println!("     🧹 Removed {} expired cache entries", removed);
            }
        }

        fn evict_lru(&self, data: &mut HashMap<String, CacheEntry<T>>) {
            if let Some((lru_key, _)) = data
                .iter()
                .min_by_key(|(_, entry)| entry.last_accessed)
                .map(|(k, v)| (k.clone(), v.last_accessed))
            {
                data.remove(&lru_key);
                println!("     🗑️ Evicted LRU cache entry: {}", lru_key);
            }
        }

        fn stats(&self) -> (usize, usize) {
            let data_size = self.data.lock().unwrap().len();
            let stats_size = self.access_stats.lock().unwrap().len();
            (data_size, stats_size)
        }
    }

    fn demonstrate_intelligent_caching() {
        println!("   🧠 Intelligent cache with memory management:");

        let cache = IntelligentCache::new(3, Duration::from_millis(200)); // Max 3 items, 200ms TTL

        // Add items to cache
        cache.insert("menu_1".to_string(), "Pizza Menu Data".to_string());
        cache.insert("menu_2".to_string(), "Pasta Menu Data".to_string());
        cache.insert("menu_3".to_string(), "Salad Menu Data".to_string());

        let (data_size, stats_size) = cache.stats();
        println!("     📊 Cache filled: {} items, {} stat entries", data_size, stats_size);

        // Access some items
        if let Some(data) = cache.get("menu_1") {
            println!("     📖 Retrieved: {}", data);
        }

        // Add more items (should trigger eviction)
        cache.insert("menu_4".to_string(), "Dessert Menu Data".to_string());
        cache.insert("menu_5".to_string(), "Beverage Menu Data".to_string());

        let (data_size, stats_size) = cache.stats();
        println!("     📊 After additions: {} items, {} stat entries", data_size, stats_size);

        // Wait for TTL expiration
        thread::sleep(Duration::from_millis(250));

        // Try to access expired items
        if cache.get("menu_1").is_none() {
            println!("     ⏰ Cache entry expired as expected");
        }

        let (data_size, stats_size) = cache.stats();
        println!("     📊 After expiration: {} items, {} stat entries", data_size, stats_size);

        println!("     ✅ Cache automatically manages memory with TTL and LRU eviction");
    }

    demonstrate_intelligent_caching();

    // Case Study 3: Worker Thread Pool with Resource Management
    println!("\n3️⃣ Case Study: Worker Thread Pool with Cleanup");

    struct WorkerPool {
        workers: Vec<thread::JoinHandle<()>>,
        task_sender: std::sync::mpsc::Sender<Box<dyn FnOnce() + Send>>,
        shutdown_sender: std::sync::mpsc::Sender<()>,
    }

    impl WorkerPool {
        fn new(worker_count: usize) -> Self {
            let (task_sender, task_receiver) = std::sync::mpsc::channel();
            let (shutdown_sender, shutdown_receiver) = std::sync::mpsc::channel();

            let task_receiver = Arc::new(Mutex::new(task_receiver));
            let shutdown_receiver = Arc::new(Mutex::new(shutdown_receiver));

            let mut workers = Vec::new();

            for worker_id in 0..worker_count {
                let task_receiver = Arc::clone(&task_receiver);
                let shutdown_receiver = Arc::clone(&shutdown_receiver);

                let worker = thread::spawn(move || {
                    println!("     👷 Worker {} started", worker_id);

                    loop {
                        // Check for shutdown signal
                        if let Ok(_) = shutdown_receiver.lock().unwrap().try_recv() {
                            println!("     👷 Worker {} shutting down", worker_id);
                            break;
                        }

                        // Process tasks
                        if let Ok(task) = task_receiver.lock().unwrap().try_recv() {
                            println!("     ⚙️ Worker {} executing task", worker_id);
                            task();
                        } else {
                            // No tasks available, sleep briefly
                            thread::sleep(Duration::from_millis(10));
                        }
                    }

                    println!("     👷 Worker {} finished", worker_id);
                });

                workers.push(worker);
            }

            WorkerPool {
                workers,
                task_sender,
                shutdown_sender,
            }
        }

        fn submit_task<F>(&self, task: F) -> Result<(), String>
        where
            F: FnOnce() + Send + 'static,
        {
            self.task_sender
                .send(Box::new(task))
                .map_err(|_| "Failed to submit task".to_string())
        }

        fn shutdown(self) {
            println!("     🔄 Initiating worker pool shutdown...");

            // Send shutdown signal to all workers
            for _ in 0..self.workers.len() {
                self.shutdown_sender.send(()).ok();
            }

            // Wait for all workers to finish
            for (i, worker) in self.workers.into_iter().enumerate() {
                if let Err(e) = worker.join() {
                    println!("     ⚠️ Worker {} panicked: {:?}", i, e);
                }
            }

            println!("     ✅ All workers shut down cleanly");
        }
    }

    fn demonstrate_worker_pool() {
        println!("   👷 Worker pool with proper resource cleanup:");

        let pool = WorkerPool::new(3);

        // Submit various tasks
        for i in 0..8 {
            let task_id = i;
            pool.submit_task(move || {
                println!("       🔧 Executing task {}", task_id);
                thread::sleep(Duration::from_millis(50));
                println!("       ✅ Task {} completed", task_id);
            }).ok();
        }

        // Let tasks execute
        thread::sleep(Duration::from_millis(200));

        // Graceful shutdown ensures all resources are cleaned up
        pool.shutdown();
    }

    demonstrate_worker_pool();

    println!("\n🎯 Real-World Memory Management Benefits:");
    println!("   📊 Bounded data structures prevent unbounded growth");
    println!("   ⏰ Time-based cleanup removes stale data automatically");
    println!("   🧠 Intelligent caching balances performance and memory usage");
    println!("   👷 Worker pools ensure thread resources are properly managed");
    println!("   🔄 Graceful shutdown guarantees complete resource cleanup");

    println!("\n💡 Professional Implementation Guidelines:");
    println!("   📋 Design bounded collections from the start");
    println!("   🕐 Implement time-based cleanup for long-running applications");
    println!("   📊 Monitor memory usage and implement alerts");
    println!("   🧹 Build cleanup into system architecture, not as an afterthought");
    println!("   🔄 Test shutdown procedures to ensure complete resource cleanup");
}

fn main() {
    demonstrate_real_world_memory_management();
}
```


## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Resource Management System**

Remember our comprehensive professional restaurant resource management analogy:

- 🛡️ **Memory leak prevention** = **Comprehensive resource tracking** - Every resource has clear ownership and automatic return systems
- 🔄 **Automatic cleanup** = **Built-in return protocols** - Resources automatically returned when no longer needed
- ⚠️ **Leak scenarios** = **System vulnerabilities** - Specific patterns that can bypass normal resource management
- 🔬 **Advanced techniques** = **Professional monitoring** - Sophisticated tools for complex resource management
- 💼 **Real-world applications** = **Complete operational systems** - Professional implementations handling all resource types


### **Rust's Memory Leak Prevention Mechanisms**

**Core Prevention Systems:**

- **Ownership Model** - Every value has exactly one owner responsible for cleanup
- **RAII (Resource Acquisition Is Initialization)** - Resources freed when owners go out of scope
- **Automatic Drop** - Destructor called automatically, no manual memory management needed
- **Compile-time Guarantees** - Memory safety violations caught before runtime
- **Zero Runtime Cost** - All safety guarantees achieved without performance penalty


### **Memory Leak Risk Assessment Matrix**

| **Risk Level** | **Scenario** | **Cause** | **Prevention Strategy** |
| :-- | :-- | :-- | :-- |
| **High** | Reference Cycles | `Rc<RefCell<>>` circular references | Use `Weak<T>` to break cycles |
| **Medium** | Global State Growth | Unbounded collections in static variables | Implement bounded data structures |
| **Low** | Intentional Leaks | `Box::leak`, `mem::forget` usage | Use sparingly, document thoroughly |
| **Medium** | Missing Drop | Custom resources without cleanup | Implement `Drop` trait consistently |
| **Low** | Unsafe Code | Manual memory management errors | Minimize unsafe, audit carefully |

### **Essential Prevention Patterns**

**✅ Safe Reference Relationships:**

```rust
// Use Weak references to prevent cycles
struct Parent {
    children: Vec<Rc<Child>>,
}
struct Child {
    parent: Weak<Parent>, // Weak prevents cycle
}
```

**✅ Bounded Data Structures:**

```rust
// Implement size limits and cleanup
struct BoundedCache<T> {
    data: HashMap<String, T>,
    max_size: usize,
}
impl<T> BoundedCache<T> {
    fn insert(&mut self, key: String, value: T) {
        if self.data.len() >= self.max_size {
            // Remove oldest entries
        }
        self.data.insert(key, value);
    }
}
```

**✅ Resource Cleanup with Drop:**

```rust
struct ManagedResource {
    handle: ResourceHandle,
}
impl Drop for ManagedResource {
    fn drop(&mut self) {
        // Custom cleanup logic
        self.handle.close();
    }
}
```


### **Professional Best Practices Checklist**

**✅ Design Guidelines:**

- Design ownership hierarchies to prevent cycles from the start
- Use bounded collections to prevent unbounded growth
- Implement `Drop` trait for custom resource management
- Plan cleanup strategies as part of system architecture

**✅ Monitoring Guidelines:**

- Implement memory usage monitoring for long-running applications
- Set up alerts for unusual memory growth patterns
- Use profiling tools to detect actual leaks in production
- Test graceful shutdown procedures regularly

**✅ Code Review Guidelines:**

- Audit `Rc<RefCell<>>` patterns for potential cycles
- Review global state management for unbounded growth
- Verify `Drop` implementations for complete cleanup
- Check unsafe code blocks for proper memory management

**❌ Common Pitfalls:**

- Creating reference cycles with `Rc` without using `Weak`
- Implementing global caches without size limits or TTL
- Forgetting to implement `Drop` for types managing external resources
- Using `mem::forget` or `Box::leak` without clear justification
- Not testing memory usage under realistic load conditions


### **Memory Leak Detection Tools**

**🔧 Development Tools:**

- `cargo-leak` - Static analysis for potential leaks
- `valgrind` - Runtime memory leak detection (Linux)
- `AddressSanitizer` - Compile-time memory error detection
- `heaptrack` - Heap memory profiler

**📊 Production Monitoring:**

- Memory usage metrics and alerting
- Custom memory tracking for critical components
- Resource pool monitoring and statistics
- Graceful shutdown testing and validation


### **The Professional Advantage**

**Mastering memory leak prevention in Rust is like having a complete professional restaurant resource management system** that automatically tracks, manages, and returns all resources while preventing waste and ensuring long-term operational sustainability:

- 🛡️ **Automatic safety** - Built-in systems prevent most memory leaks without programmer intervention
- 📊 **Proactive monitoring** - Early detection systems identify problems before they become critical
- 🔄 **Graceful handling** - Proper cleanup procedures ensure resources are always returned
- ⚡ **Zero overhead** - Safety guarantees achieved without runtime performance cost
- 🎯 **Professional reliability** - Systems that can run indefinitely without resource degradation

**Understanding memory leak prevention transforms you from a programmer who worries about memory management to an architect** who builds systems with inherent resource safety. Just as a master restaurant manager can design operations that naturally prevent waste and ensure efficient resource utilization, a skilled Rust programmer leverages the ownership system to create applications that are inherently memory-safe and resource-efficient.

This comprehensive understanding of memory leak prevention - from fundamental ownership concepts through advanced monitoring techniques and real-world implementations - enables you to build Rust applications that maintain optimal memory usage throughout their entire lifecycle, whether you're creating simple utilities or complex enterprise systems that must run reliably for extended periods without resource degradation!

1. https://blog.logrocket.com/handling-memory-leaks-rust/
2. https://www.youtube.com/watch?v=CgLkktqyu98
3. https://doc.rust-lang.org/book/ch15-06-reference-cycles.html
4. https://www.reddit.com/r/rust/comments/kpsgin/how_to_find_a_memory_leak_in_a_rust_program/
5. https://www.deleaker.com/blog/2022/04/12/memory-leaks-in-rust/
6. https://www.trust-in-soft.com/resources/blogs/how-rust-memory-safety-prevents-in-the-field-fixes
7. https://rust-exercises.com/100-exercises/07_threads/03_leak.html
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch15-06-reference-cycles.html
9. https://dev.to/mesmacosta/profiling-memory-leaks-in-rust-a-tale-of-unexpected-challenges-1p3b
10. https://internals.rust-lang.org/t/a-new-memory-leak-example-with-channels/16565
11. https://greptime.com/blogs/2023-06-15-rust-memory-leaks
12. https://www.reddit.com/r/rust/comments/qykr45/is_there_ever_a_legitimate_worry_of_memory_leaks/
13. https://softwaremill.com/leaking-memory-on-purpose-in-rust/
14. https://dev.to/pratikcodes/understanding-memory-management-in-rust-48pi
15. https://news.ycombinator.com/item?id=43091396
16. https://stackoverflow.com/questions/55553048/is-it-possible-to-cause-a-memory-leak-in-rust
17. https://www.youtube.com/watch?v=DJdUjjOmyx8
18. https://onesignal.com/blog/solving-memory-leaks-in-rust/
