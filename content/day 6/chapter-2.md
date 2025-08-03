+++
title = "Stack vs Heap in Rust"
description = "Understand the difference between stack and heap memory allocation in Rust."
date = 2025-08-02
draft = false
weight = 2
+++

# Stack vs Heap Memory in Rust: A Detailed Documentation Guide

Building on the ownership concepts we just covered, let's dive deep into **stack and heap memory** - the fundamental memory models that underpin Rust's ownership system. Understanding how these memory regions work is crucial for writing efficient Rust code and grasping why ownership rules exist.

## What are Stack and Heap?

**Stack** and **heap** are two distinct regions of memory that your program uses to store data. Each has different characteristics, performance implications, and use cases. Rust's ownership system is designed around efficiently managing both types of memory.

### Memory Layout Overview

```
┌─────────────────────────────────────┐
│            PROGRAM MEMORY           │
├─────────────────────────────────────┤
│              CODE                   │  ← Your compiled program
├─────────────────────────────────────┤
│              STACK                  │  ← Fast, automatic, limited
├─────────────────────────────────────┤
│              HEAP                   │  ← Flexible, manual, unlimited
└─────────────────────────────────────┘
```

## Stack Memory - The Fast Lane

The **stack** is a region of memory that operates on a **Last In, First Out (LIFO)** principle, like a stack of plates. It's extremely fast but has limitations.

### Stack Characteristics

| Feature | Description |
|---------|-------------|
| **Speed** | Very fast allocation and deallocation |
| **Organization** | LIFO (Last In, First Out) |
| **Size Limit** | Small (typically 1-8 MB per thread) |
| **Data Types** | Fixed-size data known at compile time |
| **Management** | Automatic - compiler handles everything |
| **Memory Layout** | Contiguous, predictable addresses |

### How Stack Memory Works

```rust
fn main() {
    let x = 5;        // Pushed onto stack
    let y = 10;       // Pushed onto stack  
    let z = x + y;    // Pushed onto stack
    
    {
        let a = 20;   // Pushed onto stack
        let b = 30;   // Pushed onto stack
        // Stack now contains: x, y, z, a, b (bottom to top)
    }  // a and b automatically popped from stack
    
    // Stack now contains: x, y, z
    println!("{}", z);
}  // x, y, z automatically popped when main ends
```

### Stack Memory Layout Visualization

```
Stack Growth (downward)
┌─────────────────────┐ ← High Memory Address
│                     │
│        ...          │
├─────────────────────┤
│    b: 30 (i32)     │ ← Most recent
├─────────────────────┤
│    a: 20 (i32)     │
├─────────────────────┤
│    z: 15 (i32)     │
├─────────────────────┤
│    y: 10 (i32)     │
├─────────────────────┤
│    x: 5 (i32)      │ ← First pushed
└─────────────────────┘ ← Low Memory Address (Stack Pointer)
```

### Data Types That Use Stack Memory

**All these types are stored entirely on the stack:**

```rust
fn main() {
    // Primitive types
    let integer: i32 = 42;           // 4 bytes on stack
    let float: f64 = 3.14;           // 8 bytes on stack
    let boolean: bool = true;        // 1 byte on stack
    let character: char = 'A';       // 4 bytes on stack (Unicode)
    
    // Fixed-size arrays
    let array: [i32; 5] = [1, 2, 3, 4, 5];  // 20 bytes on stack
    
    // Tuples (if all elements are stack types)
    let tuple: (i32, f64, bool) = (42, 3.14, true);  // 13 bytes on stack
    
    // Structs (if all fields are stack types)
    struct Point {
        x: f64,  // 8 bytes
        y: f64,  // 8 bytes
    }
    let point = Point { x: 10.0, y: 20.0 };  // 16 bytes on stack
    
    println!("All data is on the stack!");
}
```

### Stack Allocation Performance

```rust
fn stack_performance_demo() {
    // These operations are extremely fast
    let start = std::time::Instant::now();
    
    for _ in 0..1_000_000 {
        let x = 42;           // Stack allocation - nanoseconds
        let y = x * 2;        // Stack operation
        let z = (x, y);       // Stack tuple creation
        // All automatically cleaned up
    }
    
    let duration = start.elapsed();
    println!("Stack operations took: {:?}", duration);
}
```

## Heap Memory - The Flexible Space

The **heap** is a region of memory used for **dynamic allocation** - data whose size might not be known at compile time or might change during program execution.

### Heap Characteristics

| Feature | Description |
|---------|-------------|
| **Speed** | Slower allocation and deallocation |
| **Organization** | Unordered, fragmented |
| **Size Limit** | Large (limited by system RAM) |
| **Data Types** | Dynamic-size data, unknown at compile time |
| **Management** | Manual - programmer/runtime manages |
| **Memory Layout** | Non-contiguous, unpredictable addresses |

### How Heap Memory Works

```rust
fn main() {
    // Stack: pointer to heap data
    let s1 = String::from("hello");  
    // ┌─────────────────────┐ Stack
    // │ ptr: 0x1234        │ ← Points to heap
    // │ len: 5             │
    // │ cap: 5             │
    // └─────────────────────┘
    //           │
    //           ▼
    // ┌─────────────────────┐ Heap
    // │ h e l l o          │ ← Actual string data
    // └─────────────────────┘
    
    let s2 = s1;  // Move: heap data unchanged, stack pointer copied
    // s1 is now invalid, s2 owns the heap data
}
```

### Heap Memory Layout Visualization

```
Stack                           Heap
┌─────────────────────┐        ┌──────────────────────────┐
│ String s1:          │        │                          │
│   ptr: 0x2000   ────┼────────┼→ "hello" (5 bytes)      │
│   len: 5            │        │                          │
│   cap: 8            │        │    [unused capacity]     │
├─────────────────────┤        ├──────────────────────────┤
│ Vec numbers:   │        │                          │
│   ptr: 0x3000   ────┼────────┼→ [1, 2, 3, 4] (16 bytes)│
│   len: 4            │        │                          │
│   cap: 4            │        │                          │
└─────────────────────┘        └──────────────────────────┘
```

### Data Types That Use Heap Memory

**These types store their data on the heap:**

```rust
fn main() {
    // Dynamic strings
    let string = String::from("Hello, world!");  // Data on heap
    let mut growable = String::new();            // Can grow dynamically
    growable.push_str("Growing...");
    
    // Vectors (dynamic arrays)
    let mut numbers = Vec::new();                // Data on heap
    numbers.push(1);
    numbers.push(2);
    numbers.push(3);
    
    // Hash maps
    use std::collections::HashMap;
    let mut map = HashMap::new();                // Data on heap
    map.insert("key", "value");
    
    // Boxed values
    let boxed_int = Box::new(42);                // Forces heap allocation
    
    // Dynamic trait objects
    let dynamic: Box = Box::new("Hello");
}
```

### Heap Allocation Process

```rust
fn heap_allocation_demo() {
    println!("Creating heap-allocated data...");
    
    // 1. Request memory from OS
    let mut data = Vec::with_capacity(1000);
    println!("Allocated capacity for 1000 elements");
    
    // 2. Use the allocated memory  
    for i in 0..500 {
        data.push(i);
    }
    println!("Used 500 elements, 500 unused");
    
    // 3. Memory is automatically freed when `data` goes out of scope
    println!("Memory will be freed automatically");
}
```

## Performance Comparison

### Speed Differences

```rust
use std::time::Instant;

fn performance_comparison() {
    const ITERATIONS: usize = 1_000_000;
    
    // Stack allocation performance
    let start = Instant::now();
    for _ in 0..ITERATIONS {
        let _x = [0; 100];  // 400 bytes on stack
    }
    let stack_time = start.elapsed();
    
    // Heap allocation performance
    let start = Instant::now();
    for _ in 0..ITERATIONS {
        let _v = vec![0; 100];  // 400 bytes on heap
    }
    let heap_time = start.elapsed();
    
    println!("Stack allocation: {:?}", stack_time);
    println!("Heap allocation: {:?}", heap_time);
    println!("Heap is ~{:.1}x slower", 
             heap_time.as_nanos() as f64 / stack_time.as_nanos() as f64);
}
```

**Typical results:** Heap allocation is often 10-100x slower than stack allocation.

### Memory Access Patterns

```rust
fn access_pattern_demo() {
    // Stack: Excellent cache locality
    let stack_array = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let mut sum = 0;
    for &value in &stack_array {
        sum += value;  // Very fast - contiguous memory
    }
    
    // Heap: Potential cache misses
    let heap_vec = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let mut sum2 = 0;
    for &value in &heap_vec {
        sum2 += value;  // Slower - requires pointer indirection
    }
    
    println!("Sums: {} (stack), {} (heap)", sum, sum2);
}
```

## Mixed Stack-Heap Data Structures

Many Rust types combine stack and heap storage:

### String Structure

```rust
fn string_memory_layout() {
    let s = String::from("Hello, Rust programming!");
    
    // Stack portion (String struct):
    // - ptr: 8 bytes (pointer to heap)
    // - len: 8 bytes (current length)
    // - cap: 8 bytes (allocated capacity)
    // Total: 24 bytes on stack
    
    // Heap portion:
    // - Actual character data: 26 bytes
    // - May have additional capacity for growth
    
    println!("String length: {}", s.len());
    println!("String capacity: {}", s.capacity());
    println!("Stack size: {} bytes", std::mem::size_of_val(&s));
}
```

### Vector Structure

```rust
fn vector_memory_layout() {
    let mut v = Vec::with_capacity(10);
    v.push(1);
    v.push(2);
    v.push(3);
    
    // Stack portion (Vec struct):
    // - ptr: 8 bytes (pointer to heap)
    // - len: 8 bytes (current length: 3)
    // - cap: 8 bytes (allocated capacity: 10)
    // Total: 24 bytes on stack
    
    // Heap portion:
    // - 3 i32 values: 12 bytes used
    // - 7 i32 slots unused: 28 bytes reserved
    // - Total heap allocation: 40 bytes
    
    println!("Vector length: {}", v.len());
    println!("Vector capacity: {}", v.capacity());
    println!("Stack size: {} bytes", std::mem::size_of_val(&v));
}
```

## Ownership and Memory Management

### Stack Memory - Automatic Management

```rust
fn stack_scope_demo() {
    println!("Entering outer scope");
    let x = 10;  // Allocated on stack
    
    {
        println!("Entering inner scope");
        let y = 20;  // Allocated on stack
        let z = x + y;  // Allocated on stack
        println!("x: {}, y: {}, z: {}", x, y, z);
    }  // y and z automatically deallocated (popped from stack)
    
    println!("Back in outer scope, x: {}", x);
}  // x automatically deallocated
```

### Heap Memory - Ownership-Based Management

```rust
fn heap_ownership_demo() {
    println!("Creating heap data");
    let s1 = String::from("Hello");  // Heap allocation
    
    {
        let s2 = s1;  // Ownership transfer - no new allocation
        println!("s2 owns the data: {}", s2);
    }  // s2 goes out of scope, heap memory freed
    
    // println!("{}", s1);  // Error: s1 no longer owns the data
    
    // Create new heap data
    let s3 = String::from("World");
    let s4 = s3.clone();  // New heap allocation - expensive!
    println!("Both valid: {} {}", s3, s4);
}  // Both s3 and s4 heap data freed
```

## Choosing Between Stack and Heap

### Use Stack When:

```rust
fn prefer_stack() {
    // Fixed-size data known at compile time
    let coordinates = (10.5, 20.3);      // Good: tuple on stack
    let color = [255, 128, 0];           // Good: array on stack
    let point = Point { x: 1.0, y: 2.0 }; // Good: struct on stack
    
    // Small, temporary calculations
    let result = calculate_area(10.0, 20.0);  // Good: function params/returns on stack
}

struct Point {
    x: f64,
    y: f64,
}

fn calculate_area(width: f64, height: f64) -> f64 {
    width * height  // All on stack - very fast
}
```

### Use Heap When:

```rust
fn prefer_heap() {
    // Dynamic size or unknown at compile time
    let user_input = String::new();           // Good: can grow
    let mut items = Vec::new();               // Good: dynamic collection
    
    // Large data structures
    let large_buffer = vec![0u8; 1_000_000];  // Good: too big for stack
    
    // Data that needs to outlive current scope
    let persistent_data = create_user_data(); // Good: returned from function
    
    // Polymorphic data
    let drawable: Box = Box::new(Circle::new(5.0)); // Good: trait object
}

fn create_user_data() -> Vec {
    vec!["Alice".to_string(), "Bob".to_string()]
}

trait Draw {
    fn draw(&self);
}

struct Circle {
    radius: f64,
}

impl Circle {
    fn new(radius: f64) -> Self {
        Circle { radius }
    }
}

impl Draw for Circle {
    fn draw(&self) {
        println!("Drawing circle with radius {}", self.radius);
    }
}
```

## Stack Overflow and Heap Exhaustion

### Stack Overflow

Stack space is limited. Deep recursion or large stack allocations can cause stack overflow:

```rust
fn stack_overflow_example() {
    // This will cause stack overflow
    let large_array = [0u8; 10_000_000];  // Too big for stack!
    println!("Array created: {}", large_array.len());
}

fn infinite_recursion(n: u32) -> u32 {
    infinite_recursion(n + 1)  // Stack overflow - no base case
}

// Better: Use heap for large data
fn heap_alternative() {
    let large_vec = vec![0u8; 10_000_000];  // Safe: on heap
    println!("Vector created: {}", large_vec.len());
}

// Better: Use iteration instead of deep recursion
fn iterative_solution(mut n: u32) -> u32 {
    loop {
        n += 1;
        if n > 1000 { break n; }
    }
}
```

### Heap Exhaustion

Heap memory is limited by system RAM:

```rust
fn heap_exhaustion_example() {
    let mut vectors = Vec::new();
    
    // This will eventually exhaust heap memory
    loop {
        let big_vec = vec![0u8; 1_000_000];
        vectors.push(big_vec);
        println!("Allocated {} MB", vectors.len());
        
        // In a real program, you'd have proper error handling
        if vectors.len() > 1000 {
            break;  // Prevent actual exhaustion
        }
    }
}
```

## Memory-Efficient Programming Patterns

### 1. Prefer Stack When Possible

```rust
// Good: Use stack-allocated arrays for known sizes
fn process_coordinates(points: &[(f64, f64)]) -> f64 {
    let mut sum = 0.0;
    for (x, y) in points {
        sum += (x * x + y * y).sqrt();
    }
    sum
}

// Less optimal: Unnecessary heap allocation
fn process_coordinates_heap(points: &[(f64, f64)]) -> f64 {
    let distances: Vec = points.iter()
        .map(|(x, y)| (x * x + y * y).sqrt())
        .collect();  // Heap allocation
    distances.iter().sum()
}
```

### 2. Reuse Heap Allocations

```rust
fn efficient_string_processing() {
    let mut buffer = String::with_capacity(1000);  // Pre-allocate
    
    for i in 0..100 {
        buffer.clear();  // Reuse existing capacity
        buffer.push_str(&format!("Processing item {}", i));
        
        // Process buffer
        println!("{}", buffer);
    }
    // Only one heap allocation for all iterations
}
```

### 3. Use References to Avoid Cloning

```rust
// Good: Pass references to avoid heap allocation
fn process_data(data: &[String]) -> usize {
    data.iter().map(|s| s.len()).sum()
}

// Less efficient: Cloning creates new heap allocations
fn process_data_clone(data: Vec) -> usize {
    let cloned_data = data.clone();  // Expensive heap copying
    cloned_data.iter().map(|s| s.len()).sum()
}
```

## Debugging Memory Usage

### Tools and Techniques

```rust
fn memory_debugging() {
    // Check size of types
    println!("Size of i32: {} bytes", std::mem::size_of::());
    println!("Size of String: {} bytes", std::mem::size_of::());
    println!("Size of Vec: {} bytes", std::mem::size_of::>());
    
    // Check alignment
    println!("Alignment of String: {} bytes", std::mem::align_of::());
    
    // Check actual memory usage
    let s = String::from("Hello");
    println!("String heap capacity: {} bytes", s.capacity());
    
    let v = vec![1, 2, 3, 4, 5];
    println!("Vector heap capacity: {} elements", v.capacity());
}
```

## Best Practices Summary

### **Design for Performance**

```rust
// Good: Stack-first approach
struct Config {
    max_connections: u32,    // Stack
    timeout_seconds: u32,    // Stack
    server_name: String,     // Heap only when necessary
}

// Good: Use heap when it provides value
struct UserSession {
    user_id: u64,           // Stack
    permissions: Vec, // Heap - dynamic list
    data: HashMap, // Heap - dynamic mapping
}
```

### **Measure, Don't Assume**

```rust
fn benchmark_memory_patterns() {
    use std::time::Instant;
    
    let start = Instant::now();
    
    // Test your memory usage patterns
    for _ in 0..10_000 {
        let _data = [0; 1000];  // Stack allocation
    }
    
    println!("Stack pattern took: {:?}", start.elapsed());
}
```

Understanding stack vs heap memory is fundamental to writing efficient Rust code. **The stack provides speed and automatic management for fixed-size data, while the heap offers flexibility for dynamic data**. Rust's ownership system ensures memory safety for both, but knowing when to use each type of memory helps you write programs that are both safe and performant. The key is to **default to stack allocation when possible and use heap allocation when you need the flexibility**.