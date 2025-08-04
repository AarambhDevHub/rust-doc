+++
title = "&self, &mut self, and self"
description = "Understand the differences between `&self`, `&mut self`, and `self` in Rust method definitions, and how they affect ownership and mutability."
date = 2025-08-04
draft = false
weight = 2
+++

# Understanding &self, &mut self, and self in Rust: Comprehensive Documentation for Beginners

Understanding the different forms of `self` in Rust method parameters is crucial for writing effective Rust code. These three receiver types - `&self`, `&mut self`, and `self` - represent different ways that methods can interact with the data they operate on. This comprehensive guide will help beginners understand when and how to use each type.

## What are Method Receivers?

**Method receivers** are the first parameter in Rust methods that determine how the method accesses the instance it's called on. They control ownership, mutability, and borrowing behavior. Think of them as different ways to "receive" the object that the method is working with.

| Receiver   | Description        | Explanation                      | Capabilities                                    | Typical Use                                           |
|:-----------|:-------------------|:---------------------------------|:------------------------------------------------|:------------------------------------------------------|
| &self      | Borrowed reference | Immutable borrow of the instance | Can read data, cannot modify                    | Use for read-only access, querying data               |
| &mut self  | Mutable reference  | Mutable borrow of the instance   | Can modify data                                 | Use when method modifies the instance                 |
| self       | Owned value        | Takes ownership of the instance  | Consumes the instance, may transform or drop it | Use for consuming methods, transforming or moving out |

## 1. &self - Immutable Borrowing (Most Common)

### What is &self?

**`&self`** is syntactic sugar for `self: &Self`. It means the method **borrows** the instance immutably. The method can read data but cannot modify it, and the original instance remains usable after the method call.

### Basic Example

```rust
#[derive(Debug)]
struct Book {
    title: String,
    author: String,
    pages: u32,
    published_year: u32,
}

impl Book {
    // Methods using &self - read-only access
    fn get_title(&self) -> &str {
        &self.title
    }

    fn get_author(&self) -> &str {
        &self.author
    }

    fn page_count(&self) -> u32 {
        self.pages
    }

    fn is_classic(&self) -> bool {
        self.published_year  String {
        format!("'{}' by {} ({} pages, {})",
                self.title, self.author, self.pages, self.published_year)
    }

    // Calculate estimated reading time (2 minutes per page)
    fn reading_time_minutes(&self) -> u32 {
        self.pages * 2
    }
}

fn main() {
    let book = Book {
        title: String::from("The Rust Programming Language"),
        author: String::from("Steve Klabnik and Carol Nichols"),
        pages: 560,
        published_year: 2018,
    };

    // Can call multiple &self methods
    println!("Title: {}", book.get_title());
    println!("Author: {}", book.get_author());
    println!("Pages: {}", book.page_count());
    println!("Is classic: {}", book.is_classic());
    println!("Summary: {}", book.summary());
    println!("Reading time: {} minutes", book.reading_time_minutes());

    // book is still accessible after all method calls
    println!("Book still available: {:?}", book);
}
```

**Key characteristics of &self:**
- **Non-destructive**: The original instance remains valid after the method call
- **Multiple calls allowed**: You can call many `&self` methods in sequence
- **Read-only access**: Cannot modify the instance's fields
- **Shared borrowing**: Multiple `&self` borrows can exist simultaneously

### When to Use &self

Use `&self` when your method needs to:
- **Read data** from the instance
- **Calculate values** based on instance fields
- **Return information** about the instance
- **Check conditions** or validate data
- **Format or display** the instance

```rust
struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    // Perfect examples of &self usage
    fn area(&self) -> f64 {
        self.width * self.height
    }

    fn perimeter(&self) -> f64 {
        2.0 * (self.width + self.height)
    }

    fn is_square(&self) -> bool {
        self.width == self.height
    }

    fn can_contain(&self, other: &Rectangle) -> bool {
        self.width >= other.width && self.height >= other.height
    }

    fn describe(&self) -> String {
        format!("Rectangle: {}x{} (area: {:.2})",
                self.width, self.height, self.area())
    }
}

fn main() {
    let rect1 = Rectangle { width: 10.0, height: 5.0 };
    let rect2 = Rectangle { width: 3.0, height: 3.0 };

    println!("Rect1 area: {}", rect1.area());
    println!("Rect1 perimeter: {}", rect1.perimeter());
    println!("Is rect2 square? {}", rect2.is_square());
    println!("Can rect1 contain rect2? {}", rect1.can_contain(&rect2));
    println!("{}", rect1.describe());

    // Both rectangles still usable
    println!("rect1: {:?}", rect1);
    println!("rect2: {:?}", rect2);
}
```

## 2. &mut self - Mutable Borrowing

### What is &mut self?

**`&mut self`** is syntactic sugar for `self: &mut Self`. It means the method **borrows** the instance mutably. The method can both read and modify the instance's data, but the instance remains usable after the method call.

### Basic Example

```rust
#[derive(Debug)]
struct Counter {
    value: i32,
    name: String,
}

impl Counter {
    fn new(name: String) -> Self {
        Counter { value: 0, name }
    }

    // Methods using &mut self - can modify the instance
    fn increment(&mut self) {
        self.value += 1;
    }

    fn decrement(&mut self) {
        self.value -= 1;
    }

    fn add(&mut self, amount: i32) {
        self.value += amount;
    }

    fn reset(&mut self) {
        self.value = 0;
        println!("Counter '{}' reset to 0", self.name);
    }

    fn set_value(&mut self, new_value: i32) {
        let old_value = self.value;
        self.value = new_value;
        println!("Counter '{}' changed from {} to {}",
                 self.name, old_value, new_value);
    }

    fn rename(&mut self, new_name: String) {
        self.name = new_name;
    }

    // Read-only methods can still use &self
    fn get_value(&self) -> i32 {
        self.value
    }

    fn get_name(&self) -> &str {
        &self.name
    }
}

fn main() {
    let mut counter = Counter::new("MyCounter".to_string());

    println!("Initial: {:?}", counter);

    // Modify the counter using &mut self methods
    counter.increment();
    counter.increment();
    counter.add(5);

    println!("After modifications: {:?}", counter);

    counter.set_value(100);
    counter.rename("RenamedCounter".to_string());

    println!("After set and rename: {:?}", counter);

    // Can still read using &self methods
    println!("Current value: {}", counter.get_value());
    println!("Current name: {}", counter.get_name());

    counter.reset();
    println!("Final state: {:?}", counter);
}
```

**Key characteristics of &mut self:**
- **Modifying capability**: Can change the instance's fields
- **Exclusive access**: Only one `&mut self` borrow can exist at a time
- **Instance survives**: The original instance remains valid after the method call
- **Requires mut**: The instance must be declared as `mut` to call these methods

### When to Use &mut self

Use `&mut self` when your method needs to:
- **Modify fields** of the instance
- **Update state** based on parameters or calculations
- **Change configuration** or settings
- **Perform operations** that alter the instance

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

    // Methods that modify the account (&mut self)
    fn deposit(&mut self, amount: f64) -> Result {
        if !self.is_active {
            return Err("Account is not active".to_string());
        }

        if amount  Result {
        if !self.is_active {
            return Err("Account is not active".to_string());
        }

        if amount  self.balance {
            return Err("Insufficient funds".to_string());
        }

        self.balance -= amount;
        println!("Withdrew ${:.2}. New balance: ${:.2}", amount, self.balance);
        Ok(amount)
    }

    fn close_account(&mut self) {
        self.is_active = false;
        println!("Account {} closed", self.account_number);
    }

    fn reactivate(&mut self) -> Result {
        if self.balance  f64 {
        self.balance
    }

    fn is_account_active(&self) -> bool {
        self.is_active
    }

    fn get_account_info(&self) -> String {
        format!("Account: {} | Balance: ${:.2} | Status: {}",
                self.account_number,
                self.balance,
                if self.is_active { "Active" } else { "Closed" })
    }
}

fn main() -> Result {
    let mut account = BankAccount::new("ACC123".to_string(), 1000.0);

    println!("Created: {}", account.get_account_info());

    // Perform transactions using &mut self methods
    account.deposit(250.0)?;
    account.withdraw(100.0)?;

    println!("After transactions: {}", account.get_account_info());

    account.close_account();

    // Try to deposit after closing (will fail)
    match account.deposit(50.0) {
        Ok(_) => println!("Deposit successful"),
        Err(e) => println!("Deposit failed: {}", e),
    }

    account.reactivate()?;
    println!("Final state: {}", account.get_account_info());

    Ok(())
}
```

## 3. self - Taking Ownership (Consuming Methods)

### What is self?

**`self`** (without `&`) means the method **takes ownership** of the instance. The method consumes the instance, and after the method call, the original variable can no longer be used. This is typically used for transformations, conversions, or when you want to prevent further use of the object.

### Basic Example

```rust
#[derive(Debug)]
struct Student {
    name: String,
    grade: f64,
    credits: u32,
}

impl Student {
    fn new(name: String, grade: f64, credits: u32) -> Self {
        Student { name, grade, credits }
    }

    // Methods that consume the instance (self)
    fn graduate(self) -> Graduate {
        println!("{} is graduating with grade {:.2}!", self.name, self.grade);
        Graduate {
            name: self.name,
            final_grade: self.grade,
            total_credits: self.credits,
            graduation_year: 2024,
        }
    }

    fn drop_out(self, reason: String) -> DroppedOut {
        println!("{} dropped out: {}", self.name, reason);
        DroppedOut {
            former_name: self.name,
            credits_completed: self.credits,
            reason,
        }
    }

    fn transfer_to_university(self, university: String) -> TransferStudent {
        println!("{} is transferring to {}", self.name, university);
        TransferStudent {
            name: self.name,
            previous_grade: self.grade,
            transfer_credits: self.credits,
            new_university: university,
        }
    }

    // Read-only methods (&self) - don't consume
    fn get_name(&self) -> &str {
        &self.name
    }

    fn get_grade(&self) -> f64 {
        self.grade
    }

    fn is_passing(&self) -> bool {
        self.grade >= 60.0
    }
}

#[derive(Debug)]
struct Graduate {
    name: String,
    final_grade: f64,
    total_credits: u32,
    graduation_year: u32,
}

#[derive(Debug)]
struct DroppedOut {
    former_name: String,
    credits_completed: u32,
    reason: String,
}

#[derive(Debug)]
struct TransferStudent {
    name: String,
    previous_grade: f64,
    transfer_credits: u32,
    new_university: String,
}

fn main() {
    let student1 = Student::new("Alice".to_string(), 85.5, 120);
    let student2 = Student::new("Bob".to_string(), 92.0, 115);
    let student3 = Student::new("Charlie".to_string(), 78.0, 100);

    // Read information before consuming
    println!("Student 1: {} (Grade: {:.1})", student1.get_name(), student1.get_grade());
    println!("Student 2: {} (Grade: {:.1})", student2.get_name(), student2.get_grade());
    println!("Student 3: {} (Grade: {:.1})", student3.get_name(), student3.get_grade());

    // Consume students by calling methods that take ownership
    let graduate = student1.graduate();  // student1 is consumed
    let transfer = student2.transfer_to_university("MIT".to_string());  // student2 is consumed
    let dropout = student3.drop_out("Personal reasons".to_string());  // student3 is consumed

    // student1, student2, student3 are no longer accessible
    // println!("{:?}", student1);  // Error: value borrowed after move

    // But we can use the new objects
    println!("Graduate: {:?}", graduate);
    println!("Transfer: {:?}", transfer);
    println!("Dropout: {:?}", dropout);
}
```

**Key characteristics of self:**
- **Consumes the instance**: The original variable becomes inaccessible
- **Ownership transfer**: The method takes full ownership
- **Transformation**: Often used to convert to a different type
- **Prevention of reuse**: Ensures the original cannot be used after transformation

### When to Use self

Use `self` when your method needs to:
- **Transform** the instance into a different type
- **Consume** the instance as part of the operation
- **Move** the data to prevent further use
- **Convert** or **extract** values from the instance

```rust
#[derive(Debug)]
struct RawData {
    values: Vec,
    timestamp: String,
}

impl RawData {
    fn new(values: Vec, timestamp: String) -> Self {
        RawData { values, timestamp }
    }

    // Consuming methods that transform the data
    fn into_processed_data(self) -> ProcessedData {
        let processed_values: Vec = self.values
            .into_iter()
            .map(|x| x as f64 * 1.5)
            .collect();

        ProcessedData {
            processed_values,
            original_timestamp: self.timestamp,
            processing_timestamp: "2024-01-01".to_string(),
        }
    }

    fn into_summary(self) -> DataSummary {
        let sum: i32 = self.values.iter().sum();
        let count = self.values.len();
        let average = if count > 0 { sum as f64 / count as f64 } else { 0.0 };

        DataSummary {
            original_timestamp: self.timestamp,
            total_count: count,
            sum,
            average,
        }
    }

    fn extract_values(self) -> Vec {
        self.values  // Return just the values, consuming the whole struct
    }

    // Non-consuming methods for comparison
    fn peek_values(&self) -> &[i32] {
        &self.values
    }

    fn count(&self) -> usize {
        self.values.len()
    }
}

#[derive(Debug)]
struct ProcessedData {
    processed_values: Vec,
    original_timestamp: String,
    processing_timestamp: String,
}

#[derive(Debug)]
struct DataSummary {
    original_timestamp: String,
    total_count: usize,
    sum: i32,
    average: f64,
}

fn main() {
    let data1 = RawData::new(vec![1, 2, 3, 4, 5], "2024-01-01T10:00:00".to_string());
    let data2 = RawData::new(vec![10, 20, 30], "2024-01-01T11:00:00".to_string());
    let data3 = RawData::new(vec![100, 200, 300, 400], "2024-01-01T12:00:00".to_string());

    // Peek at data before consuming (using &self methods)
    println!("Data1 has {} values", data1.count());
    println!("Data2 values: {:?}", data2.peek_values());

    // Transform data using consuming methods
    let processed = data1.into_processed_data();  // data1 consumed
    let summary = data2.into_summary();           // data2 consumed
    let just_values = data3.extract_values();     // data3 consumed

    // Original data objects are no longer available
    // println!("{:?}", data1);  // Error: value borrowed after move

    // Use the transformed results
    println!("Processed: {:?}", processed);
    println!("Summary: {:?}", summary);
    println!("Extracted values: {:?}", just_values);
}
```

## Advanced Examples and Patterns

### Builder Pattern with Different Self Types

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

    // Builder methods that consume and return Self (chainable)
    fn header(mut self, name: &str, value: &str) -> Self {
        self.headers.push((name.to_string(), value.to_string()));
        self
    }

    fn body(mut self, body: String) -> Self {
        self.body = Some(body);
        self
    }

    // Final method that consumes builder and returns the built object
    fn build(self) -> HttpRequest {
        HttpRequest {
            method: self.method,
            url: self.url,
            headers: self.headers,
            body: self.body,
        }
    }

    // Read-only inspection methods (&self)
    fn get_method(&self) -> &str {
        &self.method
    }

    fn header_count(&self) -> usize {
        self.headers.len()
    }
}

impl HttpRequest {
    // Convenience constructor
    fn get(url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new("GET", url)
    }

    fn post(url: &str) -> HttpRequestBuilder {
        HttpRequestBuilder::new("POST", url)
    }

    // Methods for the built request
    fn execute(&self) -> Result {
        println!("Executing {} {}", self.method, self.url);
        for (name, value) in &self.headers {
            println!("Header: {}: {}", name, value);
        }
        if let Some(ref body) = self.body {
            println!("Body: {}", body);
        }
        Ok("Request executed successfully".to_string())
    }
}

fn main() {
    // Build requests using the chainable builder pattern
    let get_request = HttpRequest::get("https://api.example.com/users")
        .header("Authorization", "Bearer token123")
        .header("Accept", "application/json")
        .build();  // Consumes the builder

    let post_request = HttpRequest::post("https://api.example.com/users")
        .header("Content-Type", "application/json")
        .body(r#"{"name": "John", "email": "john@example.com"}"#.to_string())
        .build();  // Consumes the builder

    // Execute the requests (using &self)
    match get_request.execute() {
        Ok(response) => println!("GET Response: {}", response),
        Err(e) => println!("GET Error: {}", e),
    }

    match post_request.execute() {
        Ok(response) => println!("POST Response: {}", response),
        Err(e) => println!("POST Error: {}", e),
    }

    // Requests are still usable after execute() because it uses &self
    println!("GET Request: {:?}", get_request);
    println!("POST Request: {:?}", post_request);
}
```

### State Machine with Ownership Transfer

```rust
#[derive(Debug)]
struct Document {
    content: String,
    title: String,
    _state: std::marker::PhantomData,
}

// State markers
#[derive(Debug)]
struct Draft;
#[derive(Debug)]
struct Review;
#[derive(Debug)]
struct Published;

impl Document {
    fn new(title: String, content: String) -> Self {
        Document {
            content,
            title,
            _state: std::marker::PhantomData,
        }
    }

    // Consume draft and return review document
    fn submit_for_review(self) -> Document {
        println!("Submitting '{}' for review", self.title);
        Document {
            content: self.content,
            title: self.title,
            _state: std::marker::PhantomData,
        }
    }

    // Methods available only in draft state (&mut self)
    fn edit_content(&mut self, new_content: String) {
        self.content = new_content;
        println!("Content updated for '{}'", self.title);
    }

    fn change_title(&mut self, new_title: String) {
        let old_title = std::mem::replace(&mut self.title, new_title);
        println!("Title changed from '{}' to '{}'", old_title, self.title);
    }
}

impl Document {
    // Consume review document and return published document
    fn approve_and_publish(self) -> Document {
        println!("Publishing '{}'", self.title);
        Document {
            content: self.content,
            title: self.title,
            _state: std::marker::PhantomData,
        }
    }

    // Consume review document and return draft document
    fn reject_and_return_to_draft(self, feedback: String) -> Document {
        println!("Rejecting '{}' - Feedback: {}", self.title, feedback);
        Document {
            content: self.content,
            title: self.title,
            _state: std::marker::PhantomData,
        }
    }
}

impl Document {
    // Published documents are read-only
    fn get_permalink(&self) -> String {
        format!("https://example.com/docs/{}",
                self.title.to_lowercase().replace(" ", "-"))
    }
}

// Common methods available in all states
impl Document {
    fn get_title(&self) -> &str {
        &self.title
    }

    fn get_content(&self) -> &str {
        &self.content
    }

    fn word_count(&self) -> usize {
        self.content.split_whitespace().count()
    }
}

fn main() {
    // Create a draft document
    let mut draft = Document::new(
        "Rust Ownership Guide".to_string(),
        "This is the initial content about Rust ownership.".to_string()
    );

    // Edit the draft (using &mut self)
    draft.edit_content("This is the updated content about Rust ownership with more details.".to_string());
    draft.change_title("Complete Rust Ownership Guide".to_string());

    println!("Draft word count: {}", draft.word_count());

    // Submit for review (consumes draft)
    let review_doc = draft.submit_for_review();
    // draft is no longer accessible

    println!("Review document title: {}", review_doc.get_title());

    // Approve and publish (consumes review document)
    let published_doc = review_doc.approve_and_publish();
    // review_doc is no longer accessible

    // Use published document (only read operations available)
    println!("Published at: {}", published_doc.get_permalink());
    println!("Final title: {}", published_doc.get_title());
    println!("Final word count: {}", published_doc.word_count());
}
```

## Common Beginner Mistakes and How to Fix Them

### Mistake 1: Using self When You Meant &self

```rust
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    // Wrong: This consumes the point
    // fn distance_from_origin(self) -> f64 {
    //     ((self.x * self.x + self.y * self.y) as f64).sqrt()
    // }

    // Correct: This borrows the point
    fn distance_from_origin(&self) -> f64 {
        ((self.x * self.x + self.y * self.y) as f64).sqrt()
    }
}

fn main() {
    let point = Point { x: 3, y: 4 };

    // With the correct version, we can use point multiple times
    println!("Distance 1: {}", point.distance_from_origin());
    println!("Distance 2: {}", point.distance_from_origin());
    println!("Point is still available: {:?}", point);
}
```

### Mistake 2: Forgetting to Make Instance Mutable

```rust
impl Point {
    // This method needs &mut self to modify the point
    fn move_by(&mut self, dx: i32, dy: i32) {
        self.x += dx;
        self.y += dy;
    }
}

fn main() {
    // Wrong: point is immutable
    // let point = Point { x: 0, y: 0 };
    // point.move_by(1, 1);  // Error!

    // Correct: declare as mutable
    let mut point = Point { x: 0, y: 0 };
    point.move_by(1, 1);  // Works!
    println!("Moved point: ({}, {})", point.x, point.y);
}
```

### Mistake 3: Trying to Use Instance After Consuming Method

```rust
impl Point {
    fn into_tuple(self) -> (i32, i32) {
        (self.x, self.y)
    }
}

fn main() {
    let point = Point { x: 5, y: 10 };

    let tuple = point.into_tuple();  // point is consumed
    println!("Tuple: {:?}", tuple);

    // Wrong: trying to use point after it's consumed
    // println!("Point: {:?}", point);  // Error: value borrowed after move

    // Correct: create a new point if needed
    let new_point = Point { x: 5, y: 10 };
    println!("New point: {:?}", new_point);
}
```

## Decision Guide: Which Receiver to Use?

### Quick Decision Tree

1. **Does your method need to modify the instance?**
   - **Yes** → Use `&mut self`
   - **No** → Continue to step 2

2. **Does your method need to consume/transform the instance?**
   - **Yes** → Use `self`
   - **No** → Use `&self`

3. **Do you want the instance to remain usable after the method call?**
   - **Yes** → Use `&self` or `&mut self`
   - **No** → Use `self`

### Examples by Category

```rust
struct Example {
    value: i32,
    name: String,
}

impl Example {
    // READ-ONLY: Use &self
    fn get_value(&self) -> i32 { self.value }
    fn get_name(&self) -> &str { &self.name }
    fn is_positive(&self) -> bool { self.value > 0 }
    fn description(&self) -> String {
        format!("{}: {}", self.name, self.value)
    }

    // MODIFY: Use &mut self
    fn set_value(&mut self, new_value: i32) { self.value = new_value; }
    fn increment(&mut self) { self.value += 1; }
    fn rename(&mut self, new_name: String) { self.name = new_name; }

    // CONSUME/TRANSFORM: Use self
    fn into_tuple(self) -> (String, i32) { (self.name, self.value) }
    fn into_different_type(self) -> DifferentType {
        DifferentType { data: self.name }
    }
    fn consume_and_print(self) {
        println!("Consuming: {} = {}", self.name, self.value);
    }
}

struct DifferentType {
    data: String,
}
```

## Best Practices for Beginners

### 1. Start with &self by Default

When in doubt, start with `&self`. You can always change it to `&mut self` or `self` if needed:

```rust
impl MyStruct {
    // Start with this
    fn my_method(&self) {
        // If you need to modify fields, change to &mut self
        // If you need to consume the instance, change to self
    }
}
```

### 2. Use Descriptive Method Names

Make it clear from the method name what it does:

```rust
impl User {
    // Clear that these don't modify
    fn get_name(&self) -> &str { &self.name }
    fn is_admin(&self) -> bool { self.is_admin }

    // Clear that these modify
    fn set_name(&mut self, name: String) { self.name = name; }
    fn promote_to_admin(&mut self) { self.is_admin = true; }

    // Clear that these consume
    fn into_profile(self) -> UserProfile { /* ... */ }
    fn destroy_account(self) { /* ... */ }
}
```

### 3. Document Ownership Behavior

```rust
impl Document {
    /// Returns the document title without taking ownership.
    /// The document can be used again after this call.
    fn title(&self) -> &str {
        &self.title
    }

    /// Updates the document content.
    /// Requires a mutable reference to the document.
    fn update_content(&mut self, content: String) {
        self.content = content;
    }

    /// Converts this document into a published version.
    /// This method consumes the document - it cannot be used afterwards.
    fn publish(self) -> PublishedDocument {
        PublishedDocument {
            content: self.content,
            title: self.title,
            published_date: chrono::Utc::now(),
        }
    }
}
```

## Summary and Key Takeaways

### **Essential Rules**

| Receiver | Use When | Instance After Call | Can Modify | Typical Methods |
|----------|----------|-------------------|------------|-----------------|
| **&self** | Reading data | ✅ Still usable | ❌ No | getters, queries, calculations |
| **&mut self** | Modifying data | ✅ Still usable | ✅ Yes | setters, updates, mutations |
| **self** | Consuming/transforming | ❌ No longer usable | ✅ Yes | converters, destructors, builders |

### **Memory Model**

```rust
// &self: Borrowing (shared, immutable)
let result = instance.read_method();  // instance still usable

// &mut self: Borrowing (exclusive, mutable)
instance.modify_method();  // instance still usable but was temporarily borrowed

// self: Moving (ownership transfer)
let new_thing = instance.transform_method();  // instance consumed, no longer usable
```

### **Golden Rules for Beginners**

1. **Default to `&self`** - it's the safest and most common choice
2. **Use `&mut self`** only when you need to modify the instance
3. **Use `self`** only when you want to consume or transform the instance
4. **Make variables `mut`** when calling `&mut self` methods
5. **Understand that `self` methods make the original unusable**
6. **Name methods clearly** to indicate their behavior

Understanding the three receiver types is fundamental to writing idiomatic Rust code. **The choice between `&self`, `&mut self`, and `self` directly impacts the ownership, lifetime, and usability of your data**, so choose wisely based on what your method needs to accomplish!

1. https://www.jerrylsu.net/articles/Rust-self-&self-&mut-self.html
2. https://www.reddit.com/r/rust/comments/xdx1l1/when_should_i_use_selfmut_self_and_when_selfmut/
3. https://users.rust-lang.org/t/what-is-the-difference-between-self-and-self/103969
4. https://stackoverflow.com/questions/59018413/when-to-use-self-self-mut-self-in-methods
5. http://rust-unofficial.github.io/too-many-lists/first-ownership.html
6. https://paytonrules.com/post/immutable-rust-with-mut-self/
7. https://www.linkedin.com/pulse/methods-receiver-rust-amit-nadiger
8. https://users.rust-lang.org/t/what-is-different-between-mut-self-and-mut-self/59708
9. https://mdgsf.github.io/2019/07/26/rust-self/
10. https://comp-rust-2day.rust-edu.org/methods/receiver.html
11. https://users.rust-lang.org/t/method-first-parameter-self-or-self/53646
12. https://news.ycombinator.com/item?id=33822026
13. https://www.possiblerust.com/guide/how-to-read-rust-functions-part-1
14. https://users.rust-lang.org/t/builder-pattern-in-rust-self-vs-mut-self-and-method-vs-associated-function/72892
15. https://doc.rust-lang.org/std/ops/trait.Receiver.html
16. https://users.rust-lang.org/t/calling-self-method-inside-a-mut-self-method/45752
17. https://google.github.io/comprehensive-rust/methods-and-traits/methods.html
18. https://doc.rust-lang.org/book/ch05-03-method-syntax.html
19. https://www.reddit.com/r/learnrust/comments/ums8uw/difference_between_associated_function_and_method/
20. https://doc.rust-lang.org/reference/expressions/method-call-expr.html
