+++
title = "Data Types in Rust: Integers, Floats, Booleans, and Characters"
description = "A detailed guide to Rust’s scalar data types, including integers, floats, booleans, and characters. Learn about their syntax, usage, memory behavior, and best practices."
date = 2025-08-01
draft = false
weight = 2
+++

# Data Types in Rust: Integers, Floats, Booleans, and Characters

Building on your understanding of variables and mutability, let's explore Rust's **scalar data types** - the fundamental building blocks for storing and manipulating data. Rust has four primary scalar types that represent single values: **integers**, **floating-point numbers**, **booleans**, and **characters**.

## Integer Types

Integers in Rust represent **whole numbers** (positive or negative) without fractional components. Rust provides multiple integer types with different sizes and signed/unsigned variants.

## Integer Type Categories

**Signed Integers** (can store negative numbers):

* `i8`: -128 to 127 (8-bit)
* `i16`: -32,768 to 32,767 (16-bit)
* `i32`: -2,147,483,648 to 2,147,483,647 (32-bit) - **default**
* `i64`: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 (64-bit)
* `i128`: 128-bit signed integer
* `isize`: pointer-sized (32-bit on 32-bit systems, 64-bit on 64-bit systems)

**Unsigned Integers** (only positive numbers):

* `u8`: 0 to 255 (8-bit)
* `u16`: 0 to 65,535 (16-bit)
* `u32`: 0 to 4,294,967,295 (32-bit)
* `u64`: 0 to 18,446,744,073,709,551,615 (64-bit)
* `u128`: 128-bit unsigned integer
* `usize`: pointer-sized unsigned integer

## Integer Examples

```rust
fn main() {
    // Default i32 type (inferred)
    let default_int = 42;
    
    // Explicit type annotations
    let small_number: i8 = -50;
    let large_number: u64 = 1_000_000_000;
    let byte_value: u8 = 255;
    
    // Using underscores for readability
    let million: i32 = 1_000_000;
    
    println!("Default: {}", default_int);
    println!("Small: {}", small_number);
    println!("Large: {}", large_number);
}
```

## Integer Literals

Rust supports multiple number literal formats:

```rust
fn main() {
    let decimal = 98_222;        // Decimal
    let hex = 0xff;              // Hexadecimal
    let octal = 0o77;            // Octal
    let binary = 0b1111_0000;    // Binary
    let byte = b'A';             // Byte (u8 only)
}
```

## Floating-Point Types

Floating-point numbers represent **numbers with decimal points**. Rust has two floating-point types that follow the IEEE-754 standard:

* `f32`: 32-bit single-precision floating point
* `f64`: 64-bit double-precision floating point - **default**

## Floating-Point Examples

```rust
fn main() {
    // Default f64 type (inferred)
    let pi = 3.14159;
    
    // Explicit type annotations
    let price: f32 = 19.99;
    let scientific: f64 = 2.5e10;  // Scientific notation
    
    // Mathematical operations
    let sum = 5.5 + 10.2;
    let difference = 95.5 - 4.3;
    let product = 4.0 * 30.0;
    let quotient = 56.7 / 32.2;
    let remainder = 43.0 % 5.0;
    
    println!("Pi: {}", pi);
    println!("Price: ${}", price);
    println!("Scientific: {}", scientific);
}
```

## Floating-Point Precision

```rust
fn main() {
    let f32_val: f32 = 3.14159265359;
    let f64_val: f64 = 3.14159265359;
    
    println!("f32: {}", f32_val);  // Less precision
    println!("f64: {}", f64_val);  // More precision
}
```

## Boolean Type

The `bool` type represents **logical values** with only two possible states: `true` or `false`. Booleans are **one byte in size** and are primarily used in conditional statements and logical operations.

## Boolean Examples

```rust
fn main() {
    // Direct assignment
    let is_active = true;
    let is_complete: bool = false;  // Explicit type annotation
    
    // From comparisons
    let is_greater = 5 > 3;        // true
    let is_equal = 10 == 15;       // false
    
    // Logical operations
    let and_result = true && false;   // false
    let or_result = true || false;    // true
    let not_result = !true;           // false
    
    // Using in conditionals
    if is_active {
        println!("System is active");
    }
    
    if !is_complete {
        println!("Task is not complete");
    }
}
```

## Boolean in Practice

```rust
fn main() {
    let temperature = 25;
    let is_sunny = true;
    
    let is_good_weather = temperature > 20 && is_sunny;
    
    if is_good_weather {
        println!("Perfect day for outdoor activities!");
    } else {
        println!("Maybe stay inside today.");
    }
}
```

## Character Type

The `char` type represents a **single Unicode character** and is Rust's most primitive alphabetic type. Characters are **four bytes in size** and can represent much more than just ASCII characters.

## Character Features

* Uses **single quotes** (not double quotes like strings)
* Represents **Unicode Scalar Values** from `U+0000` to `U+D7FF` and `U+E000` to `U+10FFFF`
* Can store ASCII letters, accented characters, Chinese/Japanese/Korean characters, emojis, and more

## Character Examples:

```rust
fn main() {
    // Basic ASCII characters
    let letter = 'A';
    let digit: char = '5';
    let symbol = '@';
    
    // Unicode characters
    let accented = 'ñ';
    let chinese = '中';
    let mathematical = 'ℤ';
    let emoji = '😻';
    let heart = '♥';
    
    // Special characters
    let newline = '\n';
    let tab = '\t';
    let backslash = '\\';
    let quote = '\'';
    
    println!("Letter: {}", letter);
    println!("Emoji: {}", emoji);
    println!("Math symbol: {}", mathematical);
    println!("Chinese: {}", chinese);
}
```

## Character vs String

```rust
fn main() {
    let single_char = 'A';        // char - single quotes
    let string_slice = "A";       // &str - double quotes
    let string_literal = "Hello"; // &str - multiple characters
    
    // This would be an error:
    // let invalid = 'AB';  // Error: char can only hold one character
}
```

## Type Inference vs Explicit Typing

Rust can often **infer types** from context, but you can always specify them explicitly:

```rust
fn main() {
    // Type inference
    let age = 25;          // Inferred as i32
    let price = 19.99;     // Inferred as f64
    let is_valid = true;   // Inferred as bool
    let grade = 'A';       // Inferred as char
    
    // Explicit typing
    let age: u16 = 25;
    let price: f32 = 19.99;
    let is_valid: bool = true;
    let grade: char = 'A';
}
```

## Type Safety and Conversion

Rust is **strongly typed** and doesn't allow implicit conversions between different numeric types:

```rust
fn main() {
    let integer: i32 = 42;
    let float: f64 = 3.14;
    
    // This would cause a compilation error:
    // let result = integer + float;  // Error: mismatched types
    
    // Explicit conversion is required:
    let result = integer as f64 + float;  // OK
    let converted = float as i32;         // OK (truncates decimal)
    
    println!("Result: {}", result);
    println!("Converted: {}", converted);
}
```

## Best Practices

## **Use Appropriate Sizes**

Choose the smallest type that can hold your data to save memory, but don't micro-optimize unless necessary:

```rust
fn main() {
    let age: u8 = 25;        // Age rarely exceeds 255
    let population: u32 = 1_000_000;  // Large numbers need larger types
}
```

## **Prefer Defaults When Unsure**

Use `i32` for integers and `f64` for floating-point numbers unless you have specific requirements.

## **Use Type Annotations for Clarity**

When the type isn't obvious from context, explicit annotations improve code readability:

```rust
fn main() {
    let user_input: i32 = "42".parse().unwrap();  // Clear intent
    let percentage: f32 = calculate_percentage();   // Explicit precision
}
```

Understanding these fundamental data types is crucial for effective Rust programming. Each type has specific use cases, memory requirements, and performance characteristics that influence how you structure your programs. These scalar types form the foundation for more complex data structures you'll encounter as you continue learning Rust.
