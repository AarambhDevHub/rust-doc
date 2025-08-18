+++
title = "Shrinking and Debugging"
description = "Learn how shrinking helps simplify failing test cases and aids debugging in property-based testing."
date = "2025-08-13"
weight = 5
+++

# Shrinking and Debugging in Property-Based Testing: Comprehensive Documentation for Beginners

Understanding shrinking in property-based testing is like learning to **pinpoint the exact cause of a kitchen disaster in a professional restaurant** – when a complex dish fails, you don't just report "the 15-ingredient seafood paella is bad." Instead, you systematically simplify the recipe, removing ingredients one by one, until you discover that the problem was just a single spoiled mussel. This process of simplifying a failing test case to its smallest, most understandable form is what shrinking is all about, and it's a crucial part of making property-based testing an effective debugging tool.

## The Professional Kitchen Recipe Debugging Analogy 🍽️

### Imagine You're Debugging a Failed Recipe in a High-Stakes Kitchen

**The Problem without Shrinking (A Complex Failure Report):**

```rust
// ❌ Failure without shrinking - like being told a complex dish is just "bad"
fn test_complex_dish_fails() {
    // Property-based test generates a complex, random set of ingredients
    let failing_ingredients = vec![
        "Saffron", "Rice", "Chicken", "Chorizo", "Shrimp",
        "Mussels", "Clams", "Peas", "Red Peppers", "Onions",
        "Garlic", "Tomatoes", "Olive Oil", "Paprika", "White Wine"
    ];

    // The test fails with this large, complex input.
    // Debugging this is a nightmare. Where do you even start?
}
```

**The Power of Shrinking (A Simple, Actionable Failure Report):**

```rust
// ✅ Failure with shrinking - systematically simplifying the recipe to find the root cause
fn test_dish_with_shrinking() {
    // The test initially fails with the same complex set of 15 ingredients.
    // The shrinking process then begins automatically:
    // 1. "Does it still fail with only 10 ingredients?" -> Yes.
    // 2. "Does it still fail with only 5 ingredients?" -> Yes.
    // 3. "Does it still fail with just 'Mussels'?" -> Yes!

    // The final, shrunken failure report is simple and actionable:
    let minimal_failing_ingredient = vec!["Spoiled Mussels"];
    // Now you know exactly what to investigate.
}
```

**Why Shrinking Is a Superpower for Debugging:**

- 🎯 **Pinpoints the Root Cause**: Reduces a complex failure to the simplest possible case, making it easier to understand the bug.[^1][^2]
- ⚡ **Saves Debugging Time**: Instead of manually trying to simplify a large, random input, the test framework does it for you automatically.[^1]
- 🔄 **Provides Actionable Feedback**: A failure report like `test failed with input: 0` is much more helpful than `test failed with input: -1_234_567_890`.[^1]
- 🛡️ **Uncovers Edge Cases**: Shrinking often reveals simple edge cases (like empty strings, zero, or the smallest possible number) that are the true source of the bug.[^2]


## What is Shrinking in Property-Based Testing?

Shrinking is the process that property-based testing libraries use after they find a failing test case. Once a random input causes a property to fail, the library doesn't just report that input. Instead, it systematically tries to generate "smaller" or "simpler" versions of that input to see if they also cause the test to fail. It continues this process until it finds a minimal failing example, which it then reports to the user.[^2][^1]

The definition of "smaller" depends on the data type:

* For **integers**, smaller means closer to zero.
* For **collections** (like `Vec` or `String`), smaller means having fewer elements.
* For **tuples**, smaller means shrinking each element of the tuple.


## How Shrinking Works: The `quickcheck` Crate in Rust

Let's explore shrinking using `quickcheck`, one of the most popular property-based testing libraries in Rust.[^1]

**Project Setup:**

1. Create a new Rust library project: `cargo new property_testing_demo --lib`
2. Add `quickcheck` and `quickcheck_macros` to your `Cargo.toml`:

```toml
[dev-dependencies]
quickcheck = "1.0.3"
quickcheck_macros = "1.0.0"
```


### A Simple Example: Testing a Flawed `abs` Function

Let's write a property test for a function that should calculate the absolute value of a number. We'll intentionally introduce a bug to see how shrinking helps us find it.

**The Flawed Function:**
Imagine we wrote this incorrect `abs` function:

```rust
// In src/lib.rs
pub fn my_abs(n: i32) -> i32 {
    if n == i32::MIN {
        // A common edge case bug: abs(i32::MIN) overflows
        return i32::MAX;
    }
    if n < 0 {
        -n
    } else {
        n
    }
}
```

**The Property:**
A good property for an `abs` function is that its output should always be non-negative.

```rust
// In src/lib.rs
#[cfg(test)]
mod tests {
    use super::*;
    use quickcheck_macros::quickcheck;

    #[quickcheck]
    fn prop_abs_is_non_negative(n: i32) -> bool {
        my_abs(n) >= 0
    }
}
```

Now, let's introduce a more subtle bug. What if our `abs` function was flawed for *all* negative numbers?

```rust
// A more flawed version of my_abs in src/lib.rs
pub fn my_abs(n: i32) -> i32 {
    if n < -100 {
        // Bug: returns a negative number for small inputs
        return -1;
    }
    if n < 0 {
        -n
    } else {
        n
    }
}
```

**Running the Test without Shrinking (Conceptual):**
If `quickcheck` didn't have shrinking, it might find a random failing input like `-12345`.

**Report without Shrinking:**

```
[quickcheck] TEST FAILED. Arguments: (-12345)
```

This tells you there's a problem, but it's not the simplest case.

**Running the Test with Shrinking (Actual `quickcheck` Behavior):**
`quickcheck` will find a failing case (e.g., `-12345`) and then start shrinking it.

1. **Try a smaller negative number**: How about `-6172`? Still fails.
2. **Try even smaller**: How about `-3086`? Still fails.
3. ...This process continues, usually using a binary search-like approach, until it finds the *smallest* input that triggers the failure. In our case, the bug triggers for anything less than -100, so the smallest failing integer is `-101`.

**Report with Shrinking:**

```
[quickcheck] TEST FAILED. Arguments: (-101)
```

This shrunken result is much more valuable for debugging. It immediately points you to the boundary condition (`-100`) where your function's logic changes and the bug appears.

## A More Complex Example: Reversing a `Vec`

This is a classic example that demonstrates the power of shrinking on collections.[^1]

**The Flawed `reverse` Function:**
Let's write a `reverse` function that has an off-by-one error and misses the first element.

```rust
// In src/lib.rs
pub fn flawed_reverse<T: Clone>(xs: &[T]) -> Vec<T> {
    let mut rev = vec![];
    // Bug: starts from index 1, misses the first element
    for i in 1..xs.len() {
        rev.insert(0, xs[i].clone());
    }
    rev
}
```

**The Property:**
A great property for `reverse` is that reversing a list twice should give you back the original list.

```rust
// In src/lib.rs tests module
#[quickcheck]
fn prop_double_reverse_is_identity(xs: Vec<i32>) -> bool {
    xs == flawed_reverse(&flawed_reverse(&xs))
}
```

**Running the Test:**

1. **Initial Failure**: `quickcheck` might generate a random `Vec` like `[-5, 8, 0, -12, 3]` and find that the property fails.
    - `flawed_reverse([-5, 8, 0, -12, 3])` -> `[3, -12, 0, 8]` (misses `-5`)
    - `flawed_reverse([3, -12, 0, 8])` -> `[8, 0, -12]` (misses `3`)
    - The final result `[8, 0, -12]` does not equal the original `[-5, 8, 0, -12, 3]`. The test fails.
2. **Shrinking Process**: `quickcheck` now tries to simplify `[-5, 8, 0, -12, 3]`.
    - **Shrink the length**: Does the test still fail for `[-5, 8, 0]`? Yes.
    - **Shrink the length again**: Does it fail for `[-5, 8]`? Yes.
    - **Shrink the length again**: Does it fail for `[-5]`? Let's check:
        - `flawed_reverse([-5])` -> `[]` (the loop `1..1` is empty)
        - `flawed_reverse([])` -> `[]`
        - The final `[]` does not equal `[-5]`. Yes, it still fails!
    - **Shrink the value**: The length is now minimal (1). Can we shrink the *value* inside? Let's try ``.
        - `flawed_reverse()` -> `[]`
        - `flawed_reverse([])` -> `[]`
        - `[]` does not equal ``. Yes, it still fails.
3. **Final Report**: `quickcheck` reports the simplest failing case it found.

**Final `quickcheck` Output:**

```
[quickcheck] TEST FAILED. Arguments: ()
```

This minimal example, ``, makes debugging trivial. You can immediately trace the execution with `` and see that the loop in `flawed_reverse` never runs, resulting in an empty `Vec`. This instantly reveals the off-by-one error in the loop's range.

## How to Implement Shrinking for Your Own Types

For `quickcheck` to be able to shrink your custom types, you need to implement the `quickcheck::Arbitrary` trait for them. This trait has two main parts:

1. `arbitrary(g: &mut Gen) -> Self`: Tells `quickcheck` how to generate a random instance of your type.
2. `shrink(&self) -> Box<dyn Iterator<Item = Self>>`: Tells `quickcheck` how to generate "smaller" versions of an instance of your type.

**Example: A Custom `NonEmptyVec` Type**

Let's create a type that represents a `Vec` that must not be empty.

```rust
// In src/lib.rs
use quickcheck::{Arbitrary, Gen};
use std::fmt::Debug;

#[derive(Clone, Debug, PartialEq)]
pub struct NonEmptyVec<T>(pub Vec<T>);

impl<T: Arbitrary + Clone + Debug> Arbitrary for NonEmptyVec<T> {
    // 1. How to generate a NonEmptyVec
    fn arbitrary(g: &mut Gen) -> Self {
        // Ensure the Vec has at least one element
        let mut vec = Vec::<T>::arbitrary(g);
        if vec.is_empty() {
            vec.push(T::arbitrary(g));
        }
        NonEmptyVec(vec)
    }

    // 2. How to shrink a NonEmptyVec
    fn shrink(&self) -> Box<dyn Iterator<Item = Self>> {
        // The core shrinking logic:
        // Try to shrink the inner Vec, but only yield new NonEmptyVecs
        // if they are still non-empty.
        let shrunk_vecs = self.0.shrink() // Use the default Vec shrinker
            .filter(|v| !v.is_empty()) // IMPORTANT: Keep the invariant
            .map(NonEmptyVec); // Wrap the valid shrunk Vecs in our type

        Box::new(shrunked_vecs)
    }
}
```

Now you can use `NonEmptyVec` in your `quickcheck` properties, and it will be generated and shrunk correctly.

## Best Practices for Shrinking and Debugging

- **Trust the Shrinker**: Let the library do the hard work. Your first step in debugging should always be to analyze the minimal failing case provided by the shrinker.
- **Keep Invariants**: When implementing `shrink` for your own types, make sure that every shrunken value you produce still satisfies the type's invariants (like our `NonEmptyVec` still being non-empty).[^4]
- **Focus on Properties**: Write strong properties that capture the essential correctness of your code. A good property, combined with shrinking, is a powerful debugging tool.
- **Analyze the Boundary**: The minimal failing example often points directly to a boundary condition or edge case that you didn't consider (e.g., `0`, `1`, `i32::MIN`, empty list, single-element list).
- **Start Simple**: When debugging a shrunken failure, start by manually tracing the code with that exact simple input. The bug often becomes immediately obvious.


## Summary and Key Takeaways

### **Mental Model: The Professional Kitchen Recipe Debugging System**

Remember our comprehensive professional kitchen recipe debugging analogy:

- 🧪 **Property-Based Test**: A system for testing a recipe with many random combinations of ingredients.
- ❌ **Failing Test Case**: A complex, 15-ingredient dish that comes out wrong.
- 🔄 **Shrinking**: The process of systematically simplifying the failed recipe to find the single root-cause ingredient.
- 🎯 **Minimal Failing Example**: The final report: "The dish fails because of the spoiled mussels."
- 🐛 **Debugging**: Investigating why the mussels were spoiled, instead of re-tasting the entire 15-ingredient dish.


### **What is Shrinking?**

Shrinking is an automated process in property-based testing that takes a large, complex failing input and simplifies it to the smallest possible input that still triggers the failure. This is done to make debugging significantly easier and faster.

### **How Does Shrinking Work?**

1. **Find a Failure**: The testing framework generates random inputs until one causes the property test to fail.
2. **Generate "Smaller" Candidates**: The framework then generates a series of "smaller" versions of the failing input. For example, it might try halving a number, removing half the elements from a list, or trying the empty list.[^4]
3. **Test Candidates**: It tests each smaller candidate against the property.
4. **Repeat**: If a smaller candidate also fails, the framework recursively repeats the shrinking process on that new, smaller failing case.
5. **Report the Minimum**: Once it can no longer find a smaller input that fails, it reports the last known minimal failing case.

### **Practical Benefits for Debugging**

- **Clarity**: Turns a complex, hard-to-reason-about failure into a simple, obvious one.
- **Focus**: Directs your attention to the specific edge case or boundary condition that is causing the problem.
- **Efficiency**: Saves hours of manual debugging time that would be spent trying to simplify the failing case by hand.


### **Implementing Shrinking in Rust**

- **Use `quickcheck`**: The `quickcheck` crate provides built-in shrinking for all standard Rust types.[^1]
- **Implement `Arbitrary`**: For your own custom types, implement the `Arbitrary` trait, which includes the `shrink` method.
- **Preserve Invariants**: When writing a `shrink` method, ensure that all generated "smaller" values still meet the fundamental rules (invariants) of your type.


### **The Professional Advantage**

**Mastering shrinking in property-based testing is like having a complete professional root cause analysis system** for your code, allowing you to debug with unparalleled efficiency and precision:

- 🎯 **Laser-Focused Debugging**: Go directly from a complex failure to the simplest reproducible case.
- ⚡ **Rapid Bug Fixes**: Dramatically reduce the time it takes to understand and fix bugs found by your tests.
- 🛡️ **Robust Edge Case Handling**: Automatically discover and simplify the edge cases that often cause the most subtle bugs.
- 🏆 **Professional Quality Assurance**: Build software that is not only tested against a wide range of inputs but is also easier to debug when failures occur.

**Understanding shrinking transforms you from a programmer who just finds bugs to an engineer** who can efficiently diagnose and fix them. Just as a master chef can quickly pinpoint a single flawed ingredient in a complex dish, a skilled Rust programmer can use shrinking to turn a confusing test failure into a clear, actionable bug report, leading to more robust and reliable software.

1. https://github.com/BurntSushi/quickcheck
2. https://lpalmieri.com/posts/an-introduction-to-property-based-testing-in-rust/
3. https://blog.lambdaclass.com/what-is-property-based-testing/
4. https://getcode.substack.com/p/property-based-testing-5-shrinking
5. https://readyset.io/blog/stateful-property-testing-in-rust
6. https://rustprojectprimer.com/testing/property.html
7. https://rtpg.co/2024/02/02/property-testing-with-imperative-rust/
8. https://www.reddit.com/r/rust/comments/1awmj7n/arbtest_powerfully_tiny_propertybased_testing/
9. https://sunshowers.io/posts/monads-through-pbt/
