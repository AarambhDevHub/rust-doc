+++
title = "Practical Applications"
description = "Explore real-world scenarios where Rust's ownership, borrowing, and slicing concepts enhance performance and safety."
date = 2025-08-03
draft = false
weight = 4
+++

# Rust Memory and Ownership: Comprehensive Practical Documentation

This guide covers essential Rust concepts and their **practical applications**, organized by topic and day. Each section provides detailed explanations, examples, and best practices for real-world usage.

## 1. Ownership

### What Is Ownership?
- **Each value** has a single **owner** (a variable).
- **Only one** owner at a time; assignments and function calls **move** ownership.
- When the owner **goes out of scope**, Rust calls `drop` to free resources.

```rust
fn main() {
    let s = String::from("hello"); // s owns the data
    takes_ownership(s);             // s moved, no longer valid
    // println!("{}", s);           // Error!

    let x = 5;                      // x is Copy, not moved
    makes_copy(x);                  // x still valid
    println!("{}", x);              // 5
}

fn takes_ownership(s: String) { /* ... */ }
fn makes_copy(n: i32) { /* ... */ }
```

## 2. Stack vs. Heap Memory

| Feature        | Stack                         | Heap                          |
|----------------|-------------------------------|-------------------------------|
| Allocation     | Fast, LIFO                    | Slower, dynamic               |
| Size           | Limited, compile-time         | Large, runtime                |
| Data Types     | Fixed-size (primitives, arrays of known size) | Dynamic-size (`String`, `Vec`) |
| Cleanup        | Automatic when scope ends     | Ownership-based `drop`        |

```rust
fn main() {
    let x = 42;                // stack
    let s = String::from("hi"); // heap
} // x and s dropped; s’s heap freed
```

## 3. Ownership Rules

1. **Each value** has an **owner**.
2. **One owner** at a time.
3. When the owner **goes out of scope**, the value is **dropped**.

Violations raise compile-time errors (e.g., use-after-move, double borrow).

## 4. Move Semantics

- Assigning a non-`Copy` type **moves** ownership, not data.
- **No runtime cost**; only a pointer and metadata copy.

```rust
let v1 = vec![1, 2, 3];
let v2 = v1; // move
// println!("{:?}", v1); // Error

// Cloning for deep copy
let v3 = v2.clone(); // explicit heap copy
```

## 5. `Copy` Trait Basics

- Types implementing `Copy` are **bitwise-copied** on assignment.
- Simple, stack-only types (`i32`, `bool`, char, tuples of Copy types).

```rust
let a = 5;  // i32 Copy
let b = a;  // a still valid
```

Custom types can derive `Copy` iff **all fields** implement `Copy`.

## Day 7 (Tuesday): References & Borrowing

### 6. Immutable References (`&T`)
- **Read-only** borrows can coexist.

```rust
let s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);
```

### 7. Mutable References (`&mut T`)
- **Exclusive** read-write borrow; only one at a time.
- Original variable **must be mutable**.

```rust
let mut data = vec![1, 2, 3];
{
    let m = &mut data;
    m.push(4);
} // m ends
let m2 = &mut data;
m2.push(5);
```

### 8. Borrowing Rules
- **One** `&mut` or **many** `&T` simultaneously.
- Cannot mix `&` and `&mut` until earlier borrows end.
- Prevents data races and ensures safe mutation.

### 9. Reference Lifetime Basics
- References have **lifetimes** tied to their data’s scope.
- **No dangling** references: compiler enforces that a reference never outlives its owner.

```rust
fn longest(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

### 10. Common Borrowing Errors & Solutions

| Error                                      | Cause                                      | Solution                           |
|--------------------------------------------|--------------------------------------------|------------------------------------|
| Cannot borrow as mutable while borrowed    | `&` active, then `&mut` attempted          | End the immutable borrow first     |
| Cannot return reference to local data      | Returning `&T` to data dropped after func  | Return `String` or accept `&mut`   |
| Multiple `&mut` in scope                   | Two mutable borrows                        | Scope borrows, split data with `split_at_mut` |

## Day 8 (Wednesday): Slices

### 11. String Slices (`&str`)
- Immutable view into UTF-8 data.
- Zero-cost: pointer + length.
- Must slice on **character boundaries**.

```rust
let text = String::from("Hello, world!");
let hello = &text[0..5]; // "Hello"
let whole = &text[..];
let literal: &str = "static str";
```

### 12. Array Slices (`&[T]`, `&mut [T]`)
- View into an array or vector.

```rust
let arr = [1,2,3,4];
let s = &arr[1..3];         // &[2,3]
let ms = &mut arr[..2];     // &mut [1,2]
```

- Common methods: `.len()`, `.is_empty()`, `.get()`, `.iter()`, `.split_at()`

### 13. Slice Syntax & Usage
- Range syntax: `start..end`, `..end`, `start..`, `..`
- Pattern matching on slices:

```rust
fn classify(nums: &[i32]) -> &str {
    match nums {
        [] => "empty",
        [x] => "one",
        [x, y, ..] => "many",
    }
}
```

### 14. Practical Applications
- **Parsing**: splitting CSV lines, text processing without allocation.
- **Algorithms**: in-place sorting/mutation via mutable slices.
- **APIs**: functions accepting `&[T]` or `&str` for flexible data handling.
- **Data analysis**: summing, filtering, transforming slices efficiently.

```rust
fn sum_slice(s: &[i32]) -> i32 { s.iter().sum() }
fn find_max(s: &[i32]) -> Option { s.iter().cloned().max() }
fn normalize(v: &mut [f64]) {
    let max = v.iter().cloned().fold(0./0., f64::max);
    for x in v { *x /= max; }
}
```

This documentation equips you with detailed knowledge of Rust’s memory, ownership, borrowing, and slicing mechanisms—foundation for safe, high-performance Rust programming.
