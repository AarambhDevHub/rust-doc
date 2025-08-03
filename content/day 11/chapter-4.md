+++
title = "Struct Update Syntax"
description = "Learn how to use Rust's struct update syntax to create new instances by reusing fields from existing structs efficiently."
date = 2025-08-03
draft = false
weight = 4
+++

# Struct Update Syntax in Rust: Comprehensive Documentation

Struct update syntax is a powerful Rust feature that allows you to create new struct instances by copying most fields from existing instances while only specifying the fields you want to change. This feature provides an elegant way to create variations of existing structs without verbose field-by-field copying.

## What is Struct Update Syntax?

**Struct update syntax** uses the `..` operator to specify that remaining fields (those not explicitly listed) should have the same values as the corresponding fields in a given instance[1]. This is also known as **Functional Record Update (FRU)** syntax[2].

### Basic Syntax

```rust
StructName {
    field1: new_value1,
    field2: new_value2,
    ..existing_instance
}
```

The `..existing_instance` must come **last** in the struct literal and specifies that any remaining fields should get their values from the corresponding fields in `existing_instance`[1].

## Basic Usage Examples

### Simple Struct Update

```rust
#[derive(Debug)]
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        sign_in_count: 1,
        active: true,
    };

    // Create user2 with different email but same other fields
    let user2 = User {
        email: String::from("alice.work@company.com"),
        ..user1  // Copy remaining fields from user1
    };

    println!("User2: {:?}", user2);
}
```

### Multiple Field Updates

```rust
fn main() {
    let user1 = User {
        username: String::from("bob456"),
        email: String::from("bob@example.com"),
        sign_in_count: 5,
        active: true,
    };

    // Update multiple fields while keeping others
    let user2 = User {
        username: String::from("bob_updated"),
        sign_in_count: 6,
        ..user1  // Copy email and active from user1
    };

    println!("Updated user: {:?}", user2);
}
```

## Ownership Semantics and Move Behavior

### The Critical Rule: Struct Update Moves Data

**Important:** Struct update syntax uses `=` like an assignment, which means it **moves** the data from the source struct[1]. This has significant implications for ownership:

```rust
fn demonstrate_move_semantics() {
    let user1 = User {
        username: String::from("charlie"),
        email: String::from("charlie@example.com"),
        sign_in_count: 1,
        active: true,
    };

    let user2 = User {
        email: String::from("charlie.new@example.com"),
        ..user1  // Moves username from user1 to user2
    };

    // Error! user1 is no longer accessible because username was moved
    // println!("Original user: {:?}", user1);  // Compile error!

    println!("New user: {:?}", user2);
}
```

### When the Original Struct Remains Valid

The original struct remains usable **only if** all moved fields implement the `Copy` trait:

```rust
#[derive(Debug)]
struct Point {
    x: i32,      // Copy type
    y: i32,      // Copy type
    name: String, // Non-Copy type
}

fn main() {
    let point1 = Point {
        x: 10,
        y: 20,
        name: String::from("origin"),
    };

    let point2 = Point {
        x: 30,
        ..point1  // Moves name, copies y
    };

    // Error: Cannot use point1 because name was moved
    // println!("Original point: {:?}", point1);

    // However, we can still access individual Copy fields
    println!("Original y coordinate: {}", point1.y);  // This works!
}
```

### Safe Usage: Updating Only Copy Fields

```rust
#[derive(Debug)]
struct Configuration {
    host: String,
    port: u16,        // Copy type
    timeout: u32,     // Copy type
    debug_mode: bool, // Copy type
}

fn main() {
    let config1 = Configuration {
        host: String::from("localhost"),
        port: 8080,
        timeout: 30,
        debug_mode: false,
    };

    // Only update Copy fields - original remains valid
    let config2 = Configuration {
        host: String::from("production.server.com"), // Provide new String
        port: 443,    // Update Copy field
        ..config1     // Copy timeout and debug_mode
    };

    // config1 is still valid because we didn't move any non-Copy fields
    println!("Original config still valid: {:?}", config1);
    println!("New config: {:?}", config2);
}
```

## Working with the Default Trait

### Using ..Default::default()

One of the most common and powerful uses of struct update syntax is with the `Default` trait[3][4]:

```rust
#[derive(Debug, Default)]
struct ServerConfig {
    host: String,          // Default: empty string
    port: u16,            // Default: 0
    max_connections: u32, // Default: 0
    timeout_seconds: u32, // Default: 0
    debug_mode: bool,     // Default: false
    ssl_enabled: bool,    // Default: false
}

fn main() {
    // Create config with only specific fields, rest use defaults
    let config = ServerConfig {
        host: String::from("api.example.com"),
        port: 443,
        ssl_enabled: true,
        ..Default::default()  // Use default values for other fields
    };

    println!("Config: {:#?}", config);
}
```

### Custom Default Implementation

```rust
#[derive(Debug)]
struct DatabaseConfig {
    host: String,
    port: u16,
    database: String,
    pool_size: u32,
    timeout_ms: u64,
}

impl Default for DatabaseConfig {
    fn default() -> Self {
        DatabaseConfig {
            host: String::from("localhost"),
            port: 5432,
            database: String::from("default_db"),
            pool_size: 10,
            timeout_ms: 5000,
        }
    }
}

fn main() {
    // Use custom defaults with specific overrides
    let config = DatabaseConfig {
        database: String::from("production_db"),
        pool_size: 50,
        ..Default::default()
    };

    println!("Database config: {:#?}", config);
}
```

## Advanced Patterns and Use Cases

### Builder Pattern Integration

Struct update syntax works well with builder patterns:

```rust
#[derive(Debug, Default)]
struct HttpRequest {
    method: String,
    url: String,
    headers: Vec,
    timeout: Option,
    follow_redirects: bool,
    max_redirects: u32,
}

impl HttpRequest {
    fn get(url: &str) -> Self {
        HttpRequest {
            method: String::from("GET"),
            url: url.to_string(),
            ..Default::default()
        }
    }

    fn post(url: &str) -> Self {
        HttpRequest {
            method: String::from("POST"),
            url: url.to_string(),
            ..Default::default()
        }
    }

    fn with_timeout(self, timeout: u32) -> Self {
        HttpRequest {
            timeout: Some(timeout),
            ..self
        }
    }

    fn with_redirects(self, max_redirects: u32) -> Self {
        HttpRequest {
            follow_redirects: true,
            max_redirects,
            ..self
        }
    }
}

fn main() {
    let request = HttpRequest::post("https://api.example.com/users")
        .with_timeout(5000)
        .with_redirects(3);

    println!("Request: {:#?}", request);
}
```

### Configuration Templates

Create configuration templates using struct update syntax:

```rust
#[derive(Debug, Clone)]
struct AppConfig {
    environment: String,
    database_url: String,
    redis_url: String,
    log_level: String,
    port: u16,
    workers: u32,
    debug: bool,
}

impl AppConfig {
    fn development_template() -> Self {
        AppConfig {
            environment: String::from("development"),
            database_url: String::from("postgres://localhost/myapp_dev"),
            redis_url: String::from("redis://localhost:6379/0"),
            log_level: String::from("debug"),
            port: 3000,
            workers: 1,
            debug: true,
        }
    }

    fn production_template() -> Self {
        AppConfig {
            environment: String::from("production"),
            log_level: String::from("info"),
            port: 8080,
            workers: 4,
            debug: false,
            ..Self::development_template()  // Use dev template as base
        }
    }

    fn staging_from_prod(database_url: String) -> Self {
        AppConfig {
            environment: String::from("staging"),
            database_url,
            ..Self::production_template()  // Use prod config as base
        }
    }
}

fn main() {
    let dev_config = AppConfig::development_template();
    let prod_config = AppConfig::production_template();
    let staging_config = AppConfig::staging_from_prod(
        String::from("postgres://staging-db/myapp_staging")
    );

    println!("Development: {:#?}", dev_config);
    println!("Production: {:#?}", prod_config);
    println!("Staging: {:#?}", staging_config);
}
```

## Type-Changing Struct Updates

Rust also supports struct update syntax between instances of the same struct with different generic parameters, as long as compatible fields have the same types[5]:

```rust
use std::marker::PhantomData;

// Generic struct with state markers
struct Connection {
    host: String,
    port: u16,
    timeout: u32,
    _state: PhantomData,
}

struct Connected;
struct Disconnected;

impl Connection {
    fn new(host: String, port: u16) -> Self {
        Connection {
            host,
            port,
            timeout: 5000,
            _state: PhantomData,
        }
    }

    fn connect(self) -> Connection {
        println!("Connecting to {}:{}", self.host, self.port);

        Connection {
            _state: PhantomData,  // Change the state
            ..self  // Copy all compatible fields
        }
    }
}

impl Connection {
    fn execute_query(&self, query: &str) {
        println!("Executing query: {}", query);
    }

    fn disconnect(self) -> Connection {
        println!("Disconnecting from {}:{}", self.host, self.port);

        Connection {
            _state: PhantomData,
            ..self
        }
    }
}

fn main() {
    let disconnected_conn = Connection::new(
        String::from("localhost"),
        5432
    );

    let connected_conn = disconnected_conn.connect();
    connected_conn.execute_query("SELECT * FROM users");

    let _disconnected_again = connected_conn.disconnect();
}
```

## Tuple Struct Updates

Struct update syntax also works with tuple structs:

```rust
#[derive(Debug)]
struct Color(u8, u8, u8);

fn main() {
    let red = Color(255, 0, 0);

    // Create variations using tuple struct update syntax
    let pink = Color {
        0: 255,     // Red component
        1: 192,     // Green component
        ..red       // Copy blue component
    };

    println!("Original red: {:?}", red);   // Still valid (Copy type)
    println!("Pink color: {:?}", pink);
}
```

## Limitations and Considerations

### Field Visibility Requirements

All fields involved in struct update must be **publicly visible** at the update location[2]:

```rust
mod private_fields {
    pub struct User {
        pub username: String,
        pub email: String,
        private_id: u64,        // Private field
    }

    impl User {
        pub fn new(username: String, email: String) -> Self {
            User {
                username,
                email,
                private_id: 12345,
            }
        }
    }
}

fn main() {
    use private_fields::User;

    let user1 = User::new(
        String::from("alice"),
        String::from("alice@example.com")
    );

    // This won't work because private_id is not accessible
    /*
    let user2 = User {
        username: String::from("bob"),
        ..user1  // Error: private_id is not accessible
    };
    */
}
```

### Performance Considerations

Struct update syntax is a **zero-cost abstraction** - it compiles to the same code as explicitly listing all fields[2]:

```rust
// These are equivalent at compile time:

// Using struct update syntax
let user2 = User {
    email: String::from("new@example.com"),
    ..user1
};

// Explicitly listing fields
let user2 = User {
    email: String::from("new@example.com"),
    username: user1.username,
    sign_in_count: user1.sign_in_count,
    active: user1.active,
};
```

## Best Practices

### 1. Be Mindful of Ownership

Always consider which fields will be moved vs copied:

```rust
#[derive(Debug)]
struct Document {
    title: String,      // Will be moved
    content: String,    // Will be moved
    line_count: u32,    // Will be copied
    created_at: u64,    // Will be copied
}

fn update_document_safely(doc: Document) -> Document {
    // Safe: we're consuming the original document
    Document {
        title: format!("Updated: {}", doc.title),
        ..doc  // Move content, copy numbers
    }
}
```

### 2. Use with Default for Configuration

Leverage the Default trait for clean configuration patterns:

```rust
#[derive(Debug, Default)]
struct ApiConfig {
    base_url: String,
    api_key: String,
    timeout: u64,
    retries: u32,
    rate_limit: u32,
}

fn create_config(api_key: String) -> ApiConfig {
    ApiConfig {
        base_url: String::from("https://api.example.com"),
        api_key,
        timeout: 30,
        ..Default::default()  // Use defaults for retries and rate_limit
    }
}
```

### 3. Document Ownership Transfer

Make ownership implications clear in documentation:

```rust
impl User {
    /// Creates a new user based on an existing user, updating the email.
    ///
    /// # Ownership
    /// This function takes ownership of `base_user` and returns a new User.
    /// The original `base_user` cannot be used after this call.
    pub fn with_new_email(base_user: User, new_email: String) -> User {
        User {
            email: new_email,
            ..base_user
        }
    }
}
```

### 4. Consider Clone When You Need the Original

```rust
fn update_preserving_original(user: &User) -> User {
    User {
        email: String::from("updated@example.com"),
        ..user.clone()  // Clone to preserve original
    }
}
```

## Summary and Key Points

### **Essential Characteristics**

**Struct update syntax** provides:
- **Concise creation** of struct variants with minimal field changes
- **Zero-cost abstraction** that compiles to explicit field assignment
- **Ownership transfer** of non-Copy fields from source to destination
- **Integration with Default trait** for elegant configuration patterns

### **Key Rules**

- The `..source` expression **must come last** in the struct literal[1]
- **Non-Copy fields are moved** from source to destination
- **Copy fields are copied** and source remains valid for those fields
- **All fields must be visible** at the update location[2]
- Source struct becomes **partially or fully unusable** after update

### **Best Use Cases**

```rust
// Configuration with defaults
let config = Config {
    host: "production.com".to_string(),
    ..Default::default()
};

// State transitions
let connected = Connection {
    state: Connected,
    ..disconnected_conn
};

// Template-based creation
let prod_config = Config {
    environment: "production".to_string(),
    ..dev_template
};
```

Struct update syntax is a powerful feature that makes Rust code more concise and expressive while maintaining the language's strict ownership guarantees. Understanding its ownership implications is crucial for using it effectively and avoiding common pitfalls related to moved values.

1. https://doc.rust-lang.org/book/ch05-01-defining-structs.html
2. https://doc.rust-lang.org/reference/expressions/struct-expr.html
3. https://www.linkedin.com/pulse/embracing-struct-update-syntax-rust-structs-default-traits-ali-%E6%9D%8E%E6%96%AF%E8%B3%A6
4. https://zerotomastery.io/blog/rust-struct-guide/
5. https://rust-lang.github.io/rfcs/2528-type-changing-struct-update-syntax.html
6. https://users.rust-lang.org/t/the-struct-update-syntax/16519
7. https://www.reddit.com/r/rust/comments/pchp8h/media_struct_update_syntax_in_rust/
8. https://users.rust-lang.org/t/why-the-struct-update-syntax-is-confusing/105305
9. https://github.com/rust-lang/book/issues/1996
10. https://www.rustfinity.com/learn/rust/structs/structs-and-ownership
11. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/structs.html
12. https://www.reddit.com/r/rust/comments/tf9xv4/question_about_struct_ownership/
13. https://dev.to/ajtech0001/mastering-structs-in-rust-from-basic-data-organization-to-method-implementation-11ol
14. https://shift.click/blog/non-exhaustive-fru-sad/
15. https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
16. https://stackoverflow.com/questions/73221584/how-to-use-struct-update-syntax-with-immutable-borrow
17. https://stackoverflow.com/questions/65381955/rust-struct-update-syntax-without-taking-ownership
18. https://educatedguesswork.org/posts/memory-management-4/
19. https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html
20. https://google.github.io/comprehensive-rust/std-traits/default.html
21. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch05-01-defining-structs.html
22. https://www.linkedin.com/learning/rust-essential-training/struct-update-syntax
