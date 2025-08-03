+++
title = "Mutable References"
description = "Explore how mutable references (&mut T) work in Rust and how they enable safe, exclusive write access to data."
date = 2025-08-02
draft = false
weight = 2
+++

# Mutable References (&mut T) in Rust: Comprehensive Documentation

Building on your understanding of immutable references and ownership rules, let's explore **mutable references** (`&mut T`) - Rust's mechanism for **exclusive, temporary ownership** that allows you to modify borrowed data. Mutable references are fundamental to Rust's borrowing system and enable safe mutation without violating ownership principles.

## What are Mutable References?

A **mutable reference** (`&mut T`) is an **exclusive reference** that provides **read and write access** to borrowed data. Unlike immutable references, mutable references grant the ability to modify the data they point to, but with strict exclusivity rules that prevent data races and ensure memory safety.

### Core Characteristics

- **Exclusive access** - Only one mutable reference can exist at a time
- **Read and write** - Can both read from and modify the referenced data
- **Non-owning** - Don't own the data they point to, just borrow it exclusively
- **Temporary** - Have limited lifetimes tied to the borrowed data
- **Safe mutation** - Enable safe modification without ownership transfer

```rust
fn main() {
    let mut data = String::from("Hello");  // data must be mutable
    let mutable_ref = &mut data;           // Create mutable reference
    
    mutable_ref.push_str(", World!");     // Modify through reference
    
    println!("Modified data: {}", data);   // Prints: "Hello, World!"
}
```

## Creating Mutable References

### Basic Mutable Reference Creation

```rust
fn main() {
    // Original data must be declared mutable
    let mut number = 42;
    let mut text = String::from("mutable");
    let mut vector = vec![1, 2, 3];
    
    // Create mutable references
    let number_ref = &mut number;      // &mut i32
    let text_ref = &mut text;          // &mut String  
    let vector_ref = &mut vector;      // &mut Vec
    
    // Modify through references
    *number_ref += 10;
    text_ref.push_str(" data");
    vector_ref.push(4);
    
    println!("number: {}", number);       // 52
    println!("text: {}", text);           // "mutable data"
    println!("vector: {:?}", vector);     // [1, 2, 3, 4]
}
```

### Requirements for Mutable References

```rust
fn main() {
    // 1. Original variable must be declared with `mut`
    let mut mutable_data = String::from("can change");
    let immutable_data = String::from("cannot change");
    
    // This works - mutable data can be mutably borrowed
    let mutable_ref = &mut mutable_data;
    mutable_ref.push_str("!");
    
    // This fails - immutable data cannot be mutably borrowed
    // let invalid_ref = &mut immutable_data;  // Error!
    
    println!("Modified: {}", mutable_data);
}
```

## The Exclusivity Rule

The most important rule for mutable references is **exclusivity**: you can have either **one mutable reference** OR **any number of immutable references**, but **never both simultaneously**.

### Exclusive Mutable Access

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    
    let mutable_ref1 = &mut data;  // First mutable reference
    
    // This would be an error - can't have two mutable references
    // let mutable_ref2 = &mut data;  // Error: second mutable borrow
    
    mutable_ref1.push(4);
    println!("Data: {:?}", mutable_ref1);
    
    // mutable_ref1 goes out of scope, so we can create another
    let mutable_ref2 = &mut data;
    mutable_ref2.push(5);
    
    println!("Final data: {:?}", data);
}
```

### Mutable vs Immutable Reference Conflicts

```rust
fn main() {
    let mut text = String::from("hello");
    
    // Multiple immutable references are fine
    let read_ref1 = &text;
    let read_ref2 = &text;
    println!("Reading: {} and {}", read_ref1, read_ref2);
    
    // After immutable references are done, mutable reference is allowed
    let write_ref = &mut text;
    write_ref.push_str(", world!");
    
    // This would be an error - can't mix mutable and immutable
    // println!("Reading: {}", read_ref1);  // Error if used here
    
    println!("Modified: {}", text);
}
```

### Scope-Based Reference Lifetimes

```rust
fn main() {
    let mut data = String::from("scoped");
    
    {
        let inner_ref = &mut data;  // Mutable reference in inner scope
        inner_ref.push_str(" inner");
        println!("Inner: {}", inner_ref);
    }  // inner_ref goes out of scope here
    
    // Now we can create another mutable reference
    let outer_ref = &mut data;
    outer_ref.push_str(" outer");
    
    println!("Final: {}", data);
}
```

## Using Mutable References

### Modifying Data Through References

```rust
fn main() {
    let mut numbers = vec![1, 2, 3];
    let numbers_ref = &mut numbers;
    
    // Various ways to modify through mutable reference
    numbers_ref.push(4);           // Add element
    numbers_ref[0] = 10;           // Modify existing element
    numbers_ref.sort();            // Call mutating method
    numbers_ref.reverse();         // Another mutating method
    
    println!("Modified numbers: {:?}", numbers);
}
```

### Dereferencing Mutable References

```rust
fn main() {
    let mut value = 42;
    let value_ref = &mut value;
    
    // Modify through dereferencing
    *value_ref = 100;              // Direct assignment
    *value_ref += 50;              // Compound assignment
    
    println!("Modified value: {}", value);  // 150
    
    // For more complex types
    let mut text = String::from("hello");
    let text_ref = &mut text;
    
    // Method calls work without explicit dereferencing
    text_ref.push_str(", world!");        // Automatic dereferencing
    (*text_ref).push('!');                 // Explicit dereferencing
    
    println!("Modified text: {}", text);
}
```

### Mutable References in Functions

```rust
fn modify_string(s: &mut String) {
    s.push_str(" modified");
    s.push('!');
}

fn double_number(n: &mut i32) {
    *n *= 2;
}

fn clear_and_add(v: &mut Vec, new_value: i32) {
    v.clear();
    v.push(new_value);
}

fn main() {
    let mut text = String::from("original");
    let mut number = 21;
    let mut numbers = vec![1, 2, 3, 4, 5];
    
    // Pass mutable references to functions
    modify_string(&mut text);
    double_number(&mut number);
    clear_and_add(&mut numbers, 42);
    
    println!("Text: {}", text);           // "original modified!"
    println!("Number: {}", number);       // 42
    println!("Numbers: {:?}", numbers);   // [42]
}
```

## Advanced Mutable Reference Patterns

### Reborrowing

**Reborrowing** allows you to create a new mutable reference from an existing one with a shorter lifetime:

```rust
fn modify_first_element(slice: &mut [i32]) {
    if let Some(first) = slice.get_mut(0) {
        *first *= 10;
    }
}

fn main() {
    let mut data = vec![1, 2, 3, 4, 5];
    let data_ref = &mut data;
    
    // Reborrow: create a new mutable reference with shorter lifetime
    modify_first_element(&mut *data_ref);  // Explicit reborrow
    
    // Or more commonly, automatic reborrow
    modify_first_element(data_ref);        // Automatic reborrow
    
    println!("Modified data: {:?}", data);
}
```

### Splitting Mutable References

You can split mutable references to different parts of the same data structure:

```rust
fn main() {
    let mut array = [1, 2, 3, 4, 5];
    
    // Split into two mutable slices
    let (left, right) = array.split_at_mut(2);
    
    // Both parts can be mutated simultaneously
    left[0] = 10;
    right[0] = 30;  // This is array[2]
    
    println!("Modified array: {:?}", array);  // [10, 2, 30, 4, 5]
    
    // Manual splitting with indexing
    let mut vector = vec![1, 2, 3, 4, 5, 6];
    let (first_half, second_half) = vector.split_at_mut(3);
    
    first_half[0] = 100;
    second_half[0] = 400;  // This is vector[3]
    
    println!("Modified vector: {:?}", vector);  // [100, 2, 3, 400, 5, 6]
}
```

### Mutable References to Struct Fields

```rust
struct Person {
    name: String,
    age: u32,
    email: String,
}

impl Person {
    fn update_name(&mut self, new_name: String) {
        self.name = new_name;
    }
    
    fn increment_age(&mut self) {
        self.age += 1;
    }
    
    fn get_name_mut(&mut self) -> &mut String {
        &mut self.name
    }
}

fn main() {
    let mut person = Person {
        name: String::from("Alice"),
        age: 30,
        email: String::from("alice@example.com"),
    };
    
    // Method calls with mutable reference to self
    person.update_name(String::from("Alice Smith"));
    person.increment_age();
    
    // Get mutable reference to field
    let name_ref = person.get_name_mut();
    name_ref.push_str(" (Updated)");
    
    // Direct field access
    person.email = String::from("alice.smith@example.com");
    
    println!("Person: {} ({}), Email: {}", person.name, person.age, person.email);
}
```

## Mutable References with Collections

### Vector Mutations

```rust
fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    let numbers_ref = &mut numbers;
    
    // Various vector mutations
    numbers_ref.push(6);              // Add element
    numbers_ref.insert(0, 0);         // Insert at index
    numbers_ref.remove(1);            // Remove element
    numbers_ref.sort();               // Sort in place
    numbers_ref.reverse();            // Reverse in place
    
    // Modify elements
    for element in numbers_ref.iter_mut() {
        *element *= 2;
    }
    
    println!("Final numbers: {:?}", numbers);
}
```

### HashMap Mutations

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert("Alice", 50);
    scores.insert("Bob", 60);
    
    let scores_ref = &mut scores;
    
    // Modify existing entries
    if let Some(alice_score) = scores_ref.get_mut("Alice") {
        *alice_score += 20;  // Alice now has 70
    }
    
    // Add new entries
    scores_ref.insert("Charlie", 80);
    
    // Modify all values
    for (_, score) in scores_ref.iter_mut() {
        *score += 10;  // Bonus points for everyone
    }
    
    println!("Final scores: {:?}", scores);
}
```

### String Mutations

```rust
fn main() {
    let mut text = String::from("Hello");
    let text_ref = &mut text;
    
    // Various string mutations
    text_ref.push_str(", World");     // Append string
    text_ref.push('!');               // Append character
    text_ref.insert(5, ',');          // Insert at position
    text_ref.replace_range(0..5, "Hi"); // Replace substring
    
    // Access as mutable bytes (unsafe for UTF-8)
    unsafe {
        let bytes = text_ref.as_mut_vec();
        bytes[0] = b'h';  // Change 'H' to 'h'
    }
    
    println!("Final text: {}", text);
}
```

## Mutable References and Iterators

### Iterator Mutations

```rust
fn main() {
    let mut data = vec![1, 2, 3, 4, 5];
    
    // Iterate with mutable references
    for item in &mut data {
        *item *= 2;  // Double each element
    }
    
    println!("Doubled: {:?}", data);
    
    // Using iter_mut()
    let mut words = vec![
        String::from("hello"),
        String::from("world"),
        String::from("rust"),
    ];
    
    for word in words.iter_mut() {
        word.push('!');  // Add exclamation to each word
    }
    
    println!("Exclaimed: {:?}", words);
    
    // Enumerate with mutable references
    let mut numbers = vec![10, 20, 30, 40, 50];
    for (index, value) in numbers.iter_mut().enumerate() {
        *value += index as i32;
    }
    
    println!("Indexed: {:?}", numbers);  // [10, 21, 32, 43, 54]
}
```

### Filter and Modify Pattern

```rust
fn main() {
    let mut scores = vec![85, 92, 78, 96, 88, 73];
    
    // Find and modify specific elements
    for score in scores.iter_mut() {
        if *score  Self {
        Counter { value: 0 }
    }
    
    fn increment(&mut self) {
        self.value += 1;
    }
    
    fn add(&mut self, amount: i32) {
        self.value += amount;
    }
    
    fn reset(&mut self) {
        self.value = 0;
    }
    
    fn get_value(&self) -> i32 {  // Immutable method for reading
        self.value
    }
}

fn main() {
    let mut counter = Counter::new();
    
    counter.increment();
    counter.add(5);
    println!("Counter: {}", counter.get_value());  // 6
    
    counter.reset();
    println!("After reset: {}", counter.get_value());  // 0
}
```

### The Builder Pattern with Mutable References

```rust
struct ConfigBuilder {
    host: String,
    port: u16,
    debug: bool,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            host: String::from("localhost"),
            port: 8080,
            debug: false,
        }
    }
    
    fn set_host(&mut self, host: &str) -> &mut Self {
        self.host = host.to_string();
        self  // Return mutable reference for chaining
    }
    
    fn set_port(&mut self, port: u16) -> &mut Self {
        self.port = port;
        self
    }
    
    fn enable_debug(&mut self) -> &mut Self {
        self.debug = true;
        self
    }
    
    fn build(&self) -> Config {
        Config {
            host: self.host.clone(),
            port: self.port,
            debug: self.debug,
        }
    }
}

struct Config {
    host: String,
    port: u16,
    debug: bool,
}

fn main() {
    let mut builder = ConfigBuilder::new();
    
    // Method chaining with mutable references
    let config = builder
        .set_host("example.com")
        .set_port(9000)
        .enable_debug()
        .build();
    
    println!("Config: {}:{} (debug: {})", config.host, config.port, config.debug);
}
```

### The Swap and Exchange Pattern

```rust
fn main() {
    let mut a = String::from("first");
    let mut b = String::from("second");
    
    // Swap using mutable references
    std::mem::swap(&mut a, &mut b);
    
    println!("a: {}, b: {}", a, b);  // "second", "first"
    
    // Replace and get old value
    let mut value = vec![1, 2, 3];
    let old_value = std::mem::replace(&mut value, vec![4, 5, 6]);
    
    println!("New: {:?}, Old: {:?}", value, old_value);
    
    // Take ownership and replace with default
    let mut optional = Some(String::from("taken"));
    let taken_value = optional.take();  // Replaces with None
    
    println!("Taken: {:?}, Remaining: {:?}", taken_value, optional);
}
```

## Borrowing Rules in Practice

### The Non-Lexical Lifetimes (NLL) System

Modern Rust uses **Non-Lexical Lifetimes**, which means references are only considered "live" while they're actually being used:

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    
    let immutable_ref = &data;
    println!("Immutable: {:?}", immutable_ref);
    // immutable_ref is no longer used after this point
    
    let mutable_ref = &mut data;  // This is now allowed!
    mutable_ref.push(4);
    println!("Mutable: {:?}", mutable_ref);
    
    // Both references can coexist if their usage doesn't overlap
}
```

### Reference Lifetime Inference

```rust
fn main() {
    let mut text = String::from("hello");
    
    // These references have non-overlapping lifetimes
    {
        let read_ref = &text;
        println!("Reading: {}", read_ref);
    }  // read_ref lifetime ends here
    
    {
        let write_ref = &mut text;
        write_ref.push_str(", world!");
        println!("Writing: {}", write_ref);
    }  // write_ref lifetime ends here
    
    println!("Final: {}", text);
}
```

### Function Boundaries and Lifetimes

```rust
fn process_data(data: &mut Vec) -> &mut i32 {
    data.push(42);
    data.last_mut().unwrap()  // Return mutable reference to last element
}

fn main() {
    let mut numbers = vec![1, 2, 3];
    
    let last_ref = process_data(&mut numbers);
    *last_ref = 100;  // Modify the last element
    
    println!("Numbers: {:?}", numbers);  // [1, 2, 3, 100]
}
```

## Performance Characteristics

### Zero-Cost Mutable Borrowing

```rust
use std::time::Instant;

fn performance_comparison() {
    let mut large_vec = vec![0i32; 1_000_000];
    
    // Direct mutation performance
    let start = Instant::now();
    for i in 0..large_vec.len() {
        large_vec[i] = i as i32;
    }
    let direct_time = start.elapsed();
    
    // Mutable reference performance
    let mut large_vec2 = vec![0i32; 1_000_000];
    let vec_ref = &mut large_vec2;
    let start = Instant::now();
    for i in 0..vec_ref.len() {
        vec_ref[i] = i as i32;
    }
    let reference_time = start.elapsed();
    
    println!("Direct mutation: {:?}", direct_time);
    println!("Reference mutation: {:?}", reference_time);
    println!("Performance difference: minimal (zero-cost abstraction)");
}
```

### Memory Layout of Mutable References

```rust
fn main() {
    let mut value = 42i32;
    let mutable_ref = &mut value;
    
    // Mutable reference is just a pointer - same size as usize
    println!("Size of i32: {}", std::mem::size_of::());
    println!("Size of &mut i32: {}", std::mem::size_of::());
    println!("Size of usize: {}", std::mem::size_of::());
    
    // Addresses show the relationship
    println!("Address of value: {:p}", &value);
    println!("Address of reference: {:p}", &mutable_ref);
    println!("Value of reference (points to value): {:p}", mutable_ref);
}
```

## Error Handling with Mutable References

### Safe Mutable Access

```rust
fn safe_modify_element(vec: &mut Vec, index: usize, new_value: i32) -> Result {
    match vec.get_mut(index) {
        Some(element) => {
            *element = new_value;
            Ok(())
        },
        None => Err("Index out of bounds"),
    }
}

fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    
    match safe_modify_element(&mut numbers, 2, 100) {
        Ok(()) => println!("Successfully modified"),
        Err(e) => println!("Error: {}", e),
    }
    
    match safe_modify_element(&mut numbers, 10, 200) {
        Ok(()) => println!("Successfully modified"),
        Err(e) => println!("Error: {}", e),
    }
    
    println!("Numbers: {:?}", numbers);
}
```

### Option and Result with Mutable References

```rust
fn find_and_modify(data: &mut Vec, target: &str, replacement: String) -> Option {
    for item in data.iter_mut() {
        if item == target {
            *item = replacement;
            return Some(item);
        }
    }
    None
}

fn main() {
    let mut words = vec![
        String::from("apple"),
        String::from("banana"),
        String::from("cherry"),
    ];
    
    match find_and_modify(&mut words, "banana", String::from("orange")) {
        Some(modified) => {
            println!("Modified item: {}", modified);
            modified.push_str("_extra");  // Can continue modifying
        },
        None => println!("Item not found"),
    }
    
    println!("Words: {:?}", words);
}
```

## Common Pitfalls and Solutions

### Borrow Checker Errors

#### Multiple Mutable References

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    
    // This doesn't work - multiple mutable references
    // let ref1 = &mut data;
    // let ref2 = &mut data;  // Error: cannot borrow as mutable more than once
    // ref1.push(4);
    // ref2.push(5);
    
    // Solution: Use references sequentially
    {
        let ref1 = &mut data;
        ref1.push(4);
    }  // ref1 goes out of scope
    
    {
        let ref2 = &mut data;
        ref2.push(5);
    }  // ref2 goes out of scope
    
    println!("Data: {:?}", data);
}
```

#### Mixing Mutable and Immutable References

```rust
fn main() {
    let mut text = String::from("hello");
    
    // This doesn't work - mixing reference types
    // let immutable = &text;
    // let mutable = &mut text;  // Error: cannot borrow as mutable
    // println!("{}", immutable);
    
    // Solution: Separate the lifetimes
    {
        let immutable = &text;
        println!("Reading: {}", immutable);
    }  // immutable goes out of scope
    
    {
        let mutable = &mut text;
        mutable.push_str(", world!");
    }  // mutable goes out of scope
    
    println!("Final: {}", text);
}
```

#### Use After Move with Mutable References

```rust
fn take_ownership(s: String) {
    println!("Took ownership: {}", s);
}

fn main() {
    let mut text = String::from("movable");
    
    // This doesn't work - can't use mutable reference after move
    // let text_ref = &mut text;
    // take_ownership(text);  // Error: text moved while borrowed
    // text_ref.push('!');
    
    // Solution: Don't create reference before move
    let text_copy = text.clone();
    take_ownership(text_copy);  // Move the copy
    
    let text_ref = &mut text;  // Now we can borrow the original
    text_ref.push('!');
    
    println!("Final text: {}", text);
}
```

## Best Practices for Mutable References

### API Design Guidelines

```rust
// Good: Take mutable reference when you need to modify
fn modify_config(config: &mut Config) {
    config.debug = true;
    config.timeout = 60;
}

// Good: Return mutable reference for chaining
fn get_name_mut(person: &mut Person) -> &mut String {
    &mut person.name
}

// Avoid: Taking ownership when modification is the goal
fn bad_modify_config(mut config: Config) -> Config {
    config.debug = true;
    config.timeout = 60;
    config  // Caller has to handle returned value
}
```

### Minimize Mutable Reference Scope

```rust
fn good_scoping_example() {
    let mut data = vec![1, 2, 3, 4, 5];
    
    // Keep mutable reference scope as small as possible
    let sum = {
        let data_ref = &mut data;
        data_ref.push(6);
        data_ref.iter().sum::()
    };  // data_ref goes out of scope here
    
    // Now data can be used again
    println!("Data: {:?}, Sum: {}", data, sum);
}
```

### Use Helper Methods for Complex Mutations

```rust
struct DataProcessor {
    data: Vec,
}

impl DataProcessor {
    fn new(data: Vec) -> Self {
        DataProcessor { data }
    }
    
    // Helper methods for specific mutations
    fn normalize(&mut self) {
        let max = *self.data.iter().max().unwrap_or(&1);
        for item in &mut self.data {
            *item = *item * 100 / max;
        }
    }
    
    fn remove_outliers(&mut self, threshold: i32) {
        self.data.retain(|&x| x  &[i32] {
        &self.data
    }
}

fn main() {
    let mut processor = DataProcessor::new(vec![1, 5, 3, 8, 2, 8, 10, 1]);
    
    processor.normalize();
    processor.remove_outliers(90);
    processor.sort_and_dedupe();
    
    println!("Processed data: {:?}", processor.get_data());
}
```

## Summary and Key Takeaways

### **Core Concepts**

**Mutable references** provide:
- **Exclusive write access** to borrowed data
- **Safe mutation** without ownership transfer
- **Compile-time enforcement** of exclusivity rules
- **Zero-cost abstractions** with no runtime overhead

### **Key Rules**

- **Only one mutable reference** can exist at a time
- **Cannot mix mutable and immutable references** simultaneously
- **Original data must be declared mutable** (`mut`)
- **Mutable references don't own** the data they modify

### **Best Practices**

```rust
// Good: Clear, scoped mutable borrowing
fn process_data(data: &mut Vec) {
    data.retain(|&x| x > 0);  // Modify in place
    data.sort();              // Another in-place operation
}

// Good: Return mutable references for chaining
fn get_field_mut(&mut self) -> &mut FieldType {
    &mut self.field
}

// Avoid: Overly complex mutable reference patterns
fn complex_bad_example(data: &mut Vec) -> &mut &mut String {
    // Too many levels of indirection
    data.get_mut(0).unwrap()
}
```

### **Performance Benefits**

- **In-place modifications** avoid unnecessary copying
- **Zero runtime overhead** - references are compile-time constructs
- **Memory efficiency** - no additional allocations for simple mutations
- **Cache-friendly** - modifications happen in original memory locations

### **Memory Safety Guarantees**

- **Prevents data races** through exclusivity enforcement
- **Eliminates iterator invalidation** bugs common in other languages
- **Ensures consistent data** by preventing concurrent modification
- **Compile-time checking** catches borrowing violations early

Understanding mutable references is crucial for effective Rust programming. **They enable safe, efficient data modification while maintaining Rust's ownership guarantees**. The exclusivity rules may seem restrictive at first, but they prevent entire classes of bugs that are common in other systems programming languages. Use mutable references whenever you need to modify data without taking ownership, and design your APIs to accept mutable references for maximum flexibility and safety.

- [1] https://dhghomon.github.io/easy_rust/Chapter_17.html
- [2] https://stackoverflow.com/questions/60324626/how-borrow-as-mutable-vs-immutable-in-rust
- [3] https://www.youtube.com/watch?v=4RZzjXmXcKg
- [4] https://notes.kodekloud.com/docs/Rust-Programming/Ownership/Rules-of-References
- [5] https://rainingcomputers.blog/dist/the_intuition_behind_rusts_borrowing_rules_and_ownership.md
- [6] https://www.reddit.com/r/rust/comments/16faoil/understanding_references_coming_from_other/
- [7] https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/mutability.html
- [8] https://users.rust-lang.org/t/borrowing-from-a-mutable-reference/94495
- [9] https://users.rust-lang.org/t/mutable-vs-exclusive-reference/88286?page=2
- [10] https://users.rust-lang.org/t/does-the-mutable-variable-is-a-mutable-reference-in-rust/91113
- [11] https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/references-and-borrowing.html
- [12] https://users.rust-lang.org/t/mutable-vs-exclusive-reference/88286
- [13] https://www.reddit.com/r/learnrust/comments/157xevu/mutable_references_and_moves/
- [14] https://www.youtube.com/watch?v=Q_0yoX07Fhs
- [15] https://rust-book.cs.brown.edu/ch04-02-references-and-borrowing.html
- [16] https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html
- [17] https://www.reddit.com/r/rust/comments/xmlmso/rust_beginner_struggling_to_understand_why_this/
- [18] https://users.rust-lang.org/t/exclusive-references-and-dereference-operator/58874
- [19] https://doc.rust-lang.org/std/primitive.reference.html
- [20] https://www.andy-pearce.com/blog/posts/2019/Jun/uncovering-rust-references-and-ownership/
