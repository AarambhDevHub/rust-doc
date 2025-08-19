---
title: "Advanced Unsafe Operations - Union Types"
description: "Understand union types in Rust, their use cases, and how they interact with unsafe code."
date: 2025-10-14
weight: 3
---

# Advanced Unsafe Operations: Understanding Union Types in Rust

To understand `union` types in Rust, you must first distinguish them from `enum`s. Think of it like this: an `enum` is a **well-organized, labeled pantry shelf**, while a `union` is a **single, unlabeled, multi-purpose container**. Both can hold different types of ingredients, but they do so in fundamentally different ways, with vastly different safety implications.

## The Pantry Analogy: Enum vs. Union 🍽️

### `enum`: The Labeled Pantry Shelf (Safe and Descriptive)

An `enum` in Rust is a **tagged union**. It's like a pantry shelf with clearly labeled sections for "Flour," "Sugar," and "Salt."

```rust
enum PantryItem {
    Flour(f32), // in kilograms
    Sugar(f32),
    Salt(f32),
}
```

- **It knows what it is**: Each `PantryItem` value stores a "tag" that tells you *which* variant it is (`Flour`, `Sugar`, or `Salt`).
- **It has dedicated space**: The value (e.g., the amount of flour) is stored alongside its tag.
- **It's safe**: You can use a `match` statement to safely check the tag and access the correct data. The compiler guarantees you can't mistake flour for sugar.

This is the standard, safe, and idiomatic way to represent a value that can be one of several types in Rust.[^8]

### `union`: The Single, Unlabeled Container (Unsafe and Efficient)

A `union` is a raw, low-level construct. It's like a single container on your counter. You can put flour in it. Or, you can empty it and put sugar in it. But the container itself has no label. It's just a container.

```rust
#[repr(C)] // Common representation for FFI
union MultiPurposeContainer {
    flour_kg: f32,
    sugar_kg: f32,
    salt_kg: f32,
}
```

- **It doesn't know what it is**: The `union` has no tag. It doesn't keep track of which field was last written to. That responsibility is entirely on you, the programmer.[^3]
- **It shares memory**: All fields of a `union` occupy the **same memory location**. The size of the `union` is the size of its largest field. Writing to `sugar_kg` will overwrite the bits that were previously interpreted as `flour_kg`.[^1][^6]
- **It's inherently `unsafe`**: Because the compiler has no idea what's currently in the container, accessing *any* field is an `unsafe` operation. You are promising the compiler, "I know for a fact that this container currently holds sugar, so I will access the `sugar_kg` field." If you are wrong, you will get garbage data or trigger Undefined Behavior.[^3]


## How to Use a `union`

Accessing a `union`'s fields requires an `unsafe` block, because you are taking responsibility for interpreting the raw bytes correctly.

**Example: A Simple `union` in Action**

```rust
union IntOrFloat {
    i: i32,
    f: f32,
}

fn main() {
    // Create a union instance. We must initialize exactly one field.
    let mut value = IntOrFloat { i: 42 };

    // To access any field, we need an `unsafe` block.
    unsafe {
        // We know we just stored an integer, so this is safe.
        println!("Value as integer: {}", value.i);
    }

    // Now, let's change the value to a float.
    // This overwrites the memory that held the integer.
    value.f = 3.14;

    unsafe {
        // We know we just stored a float, so this is safe.
        println!("Value as float: {}", value.f);

        // --- DANGER! ---
        // What happens if we access the WRONG field?
        // We are telling the compiler to interpret the bits of the float 3.14
        // as if they were an integer. This is Undefined Behavior.
        println!("Value interpreted as integer (garbage): {}", value.i);
    }
}
```


## Why and When to Use `union`s

Given their danger, `union`s are a highly specialized tool. You should almost always prefer `enum`s. However, there are a few key scenarios where `union`s are the right choice.

### Use Case 1: Interfacing with C Libraries (FFI)

This is the most common and legitimate use case. C programs frequently use unions (often within a `struct` that contains a tag) to represent data that can take multiple forms. To create a compatible data structure in Rust for FFI, you must use a `union`.

**Example: A C-style Tagged Union**
Imagine a C library that uses this structure:

```c
// C Code
typedef enum {
    IS_INT,
    IS_FLOAT
} ValueTag;

typedef struct {
    ValueTag tag;
    union {
        int i;
        float f;
    } data;
} CValue;
```

To use this in Rust, you would replicate the structure precisely:

```rust
#[repr(u32)]
enum Tag {
    Int,
    Float,
}

#[repr(C)]
union Data {
    i: i32,
    f: f32,
}

#[repr(C)]
struct CValue {
    tag: Tag,
    data: Data,
}

fn process_c_value(value: CValue) {
    unsafe {
        match value.tag {
            Tag::Int => {
                // Because we checked the tag, we can now safely access the `i` field.
                println!("C value is an integer: {}", value.data.i);
            }
            Tag::Float => {
                // Similarly, it's safe to access `f` here.
                println!("C value is a float: {}", value.data.f);
            }
        }
    }
}
```

This pattern, where you use an `enum` as a tag alongside a `union`, is the safest way to use unions, as you are manually keeping track of which field is active.

### Use Case 2: Extreme Performance-Critical Code or Memory Optimization

In some rare cases, you might use a `union` to save memory. An `enum` always has to store its tag, which adds a small amount of overhead. If you have millions of instances of a type and you can track the active variant through other means (e.g., the state of your program), a `union` can be slightly more memory-efficient.

**Example: A "Value" type in a programming language interpreter**
An interpreter might use a type that can be either a number or a pointer to an object. By using a `union`, it can avoid the overhead of an `enum` tag for every single value. This is a very advanced use case and should only be considered after profiling has shown it to be a significant bottleneck.

## Restrictions and Important Rules

- **Fields Cannot Be Dropped**: By default, types you put in a `union` must be `Copy`. This is because the compiler doesn't know which field is active, so it cannot know which field's `drop` implementation to call. If you try to put a `String` or a `Vec` in a union, the compiler will complain.
- **Using `ManuallyDrop<T>`**: To put a non-`Copy` type like `String` in a union, you must wrap it in `std::mem::ManuallyDrop<T>`. This tells the compiler, "Don't automatically drop this." It then becomes **your responsibility** to manually call `std::ptr::drop_in_place` on the correct field when the union is no longer needed. This is extremely easy to get wrong and can lead to memory leaks.[^2]

```rust
use std::mem::ManuallyDrop;
use std::ptr;

union StringOrInt {
    s: ManuallyDrop<String>,
    i: i32,
}

// This is complex and dangerous!
fn use_string_union() {
    let mut u = StringOrInt { s: ManuallyDrop::new("Hello".to_string()) };

    // ... use the union ...

    // We MUST manually drop the string when we are done with it.
    unsafe {
        ManuallyDrop::drop(&mut u.s);
    }
}
```


## Summary: `enum` vs. `union`

| **Feature** | **`enum` (Safe Tagged Union)** | **`union` (Unsafe Untagged Union)** |
| :-- | :-- | :-- |
| **Safety** | **Safe**. The compiler guarantees correct access. | **`unsafe`**. The programmer is responsible for correct access. |
| **Memory** | Stores a tag plus the data for the largest variant. | Stores only the data. All fields share the same memory. |
| **Tracking** | **Self-describing**. The `enum` knows its own variant. | **Not self-describing**. You must track the active field externally. |
| **Drop Behavior** | Automatic and safe. | Fields must be `Copy` or wrapped in `ManuallyDrop`. |
| **Primary Use Case** | The standard, idiomatic way to represent "one of" types. | FFI with C libraries; niche performance optimizations. |

**Final Guideline for Beginners**: Always use an `enum`. Only consider using a `union` if you are interfacing directly with a C library that requires it. A `union` is a tool of last resort for when you need to manually manage memory layout at a very low level, and it requires extreme care to use correctly.

1. https://doc.rust-lang.org/reference/items/unions.html
2. https://stackoverflow.com/questions/64498340/how-can-i-use-structs-containing-string-in-a-union
3. https://rust-dd.com/post/hack-rust-s-type-system-with-union-types
4. https://www.reddit.com/r/rust/comments/u5a5e9/how_do_you_put_unions_in_structs_and_vice_versa/
5. https://blog.shun-k.dev/posts/rust-union-types/
6. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/reference/items/unions.html
7. https://internals.rust-lang.org/t/pre-rfc-sum-union-types/18321
8. https://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions
9. https://tonyarcieri.com/a-quick-tour-of-rusts-type-system-part-1-sum-types-a-k-a-tagged-unions
