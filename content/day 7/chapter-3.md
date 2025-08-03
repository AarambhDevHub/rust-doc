+++
title = "Borrowing Rules"
description = "Understand Rust's borrowing rules that ensure memory safety through references without a garbage collector."
date = 2025-08-02
draft = false
weight = 3
+++

# Borrowing Rules in Rust: Detailed Documentation

**Borrowing** in Rust is the mechanism that allows you to access data without taking ownership of it. Borrowing comes in two forms: **immutable references** (`&T`) and **mutable references** (`&mut T`). The Rust compiler enforces strict borrowing rules at compile time to guarantee memory safety and prevent data races.

## Core Borrowing Principles

1. **At any given time, you can have either:**
   - **Any number of immutable references**, or
   - **Exactly one mutable reference**  
2. **References must always be valid**; you cannot have a reference to data that has gone out of scope.

These rules ensure that:
- Immutable borrows (`&T`) allow concurrent reads with no risk of data races.
- Mutable borrows (`&mut T`) allow safe mutation, with exclusive access.

## Immutable References (`&T`)

- **Read-only access** to borrowed data.
- Multiple immutable references to the same data may coexist.
- Do **not** require the original value to be declared `mut`.

```rust
let data = String::from("hello");
let r1 = &data;
let r2 = &data;        // Allowed: two immutable references
println!("{} and {}", r1, r2);  // Reads via references
// data remains owned by its original variable
```

### Key Points
- References do not take ownership; they borrow.
- Data remains valid for the lifetime of all references.
- Borrowed data cannot be mutated through immutable references.

## Mutable References (`&mut T`)

- **Read–write access** to borrowed data.
- **Exclusive access**: only one mutable reference may exist at a time.
- Original variable must be declared `mut`.

```rust
let mut value = 10;
{
    let m = &mut value;   // First mutable borrow
    *m += 5;              // Modify through reference
}                         // `m` goes out of scope
let m2 = &mut value;     // Allowed once first borrow ends
*m2 *= 2;                // Further modification
```

### Key Points
- Only one mutable borrow allowed at a time.
- Cannot mix mutable and immutable borrows simultaneously.
- Ensures exclusive, safe mutation with no data races.

## Mixing Immutable and Mutable Borrows

```rust
let mut text = String::from("rust");
// Immutable borrows
let r1 = &text;
let r2 = &text;
println!("{}, {}", r1, r2);  // OK: reads only

// Mutable borrow only allowed after immutable borrows end
let r3 = &mut text;          // OK: r1 and r2 no longer used
r3.push_str("lang");
println!("{}", r3);
```

- The compiler analyzes **reference scopes**: once an immutable reference is last used, a mutable borrow may begin.
- Prevents simultaneous reads and writes.

## Preventing Dangling References

Rust prohibits references to data that goes out of scope:

```rust
fn dangle() -> &String {        // Error: returning reference to local data
    let s = String::from("hello");
    &s                           // `s` is dropped at end of function
}
```

**Solution:** Return the owned value instead of a reference.

## Borrowing with Collections

### Vectors

```rust
let mut nums = vec![1, 2, 3];
// Immutable borrow
let r = &nums;
println!("Length: {}", r.len());
// r goes out of scope here

// Mutable borrow
let m = &mut nums;
m.push(4);
println!("After mutation: {:?}", nums);
```

### Slices

```rust
let arr = [10, 20, 30, 40];
let slice = &arr[1..3];   // Immutable slice
println!("Slice: {:?}", slice);

// Mutable slice
let mut arr2 = [1, 2, 3, 4];
let slice_mut = &mut arr2[..];
slice_mut[0] = 10;
println!("Modified array: {:?}", arr2);
```

## Function Signatures and Borrowing

### Immutable Borrowing

```rust
fn read_data(data: &Vec) -> usize {
    data.len()
}
let v = vec![1, 2, 3];
let len = read_data(&v);  // v is borrowed immutably
```

### Mutable Borrowing

```rust
fn add_item(v: &mut Vec, item: String) {
    v.push(item);
}
let mut v = Vec::new();
add_item(&mut v, "hello".to_string());  // v borrowed mutably
```

## Lifetime Considerations

- References have **lifetimes** tied to the data they borrow.
- Rust’s compiler ensures that no reference outlives the data it points to.
- Non-lexical lifetimes (NLL) allow more flexible borrowing scopes when references are no longer used.

```rust
let mut s = String::from("hello");
{
    let r = &s;    // Immutable borrow begins
    println!("{}", r);
}                  // Immutable borrow ends
let mr = &mut s;  // Mutable borrow allowed
mr.push('!');
```

## Best Practices

1. **Minimize mutable borrow scope**  
   ```rust
   {
       let m = &mut data;
       // modify
   }  // mutable borrow ends here
   // now safe to create other borrows
   ```

2. **Prefer immutable borrows** when mutation is not needed.

3. **Use references in function parameters** to avoid unnecessary moves and clones.

4. **Leverage Rust’s error messages**: they pinpoint where borrowing rules are violated.

## Summary

Rust’s borrowing rules enforce safe concurrency and memory management without runtime overhead:

- **Immutable references** (`&T`) allow shared, read-only access.
- **Mutable references** (`&mut T`) allow exclusive, read-write access.
- **Exclusivity rule**: at most one mutable borrow or many immutable borrows simultaneously.
- **Lifetime guarantees** prevent dangling references.

These rules are enforced at compile time, eliminating data races and ensuring memory safety while allowing expressive and efficient code.