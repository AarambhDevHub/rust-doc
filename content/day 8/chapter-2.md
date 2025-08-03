+++
title = "Array Slices"
description = "Learn how array slices ([T]) work in Rust, enabling safe and flexible views into arrays and vectors."
date = 2025-08-03
draft = false
weight = 2
+++

# Array Slices (`&[T]`) in Rust: Comprehensive Documentation

Array slices in Rust are views into contiguous segments of an array or other sequence without copying the data. A slice is a **fat pointer** (pointer + length) that provides **shared, read-only access** to a portion of an array, vector, or other collection of type `T`.

## Definition and Syntax

A slice’s type is `&[T]` (immutable) or `&mut [T]` (mutable). Slices are created using range syntax:

```rust
let arr = [1, 2, 3, 4, 5];
let slice: &[i32] = &arr[start_index..end_index];
```

- **`start_index`** (inclusive)
- **`end_index`** (exclusive)

You may omit one or both indices:
- `&arr[..n]` → from 0 to `n`
- `&arr[n..]` → from `n` to end
- `&arr[..]` → entire array

## Memory Layout

Slices consist of:
- A **pointer** to the first element
- A **length** of elements

```
  Stack:                   Heap/Stack Data:
┌───────────────────┐      ┌────────────────────────┐
│ slice.ptr  ──────┼─────▶│ arr[0]  arr[1] ...    │
│ slice.len  = 3   │      └────────────────────────┘
└───────────────────┘
```

For `&arr[1..4]`, `slice.ptr` points at `arr[1]` and `slice.len = 3`.

## Immutable Slice Operations

### Basic Access

```rust
let arr = [10, 20, 30, 40, 50];
let s = &arr[1..4];  // [20, 30, 40]
println!("Slice: {:?}", s);
println!("First: {}", s[0]);            // 20
if let Some(val) = s.get(2) { println!("{}", val); }  // 40
```

- `s.len()`: number of elements
- `s.is_empty()`: checks if length=0
- `s.iter()`: iterator over `&T`

### Iteration

```rust
for (i, &v) in s.iter().enumerate() {
    println!("s[{}] = {}", i, v);
}
```

### Slicing Slices

```rust
let arr = [1,2,3,4,5];
let s1 = &arr[..];    // [1,2,3,4,5]
let s2 = &arr[2..];   // [3,4,5]
let s3 = &arr[1..4];  // [2,3,4]
```

## Mutable Slices (`&mut [T]`)

Mutable slices grant exclusive, read-write access:

```rust
let mut arr = [1,2,3,4,5];
let s = &mut arr[1..4];  // &mut [2,3,4]
s[0] = 20;
s.reverse();            // Now [4,3,20]
println!("arr: {:?}", arr);  // [1,4,3,20,5]
```

- Only **one** `&mut [T]` may exist at a time
- Cannot coexist with any immutable `&[T]` to the same data

## Common Patterns

### Splitting Slices

```rust
let mut arr = [1,2,3,4,5,6];
let (left, right) = arr.split_at_mut(3);
// left: &mut [1,2,3], right: &mut [4,5,6]
left.reverse();
right.reverse();
println!("{:?}", arr);  // [3,2,1,6,5,4]
```

### Functions with Slices

```rust
fn sum_slice(s: &[i32]) -> i32 { s.iter().sum() }
fn increment_slice(s: &mut [i32]) { for x in s { *x += 1; } }

let mut data = [10,20,30];
let total = sum_slice(&data);
increment_slice(&mut data);
```

### Lifetimes and Slices

Slices borrow data; their lifetimes must not outlive the original array:

```rust
fn get_slice(arr: &'a [i32]) -> &'a [i32] { &arr[1..] }

let arr = vec![1,2,3];
let s1 = get_slice(&arr);  // s1 valid as long as `arr` is alive
```

## Tips and Pitfalls

- **Bounds checking**: `s[i]` panics if `i >= s.len()`; use `s.get(i)` to get `Option`.
- **UTF-8 caution**: For `[u8]` slices used as strings, avoid cutting multi-byte characters; use `str::from_utf8` or safe string methods.
- **Empty slices**: `&arr[2..2]` yields an empty slice without panic.

## Summary

- **Array slices (`&[T]`)** provide efficient, non-owning views into contiguous data.
- **Mutable slices (`&mut [T]`)** allow exclusive mutation with the same safety guarantees.
- Slices are central to idiomatic Rust for functions operating on parts of collections.
- Lifetimes ensure slices never outlive the data they reference, preventing dangling pointers.

1. https://www.w3schools.com/rust/rust_if_else.php
2. https://notes.kodekloud.com/docs/Rust-Programming/Ownership/The-Slice-Type
3. https://www.programiz.com/rust/slice
4. https://www.codecademy.com/resources/docs/rust/slices
5. https://doc.rust-lang.org/rust-by-example/primitives/array.html
6. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/rust-by-example/primitives/array.html
7. https://doc.rust-lang.org/std/primitive.slice.html
8. https://doc.rust-lang.org/std/primitive.array.html
9. https://www.geeksforgeeks.org/rust/rust-slices/
10. https://doc.rust-lang.org/book/ch04-03-slices.html
11. https://docs.rs/slice-of-array
