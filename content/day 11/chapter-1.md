+++
title = "Defining and Instantiating Structs"
description = "Learn how to define and create instances of structs in Rust to model complex data with named fields and strong typing."
date = 2025-08-03
draft = false
weight = 1
+++

# Defining and Instantiating Structs in Rust: Comprehensive Documentation

Structs are one of Rust's fundamental data types that allow you to create custom data structures by grouping related values together. Unlike tuples, structs provide meaningful names for each piece of data, making code more readable and self-documenting. This comprehensive guide covers everything you need to know about defining and instantiating structs in Rust.

## What are Structs?

A **struct** (short for "structure") is a custom data type that lets you package together and name multiple related values that make up a meaningful group[1][2]. Structs are similar to tuples in that both hold multiple related values of different types, but structs provide names for each piece of data, making them more flexible and expressive[1].

### Key Characteristics

- **Named fields** - Each piece of data has a descriptive name
- **Type safety** - Each field has a specific type enforced at compile time
- **Flexible ordering** - Fields can be accessed by name, not position
- **Custom behavior** - Can have associated methods and implementations
- **Memory efficient** - Zero-cost abstractions with predictable layout

## Basic Struct Definition

### Syntax and Structure

To define a struct, use the `struct` keyword followed by the struct name and field definitions within curly braces[1]:

```rust
struct StructName {
    field1: Type1,
    field2: Type2,
    field3: Type3,
}
```

### Simple Example

```rust
// Define a struct to represent a user account
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

**Key elements:**
- **`struct` keyword** - Declares a new struct type
- **`User`** - The struct name (follows PascalCase convention)
- **Fields** - Each field has a name and type
- **Types** - Can be primitives, other structs, or complex types

### Naming Conventions

```rust
// Good: PascalCase for struct names
struct UserAccount { }
struct GameCharacter { }
struct DatabaseConnection { }

// Good: snake_case for field names
struct Product {
    product_name: String,
    unit_price: f64,
    in_stock: bool,
}
```

## Creating Struct Instances

### Basic Instantiation

To create an instance of a struct, specify concrete values for each field[1]:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    // Create a new instance of User
    let user1 = User {
        active: true,
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        sign_in_count: 1,
    };

    // Field order doesn't matter
    let user2 = User {
        email: String::from("bob@example.com"),
        username: String::from("bob456"),
        active: true,
        sign_in_count: 1,
    };
}
```

### Accessing Struct Fields

Use dot notation to access individual fields[1]:

```rust
fn main() {
    let user = User {
        active: true,
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        sign_in_count: 1,
    };

    // Access fields using dot notation
    println!("Username: {}", user.username);
    println!("Email: {}", user.email);
    println!("Active: {}", user.active);
    println!("Sign-in count: {}", user.sign_in_count);
}
```

### Modifying Struct Fields

To modify struct fields, the entire instance must be declared as mutable[1]:

```rust
fn main() {
    let mut user = User {
        active: true,
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        sign_in_count: 1,
    };

    // Modify individual fields
    user.email = String::from("alice.new@example.com");
    user.sign_in_count += 1;
    user.active = false;

    println!("Updated email: {}", user.email);
    println!("New sign-in count: {}", user.sign_in_count);
}
```

**Important:** Rust doesn't allow marking only certain fields as mutable - the entire instance must be mutable[1].

## Advanced Instantiation Techniques

### Constructor Functions

Create functions that return struct instances for convenient initialization[1]:

```rust
impl User {
    // Constructor function with all parameters
    fn new(username: String, email: String) -> User {
        User {
            active: true,
            username,
            email,
            sign_in_count: 1,
        }
    }

    // Constructor with default values
    fn new_inactive(username: String, email: String) -> User {
        User {
            active: false,
            username,
            email,
            sign_in_count: 0,
        }
    }
}

fn main() {
    let user1 = User::new(
        String::from("alice"),
        String::from("alice@example.com")
    );

    let user2 = User::new_inactive(
        String::from("bob"),
        String::from("bob@example.com")
    );
}
```

### Field Init Shorthand

When parameter names match field names, you can use shorthand syntax[1]:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,        // Shorthand for username: username,
        email,          // Shorthand for email: email,
        sign_in_count: 1,
    }
}

fn main() {
    let user = build_user(
        String::from("user@example.com"),
        String::from("username123")
    );
}
```

### Struct Update Syntax

Create new instances based on existing ones, changing only specific fields[1]:

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        sign_in_count: 1,
    };

    // Create user2 based on user1, changing only email
    let user2 = User {
        email: String::from("alice.work@company.com"),
        ..user1  // Use remaining fields from user1
    };

    // Note: user1 is no longer accessible if non-Copy fields were moved
    println!("User2 email: {}", user2.email);
    println!("User2 username: {}", user2.username);
}
```

**Ownership Note:** The struct update syntax moves data, so the original instance may become partially or fully inaccessible[1].

## Types of Structs

### 1. Named Field Structs (Regular Structs)

Most common type with named fields[3]:

```rust
#[derive(Debug)]
struct Rectangle {
    width: f64,
    height: f64,
}

#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
    city: String,
}

fn main() {
    let rect = Rectangle {
        width: 10.0,
        height: 5.0,
    };

    let person = Person {
        name: String::from("Alice"),
        age: 30,
        city: String::from("New York"),
    };

    println!("Rectangle: {:?}", rect);
    println!("Person: {:?}", person);
}
```

### 2. Tuple Structs

Structs that look like tuples but have a name[3][4]:

```rust
// Define tuple structs
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    // Create instances
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

    // Access fields by index
    println!("Color RGB: ({}, {}, {})", black.0, black.1, black.2);
    println!("Point coordinates: ({}, {}, {})", origin.0, origin.1, origin.2);

    // Destructure tuple structs
    let Color(r, g, b) = black;
    println!("Destructured color: R={}, G={}, B={}", r, g, b);

    let Point(x, y, z) = origin;
    println!("Destructured point: X={}, Y={}, Z={}", x, y, z);
}
```

**Use cases for tuple structs:**
- When you want type safety but field names would be redundant
- Creating distinct types from the same underlying data
- Wrapping single values with semantic meaning

```rust
struct UserId(u64);
struct ProductId(u64);

// These are different types even though both contain u64
fn get_user(id: UserId) { /* ... */ }
fn get_product(id: ProductId) { /* ... */ }

fn main() {
    let user_id = UserId(123);
    let product_id = ProductId(456);

    get_user(user_id);
    get_product(product_id);

    // This would cause a compile error:
    // get_user(product_id);  // Error: expected UserId, found ProductId
}
```

### 3. Unit-Like Structs

Structs without any fields, useful for implementing traits[4]:

```rust
// Define unit-like structs
struct AlwaysEqual;
struct Marker;

fn main() {
    // Create instances
    let subject = AlwaysEqual;
    let marker = Marker;

    // Unit-like structs are often used with traits
    println!("Created unit-like struct instances");
}

// Example with trait implementation
trait Behavior {
    fn do_something(&self);
}

impl Behavior for AlwaysEqual {
    fn do_something(&self) {
        println!("AlwaysEqual is doing something!");
    }
}
```

## Practical Examples

### Example 1: Game Character System

```rust
#[derive(Debug)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Debug)]
struct GameCharacter {
    name: String,
    health: u32,
    level: u32,
    position: Position,
    inventory: Vec,
}

impl GameCharacter {
    fn new(name: String, x: f32, y: f32) -> GameCharacter {
        GameCharacter {
            name,
            health: 100,
            level: 1,
            position: Position { x, y },
            inventory: Vec::new(),
        }
    }

    fn move_to(&mut self, x: f32, y: f32) {
        self.position.x = x;
        self.position.y = y;
    }

    fn add_item(&mut self, item: String) {
        self.inventory.push(item);
    }

    fn take_damage(&mut self, damage: u32) {
        self.health = self.health.saturating_sub(damage);
    }
}

fn main() {
    let mut character = GameCharacter::new(
        String::from("Hero"),
        0.0,
        0.0
    );

    println!("Created character: {:?}", character);

    character.move_to(10.0, 15.0);
    character.add_item(String::from("Sword"));
    character.add_item(String::from("Health Potion"));
    character.take_damage(25);

    println!("Updated character: {:?}", character);
}
```

### Example 2: Configuration Management

```rust
#[derive(Debug, Clone)]
struct DatabaseConfig {
    host: String,
    port: u16,
    database: String,
    username: String,
    password: String,
}

#[derive(Debug, Clone)]
struct ServerConfig {
    bind_address: String,
    port: u16,
    worker_threads: usize,
}

#[derive(Debug, Clone)]
struct AppConfig {
    database: DatabaseConfig,
    server: ServerConfig,
    debug_mode: bool,
    log_level: String,
}

impl DatabaseConfig {
    fn new(host: String, database: String, username: String, password: String) -> Self {
        DatabaseConfig {
            host,
            port: 5432,  // Default PostgreSQL port
            database,
            username,
            password,
        }
    }

    fn with_port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
}

impl ServerConfig {
    fn new(bind_address: String) -> Self {
        ServerConfig {
            bind_address,
            port: 8080,        // Default HTTP port
            worker_threads: 4,  // Default worker count
        }
    }

    fn with_port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }

    fn with_workers(mut self, workers: usize) -> Self {
        self.worker_threads = workers;
        self
    }
}

impl AppConfig {
    fn new(database: DatabaseConfig, server: ServerConfig) -> Self {
        AppConfig {
            database,
            server,
            debug_mode: false,
            log_level: String::from("info"),
        }
    }

    fn development() -> Self {
        let db_config = DatabaseConfig::new(
            String::from("localhost"),
            String::from("myapp_dev"),
            String::from("dev_user"),
            String::from("dev_pass"),
        );

        let server_config = ServerConfig::new(String::from("127.0.0.1"))
            .with_port(3000)
            .with_workers(2);

        AppConfig {
            database: db_config,
            server: server_config,
            debug_mode: true,
            log_level: String::from("debug"),
        }
    }

    fn production() -> Self {
        let db_config = DatabaseConfig::new(
            String::from("prod-db.example.com"),
            String::from("myapp_prod"),
            String::from("prod_user"),
            String::from("secure_password"),
        ).with_port(5432);

        let server_config = ServerConfig::new(String::from("0.0.0.0"))
            .with_port(80)
            .with_workers(8);

        AppConfig {
            database: db_config,
            server: server_config,
            debug_mode: false,
            log_level: String::from("warn"),
        }
    }
}

fn main() {
    let dev_config = AppConfig::development();
    let prod_config = AppConfig::production();

    println!("Development config: {:#?}", dev_config);
    println!("Production config: {:#?}", prod_config);
}
```

## Ownership and Borrowing with Structs

### Owned vs Borrowed Data

```rust
// Struct with owned data
struct UserOwned {
    name: String,     // Owned String
    email: String,    // Owned String
    age: u32,         // Copy type
}

// Struct with borrowed data (requires lifetime parameters)
struct UserBorrowed {
    name: &'a str,    // Borrowed string slice
    email: &'a str,   // Borrowed string slice
    age: u32,         // Copy type
}

fn main() {
    // Owned data - struct owns all its data
    let user_owned = UserOwned {
        name: String::from("Alice"),
        email: String::from("alice@example.com"),
        age: 30,
    };

    // Borrowed data - struct borrows data from elsewhere
    let name = "Bob";
    let email = "bob@example.com";
    let user_borrowed = UserBorrowed {
        name,
        email,
        age: 25,
    };

    println!("Owned user: {} ({})", user_owned.name, user_owned.age);
    println!("Borrowed user: {} ({})", user_borrowed.name, user_borrowed.age);
}
```

### Field-Level Borrowing

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut point = Point { x: 5, y: 10 };

    // Borrow specific fields
    let x_ref = &mut point.x;
    *x_ref = 15;

    // point.y is still accessible
    println!("Y coordinate: {}", point.y);

    // After x_ref goes out of scope, we can use the whole struct again
    println!("Updated point: {:?}", point);
}
```

## Pattern Matching with Structs

### Destructuring Structs

```rust
struct Point {
    x: i32,
    y: i32,
}

struct Color {
    r: u8,
    g: u8,
    b: u8,
}

fn main() {
    let point = Point { x: 10, y: 20 };
    let color = Color { r: 255, g: 0, b: 0 };

    // Destructure all fields
    let Point { x, y } = point;
    println!("Point coordinates: ({}, {})", x, y);

    // Destructure with renaming
    let Color { r: red, g: green, b: blue } = color;
    println!("RGB values: ({}, {}, {})", red, green, blue);

    // Partial destructuring
    let Point { x, .. } = Point { x: 5, y: 15 };
    println!("X coordinate only: {}", x);
}
```

### Match Expressions with Structs

```rust
#[derive(Debug)]
enum Shape {
    Circle { radius: f64 },
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

fn calculate_area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle { radius } => {
            std::f64::consts::PI * radius * radius
        }
        Shape::Rectangle { width, height } => {
            width * height
        }
        Shape::Triangle { base, height } => {
            0.5 * base * height
        }
    }
}

fn main() {
    let shapes = vec![
        Shape::Circle { radius: 5.0 },
        Shape::Rectangle { width: 10.0, height: 8.0 },
        Shape::Triangle { base: 6.0, height: 4.0 },
    ];

    for shape in &shapes {
        let area = calculate_area(shape);
        println!("{:?} has area: {:.2}", shape, area);
    }
}
```

## Common Patterns and Best Practices

### Builder Pattern

```rust
#[derive(Debug)]
struct HttpRequest {
    method: String,
    url: String,
    headers: Vec,
    body: Option,
}

struct HttpRequestBuilder {
    method: String,
    url: String,
    headers: Vec,
    body: Option,
}

impl HttpRequestBuilder {
    fn new(method: &str, url: &str) -> Self {
        HttpRequestBuilder {
            method: method.to_string(),
            url: url.to_string(),
            headers: Vec::new(),
            body: None,
        }
    }

    fn header(mut self, key: &str, value: &str) -> Self {
        self.headers.push((key.to_string(), value.to_string()));
        self
    }

    fn body(mut self, body: String) -> Self {
        self.body = Some(body);
        self
    }

    fn build(self) -> HttpRequest {
        HttpRequest {
            method: self.method,
            url: self.url,
            headers: self.headers,
            body: self.body,
        }
    }
}

fn main() {
    let request = HttpRequestBuilder::new("POST", "https://api.example.com/users")
        .header("Content-Type", "application/json")
        .header("Authorization", "Bearer token123")
        .body(r#"{"name": "Alice", "email": "alice@example.com"}"#.to_string())
        .build();

    println!("HTTP Request: {:#?}", request);
}
```

### Default Implementation

```rust
#[derive(Debug)]
struct Config {
    host: String,
    port: u16,
    timeout: u64,
    retries: u32,
    debug: bool,
}

impl Default for Config {
    fn default() -> Self {
        Config {
            host: String::from("localhost"),
            port: 8080,
            timeout: 30,
            retries: 3,
            debug: false,
        }
    }
}

impl Config {
    fn new() -> Self {
        Self::default()
    }

    fn with_host(mut self, host: String) -> Self {
        self.host = host;
        self
    }

    fn with_port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }

    fn production() -> Self {
        Self::default()
            .with_host(String::from("prod.example.com"))
            .with_port(443)
    }
}

fn main() {
    let default_config = Config::default();
    let custom_config = Config::new()
        .with_host(String::from("dev.example.com"))
        .with_port(3000);
    let prod_config = Config::production();

    println!("Default: {:?}", default_config);
    println!("Custom: {:?}", custom_config);
    println!("Production: {:?}", prod_config);
}
```

## Debugging and Display

### Debug Trait

```rust
// Automatically derive Debug for simple structs
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

// Custom Debug implementation
struct CustomStruct {
    important: String,
    secret: String,
}

impl std::fmt::Debug for CustomStruct {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        f.debug_struct("CustomStruct")
            .field("important", &self.important)
            .field("secret", &"[REDACTED]")  // Hide sensitive data
            .finish()
    }
}

fn main() {
    let point = Point { x: 10, y: 20 };
    println!("Point: {:?}", point);
    println!("Point (pretty): {:#?}", point);

    let custom = CustomStruct {
        important: String::from("public data"),
        secret: String::from("sensitive information"),
    };
    println!("Custom: {:?}", custom);
}
```

### Display Trait

```rust
use std::fmt;

struct User {
    username: String,
    email: String,
    active: bool,
}

impl fmt::Display for User {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} ({}) - {}",
               self.username,
               self.email,
               if self.active { "Active" } else { "Inactive" })
    }
}

impl fmt::Debug for User {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("User")
            .field("username", &self.username)
            .field("email", &self.email)
            .field("active", &self.active)
            .finish()
    }
}

fn main() {
    let user = User {
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        active: true,
    };

    println!("Display: {}", user);
    println!("Debug: {:?}", user);
}
```

## Performance Considerations

### Memory Layout

```rust
use std::mem;

#[repr(C)]  // Use C-compatible layout
struct OptimizedStruct {
    // Order fields by size (largest first) to minimize padding
    large_field: u64,     // 8 bytes
    medium_field: u32,    // 4 bytes
    small_field: u16,     // 2 bytes
    tiny_field: u8,       // 1 byte
    boolean_field: bool,  // 1 byte
}

struct UnoptimizedStruct {
    // Poor ordering leads to padding
    tiny_field: u8,       // 1 byte + 7 bytes padding
    large_field: u64,     // 8 bytes
    small_field: u16,     // 2 bytes + 6 bytes padding
    medium_field: u32,    // 4 bytes + 4 bytes padding
    boolean_field: bool,  // 1 byte + 7 bytes padding
}

fn main() {
    println!("Optimized size: {} bytes", mem::size_of::());
    println!("Unoptimized size: {} bytes", mem::size_of::());

    println!("Optimized alignment: {} bytes", mem::align_of::());
    println!("Unoptimized alignment: {} bytes", mem::align_of::());
}
```

### Zero-Cost Abstractions

```rust
// Structs are zero-cost abstractions - they have no runtime overhead
struct Point3D {
    x: f64,
    y: f64,
    z: f64,
}

impl Point3D {
    fn new(x: f64, y: f64, z: f64) -> Self {
        Point3D { x, y, z }
    }

    fn distance_from_origin(&self) -> f64 {
        (self.x * self.x + self.y * self.y + self.z * self.z).sqrt()
    }

    // This compiles to the same assembly as manual computation
    fn add(&self, other: &Point3D) -> Point3D {
        Point3D {
            x: self.x + other.x,
            y: self.y + other.y,
            z: self.z + other.z,
        }
    }
}

fn main() {
    let p1 = Point3D::new(1.0, 2.0, 3.0);
    let p2 = Point3D::new(4.0, 5.0, 6.0);
    let p3 = p1.add(&p2);

    println!("Point: ({}, {}, {})", p3.x, p3.y, p3.z);
    println!("Distance from origin: {:.2}", p3.distance_from_origin());
}
```

## Common Errors and Solutions

### Ownership Errors

```rust
// Error: Partial move
struct Container {
    data: String,
    count: u32,
}

fn problematic_usage() {
    let container = Container {
        data: String::from("hello"),
        count: 5,
    };

    let extracted_data = container.data;  // Partial move
    // println!("{}", container.count);   // Error: container partially moved

    // Solution: Use references or clone
    let container2 = Container {
        data: String::from("hello"),
        count: 5,
    };

    let data_ref = &container2.data;      // Borrow instead
    println!("Data: {}, Count: {}", data_ref, container2.count);  // OK

    let data_clone = container2.data.clone();  // Clone if needed
    println!("Cloned: {}, Count: {}", data_clone, container2.count);  // OK
}
```

### Lifetime Errors

```rust
// Error: Missing lifetime specifiers
/*
struct UserRef {
    name: &str,     // Error: missing lifetime
    email: &str,    // Error: missing lifetime
}
*/

// Solution: Add lifetime parameters
struct UserRef {
    name: &'a str,
    email: &'a str,
}

fn create_user_ref() -> UserRef {
    UserRef {
        name: "Alice",      // String literals have 'static lifetime
        email: "alice@example.com",
    }
}

fn main() {
    let user = create_user_ref();
    println!("User: {} ({})", user.name, user.email);
}
```

## Summary and Key Takeaways

### **Essential Concepts**

**Structs** are fundamental data types in Rust that enable:
- **Grouping related data** with meaningful names
- **Type safety** through compile-time checking
- **Zero-cost abstractions** with no runtime overhead
- **Flexible data modeling** for complex applications

### **Key Syntax Patterns**

```rust
// Definition
struct TypeName {
    field1: Type1,
    field2: Type2,
}

// Instantiation
let instance = TypeName {
    field1: value1,
    field2: value2,
};

// Field access
let value = instance.field1;

// Modification (requires mut)
let mut instance = TypeName { /* ... */ };
instance.field1 = new_value;
```

### **Best Practices**

- **Use PascalCase** for struct names, snake_case for fields
- **Prefer owned types** (`String`) over references (`&str`) in struct fields
- **Implement common traits** like `Debug`, `Clone`, `Default` when appropriate
- **Use constructor functions** for complex initialization logic
- **Consider field ordering** for optimal memory layout
- **Leverage pattern matching** for elegant struct destructuring

### **Advanced Features**

- **Field init shorthand** for reducing boilerplate
- **Struct update syntax** for creating variants of existing instances
- **Tuple structs** for type safety without field names
- **Unit structs** for marker types and trait implementations
- **Builder patterns** for complex configuration objects

**Structs are the foundation of data modeling in Rust**, providing type safety, expressiveness, and performance. Understanding how to define and instantiate them effectively is crucial for writing idiomatic Rust code that is both safe and efficient[1][3][4].

1. https://doc.rust-lang.org/book/ch05-01-defining-structs.html
2. https://www.w3schools.com/rust/rust_structs.php
3. https://doc.rust-lang.org/rust-by-example/custom_types/structs.html
4. https://doc.rust-lang.org/std/keyword.struct.html
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch05-01-defining-structs.html
6. https://www.programiz.com/rust/struct
7. https://stackoverflow.com/questions/68218315/how-can-i-run-a-function-on-initialization-of-a-struct
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/structs.html
9. https://users.rust-lang.org/t/initializing-struct-field-by-field/103607
10. https://www.risein.com/courses/rust-programming/rust-programming-structs
11. https://serokell.io/blog/structs-in-rust
12. https://users.rust-lang.org/t/instantiate-struct-without-setting-values/20353
13. https://doc.rust-lang.org/book/ch05-02-example-structs.html
14. https://www.youtube.com/watch?v=PCjuO-Bv5FI
15. https://www.reddit.com/r/rust/comments/1al5r1t/initialize_struct_from_memory_address/
16. https://zerotomastery.io/blog/rust-struct-guide/
17. https://dev.to/francescoxx/structs-in-rust-2a9a
18. https://xaeroxe.github.io/init-struct-pattern/
19. https://dev.to/ajtech0001/mastering-structs-in-rust-from-basic-data-organization-to-method-implementation-11ol
20. https://users.rust-lang.org/t/how-can-i-initialize-fields-of-struct-asynchronously/97769
21. https://www.reddit.com/r/rust/comments/1hakgoq/can_i_have_a_struct_with_uninitialized_fields/
22. https://doc.rust-lang.org/reference/items/structs.html
23. https://dev.to/sonubardai/structs-and-enums-in-rust-hme
24. https://doc.rust-lang.org/book/ch05-00-structs.html
25. https://doc.rust-lang.org/rust-by-example/scope/lifetime/struct.html
26. https://rust-book.cs.brown.edu/ch05-01-defining-structs.html
27. https://www.reddit.com/r/ProgrammingLanguages/comments/18pauxf/rusts_struct_initializer_syntax_was_a_mistake/
