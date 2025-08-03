+++
title = "Code Refactoring Exercises"
description = "Practice improving Rust code by applying ownership principles through hands-on refactoring exercises focused on clarity, safety, and efficiency."
date = 2025-08-03
draft = false
weight = 3
+++

# Code Refactoring Exercises for Rust: Comprehensive Documentation

Code refactoring in Rust is a critical skill that transforms working but suboptimal code into clean, idiomatic, and maintainable Rust. This comprehensive guide provides detailed exercises progressing from basic improvements to advanced architectural refactoring, focusing on Rust-specific patterns and ownership principles.

## Fundamentals of Rust Code Refactoring

### What Makes Rust Refactoring Unique

**Rust's ownership system** provides unique advantages and challenges for refactoring:
- **Compile-time safety** ensures refactored code maintains memory safety
- **Type system guidance** helps identify necessary changes during refactoring
- **Zero-cost abstractions** allow high-level refactoring without performance penalties
- **Ownership constraints** may require architectural changes during refactoring[1][2]

### Core Refactoring Principles in Rust

1. **Maintain behavioral equivalence** while improving code structure
2. **Leverage the type system** to guide refactoring decisions
3. **Embrace Rust idioms** like iterators, pattern matching, and error handling
4. **Use the compiler** as a refactoring assistant
5. **Test continuously** to ensure correctness throughout the process

## Exercise Level 1: Basic Rust Idiom Refactoring

### Exercise 1.1: Loop to Iterator Transformation

**Scenario:** Transform C-style loops into idiomatic Rust iterators.

**Before (Non-idiomatic):**
```rust
// Bad: C-style loop with manual indexing
fn sum_positive_numbers(numbers: &Vec) -> i32 {
    let mut sum = 0;
    for i in 0..numbers.len() {
        if numbers[i] > 0 {
            sum += numbers[i];
        }
    }
    sum
}

// Bad: Manual iteration with while loop
fn find_first_even(numbers: &Vec) -> Option {
    let mut i = 0;
    while i  i32 {
    numbers
        .iter()
        .filter(|&&num| num > 0)
        .sum()
}

// Good: Iterator methods for searching
fn find_first_even(numbers: &[i32]) -> Option {
    numbers
        .iter()
        .find(|&&num| num % 2 == 0)
        .copied()
}

// Advanced: More complex transformation
fn process_data(numbers: &[i32]) -> Vec {
    numbers
        .iter()
        .enumerate()
        .filter(|(_, &num)| num > 0)
        .map(|(idx, &num)| format!("Item {}: {}", idx, num * 2))
        .collect()
}
```

**Key Improvements:**
- **Performance:** Iterator chains are zero-cost abstractions
- **Readability:** Declarative style shows intent clearly
- **Safety:** No bounds checking needed with iterators
- **Composability:** Easy to chain additional operations

### Exercise 1.2: Error Handling Refactoring

**Scenario:** Transform primitive error handling into Result-based patterns.

**Before (Primitive Error Handling):**
```rust
// Bad: Using tuples for success/failure
fn divide_numbers(a: f64, b: f64) -> (f64, bool) {
    if b == 0.0 {
        (0.0, false) // Error indicated by boolean
    } else {
        (a / b, true)
    }
}

// Bad: Using Option when specific error info needed
fn parse_and_validate(input: &str) -> Option {
    match input.parse::() {
        Ok(num) if num > 0 => Some(num),
        _ => None, // Loss of error information
    }
}

// Bad: Panic-heavy error handling
fn read_file_content(filename: &str) -> String {
    std::fs::read_to_string(filename)
        .expect("File should exist and be readable")
}
```

**After (Idiomatic Error Handling):**
```rust
use std::fmt;

// Good: Custom error types with proper Error trait implementation
#[derive(Debug)]
enum MathError {
    DivisionByZero,
    InvalidInput(String),
}

impl fmt::Display for MathError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            MathError::DivisionByZero => write!(f, "Division by zero is not allowed"),
            MathError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
        }
    }
}

impl std::error::Error for MathError {}

// Good: Result-based error handling
fn divide_numbers(a: f64, b: f64) -> Result {
    if b == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(a / b)
    }
}

// Good: Detailed error information preserved
fn parse_and_validate(input: &str) -> Result {
    let num = input.parse::()
        .map_err(|_| MathError::InvalidInput(format!("Cannot parse '{}' as integer", input)))?;

    if num  Result> {
    let content = std::fs::read_to_string(filename)?;
    let processed = content.trim().to_uppercase();
    Ok(processed)
}

// Example usage showing error handling flow
fn example_usage() -> Result> {
    let result = divide_numbers(10.0, 2.0)?;
    println!("Division result: {}", result);

    let validated = parse_and_validate("42")?;
    println!("Validated number: {}", validated);

    match read_and_process_file("config.txt") {
        Ok(content) => println!("File content: {}", content),
        Err(e) => eprintln!("Error reading file: {}", e),
    }

    Ok(())
}
```

## Exercise Level 2: Ownership and Borrowing Refactoring

### Exercise 2.1: Unnecessary Cloning Elimination

**Scenario:** Eliminate unnecessary clones and moves to improve performance.

**Before (Inefficient Ownership):**
```rust
// Bad: Excessive cloning
#[derive(Clone, Debug)]
struct Student {
    name: String,
    grades: Vec,
    id: u32,
}

impl Student {
    // Bad: Cloning when borrowing would suffice
    fn calculate_average(student: Student) -> f64 {
        let total: f64 = student.grades.iter().sum();
        total / student.grades.len() as f64
    }

    // Bad: Unnecessary cloning in processing
    fn process_students(students: Vec) -> Vec {
        let mut results = Vec::new();
        for student in students {
            let cloned_student = student.clone(); // Unnecessary
            let avg = Self::calculate_average(student);
            results.push(format!("{}: {:.2}", cloned_student.name, avg));
        }
        results
    }
}

// Bad: String concatenation with excessive allocation
fn format_student_list(students: &Vec) -> String {
    let mut result = String::new();
    for student in students {
        // Creates temporary string for each iteration
        result = result + &format!("ID: {}, Name: {}\n", student.id, student.name);
    }
    result
}
```

**After (Efficient Ownership):**
```rust
#[derive(Clone, Debug)]
struct Student {
    name: String,
    grades: Vec,
    id: u32,
}

impl Student {
    // Good: Borrowing instead of taking ownership
    fn calculate_average(&self) -> f64 {
        if self.grades.is_empty() {
            return 0.0;
        }
        let total: f64 = self.grades.iter().sum();
        total / self.grades.len() as f64
    }

    // Good: Processing with references
    fn process_students(students: &[Student]) -> Vec {
        students
            .iter()
            .map(|student| {
                let avg = student.calculate_average();
                format!("{}: {:.2}", student.name, avg)
            })
            .collect()
    }

    // Good: Zero-copy operations where possible
    fn find_top_student(students: &[Student]) -> Option {
        students
            .iter()
            .max_by(|a, b| {
                a.calculate_average()
                    .partial_cmp(&b.calculate_average())
                    .unwrap_or(std::cmp::Ordering::Equal)
            })
    }
}

// Good: Efficient string building
fn format_student_list(students: &[Student]) -> String {
    students
        .iter()
        .map(|student| format!("ID: {}, Name: {}", student.id, student.name))
        .collect::>()
        .join("\n")
}

// Alternative: Using write! for even more efficiency
use std::fmt::Write;

fn format_student_list_optimized(students: &[Student]) -> String {
    let mut result = String::with_capacity(students.len() * 50); // Pre-allocate
    for student in students {
        writeln!(result, "ID: {}, Name: {}", student.id, student.name)
            .expect("Writing to string should never fail");
    }
    result
}
```

### Exercise 2.2: Smart Pointer Optimization

**Scenario:** Refactor code to use appropriate smart pointers for different ownership patterns.

**Before (Suboptimal Memory Management):**
```rust
use std::collections::HashMap;

// Bad: Overuse of Box when not needed
struct DataManager {
    data: Box>,
    cache: Box>,
    config: Box,
}

#[derive(Clone)]
struct Configuration {
    max_size: usize,
    timeout: u64,
}

impl DataManager {
    fn new() -> Self {
        DataManager {
            data: Box::new(Vec::new()),
            cache: Box::new(HashMap::new()),
            config: Box::new(Configuration {
                max_size: 1000,
                timeout: 30,
            }),
        }
    }

    // Bad: Cloning boxed data unnecessarily
    fn get_config_copy(&self) -> Configuration {
        (*self.config).clone()
    }
}

// Bad: Shared ownership without proper reference counting
struct SharedResource {
    name: String,
    data: Vec,
}

fn share_resource_badly() {
    let resource = SharedResource {
        name: "shared".to_string(),
        data: vec![1, 2, 3, 4, 5],
    };

    // This won't compile due to move semantics
    // let user1 = process_resource(resource);
    // let user2 = process_resource(resource); // Error: use of moved value
}
```

**After (Optimal Smart Pointer Usage):**
```rust
use std::sync::{Arc, Mutex};
use std::rc::Rc;
use std::collections::HashMap;

// Good: Use direct ownership for simple cases
struct DataManager {
    data: Vec,                    // Direct ownership
    cache: HashMap,       // Direct ownership
    config: Configuration,                // Direct ownership for small structs
}

#[derive(Clone, Debug)]
struct Configuration {
    max_size: usize,
    timeout: u64,
}

impl DataManager {
    fn new() -> Self {
        DataManager {
            data: Vec::new(),
            cache: HashMap::new(),
            config: Configuration {
                max_size: 1000,
                timeout: 30,
            },
        }
    }

    // Good: Return reference when possible
    fn get_config(&self) -> &Configuration {
        &self.config
    }

    // Good: Clone only when necessary
    fn get_config_copy(&self) -> Configuration {
        self.config.clone()
    }
}

// Good: Appropriate smart pointer usage for shared ownership
#[derive(Debug)]
struct SharedResource {
    name: String,
    data: Vec,
}

// Single-threaded shared ownership
fn share_resource_single_threaded() {
    let resource = Rc::new(SharedResource {
        name: "shared".to_string(),
        data: vec![1, 2, 3, 4, 5],
    });

    let user1 = resource.clone(); // Increment reference count
    let user2 = resource.clone(); // Increment reference count

    println!("Resource used by multiple owners: {}", user1.name);
    println!("Reference count: {}", Rc::strong_count(&resource));
}

// Multi-threaded shared ownership
fn share_resource_multi_threaded() {
    let resource = Arc::new(SharedResource {
        name: "shared".to_string(),
        data: vec![1, 2, 3, 4, 5],
    });

    let handles: Vec = (0..3)
        .map(|i| {
            let resource_clone = resource.clone();
            std::thread::spawn(move || {
                println!("Thread {} accessing: {}", i, resource_clone.name);
            })
        })
        .collect();

    for handle in handles {
        handle.join().unwrap();
    }
}

// Good: Shared mutable state with proper synchronization
struct SharedCounter {
    value: Arc>,
}

impl SharedCounter {
    fn new() -> Self {
        SharedCounter {
            value: Arc::new(Mutex::new(0)),
        }
    }

    fn increment(&self) {
        let mut value = self.value.lock().unwrap();
        *value += 1;
    }

    fn get(&self) -> u32 {
        *self.value.lock().unwrap()
    }

    fn clone_handle(&self) -> Self {
        SharedCounter {
            value: self.value.clone(),
        }
    }
}
```

## Exercise Level 3: Architectural Refactoring

### Exercise 3.1: Monolithic Function Decomposition

**Scenario:** Break down a large, monolithic function into smaller, focused components.

**Before (Monolithic Function):**
```rust
use std::collections::HashMap;
use std::fs;
use std::io;

// Bad: One massive function doing everything
fn process_student_data(filename: &str) -> Result> {
    // File reading
    let content = fs::read_to_string(filename)?;

    // Parsing
    let mut students = Vec::new();
    for line in content.lines() {
        let parts: Vec = line.split(',').collect();
        if parts.len() != 4 {
            continue;
        }
        let name = parts[0].trim();
        let id: u32 = parts[1].trim().parse()?;
        let grade1: f64 = parts[2].trim().parse()?;
        let grade2: f64 = parts[3].trim().parse()?;

        students.push((name.to_string(), id, vec![grade1, grade2]));
    }

    // Processing and analysis
    let mut grade_stats = HashMap::new();
    let mut total_average = 0.0;
    let mut student_count = 0;

    for (name, id, grades) in &students {
        let average = grades.iter().sum::() / grades.len() as f64;
        total_average += average;
        student_count += 1;

        let letter_grade = if average >= 90.0 {
            'A'
        } else if average >= 80.0 {
            'B'
        } else if average >= 70.0 {
            'C'
        } else if average >= 60.0 {
            'D'
        } else {
            'F'
        };

        grade_stats.insert(*id, (average, letter_grade));

        // Reporting
        println!("Student: {} (ID: {})", name, id);
        println!("  Grades: {:?}", grades);
        println!("  Average: {:.2}", average);
        println!("  Letter Grade: {}", letter_grade);

        if average ,
}

impl Student {
    fn new(name: String, id: u32, grades: Vec) -> Self {
        Student { name, id, grades }
    }

    fn calculate_average(&self) -> f64 {
        if self.grades.is_empty() {
            0.0
        } else {
            self.grades.iter().sum::() / self.grades.len() as f64
        }
    }

    fn get_letter_grade(&self) -> LetterGrade {
        LetterGrade::from_average(self.calculate_average())
    }

    fn is_failing(&self) -> bool {
        self.calculate_average()  Self {
        match average {
            a if a >= 90.0 => LetterGrade::A,
            a if a >= 80.0 => LetterGrade::B,
            a if a >= 70.0 => LetterGrade::C,
            a if a >= 60.0 => LetterGrade::D,
            _ => LetterGrade::F,
        }
    }
}

impl fmt::Display for LetterGrade {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            LetterGrade::A => write!(f, "A"),
            LetterGrade::B => write!(f, "B"),
            LetterGrade::C => write!(f, "C"),
            LetterGrade::D => write!(f, "D"),
            LetterGrade::F => write!(f, "F"),
        }
    }
}

// Good: Separate error handling
#[derive(Debug)]
enum StudentDataError {
    IoError(std::io::Error),
    ParseError(String),
    InvalidFormat(String),
}

impl fmt::Display for StudentDataError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            StudentDataError::IoError(e) => write!(f, "IO error: {}", e),
            StudentDataError::ParseError(msg) => write!(f, "Parse error: {}", msg),
            StudentDataError::InvalidFormat(msg) => write!(f, "Invalid format: {}", msg),
        }
    }
}

impl std::error::Error for StudentDataError {}

impl From for StudentDataError {
    fn from(error: std::io::Error) -> Self {
        StudentDataError::IoError(error)
    }
}

// Good: Focused parsing functionality
struct StudentDataParser;

impl StudentDataParser {
    fn parse_file(filename: &str) -> Result, StudentDataError> {
        let content = fs::read_to_string(filename)?;
        Self::parse_content(&content)
    }

    fn parse_content(content: &str) -> Result, StudentDataError> {
        content
            .lines()
            .filter(|line| !line.trim().is_empty())
            .enumerate()
            .map(|(line_num, line)| Self::parse_line(line, line_num + 1))
            .collect()
    }

    fn parse_line(line: &str, line_number: usize) -> Result {
        let parts: Vec = line.split(',').map(|s| s.trim()).collect();

        if parts.len() != 4 {
            return Err(StudentDataError::InvalidFormat(
                format!("Line {}: Expected 4 fields, got {}", line_number, parts.len())
            ));
        }

        let name = parts[0].to_string();
        let id = parts[1].parse::()
            .map_err(|_| StudentDataError::ParseError(
                format!("Line {}: Invalid student ID '{}'", line_number, parts[1])
            ))?;

        let grade1 = parts[2].parse::()
            .map_err(|_| StudentDataError::ParseError(
                format!("Line {}: Invalid grade '{}'", line_number, parts[2])
            ))?;

        let grade2 = parts[3].parse::()
            .map_err(|_| StudentDataError::ParseError(
                format!("Line {}: Invalid grade '{}'", line_number, parts[3])
            ))?;

        Ok(Student::new(name, id, vec![grade1, grade2]))
    }
}

// Good: Statistical analysis component
struct StudentAnalyzer;

impl StudentAnalyzer {
    fn calculate_statistics(students: &[Student]) -> StudentStatistics {
        if students.is_empty() {
            return StudentStatistics::default();
        }

        let total_average = students
            .iter()
            .map(|s| s.calculate_average())
            .sum::() / students.len() as f64;

        let grade_distribution = students
            .iter()
            .map(|s| s.get_letter_grade())
            .fold(HashMap::new(), |mut acc, grade| {
                *acc.entry(grade).or_insert(0) += 1;
                acc
            });

        let failing_count = students
            .iter()
            .filter(|s| s.is_failing())
            .count();

        StudentStatistics {
            total_students: students.len(),
            overall_average: total_average,
            grade_distribution,
            failing_count,
        }
    }
}

#[derive(Debug, Default)]
struct StudentStatistics {
    total_students: usize,
    overall_average: f64,
    grade_distribution: HashMap,
    failing_count: usize,
}

// Good: Separate reporting component
struct StudentReporter;

impl StudentReporter {
    fn print_individual_reports(students: &[Student]) {
        println!("📋 Individual Student Reports:");
        for student in students {
            Self::print_student_report(student);
        }
    }

    fn print_student_report(student: &Student) {
        println!("\nStudent: {} (ID: {})", student.name, student.id);
        println!("  Grades: {:?}", student.grades);
        println!("  Average: {:.2}", student.calculate_average());
        println!("  Letter Grade: {}", student.get_letter_grade());

        if student.is_failing() {
            println!("  ⚠️  Student is failing!");
        }
    }

    fn print_summary_statistics(stats: &StudentStatistics) {
        println!("\n📊 Summary Statistics:");
        println!("  Total Students: {}", stats.total_students);

        if stats.total_students > 0 {
            println!("  Overall Average: {:.2}", stats.overall_average);
            println!("  Failing Students: {}", stats.failing_count);

            println!("  Grade Distribution:");
            for grade in [LetterGrade::A, LetterGrade::B, LetterGrade::C, LetterGrade::D, LetterGrade::F] {
                let count = stats.grade_distribution.get(&grade).unwrap_or(&0);
                println!("    {}: {}", grade, count);
            }
        }
    }
}

// Good: Clean, focused main function
fn process_student_data(filename: &str) -> Result {
    let students = StudentDataParser::parse_file(filename)?;
    let statistics = StudentAnalyzer::calculate_statistics(&students);

    StudentReporter::print_individual_reports(&students);
    StudentReporter::print_summary_statistics(&statistics);

    Ok(())
}

// Usage example
fn main() -> Result> {
    process_student_data("students.csv")?;
    Ok(())
}
```

**Key Improvements:**
- **Single Responsibility:** Each component has one clear purpose
- **Testability:** Individual components can be unit tested
- **Maintainability:** Changes to parsing don't affect reporting
- **Error Handling:** Proper error types with context
- **Reusability:** Components can be used independently

## Exercise Level 4: Design Pattern Refactoring

### Exercise 4.1: State Pattern Implementation

**Scenario:** Refactor a state machine using enums and pattern matching instead of inheritance.

**Before (Procedural State Management):**
```rust
// Bad: State management with flags and conditional logic
struct DocumentProcessor {
    content: String,
    is_draft: bool,
    is_reviewing: bool,
    is_approved: bool,
    is_published: bool,
    reviewer_comments: Vec,
    approval_date: Option,
}

impl DocumentProcessor {
    fn new(content: String) -> Self {
        DocumentProcessor {
            content,
            is_draft: true,
            is_reviewing: false,
            is_approved: false,
            is_published: false,
            reviewer_comments: Vec::new(),
            approval_date: None,
        }
    }

    // Bad: Complex conditional logic for state transitions
    fn submit_for_review(&mut self) -> Result {
        if self.is_draft && !self.is_reviewing && !self.is_approved && !self.is_published {
            self.is_draft = false;
            self.is_reviewing = true;
            Ok(())
        } else {
            Err("Cannot submit for review from current state".to_string())
        }
    }

    fn approve(&mut self, approval_date: String) -> Result {
        if !self.is_draft && self.is_reviewing && !self.is_approved && !self.is_published {
            self.is_reviewing = false;
            self.is_approved = true;
            self.approval_date = Some(approval_date);
            Ok(())
        } else {
            Err("Cannot approve from current state".to_string())
        }
    }

    fn publish(&mut self) -> Result {
        if !self.is_draft && !self.is_reviewing && self.is_approved && !self.is_published {
            self.is_published = true;
            Ok(())
        } else {
            Err("Cannot publish from current state".to_string())
        }
    }

    // Bad: State checking scattered throughout
    fn add_comment(&mut self, comment: String) -> Result {
        if self.is_reviewing {
            self.reviewer_comments.push(comment);
            Ok(())
        } else {
            Err("Can only add comments during review".to_string())
        }
    }

    fn get_status(&self) -> String {
        if self.is_published {
            "Published".to_string()
        } else if self.is_approved {
            "Approved".to_string()
        } else if self.is_reviewing {
            "Under Review".to_string()
        } else if self.is_draft {
            "Draft".to_string()
        } else {
            "Unknown".to_string()
        }
    }
}
```

**After (Enum-Based State Machine):**
```rust
use chrono::{DateTime, Utc};

// Good: Explicit state representation with associated data
#[derive(Debug, Clone)]
enum DocumentState {
    Draft,
    UnderReview {
        submitted_at: DateTime,
        comments: Vec
    },
    Approved {
        approved_at: DateTime,
        approved_by: String
    },
    Published {
        published_at: DateTime
    },
    Rejected {
        rejected_at: DateTime,
        reason: String
    },
}

#[derive(Debug)]
struct DocumentProcessor {
    content: String,
    state: DocumentState,
    version: u32,
    author: String,
}

// Good: Custom error type for state transition errors
#[derive(Debug)]
struct StateTransitionError {
    from_state: String,
    attempted_action: String,
    message: String,
}

impl std::fmt::Display for StateTransitionError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Cannot {} from {} state: {}",
               self.attempted_action, self.from_state, self.message)
    }
}

impl std::error::Error for StateTransitionError {}

impl DocumentProcessor {
    fn new(content: String, author: String) -> Self {
        DocumentProcessor {
            content,
            state: DocumentState::Draft,
            version: 1,
            author,
        }
    }

    // Good: Clean state transition methods
    fn submit_for_review(mut self) -> Result {
        match self.state {
            DocumentState::Draft => {
                self.state = DocumentState::UnderReview {
                    submitted_at: Utc::now(),
                    comments: Vec::new(),
                };
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "submit for review".to_string(),
                message: "Can only submit drafts for review".to_string(),
            })
        }
    }

    fn add_review_comment(mut self, comment: String) -> Result {
        match &mut self.state {
            DocumentState::UnderReview { comments, .. } => {
                comments.push(comment);
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "add comment".to_string(),
                message: "Can only add comments during review".to_string(),
            })
        }
    }

    fn approve(mut self, approved_by: String) -> Result {
        match self.state {
            DocumentState::UnderReview { .. } => {
                self.state = DocumentState::Approved {
                    approved_at: Utc::now(),
                    approved_by,
                };
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "approve".to_string(),
                message: "Can only approve documents under review".to_string(),
            })
        }
    }

    fn reject(mut self, reason: String) -> Result {
        match self.state {
            DocumentState::UnderReview { .. } => {
                self.state = DocumentState::Rejected {
                    rejected_at: Utc::now(),
                    reason,
                };
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "reject".to_string(),
                message: "Can only reject documents under review".to_string(),
            })
        }
    }

    fn publish(mut self) -> Result {
        match self.state {
            DocumentState::Approved { .. } => {
                self.state = DocumentState::Published {
                    published_at: Utc::now(),
                };
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "publish".to_string(),
                message: "Can only publish approved documents".to_string(),
            })
        }
    }

    fn revise_content(mut self, new_content: String) -> Result {
        match self.state {
            DocumentState::Draft | DocumentState::Rejected { .. } => {
                self.content = new_content;
                self.version += 1;
                self.state = DocumentState::Draft;
                Ok(self)
            }
            _ => Err(StateTransitionError {
                from_state: self.get_state_name(),
                attempted_action: "revise content".to_string(),
                message: "Can only revise drafts or rejected documents".to_string(),
            })
        }
    }

    // Good: Pattern matching for state queries
    fn get_state_name(&self) -> String {
        match &self.state {
            DocumentState::Draft => "Draft".to_string(),
            DocumentState::UnderReview { .. } => "Under Review".to_string(),
            DocumentState::Approved { .. } => "Approved".to_string(),
            DocumentState::Published { .. } => "Published".to_string(),
            DocumentState::Rejected { .. } => "Rejected".to_string(),
        }
    }

    fn get_review_comments(&self) -> Option {
        match &self.state {
            DocumentState::UnderReview { comments, .. } => Some(comments),
            _ => None,
        }
    }

    fn is_published(&self) -> bool {
        matches!(self.state, DocumentState::Published { .. })
    }

    fn can_edit(&self) -> bool {
        matches!(self.state, DocumentState::Draft | DocumentState::Rejected { .. })
    }

    // Good: Comprehensive state information
    fn get_state_info(&self) -> StateInfo {
        match &self.state {
            DocumentState::Draft => StateInfo {
                name: "Draft".to_string(),
                description: "Document is being authored".to_string(),
                available_actions: vec!["submit_for_review", "revise_content"],
                timestamp: None,
            },
            DocumentState::UnderReview { submitted_at, comments } => StateInfo {
                name: "Under Review".to_string(),
                description: format!("Document submitted for review ({} comments)", comments.len()),
                available_actions: vec!["add_comment", "approve", "reject"],
                timestamp: Some(*submitted_at),
            },
            DocumentState::Approved { approved_at, approved_by } => StateInfo {
                name: "Approved".to_string(),
                description: format!("Document approved by {}", approved_by),
                available_actions: vec!["publish"],
                timestamp: Some(*approved_at),
            },
            DocumentState::Published { published_at } => StateInfo {
                name: "Published".to_string(),
                description: "Document is live".to_string(),
                available_actions: vec![],
                timestamp: Some(*published_at),
            },
            DocumentState::Rejected { rejected_at, reason } => StateInfo {
                name: "Rejected".to_string(),
                description: format!("Document rejected: {}", reason),
                available_actions: vec!["revise_content"],
                timestamp: Some(*rejected_at),
            },
        }
    }
}

#[derive(Debug)]
struct StateInfo {
    name: String,
    description: String,
    available_actions: Vec,
    timestamp: Option>,
}

// Usage example showing clean API
fn demonstrate_document_workflow() -> Result> {
    let mut doc = DocumentProcessor::new(
        "Initial document content".to_string(),
        "Alice".to_string(),
    );

    println!("Created document in {} state", doc.get_state_name());

    // Submit for review
    doc = doc.submit_for_review()?;
    println!("Document submitted for review");

    // Add review comments
    doc = doc.add_review_comment("Looks good overall".to_string())?;
    doc = doc.add_review_comment("Minor formatting issues".to_string())?;

    if let Some(comments) = doc.get_review_comments() {
        println!("Review comments: {:?}", comments);
    }

    // Approve the document
    doc = doc.approve("Bob".to_string())?;
    println!("Document approved, ready for publishing");

    // Publish
    doc = doc.publish()?;
    println!("Document published!");

    println!("Final state: {:?}", doc.get_state_info());

    Ok(())
}
```

## Exercise Level 5: Advanced Performance Refactoring

### Exercise 5.1: Zero-Copy and Allocation Optimization

**Scenario:** Optimize string processing to minimize allocations and copies.

**Before (Allocation-Heavy Processing):**
```rust
use std::collections::HashMap;

// Bad: Excessive string allocation and copying
struct TextProcessor {
    stop_words: Vec,
    word_counts: HashMap,
}

impl TextProcessor {
    fn new() -> Self {
        TextProcessor {
            stop_words: vec![
                "the".to_string(), "a".to_string(), "an".to_string(),
                "and".to_string(), "or".to_string(), "but".to_string(),
                "in".to_string(), "on".to_string(), "at".to_string(),
            ],
            word_counts: HashMap::new(),
        }
    }

    // Bad: Multiple unnecessary allocations
    fn process_text(&mut self, text: &str) -> Vec {
        let mut results = Vec::new();

        // Bad: Converting everything to lowercase creates new strings
        let lower_text = text.to_lowercase();

        // Bad: Split creates temporary strings
        let words: Vec = lower_text
            .split_whitespace()
            .map(|word| {
                // Bad: Removing punctuation creates new strings
                word.chars()
                    .filter(|c| c.is_alphabetic())
                    .collect::()
            })
            .collect();

        for word in words {
            // Bad: Multiple string comparisons and clones
            if !self.stop_words.contains(&word) && word.len() > 2 {
                *self.word_counts.entry(word.clone()).or_insert(0) += 1;
                results.push(word);
            }
        }

        results
    }

    // Bad: Creating formatted strings for every entry
    fn get_report(&self) -> String {
        let mut report = String::new();
        let mut sorted_words: Vec = self.word_counts.iter().collect();
        sorted_words.sort_by(|a, b| b.1.cmp(a.1));

        for (word, count) in sorted_words {
            let line = format!("{}: {}\n", word, count);
            report.push_str(&line);
        }

        report
    }
}
```

**After (Zero-Copy Optimized):**
```rust
use std::collections::HashMap;
use std::borrow::Cow;

// Good: Use string slices and avoid unnecessary allocations
struct TextProcessor {
    stop_words: &'static [&'static str], // Static data, no allocation needed
    word_counts: HashMap<String, usize>,
}

impl TextProcessor {
    fn new() -> Self {
        TextProcessor {
            stop_words: &[
                "the", "a", "an", "and", "or", "but",
                "in", "on", "at", "is", "are", "was", "were"
            ],
            word_counts: HashMap::new(),
        }
    }

    // Good: Zero-copy text processing where possible
    fn process_text<'a>(&mut self, text: &'a str) -> Vec<Cow<'a, str>> {
        text.split_whitespace()
            .filter_map(|word| self.process_word(word))
            .collect()
    }

    // Good: Process individual words with minimal allocation
    fn process_word<'a>(&mut self, word: &'a str) -> Option<Cow<'a, str>> {
        if word.len() <= 2 {
            return None;
        }

        // Check if word needs cleaning (has non-alphabetic characters)
        let needs_cleaning = word.chars().any(|c| !c.is_alphabetic());

        let cleaned_word = if needs_cleaning {
            // Only allocate when necessary
            Cow::Owned(
                word.chars()
                    .filter(|c| c.is_alphabetic())
                    .collect::<String>()
                    .to_lowercase()
            )
        } else {
            // Use original word if possible, convert case only if needed
            if word.chars().any(|c| c.is_uppercase()) {
                Cow::Owned(word.to_lowercase())
            } else {
                Cow::Borrowed(word)
            }
        };

        // Efficient stop word checking
        if self.is_stop_word(&cleaned_word) {
            return None;
        }

        // Update word count (this may require allocation for the key)
        let word_string = cleaned_word.to_string();
        *self.word_counts.entry(word_string).or_insert(0) += 1;

        Some(cleaned_word)
    }

    // Good: Efficient stop word checking
    fn is_stop_word(&self, word: &str) -> bool {
        self.stop_words.binary_search(&word).is_ok() // Requires sorted stop_words
        // Alternative: self.stop_words.contains(&word) for unsorted
    }

    // Good: Efficient report generation
    fn get_report(&self) -> String {
        if self.word_counts.is_empty() {
            return "No words processed yet.".to_string();
        }

        let mut sorted_words: Vec<_> = self.word_counts.iter().collect();
        sorted_words.sort_unstable_by(|a, b| b.1.cmp(a.1).then_with(|| a.0.cmp(b.0)));

        // Pre-allocate string with estimated capacity
        let estimated_size = sorted_words.len() * 20; // Rough estimate
        let mut report = String::with_capacity(estimated_size);

        for (word, count) in sorted_words {
            use std::fmt::Write;
            writeln!(report, "{}: {}", word, count)
                .expect("Writing to string should never fail");
        }

        report
    }

    // Good: Zero-copy text analysis
    fn analyze_text(&self, text: &str) -> TextAnalysis {
        let mut char_count = 0;
        let mut word_count = 0;
        let mut sentence_count = 0;

        for line in text.lines() {
            char_count += line.chars().count();
            word_count += line.split_whitespace().count();
            sentence_count += line.matches(&['.', '!', '?'][..]).count();
        }

        TextAnalysis {
            char_count,
            word_count,
            sentence_count,
            line_count: text.lines().count(),
            avg_words_per_sentence: if sentence_count > 0 {
                word_count as f64 / sentence_count as f64
            } else {
                0.0
            },
        }
    }
}

#[derive(Debug)]
struct TextAnalysis {
    char_count: usize,
    word_count: usize,
    sentence_count: usize,
    line_count: usize,
    avg_words_per_sentence: f64,
}

// Advanced: Custom iterator for memory-efficient processing
struct WordIterator<'a> {
    text: &'a str,
    position: usize,
}

impl<'a> WordIterator<'a> {
    fn new(text: &'a str) -> Self {
        WordIterator { text, position: 0 }
    }
}

impl<'a> Iterator for WordIterator<'a> {
    type Item = &'a str;

    fn next(&mut self) -> Option<Self::Item> {
        // Skip whitespace
        while self.position < self.text.len() {
            if !self.text.chars().nth(self.position)?.is_whitespace() {
                break;
            }
            self.position += 1;
        }

        if self.position >= self.text.len() {
            return None;
        }

        let start = self.position;

        // Find end of word
        while self.position < self.text.len() {
            if self.text.chars().nth(self.position)?.is_whitespace() {
                break;
            }
            self.position += 1;
        }

        Some(&self.text[start..self.position])
    }
}

// Usage example showing memory efficiency
fn demonstrate_efficient_processing() {
    let mut processor = TextProcessor::new();

    let sample_text = r#"
        The quick brown fox jumps over the lazy dog.
        This is a sample text for demonstrating efficient processing.
        Memory allocation should be minimized where possible.
    "#;

    // Process text efficiently
    let words = processor.process_text(sample_text);
    println!("Processed {} unique words", words.len());

    // Analyze without additional allocation
    let analysis = processor.analyze_text(sample_text);
    println!("Analysis: {:?}", analysis);

    // Generate report
    let report = processor.get_report();
    println!("Word frequency report:\n{}", report);
}
```
## Key Performance Improvements
- **Zero-copy operations using** `Cow<str>` and string slices[1]
- **Reduced allocations** by avoiding unnecessary string creation[2]
- **Efficient data structures** with pre-allocated capacity
- **Custom iterators** for memory-efficient processing
- **Compile-time optimization** through static data usage

## Best Practices Summary

### **Refactoring Workflow**

1. **Identify Code Smells**
   - Long functions (>50 lines)
   - Repeated code patterns
   - Excessive nesting
   - Poor error handling
   - Inefficient ownership patterns

2. **Plan Refactoring Steps**
   - Write comprehensive tests first
   - Identify module boundaries
   - Plan ownership and borrowing changes
   - Consider performance implications

3. **Execute Incrementally**
   - Make small, focused changes
   - Test after each modification
   - Use compiler guidance
   - Maintain git history for rollbacks

4. **Validate Improvements**
   - Run performance benchmarks
   - Verify test coverage
   - Check for regression issues
   - Review code with team

### **Rust-Specific Considerations**

- **Leverage the type system** to guide refactoring decisions[3]
- **Use the borrow checker** as a design constraint, not a limitation
- **Prefer borrowing over cloning** unless ownership transfer is needed
- **Design APIs around references** for maximum flexibility
- **Use appropriate smart pointers** for different sharing patterns
- **Apply zero-cost abstractions** to maintain performance during refactoring

These refactoring exercises demonstrate how to transform suboptimal Rust code into idiomatic, efficient, and maintainable solutions. The key is understanding Rust's unique features and designing code that works with the ownership system rather than against it.

1. https://www.reddit.com/r/learnrust/comments/18ffdz7/refactor_rust_exercises/
2. https://betterprogramming.pub/rust-refactoring-for-beginners-15a3270ce45d
3. https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html
4. https://understandlegacycode.com/blog/5-coding-exercises-to-practice-refactoring-legacy-code
5. https://www.youtube.com/watch?v=Abb52stbFOo
6. https://www.netsolutions.com/insights/what-is-code-refactoring/
7. https://codenotary.com/blog/step-by-step-guide-refactoring-a-large-rust-codebase-with-aiderdev-and-custom-llms
8. https://blog.frankel.ch/start-rust/10/
9. https://www.reddit.com/r/rust/comments/17jeezg/so_is_rust_easy_to_refactor_or_bad_to_refactor/
10. https://martinfowler.com/articles/2024-refactoring-code-samples.html
11. https://users.rust-lang.org/t/we-need-a-great-general-refactoring-tool/103985
12. https://www.manning.com/books/refactoring-to-rust
13. https://rust-exercises.com
14. https://dev.to/ietxaniz/rust-concurrency-cleaning-code-with-traits-and-simplifying-services-4lha
15. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch12-03-improving-error-handling-and-modularity.html
16. https://refactoring.guru/design-patterns/rust
17. https://www.youtube.com/watch?v=wuBkzT_3CDU
18. https://users.rust-lang.org/t/difficult-long-refactoring-suggestions/30900
19. https://www.bellwether-softworks.com/blog/graph-notes-03/
20. https://refactoring.guru/refactoring
21. https://www.aalpha.net/blog/what-you-should-know-about-code-refactoring-in-agile/
22. https://users.rust-lang.org/t/best-practices-to-write-rust-code/110040
23. https://www.designisrefactoring.com/2016/08/31/refactoring-rust-primitive-obsession/
24. https://www.codewars.com/kata/search/rust
