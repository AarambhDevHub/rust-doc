+++
title = "Mocking and Test Doubles"
description = "A guide to using mocking and test doubles in Rust to simulate dependencies during testing."
date = "2025-08-13"
weight = 4
+++

# Mocking and Test Doubles in Rust: Comprehensive Documentation for Beginners

Understanding mocking and test doubles in Rust is like learning to **run drills in a professional restaurant kitchen using stand-in ingredients and equipment** – you want to test your chefs' skills and workflows without using expensive ingredients or relying on external suppliers who might be unavailable. Just as a restaurant manager might use water instead of wine for a training session, or a detailed printout instead of a real-time order from the dining room, a Rust programmer uses test doubles to simulate dependencies and isolate the code being tested, ensuring that tests are fast, reliable, and independent of external systems.

## The Professional Restaurant Drill Analogy 🍽️

### Imagine You're Running Drills for Your Kitchen Staff Without Using Real Resources

**The Problem with Testing on Live Systems:**

```rust
// ❌ Testing on live systems - like training a new chef using expensive, real truffles
// and relying on a real-time connection to your supplier's database.
fn test_dish_preparation_on_live_system() {
    let expensive_ingredient = get_real_truffles_from_supplier(); // Slow, expensive, depends on network
    let live_order = get_live_order_from_pos_system(); // Depends on another live system

    // Test the chef's preparation logic
    let dish = prepare_dish(expensive_ingredient, live_order);

    // Assertions...
}
// This is slow, expensive, unreliable, and not isolated.
```

**The Power of Mocking - Simulating Dependencies for Focused Drills:**

```rust
// ✅ Mocking approach - using stand-in ingredients and simulated orders for drills
fn test_dish_preparation_with_mocks() {
    // Use a "fake" truffle that behaves like a real one for the test
    let fake_truffle = create_fake_truffle();

    // Create a "mock" order system that returns a predictable test order
    let mock_order = create_mock_order_system();

    // Test only the chef's preparation logic in isolation
    let dish = prepare_dish(fake_truffle, mock_order.get_order());

    // Assertions...
}
// This is fast, cheap, reliable, and tests the chef's logic in isolation.
```

**Why Mocking and Test Doubles Are Essential in Rust:**

- 🎯 **Isolation**: Test a unit of code (a "unit") without interference from its dependencies.[^3]
- 🚀 **Speed**: Avoid slow operations like network calls, database queries, or file I/O.
- 💰 **Cost**: Avoid using expensive or limited resources (e.g., API rate limits).
- 🔮 **Determinism**: Control the behavior of dependencies to create predictable and repeatable tests.
- 🛡️ **Design Improvement**: Encourages designing loosely-coupled components that are easier to test and maintain.


## What Are Test Doubles? The Different Types of Stand-ins

"Test Double" is the general term for any object or function that "stands in" for a real one during a test. There are several types :[^6]

* **Dummy**: An object that is passed around but never actually used. It's just there to fill a parameter list.
* **Fake**: A working implementation, but a simplified one. For example, an in-memory database instead of a real one.
* **Stub**: Provides canned answers to calls made during the test. It doesn't respond to anything outside of what's pre-programmed for the test.
* **Spy**: A stub that also records some information about how it was called (e.g., how many times, with what arguments).
* **Mock**: An object pre-programmed with expectations of how it will be called. A mock will cause the test to fail if it's not called as expected.[^6]

In practice, the term "mock" is often used loosely to refer to any of these, but it's useful to know the distinctions.

## How to Implement Mocking in Rust: Common Patterns

Rust's strong type system and focus on compile-time guarantees make mocking different from dynamically-typed languages. The most common approach is to use **traits** to define dependencies and then provide a "real" implementation for production and a "mock" implementation for tests.

### Step-by-Step Example: Mocking a Database Connection

Let's build a simple `OrderProcessor` that needs to save orders to a database. We'll mock the database so we can test the processor's logic in isolation.

**1. Define a Trait for the Dependency**

First, we define a trait that describes what our dependency can do. This is our "contract."

```rust
// src/lib.rs

pub struct Order {
    pub order_id: u32,
    pub amount: f64,
}

// This trait defines the contract for our database.
// Any struct that implements this trait can act as our database.
pub trait Database {
    fn save_order(&mut self, order: &Order) -> Result<(), String>;
    fn get_order(&self, order_id: u32) -> Option<Order>;
}
```

**2. Implement the Real Dependency**

For our production code, we would have a real database implementation. For this example, we'll create a simple in-memory "fake" that acts as our real implementation.

```rust
// A simple "real" implementation (in this case, a Fake)
pub struct InMemoryDatabase {
    orders: std::collections::HashMap<u32, Order>,
}

impl InMemoryDatabase {
    pub fn new() -> Self {
        InMemoryDatabase { orders: std::collections::HashMap::new() }
    }
}

impl Database for InMemoryDatabase {
    fn save_order(&mut self, order: &Order) -> Result<(), String> {
        self.orders.insert(order.order_id, order.clone()); // Simplified
        Ok(())
    }

    fn get_order(&self, order_id: u32) -> Option<Order> {
        self.orders.get(&order_id).cloned() // Simplified
    }
}
```

*(Note: To allow cloning, we'd add `#[derive(Clone)]` to the `Order` struct)*

**3. Implement the Component that Uses the Dependency**

Our `OrderProcessor` will be generic over any type that implements the `Database` trait. This is the key to dependency injection in Rust.

```rust
pub struct OrderProcessor<T: Database> {
    db: T, // The processor owns its database dependency
}

impl<T: Database> OrderProcessor<T> {
    pub fn new(db: T) -> Self {
        OrderProcessor { db }
    }

    // Business logic to test: process a new order and save it
    pub fn process_new_order(&mut self, order_id: u32, amount: f64) -> Result<Order, String> {
        if amount <= 0.0 {
            return Err("Order amount must be positive".to_string());
        }

        let new_order = Order { order_id, amount };
        self.db.save_order(&new_order)?;

        Ok(new_order)
    }
}
```

**4. Create a Mock Implementation for Testing**

Now, we create a mock version of our database inside a `#[cfg(test)]` module. This mock will allow us to control its behavior and check how it was called.

```rust
// ... inside src/lib.rs

#[cfg(test)]
mod tests {
    use super::*;

    // This is our mock database. It will simulate the real one.
    struct MockDatabase {
        // We can add fields to track calls or pre-program responses
        save_order_was_called: bool,
        should_save_fail: bool,
    }

    impl MockDatabase {
        fn new() -> Self {
            MockDatabase {
                save_order_was_called: false,
                should_save_fail: false,
            }
        }
    }

    // Implement the `Database` trait for our mock
    impl Database for MockDatabase {
        fn save_order(&mut self, _order: &Order) -> Result<(), String> {
            self.save_order_was_called = true;
            if self.should_save_fail {
                Err("Mock DB error: save failed".to_string())
            } else {
                Ok(())
            }
        }

        fn get_order(&self, _order_id: u32) -> Option<Order> {
            // For this test, we don't need `get_order`, so we can leave it simple.
            None
        }
    }

    // --- Our Tests ---

    #[test]
    fn test_process_new_order_successfully() {
        // Arrange
        let mock_db = MockDatabase::new();
        let mut order_processor = OrderProcessor::new(mock_db);

        // Act
        let result = order_processor.process_new_order(101, 50.0);

        // Assert
        assert!(result.is_ok());
        let saved_order = result.unwrap();
        assert_eq!(saved_order.order_id, 101);
        assert_eq!(saved_order.amount, 50.0);

        // Assert that our mock was used correctly
        assert!(order_processor.db.save_order_was_called);
    }

    #[test]
    fn test_process_new_order_with_invalid_amount() {
        // Arrange
        let mock_db = MockDatabase::new();
        let mut order_processor = OrderProcessor::new(mock_db);

        // Act
        let result = order_processor.process_new_order(102, -10.0);

        // Assert
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), "Order amount must be positive");

        // Assert that the database was NOT called for an invalid order
        assert!(!order_processor.db.save_order_was_called);
    }

    #[test]
    fn test_process_new_order_db_failure() {
        // Arrange
        let mut mock_db = MockDatabase::new();
        mock_db.should_save_fail = true; // Configure the mock to fail
        let mut order_processor = OrderProcessor::new(mock_db);

        // Act
        let result = order_processor.process_new_order(103, 75.0);

        // Assert
        assert!(result.is_err());
        assert!(result.unwrap_err().contains("Mock DB error"));
    }
}
```

This example shows how we "inject" a dependency (our `MockDatabase`) into the `OrderProcessor` to test its logic without ever touching a real database.

## Using Mocking Libraries: `mockall`

Manually creating mocks can be tedious. Libraries like `mockall` can automate this process.

`mockall` uses procedural macros to generate mock objects for your traits automatically.

**1. Add `mockall` to `Cargo.toml`:**

```toml
[dev-dependencies]
mockall = "0.12.1"
```

**2. Use `mockall` to generate a mock:**

```rust
// src/lib.rs

// ... (Order struct and Database trait definitions remain the same) ...

#[cfg(test)]
mod tests {
    use super::*;
    use mockall::*; // Import mockall macros

    // Use mockall to automatically create a MockDatabase
    mock! {
        pub Database {
            fn save_order(&mut self, order: &Order) -> Result<(), String>;
            fn get_order(&self, order_id: u32) -> Option<Order>;
        }
    }

    #[test]
    fn test_process_order_with_mockall() {
        // Arrange
        let mut mock_db = MockDatabase::new();

        // Set up expectations: we expect `save_order` to be called once
        // with an order where the amount is 50.0, and it should return Ok.
        mock_db.expect_save_order()
            .withf(|order| order.amount == 50.0) // Check the arguments
            .times(1)                            // Expect it to be called once
            .returning(|_| Ok(()));              // What it should return

        let mut order_processor = OrderProcessor::new(mock_db);

        // Act
        let result = order_processor.process_new_order(101, 50.0);

        // Assert
        assert!(result.is_ok());
    }

    #[test]
    fn test_invalid_order_does_not_call_db_with_mockall() {
        // Arrange
        let mut mock_db = MockDatabase::new();

        // Set up expectations: `save_order` should NEVER be called.
        mock_db.expect_save_order()
            .times(0); // Expect zero calls

        let mut order_processor = OrderProcessor::new(mock_db);

        // Act
        let result = order_processor.process_new_order(102, -10.0);

        // Assert
        assert!(result.is_err());
    }
}
```

`mockall` automatically checks if the expectations were met when the `mock_db` goes out of scope. If `save_order` wasn't called as expected, the test will panic and fail. This makes `mockall` a true "mocking" library, not just a stubbing one.

## Summary and Key Takeaways

### **Mental Model: The Complete Professional Restaurant Drill System**

Remember our comprehensive professional restaurant drill analogy:

- 🎯 **Mocking \& Test Doubles** = **Stand-in ingredients and simulated systems** - Allows for focused, repeatable, and isolated training drills.
- 📝 **Traits as Contracts** = **Drill Protocols** - Defining the expected interactions with dependencies.
- ⚙️ **Dependency Injection** = **Providing Drill Equipment** - Supplying the system-under-test with a mock or real dependency.
- 🤖 **Mocking Libraries (`mockall`)** = **Automated Drill Setup** - Automatically generating stand-ins and verifying drill outcomes.
- 🛡️ **Confidence** = **Operational Readiness** - Knowing each component works perfectly in isolation before integrating.


### **The TDD \& Mocking Workflow**

1. **Identify the Dependency**: Determine which external component you need to simulate (e.g., a database, a web service, the file system).
2. **Define a Trait**: Create a trait that defines the public API of that dependency.
3. **Use Generic Constraints**: Write your component to be generic over any type that implements your trait. This is the key to dependency injection.
4. **Write the Test**:
    * In your test module, create a mock implementation of the trait (either manually or with a library like `mockall`).
    * Configure the mock's behavior (e.g., what it should return, if it should fail).
    * Set expectations on the mock (e.g., it must be called exactly once).
5. **Inject the Mock**: Create an instance of your component-under-test and pass it the mock object.
6. **Run and Assert**: Execute your test and let the mocking library (or your manual assertions) verify that the interactions were correct.

### **Why This Pattern Is Idiomatic in Rust**

- **Compile-Time Safety**: Using traits and generics ensures that both your real implementation and your mock implementation adhere to the same contract. The compiler verifies this for you.
- **No Runtime Overhead**: Because this pattern uses generics, Rust's monomorphization process creates specialized versions of your code at compile time. In production, your `OrderProcessor<InMemoryDatabase>` will have no overhead from the generic design.
- **Clear Boundaries**: This approach forces you to think about the boundaries and contracts between different parts of your application, leading to better software design.


### **Best Practices for Mocking in Rust**

- **Mock Traits, Not Structs**: The most flexible approach is to mock behavior defined by traits.
- **Keep Mocks Simple**: A mock should only contain the logic needed for the test. Avoid making your mocks too complex.
- **Use Mocking Libraries**: For non-trivial cases, libraries like `mockall` save a lot of boilerplate and provide powerful features like call count verification and argument matching.
- **Test Behavior, Not Implementation**: Focus your tests on "what" your component does, not "how" it does it. Mocks help with this by abstracting away the implementation details of dependencies.


### **The Professional Advantage**

**Mastering mocking and test doubles in Rust is like having a complete professional restaurant simulation environment** that allows you to train and test every aspect of your operation with precision and control:

- 🎯 **Isolated Testing**: Confidently test individual components without external interference.
- 🚀 **Fast and Reliable Tests**: Create a test suite that runs quickly and consistently, encouraging frequent testing.
- 💡 **Improved Design**: Promotes loosely-coupled, modular architecture that is easier to maintain and extend.
- 🛡️ **Total Confidence**: Build complex systems with the assurance that each part has been individually verified.
- 🏆 **Professional Engineering**: Applying industry-standard testing practices to build robust and reliable software.

**Understanding mocking transforms you from a programmer who tests integrated systems to an engineer** who builds verifiable components. Just as a master restaurant manager uses controlled drills to perfect every role in the kitchen, a skilled Rust programmer uses mocks to perfect every component of their application, leading to more resilient, maintainable, and high-quality software.

1. https://docs.rs/mockall_double/latest/mockall_double/attr.double.html
2. https://www.reddit.com/r/rust/comments/182aloz/mocking_in_rust/
3. https://blog.logrocket.com/mocking-rust-mockall-alternatives/
4. https://www.youtube.com/watch?v=XcoTYFdq_L0
5. https://archive.fosdem.org/2018/schedule/event/rust_testing_mocking/attachments/slides/2113/export/events/attachments/rust_testing_mocking/slides/2113/testing_in_rust_by_donald_whyte.pdf
6. https://dev.to/_cds_/not-everything-is-a-mock-lets-explore-test-doubles-1n13
7. https://www.youtube.com/watch?v=8XaVlL3lObQ
8. https://stackoverflow.com/questions/77120851/rust-mocking-stdprocesschild-for-test
9. https://keploy.io/blog/community/mastering-mocking-a-complete-guide-to-mocks-and-other-test-doubles
10. https://users.rust-lang.org/t/how-to-properly-unit-test-with-mock-injection/60977
