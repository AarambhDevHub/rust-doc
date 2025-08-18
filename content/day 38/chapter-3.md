+++
title = "Generating Test Data"
description = "Techniques for generating random and structured test data for property-based testing."
date = "2025-08-13"
weight = 3
+++

# Generating Test Data in Rust: A Guide to Property-Based Testing

Understanding how to generate test data in Rust is like learning to **design a rigorous quality control system for a professional restaurant kitchen** – instead of just tasting one specific dish to see if it's good (example-based testing), you create a system that can automatically check *every possible variation* of a recipe to ensure it always meets certain standards (property-based testing). Just as a master chef defines rules like "a soup must never be served cold" and then tests many batches to verify this, a skilled Rust programmer defines properties about their code and uses test data generation to verify these properties against thousands of random inputs.

## The Professional Restaurant Quality Control Analogy 🍽️

### Imagine You're the Head of Quality Control for a Global Restaurant Chain

**The Problem with Traditional, Manual Testing (Example-Based Testing):**

```rust
// ❌ Manual "spot check" approach - like tasting just one bowl of soup
#[test]
fn test_specific_soup_order() {
    let soup = prepare_soup("tomato", 2); // A single, specific input
    assert!(soup.temperature > 70.0); // Check a single output
}
// This test tells you nothing about the "chicken" soup or an order for 10 people.
```

**The Power of Property-Based Testing (Automated Quality Control):**

```rust
// ✅ Automated quality control - defining a rule and testing it against hundreds of random orders
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_any_soup_is_always_hot(soup_type in "\\PC*", portion_size in 1..=10) {
        // proptest generates random soup_type strings and portion_size numbers
        let soup = prepare_soup(&soup_type, portion_size);
        prop_assert!(soup.temperature > 70.0); // The "property" that must always be true
    }
}
```

**Why Generating Test Data for Property-Based Testing Is a Game-Changer:**

- 🐛 **Finds Edge Cases**: Automatically discovers bugs in scenarios you never thought to test (e.g., empty strings, very large numbers, unusual characters).[^1]
- 🛡️ **Increases Confidence**: A single property test can be worth hundreds of example-based tests, giving you much higher confidence in your code's correctness.[^2]
- 💡 **Forces Better Design**: Writing properties makes you think about the essential invariants and contracts of your code, leading to a clearer design.
- 🔄 **Automatic and Efficient**: The computer does the hard work of creating test cases, freeing you up to focus on defining what "correct" means.
- **Minimal Failing Cases**: When a property test fails, libraries like `proptest` and `quickcheck` will "shrink" the failing input to the smallest possible example, making debugging much easier.[^1]


## Core Concepts: Example vs. Property Testing

* **Example-Based Testing**: You, the developer, provide a *specific input* and assert a *specific output*.
    * `assert_eq!(add(2, 2), 4);`
* **Property-Based Testing**: You define a *property* (a rule) that should be true for *all valid inputs*, and the testing library generates hundreds of random inputs to try and prove your property wrong.[^3]
    * Property: `a + b == b + a` (addition is commutative). The library will test this with many random `a` and `b`.


## Generating Simple Random Data with the `rand` Crate

Before diving into property-testing libraries, it's essential to understand the foundation: the `rand` crate. It's the standard way to generate random values in Rust.[^4]

**Add to `Cargo.toml`:**

```toml
[dependencies]
rand = "0.8"
```

**Example Usage:**

```rust
use rand::Rng;

fn generate_simple_random_data() {
    // 1. Get a thread-local random number generator
    let mut rng = rand::thread_rng();

    // 2. Generate different types of random data
    let random_u32: u32 = rng.gen();
    let random_f64: f64 = rng.gen();
    let random_bool: bool = rng.gen();

    println!("Random u32: {}", random_u32);
    println!("Random f64: {}", random_f64);
    println!("Random bool: {}", random_bool);

    // 3. Generate a number within a specific range
    let n: u32 = rng.gen_range(1..=100); // A number between 1 and 100 (inclusive)
    println!("Random number between 1 and 100: {}", n);
}
```

This is great for simple needs, but property-based testing libraries build on this to offer much more power and structure.

## Property-Based Testing with `proptest`

`proptest` is a powerful and popular property-testing framework for Rust, inspired by Python's `Hypothesis`.

**Add to `Cargo.toml`:**

```toml
[dev-dependencies]
proptest = "1.2.0"
```


### 1. Writing a Simple Property Test

Let's test a simple property: for any two strings `a` and `b`, the length of `a` concatenated with `b` should be the sum of their individual lengths.

**Example (`src/lib.rs`):**

```rust
pub fn concatenate_strings(s1: &str, s2: &str) -> String {
    format!("{}{}", s1, s2)
}

#[cfg(test)]
mod tests {
    use super::*;
    use proptest::prelude::*;

    proptest! {
        #[test]
        fn test_concatenation_length_property(s1 in "\\PC*", s2 in "\\PC*") {
            // proptest generates two random strings for s1 and s2.
            // "\\PC*" is a strategy for generating any string of unicode characters.

            let result = concatenate_strings(&s1, &s2);

            // Assert the property
            prop_assert_eq!(result.len(), s1.len() + s2.len());
        }
    }
}
```

When you run `cargo test`, `proptest` will execute this test many times (by default, 256 times) with different random strings. If it finds a single case where `result.len()` is not equal to `s1.len() + s2.len()`, the test will fail, and `proptest` will show you the exact inputs that caused the failure.

### 2. Generating Structured Test Data

The real power of property-based testing comes from generating random instances of your own custom `struct`s. `proptest` can do this automatically if you derive `Arbitrary`.

**Example: Testing a Restaurant Order System**

Let's define an `Order` and test a function that calculates its total price.

```rust
// Add `proptest-derive` to your dev-dependencies in Cargo.toml
// [dev-dependencies]
// proptest = "1.2.0"
// proptest-derive = "0.4.0"

use proptest_derive::Arbitrary;

#[derive(Debug, Clone, Arbitrary)]
pub struct Order {
    // proptest will generate random values for these fields.
    // We can add constraints if needed.
    #[proptest(strategy = "1..1000u32")] // Constrain the id
    pub order_id: u32,
    #[proptest(strategy = "1..=10u32")] // Constrain the quantity
    pub quantity: u32,
    #[proptest(strategy = "1.0..=100.0f64")] // Constrain the price
    pub price_per_item: f64,
}

pub fn calculate_order_total(order: &Order) -> f64 {
    order.quantity as f64 * order.price_per_item
}

#[cfg(test)]
mod tests {
    use super::*;
    use proptest::prelude::*;

    proptest! {
        #[test]
        fn test_order_total_is_always_positive(order: Order) {
            // proptest automatically generates a valid `Order` instance
            // based on the strategies we defined in the struct.

            let total = calculate_order_total(&order);

            // Property: The total price should never be negative.
            prop_assert!(total >= 0.0, "Order total should not be negative");
        }

        #[test]
        fn test_order_total_calculation(
            // We can also define strategies right in the test function
            quantity in 1..=50u32,
            price in 1.0..=200.0f64
        ) {
            let order = Order { order_id: 1, quantity, price_per_item: price };
            let total = calculate_order_total(&order);

            // Property: The calculation must be correct.
            prop_assert_eq!(total, quantity as f64 * price);
        }
    }
}
```


### 3. Controlling Generation Strategies

`proptest` provides powerful "strategies" to control how data is generated, ensuring it's valid for your tests.[^2]

- **Ranges**: `1..100`, `0.0..=1.0`
- **Regular Expressions for Strings**: `"\\PC*"` (any string), `"[a-z]{1,10}"` (1 to 10 lowercase letters).
- **Collections**: `prop::collection::vec(any::<u8>(), 0..10)` generates a `Vec<u8>` of 0 to 10 elements.
- **Filtering and Mapping**: You can `prop_filter` or `prop_map` existing strategies to create more complex ones.

**Example: Generating a non-empty list of positive numbers.**

```rust
// ... inside a proptest! macro ...
#[test]
fn test_sum_of_positives_is_positive(
    // Strategy: a vector of u32s, with 1 to 100 elements.
    // The `prop::collection::vec` strategy takes another strategy for the elements
    // and a range for the size. `any::<u32>()` generates any u32.
    nums in prop::collection::vec(1..=1_000_000u32, 1..=100)
) {
    let sum: u32 = nums.iter().sum();
    prop_assert!(sum > 0, "Sum of positive numbers must be positive");
}
```


## Generating Realistic "Fake" Data with `fake`

Sometimes, you need test data that isn't just random but looks *realistic* (e.g., names, email addresses, company names). This is where the `fake` crate shines.

**Add to `Cargo.toml`:**

```toml
[dev-dependencies]
fake = "2.9"
```

**Example: Generating a Fake Customer Profile**

```rust
use fake::{Fake, Faker};
use fake::faker::name::en::Name;
use fake::faker::internet::en::SafeEmail;
use fake::faker::phone_number::en::PhoneNumber;

#[derive(Debug)]
pub struct CustomerProfile {
    name: String,
    email: String,
    phone: String,
}

fn generate_fake_customer() -> CustomerProfile {
    CustomerProfile {
        name: Name().fake(),
        email: SafeEmail().fake(),
        phone: PhoneNumber().fake(),
    }
}

#[test]
fn test_generate_realistic_customer() {
    let customer = generate_fake_customer();
    println!("Generated fake customer: {:?}", customer);

    assert!(!customer.name.is_empty());
    assert!(customer.email.contains('@'));
}
```


## Summary of Best Practices

| **Technique** | **When to Use** | **Key Crates** | **Why It's Good** |
| :-- | :-- | :-- | :-- |
| **Simple Random Generation** | Basic randomness needs, simple tests, or as a building block for other data. | `rand` | Simple, lightweight, and the foundation for randomness in Rust . |
| **Property-Based Testing** | To test logical properties, find edge cases, and increase confidence in algorithms. | `proptest`, `quickcheck` | Automatically finds bugs and provides minimal failing examples, leading to robust code [^1][^2]. |
| **Realistic "Fake" Data** | Integration tests, UI tests, or when you need semantically meaningful data. | `fake` | Creates data that looks real, making tests more intuitive and useful for demonstrations [^5]. |
| **Structured Data with Constraints** | Testing complex `struct`s and business logic with specific data requirements. | `proptest` (with strategies) | Allows you to generate random but *valid* data that adheres to your domain rules. |

## Final Recommendations for Beginners

1. **Start with Property-Based Testing Early**: For any function that has clear logical rules (e.g., sorting, encoding/decoding, calculations), try writing a property test instead of an example-based one. It will force you to think more deeply about what your code should *always* do.
2. **Let the Library Do the Work**: Trust `proptest` or `quickcheck` to generate inputs. Your job is to define the *properties* that must hold true.
3. **Use Strategies to Guide Generation**: As your data models become more complex, learn to use strategies to generate valid data. This is where the real power of these libraries lies.
4. **Use `fake` for Realistic Scenarios**: When you're testing higher-level features, like user registration or report generation, use `fake` to make your tests more realistic and your test data more readable.

By generating test data effectively, you move from testing a few known paths to stress-testing your code against a vast and varied landscape of inputs, building incredibly robust and reliable software in the process.

1. https://rustprojectprimer.com/testing/property.html
2. https://ivanyu.me/blog/2024/09/22/proptest-property-testing-in-rust/
3. https://blog.lambdaclass.com/what-is-property-based-testing/
4. https://rust-lang-nursery.github.io/rust-cookbook/algorithms/randomness.html
5. https://qxf2.com/blog/data-generation-in-rust-using-fake-crate/
6. https://lpalmieri.com/posts/an-introduction-to-property-based-testing-in-rust/
7. https://github.com/BurntSushi/quickcheck
8. https://github.com/j5ik2o/prop-check-rs
9. https://www.reddit.com/r/rust/comments/yi3y33/propertybased_testing_in_rust_with_arbitrary/
10. https://docs.rs/rand
11. https://rtpg.co/2024/02/02/property-testing-with-imperative-rust/
