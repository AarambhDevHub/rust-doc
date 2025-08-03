+++
title = "Vector Basics in Rust"
description = "A complete beginner-to-advanced guide to using Vec<T> in Rust. Learn creation, access, methods, performance tips, and real-world use cases like shopping cart systems and student grade tracking."
date = 2025-08-01
draft = false
weight = 3
+++

# Vector Basics in Rust: A Detailed Guide

Building on your understanding of compound types like tuples, arrays, and strings, let's explore **Vectors** (`Vec<T>`) - Rust's most versatile and commonly used collection type. While arrays have fixed sizes known at compile time, vectors are **resizable arrays that can grow and shrink dynamically at runtime**.

## What is a Vector?

A **Vector** (`Vec<T>`) is Rust's growable array type that stores elements of the same type in a contiguous, heap-allocated memory block. Unlike arrays, vectors can change their size during program execution, making them perfect for collections where the number of elements isn't known at compile time.

### Key Characteristics

* **Homogeneous**: All elements must be of the same type
* **Dynamic size**: Can grow and shrink at runtime
* **Heap-allocated**: Data is stored on the heap, not the stack
* **Contiguous memory**: Elements are stored sequentially in memory
* **Zero-indexed**: First element is at index 0

### Memory Structure

A vector is represented using three components:

* **Pointer**: Points to the heap-allocated data
* **Length**: Current number of elements
* **Capacity**: Total allocated space (may be larger than length for efficiency)

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];
    
    println!("Length: {}", numbers.len());      // Current number of elements
    println!("Capacity: {}", numbers.capacity()); // Allocated space
    println!("Size in memory: {} bytes", std::mem::size_of_val(&numbers));
}
```

## Creating Vectors

There are several ways to create vectors in Rust:

### 1. Using the `vec!` Macro

The most common and idiomatic way to create a vector with initial values:

```rust
fn main() {
    // Create vector with initial values
    let fruits = vec!["apple", "banana", "cherry"];
    let numbers = vec![1, 2, 3, 4, 5];
    let mixed_types = vec![1.5, 2.7, 3.14]; // All elements must be same type
    
    println!("Fruits: {:?}", fruits);
    println!("Numbers: {:?}", numbers);
    println!("Floats: {:?}", mixed_types);
}
```

### 2. Using `Vec::new()` Constructor

Create an empty vector and add elements later:

```rust
fn main() {
    // Create empty vector with explicit type annotation
    let mut empty_numbers: Vec<i32> = Vec::new();
    
    // Add elements
    empty_numbers.push(10);
    empty_numbers.push(20);
    empty_numbers.push(30);
    
    println!("Numbers: {:?}", empty_numbers); // [10, 20, 30]
}
```

### 3. Using `Vec::with_capacity()`

Create a vector with pre-allocated capacity for performance optimization:

```rust
fn main() {
    // Pre-allocate space for 100 elements
    let mut optimized_vec: Vec<i32> = Vec::with_capacity(100);
    
    println!("Length: {}", optimized_vec.len());      // 0
    println!("Capacity: {}", optimized_vec.capacity()); // 100
    
    // Adding elements won't trigger reallocation until capacity is exceeded
    for i in 1..=50 {
        optimized_vec.push(i);
    }
    
    println!("After adding 50 elements:");
    println!("Length: {}", optimized_vec.len());      // 50
    println!("Capacity: {}", optimized_vec.capacity()); // Still 100
}
```

### 4. Repeat Initialization

Create a vector with repeated values:

```rust
fn main() {
    // Create vector with 5 zeros
    let zeros = vec![0; 5];  // [0, 0, 0, 0, 0]
    
    // Create vector with 3 strings
    let default_names = vec!["Unknown".to_string(); 3];
    
    println!("Zeros: {:?}", zeros);
    println!("Names: {:?}", default_names);
}
```

## Accessing Vector Elements

### 1. Index Access (Direct Access)

Direct access using square bracket notation, which can panic if index is out of bounds:

```rust
fn main() {
    let colors = vec!["red", "green", "blue"];
    
    // Direct access - will panic if index is out of bounds
    println!("First color: {}", colors[^0]);    // "red"
    println!("Second color: {}", colors[^1]);   // "green"
    
    // This would panic at runtime:
    // println!("Invalid: {}", colors[^10]); // thread 'main' panicked
}
```

### 2. Safe Access with `.get()` Method

Returns `Option<&T>` for safe access that won't panic:

```rust
fn main() {
    let numbers = vec![10, 20, 30, 40, 50];
    
    // Safe access returns Option
    match numbers.get(2) {
        Some(value) => println!("Found value: {}", value), // "Found value: 30"
        None => println!("Index out of bounds"),
    }
    
    // Out of bounds access
    match numbers.get(10) {
        Some(value) => println!("Found: {}", value),
        None => println!("Index 10 is out of bounds"), // This executes
    }
    
    // Using if let for cleaner code
    if let Some(first) = numbers.get(0) {
        println!("First element: {}", first);
    }
}
```

### 3. First and Last Element Access

```rust
fn main() {
    let data = vec![1, 2, 3, 4, 5];
    
    // Get first and last elements safely
    if let Some(first) = data.first() {
        println!("First: {}", first);
    }
    
    if let Some(last) = data.last() {
        println!("Last: {}", last);
    }
    
    // Empty vector handling
    let empty_vec: Vec<i32> = vec![];
    println!("Empty first: {:?}", empty_vec.first()); // None
}
```

## Modifying Vectors

Vectors must be declared as mutable (`mut`) to modify their contents:

### Adding Elements

```rust
fn main() {
    let mut fruits = vec!["apple", "banana"];
    
    // Add single element to the end
    fruits.push("cherry");
    fruits.push("date");
    
    println!("After push: {:?}", fruits); // ["apple", "banana", "cherry", "date"]
    
    // Add multiple elements
    let mut more_fruits = vec!["elderberry", "fig"];
    fruits.append(&mut more_fruits);
    
    println!("After append: {:?}", fruits);
    println!("More fruits (now empty): {:?}", more_fruits); // []
}
```

### Removing Elements

```rust
fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    
    // Remove last element
    if let Some(last) = numbers.pop() {
        println!("Popped: {}", last); // 5
    }
    println!("After pop: {:?}", numbers); // [1, 2, 3, 4]
    
    // Remove element at specific index
    let removed = numbers.remove(1); // Removes element at index 1
    println!("Removed: {}", removed); // 2
    println!("After remove: {:?}", numbers); // [1, 3, 4]
    
    // Remove first element
    if !numbers.is_empty() {
        let first = numbers.remove(0);
        println!("Removed first: {}", first); // 1
    }
}
```

### Modifying Existing Elements

```rust
fn main() {
    let mut scores = vec![85, 90, 78, 92];
    
    // Modify element by index
    scores[^2] = 88; // Change third score from 78 to 88
    
    // Safely modify element
    if let Some(first_score) = scores.get_mut(0) {
        *first_score += 5; // Increase first score by 5
    }
    
    println!("Updated scores: {:?}", scores); // [90, 90, 88, 92]
}
```

## Vector Methods and Properties

### Common Methods

```rust
fn main() {
    let mut data = vec![1, 2, 3, 4, 5];
    
    // Basic properties
    println!("Length: {}", data.len());           // 5
    println!("Is empty: {}", data.is_empty());    // false
    println!("Capacity: {}", data.capacity());    // varies
    
    // Check if contains element
    println!("Contains 3: {}", data.contains(&3)); // true
    println!("Contains 10: {}", data.contains(&10)); // false
    
    // Clear all elements
    data.clear();
    println!("After clear - Length: {}", data.len());     // 0
    println!("After clear - Is empty: {}", data.is_empty()); // true
    println!("After clear - Capacity: {}", data.capacity()); // Still allocated
}
```

### Insertion at Specific Positions

```rust
fn main() {
    let mut words = vec!["hello", "world"];
    
    // Insert at specific index
    words.insert(1, "beautiful");
    println!("After insert: {:?}", words); // ["hello", "beautiful", "world"]
    
    // Insert at beginning
    words.insert(0, "say");
    println!("After insert at 0: {:?}", words); // ["say", "hello", "beautiful", "world"]
}
```

## Iterating Over Vectors

### Basic Iteration

```rust
fn main() {
    let numbers = vec![10, 20, 30, 40, 50];
    
    // Iterate by value (consumes the vector)
    for number in numbers {
        println!("Number: {}", number);
    }
    // numbers is no longer accessible here
    
    // Iterate by reference (doesn't consume)
    let colors = vec!["red", "green", "blue"];
    for color in &colors {
        println!("Color: {}", color);
    }
    // colors is still accessible here
    
    // Iterate with index
    for (index, color) in colors.iter().enumerate() {
        println!("Index {}: {}", index, color);
    }
}
```

### Iterator Methods

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    // Filter even numbers
    let even_numbers: Vec<i32> = numbers.iter()
        .filter(|&n| n % 2 == 0)
        .cloned()
        .collect();
    println!("Even numbers: {:?}", even_numbers); // [2, 4, 6, 8, 10]
    
    // Transform elements
    let doubled: Vec<i32> = numbers.iter()
        .map(|&n| n * 2)
        .collect();
    println!("Doubled: {:?}", doubled); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
    
    // Find element
    let found = numbers.iter().find(|&n| *n > 5);
    println!("First number > 5: {:?}", found); // Some(6)
    
    // Sum all elements
    let sum: i32 = numbers.iter().sum();
    println!("Sum: {}", sum); // 55
}
```

## Vector Slicing

Vectors support slicing similar to arrays:

```rust
fn main() {
    let data = vec![0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
    
    // Get slice of vector
    let slice = &data[2..6];  // Elements at indices 2, 3, 4, 5
    println!("Slice: {:?}", slice); // [2, 3, 4, 5]
    
    // Different slice ranges
    let first_half = &data[..5];       // First 5 elements
    let second_half = &data[5..];      // From index 5 to end
    let middle = &data[3..7];          // Indices 3 to 6
    
    println!("First half: {:?}", first_half);
    println!("Second half: {:?}", second_half);
    println!("Middle: {:?}", middle);
}
```

## Memory Management and Performance

### Growth Strategy

Vectors use a **doubling strategy** for capacity growth to minimize allocations:

```rust
fn main() {
    let mut vec = Vec::new();
    
    // Watch capacity grow
    for i in 0..10 {
        vec.push(i);
        println!("Length: {}, Capacity: {}", vec.len(), vec.capacity());
    }
    
    // Manually control capacity
    vec.reserve(50); // Reserve additional space
    println!("After reserve - Capacity: {}", vec.capacity());
    
    // Shrink to fit actual size
    vec.shrink_to_fit();
    println!("After shrink_to_fit - Capacity: {}", vec.capacity());
}
```

### Performance Considerations

```rust
fn main() {
    // Inefficient - multiple allocations
    let mut slow_vec = Vec::new();
    for i in 0..1000 {
        slow_vec.push(i); // May trigger multiple reallocations
    }
    
    // Efficient - single allocation
    let mut fast_vec = Vec::with_capacity(1000);
    for i in 0..1000 {
        fast_vec.push(i); // No reallocation needed
    }
    
    println!("Both vectors created efficiently!");
}
```

## Practical Examples

### Student Grade Manager

```rust
#[derive(Debug)]
struct Student {
    name: String,
    grades: Vec<f64>,
}

impl Student {
    fn new(name: &str) -> Self {
        Student {
            name: name.to_string(),
            grades: Vec::new(),
        }
    }
    
    fn add_grade(&mut self, grade: f64) {
        if grade >= 0.0 && grade <= 100.0 {
            self.grades.push(grade);
        }
    }
    
    fn average(&self) -> Option<f64> {
        if self.grades.is_empty() {
            None
        } else {
            Some(self.grades.iter().sum::<f64>() / self.grades.len() as f64)
        }
    }
    
    fn highest_grade(&self) -> Option<f64> {
        self.grades.iter().cloned().fold(None, |max, grade| {
            match max {
                None => Some(grade),
                Some(current_max) => Some(current_max.max(grade)),
            }
        })
    }
}

fn main() {
    let mut student = Student::new("Alice");
    
    // Add grades
    student.add_grade(85.5);
    student.add_grade(92.0);
    student.add_grade(78.5);
    student.add_grade(96.0);
    
    println!("Student: {:?}", student);
    println!("Average: {:?}", student.average());
    println!("Highest: {:?}", student.highest_grade());
}
```

### Shopping Cart Implementation

```rust
#[derive(Debug, Clone)]
struct Item {
    name: String,
    price: f64,
    quantity: u32,
}

impl Item {
    fn new(name: &str, price: f64, quantity: u32) -> Self {
        Item {
            name: name.to_string(),
            price,
            quantity,
        }
    }
    
    fn total_cost(&self) -> f64 {
        self.price * self.quantity as f64
    }
}

struct ShoppingCart {
    items: Vec<Item>,
}

impl ShoppingCart {
    fn new() -> Self {
        ShoppingCart {
            items: Vec::new(),
        }
    }
    
    fn add_item(&mut self, item: Item) {
        // Check if item already exists
        if let Some(existing_item) = self.items.iter_mut()
            .find(|i| i.name == item.name) {
            existing_item.quantity += item.quantity;
        } else {
            self.items.push(item);
        }
    }
    
    fn remove_item(&mut self, name: &str) -> Option<Item> {
        if let Some(pos) = self.items.iter().position(|item| item.name == name) {
            Some(self.items.remove(pos))
        } else {
            None
        }
    }
    
    fn total_cost(&self) -> f64 {
        self.items.iter().map(|item| item.total_cost()).sum()
    }
    
    fn item_count(&self) -> u32 {
        self.items.iter().map(|item| item.quantity).sum()
    }
}

fn main() {
    let mut cart = ShoppingCart::new();
    
    // Add items
    cart.add_item(Item::new("Apple", 0.99, 6));
    cart.add_item(Item::new("Bread", 2.50, 1));
    cart.add_item(Item::new("Apple", 0.99, 2)); // Will update quantity
    
    println!("Cart items: {:?}", cart.items);
    println!("Total cost: ${:.2}", cart.total_cost());
    println!("Total items: {}", cart.item_count());
    
    // Remove an item
    if let Some(removed) = cart.remove_item("Bread") {
        println!("Removed: {:?}", removed);
    }
    
    println!("After removal - Total cost: ${:.2}", cart.total_cost());
}
```

## Best Practices

### **Choose the Right Initialization Method**

```rust
// Use vec! for known initial values
let known_data = vec![1, 2, 3, 4, 5];

// Use Vec::new() for empty vectors where type can be inferred
let mut dynamic_data = Vec::new();
dynamic_data.push(42); // Type inferred as Vec<i32>

// Use Vec::with_capacity() for performance-critical code
let mut optimized = Vec::with_capacity(1000);
```

### **Prefer Safe Access Methods**

```rust
// Good - safe access
if let Some(value) = vector.get(index) {
    println!("Value: {}", value);
}

// Risky - can panic
// let value = vector[index]; // Use only when you're certain index is valid
```

### **Use Iterator Methods for Efficiency**

```rust
// Efficient - uses iterator chain
let processed: Vec<i32> = data.iter()
    .filter(|&x| x > 0)
    .map(|&x| x * 2)
    .collect();

// Less efficient - multiple passes
let mut temp = Vec::new();
for &item in &data {
    if item > 0 {
        temp.push(item);
    }
}
let mut processed = Vec::new();
for item in temp {
    processed.push(item * 2);
}
```

### **Consider Memory Usage**

```rust
// If you know the approximate size, pre-allocate
let mut large_vec = Vec::with_capacity(10000);

// Clean up unused capacity when done growing
large_vec.shrink_to_fit();
```

Understanding vectors is crucial for Rust programming as they're the go-to choice for dynamic collections. **Vectors provide the flexibility of dynamic sizing while maintaining memory safety and performance**. They serve as the foundation for many other data structures and are essential for building real-world applications. The combination of safety, performance, and ease of use makes vectors one of Rust's most powerful features.

###### Sources

https://www.programiz.com/rust/vector

https://dev.to/iamdipankarpaul/understanding-vectors-in-rust-a-comprehensive-guide-1j7p

https://www.w3schools.com/rust/rust\_vectors.php

https://www.youtube.com/watch?v=f4a8rWkSl8U

https://www.geeksforgeeks.org/rust/rust-vectors/

https://codesignal.com/learn/courses/exploring-data-structures-in-rust/lessons/vectors-in-rust

https://www.youtube.com/watch?v=Xew671U8-to

https://doc.rust-lang.org/rust-by-example/std/vec.html

https://onecompiler.com/tutorials/rust/collections/vectors
