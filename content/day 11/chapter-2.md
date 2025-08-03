+++
title = "Field Access and Modification"
description = "Understand how to access and modify struct fields in Rust, including the use of mutable instances and dot notation."
date = 2025-08-03
draft = false
weight = 2
+++

# Field Access and Modification in Rust: Comprehensive Documentation

Field access and modification are fundamental operations when working with structs in Rust. Understanding how to safely and efficiently access and modify struct fields is crucial for writing effective Rust code, especially considering Rust's ownership and borrowing rules.

## Basic Field Access

### Dot Notation Syntax

The primary way to access struct fields is through **dot notation**, which provides direct access to individual fields by name:

```rust
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    let user = User {
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };

    // Basic field access
    println!("Username: {}", user.username);
    println!("Email: {}", user.email);
    println!("Age: {}", user.age);
    println!("Active: {}", user.active);

    // Fields can be used in expressions
    let display_name = &user.username;
    let is_adult = user.age >= 18;
    let contact_info = format!("{} ", user.username, user.email);

    println!("Display name: {}", display_name);
    println!("Is adult: {}", is_adult);
    println!("Contact: {}", contact_info);
}
```

### Field Access with References

When working with borrowed structs, field access works seamlessly through references[1]:

```rust
fn analyze_user(user: &User) {
    // Access fields through reference
    println!("Analyzing user: {}", user.username);
    println!("Email domain: {}",
             user.email.split('@').nth(1).unwrap_or("unknown"));

    // Fields can be borrowed independently
    let username_ref = &user.username;
    let age_ref = &user.age;

    println!("Username length: {}", username_ref.len());
    println!("Age in months: {}", age_ref * 12);
}

fn main() {
    let user = User {
        username: String::from("bob456"),
        email: String::from("bob@company.com"),
        age: 25,
        active: true,
    };

    // Pass reference to function
    analyze_user(&user);

    // Original struct still accessible
    println!("User still available: {}", user.username);
}
```

### Nested Field Access

Access fields in nested structures using chained dot notation:

```rust
#[derive(Debug)]
struct Address {
    street: String,
    city: String,
    country: String,
    postal_code: String,
}

#[derive(Debug)]
struct UserProfile {
    user: User,
    address: Address,
    preferences: Vec,
}

fn main() {
    let profile = UserProfile {
        user: User {
            username: String::from("charlie789"),
            email: String::from("charlie@example.com"),
            age: 28,
            active: true,
        },
        address: Address {
            street: String::from("123 Main St"),
            city: String::from("Surat"),
            country: String::from("India"),
            postal_code: String::from("395001"),
        },
        preferences: vec!["rust".to_string(), "programming".to_string()],
    };

    // Nested field access
    println!("User's name: {}", profile.user.username);
    println!("User's city: {}", profile.address.city);
    println!("User's age: {}", profile.user.age);

    // Access fields in collections
    if let Some(first_pref) = profile.preferences.first() {
        println!("First preference: {}", first_pref);
    }

    // Complex nested access
    let full_location = format!("{}, {}, {}",
                               profile.address.city,
                               profile.address.country,
                               profile.address.postal_code);
    println!("Location: {}", full_location);
}
```

## Field Modification

### Mutability Requirements

**Critical Rule:** To modify struct fields, the entire struct instance must be declared as mutable[1]:

```rust
fn main() {
    // Immutable struct - cannot modify fields
    let user = User {
        username: String::from("alice123"),
        email: String::from("alice@example.com"),
        age: 30,
        active: true,
    };

    // This would cause a compile error:
    // user.age = 31;  // Error: cannot assign to immutable field

    // Mutable struct - can modify fields
    let mut user_mut = User {
        username: String::from("bob456"),
        email: String::from("bob@example.com"),
        age: 25,
        active: true,
    };

    // Field modification works
    user_mut.age = 26;
    user_mut.email = String::from("bob.updated@example.com");
    user_mut.active = false;

    println!("Updated user: age={}, email={}, active={}",
             user_mut.age, user_mut.email, user_mut.active);
}
```

### Modifying Through Mutable References

When working with mutable references, field modification follows the same pattern:

```rust
fn update_user_info(user: &mut User) {
    // Modify fields through mutable reference
    user.age += 1;  // Birthday increment
    user.active = true;  // Reactivate account

    // Modify String fields
    if user.email.ends_with("@old-domain.com") {
        user.email = user.email.replace("@old-domain.com", "@new-domain.com");
    }

    // Add prefix to username
    user.username = format!("updated_{}", user.username);
}

fn main() {
    let mut user = User {
        username: String::from("testuser"),
        email: String::from("test@old-domain.com"),
        age: 30,
        active: false,
    };

    println!("Before update: {:?}", user);

    // Pass mutable reference to function
    update_user_info(&mut user);

    println!("After update: {:?}", user);
}
```

### Partial Field Updates

You can modify specific fields while leaving others unchanged:

```rust
fn main() {
    let mut config = ServerConfig {
        host: String::from("localhost"),
        port: 8080,
        max_connections: 100,
        timeout_seconds: 30,
        debug_mode: false,
    };

    println!("Initial config: {:?}", config);

    // Update specific fields for production
    config.host = String::from("production.server.com");
    config.port = 443;
    config.max_connections = 1000;
    config.debug_mode = false;
    // timeout_seconds remains unchanged

    println!("Production config: {:?}", config);

    // Conditional updates
    if config.debug_mode {
        config.timeout_seconds = 60;  // Longer timeout for debugging
    }
}

#[derive(Debug)]
struct ServerConfig {
    host: String,
    port: u16,
    max_connections: u32,
    timeout_seconds: u32,
    debug_mode: bool,
}
```

## Advanced Field Access Patterns

### Method-Based Field Access

Implement methods to provide controlled access to fields:

```rust
#[derive(Debug)]
struct BankAccount {
    account_number: String,
    balance: f64,
    is_active: bool,
}

impl BankAccount {
    fn new(account_number: String, initial_balance: f64) -> Self {
        BankAccount {
            account_number,
            balance: initial_balance,
            is_active: true,
        }
    }

    // Read-only access to sensitive fields
    fn get_account_number(&self) -> &str {
        &self.account_number
    }

    fn get_balance(&self) -> f64 {
        self.balance
    }

    fn is_account_active(&self) -> bool {
        self.is_active
    }

    // Controlled modification methods
    fn deposit(&mut self, amount: f64) -> Result {
        if !self.is_active {
            return Err("Account is not active".to_string());
        }

        if amount  Result {
        if !self.is_active {
            return Err("Account is not active".to_string());
        }

        if amount  println!("Deposit successful"),
        Err(e) => println!("Deposit failed: {}", e),
    }

    match account.withdraw(100.0) {
        Ok(()) => println!("Withdrawal successful"),
        Err(e) => println!("Withdrawal failed: {}", e),
    }

    println!("Final balance: ${:.2}", account.get_balance());
}
```

### Field Access with Validation

Implement getters and setters with validation logic:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
    email: String,
}

impl Person {
    fn new(name: String, age: u8, email: String) -> Result {
        let person = Person { name, age, email };
        person.validate()?;
        Ok(person)
    }

    // Validated setters
    fn set_name(&mut self, name: String) -> Result {
        if name.trim().is_empty() {
            return Err("Name cannot be empty".to_string());
        }
        if name.len() > 100 {
            return Err("Name too long".to_string());
        }
        self.name = name;
        Ok(())
    }

    fn set_age(&mut self, age: u8) -> Result {
        if age > 120 {
            return Err("Age must be realistic".to_string());
        }
        self.age = age;
        Ok(())
    }

    fn set_email(&mut self, email: String) -> Result {
        if !email.contains('@') {
            return Err("Invalid email format".to_string());
        }
        self.email = email;
        Ok(())
    }

    // Getters
    fn get_name(&self) -> &str {
        &self.name
    }

    fn get_age(&self) -> u8 {
        self.age
    }

    fn get_email(&self) -> &str {
        &self.email
    }

    fn validate(&self) -> Result {
        if self.name.trim().is_empty() {
            return Err("Name cannot be empty".to_string());
        }
        if !self.email.contains('@') {
            return Err("Invalid email format".to_string());
        }
        if self.age > 120 {
            return Err("Age must be realistic".to_string());
        }
        Ok(())
    }
}

fn main() {
    let mut person = Person::new(
        "Alice".to_string(),
        30,
        "alice@example.com".to_string(),
    ).expect("Failed to create person");

    println!("Created person: {:?}", person);

    // Validated updates
    if let Err(e) = person.set_age(150) {
        println!("Age update failed: {}", e);
    }

    if let Ok(()) = person.set_name("Alice Smith".to_string()) {
        println!("Name updated successfully");
    }

    println!("Updated person: {:?}", person);
}
```

## Pattern Matching and Destructuring

### Basic Struct Destructuring

Extract field values using pattern matching:

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

#[derive(Debug)]
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

fn analyze_point_and_color(point: Point, color: Color) {
    // Destructure all fields
    let Point { x, y } = point;
    println!("Point coordinates: ({}, {})", x, y);

    // Destructure with field renaming
    let Color { r: red, g: green, b: blue } = color;
    println!("RGB values: ({}, {}, {})", red, green, blue);

    // Use destructured values
    let distance_from_origin = ((x * x + y * y) as f64).sqrt();
    let brightness = (red as f64 + green as f64 + blue as f64) / 3.0;

    println!("Distance from origin: {:.2}", distance_from_origin);
    println!("Average brightness: {:.2}", brightness);
}

fn main() {
    let point = Point { x: 3, y: 4 };
    let color = Color { r: 255, g: 128, b: 0 };

    analyze_point_and_color(point, color);
}
```

### Partial Destructuring

Extract only needed fields while ignoring others:

```rust
#[derive(Debug)]
struct UserSession {
    user_id: u64,
    username: String,
    session_token: String,
    login_time: String,
    last_activity: String,
    ip_address: String,
}

fn log_user_activity(session: &UserSession) {
    // Extract only needed fields
    let UserSession { user_id, username, last_activity, .. } = session;

    println!("User {} (ID: {}) last active at: {}",
             username, user_id, last_activity);
}

fn validate_session(session: &UserSession) -> bool {
    // Pattern match for specific validation
    match session {
        UserSession { session_token, .. } if session_token.is_empty() => {
            println!("Invalid session: empty token");
            false
        }
        UserSession { user_id: 0, .. } => {
            println!("Invalid session: zero user ID");
            false
        }
        UserSession { username, .. } if username.trim().is_empty() => {
            println!("Invalid session: empty username");
            false
        }
        _ => {
            println!("Session is valid");
            true
        }
    }
}

fn main() {
    let session = UserSession {
        user_id: 12345,
        username: String::from("alice123"),
        session_token: String::from("abc123xyz789"),
        login_time: String::from("2024-01-01 10:00:00"),
        last_activity: String::from("2024-01-01 14:30:00"),
        ip_address: String::from("192.168.1.100"),
    };

    log_user_activity(&session);
    validate_session(&session);
}
```

### Match Expressions with Structs

Use pattern matching for conditional field access:

```rust
#[derive(Debug)]
enum Status {
    Active,
    Inactive,
    Suspended,
}

#[derive(Debug)]
struct Account {
    id: u64,
    name: String,
    balance: f64,
    status: Status,
}

fn process_account(account: &Account) {
    match account {
        // Match specific field values
        Account { status: Status::Suspended, name, .. } => {
            println!("Account '{}' is suspended - cannot process", name);
        }

        // Match with conditions
        Account { balance, name, .. } if *balance  {
            println!("Account '{}' has negative balance: ${:.2}", name, balance);
        }

        // Match active accounts with sufficient balance
        Account {
            status: Status::Active,
            balance,
            name,
            ..
        } if *balance >= 100.0 => {
            println!("Processing active account '{}' with balance ${:.2}", name, balance);
        }

        // Default case
        Account { name, status, balance, .. } => {
            println!("Account '{}' ({:?}) - Balance: ${:.2}", name, status, balance);
        }
    }
}

fn main() {
    let accounts = vec![
        Account {
            id: 1,
            name: String::from("Alice"),
            balance: 1500.0,
            status: Status::Active,
        },
        Account {
            id: 2,
            name: String::from("Bob"),
            balance: -50.0,
            status: Status::Active,
        },
        Account {
            id: 3,
            name: String::from("Charlie"),
            balance: 750.0,
            status: Status::Suspended,
        },
    ];

    for account in &accounts {
        process_account(account);
    }
}
```

## Ownership and Borrowing in Field Access

### Field-Level Borrowing

Rust allows borrowing individual fields independently:

```rust
#[derive(Debug)]
struct Document {
    title: String,
    content: String,
    metadata: Vec,
    version: u32,
}

fn process_document_parts(doc: &mut Document) {
    // Borrow different fields independently
    let title_ref = &doc.title;
    let content_ref = &mut doc.content;
    let version_ref = &doc.version;

    // Can use different borrowed fields simultaneously
    println!("Processing document '{}' (version {})", title_ref, version_ref);

    // Modify content while reading title and version
    content_ref.push_str("\n\n--- Updated ---");

    println!("Content length after update: {}", content_ref.len());

    // title_ref and version_ref are still valid here
    println!("Title: {}", title_ref);
}

fn separate_field_operations(doc: &mut Document) {
    // Borrow metadata mutably
    {
        let metadata = &mut doc.metadata;
        metadata.push("last_modified".to_string());
        metadata.push("auto_saved".to_string());
    } // metadata borrow ends here

    // Now we can borrow other fields
    let title = &doc.title;
    let version = &doc.version;

    println!("Document '{}' v{} has {} metadata entries",
             title, version, doc.metadata.len());
}

fn main() {
    let mut doc = Document {
        title: String::from("Rust Guide"),
        content: String::from("This is the content of the document."),
        metadata: vec!["created".to_string()],
        version: 1,
    };

    process_document_parts(&mut doc);
    separate_field_operations(&mut doc);

    println!("Final document: {:#?}", doc);
}
```

### Moving vs Borrowing Fields

Understanding when fields are moved vs borrowed:

```rust
#[derive(Debug)]
struct Container {
    name: String,
    data: Vec,
    flag: bool,
}

fn demonstrate_field_ownership() {
    let mut container = Container {
        name: String::from("data_container"),
        data: vec![1, 2, 3, 4, 5],
        flag: true,
    };

    // Borrowing fields (no move)
    let name_ref = &container.name;
    let data_ref = &container.data;
    let flag_copy = container.flag;  // Copy type - no move

    println!("Name: {}", name_ref);
    println!("Data length: {}", data_ref.len());
    println!("Flag: {}", flag_copy);

    // container is still fully accessible
    println!("Full container: {:?}", container);

    // Partial move (problematic)
    // let moved_name = container.name;  // Would move name field
    // println!("{:?}", container);      // Error: container partially moved

    // Safe approach: clone or use references
    let name_clone = container.name.clone();
    println!("Cloned name: {}", name_clone);

    // container is still fully accessible after cloning
    container.flag = false;
    println!("Updated container: {:?}", container);
}

fn extract_fields_safely(container: Container) -> (String, Vec, bool) {
    // Destructure to extract all fields (moves container)
    let Container { name, data, flag } = container;
    (name, data, flag)
}

fn main() {
    demonstrate_field_ownership();

    let container = Container {
        name: String::from("extracted_container"),
        data: vec![10, 20, 30],
        flag: true,
    };

    let (name, data, flag) = extract_fields_safely(container);
    // container is no longer accessible after move

    println!("Extracted - Name: {}, Data: {:?}, Flag: {}", name, data, flag);
}
```

## Error Handling and Field Access

### Safe Field Access with Option Types

Handle cases where fields might be optional or inaccessible:

```rust
#[derive(Debug)]
struct UserProfile {
    username: String,
    email: Option,
    phone: Option,
    address: Option,
}

#[derive(Debug)]
struct Address {
    street: String,
    city: String,
    country: String,
}

impl UserProfile {
    fn get_contact_info(&self) -> String {
        let mut contact_parts = Vec::new();

        // Always available
        contact_parts.push(format!("Username: {}", self.username));

        // Handle optional email
        if let Some(ref email) = self.email {
            contact_parts.push(format!("Email: {}", email));
        } else {
            contact_parts.push("Email: Not provided".to_string());
        }

        // Handle optional phone
        match &self.phone {
            Some(phone) => contact_parts.push(format!("Phone: {}", phone)),
            None => contact_parts.push("Phone: Not provided".to_string()),
        }

        // Handle nested optional fields
        if let Some(ref address) = self.address {
            contact_parts.push(format!("Address: {}, {}, {}",
                                     address.street, address.city, address.country));
        } else {
            contact_parts.push("Address: Not provided".to_string());
        }

        contact_parts.join("\n")
    }

    fn update_email(&mut self, email: String) {
        self.email = Some(email);
    }

    fn clear_email(&mut self) {
        self.email = None;
    }

    fn has_complete_profile(&self) -> bool {
        self.email.is_some() && self.phone.is_some() && self.address.is_some()
    }
}

fn main() {
    let mut profile = UserProfile {
        username: String::from("alice123"),
        email: Some(String::from("alice@example.com")),
        phone: None,
        address: Some(Address {
            street: String::from("123 Main St"),
            city: String::from("Surat"),
            country: String::from("India"),
        }),
    };

    println!("Profile info:\n{}", profile.get_contact_info());
    println!("Complete profile: {}", profile.has_complete_profile());

    // Update optional fields
    profile.phone = Some(String::from("+91-9876543210"));
    println!("\nAfter adding phone:");
    println!("Complete profile: {}", profile.has_complete_profile());

    // Clear optional field
    profile.clear_email();
    println!("\nAfter clearing email:");
    println!("Profile info:\n{}", profile.get_contact_info());
}
```

### Result-Based Field Validation

Implement field access with validation using Result types:

```rust
#[derive(Debug)]
struct ConfigurationManager {
    settings: std::collections::HashMap,
}

#[derive(Debug)]
enum ConfigError {
    KeyNotFound(String),
    InvalidValue(String),
    ParseError(String),
}

impl std::fmt::Display for ConfigError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            ConfigError::KeyNotFound(key) => write!(f, "Configuration key '{}' not found", key),
            ConfigError::InvalidValue(msg) => write!(f, "Invalid configuration value: {}", msg),
            ConfigError::ParseError(msg) => write!(f, "Failed to parse configuration: {}", msg),
        }
    }
}

impl std::error::Error for ConfigError {}

impl ConfigurationManager {
    fn new() -> Self {
        ConfigurationManager {
            settings: std::collections::HashMap::new(),
        }
    }

    fn set(&mut self, key: &str, value: &str) {
        self.settings.insert(key.to_string(), value.to_string());
    }

    fn get_string(&self, key: &str) -> Result {
        self.settings.get(key)
            .ok_or_else(|| ConfigError::KeyNotFound(key.to_string()))
    }

    fn get_int(&self, key: &str) -> Result {
        let value = self.get_string(key)?;
        value.parse::()
            .map_err(|_| ConfigError::ParseError(
                format!("Cannot parse '{}' as integer", value)
            ))
    }

    fn get_bool(&self, key: &str) -> Result {
        let value = self.get_string(key)?;
        match value.to_lowercase().as_str() {
            "true" | "yes" | "1" => Ok(true),
            "false" | "no" | "0" => Ok(false),
            _ => Err(ConfigError::ParseError(
                format!("Cannot parse '{}' as boolean", value)
            )),
        }
    }

    fn get_port(&self, key: &str) -> Result {
        let port = self.get_int(key)?;
        if port  65535 {
            return Err(ConfigError::InvalidValue(
                format!("Port {} is out of valid range (1-65535)", port)
            ));
        }
        Ok(port as u16)
    }
}

fn main() {
    let mut config = ConfigurationManager::new();

    // Set configuration values
    config.set("host", "localhost");
    config.set("port", "8080");
    config.set("debug", "true");
    config.set("max_connections", "100");
    config.set("timeout", "not_a_number");  // Invalid value

    // Access with error handling
    match config.get_string("host") {
        Ok(host) => println!("Host: {}", host),
        Err(e) => println!("Error getting host: {}", e),
    }

    match config.get_port("port") {
        Ok(port) => println!("Port: {}", port),
        Err(e) => println!("Error getting port: {}", e),
    }

    match config.get_bool("debug") {
        Ok(debug) => println!("Debug mode: {}", debug),
        Err(e) => println!("Error getting debug: {}", e),
    }

    // This will fail due to invalid value
    match config.get_int("timeout") {
        Ok(timeout) => println!("Timeout: {}", timeout),
        Err(e) => println!("Error getting timeout: {}", e),
    }

    // This will fail due to missing key
    match config.get_string("missing_key") {
        Ok(value) => println!("Missing key: {}", value),
        Err(e) => println!("Error: {}", e),
    }
}
```

## Performance Considerations

### Memory Layout and Field Access

Understanding how field ordering affects performance:

```rust
use std::mem;

// Optimized struct layout (fields ordered by size)
#[repr(C)]
struct OptimizedStruct {
    large_field: u64,      // 8 bytes
    medium_field: u32,     // 4 bytes
    small_field: u16,      // 2 bytes
    tiny_field: u8,        // 1 byte
    boolean: bool,         // 1 byte
}

// Unoptimized struct layout (random field ordering)
struct UnoptimizedStruct {
    tiny_field: u8,        // 1 byte + 7 bytes padding
    large_field: u64,      // 8 bytes
    boolean: bool,         // 1 byte + 1 byte padding
    small_field: u16,      // 2 bytes + 4 bytes padding
    medium_field: u32,     // 4 bytes
}

fn demonstrate_memory_layout() {
    println!("Optimized struct size: {} bytes", mem::size_of::());
    println!("Unoptimized struct size: {} bytes", mem::size_of::());

    let optimized = OptimizedStruct {
        large_field: 12345678901234,
        medium_field: 42,
        small_field: 100,
        tiny_field: 5,
        boolean: true,
    };

    // Field access has the same performance regardless of layout
    // but memory usage differs significantly
    println!("Accessing optimized fields: {}, {}, {}",
             optimized.large_field, optimized.medium_field, optimized.boolean);
}

fn main() {
    demonstrate_memory_layout();
}
```

### Zero-Cost Field Access

Rust's field access is a zero-cost abstraction - it compiles to direct memory access:

```rust
#[derive(Debug)]
struct Point3D {
    x: f64,
    y: f64,
    z: f64,
}

impl Point3D {
    // This compiles to direct memory access - no function call overhead
    fn distance_squared(&self) -> f64 {
        self.x * self.x + self.y * self.y + self.z * self.z
    }

    // Field modification also has zero overhead
    fn scale(&mut self, factor: f64) {
        self.x *= factor;
        self.y *= factor;
        self.z *= factor;
    }

    // Complex field operations are optimized by the compiler
    fn normalize(&mut self) {
        let length = self.distance_squared().sqrt();
        if length > 0.0 {
            self.scale(1.0 / length);
        }
    }
}

fn demonstrate_zero_cost_access() {
    let mut point = Point3D {
        x: 3.0,
        y: 4.0,
        z: 5.0,
    };

    println!("Original point: {:?}", point);
    println!("Distance squared: {}", point.distance_squared());

    point.scale(2.0);
    println!("Scaled point: {:?}", point);

    point.normalize();
    println!("Normalized point: {:?}", point);
}

fn main() {
    demonstrate_zero_cost_access();
}
```

## Best Practices for Field Access and Modification

### Encapsulation and Data Integrity

Design APIs that maintain data consistency:

```rust
#[derive(Debug)]
pub struct Temperature {
    celsius: f64,
}

impl Temperature {
    // Constructor with validation
    pub fn from_celsius(celsius: f64) -> Result {
        if celsius  Result {
        let celsius = (fahrenheit - 32.0) * 5.0 / 9.0;
        Self::from_celsius(celsius)
    }

    pub fn from_kelvin(kelvin: f64) -> Result {
        if kelvin  f64 {
        self.celsius
    }

    pub fn fahrenheit(&self) -> f64 {
        self.celsius * 9.0 / 5.0 + 32.0
    }

    pub fn kelvin(&self) -> f64 {
        self.celsius + 273.15
    }

    // Controlled modification
    pub fn set_celsius(&mut self, celsius: f64) -> Result {
        if celsius  Result {
        self.set_celsius(self.celsius + delta_celsius)
    }
}

fn main() {
    // Safe construction
    let mut temp = Temperature::from_fahrenheit(68.0)
        .expect("Valid temperature");

    println!("Temperature: {:.1}°C, {:.1}°F, {:.1}K",
             temp.celsius(), temp.fahrenheit(), temp.kelvin());

    // Safe modification
    match temp.adjust(10.0) {
        Ok(()) => println!("Temperature adjusted successfully"),
        Err(e) => println!("Error: {}", e),
    }

    println!("New temperature: {:.1}°C", temp.celsius());

    // This would fail validation
    match temp.set_celsius(-300.0) {
        Ok(()) => println!("Temperature set successfully"),
        Err(e) => println!("Error: {}", e),
    }
}
```

### Builder Pattern for Complex Field Management

Use builder pattern for structs with many optional fields:

```rust
#[derive(Debug, Clone)]
pub struct HttpRequest {
    method: String,
    url: String,
    headers: std::collections::HashMap,
    body: Option,
    timeout: Option,
    follow_redirects: bool,
}

pub struct HttpRequestBuilder {
    request: HttpRequest,
}

impl HttpRequestBuilder {
    pub fn new(method: &str, url: &str) -> Self {
        HttpRequestBuilder {
            request: HttpRequest {
                method: method.to_string(),
                url: url.to_string(),
                headers: std::collections::HashMap::new(),
                body: None,
                timeout: None,
                follow_redirects: true,
            },
        }
    }

    pub fn header(mut self, key: &str, value: &str) -> Self {
        self.request.headers.insert(key.to_string(), value.to_string());
        self
    }

    pub fn body(mut self, body: String) -> Self {
        self.request.body = Some(body);
        self
    }

    pub fn timeout(mut self, seconds: u64) -> Self {
        self.request.timeout = Some(seconds);
        self
    }

    pub fn follow_redirects(mut self, follow: bool) -> Self {
        self.request.follow_redirects = follow;
        self
    }

    pub fn build(self) -> HttpRequest {
        self.request
    }
}

impl HttpRequest {
    pub fn builder(method: &str, url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new(method, url)
    }

    // Access methods
    pub fn method(&self) -> &str {
        &self.method
    }

    pub fn url(&self) -> &str {
        &self.url
    }

    pub fn headers(&self) -> &std::collections::HashMap {
        &self.headers
    }

    pub fn body(&self) -> Option {
        self.body.as_deref()
    }

    pub fn timeout(&self) -> Option {
        self.timeout
    }

    pub fn follows_redirects(&self) -> bool {
        self.follow_redirects
    }
}

fn main() {
    let request = HttpRequest::builder("POST", "https://api.example.com/users")
        .header("Content-Type", "application/json")
        .header("Authorization", "Bearer token123")
        .body(r#"{"name": "Alice", "email": "alice@example.com"}"#.to_string())
        .timeout(30)
        .follow_redirects(false)
        .build();

    println!("Request: {:#?}", request);

    // Access fields through methods
    println!("Method: {}", request.method());
    println!("URL: {}", request.url());

    if let Some(body) = request.body() {
        println!("Body length: {}", body.len());
    }
}
```

## Summary and Key Takeaways

### **Essential Concepts**

**Field access and modification** in Rust involves:
- **Dot notation** for accessing struct fields by name
- **Mutability requirements** for field modification
- **Ownership and borrowing rules** that apply to field-level operations
- **Pattern matching** for elegant field extraction and processing

### **Key Rules and Patterns**

```rust
// Basic field access
let value = struct_instance.field_name;

// Field modification requires mut
let mut instance = StructType { /* ... */ };
instance.field_name = new_value;

// Independent field borrowing
let field1_ref = &instance.field1;
let field2_mut = &mut instance.field2;

// Pattern destructuring
let StructType { field1, field2, .. } = instance;
```

### **Best Practices**

- **Use encapsulation** to control field access and maintain data integrity
- **Implement validation** in setter methods to ensure consistent state
- **Prefer references** over moves when possible to avoid ownership issues
- **Design clear APIs** that make field access patterns obvious
- **Consider performance** implications of field ordering and access patterns
- **Use pattern matching** for elegant field extraction and conditional logic

### **Advanced Techniques**

- **Method-based access** for controlled field operations
- **Builder patterns** for complex struct construction
- **Result-based validation** for error handling in field operations
- **Option types** for handling optional or nullable fields
- **Zero-cost abstractions** that compile to efficient memory access

**Field access and modification are fundamental operations in Rust that balance safety, performance, and expressiveness**. Understanding these concepts thoroughly enables you to write robust, efficient code that leverages Rust's ownership system while maintaining clear, maintainable APIs for struct operations.

1. https://doc.rust-lang.org/book/ch05-01-defining-structs.html
2. https://doc.rust-lang.org/reference/visibility-and-privacy.html
3. https://crates.io/crates/field_access
4. https://users.rust-lang.org/t/share-access-to-a-field-in-the-outer-struct/76053
5. https://notes.kodekloud.com/docs/Rust-Programming/Collections-Error-Handling/Structs
6. https://stackoverflow.com/questions/75780784/why-mut-self-in-method-when-modifying-an-object-as-field-in-a-struct
7. https://docs.rs/fieldx
8. https://www.reddit.com/r/rust/comments/p1jfgu/how_to_use_object_within_structs_field_vec_as_the/
9. https://internals.rust-lang.org/t/fields-in-traits/6933
10. https://crates.io/crates/fieldname-access
