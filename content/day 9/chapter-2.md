+++
title = "Drop Trait"
description = "Learn about the Drop trait in Rust, how it enables custom cleanup logic, and its role in resource management and memory safety."
date = 2025-08-03
draft = false
weight = 2
+++

# The Drop Trait in Rust: Comprehensive Documentation

The **Drop trait** is a fundamental component of Rust's ownership system that enables **automatic resource cleanup** when values go out of scope. It provides a mechanism for defining custom cleanup logic that runs deterministically when a value is no longer needed, ensuring safe and efficient resource management without garbage collection overhead.

## What is the Drop Trait?

The **Drop trait** allows you to specify what happens when a value goes out of scope[1]. It serves as Rust's equivalent to destructors in C++, providing deterministic cleanup of resources. When a value implementing Drop goes out of scope, Rust automatically calls the `drop` method to perform cleanup operations.

### Core Definition

```rust
pub trait Drop {
    fn drop(&mut self);
}
```

The Drop trait contains a single required method:
- **`drop(&mut self)`** - Takes a mutable reference to `self` and performs cleanup operations

## How Drop Works

### Automatic Invocation

Drop is called automatically by Rust's **drop glue** when a value goes out of scope[2]:

```rust
struct Resource {
    name: String,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Dropping resource: {}", self.name);
    }
}

fn main() {
    let resource = Resource {
        name: String::from("database_connection"),
    };

    println!("Resource created");

    // resource.drop() is automatically called here when resource goes out of scope
}
```

**Output:**
```
Resource created
Dropping resource: database_connection
```

### Drop Order

Variables are dropped in **reverse order** of their creation (LIFO - Last In, First Out)[2]:

```rust
struct Firework {
    name: String,
}

impl Drop for Firework {
    fn drop(&mut self) {
        println!("💥 {} explodes!", self.name);
    }
}

fn main() {
    let firecracker = Firework { name: "Firecracker".to_string() };
    let tnt = Firework { name: "TNT".to_string() };
    let bomb = Firework { name: "Bomb".to_string() };

    println!("All fireworks ready!");
    // Drop order: bomb, tnt, firecracker (reverse order)
}
```

**Output:**
```
All fireworks ready!
💥 Bomb explodes!
💥 TNT explodes!
💥 Firecracker explodes!
```

## When to Implement Drop

### Resource Management Scenarios

Implement Drop when your type manages resources that need explicit cleanup[3][4]:

1. **File handles** - Closing files and flushing buffers
2. **Network connections** - Closing sockets and streams
3. **Database connections** - Closing database connections
4. **Locks and synchronization primitives** - Releasing locks
5. **Custom memory management** - Deallocating raw pointers
6. **GPU resources** - Releasing graphics buffers
7. **Logging and monitoring** - Final logging or metrics collection

### File Handle Management

```rust
use std::fs::File;
use std::io::Write;

struct SafeFileWriter {
    file: File,
    filename: String,
}

impl SafeFileWriter {
    fn new(filename: &str) -> std::io::Result {
        let file = File::create(filename)?;
        Ok(SafeFileWriter {
            file,
            filename: filename.to_string(),
        })
    }

    fn write_data(&mut self, data: &str) -> std::io::Result {
        self.file.write_all(data.as_bytes())
    }
}

impl Drop for SafeFileWriter {
    fn drop(&mut self) {
        // Ensure file is properly flushed and closed
        if let Err(e) = self.file.flush() {
            eprintln!("Error flushing file {}: {}", self.filename, e);
        }
        println!("Safely closed file: {}", self.filename);
    }
}

fn main() -> std::io::Result {
    {
        let mut writer = SafeFileWriter::new("output.txt")?;
        writer.write_data("Hello, world!\n")?;
        // File automatically flushed and closed when writer goes out of scope
    }

    println!("File operations completed");
    Ok(())
}
```

### Network Connection Management

```rust
use std::net::TcpStream;
use std::io::{Read, Write};

struct ManagedConnection {
    stream: TcpStream,
    connection_id: u32,
}

impl ManagedConnection {
    fn new(address: &str, id: u32) -> std::io::Result {
        let stream = TcpStream::connect(address)?;
        println!("Connection {} established to {}", id, address);

        Ok(ManagedConnection {
            stream,
            connection_id: id,
        })
    }

    fn send_data(&mut self, data: &[u8]) -> std::io::Result {
        self.stream.write_all(data)
    }
}

impl Drop for ManagedConnection {
    fn drop(&mut self) {
        // Gracefully close connection
        if let Err(e) = self.stream.shutdown(std::net::Shutdown::Both) {
            eprintln!("Error closing connection {}: {}", self.connection_id, e);
        } else {
            println!("Connection {} gracefully closed", self.connection_id);
        }
    }
}
```

### Database Connection Pool

```rust
struct DatabaseConnection {
    connection_id: u64,
    is_active: bool,
}

impl DatabaseConnection {
    fn new(id: u64) -> Self {
        println!("Opening database connection {}", id);
        DatabaseConnection {
            connection_id: id,
            is_active: true,
        }
    }

    fn execute_query(&mut self, query: &str) -> Result, &'static str> {
        if !self.is_active {
            return Err("Connection is closed");
        }

        println!("Executing query: {}", query);
        // Simulate query execution
        Ok(vec!["result1".to_string(), "result2".to_string()])
    }

    fn close(&mut self) {
        self.is_active = false;
        println!("Manually closed database connection {}", self.connection_id);
    }
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        if self.is_active {
            println!("Auto-closing database connection {}", self.connection_id);
            // Perform cleanup: close connection, return to pool, etc.
            self.is_active = false;
        }
    }
}

fn main() {
    {
        let mut conn1 = DatabaseConnection::new(1);
        conn1.execute_query("SELECT * FROM users").unwrap();
        // conn1 automatically closed when going out of scope
    }

    {
        let mut conn2 = DatabaseConnection::new(2);
        conn2.execute_query("SELECT * FROM orders").unwrap();
        conn2.close(); // Manual close
        // Drop still called, but no action needed since already closed
    }

    println!("All database operations completed");
}
```

## Manual Dropping with `std::mem::drop`

You cannot call the `drop` method directly, but you can force early cleanup using `std::mem::drop`[5]:

```rust
struct Resource {
    name: String,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Cleaning up: {}", self.name);
    }
}

fn main() {
    let resource1 = Resource { name: "First".to_string() };
    let resource2 = Resource { name: "Second".to_string() };

    println!("Resources created");

    // Force early cleanup of resource1
    std::mem::drop(resource1);

    println!("resource1 dropped early, resource2 still alive");

    // resource2 will be dropped automatically at end of scope
}
```

**Attempting to call drop directly results in a compiler error:**

```rust
// This will NOT compile
fn main() {
    let resource = Resource { name: "test".to_string() };
    resource.drop(); // Error: explicit destructor calls not allowed
}
```

## Advanced Drop Patterns

### Reference Counting and Cleanup

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;

static INSTANCE_COUNT: AtomicUsize = AtomicUsize::new(0);

struct TrackedResource {
    id: usize,
    data: Vec,
}

impl TrackedResource {
    fn new(size: usize) -> Self {
        let id = INSTANCE_COUNT.fetch_add(1, Ordering::SeqCst);
        println!("Creating resource {} with {} bytes", id, size);

        TrackedResource {
            id,
            data: vec![0; size],
        }
    }
}

impl Drop for TrackedResource {
    fn drop(&mut self) {
        let remaining = INSTANCE_COUNT.fetch_sub(1, Ordering::SeqCst) - 1;
        println!("Dropping resource {} (remaining: {})", self.id, remaining);

        // Cleanup: could involve notifying other systems,
        // updating metrics, etc.
    }
}

fn main() {
    println!("Starting resource management demo");

    let _resource1 = TrackedResource::new(1024);
    let _resource2 = TrackedResource::new(2048);

    {
        let _resource3 = TrackedResource::new(512);
        // resource3 dropped here
    }

    println!("Middle of main function");

    // resource1 and resource2 dropped at end of main
}
```

### Custom Smart Pointer

```rust
use std::ptr::NonNull;
use std::marker::PhantomData;

struct MyBox {
    ptr: NonNull,
    _marker: PhantomData,
}

impl MyBox {
    fn new(value: T) -> Self {
        let boxed = Box::new(value);
        let ptr = NonNull::new(Box::into_raw(boxed)).unwrap();

        MyBox {
            ptr,
            _marker: PhantomData,
        }
    }

    fn as_ref(&self) -> &T {
        unsafe { self.ptr.as_ref() }
    }

    fn as_mut(&mut self) -> &mut T {
        unsafe { self.ptr.as_mut() }
    }
}

impl Drop for MyBox {
    fn drop(&mut self) {
        unsafe {
            // Convert back to Box to trigger proper cleanup
            let _boxed = Box::from_raw(self.ptr.as_ptr());
            println!("MyBox dropped, memory freed");
        }
    }
}

fn main() {
    let mut my_box = MyBox::new(String::from("Hello, custom smart pointer!"));
    println!("Value: {}", my_box.as_ref());

    my_box.as_mut().push_str(" - Modified!");
    println!("Modified: {}", my_box.as_ref());

    // my_box automatically cleaned up here
}
```

### Lock Management

```rust
use std::sync::{Mutex, Arc};
use std::thread;
use std::time::Duration;

struct LockGuard {
    mutex: Arc>,
    lock_id: u32,
}

impl LockGuard {
    fn acquire(mutex: Arc>, id: u32) -> Result {
        println!("Thread {} attempting to acquire lock", id);

        // This would normally use a timeout or try_lock
        match mutex.try_lock() {
            Ok(_guard) => {
                println!("Thread {} acquired lock", id);
                Ok(LockGuard { mutex, lock_id: id })
            },
            Err(_) => {
                println!("Thread {} failed to acquire lock", id);
                Err("Lock acquisition failed")
            }
        }
    }

    fn access_data(&self) -> Result, std::sync::PoisonError>> {
        self.mutex.lock()
    }
}

impl Drop for LockGuard {
    fn drop(&mut self) {
        println!("Thread {} releasing lock", self.lock_id);
        // Lock is automatically released when MutexGuard goes out of scope
        // Additional cleanup logic could go here
    }
}
```

## Drop Trait and Other Traits

### Drop and Copy Mutual Exclusivity

Types that implement Drop **cannot** implement Copy[6]:

```rust
#[derive(Clone)] // Copy is NOT allowed
struct Resource {
    data: String,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Dropping resource");
    }
}

// This would cause a compiler error:
// impl Copy for Resource {} // Error: Copy not allowed on types with destructors
```

**Why this restriction exists:**
- Copy types are duplicated on assignment
- Drop types need cleanup when going out of scope
- If a Drop type could be Copy, multiple copies would all try to clean up the same resources

### Drop and Clone Interaction

Drop types can implement Clone for explicit duplication:

```rust
#[derive(Clone)]
struct ManagedResource {
    resource_id: u32,
    data: Vec,
}

impl ManagedResource {
    fn new(id: u32, size: usize) -> Self {
        println!("Creating resource {}", id);
        ManagedResource {
            resource_id: id,
            data: vec![0; size],
        }
    }
}

impl Drop for ManagedResource {
    fn drop(&mut self) {
        println!("Dropping resource {}", self.resource_id);
    }
}

fn main() {
    let resource1 = ManagedResource::new(1, 1024);
    let resource2 = resource1.clone(); // Explicit clone creates new resource

    println!("Both resources exist");

    // Both resource1 and resource2 will be dropped
}
```

## Common Patterns and Idioms

### RAII (Resource Acquisition Is Initialization)

```rust
struct FileProcessor {
    input_file: std::fs::File,
    output_file: std::fs::File,
    temp_dir: tempfile::TempDir, // Hypothetical temp directory
}

impl FileProcessor {
    fn new(input_path: &str, output_path: &str) -> std::io::Result {
        let input_file = std::fs::File::open(input_path)?;
        let output_file = std::fs::File::create(output_path)?;
        let temp_dir = tempfile::TempDir::new()?;

        println!("FileProcessor initialized with all resources");

        Ok(FileProcessor {
            input_file,
            output_file,
            temp_dir,
        })
    }

    fn process(&mut self) -> std::io::Result {
        // Processing logic here
        println!("Processing files...");
        Ok(())
    }
}

impl Drop for FileProcessor {
    fn drop(&mut self) {
        println!("FileProcessor cleanup:");
        println!("- Closing input file");
        println!("- Closing output file");
        println!("- Cleaning up temporary directory");
        // All cleanup happens automatically through each field's Drop implementation
    }
}
```

### Scope-Based Resource Management

```rust
fn process_with_timeout() {
    struct TimeoutGuard {
        start_time: std::time::Instant,
        operation: String,
    }

    impl TimeoutGuard {
        fn new(operation: String) -> Self {
            println!("Starting operation: {}", operation);
            TimeoutGuard {
                start_time: std::time::Instant::now(),
                operation,
            }
        }
    }

    impl Drop for TimeoutGuard {
        fn drop(&mut self) {
            let elapsed = self.start_time.elapsed();
            println!("Operation '{}' completed in {:?}", self.operation, elapsed);
        }
    }

    let _guard = TimeoutGuard::new("database_query".to_string());

    // Simulate some work
    std::thread::sleep(std::time::Duration::from_millis(100));

    // Timer automatically logs when _guard goes out of scope
}
```

### Builder Pattern with Drop

```rust
struct ConfigBuilder {
    values: std::collections::HashMap,
    built: bool,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            values: std::collections::HashMap::new(),
            built: false,
        }
    }

    fn set(mut self, key: &str, value: &str) -> Self {
        self.values.insert(key.to_string(), value.to_string());
        self
    }

    fn build(mut self) -> Config {
        self.built = true;
        Config {
            values: self.values,
        }
    }
}

impl Drop for ConfigBuilder {
    fn drop(&mut self) {
        if !self.built {
            eprintln!("Warning: ConfigBuilder dropped without calling build()!");
            eprintln!("Unused configuration: {:?}", self.values);
        }
    }
}

struct Config {
    values: std::collections::HashMap,
}

fn main() {
    // Proper usage
    let _config = ConfigBuilder::new()
        .set("host", "localhost")
        .set("port", "8080")
        .build();

    // This will trigger the warning
    let _unused_builder = ConfigBuilder::new()
        .set("timeout", "30");
    // Warning printed when _unused_builder is dropped
}
```

## Performance Considerations

### Drop Performance

```rust
use std::time::Instant;

struct ExpensiveDrop {
    data: Vec,
    id: usize,
}

impl ExpensiveDrop {
    fn new(id: usize, size: usize) -> Self {
        ExpensiveDrop {
            data: vec![0; size],
            id,
        }
    }
}

impl Drop for ExpensiveDrop {
    fn drop(&mut self) {
        // Simulate expensive cleanup
        let start = Instant::now();

        // Some expensive operation
        self.data.iter().sum::();

        let elapsed = start.elapsed();
        println!("Expensive cleanup for {} took {:?}", self.id, elapsed);
    }
}

fn benchmark_drops() {
    let start = Instant::now();

    {
        let _items: Vec = (0..100)
            .map(|i| ExpensiveDrop::new(i, 1000))
            .collect();
        // All items dropped here
    }

    let total_time = start.elapsed();
    println!("Total drop time: {:?}", total_time);
}
```

### Optimizing Drop Performance

```rust
struct OptimizedResource {
    data: Vec,
    needs_cleanup: bool,
}

impl OptimizedResource {
    fn new(size: usize) -> Self {
        OptimizedResource {
            data: vec![0; size],
            needs_cleanup: true,
        }
    }

    fn consume(mut self) -> Vec {
        self.needs_cleanup = false; // Skip expensive cleanup
        self.data
    }
}

impl Drop for OptimizedResource {
    fn drop(&mut self) {
        if self.needs_cleanup {
            // Only do expensive cleanup if necessary
            println!("Performing cleanup for resource");
            // Expensive cleanup logic here
        }
    }
}
```

## Error Handling in Drop

### Drop Should Not Panic

```rust
use std::fs::File;
use std::io::Write;

struct SafeWriter {
    file: Option,
    filename: String,
}

impl SafeWriter {
    fn new(filename: &str) -> std::io::Result {
        let file = File::create(filename)?;
        Ok(SafeWriter {
            file: Some(file),
            filename: filename.to_string(),
        })
    }

    fn write(&mut self, data: &str) -> std::io::Result {
        if let Some(ref mut file) = self.file {
            file.write_all(data.as_bytes())
        } else {
            Err(std::io::Error::new(
                std::io::ErrorKind::Other,
                "File already closed"
            ))
        }
    }

    fn close(&mut self) -> std::io::Result {
        if let Some(mut file) = self.file.take() {
            file.flush()?;
            println!("File {} closed successfully", self.filename);
        }
        Ok(())
    }
}

impl Drop for SafeWriter {
    fn drop(&mut self) {
        // Drop should never panic - handle errors gracefully
        if let Some(mut file) = self.file.take() {
            if let Err(e) = file.flush() {
                // Log error but don't panic
                eprintln!("Warning: Failed to flush {}: {}", self.filename, e);
            }
        }
    }
}
```

### Logging Errors from Drop

```rust
struct NetworkConnection {
    connection_id: u32,
    is_connected: bool,
}

impl NetworkConnection {
    fn new(id: u32) -> Self {
        println!("Establishing connection {}", id);
        NetworkConnection {
            connection_id: id,
            is_connected: true,
        }
    }

    fn disconnect(&mut self) -> Result {
        if self.is_connected {
            // Simulate potential failure
            if self.connection_id % 3 == 0 {
                return Err("Network timeout during disconnect");
            }

            self.is_connected = false;
            println!("Connection {} disconnected successfully", self.connection_id);
            Ok(())
        } else {
            Ok(()) // Already disconnected
        }
    }
}

impl Drop for NetworkConnection {
    fn drop(&mut self) {
        if self.is_connected {
            match self.disconnect() {
                Ok(()) => {},
                Err(e) => {
                    // Log the error but don't panic
                    eprintln!("Error during cleanup of connection {}: {}",
                             self.connection_id, e);
                }
            }
        }
    }
}
```

## Testing Drop Implementation

### Unit Testing Drop Behavior

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::sync::{Arc, Mutex};

    struct TestResource {
        id: u32,
        drop_counter: Arc>>,
    }

    impl TestResource {
        fn new(id: u32, counter: Arc>>) -> Self {
            TestResource {
                id,
                drop_counter: counter,
            }
        }
    }

    impl Drop for TestResource {
        fn drop(&mut self) {
            let mut counter = self.drop_counter.lock().unwrap();
            counter.push(self.id);
        }
    }

    #[test]
    fn test_drop_order() {
        let counter = Arc::new(Mutex::new(Vec::new()));

        {
            let _resource1 = TestResource::new(1, counter.clone());
            let _resource2 = TestResource::new(2, counter.clone());
            let _resource3 = TestResource::new(3, counter.clone());
        } // All resources dropped here

        let dropped = counter.lock().unwrap();
        assert_eq!(*dropped, vec![3, 2, 1]); // LIFO order
    }

    #[test]
    fn test_early_drop() {
        let counter = Arc::new(Mutex::new(Vec::new()));

        let resource1 = TestResource::new(1, counter.clone());
        let resource2 = TestResource::new(2, counter.clone());

        // Early drop of resource1
        std::mem::drop(resource1);

        {
            let dropped = counter.lock().unwrap();
            assert_eq!(*dropped, vec![1]); // Only resource1 dropped
        }

        std::mem::drop(resource2);

        {
            let dropped = counter.lock().unwrap();
            assert_eq!(*dropped, vec![1, 2]); // Both dropped
        }
    }
}
```

## Best Practices

### **Do**

1. **Keep Drop implementation simple and fast** - Avoid expensive operations
2. **Handle errors gracefully** - Never panic in Drop
3. **Use Drop for cleanup only** - Not for business logic
4. **Document Drop behavior** - Especially for public APIs
5. **Test Drop implementation** - Ensure proper cleanup in all scenarios

```rust
// Good Drop implementation
impl Drop for GoodResource {
    fn drop(&mut self) {
        // Simple, fast cleanup
        if let Err(e) = self.cleanup() {
            // Log error, don't panic
            eprintln!("Cleanup error: {}", e);
        }
    }
}
```

### **Don't**

1. **Don't panic in Drop** - Can cause program termination
2. **Don't call Drop directly** - Use `std::mem::drop` instead
3. **Don't implement Drop unnecessarily** - Only when custom cleanup is needed
4. **Don't do expensive work in Drop** - Keep it lightweight
5. **Don't forget about Drop order** - Be aware of LIFO behavior

```rust
// Problematic Drop implementation
impl Drop for ProblematicResource {
    fn drop(&mut self) {
        // Don't do this - expensive work in Drop
        self.perform_expensive_computation();

        // Don't do this - panic in Drop
        self.cleanup().expect("Cleanup must succeed!");
    }
}
```

### **Design Guidelines**

```rust
// Well-designed Drop implementation
struct WellDesignedResource {
    handle: Option,
    name: String,
}

impl WellDesignedResource {
    fn new(name: String) -> Self {
        let handle = acquire_resource();
        WellDesignedResource {
            handle: Some(handle),
            name,
        }
    }

    // Allow explicit cleanup
    fn close(&mut self) -> Result {
        if let Some(handle) = self.handle.take() {
            release_resource(handle)?;
            println!("Resource {} explicitly closed", self.name);
        }
        Ok(())
    }
}

impl Drop for WellDesignedResource {
    fn drop(&mut self) {
        // Idempotent cleanup - safe to call multiple times
        if let Some(handle) = self.handle.take() {
            if let Err(e) = release_resource(handle) {
                eprintln!("Warning: Failed to release {}: {}", self.name, e);
            }
        }
    }
}
```

## Summary and Key Takeaways

### **Core Concepts**

The **Drop trait** provides:
- **Automatic resource cleanup** when values go out of scope
- **Deterministic destruction** without garbage collection overhead
- **Safe resource management** through Rust's ownership system
- **Custom cleanup logic** for complex resource management scenarios

### **Key Rules**

- **Drop is called automatically** when values go out of scope
- **Variables drop in reverse order** (LIFO) of creation
- **Drop cannot be called directly** - use `std::mem::drop` for early cleanup
- **Drop types cannot implement Copy** due to resource management concerns
- **Drop should never panic** - handle errors gracefully

### **Best Use Cases**

```rust
// Excellent Drop use cases:
// 1. File handle management
impl Drop for FileManager {
    fn drop(&mut self) {
        self.close_all_files();
    }
}

// 2. Network connection cleanup
impl Drop for ConnectionPool {
    fn drop(&mut self) {
        self.close_all_connections();
    }
}

// 3. Custom memory management
impl Drop for MemoryArena {
    fn drop(&mut self) {
        unsafe { self.deallocate_all(); }
    }
}
```

### **Performance Benefits**

- **Zero-cost abstraction** - Drop calls are optimized away when possible
- **Deterministic cleanup** - Resources freed immediately when no longer needed
- **No garbage collection overhead** - Cleanup happens at compile-time-determined points
- **Cache-friendly** - Drop runs in stack order, maintaining memory locality

### **Memory Safety Guarantees**

- **Prevents resource leaks** through automatic cleanup
- **Ensures proper cleanup order** via ownership and lifetime rules
- **Eliminates double-free bugs** through move semantics
- **Provides deterministic destruction** without runtime uncertainty

Understanding the Drop trait is essential for effective Rust programming, especially when working with resources beyond simple memory allocation. **It enables safe, efficient resource management while maintaining Rust's zero-cost abstraction principles**. The Drop trait, combined with Rust's ownership system, provides a powerful foundation for building robust, resource-efficient applications without the complexity and overhead of manual memory management or garbage collection.

1. https://doc.rust-lang.org/std/ops/trait.Drop.html
2. https://doc.rust-lang.org/rust-by-example/trait/drop.html
3. https://www.linkedin.com/pulse/destructor-drop-trait-rust-amit-nadiger
4. https://www.linkedin.com/pulse/rusts-drop-trait-unlock-efficient-resource-management-abhishek-kumar-ycr2c
5. https://doc.rust-lang.org/book/ch15-03-drop.html
6. https://rust-exercises.com/100-exercises/04_traits/13_drop.html
7. https://rust-for-linux.github.io/docs/core/ops/drop/trait.Drop.html
8. https://www.geeksforgeeks.org/rust-drop-trait/
9. https://stackoverflow.com/questions/75991587/actual-use-cases-for-the-drop-trait-in-rust
10. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/drop.html
11. https://gencmurat.com/en/posts/drop-trait/
12. https://www.youtube.com/watch?v=K6TeVyGm8cs
13. https://yongliangliu.com/blog/rust-drop-trait
14. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch15-03-drop.html
15. https://docs.rs/drop_code
16. https://labex.io/tutorials/rust-exploring-rust-s-drop-trait-99217
17. https://www.reddit.com/r/rust/comments/ztv98j/need_for_help_to_understand_an_explanation_about/
18. https://v5.chriskrycho.com/journal/read-the-code/using-drop-safely-in-rust/
19. https://www.youtube.com/watch?v=LJKpr09k5jE
20. https://klizos.com/rust-memory-management/
21. https://www.reddit.com/r/rust/comments/121owt6/why_is_drop_a_trait/
22. https://reintech.io/blog/understanding-implementing-rust-drop-trait
23. https://phaiax.github.io/mdBook/rustbook/ch15-03-drop.html
