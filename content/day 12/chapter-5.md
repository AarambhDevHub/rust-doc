+++
title = "Method Chaining"
description = "Discover how to implement method chaining in Rust to call multiple methods in a concise and readable way using fluent interfaces."
date = 2025-08-04
draft = false
weight = 5
+++

# Method Chaining in Rust: Comprehensive Documentation for Beginners

Method chaining is one of the most elegant and powerful patterns in Rust that allows you to write fluent, readable code by calling multiple methods in sequence on the same object. Think of it like an assembly line where each worker (method) performs a specific task and passes the product to the next worker.

## Understanding Method Chaining

### What is Method Chaining?

**Method chaining** is a programming technique where multiple methods are called sequentially on the same object in a single expression. Each method returns an object (usually `self`) that can be used to call the next method.

| Concept | Explanation |
|:--------|:------------|
| **What is method chaining?** | Method chaining is a technique where multiple methods are called sequentially on the same object in a single expression. |
| **How does it work?** | Each method returns the object itself (`self`), enabling the next method call on the returned instance. |
| **Visualization** | Imagine a factory assembly line where the product moves through different stages; each stage (method) changes or configures the product, passes it forward. |
| **Benefits** | Cleaner, more readable code; compact expression of successive transformations. |
| **Typical use** | Fluent APIs, builder patterns, configuration setups. |

### Visual Concept: The Assembly Line

Imagine a car manufacturing assembly line:

```rust
// Think of this like a car going through an assembly line
let finished_car = Car::new()           // 🏭 Start with basic frame
    .install_engine("V8")               // 🔧 Add engine
    .paint_color("Red")                 // 🎨 Paint the car
    .add_wheels(4)                      // ⚪ Install wheels
    .install_interior("Leather")        // 🪑 Add interior
    .finalize();                        // ✅ Final inspection

// Each method transforms the car and passes it to the next station
```

## Basic Method Chaining Examples

### Simple Number Calculator

Let's start with a basic example to understand the mechanics:

```rust
#[derive(Debug)]
struct Calculator {
    value: f64,
}

impl Calculator {
    // Constructor - starts the chain
    fn new(initial: f64) -> Self {
        Calculator { value: initial }
    }

    // Each method takes ownership, modifies, and returns self
    fn add(mut self, num: f64) -> Self {
        self.value += num;
        self  // Return self to enable chaining
    }

    fn multiply(mut self, num: f64) -> Self {
        self.value *= num;
        self  // Return self to enable chaining
    }

    fn subtract(mut self, num: f64) -> Self {
        self.value -= num;
        self  // Return self to enable chaining
    }

    fn divide(mut self, num: f64) -> Self {
        if num != 0.0 {
            self.value /= num;
        }
        self  // Return self to enable chaining
    }

    // Final method - doesn't need to return self
    fn result(self) -> f64 {
        self.value
    }

    // Alternative final method
    fn display(self) {
        println!("Final result: {}", self.value);
    }
}

fn main() {
    // Method chaining in action!
    let result = Calculator::new(10.0)
        .add(5.0)           // 10 + 5 = 15
        .multiply(2.0)      // 15 * 2 = 30
        .subtract(8.0)      // 30 - 8 = 22
        .divide(2.0)        // 22 / 2 = 11
        .result();          // Get final value

    println!("Result: {}", result); // Result: 11

    // Another example with display
    Calculator::new(100.0)
        .multiply(2.0)
        .add(50.0)
        .display();  // Final result: 250
}
```

**Key Points:**
- Each method takes `self` (ownership) and returns `Self`
- The object "moves" through each method call
- Only the final method needs to extract the result

### Step-by-Step Breakdown

Let's trace through what happens in the chaining:

```rust
fn demonstrate_step_by_step() {
    // This single chained expression...
    let result = Calculator::new(5.0)
        .add(3.0)
        .multiply(2.0)
        .result();

    // ...is equivalent to this step-by-step process:
    let step1 = Calculator::new(5.0);           // Calculator { value: 5.0 }
    let step2 = step1.add(3.0);                 // Calculator { value: 8.0 }
    let step3 = step2.multiply(2.0);            // Calculator { value: 16.0 }
    let final_result = step3.result();          // 16.0

    println!("Chained result: {}", result);     // 16.0
    println!("Step-by-step result: {}", final_result); // 16.0
}
```

## Builder Pattern with Method Chaining

### HTTP Request Builder

One of the most common uses of method chaining is the **builder pattern**:

```rust
#[derive(Debug)]
struct HttpRequest {
    method: String,
    url: String,
    headers: Vec,
    body: Option,
    timeout: u64,
}

struct HttpRequestBuilder {
    method: String,
    url: String,
    headers: Vec,
    body: Option,
    timeout: u64,
}

impl HttpRequestBuilder {
    // Start the chain
    fn new(method: &str, url: &str) -> Self {
        HttpRequestBuilder {
            method: method.to_string(),
            url: url.to_string(),
            headers: Vec::new(),
            body: None,
            timeout: 30,
        }
    }

    // Each builder method returns Self for chaining
    fn header(mut self, name: &str, value: &str) -> Self {
        self.headers.push((name.to_string(), value.to_string()));
        self
    }

    fn body(mut self, body: String) -> Self {
        self.body = Some(body);
        self
    }

    fn timeout(mut self, seconds: u64) -> Self {
        self.timeout = seconds;
        self
    }

    // Terminal method - builds the final object
    fn build(self) -> HttpRequest {
        HttpRequest {
            method: self.method,
            url: self.url,
            headers: self.headers,
            body: self.body,
            timeout: self.timeout,
        }
    }
}

impl HttpRequest {
    // Convenience constructors that return builders
    fn get(url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new("GET", url)
    }

    fn post(url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new("POST", url)
    }

    fn put(url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new("PUT", url)
    }
}

fn main() {
    // Build a GET request
    let get_request = HttpRequest::get("https://api.example.com/users")
        .header("Authorization", "Bearer token123")
        .header("Accept", "application/json")
        .timeout(60)
        .build();

    // Build a POST request
    let post_request = HttpRequest::post("https://api.example.com/users")
        .header("Content-Type", "application/json")
        .header("Authorization", "Bearer token123")
        .body(r#"{"name": "Alice", "email": "alice@example.com"}"#.to_string())
        .timeout(45)
        .build();

    println!("GET Request: {:#?}", get_request);
    println!("POST Request: {:#?}", post_request);
}
```

### Configuration Builder

Another practical example - building application configuration:

```rust
#[derive(Debug)]
struct AppConfig {
    database_url: String,
    port: u16,
    log_level: String,
    debug_mode: bool,
    max_connections: u32,
    timeout: u64,
}

struct ConfigBuilder {
    database_url: Option,
    port: u16,
    log_level: String,
    debug_mode: bool,
    max_connections: u32,
    timeout: u64,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            database_url: None,
            port: 8080,              // Default values
            log_level: "info".to_string(),
            debug_mode: false,
            max_connections: 100,
            timeout: 30,
        }
    }

    fn database_url(mut self, url: &str) -> Self {
        self.database_url = Some(url.to_string());
        self
    }

    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }

    fn log_level(mut self, level: &str) -> Self {
        self.log_level = level.to_string();
        self
    }

    fn debug_mode(mut self, enabled: bool) -> Self {
        self.debug_mode = enabled;
        self
    }

    fn max_connections(mut self, max: u32) -> Self {
        self.max_connections = max;
        self
    }

    fn timeout(mut self, seconds: u64) -> Self {
        self.timeout = seconds;
        self
    }

    fn build(self) -> Result {
        let database_url = self.database_url
            .ok_or("Database URL is required")?;

        Ok(AppConfig {
            database_url,
            port: self.port,
            log_level: self.log_level,
            debug_mode: self.debug_mode,
            max_connections: self.max_connections,
            timeout: self.timeout,
        })
    }
}

fn main() -> Result {
    // Development configuration
    let dev_config = ConfigBuilder::new()
        .database_url("postgresql://localhost:5432/myapp_dev")
        .port(3000)
        .log_level("debug")
        .debug_mode(true)
        .max_connections(10)
        .build()?;

    // Production configuration
    let prod_config = ConfigBuilder::new()
        .database_url("postgresql://prod-server:5432/myapp_prod")
        .port(80)
        .log_level("warn")
        .debug_mode(false)
        .max_connections(200)
        .timeout(60)
        .build()?;

    println!("Dev config: {:#?}", dev_config);
    println!("Prod config: {:#?}", prod_config);

    Ok(())
}
```

## Data Processing Chains

### Text Processor Example

Method chaining is excellent for data transformation pipelines:

```rust
#[derive(Debug)]
struct TextProcessor {
    text: String,
}

impl TextProcessor {
    fn new(text: &str) -> Self {
        TextProcessor {
            text: text.to_string(),
        }
    }

    fn to_lowercase(mut self) -> Self {
        self.text = self.text.to_lowercase();
        self
    }

    fn remove_whitespace(mut self) -> Self {
        self.text = self.text.chars()
            .filter(|c| !c.is_whitespace())
            .collect();
        self
    }

    fn remove_punctuation(mut self) -> Self {
        self.text = self.text.chars()
            .filter(|c| c.is_alphanumeric())
            .collect();
        self
    }

    fn reverse(mut self) -> Self {
        self.text = self.text.chars().rev().collect();
        self
    }

    fn add_prefix(mut self, prefix: &str) -> Self {
        self.text = format!("{}{}", prefix, self.text);
        self
    }

    fn add_suffix(mut self, suffix: &str) -> Self {
        self.text = format!("{}{}", self.text, suffix);
        self
    }

    fn repeat(mut self, times: usize) -> Self {
        self.text = self.text.repeat(times);
        self
    }

    fn finalize(self) -> String {
        self.text
    }

    fn length(&self) -> usize {
        self.text.len()
    }
}

fn main() {
    // Clean and process text in one fluent chain
    let processed = TextProcessor::new("  Hello, World!  ")
        .to_lowercase()           // "  hello, world!  "
        .remove_whitespace()      // "hello,world!"
        .remove_punctuation()     // "helloworld"
        .add_prefix(">> ")        // ">> helloworld"
        .add_suffix(" <<")        // ">> helloworld <<"
        .finalize();

    println!("Processed text: '{}'", processed);

    // Another example - create a pattern
    let pattern = TextProcessor::new("Rust")
        .to_lowercase()           // "rust"
        .repeat(3)                // "rustrurust"
        .add_prefix("=== ")       // "=== rustrurust"
        .add_suffix(" ===")       // "=== rustrurust ==="
        .finalize();

    println!("Pattern: '{}'", pattern);
}
```

### Number Sequence Builder
```rust
#[derive(Debug)]
struct NumberSequence {
    numbers: Vec<i32>,
}

impl NumberSequence {
    fn new() -> Self {
        NumberSequence {
            numbers: Vec::new(),
        }
    }

    fn add_range(mut self, start: i32, end: i32) -> Self {
        for num in start..=end {
            self.numbers.push(num);
        }
        self
    }

    fn filter_even(mut self) -> Self {
        self.numbers.retain(|&x| x % 2 == 0);
        self
    }

    fn filter_odd(mut self) -> Self {
        self.numbers.retain(|&x| x % 2 != 0);
        self
    }

    fn multiply_by(mut self, factor: i32) -> Self {
        self.numbers = self.numbers.iter().map(|&x| x * factor).collect();
        self
    }

    fn add_to_all(mut self, value: i32) -> Self {
        self.numbers = self.numbers.iter().map(|&x| x + value).collect();
        self
    }

    fn sort(mut self) -> Self {
        self.numbers.sort();
        self
    }

    fn reverse(mut self) -> Self {
        self.numbers.reverse();
        self
    }

    fn take_first(mut self, count: usize) -> Self {
        self.numbers.truncate(count);
        self
    }

    fn collect(self) -> Vec<i32> {
        self.numbers
    }

    fn sum(&self) -> i32 {
        self.numbers.iter().sum()
    }
}

fn main() {
    // Build a sequence of processed numbers
    let result = NumberSequence::new()
        .add_range(1, 10)         // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
        .filter_even()            // [2, 4, 6, 8, 10]
        .multiply_by(3)           // [6, 12, 18, 24, 30]
        .add_to_all(5)            // [11, 17, 23, 29, 35]
        .reverse()                // [35, 29, 23, 17, 11]
        .take_first(3)            // [35, 29, 23]
        .collect();

    println!("Final sequence: {:?}", result);

    // Calculate sum in a chain
    let sum = NumberSequence::new()
        .add_range(1, 100)
        .filter_odd()
        .take_first(10)
        .sum();

    println!("Sum of first 10 odd numbers: {}", sum);
}
```

## Different Return Patterns

### Returning Self vs &mut Self

There are different approaches to method chaining, each with trade-offs:

```rust
#[derive(Debug)]
struct Counter {
    value: i32,
}

// Pattern 1: Consuming methods (take and return Self)
impl Counter {
    fn new(value: i32) -> Self {
        Counter { value }
    }

    fn add_consuming(mut self, n: i32) -> Self {
        self.value += n;
        self  // Returns owned value
    }

    fn multiply_consuming(mut self, n: i32) -> Self {
        self.value *= n;
        self  // Returns owned value
    }

    fn finalize_consuming(self) -> i32 {
        self.value
    }
}

// Pattern 2: Mutating methods (take and return &mut Self)
impl Counter {
    fn add_mutating(&mut self, n: i32) -> &mut Self {
        self.value += n;
        self  // Returns mutable reference
    }

    fn multiply_mutating(&mut self, n: i32) -> &mut Self {
        self.value *= n;
        self  // Returns mutable reference
    }

    fn get_value(&self) -> i32 {
        self.value
    }
}

fn main() {
    // Consuming pattern - object moves through chain
    let result1 = Counter::new(5)
        .add_consuming(3)         // Counter moves to add_consuming
        .multiply_consuming(2)    // Counter moves to multiply_consuming
        .finalize_consuming();    // Counter consumed, returns i32

    println!("Consuming result: {}", result1);

    // Mutating pattern - same object is modified
    let mut counter = Counter::new(5);
    counter
        .add_mutating(3)          // Returns &mut Counter
        .multiply_mutating(2);    // Returns &mut Counter

    println!("Mutating result: {}", counter.get_value());

    // The counter is still usable after chaining!
    counter.add_mutating(10);
    println!("After additional operation: {}", counter.get_value());
}
```

### When to Use Each Pattern

```rust
// Use consuming pattern when:
// 1. Building immutable objects
// 2. Each step creates a new logical state
// 3. You want to prevent further modification

let config = ConfigBuilder::new()
    .database_url("...")
    .port(8080)
    .build();  // config is immutable

// Use mutating pattern when:
// 1. You need to reuse the object
// 2. Performance is critical (avoid moves)
// 3. You're modifying existing state

let mut logger = Logger::new();
logger
    .set_level("debug")
    .add_output("file.log")
    .enable_colors();

// logger is still available for further use
logger.log("Application started");
```

## Advanced Chaining Patterns

### Conditional Chaining

```rust
#[derive(Debug)]
struct RequestBuilder {
    url: String,
    headers: Vec,
    auth_token: Option,
}

impl RequestBuilder {
    fn new(url: &str) -> Self {
        RequestBuilder {
            url: url.to_string(),
            headers: Vec::new(),
            auth_token: None,
        }
    }

    fn header(mut self, name: &str, value: &str) -> Self {
        self.headers.push((name.to_string(), value.to_string()));
        self
    }

    fn auth_token(mut self, token: &str) -> Self {
        self.auth_token = Some(token.to_string());
        self
    }

    // Conditional method - only applies auth if token exists
    fn maybe_auth(self, token: Option) -> Self {
        match token {
            Some(t) => self.auth_token(t),
            None => self,
        }
    }

    // Conditional method with closure
    fn when(self, condition: bool, f: F) -> Self
    where
        F: FnOnce(Self) -> Self,
    {
        if condition {
            f(self)
        } else {
            self
        }
    }

    fn build(self) -> String {
        format!("Request to {} with {} headers", self.url, self.headers.len())
    }
}

fn main() {
    let user_token = Some("user123");
    let is_production = true;

    let request = RequestBuilder::new("https://api.example.com/users")
        .header("Accept", "application/json")
        .maybe_auth(user_token)  // Only adds auth if token exists
        .when(is_production, |req| {
            req.header("X-Environment", "production")
                .header("X-Request-ID", "req-12345")
        })
        .header("User-Agent", "MyApp/1.0")
        .build();

    println!("{}", request);
}
```

### Error Handling in Chains

```rust
#[derive(Debug)]
enum ProcessError {
    InvalidInput(String),
    ProcessingFailed(String),
}

#[derive(Debug)]
struct DataProcessor {
    data: Vec,
}

impl DataProcessor {
    fn new(data: Vec) -> Self {
        DataProcessor { data }
    }

    fn validate(self) -> Result {
        if self.data.is_empty() {
            Err(ProcessError::InvalidInput("Data cannot be empty".to_string()))
        } else {
            Ok(self)
        }
    }

    fn filter_positive(mut self) -> Result {
        self.data.retain(|&x| x > 0);
        if self.data.is_empty() {
            Err(ProcessError::ProcessingFailed("No positive numbers found".to_string()))
        } else {
            Ok(self)
        }
    }

    fn multiply_by(mut self, factor: i32) -> Result {
        if factor == 0 {
            return Err(ProcessError::InvalidInput("Factor cannot be zero".to_string()));
        }

        self.data = self.data.iter().map(|&x| x * factor).collect();
        Ok(self)
    }

    fn finalize(self) -> Vec {
        self.data
    }
}

fn main() -> Result {
    // Chain operations that can fail
    let result = DataProcessor::new(vec![-2, -1, 0, 1, 2, 3])
        .validate()?                    // Check if data is valid
        .filter_positive()?             // Keep only positive numbers
        .multiply_by(10)?               // Multiply by 10
        .finalize();                    // Get final result

    println!("Successful processing: {:?}", result);

    // This will fail at filter_positive
    match DataProcessor::new(vec![-5, -3, -1])
        .validate()
        .and_then(|p| p.filter_positive())
        .and_then(|p| p.multiply_by(2))
    {
        Ok(processor) => println!("Success: {:?}", processor.finalize()),
        Err(e) => println!("Error: {:?}", e),
    }

    Ok(())
}
```

## Iterator-Style Chaining

### Built-in Iterator Chaining

Rust's iterators provide excellent examples of method chaining:

```rust
fn iterator_chaining_examples() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Complex processing in a single chain
    let result: Vec = numbers
        .iter()                           // Create iterator
        .filter(|&&x| x % 2 == 0)        // Keep even numbers: [2, 4, 6, 8, 10]
        .map(|&x| x * x)                 // Square them: [4, 16, 36, 64, 100]
        .filter(|&x| x > 20)             // Keep > 20: [36, 64, 100]
        .map(|x| format!("Value: {}", x)) // Format as strings
        .collect();                      // Collect into Vec

    println!("Iterator result: {:?}", result);

    // Calculate sum of squares of even numbers
    let sum_of_squares: i32 = numbers
        .iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .sum();

    println!("Sum of squares of even numbers: {}", sum_of_squares);

    // Find first number > 5, double it
    let first_doubled = numbers
        .iter()
        .find(|&&x| x > 5)
        .map(|&x| x * 2);

    println!("First number > 5, doubled: {:?}", first_doubled);
}

fn main() {
    iterator_chaining_examples();
}
```

### Custom Iterator-like Chaining

```rust
#[derive(Debug)]
struct StringTransformer {
    value: String,
}

impl StringTransformer {
    fn new(s: &str) -> Self {
        StringTransformer {
            value: s.to_string(),
        }
    }

    fn map(mut self, f: F) -> Self
    where
        F: FnOnce(String) -> String,
    {
        self.value = f(self.value);
        self
    }

    fn filter(self, predicate: F) -> Option
    where
        F: FnOnce(&str) -> bool,
    {
        if predicate(&self.value) {
            Some(self)
        } else {
            None
        }
    }

    fn unwrap_or_default(self) -> String {
        self.value
    }
}

fn main() {
    // Chain transformations with map
    let result = StringTransformer::new("hello world")
        .map(|s| s.to_uppercase())              // "HELLO WORLD"
        .map(|s| s.replace(" ", "_"))           // "HELLO_WORLD"
        .map(|s| format!(">", s))          // ">"
        .unwrap_or_default();

    println!("Transformed: {}", result);

    // Chain with filtering
    let filtered_result = StringTransformer::new("rust programming")
        .map(|s| s.replace(" ", ""))            // "rustprogramming"
        .filter(|s| s.len() > 10)               // Check if length > 10
        .map(|transformer| transformer.map(|s| s.to_uppercase()))  // Convert to uppercase if passed filter
        .unwrap_or_else(|| StringTransformer::new("TOO_SHORT"))
        .unwrap_or_default();

    println!("Filtered result: {}", filtered_result);
}
```

## Practical Real-World Examples

### SQL Query Builder

```rust
#[derive(Debug)]
struct QueryBuilder {
    select_fields: Vec,
    from_table: Option,
    where_conditions: Vec,
    joins: Vec,
    order_by: Vec,
    limit_count: Option,
}

impl QueryBuilder {
    fn new() -> Self {
        QueryBuilder {
            select_fields: Vec::new(),
            from_table: None,
            where_conditions: Vec::new(),
            joins: Vec::new(),
            order_by: Vec::new(),
            limit_count: None,
        }
    }

    fn select(mut self, field: &str) -> Self {
        self.select_fields.push(field.to_string());
        self
    }

    fn select_all(mut self) -> Self {
        self.select_fields.push("*".to_string());
        self
    }

    fn from(mut self, table: &str) -> Self {
        self.from_table = Some(table.to_string());
        self
    }

    fn where_clause(mut self, condition: &str) -> Self {
        self.where_conditions.push(condition.to_string());
        self
    }

    fn join(mut self, join_clause: &str) -> Self {
        self.joins.push(join_clause.to_string());
        self
    }

    fn order_by(mut self, field: &str) -> Self {
        self.order_by.push(field.to_string());
        self
    }

    fn limit(mut self, count: u32) -> Self {
        self.limit_count = Some(count);
        self
    }

    fn build(self) -> String {
        let mut query = String::new();

        // SELECT clause
        query.push_str("SELECT ");
        if self.select_fields.is_empty() {
            query.push_str("*");
        } else {
            query.push_str(&self.select_fields.join(", "));
        }

        // FROM clause
        if let Some(table) = self.from_table {
            query.push_str(&format!(" FROM {}", table));
        }

        // JOIN clauses
        for join in &self.joins {
            query.push_str(&format!(" {}", join));
        }

        // WHERE clause
        if !self.where_conditions.is_empty() {
            query.push_str(&format!(" WHERE {}", self.where_conditions.join(" AND ")));
        }

        // ORDER BY clause
        if !self.order_by.is_empty() {
            query.push_str(&format!(" ORDER BY {}", self.order_by.join(", ")));
        }

        // LIMIT clause
        if let Some(limit) = self.limit_count {
            query.push_str(&format!(" LIMIT {}", limit));
        }

        query
    }
}

fn main() {
    // Build a complex SQL query using method chaining
    let query = QueryBuilder::new()
        .select("u.id")
        .select("u.name")
        .select("u.email")
        .from("users u")
        .join("LEFT JOIN profiles p ON u.id = p.user_id")
        .where_clause("u.active = true")
        .where_clause("u.created_at > '2023-01-01'")
        .order_by("u.name")
        .order_by("u.id")
        .limit(50)
        .build();

    println!("Generated SQL:\n{}", query);

    // Simple query
    let simple_query = QueryBuilder::new()
        .select_all()
        .from("products")
        .where_clause("price > 100")
        .order_by("name")
        .build();

    println!("\nSimple query:\n{}", simple_query);
}
```

### Game Character Builder

```rust
#[derive(Debug)]
struct GameCharacter {
    name: String,
    health: u32,
    mana: u32,
    level: u32,
    skills: Vec,
    equipment: Vec,
    stats: CharacterStats,
}

#[derive(Debug)]
struct CharacterStats {
    strength: u32,
    intelligence: u32,
    dexterity: u32,
    charisma: u32,
}

struct CharacterBuilder {
    name: String,
    health: u32,
    mana: u32,
    level: u32,
    skills: Vec,
    equipment: Vec,
    stats: CharacterStats,
}

impl CharacterBuilder {
    fn new(name: &str) -> Self {
        CharacterBuilder {
            name: name.to_string(),
            health: 100,
            mana: 50,
            level: 1,
            skills: Vec::new(),
            equipment: Vec::new(),
            stats: CharacterStats {
                strength: 10,
                intelligence: 10,
                dexterity: 10,
                charisma: 10,
            },
        }
    }

    fn level(mut self, level: u32) -> Self {
        self.level = level;
        // Adjust base stats based on level
        let bonus = level - 1;
        self.health += bonus * 10;
        self.mana += bonus * 5;
        self
    }

    fn add_skill(mut self, skill: &str) -> Self {
        self.skills.push(skill.to_string());
        self
    }

    fn add_equipment(mut self, item: &str) -> Self {
        self.equipment.push(item.to_string());
        self
    }

    fn strength(mut self, value: u32) -> Self {
        self.stats.strength = value;
        self.health += value * 2; // Strength affects health
        self
    }

    fn intelligence(mut self, value: u32) -> Self {
        self.stats.intelligence = value;
        self.mana += value * 3; // Intelligence affects mana
        self
    }

    fn dexterity(mut self, value: u32) -> Self {
        self.stats.dexterity = value;
        self
    }

    fn charisma(mut self, value: u32) -> Self {
        self.stats.charisma = value;
        self
    }

    // Character class presets
    fn warrior_preset(self) -> Self {
        self.strength(18)
            .dexterity(14)
            .intelligence(8)
            .charisma(12)
            .add_skill("Sword Fighting")
            .add_skill("Shield Defense")
            .add_equipment("Iron Sword")
            .add_equipment("Steel Shield")
            .add_equipment("Chain Mail")
    }

    fn mage_preset(self) -> Self {
        self.strength(8)
            .dexterity(12)
            .intelligence(18)
            .charisma(14)
            .add_skill("Fire Magic")
            .add_skill("Healing")
            .add_equipment("Magic Staff")
            .add_equipment("Spell Book")
            .add_equipment("Wizard Robes")
    }

    fn rogue_preset(self) -> Self {
        self.strength(12)
            .dexterity(18)
            .intelligence(14)
            .charisma(8)
            .add_skill("Stealth")
            .add_skill("Lock Picking")
            .add_equipment("Dagger")
            .add_equipment("Lockpicks")
            .add_equipment("Leather Armor")
    }

    fn build(self) -> GameCharacter {
        GameCharacter {
            name: self.name,
            health: self.health,
            mana: self.mana,
            level: self.level,
            skills: self.skills,
            equipment: self.equipment,
            stats: self.stats,
        }
    }
}

fn main() {
    // Create a custom warrior
    let warrior = CharacterBuilder::new("Thorgan the Brave")
        .level(5)
        .warrior_preset()
        .add_skill("Berserker Rage")
        .add_equipment("Magic Sword")
        .build();

    // Create a custom mage
    let mage = CharacterBuilder::new("Mystral the Wise")
        .level(7)
        .mage_preset()
        .intelligence(20) // Override preset intelligence
        .add_skill("Lightning Magic")
        .build();

    // Create a balanced character
    let ranger = CharacterBuilder::new("Aria Swiftarrow")
        .level(4)
        .strength(14)
        .dexterity(16)
        .intelligence(12)
        .charisma(13)
        .add_skill("Archery")
        .add_skill("Animal Handling")
        .add_skill("Tracking")
        .add_equipment("Long Bow")
        .add_equipment("Quiver")
        .add_equipment("Studded Leather")
        .build();

    println!("Warrior: {:#?}\n", warrior);
    println!("Mage: {:#?}\n", mage);
    println!("Ranger: {:#?}\n", ranger);
}
```

## Best Practices for Method Chaining

### Do's and Don'ts

**✅ DO:**
1. **Use descriptive method names** that clearly indicate what they do
2. **Return `Self` consistently** in builder methods
3. **Keep methods focused** - each should do one thing
4. **Provide good documentation** for complex chains
5. **Consider performance** - avoid unnecessary clones

**❌ DON'T:**
1. **Make chains too long** - break them up for readability
2. **Mix different concerns** in the same chain
3. **Ignore error handling** in fallible operations
4. **Force chaining** where it doesn't make sense

### Readable Chain Formatting

```rust
// ❌ Hard to read - all on one line
let result = RequestBuilder::new("url").header("auth", "token").timeout(30).retry(3).build();

// ✅ Easy to read - one method per line with comments
let result = RequestBuilder::new("https://api.example.com/data")
    .header("Authorization", "Bearer token123")    // Authentication
    .header("Accept", "application/json")          // Response format
    .timeout(30)                                   // 30 second timeout
    .retry(3)                                      // Retry up to 3 times
    .build();

// ✅ Group related operations
let character = CharacterBuilder::new("Hero")
    // Basic stats
    .level(10)
    .strength(15)
    .intelligence(12)

    // Skills and abilities
    .add_skill("Sword Fighting")
    .add_skill("Magic Resistance")

    // Equipment
    .add_equipment("Magic Sword")
    .add_equipment("Plate Armor")

    .build();
```

## Common Mistakes and Solutions

### Mistake 1: Forgetting to Return Self

```rust
// ❌ Wrong - doesn't return self
impl Builder {
    fn set_value(mut self, value: i32) {
        self.value = value;
        // Missing: self
    }
}

// ✅ Correct - returns self for chaining
impl Builder {
    fn set_value(mut self, value: i32) -> Self {
        self.value = value;
        self
    }
}
```

### Mistake 2: Inconsistent Return Types

```rust
// ❌ Wrong - mixing return types breaks chaining
impl Builder {
    fn set_name(mut self, name: String) -> Self {
        self.name = name;
        self
    }

    fn set_age(mut self, age: u32) -> u32 {  // Wrong return type!
        self.age = age;
        age  // Should return self
    }
}

// ✅ Correct - consistent return types
impl Builder {
    fn set_name(mut self, name: String) -> Self {
        self.name = name;
        self
    }

    fn set_age(mut self, age: u32) -> Self {
        self.age = age;
        self
    }
}
```

### Mistake 3: Not Handling Ownership Correctly

```rust
// ❌ Wrong - trying to use value after move
fn incorrect_usage() {
    let builder = Builder::new();
    let result1 = builder.set_value(10);  // builder moved here
    let result2 = builder.set_name("test".to_string());  // Error: use after move
}

// ✅ Correct - continue the chain or clone if needed
fn correct_usage() {
    let result = Builder::new()
        .set_value(10)
        .set_name("test".to_string());  // Chain continues

    // Or if you need multiple branches
    let base_builder = Builder::new().set_value(10);
    let result1 = base_builder.clone().set_name("test1".to_string());
    let result2 = base_builder.set_name("test2".to_string());
}
```

## Summary and Key Takeaways

### **Method Chaining Fundamentals**

**Method chaining** enables fluent, readable code by:
- **Returning `self`** from each method to enable the next call
- **Creating assembly-line processing** where each method transforms the object
- **Providing intuitive APIs** that read like natural language

### **Core Patterns**

| Pattern | Use Case | Example |
|---------|----------|---------|
| **Builder Pattern** | Object construction | `Builder::new().field1(val).field2(val).build()` |
| **Fluent API** | Configuration | `logger.level("debug").output("file").enable()` |
| **Data Processing** | Transformations | `data.filter().map().collect()` |
| **Query Building** | SQL/API queries | `query.select("*").from("table").where("condition")` |

### **Essential Guidelines**

1. **Each method should return `Self`** to enable chaining
2. **Keep methods focused** - one responsibility per method
3. **Use descriptive names** that clearly indicate the action
4. **Format chains vertically** for better readability
5. **Handle errors appropriately** in fallible operations
6. **Consider ownership patterns** - consuming vs mutating

### **Mental Model**

Think of method chaining as an **assembly line**:
- Each method is a **station** that performs a specific operation
- The object **moves through** each station, being transformed
- The **final method** produces the end result
- Each station **passes the work** to the next station

```rust
// Like an assembly line:
Product::new()           // 🏭 Start with raw materials
    .add_component(x)    // 🔧 Add component at station 1
    .paint(color)        // 🎨 Paint at station 2
    .test_quality()      // ✅ Quality check at station 3
    .package()           // 📦 Package at final station
    .ship()              // 🚚 Ship the finished product
```

**Method chaining is one of Rust's most powerful patterns for creating clean, expressive APIs** that make code both readable and efficient. When used thoughtfully, it transforms complex operations into intuitive, fluent expressions that clearly communicate intent.

1. https://dhghomon.github.io/easy_rust/Chapter_35.html
2. https://stackoverflow.com/questions/74965709/chaining-methods-in-rust
3. https://www.reddit.com/r/rust/comments/2wssch/function_chaining/
4. https://randompoison.github.io/posts/returning-self/
5. https://users.rust-lang.org/t/method-chaining/73895
6. https://www.youtube.com/watch?v=j70jq4ynrSk
7. https://refactoring.guru/design-patterns/chain-of-responsibility/rust/example
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/method-syntax.html
9. https://bfnightly.bracketproductions.com/chapter_36.html
10. https://users.rust-lang.org/t/chaining-options-and-results/67308
11. https://users.rust-lang.org/t/blog-post-chaining-functions-without-returning-self/26504
12. https://www.reddit.com/r/rust/comments/w8xlnn/how_do_i_quickly_improve_my_method_chaining/
13. https://docs.rs/chained/
14. https://labex.io/tutorials/rust-chainable-option-handling-with-and-then-99237
15. https://users.rust-lang.org/t/how-to-understand-complex-function-chaining-in-rust/67098
16. https://doc.rust-lang.org/beta/std/iter/struct.Chain.html
