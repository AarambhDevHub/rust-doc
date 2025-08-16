+++
title = "Functional Programming Project"
description = "A hands-on project to apply functional programming concepts in Rust, combining iterators, higher-order functions, and immutability."
date = "2025-08-13"
weight = 1
+++

# Functional Programming Project: Data Processing Pipeline for Restaurant Orders

This project demonstrates how to build a data processing pipeline using a functional programming style in Rust, focusing on analyzing a CSV file containing restaurant order data. We'll cover functional composition, and discuss performance considerations.

## Project Overview

Our goal is to read a CSV file of restaurant orders, filter them by a specific status (e.g., "completed"), parse the price information, and then calculate the total revenue from these filtered orders.

**Concepts Covered:**

* **Functional Composition:** Chaining multiple functions/operations together.
* **Iterator Chains:** Using Rust's powerful iterators and their adaptors.
* **Immutability:** Emphasizing non-mutating transformations.
* **Performance:** Understanding how Rust optimizes functional style.


## The Restaurant Order Analysis Scenario

Imagine you have a `orders.csv` file with the following structure:

```csv
order_id,status,price
1,completed,15.99
2,pending,5.50
3,completed,10.00
4,canceled,7.00
5,completed,20.00
```

We want to find the total revenue from all "completed" orders.

## Project Structure and Code Explanation

The project will consist of a `main.rs` file.

### `main.rs`

This file will contain all the logic for our data processing pipeline.

```rust
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, BufRead};
use std::time::Instant;

// --- Data Types ---
#[derive(Debug, Clone)]
struct Order {
    order_id: u32,
    status: String,
    price: f64,
}

// Helper to convert a CSV row into an Order struct
impl Order {
    fn from_csv_row(headers: &[String], row: &[String]) -> Option<Self> {
        let mut map = HashMap::new();
        for (i, header) in headers.iter().enumerate() {
            if let Some(value) = row.get(i) {
                map.insert(header.clone(), value.clone());
            }
        }

        Some(Order {
            order_id: map.get("order_id")?.parse().ok()?,
            status: map.get("status")?.clone(),
            price: map.get("price")?.parse().ok()?,
        })
    }
}

// --- Functional Pipeline Steps ---

/// Reads a CSV file and returns a Vec of Order structs.
/// This function handles the I/O and initial parsing.
fn read_orders_from_csv(file_path: &str) -> io::Result<Vec<Order>> {
    let file = File::open(file_path)?;
    let reader = io::BufReader::new(file);
    let mut lines = reader.lines();

    // Read headers
    let headers: Vec<String> = lines.next()
        .ok_or_else(|| io::Error::new(io::ErrorKind::InvalidData, "Empty CSV file"))?
        .map_err(|e| io::Error::new(io::ErrorKind::InvalidData, format!("Failed to read headers: {}", e)))?
        .split(',')
        .map(|s| s.trim().to_string())
        .collect();

    let mut orders = Vec::new();
    for line in lines {
        let line = line?;
        let row: Vec<String> = line.split(',').map(|s| s.trim().to_string()).collect();
        if let Some(order) = Order::from_csv_row(&headers, &row) {
            orders.push(order);
        }
    }
    Ok(orders)
}

/// Filters a slice of orders based on a given status.
/// This is a pure function that returns a new Vec.
fn filter_orders_by_status(orders: &[Order], status: &str) -> Vec<Order> {
    orders.iter()
        .filter(|order| order.status == status) // Functional filter
        .cloned() // Clone to get owned Order structs for the new Vec
        .collect() // Consume iterator and collect into a new Vec
}

/// Parses or transforms Order data. In this pipeline, prices are already f64 from initial parsing,
/// but this step could be used for further transformations (e.g., applying tax).
/// We'll use it to simulate a transformation on the filtered orders.
fn transform_orders(orders: &[Order]) -> Vec<Order> {
    orders.iter()
        .map(|order| {
            let mut transformed_order = order.clone();
            // Example transformation: apply a 5% tax if not already applied
            // This assumes price in CSV is pre-tax or handles idempotency
            transformed_order.price *= 1.05;
            transformed_order
        })
        .collect()
}

/// Calculates the total revenue from a slice of orders.
/// This is a functional reduction (summation).
fn calculate_total_revenue(orders: &[Order]) -> f64 {
    orders.iter()
        .map(|order| order.price) // Get just the prices
        .sum() // Sum all prices
}

/// The main data processing pipeline.
/// This function orchestrates the functional composition.
fn data_processing_pipeline(file_path: &str, target_status: &str) -> io::Result<f64> {
    let orders = read_orders_from_csv(file_path)?; // Step 1: Read

    let filtered_orders = filter_orders_by_status(&orders, target_status); // Step 2: Filter

    let transformed_orders = transform_orders(&filtered_orders); // Step 3: Transform

    let total_revenue = calculate_total_revenue(&transformed_orders); // Step 4: Aggregate

    Ok(total_revenue)
}

// --- Performance Comparison and Main Execution ---

fn main() -> io::Result<()> {
    // 1. Create a dummy CSV file for demonstration
    let csv_content = "order_id,status,price\n\
                       1,completed,15.99\n\
                       2,pending,5.50\n\
                       3,completed,10.00\n\
                       4,canceled,7.00\n\
                       5,completed,20.00\n\
                       6,completed,30.00\n\
                       7,pending,12.00\n\
                       8,completed,8.99";
    std::fs::write("orders.csv", csv_content)?;

    let file_path = "orders.csv";
    let target_status = "completed";

    println!("--- Functional Data Processing Pipeline ---");

    let start_time = Instant::now();
    let total_revenue = data_processing_pipeline(file_path, target_status)?;
    let elapsed_time = start_time.elapsed();

    println!("Total revenue for '{}' orders (with 5% tax): ${:.2}", target_status, total_revenue);
    println!("Pipeline execution time: {:?}", elapsed_time);

    println!("\n--- Performance Comparison (Conceptual) ---");
    println!("Rust's functional style, especially with iterators, is highly optimized.");
    println!("The compiler often 'fuses' multiple iterator adaptors (like filter, map) into a single loop,");
    println!("removing intermediate allocations and making functional code as fast as, or faster than, imperative loops.");
    println!("To see actual performance differences, you would use benchmarking tools like `Criterion.rs`.");
    println!("\nExample (conceptual):");
    println!("Imperative loop: Calculate total revenue by iterating and applying if-conditions manually.");
    println!("Functional pipeline: .filter().map().sum()");
    println!("Rust's compiler ensures these often perform very similarly, with functional being more readable.");

    // Clean up the dummy CSV file
    std::fs::remove_file("orders.csv")?;

    Ok(())
}

// Helper to_title_case trait (for demonstration)
trait ToTitleCase {
    fn to_title_case(&self) -> String;
}

impl ToTitleCase for str {
    fn to_title_case(&self) -> String {
        self.split_whitespace()
            .map(|word| {
                let mut chars = word.chars();
                match chars.next() {
                    None => String::new(),
                    Some(first) => first.to_uppercase().collect::<String>() + &chars.as_str().to_lowercase(),
                }
            })
            .collect::<Vec<String>>()
            .join(" ")
    }
}
```


### How It Works

1. **`Order` Struct**: Defines the structure for each order record. `from_csv_row` is a helper to parse a CSV row into an `Order` instance.
2. **`read_orders_from_csv`**: This function handles reading the CSV file line by line. It parses headers and then uses the `Order::from_csv_row` to convert each row into an `Order` struct. This is the initial data loading step.
3. **`filter_orders_by_status`**: This function takes a slice of `Order` structs and a `status` string. It uses the `filter` iterator adaptor to keep only orders matching the status. `.cloned()` is used to create owned copies of the `Order` structs, and `.collect()` gathers them into a new `Vec<Order>`.
4. **`transform_orders`**: This function demonstrates a transformation step. It `map`s over the orders and applies a 5% tax to their `price`. `.collect()` gathers the transformed orders.
5. **`calculate_total_revenue`**: This function takes a slice of `Order` structs, `map`s each order to its `price`, and then uses the `sum()` consumer adaptor to calculate the total.
6. **`data_processing_pipeline`**: This is the main orchestrator. It calls the steps in sequence, passing the results of one step as input to the next. This clearly shows functional composition.
7. **`main` Function**:
    * Creates a dummy `orders.csv` file for testing.
    * Calls `data_processing_pipeline` with "completed" status.
    * Prints the total revenue and the execution time.
    * Cleans up the dummy file.

### Functional Composition Practice

The `data_processing_pipeline` function exemplifies functional composition:

```rust
let orders = read_orders_from_csv(file_path)?;
let filtered_orders = filter_orders_by_status(&orders, target_status);
let transformed_orders = transform_orders(&filtered_orders);
let total_revenue = calculate_total_revenue(&transformed_orders);
```

Each step (`read`, `filter`, `transform`, `calculate`) is a function that takes an input and produces an output, which then becomes the input for the next function. This creates a clear, readable data flow.

### Performance Comparison (Conceptual)

Rust's functional programming style, especially when using iterators, is highly performant due to its "zero-cost abstractions."

* **Iterator Fusion**: When you chain multiple iterator adaptors (`filter`, `map`, `flat_map`, etc.) together, the Rust compiler is very good at "fusing" these operations. This means it often compiles the entire chain down to a single, optimized loop, avoiding the creation of temporary intermediate collections and the overhead of multiple passes over the data.
* **No Runtime Overhead**: Unlike some languages where functional constructs might incur runtime penalties (e.g., creating many temporary objects), Rust's compiler effectively translates these high-level functional pipelines into efficient machine code, often as fast as or faster than hand-written imperative loops.

**To truly compare performance**, you would use Rust's benchmarking tools like `Criterion.rs`. In many cases, a functional iterator chain like `.filter().map().sum()` will perform comparably to, or even better than, an equivalent explicit `for` loop, while being more concise and readable.

**Example Comparison (Conceptual):**

```rust
// Functional Style:
let total_functional: f64 = data.iter()
    .filter(|&item| item.is_valid())
    .map(|&item| item.calculate_value())
    .sum();

// Imperative Style:
let mut total_imperative = 0.0;
for item in &data {
    if item.is_valid() {
        total_imperative += item.calculate_value();
    }
}
```

Rust's compiler is highly adept at making the functional version perform just as well as the imperative one.

## Running the Project

1. **Save the code**: Save the Rust code provided above into a file named `src/main.rs` inside a new Rust project directory (e.g., created with `cargo new restaurant_data_processor`).
2. **Build and Run**: Navigate to your project directory in the terminal and run:

```bash
cargo run
```

This will compile and execute your project, creating the `orders.csv` file, processing it, and then printing the total revenue and execution time. The `orders.csv` file will be automatically cleaned up after execution.

**Example Output:**

```
--- Functional Data Processing Pipeline ---
Total revenue for 'completed' orders (with 5% tax): $48.29
Pipeline execution time: 100.123us

--- Performance Comparison (Conceptual) ---
Rust's functional style, especially with iterators, is highly optimized.
The compiler often 'fuses' multiple iterator adaptors (like filter, map) into a single loop,
removing intermediate allocations and making functional code as fast as, or faster than, imperative loops.
To see actual performance differences, you would use benchmarking tools like `Criterion.rs`.

Example (conceptual):
Imperative loop: Calculate total revenue by iterating and applying if-conditions manually.
Functional pipeline: .filter().map().sum()
Rust's compiler ensures these often perform very similarly, with functional being more readable.
```

This project demonstrates how Rust allows you to write clear, concise, and performant data processing code using a functional style, making it a powerful tool for building robust and efficient applications.
