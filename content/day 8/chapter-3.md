+++
title = "Slice Syntax and Usage"
description = "Understand the syntax and practical usage of slices in Rust for accessing parts of arrays, vectors, and strings safely."
date = 2025-08-03
draft = false
weight = 3
+++

# Slice Syntax and Usage in Rust: Comprehensive Documentation

Slices in Rust provide **views** into collections—arrays, vectors, and strings—without taking ownership of the data. They are fundamental for writing idiomatic, safe, and efficient Rust code. This documentation covers syntax and common usage patterns in detail.

## Slice Syntax

Given a collection `col`, you create a slice using range notation inside brackets:

```rust
&col[start..end]    // Includes start, excludes end
&col[..end]         // From 0 to end
&col[start..]       // From start to collection’s end
&col[..]            // Entire collection
```

- `start` and `end` are `usize` indices.
- `start..end` is half-open: includes `start`, excludes `end`.
- Omitting `start` or `end` defaults to beginning or end.
- The type of slice is `&[T]` for arrays and vectors, or `&str` for strings.

## Array and Vector Slices (`&[T]`, `&mut [T]`)

### Immutable Slice

```rust
let arr = [1, 2, 3, 4, 5];

// Slice of array
let s: &[i32] = &arr[1..4];  // [2, 3, 4]

assert_eq!(s, &[2, 3, 4]);
println!("s = {:?}", s);

// Full slice
let full = &arr[..];         // [1, 2, 3, 4, 5]
```

### Mutable Slice

```rust
let mut arr = [10, 20, 30, 40];
let s: &mut [i32] = &mut arr[1..3];  // [20, 30]

s[0] = 25;
s[1] = 35;

// Original array reflects changes
assert_eq!(arr, [10, 25, 35, 40]);
```

### Vector Slices

```rust
let mut v = vec![100, 200, 300, 400];
let s = &v[2..];          // &[300, 400]
let ms = &mut v[..2];     // &mut [100, 200]

ms[1] = 250;
assert_eq!(v, [100, 250, 300, 400]);
```

### Common Methods on Slices

- `.len()`, `.is_empty()`
- `.first()`, `.last()`
- `.get(index)` → `Option`
- `.iter()`, `.iter_mut()`
- `.split_at(mid)` → two slices
- `.split_at_mut(mid)` → two mutable slices

## String Slices (`&str`)

### Creating String Slices

```rust
let s = String::from("Hello, Rust!");

// Full slice
let full: &str = &s[..];           // "Hello, Rust!"

// Sub-slices
let hello = &s[0..5];               // "Hello"
let rust = &s[7..11];               // "Rust"

// Literal slices
let lit: &str = "Static string slice";
```

### String Slice Methods

- `.len()`: byte length
- `.is_empty()`
- `.contains(&str)`
- `.starts_with(&str)`, `.ends_with(&str)`
- `.find(&str)` → `Option`
- `.split(pattern)`, `.split_whitespace()`, `.lines()`
- `.chars()`, `.bytes()`, `.char_indices()`

#### Example

```rust
let text = "Data;More;Values";
for part in text.split(';') {
    println!("Part: {}", part);
}
```

### UTF-8 and Boundary Safety

Rust enforces slicing only on **character boundaries**:

```rust
let emoji = "🦀 crab";
// emoji.len() = 8 bytes

let crab = &emoji[4..8];             // Valid: “crab”
let invalid = &emoji[1..4];          // Panic: not on char boundary
```

Use methods like `.char_indices()` or `.get()` for safe slicing:

```rust
if let Some(slice) = emoji.get(4..8) {
    println!("Safe slice: {}", slice);
}
```

## Pattern Matching with Slices

### Matching on Slices

```rust
fn classify(nums: &[i32]) -> &str {
    match nums {
        [] => "empty",
        [single] => "one element",
        [first, rest @ ..] => {
            println!("First: {}, others: {:?}", first, rest);
            "multiple elements"
        }
    }
}

fn main() {
    println!("{}", classify(&[]));             // "empty"
    println!("{}", classify(&[5]));            // "one element"
    println!("{}", classify(&[1, 2, 3]));      // "multiple elements"
}
```

## Lifetimes and Slices

Slices borrow data; their lifetimes must not outlive the original data:

```rust
fn get_slice(arr: &'a [i32]) -> Option {
    if arr.len() >= 2 {
        Some(&arr[0..2])
    } else {
        None
    }
}

let data = vec![10, 20, 30];
if let Some(two) = get_slice(&data) {
    println!("Slice: {:?}", two);
}
// `data` remains valid
```

## Best Practices

- **Use slices (`&[T]`)** when you need to operate on part of a collection without ownership transfer.
- **Prefer `&str`** over `String` in function parameters for flexibility.
- **Check boundaries** with `.get()`, `.get_mut()` to avoid panics.
- **Split large mutable slices** with `.split_at_mut()` for safe parallel mutation.
- **Respect UTF-8 boundaries** when slicing strings; use safe methods or `.get()`.

Slices are crucial for writing ergonomic, safe, and efficient Rust code. Mastering slice syntax and usage unlocks powerful patterns for data processing, parsing, and transformation throughout your Rust projects.
