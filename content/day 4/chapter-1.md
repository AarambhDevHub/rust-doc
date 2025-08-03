+++
title = "Compound Types (Tuples, Arrays)"
description = "Explore compound types in Rust, focusing on tuples and arrays, how to use them, access elements, destructure, and apply them in real-world functions and patterns."
date = 2025-08-01
draft = false
weight = 1
tags = ["rust", "compound types", "tuples", "arrays", "data structures", "beginners", "guide"]
+++


# Compound Types in Rust: Tuples and Arrays in Detail

Building on your understanding of basic data types and loops, let's explore Rust's **compound types** - data structures that can group multiple values together. Rust provides several compound types, but the two most fundamental are **tuples** and **arrays**. These types allow you to store multiple values in a single variable while maintaining Rust's safety guarantees.

## Tuples - Heterogeneous Collections

**Tuples** are compound types that can hold multiple values of **different types** together. They have a **fixed length** and are created by grouping values in parentheses with comma separation.

### Basic Tuple Syntax

```rust
fn main() {
    // Basic tuple creation
    let tuple: (i32, f64, bool) = (42, 3.14, true);
    
    // Type inference also works
    let coordinates = (10, 20);        // (i32, i32)
    let person = ("Alice", 30, true);  // (&str, i32, bool)
    
    println!("Tuple: {:?}", tuple);
    println!("Person: {:?}", person);
}
```

**Output:**

```
Tuple: (42, 3.14, true)
Person: ("Alice", 30, true)
```

### Accessing Tuple Elements

There are multiple ways to access tuple elements:

#### 1. Dot Notation with Index

```rust
fn main() {
    let point = (3, 4);
    
    let x = point.0;  // First element (index 0)
    let y = point.1;  // Second element (index 1)
    
    println!("Point: ({}, {})", x, y);
    println!("X: {}, Y: {}", point.0, point.1);
}
```

#### 2. Destructuring Assignment

```rust
fn main() {
    let person = ("Bob", 25, false);
    
    // Destructure the tuple
    let (name, age, is_student) = person;
    
    println!("Name: {}", name);
    println!("Age: {}", age);
    println!("Is student: {}", is_student);
}
```

#### 3. Partial Destructuring

```rust
fn main() {
    let data = (1, 2, 3, 4, 5);
    
    // Only extract what you need
    let (first, _, third, ..) = data;  // Skip second, ignore rest
    
    println!("First: {}, Third: {}", first, third);
}
```

### Tuple Types and Mixed Data

**Tuples can contain any combination of types**:

```rust
fn main() {
    // Mixed types in a single tuple
    let mixed = (
        42,                    // i32
        3.14159,              // f64
        "hello",              // &str
        true,                 // bool
        [1, 2, 3],           // array
        ('a', 'b'),          // nested tuple
    );
    
    println!("Integer: {}", mixed.0);
    println!("Float: {}", mixed.1);
    println!("String: {}", mixed.2);
    println!("Boolean: {}", mixed.3);
    println!("Array: {:?}", mixed.4);
    println!("Nested tuple: {:?}", mixed.5);
}
```

### Empty and Single-Element Tuples

#### Unit Tuple `()`

```rust
fn main() {
    let unit = ();  // Unit type - represents no meaningful value
    println!("Unit: {:?}", unit);
    
    // Functions that don't return anything implicitly return ()
    let result = println!("Hello");  // result is ()
}
```

#### Single-Element Tuple

```rust
fn main() {
    let single = (42,);  // Note the comma! Without it, it's just parentheses
    let not_tuple = (42); // This is just an i32 in parentheses
    
    println!("Single tuple: {:?}", single);
    println!("Not a tuple: {:?}", not_tuple);
}
```

### Tuples in Functions

#### Returning Multiple Values

```rust
fn calculate_stats(numbers: &[i32]) -> (i32, i32, f64) {
    let min = *numbers.iter().min().unwrap();
    let max = *numbers.iter().max().unwrap();
    let avg = numbers.iter().sum::<i32>() as f64 / numbers.len() as f64;
    
    (min, max, avg)  // Return tuple
}

fn main() {
    let data = [1, 5, 3, 9, 2, 7];
    let (min, max, average) = calculate_stats(&data);
    
    println!("Min: {}, Max: {}, Average: {:.2}", min, max, average);
}
```

#### Function Parameters

```rust
fn distance_from_origin(point: (f64, f64)) -> f64 {
    let (x, y) = point;
    (x * x + y * y).sqrt()
}

fn print_coordinates((x, y): (i32, i32)) {  // Direct destructuring in parameter
    println!("Point at ({}, {})", x, y);
}

fn main() {
    let point = (3.0, 4.0);
    let dist = distance_from_origin(point);
    println!("Distance: {:.2}", dist);
    
    print_coordinates((10, 20));
}
```

### When to Use Tuples

**Use tuples when:**

* You need to return multiple related values from a function
* Values are of different types but conceptually related
* The grouping is temporary or internal to a function
* You have a small, fixed set of values (typically 2-4 elements)

```rust
// Good use cases
fn divide_with_remainder(dividend: i32, divisor: i32) -> (i32, i32) {
    (dividend / divisor, dividend % divisor)
}

fn parse_coordinate(input: &str) -> Option<(f64, f64)> {
    let parts: Vec<&str> = input.split(',').collect();
    if parts.len() == 2 {
        if let (Ok(x), Ok(y)) = (parts[0].trim().parse(), parts[1].trim().parse()) {
            return Some((x, y));
        }
    }
    None
}
```

## Arrays - Homogeneous Collections

**Arrays** in Rust are collections of elements that:

* All have the **same type**
* Have a **fixed size** known at compile time
* Are stored on the **stack** (not heap)
* Use **zero-based indexing**

### Basic Array Syntax

```rust
fn main() {
    // Explicit type and size
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];
    
    // Type inference
    let fruits = ["apple", "banana", "cherry"];
    
    // Repeat syntax - initialize with same value
    let zeros = [0; 10];  // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    
    println!("Numbers: {:?}", numbers);
    println!("Fruits: {:?}", fruits);
    println!("Zeros: {:?}", zeros);
}
```

### Accessing Array Elements

#### Index Access

```rust
fn main() {
    let colors = ["red", "green", "blue", "yellow"];
    
    // Direct index access
    println!("First color: {}", colors[0]);
    println!("Last color: {}", colors[3]);
    
    // Using variables as index
    let index = 2;
    println!("Color at index {}: {}", index, colors[index]);
}
```

#### Safe Access with `get()`

```rust
fn main() {
    let numbers = [10, 20, 30, 40, 50];
    
    // Safe access returns Option<T>
    match numbers.get(2) {
        Some(value) => println!("Found: {}", value),
        None => println!("Index out of bounds"),
    }
    
    // Out of bounds access
    match numbers.get(10) {
        Some(value) => println!("Found: {}", value),
        None => println!("Index 10 is out of bounds"),
    }
}
```

#### Runtime Bounds Checking

```rust
fn main() {
    let data = [1, 2, 3];
    
    // This will panic at runtime if index is out of bounds
    // let value = data[5];  // thread 'main' panicked at 'index out of bounds'
    
    // Safe alternative
    if let Some(value) = data.get(5) {
        println!("Value: {}", value);
    } else {
        println!("Index out of bounds");
    }
}
```

### Array Properties and Methods

#### Length and Properties

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5];
    
    println!("Array length: {}", numbers.len());
    println!("Is empty: {}", numbers.is_empty());
    println!("First element: {:?}", numbers.first());
    println!("Last element: {:?}", numbers.last());
}
```

#### Iteration Over Arrays

```rust
fn main() {
    let temperatures = [20, 25, 30, 28, 22];
    
    // Direct iteration
    println!("Temperatures:");
    for temp in temperatures {
        println!("{}°C", temp);
    }
    
    // Iteration with index
    println!("\nIndexed temperatures:");
    for (index, temp) in temperatures.iter().enumerate() {
        println!("Day {}: {}°C", index + 1, temp);
    }
    
    // Iterator methods
    let max_temp = temperatures.iter().max().unwrap();
    let avg_temp: f64 = temperatures.iter().sum::<i32>() as f64 / temperatures.len() as f64;
    
    println!("Max temperature: {}°C", max_temp);
    println!("Average temperature: {:.1}°C", avg_temp);
}
```

### Multi-dimensional Arrays

```rust
fn main() {
    // 2D array (array of arrays)
    let matrix: [[i32; 3]; 2] = [
        [1, 2, 3],
        [4, 5, 6]
    ];
    
    println!("Matrix: {:?}", matrix);
    
    // Accessing 2D array elements
    println!("Element [0][1]: {}", matrix[0][1]);
    
    // Iterating 2D arrays
    for (row_index, row) in matrix.iter().enumerate() {
        for (col_index, value) in row.iter().enumerate() {
            println!("matrix[{}][{}] = {}", row_index, col_index, value);
        }
    }
}
```

### Arrays in Functions

#### Function Parameters

```rust
fn sum_array(numbers: [i32; 5]) -> i32 {
    numbers.iter().sum()
}

fn print_array(arr: &[i32]) {  // Slice parameter - more flexible
    for (i, value) in arr.iter().enumerate() {
        println!("Index {}: {}", i, value);
    }
}

fn main() {
    let data = [1, 2, 3, 4, 5];
    let total = sum_array(data);
    println!("Sum: {}", total);
    
    print_array(&data);  // Pass reference to array
}
```

#### Returning Arrays

```rust
fn create_sequence(start: i32) -> [i32; 5] {
    [start, start + 1, start + 2, start + 3, start + 4]
}

fn initialize_grid() -> [[i32; 3]; 3] {
    [
        [0, 0, 0],
        [0, 0, 0], 
        [0, 0, 0]
    ]
}

fn main() {
    let sequence = create_sequence(10);
    println!("Sequence: {:?}", sequence);
    
    let grid = initialize_grid();
    println!("Grid: {:?}", grid);
}
```

## Comparison: Tuples vs Arrays

| Feature             | Tuples                     | Arrays                     |
| ------------------- | -------------------------- | -------------------------- |
| **Element Types**   | Different types allowed    | All elements same type     |
| **Size**            | Fixed at compile time      | Fixed at compile time      |
| **Access Method**   | Dot notation with index    | Square bracket notation    |
| **Indexing**        | Compile-time only          | Runtime indexing supported |
| **Bounds Checking** | Always safe (compile-time) | Runtime bounds checking    |
| **Iteration**       | Not directly iterable      | Iterable with for loops    |
| **Memory**          | Stack allocated            | Stack allocated            |
| **Use Case**        | Related but different data | Collection of same data    |

## Practical Examples

### Coordinate System with Tuples

```rust
type Point2D = (f64, f64);
type Point3D = (f64, f64, f64);

fn distance_2d(p1: Point2D, p2: Point2D) -> f64 {
    let (x1, y1) = p1;
    let (x2, y2) = p2;
    ((x2 - x1).powi(2) + (y2 - y1).powi(2)).sqrt()
}

fn main() {
    let origin: Point2D = (0.0, 0.0);
    let point: Point2D = (3.0, 4.0);
    let point3d: Point3D = (1.0, 2.0, 3.0);
    
    println!("Distance from origin: {:.2}", distance_2d(origin, point));
    println!("3D Point: {:?}", point3d);
}
```

### Grade Management with Arrays

```rust
fn analyze_grades(grades: &[i32]) -> (i32, i32, f64, char) {
    let min = *grades.iter().min().unwrap();
    let max = *grades.iter().max().unwrap();
    let avg = grades.iter().sum::<i32>() as f64 / grades.len() as f64;
    
    let letter_grade = match avg as i32 {
        90..=100 => 'A',
        80..=89 => 'B',
        70..=79 => 'C',
        60..=69 => 'D',
        _ => 'F',
    };
    
    (min, max, avg, letter_grade)
}

fn main() {
    let student_grades = [95, 87, 92, 88, 91];
    let (min, max, avg, grade) = analyze_grades(&student_grades);
    
    println!("Grades: {:?}", student_grades);
    println!("Min: {}, Max: {}, Average: {:.1}, Letter Grade: {}", 
             min, max, avg, grade);
}
```

### Data Processing Pipeline

```rust
fn process_sensor_data() -> [(String, f64, bool); 3] {
    [
        ("Temperature".to_string(), 23.5, true),
        ("Humidity".to_string(), 65.0, true),
        ("Pressure".to_string(), 1013.25, false),
    ]
}

fn main() {
    let sensor_readings = process_sensor_data();
    
    println!("Sensor Data:");
    for (i, (sensor, value, active)) in sensor_readings.iter().enumerate() {
        let status = if *active { "Active" } else { "Inactive" };
        println!("Sensor {}: {} = {:.2} ({})", i + 1, sensor, value, status);
    }
}
```

## Best Practices

### **Choose the Right Type**

```rust
// Use tuple for related but different types
let person_info = ("Alice", 30, "Engineer");  // Good

// Use array for collections of same type
let scores = [95, 87, 92, 88];  // Good

// Don't use array for different types
// let mixed = [42, "hello"];  // Error: mismatched types
```

### **Prefer Descriptive Type Aliases**

```rust
type Rgb = (u8, u8, u8);
type Matrix3x3 = [[f64; 3]; 3];

fn create_color(r: u8, g: u8, b: u8) -> Rgb {
    (r, g, b)
}

fn identity_matrix() -> Matrix3x3 {
    [
        [1.0, 0.0, 0.0],
        [0.0, 1.0, 0.0],
        [0.0, 0.0, 1.0],
    ]
}
```

### **Use Safe Access for Runtime Indices**

```rust
fn safe_array_access(arr: &[i32], index: usize) -> Option<i32> {
    arr.get(index).copied()  // Safe access
}

// Avoid direct indexing with runtime values
fn unsafe_access(arr: &[i32], index: usize) -> i32 {
    arr[index]  // Can panic!
}
```

### **Destructure for Clarity**

```rust
// Good - clear what each value represents
let (width, height) = get_dimensions();

// Less clear
let dimensions = get_dimensions();
let width = dimensions.0;
let height = dimensions.1;
```

Understanding tuples and arrays is fundamental to working with data in Rust. **Tuples excel at grouping related but different types of data**, while **arrays provide efficient storage for collections of homogeneous data**. Both types maintain Rust's safety guarantees while providing the performance benefits of stack allocation. These compound types serve as building blocks for more complex data structures you'll encounter as you continue learning Rust.
