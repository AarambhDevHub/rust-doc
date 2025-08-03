+++
title = "Copy Trait in Rust"
description = "Learn the basics of the Copy trait in Rust and how it enables value duplication without moving ownership."
date = 2025-08-02
draft = false
weight = 5
+++

# Copy Trait Basics in Rust: Comprehensive Documentation

Building on your understanding of move semantics and ownership rules, let's explore the **Copy trait** - a fundamental marker trait that fundamentally changes how Rust handles value assignment and function calls. Understanding the Copy trait is essential for comprehending why some types move while others copy, and how this affects your program's performance and memory management.

## What is the Copy Trait?

The **Copy trait** is a **marker trait** that indicates a type's values can be duplicated simply by copying bits. When a type implements Copy, assignment creates a new copy of the data rather than moving ownership. This trait is fundamental to Rust's type system and determines whether values follow move semantics or copy semantics.

### Copy Trait Definition

```rust
// Simplified version of the actual Copy trait
pub trait Copy: Clone {}

// The actual Copy trait is a marker trait with no methods
// It requires the type to also implement Clone
```

**Key characteristics:**
- **Marker trait** - Has no methods, just indicates capability
- **Automatic behavior** - Compiler generates copying behavior
- **Requires Clone** - All Copy types must also implement Clone
- **Bitwise copy** - Creates exact bit-for-bit duplicates

## Copy vs Move: The Fundamental Distinction

### Copy Types Behavior

```rust
fn copy_behavior_demo() {
    // Copy types: assignment creates a duplicate
    let x = 5;        // i32 implements Copy
    let y = x;        // x is copied to y
    
    println!("x: {}, y: {}", x, y);  // Both are valid and usable
    
    // Function calls with Copy types
    fn use_number(n: i32) {
        println!("Number: {}", n);
    }
    
    use_number(x);    // x is copied to function parameter
    println!("x is still valid: {}", x);  // x remains accessible
}
```

### Move Types Behavior (Review)

```rust
fn move_behavior_demo() {
    // Move types: assignment transfers ownership
    let s1 = String::from("hello");  // String does NOT implement Copy
    let s2 = s1;                     // s1 is moved to s2
    
    // println!("s1: {}, s2: {}", s1, s2);  // Error: s1 moved
    println!("s2: {}", s2);  // Only s2 is valid
    
    // Function calls with Move types
    fn use_string(s: String) {
        println!("String: {}", s);
    }
    
    use_string(s2);  // s2 is moved to function
    // println!("s2: {}", s2);  // Error: s2 moved
}
```

## Built-in Types That Implement Copy

### Primitive Types

All of Rust's primitive types implement Copy:

```rust
fn primitive_copy_types() {
    // Integer types
    let i8_val: i8 = 42;
    let i16_val: i16 = 1000;
    let i32_val: i32 = 100000;
    let i64_val: i64 = 1000000000;
    let i128_val: i128 = 123456789012345678901234567890;
    let isize_val: isize = 42;
    
    // Unsigned integer types
    let u8_val: u8 = 255;
    let u16_val: u16 = 65535;
    let u32_val: u32 = 4294967295;
    let u64_val: u64 = 18446744073709551615;
    let u128_val: u128 = 340282366920938463463374607431768211455;
    let usize_val: usize = 42;
    
    // Floating-point types
    let f32_val: f32 = 3.14159;
    let f64_val: f64 = 3.141592653589793;
    
    // Boolean type
    let bool_val: bool = true;
    
    // Character type
    let char_val: char = '🦀';
    
    // All of these can be copied freely
    let i32_copy = i32_val;  // Copied, not moved
    let f64_copy = f64_val;  // Copied, not moved
    let bool_copy = bool_val; // Copied, not moved
    let char_copy = char_val; // Copied, not moved
    
    println!("All values copied successfully!");
}
```

### Compound Types That Implement Copy

```rust
fn compound_copy_types() {
    // Arrays of Copy types
    let array_copy: [i32; 5] = [1, 2, 3, 4, 5];
    let array_duplicate = array_copy;  // Entire array copied
    println!("Original: {:?}, Copy: {:?}", array_copy, array_duplicate);
    
    // Tuples of Copy types
    let tuple_copy: (i32, f64, bool) = (42, 3.14, true);
    let tuple_duplicate = tuple_copy;  // Entire tuple copied
    println!("Original: {:?}, Copy: {:?}", tuple_copy, tuple_duplicate);
    
    // Structs with only Copy fields (if derive Copy)
    #[derive(Copy, Clone, Debug)]
    struct Point {
        x: f64,
        y: f64,
    }
    
    let point_original = Point { x: 10.0, y: 20.0 };
    let point_copy = point_original;  // Struct copied
    println!("Original: {:?}, Copy: {:?}", point_original, point_copy);
    
    // Enums with only Copy variants
    #[derive(Copy, Clone, Debug)]
    enum Direction {
        North,
        South,
        East,
        West,
    }
    
    let dir_original = Direction::North;
    let dir_copy = dir_original;  // Enum copied
    println!("Original: {:?}, Copy: {:?}", dir_original, dir_copy);
}
```

### Reference Types

```rust
fn reference_copy_behavior() {
    let value = 42;
    let ref1 = &value;      // Immutable reference
    let ref2 = ref1;        // Reference itself is copied
    
    // Both references point to the same data
    println!("ref1: {}, ref2: {}", ref1, ref2);  // Both valid
    println!("Same address: {}", std::ptr::eq(ref1, ref2));  // true
    
    // Raw pointers also implement Copy
    let raw_ptr = &value as *const i32;
    let raw_copy = raw_ptr;  // Pointer copied
    
    unsafe {
        println!("raw_ptr: {:?}, raw_copy: {:?}", raw_ptr, raw_copy);
    }
}
```

## Types That Do NOT Implement Copy

### Heap-Allocated Types

Types that manage heap-allocated resources cannot implement Copy:

```rust
fn non_copy_types() {
    // String - manages heap-allocated text
    let string1 = String::from("hello");
    let string2 = string1;  // Move, not copy
    // println!("{}", string1);  // Error: string1 moved
    
    // Vec - manages heap-allocated array
    let vec1 = vec![1, 2, 3];
    let vec2 = vec1;  // Move, not copy
    // println!("{:?}", vec1);  // Error: vec1 moved
    
    // HashMap - manages heap-allocated hash table
    use std::collections::HashMap;
    let mut map1 = HashMap::new();
    map1.insert("key", "value");
    let map2 = map1;  // Move, not copy
    // println!("{:?}", map1);  // Error: map1 moved
    
    // Box - manages heap-allocated value
    let box1 = Box::new(42);
    let box2 = box1;  // Move, not copy
    // println!("{}", box1);  // Error: box1 moved
    
    println!("All non-Copy types moved successfully");
}
```

### Structs with Non-Copy Fields

```rust
fn structs_with_non_copy_fields() {
    // This struct cannot implement Copy because String doesn't implement Copy
    struct Person {
        name: String,  // String does NOT implement Copy
        age: u32,      // u32 implements Copy
    }
    
    let person1 = Person {
        name: String::from("Alice"),
        age: 30,
    };
    
    let person2 = person1;  // Move, not copy
    // println!("{}", person1.name);  // Error: person1 moved
    
    // Even if only one field is non-Copy, the entire struct is non-Copy
    struct MixedStruct {
        numbers: Vec<i32>,  // Non-Copy
        count: usize,       // Copy
        active: bool,       // Copy
    }
    
    let mixed1 = MixedStruct {
        numbers: vec![1, 2, 3],
        count: 3,
        active: true,
    };
    
    let mixed2 = mixed1;  // Move, not copy
    // println!("{}", mixed1.count);  // Error: mixed1 moved
}
```

## Implementing Copy for Custom Types

### Requirements for Copy Implementation

To implement Copy for a custom type, **all fields must also implement Copy**:

```rust
// This works - all fields implement Copy
#[derive(Copy, Clone, Debug)]
struct Point2D {
    x: f64,    // f64 implements Copy
    y: f64,    // f64 implements Copy
}

#[derive(Copy, Clone, Debug)]  
struct Color {
    r: u8,     // u8 implements Copy
    g: u8,     // u8 implements Copy  
    b: u8,     // u8 implements Copy
    a: u8,     // u8 implements Copy
}

#[derive(Copy, Clone, Debug)]
struct Pixel {
    position: Point2D,  // Point2D implements Copy
    color: Color,       // Color implements Copy
}

fn custom_copy_types() {
    let point1 = Point2D { x: 10.0, y: 20.0 };
    let point2 = point1;  // Copied, not moved
    
    let color1 = Color { r: 255, g: 0, b: 0, a: 255 };
    let color2 = color1;  // Copied, not moved
    
    let pixel1 = Pixel { position: point1, color: color1 };
    let pixel2 = pixel1;  // Copied, not moved
    
    // All original values still accessible
    println!("point1: {:?}, point2: {:?}", point1, point2);
    println!("color1: {:?}, color2: {:?}", color1, color2);
    println!("pixel1: {:?}, pixel2: {:?}", pixel1, pixel2);
}
```

### Copy with Enums

```rust
// Enum with only Copy variants
#[derive(Copy, Clone, Debug)]
enum Status {
    Active,
    Inactive,
    Pending(u32),    // u32 implements Copy
    Error(i32),      // i32 implements Copy
}

// This enum CANNOT implement Copy
enum Message {
    Text(String),    // String does NOT implement Copy
    Number(i32),     // i32 implements Copy
    Quit,
}

fn enum_copy_behavior() {
    let status1 = Status::Active;
    let status2 = status1;  // Copied
    
    let status3 = Status::Pending(42);
    let status4 = status3;  // Copied
    
    println!("status1: {:?}, status2: {:?}", status1, status2);
    println!("status3: {:?}, status4: {:?}", status3, status4);
    
    // Message enum would move, not copy
    let msg1 = Message::Text(String::from("hello"));
    let msg2 = msg1;  // Moved
    // println!("{:?}", msg1);  // Error: msg1 moved
}
```

### Manual Copy Implementation

You rarely need to implement Copy manually, but here's how it looks:

```rust
// Manual implementation (usually use derive instead)
#[derive(Clone, Debug)]
struct ManualCopy {
    value: i32,
}

// Manual Copy implementation
impl Copy for ManualCopy {}

// Or more commonly, just derive it
#[derive(Copy, Clone, Debug)]
struct AutoCopy {
    value: i32,
}

fn manual_vs_auto_copy() {
    let manual1 = ManualCopy { value: 42 };
    let manual2 = manual1;  // Copied
    
    let auto1 = AutoCopy { value: 42 };
    let auto2 = auto1;  // Copied
    
    println!("manual1: {:?}, manual2: {:?}", manual1, manual2);
    println!("auto1: {:?}, auto2: {:?}", auto1, auto2);
}
```

## Copy Semantics in Different Contexts

### Function Parameters

```rust
fn copy_in_functions() {
    #[derive(Copy, Clone, Debug)]
    struct Data {
        value: i32,
        flag: bool,
    }
    
    fn process_copy_type(data: Data) {  // Parameter receives copy
        println!("Processing: {:?}", data);
        // data goes out of scope, but original is unaffected
    }
    
    fn process_reference(data: &Data) {  // Parameter receives reference
        println!("Processing reference: {:?}", data);
        // No copying, just borrowing
    }
    
    let original = Data { value: 42, flag: true };
    
    process_copy_type(original);  // original copied to function
    println!("Original still valid: {:?}", original);
    
    process_reference(&original);  // original borrowed
    println!("Original still valid: {:?}", original);
}
```

### Return Values

```rust
fn copy_in_return_values() {
    #[derive(Copy, Clone, Debug)]
    struct Config {
        timeout: u32,
        retries: u8,
    }
    
    fn create_default_config() -> Config {
        Config {
            timeout: 30,
            retries: 3,
        }
    }
    
    fn modify_config(mut config: Config) -> Config {
        config.timeout = 60;
        config  // Return by value (copied out)
    }
    
    let config1 = create_default_config();  // Copy returned
    let config2 = modify_config(config1);   // config1 copied in, result copied out
    
    // config1 is still valid because it was copied
    println!("Original config: {:?}", config1);
    println!("Modified config: {:?}", config2);
}
```

### Collection Operations

```rust
fn copy_in_collections() {
    // Vector of Copy types
    let numbers = vec![1, 2, 3, 4, 5];
    
    // Accessing elements of Copy types creates copies
    let first = numbers[0];     // i32 copied out of vector
    let second = *numbers.get(1).unwrap();  // i32 copied out
    
    // Original vector elements unchanged
    println!("first: {}, second: {}", first, second);
    println!("Vector: {:?}", numbers);
    
    // Iterator operations with Copy types
    let doubled: Vec = numbers.iter()
        .map(|&x| x * 2)  // x is copied from the reference
        .collect();
    
    println!("Doubled: {:?}", doubled);
    println!("Original: {:?}", numbers);  // Still valid
    
    // Custom Copy types in collections
    #[derive(Copy, Clone, Debug)]
    struct Point { x: i32, y: i32 }
    
    let points = vec![
        Point { x: 1, y: 2 },
        Point { x: 3, y: 4 },
        Point { x: 5, y: 6 },
    ];
    
    let first_point = points[0];  // Point copied out
    println!("First point: {:?}", first_point);
    println!("Points vector: {:?}", points);  // Still valid
}
```

## Memory Implications of Copy

### Stack Allocation

Copy types are typically stored on the stack and copying is very efficient:

```rust
fn copy_memory_layout() {
    #[derive(Copy, Clone, Debug)]
    struct SmallStruct {
        a: u32,  // 4 bytes
        b: u16,  // 2 bytes
        c: u8,   // 1 byte
        // + padding for alignment
    }
    
    let original = SmallStruct { a: 1, b: 2, c: 3 };
    let copy = original;  // Entire struct copied on stack
    
    // Both variables occupy separate stack space
    println!("Original address: {:p}", &original);
    println!("Copy address: {:p}", &copy);
    println!("Size of SmallStruct: {} bytes", std::mem::size_of::<SmallStruct>());
    
    // Copying is just a fast memory copy operation
    let mut copies = Vec::new();
    for _ in 0..1000 {
        copies.push(original);  // Fast copy operation each time
    }
    
    println!("Created {} copies efficiently", copies.len());
}
```

### Performance Characteristics

```rust
use std::time::Instant;

fn copy_performance() {
    #[derive(Copy, Clone)]
    struct LargeStruct {
        data: [u64; 100],  // 800 bytes
    }
    
    let large = LargeStruct { data: [42; 100] };
    
    // Benchmark copying
    let start = Instant::now();
    let mut copies = Vec::new();
    for _ in 0..100_000 {
        copies.push(large);  // Copy operation
    }
    let copy_time = start.elapsed();
    
    println!("Copied 100k large structs in: {:?}", copy_time);
    println!("Each copy: {} bytes", std::mem::size_of::<LargeStruct>());
    
    // Compare with move operations
    let mut vec_of_boxes = Vec::new();
    let start = Instant::now();
    for _ in 0..100_000 {
        vec_of_boxes.push(Box::new([42u64; 100]));  // Heap allocation + move
    }
    let move_time = start.elapsed();
    
    println!("Created 100k boxed arrays in: {:?}", move_time);
    println!("Move is {}x slower than copy", 
             move_time.as_nanos() / copy_time.as_nanos().max(1));
}
```

## Copy vs Clone

### Understanding the Relationship

Copy and Clone are related but distinct:

```rust
fn copy_vs_clone() {
    #[derive(Copy, Clone, Debug)]
    struct CopyType {
        value: i32,
    }
    
    #[derive(Clone, Debug)]
    struct CloneOnlyType {
        data: String,
    }
    
    let copy_val = CopyType { value: 42 };
    
    // Copy types: assignment creates implicit copy
    let copy_duplicate = copy_val;  // Automatic copying
    println!("Original: {:?}, Copy: {:?}", copy_val, copy_duplicate);
    
    // Copy types: explicit clone also works
    let copy_cloned = copy_val.clone();  // Explicit cloning
    println!("Cloned: {:?}", copy_cloned);
    
    let clone_val = CloneOnlyType { data: String::from("hello") };
    
    // Clone-only types: assignment moves
    let clone_moved = clone_val;  // Move, not copy
    // println!("{:?}", clone_val);  // Error: clone_val moved
    
    // Clone-only types: explicit clone required for duplication
    let clone_val2 = CloneOnlyType { data: String::from("world") };
    let clone_duplicated = clone_val2.clone();  // Explicit cloning
    println!("Original: {:?}, Cloned: {:?}", clone_val2, clone_duplicated);
}
```

### When to Use Each

```rust
fn when_to_use_copy_vs_clone() {
    // Use Copy for small, simple types
    #[derive(Copy, Clone, Debug)]
    struct Coordinates {
        x: f64,
        y: f64,
        z: f64,
    }
    
    // Use Clone for types that manage resources
    #[derive(Clone, Debug)]
    struct ResourceManager {
        name: String,
        connections: Vec<String>,
    }
    
    let coords = Coordinates { x: 1.0, y: 2.0, z: 3.0 };
    let coords_copy = coords;  // Fast, automatic copy
    
    let manager = ResourceManager {
        name: String::from("Database"),
        connections: vec![String::from("conn1"), String::from("conn2")],
    };
    let manager_clone = manager.clone();  // Explicit, potentially expensive clone
    
    println!("Coordinates copied: {:?}", coords_copy);
    println!("Manager cloned: {:?}", manager_clone);
}
```

## Advanced Copy Patterns

### Copy in Generic Contexts

```rust
fn copy_with_generics() {
    // Generic function that works with Copy types
    fn duplicate_value<T: Copy>(value: T) -> (T, T) {
        (value, value)  // Both copies of the same value
    }
    
    // Works with any Copy type
    let (a, b) = duplicate_value(42i32);
    let (c, d) = duplicate_value(3.14f64);
    let (e, f) = duplicate_value(true);
    
    println!("Integers: {}, {}", a, b);
    println!("Floats: {}, {}", c, d);
    println!("Booleans: {}, {}", e, f);
    
    // Generic struct that requires Copy types
    #[derive(Debug)]
    struct Pair<T: Copy>  {
        first: T,
        second: T,
    }
    
    impl<T: Copy> Pair<T> {
        fn new(value: T) -> Self {
            Pair {
                first: value,   // value copied
                second: value,  // value copied again
            }
        }
        
        fn swap(self) -> Self {  // Takes self by value
            Pair {
                first: self.second,  // Copy values
                second: self.first,  // Copy values
            }
        }
    }
    
    let pair = Pair::new(42);
    let swapped = pair.swap();  // pair consumed, swapped returned
    
    println!("Swapped pair: {:?}", swapped);
}
```

### Copy with Traits

```rust
fn copy_with_traits() {
    trait Drawable: Copy {
        fn draw(&self);
    }
    
    #[derive(Copy, Clone)]
    struct Point {
        x: i32,
        y: i32,
    }
    
    impl Drawable for Point {
        fn draw(&self) {
            println!("Drawing point at ({}, {})", self.x, self.y);
        }
    }
    
    fn draw_twice(item: T) {
        item.draw();  // item is copied when passed to draw
        item.draw();  // item can be used again because it implements Copy
    }
    
    let point = Point { x: 10, y: 20 };
    draw_twice(point);  // point is copied into function
    
    println!("Point still available after function call");
    point.draw();  // point is still valid
}
```

## Common Pitfalls and Best Practices

### When NOT to Implement Copy

```rust
// Don't implement Copy for types that manage resources
struct FileHandle {
    path: String,
    // file descriptor would go here
}

// This would be dangerous if Copy were implemented:
// impl Copy for FileHandle {}  // DON'T DO THIS!

// Problem: Multiple copies would try to close the same file
fn dangerous_if_copy_existed() {
    // If FileHandle implemented Copy:
    // let handle1 = FileHandle { path: String::from("file.txt") };
    // let handle2 = handle1;  // Copy would create two handles to same file
    // 
    // When both go out of scope, both would try to close the file
    // This could lead to double-close errors
}
```

### Large Copy Types Can Be Inefficient

```rust
fn large_copy_considerations() {
    #[derive(Copy, Clone)]
    struct LargeArray {
        data: [u8; 4096],  // 4KB of data
    }
    
    let large = LargeArray { data: [0; 4096] };
    
    // Each assignment copies 4KB
    let copy1 = large;  // 4KB copied
    let copy2 = large;  // Another 4KB copied
    let copy3 = large;  // Another 4KB copied
    
    // This might be inefficient for very large types
    // Consider using references or Box for large data
    
    // Alternative approach for large data:
    struct LargeData {
        data: Box<[u8; 4096]>,  // Heap-allocated, moves instead of copies
    }
    
    let large_data = LargeData {
        data: Box::new([0; 4096]),
    };
    
    // This moves the Box (just a pointer), not the 4KB array
    let moved_data = large_data;  // Only pointer moved, not 4KB
}
```

### Copy and Interior Mutability

```rust
use std::cell::Cell;

fn copy_with_interior_mutability() {
    // Cell implements Copy if T implements Copy
    #[derive(Copy, Clone)]
    struct Counter {
        value: Cell,  // Cell implements Copy because u32 does
    }
    
    let counter1 = Counter {
        value: Cell::new(0),
    };
    
    let counter2 = counter1;  // Copied
    
    // Both counters are independent
    counter1.value.set(10);
    counter2.value.set(20);
    
    println!("counter1: {}", counter1.value.get());  // 10
    println!("counter2: {}", counter2.value.get());  // 20
    
    // The copy created independent Cell instances
}
```

## Testing Copy Behavior

### Compile-Time Checks

```rust
fn compile_time_copy_checks() {
    // Function that only accepts Copy types
    fn requires_copy(_value: T) {
        println!("This type implements Copy");
    }
    
    // These work
    requires_copy(42i32);
    requires_copy(3.14f64);
    requires_copy(true);
    requires_copy('A');
    requires_copy([1, 2, 3, 4, 5]);
    requires_copy((1, 2, 3));
    
    // These would fail to compile:
    // requires_copy(String::from("hello"));     // String doesn't implement Copy
    // requires_copy(vec![1, 2, 3]);            // Vec doesn't implement Copy
    // requires_copy(Box::new(42));             // Box doesn't implement Copy
}
```

### Runtime Verification

```rust
fn runtime_copy_verification() {
    use std::any::type_name;
    
    fn is_copy<T: 'static>() -> bool {
        std::any::TypeId::of::() == std::any::TypeId::of::()
        // This is a simplified example; in practice, you'd use trait bounds
    }
    
    #[derive(Copy, Clone)]
    struct CopyStruct;
    
    struct NonCopyStruct;
    
    // Check at compile time with trait bounds
    fn check_copy_compile_time() {
        fn needs_copy() {}
        
        needs_copy::();          // Compiles
        needs_copy::();   // Compiles
        // needs_copy::(); // Doesn't compile
        // needs_copy::();       // Doesn't compile
    }
    
    check_copy_compile_time();
    
    println!("Copy checks completed");
}
```

## Performance Considerations

### When Copy is Beneficial

```rust
fn when_copy_is_beneficial() {
    // Small data structures benefit from Copy
    #[derive(Copy, Clone, Debug)]
    struct RGB {
        r: u8,
        g: u8,
        b: u8,
    }
    
    // Passing by value is efficient for small Copy types
    fn blend_colors(color1: RGB, color2: RGB) -> RGB {
        RGB {
            r: (color1.r as u16 + color2.r as u16 / 2) as u8,
            g: (color1.g as u16 + color2.g as u16 / 2) as u8,
            b: (color1.b as u16 + color2.b as u16 / 2) as u8,
        }
    }
    
    let red = RGB { r: 255, g: 0, b: 0 };
    let blue = RGB { r: 0, g: 0, b: 255 };
    
    // Efficient: small structures passed by value
    let purple = blend_colors(red, blue);  // red and blue copied
    
    // Original colors still available
    println!("Red: {:?}, Blue: {:?}, Purple: {:?}", red, blue, purple);
}
```

### When to Avoid Copy

```rust
fn when_to_avoid_copy() {
    // Large structures should not implement Copy
    struct LargeMatrix {
        data: [[f64; 100]; 100],  // 80KB of data
    }
    
    // If this implemented Copy, every assignment would copy 80KB
    // Instead, use references or Box for large data
    
    impl LargeMatrix {
        // Methods that work with references to avoid copying
        fn determinant(&self) -> f64 {
            // Calculate determinant without copying the matrix
            0.0  // Placeholder
        }
        
        // Methods that take ownership when transformation is needed
        fn transpose(mut self) -> Self {
            // Transpose the matrix in place and return ownership
            self  // Placeholder
        }
    }
    
    let matrix = LargeMatrix {
        data: [[0.0; 100]; 100],
    };
    
    // Use references for read-only operations
    let det = matrix.determinant();  // Borrow, don't copy
    
    // Move for transformations
    let transposed = matrix.transpose();  // Move, don't copy
    
    println!("Matrix operations completed efficiently");
}
```

## Summary and Best Practices

### **Core Concepts**

- **Copy trait enables bitwise duplication** of values
- **Assignment creates copies** instead of moves for Copy types
- **All Copy types must also implement Clone**
- **Copy is automatic** - no explicit method calls needed

### **When to Implement Copy**

```rust
// Good candidates for Copy:
// - Small, simple data structures
// - Types with only stack-allocated data
// - Types that represent values rather than resources

#[derive(Copy, Clone)]
struct GoodCopyCandidate {
    x: f64,
    y: f64,
    color: u32,
}

// Bad candidates for Copy:
// - Types that manage heap-allocated resources
// - Large data structures
// - Types that represent unique resources

struct BadCopyCandidate {
    name: String,           // Heap-allocated
    data: Vec,         // Heap-allocated
    file: std::fs::File,   // Unique resource
}
```

### **Performance Guidelines**

- **Use Copy for small types** (typically < 100 bytes)
- **Avoid Copy for large types** to prevent expensive memory copies
- **Consider the frequency of assignment** - Copy types are efficient for frequent copying
- **Benchmark when in doubt** - measure the performance impact

### API Design Principles
```rust
// Design functions appropriately for Copy vs non-Copy types

// For Copy types: pass by value is often efficient
fn process_point(point: Point2D) -> Point2D {
    // Small Copy type passed by value
    point
}

// For non-Copy types: prefer borrowing when possible
fn process_string(text: &str) -> usize {
    // Borrow instead of taking ownership
    text.len()
}
```

### 🧠 Optional Suggestions:

- Consider adding a small summary table contrasting Copy, Clone, and Move.
- A visual diagram (if part of a site) showing ownership, copying, and movement can help visual learners.
- For learning purposes, highlight common compiler errors like value borrowed here after move and explain why they happen with non-Copy types.

Understanding the Copy trait is fundamental to writing efficient Rust code. **Use Copy judiciously for small, value-like types**, and **avoid it for resource-managing types or large data structures**. The Copy trait provides a powerful way to opt into value semantics while maintaining Rust's safety guarantees, but it should be used thoughtfully to avoid performance pitfalls.