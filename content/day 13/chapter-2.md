+++
title = "Debug Trait and Custom Formatting"
description = "Learn how to derive the Debug trait for your structs and implement custom formatting using the fmt::Debug trait in Rust."
date = 2025-08-04
draft = false
weight = 2
+++

# Debug Trait and Custom Formatting in Rust: Comprehensive Documentation for Beginners

Understanding the Debug trait is like learning how to create **X-ray vision for your data** - it lets you see inside your structs and understand what's happening when your code runs. Think of it as creating a special "developer view" of your data that shows all the internal details in a readable way.

## Understanding the Debug Trait - The X-ray Vision Analogy

### The Medical Scanner Metaphor

**Imagine you're a doctor with different types of medical scanners:**
- 📱 **Display trait** = Taking a nice portrait photo (user-friendly, polished)
- 🔬 **Debug trait** = X-ray machine (shows internal structure, technical details)
- 🖨️ **Print macros** = Different printing modes on your scanner

```rust
// Think of this like a patient file
#[derive(Debug)]  // 🔬 Enable X-ray vision
struct Patient {
    name: String,
    age: u32,
    heart_rate: u32,
    temperature: f64,
}

fn main() {
    let patient = Patient {
        name: "Alice".to_string(),
        age: 30,
        heart_rate: 72,
        temperature: 98.6,
    };

    // X-ray view (Debug) - shows all internal details
    println!("{:?}", patient);
    // Output: Patient { name: "Alice", age: 30, heart_rate: 72, temperature: 98.6 }

    // High-resolution X-ray (Pretty Debug) - easier to read
    println!("{:#?}", patient);
    // Output:
    // Patient {
    //     name: "Alice",
    //     age: 30,
    //     heart_rate: 72,
    //     temperature: 98.6,
    // }
}
```

**Key Point:** Debug gives you the "technical view" - perfect for developers to understand what's inside their data structures!

## What is the Debug Trait?

### Core Concept

The **Debug trait** allows types to provide a programmer-facing, debugging representation. It's Rust's way of saying "here's how to show the internals of this data structure to a developer."

- **Purpose**: Show the internal state of data structures
- **Audience**: Developers (not end users)
- **Usage**: Debugging, logging, development tools
- **Format specifiers**: `{:?}` (compact) and `{:#?}` (pretty-printed)

### How Debug Works Under the Hood

```rust
// This is what the Debug trait looks like internally
pub trait Debug {
    fn fmt(&self, f: &mut Formatter) -> Result;
}

// When you write println!("{:?}", my_struct)
// Rust calls my_struct.fmt(formatter) behind the scenes
```

## Basic Debug Implementation

### Automatic Implementation with #[derive(Debug)]

The easiest way to implement Debug is using the `derive` macro:

```rust
#[derive(Debug)]
struct Smartphone {
    brand: String,
    model: String,
    battery_percentage: u8,
    is_connected: bool,
    apps: Vec,
}

fn main() {
    let phone = Smartphone {
        brand: "iPhone".to_string(),
        model: "14 Pro".to_string(),
        battery_percentage: 85,
        is_connected: true,
        apps: vec![
            "Messages".to_string(),
            "Camera".to_string(),
            "Settings".to_string(),
        ],
    };

    // Compact debug format
    println!("Compact: {:?}", phone);

    // Pretty debug format with indentation
    println!("Pretty:\n{:#?}", phone);
}
```

**Output:**
```
Compact: Smartphone { brand: "iPhone", model: "14 Pro", battery_percentage: 85, is_connected: true, apps: ["Messages", "Camera", "Settings"] }

Pretty:
Smartphone {
    brand: "iPhone",
    model: "14 Pro",
    battery_percentage: 85,
    is_connected: true,
    apps: [
        "Messages",
        "Camera",
        "Settings",
    ],
}
```

### When Derive Works vs When It Doesn't

```rust
// ✅ This works - all fields implement Debug
#[derive(Debug)]
struct WorkingStruct {
    number: i32,          // primitives implement Debug
    text: String,         // String implements Debug
    list: Vec,       // Vec implements Debug if T does
}

// ❌ This won't work if you have non-Debug fields
struct CustomType {
    value: i32,
}
// CustomType doesn't implement Debug

struct ProblematicStruct {
    working_field: i32,
    // problem_field: CustomType,  // This would prevent derive(Debug)
}

// Solution: Implement Debug for CustomType first
use std::fmt;

impl fmt::Debug for CustomType {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "CustomType({})", self.value)
    }
}

// Now this works
#[derive(Debug)]
struct FixedStruct {
    working_field: i32,
    custom_field: CustomType,
}
```

## Custom Debug Implementation

### Basic Manual Implementation

When you need control over the debug output:

```rust
use std::fmt;

struct BankAccount {
    account_number: String,
    balance: f64,
    pin: String,  // Sensitive data we don't want to show
}

impl fmt::Debug for BankAccount {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f,
               "BankAccount {{ account_number: \"{}\", balance: ${:.2}, pin: \"[HIDDEN]\" }}",
               self.account_number,
               self.balance)
    }
}

fn main() {
    let account = BankAccount {
        account_number: "ACC123456".to_string(),
        balance: 1250.75,
        pin: "1234".to_string(),  // This will be hidden in debug output
    };

    println!("{:?}", account);
    // Output: BankAccount { account_number: "ACC123456", balance: $1250.75, pin: "[HIDDEN]" }
}
```

### Using Debug Helpers for Better Formatting

Rust provides helper methods to make custom Debug implementations easier:

```rust
use std::fmt;

struct GameCharacter {
    name: String,
    level: u32,
    health: u32,
    max_health: u32,
    inventory: Vec,
    secret_stats: u32,  // We want to hide this
}

impl fmt::Debug for GameCharacter {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("GameCharacter")
            .field("name", &self.name)
            .field("level", &self.level)
            .field("health", &format_args!("{}/{}", self.health, self.max_health))
            .field("inventory", &self.inventory)
            .field("secret_stats", &"[CLASSIFIED]")
            .finish()
    }
}

fn main() {
    let character = GameCharacter {
        name: "DragonSlayer99".to_string(),
        level: 42,
        health: 180,
        max_health: 200,
        inventory: vec!["Sword".to_string(), "Shield".to_string(), "Potion".to_string()],
        secret_stats: 9999,
    };

    println!("{:#?}", character);
}
```

**Output:**
```
GameCharacter {
    name: "DragonSlayer99",
    level: 42,
    health: 180/200,
    inventory: [
        "Sword",
        "Shield",
        "Potion",
    ],
    secret_stats: "[CLASSIFIED]",
}
```

### Advanced Custom Formatting

```rust
use std::fmt;
use std::collections::HashMap;

struct ServerStatus {
    server_name: String,
    uptime_seconds: u64,
    active_connections: u32,
    memory_usage_mb: u32,
    errors: Vec,
    config: HashMap,
}

impl fmt::Debug for ServerStatus {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // Convert uptime to human readable format
        let hours = self.uptime_seconds / 3600;
        let minutes = (self.uptime_seconds % 3600) / 60;
        let seconds = self.uptime_seconds % 60;

        // Color code health based on errors
        let health_status = if self.errors.is_empty() {
            "🟢 HEALTHY"
        } else if self.errors.len() >())
            .finish()
    }
}

fn main() {
    let mut config = HashMap::new();
    config.insert("port".to_string(), "8080".to_string());
    config.insert("max_connections".to_string(), "1000".to_string());

    let server = ServerStatus {
        server_name: "WebServer-01".to_string(),
        uptime_seconds: 7265,  // 2 hours, 1 minute, 5 seconds
        active_connections: 234,
        memory_usage_mb: 512,
        errors: vec!["Connection timeout".to_string()],
        config,
    };

    println!("{:#?}", server);
}
```

## Practical Real-World Examples

### HTTP Request Logger

```rust
use std::fmt;
use std::collections::HashMap;

struct HttpRequest {
    method: String,
    url: String,
    headers: HashMap,
    body: Option,
    timestamp: u64,
}

impl fmt::Debug for HttpRequest {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // Determine if we should use compact or pretty format
        if f.alternate() {
            // Pretty format for {:#?}
            f.debug_struct("HttpRequest")
                .field("method", &self.method)
                .field("url", &self.url)
                .field("timestamp", &self.timestamp)
                .field("headers", &self.format_headers())
                .field("body", &self.format_body())
                .finish()
        } else {
            // Compact format for {:?}
            write!(f, "{} {} [{}] ({} headers)",
                   self.method,
                   self.url,
                   self.timestamp,
                   self.headers.len())
        }
    }
}

impl HttpRequest {
    fn format_headers(&self) -> String {
        if self.headers.is_empty() {
            "None".to_string()
        } else {
            format!("{} headers", self.headers.len())
        }
    }

    fn format_body(&self) -> String {
        match &self.body {
            None => "No body".to_string(),
            Some(body) if body.len() > 50 => format!("{}... ({} chars)", &body[..47], body.len()),
            Some(body) => body.clone(),
        }
    }
}

fn main() {
    let mut headers = HashMap::new();
    headers.insert("Content-Type".to_string(), "application/json".to_string());
    headers.insert("Authorization".to_string(), "Bearer token123".to_string());

    let request = HttpRequest {
        method: "POST".to_string(),
        url: "/api/users".to_string(),
        headers,
        body: Some(r#"{"name": "Alice", "email": "alice@example.com"}"#.to_string()),
        timestamp: 1640995200,
    };

    // Compact format
    println!("Compact: {:?}", request);

    // Pretty format
    println!("Pretty:\n{:#?}", request);
}
```

### Database Connection Pool Monitor

```rust
use std::fmt;
use std::time::{Duration, SystemTime};

#[derive(Clone)]
struct Connection {
    id: u32,
    created_at: SystemTime,
    last_used: SystemTime,
    is_active: bool,
}

struct ConnectionPool {
    connections: Vec,
    max_size: usize,
    active_count: usize,
}

impl fmt::Debug for Connection {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let age = self.created_at.elapsed().unwrap_or(Duration::from_secs(0));
        let idle_time = self.last_used.elapsed().unwrap_or(Duration::from_secs(0));

        f.debug_struct("Connection")
            .field("id", &self.id)
            .field("status", &if self.is_active { "🟢 ACTIVE" } else { "⚪ IDLE" })
            .field("age", &format!("{:.1}s", age.as_secs_f64()))
            .field("idle_for", &format!("{:.1}s", idle_time.as_secs_f64()))
            .finish()
    }
}

impl fmt::Debug for ConnectionPool {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let utilization = (self.active_count as f64 / self.max_size as f64) * 100.0;

        let health_indicator = match utilization {
            x if x  "🟢 HEALTHY",
            x if x  "🟡 BUSY",
            _ => "🔴 OVERLOADED",
        };

        f.debug_struct("ConnectionPool")
            .field("health", &health_indicator)
            .field("utilization", &format!("{:.1}%", utilization))
            .field("active", &self.active_count)
            .field("total", &self.connections.len())
            .field("max_size", &self.max_size)
            .field("connections", &if f.alternate() {
                &self.connections
            } else {
                &self.connections[..self.active_count.min(3)] // Show first 3 in compact mode
            })
            .finish()
    }
}

fn main() {
    let now = SystemTime::now();
    let connections = vec![
        Connection { id: 1, created_at: now, last_used: now, is_active: true },
        Connection { id: 2, created_at: now, last_used: now, is_active: true },
        Connection { id: 3, created_at: now, last_used: now, is_active: false },
    ];

    let pool = ConnectionPool {
        connections,
        max_size: 10,
        active_count: 2,
    };

    println!("Pool status: {:?}", pool);
    println!("\nDetailed view:\n{:#?}", pool);
}
```

## Advanced Formatting Techniques

### Conditional Formatting Based on Debug Flags

```rust
use std::fmt;

struct ApiResponse {
    status_code: u16,
    data: T,
    headers: Vec,
    processing_time_ms: u64,
}

impl fmt::Debug for ApiResponse {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // Check formatting flags
        let show_headers = f.alternate(); // {:#?} shows headers, {:?} doesn't

        let mut debug_struct = f.debug_struct("ApiResponse");

        // Always show essential info
        debug_struct
            .field("status", &self.format_status())
            .field("processing_time", &format!("{}ms", self.processing_time_ms));

        // Conditionally show headers
        if show_headers {
            debug_struct.field("headers", &self.headers);
        } else {
            debug_struct.field("header_count", &self.headers.len());
        }

        // Always show data
        debug_struct.field("data", &self.data);

        debug_struct.finish()
    }
}

impl ApiResponse {
    fn format_status(&self) -> String {
        let emoji = match self.status_code {
            200..=299 => "✅",
            300..=399 => "↩️",
            400..=499 => "❌",
            500..=599 => "💥",
            _ => "❓",
        };

        format!("{} {}", emoji, self.status_code)
    }
}

fn main() {
    let response = ApiResponse {
        status_code: 200,
        data: vec!["user1".to_string(), "user2".to_string()],
        headers: vec![
            ("Content-Type".to_string(), "application/json".to_string()),
            ("Cache-Control".to_string(), "no-cache".to_string()),
        ],
        processing_time_ms: 125,
    };

    println!("Compact: {:?}", response);
    println!("Detailed:\n{:#?}", response);
}
```

### Debug for Complex Nested Structures

```rust
use std::fmt;
use std::collections::BTreeMap;

struct Application {
    name: String,
    version: String,
    modules: Vec,
    config: BTreeMap,
}

struct Module {
    name: String,
    enabled: bool,
    dependencies: Vec,
    error_count: u32,
}

#[derive(Debug)]
enum ConfigValue {
    String(String),
    Number(f64),
    Boolean(bool),
    List(Vec),
}

impl fmt::Debug for Module {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let status = if self.enabled {
            if self.error_count == 0 {
                "🟢 RUNNING"
            } else {
                "🟡 RUNNING_WITH_ERRORS"
            }
        } else {
            "⚪ DISABLED"
        };

        f.debug_struct("Module")
            .field("name", &self.name)
            .field("status", &status)
            .field("dependencies", &self.dependencies.len())
            .field("errors", &self.error_count)
            .finish()
    }
}

impl fmt::Debug for Application {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let enabled_modules = self.modules.iter().filter(|m| m.enabled).count();
        let total_errors: u32 = self.modules.iter().map(|m| m.error_count).sum();

        f.debug_struct("Application")
            .field("name", &format!("{} v{}", self.name, self.version))
            .field("module_status", &format!("{}/{} enabled", enabled_modules, self.modules.len()))
            .field("total_errors", &total_errors)
            .field("modules", &if f.alternate() { &self.modules } else { &self.modules[..enabled_modules.min(2)] })
            .field("config_keys", &self.config.keys().collect::>())
            .finish()
    }
}

fn main() {
    let mut config = BTreeMap::new();
    config.insert("max_connections".to_string(), ConfigValue::Number(1000.0));
    config.insert("debug_mode".to_string(), ConfigValue::Boolean(true));
    config.insert("allowed_hosts".to_string(), ConfigValue::List(vec!["localhost".to_string()]));

    let app = Application {
        name: "WebServer".to_string(),
        version: "1.2.3".to_string(),
        modules: vec![
            Module {
                name: "HTTP".to_string(),
                enabled: true,
                dependencies: vec!["logging".to_string()],
                error_count: 0,
            },
            Module {
                name: "Database".to_string(),
                enabled: true,
                dependencies: vec!["connection_pool".to_string()],
                error_count: 2,
            },
            Module {
                name: "Cache".to_string(),
                enabled: false,
                dependencies: vec![],
                error_count: 0,
            },
        ],
        config,
    };

    println!("Application overview: {:?}", app);
    println!("\nDetailed view:\n{:#?}", app);
}
```

## Debugging Techniques and Patterns

### Wrapper Types for External Types

When you can't implement Debug for external types, create wrapper types:

```rust
use std::fmt;

// Imagine this is from an external crate and doesn't implement Debug
struct ExternalType {
    value: i32,
}

// Create a wrapper that implements Debug
struct DebuggableExternal(&'a ExternalType);

impl fmt::Debug for DebuggableExternal {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("ExternalType")
            .field("value", &self.0.value)
            .field("computed_square", &(self.0.value * self.0.value))
            .finish()
    }
}

// Helper function for easy wrapping
fn debug_external(ext: &ExternalType) -> DebuggableExternal {
    DebuggableExternal(ext)
}

fn main() {
    let external = ExternalType { value: 42 };

    // Can't do this: println!("{:?}", external);
    // But can do this:
    println!("{:?}", debug_external(&external));
}
```

### Debug for Collections and Custom Data Structures

```rust
use std::fmt;

struct CircularBuffer {
    data: Vec>,
    head: usize,
    tail: usize,
    size: usize,
}

impl fmt::Debug for CircularBuffer {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let current_items: Vec = self.iter().collect();

        f.debug_struct("CircularBuffer")
            .field("capacity", &self.data.len())
            .field("size", &self.size)
            .field("head", &self.head)
            .field("tail", &self.tail)
            .field("utilization", &format!("{:.1}%", (self.size as f64 / self.data.len() as f64) * 100.0))
            .field("items", &current_items)
            .finish()
    }
}

impl CircularBuffer {
    fn new(capacity: usize) -> Self {
        CircularBuffer {
            data: (0..capacity).map(|_| None).collect(),
            head: 0,
            tail: 0,
            size: 0,
        }
    }

    fn push(&mut self, item: T) {
        self.data[self.tail] = Some(item);
        self.tail = (self.tail + 1) % self.data.len();

        if self.size  impl Iterator {
        (0..self.size).map(move |i| {
            let index = (self.head + i) % self.data.len();
            self.data[index].as_ref().unwrap()
        })
    }
}

fn main() {
    let mut buffer = CircularBuffer::new(5);

    // Add some items
    for i in 1..=7 {
        buffer.push(format!("Item {}", i));
    }

    println!("{:#?}", buffer);
}
```

## Best Practices and Common Patterns

### 1. Security-Aware Debug Implementation

```rust
use std::fmt;

struct UserCredentials {
    username: String,
    password_hash: String,
    api_key: String,
    last_login: u64,
}

impl fmt::Debug for UserCredentials {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("UserCredentials")
            .field("username", &self.username)
            .field("password_hash", &"[REDACTED]")
            .field("api_key", &format!("{}...", &self.api_key[..8.min(self.api_key.len())]))
            .field("last_login", &self.last_login)
            .finish()
    }
}

struct Session {
    user: UserCredentials,
    token: String,
    expires_at: u64,
}

impl fmt::Debug for Session {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("Session")
            .field("user", &self.user)  // UserCredentials has secure Debug
            .field("token", &"[PROTECTED]")
            .field("expires_at", &self.expires_at)
            .finish()
    }
}
```

### 2. Performance-Aware Debug Implementation

```rust
use std::fmt;

struct LargeDataset {
    name: String,
    data: Vec,  // Could be millions of items
    processed: bool,
}

impl fmt::Debug for LargeDataset {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let preview_size = if f.alternate() { 10 } else { 3 };

        let data_preview = if self.data.len()  String {
        if self.data.is_empty() {
            return "No data".to_string();
        }

        let sum: f64 = self.data.iter().sum();
        let avg = sum / self.data.len() as f64;
        let min = self.data.iter().fold(f64::INFINITY, |a, &b| a.min(b));
        let max = self.data.iter().fold(f64::NEG_INFINITY, |a, &b| a.max(b));

        format!("avg: {:.2}, min: {:.2}, max: {:.2}", avg, min, max)
    }
}
```

### 3. Error-Context Debug Implementation

```rust
use std::fmt;
use std::error::Error;

#[derive(Debug)]
enum DatabaseError {
    ConnectionFailed(String),
    QueryTimeout,
    InvalidQuery(String),
}

impl fmt::Display for DatabaseError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            DatabaseError::ConnectionFailed(msg) => write!(f, "Connection failed: {}", msg),
            DatabaseError::QueryTimeout => write!(f, "Query timeout"),
            DatabaseError::InvalidQuery(query) => write!(f, "Invalid query: {}", query),
        }
    }
}

impl Error for DatabaseError {}

struct QueryResult {
    data: Option,
    error: Option,
    query: String,
    execution_time_ms: u64,
    rows_affected: usize,
}

impl fmt::Debug for QueryResult {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let status = match (&self.data, &self.error) {
            (Some(_), None) => "✅ SUCCESS",
            (None, Some(_)) => "❌ ERROR",
            (None, None) => "⚪ EMPTY",
            (Some(_), Some(_)) => "⚠️  WARNING", // Both data and error? Unusual but possible
        };

        let mut debug_struct = f.debug_struct("QueryResult");
        debug_struct
            .field("status", &status)
            .field("execution_time", &format!("{}ms", self.execution_time_ms))
            .field("rows_affected", &self.rows_affected);

        if let Some(ref data) = self.data {
            debug_struct.field("data", data);
        }

        if let Some(ref error) = self.error {
            debug_struct.field("error", error);
        }

        if f.alternate() {
            debug_struct.field("query", &self.query);
        } else {
            debug_struct.field("query", &format!("{}...", &self.query[..50.min(self.query.len())]));
        }

        debug_struct.finish()
    }
}
```

## Common Mistakes and Solutions

### Mistake 1: Infinite Recursion

```rust
use std::fmt;

// ❌ This can cause infinite recursion
struct Node {
    value: i32,
    children: Vec,
}

impl fmt::Debug for Node {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // If children contain references back to parent, this could recurse infinitely
        f.debug_struct("Node")
            .field("value", &self.value)
            .field("children", &self.children)  // Potential infinite recursion
            .finish()
    }
}

// ✅ Better approach - limit depth
struct SafeNode {
    value: i32,
    children: Vec,
}

impl fmt::Debug for SafeNode {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        self.fmt_with_depth(f, 0, 3) // Max depth of 3
    }
}

impl SafeNode {
    fn fmt_with_depth(&self, f: &mut fmt::Formatter, current_depth: usize, max_depth: usize) -> fmt::Result {
        if current_depth >= max_depth {
            return write!(f, "SafeNode {{ value: {}, children: [...] }}", self.value);
        }

        f.debug_struct("SafeNode")
            .field("value", &self.value)
            .field("children", &SafeNodeChildren {
                children: &self.children,
                depth: current_depth + 1,
                max_depth
            })
            .finish()
    }
}

struct SafeNodeChildren {
    children: &'a [SafeNode],
    depth: usize,
    max_depth: usize,
}

impl fmt::Debug for SafeNodeChildren {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let mut list = f.debug_list();
        for child in self.children {
            list.entry(&SafeNodeDebugger { node: child, depth: self.depth, max_depth: self.max_depth });
        }
        list.finish()
    }
}

struct SafeNodeDebugger {
    node: &'a SafeNode,
    depth: usize,
    max_depth: usize,
}

impl fmt::Debug for SafeNodeDebugger {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        self.node.fmt_with_depth(f, self.depth, self.max_depth)
    }
}
```

### Mistake 2: Exposing Sensitive Information

```rust
// ❌ Bad - exposes sensitive data
#[derive(Debug)]
struct BadUserAccount {
    username: String,
    password: String,      // Sensitive!
    credit_card: String,   // Very sensitive!
    email: String,
}

// ✅ Good - custom implementation that hides sensitive data
struct GoodUserAccount {
    username: String,
    password: String,
    credit_card: String,
    email: String,
}

impl fmt::Debug for GoodUserAccount {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("UserAccount")
            .field("username", &self.username)
            .field("password", &"[REDACTED]")
            .field("credit_card", &"[PROTECTED]")
            .field("email", &self.mask_email())
            .finish()
    }
}

impl GoodUserAccount {
    fn mask_email(&self) -> String {
        if let Some(at_pos) = self.email.find('@') {
            let (local, domain) = self.email.split_at(at_pos);
            if local.len() ) -> fmt::Result {
        // Custom formatting logic
    }
}

// ✅ Use debug_struct helper for structured output
f.debug_struct("MyStruct")
    .field("field1", &self.field1)
    .field("field2", &"custom_value")
    .finish()
```

### **Common Patterns Summary**

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Auto-derive** | Simple structs, all fields safe to show | `#[derive(Debug)]` |
| **Security-aware** | Hide sensitive information | `password: &"[REDACTED]"` |
| **Performance-aware** | Large datasets, expensive computations | Show summaries, not full data |
| **Context-aware** | Different detail levels | Use `f.alternate()` to detect `{:#?}` |
| **Wrapper types** | External types without Debug | `struct MyDebug(&'a ExternalType)` |

### **Best Practices Checklist**

- ✅ **Hide sensitive data** like passwords, tokens, personal information
- ✅ **Show relevant information** that helps with debugging
- ✅ **Use helpers** like `debug_struct()` for cleaner implementations
- ✅ **Consider performance** - avoid expensive operations in Debug
- ✅ **Handle large data sets** - show summaries, not everything
- ✅ **Test your Debug output** - make sure it's actually helpful
- ✅ **Document security implications** - what data is exposed?

**The Debug trait is your window into the soul of your data structures.** When implemented thoughtfully, it becomes an invaluable tool for understanding, debugging, and maintaining your Rust applications. Think of it as creating the perfect "developer view" that shows exactly what you need to know, when you need to know it!

1. https://doc.rust-lang.org/std/fmt/trait.Debug.html
2. https://doc.rust-lang.org/rust-by-example/hello/print/print_debug.html
3. https://www.reddit.com/r/rust/comments/7qqbyp/implemented_displaydebug_on_a_trait/
4. https://stackoverflow.com/questions/22243527/how-to-implement-a-custom-fmtdebug-trait
5. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/std/fmt/trait.Debug.html
6. https://users.rust-lang.org/t/customize-the-debug-formatter/127748
7. https://www.rustadventure.dev/uploading-pokemon-data-from-a-csv-into-a-planetscale-sql-database/sqlx-0.5/implementing-the-debug-trait-for-third-party-types
8. https://practice.course.rs/formatted-output/debug-display.html
9. https://www.rustcodeweb.com/2025/02/debug-traits-in-rust.html
10. https://www.dotnetperls.com/debug-rust
11. https://users.rust-lang.org/t/how-to-implement-debug-for-dyn-trait/39950
12. https://dev.to/fadygrab/learning-rust-11-debugging-custom-typesstructs-3dnp
13. https://livebook.manning.com/wiki/categories/rust/debug
14. https://users.rust-lang.org/t/idiomatic-to-implement-the-debug-trait-for-x-syntax/84955
15. https://youtrack.jetbrains.com/issue/RUST-17404/Debugger-Render-using-format-provided-by-Debug-or-Display-traits
16. https://www.reddit.com/r/rust/comments/1egz6jw/how_would_i_implement_debug_or_equivalent_for/
17. https://www.youtube.com/watch?v=0jp8oNwCsGI
