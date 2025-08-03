+++
title = "Practical Rust Exercises"
description = "Hands-on Rust exercises that reinforce core concepts like variables, control flow, functions, and data structures through realistic programming tasks."
date = 2025-08-01
draft = false
weight = 5
tags = ["rust", "exercises", "practice", "beginner", "functions", "control flow", "compound types"]
+++

# Practical Rust Exercises: Applying Core Concepts

Hands-on practice is essential for mastering Rust. Below are **progressive, detailed exercises** drawn from the topics you've learned so far. Each one is designed to solidify your understanding of variables, types, control flow, functions, compound data types, and more.

## 1. Variables and Mutability

**Task:**
Write a program that declares an immutable variable for the user's age, tries to increment it, observes the compiler error, and then correctly increments it by making it mutable.

```rust
fn main() {
    let mut age = 30;
    age += 1;
    println!("Your new age is: {}", age);
}
```

## 2. User Input and Output

**Task:**
Prompt the user to enter their name and age, parse the age as an integer, and print a personalized greeting.

```rust
use std::io;

fn main() {
    let mut name = String::new();
    println!("Enter your name:");
    io::stdin().read_line(&mut name).expect("Failed to read line");
    let name = name.trim();

    let mut age_input = String::new();
    println!("Enter your age:");
    io::stdin().read_line(&mut age_input).expect("Failed to read line");
    let age: i32 = age_input.trim().parse().expect("Please enter a number");

    println!("Hello, {}! Next year, you'll be {}.", name, age + 1);
}
```

## 3. Functions and Return Values

**Task:**
Write a function that computes the area of a rectangle. Pass length and width as parameters and return the result.

```rust
fn area(length: f64, width: f64) -> f64 {
    length * width
}

fn main() {
    let a = area(4.0, 3.0);
    println!("Area: {}", a);
}
```

## 4. Conditional Logic and Expressions

**Task:**
Write a function that takes an integer and returns whether it is positive, negative, or zero using an if/else if/else expression.

```rust
fn classify(n: i32) -> &'static str {
    if n > 0 {
        "Positive"
    } else if n < 0 {
        "Negative"
    } else {
        "Zero"
    }
}

fn main() {
    println!("{}", classify(5));
    println!("{}", classify(-7));
    println!("{}", classify(0));
}
```

## 5. Match Expressions

**Task:**
Write a match expression to map numbers 1-7 to the days of the week. Use `_` as a fallback for invalid input.

```rust
fn day_name(day: u8) -> &'static str {
    match day {
        1 => "Monday",
        2 => "Tuesday",
        3 => "Wednesday",
        4 => "Thursday",
        5 => "Friday",
        6 => "Saturday",
        7 => "Sunday",
        _ => "Invalid day"
    }
}

fn main() {
    for day in 0..9 {
        println!("{}: {}", day, day_name(day));
    }
}
```

## 6. Loops and Iteration

**Task:**
Write three loops:

* A `loop` that keeps asking the user for their favorite number until they type "42".
* A `while` loop that prints numbers from 10 down to 1.
* A `for` loop over a vector of city names.

```rust
use std::io;

fn main() {
    // Infinite loop with break
    loop {
        println!("What's your favorite number?");
        let mut input = String::new();
        io::stdin().read_line(&mut input).unwrap();
        if input.trim() == "42" {
            println!("That's the answer to life, the universe, and everything!");
            break;
        }
    }

    // While loop
    let mut n = 10;
    while n > 0 {
        println!("{}", n);
        n -= 1;
    }
    println!("Liftoff!");

    // For loop
    let cities = vec!["Surat", "Ahmedabad", "Vadodara"];
    for city in &cities {
        println!("City: {}", city);
    }
}
```

## 7. Tuples and Arrays

**Task:**
Create a tuple representing a product (name, price, stock) and print each field. Then, create an array of five test scores, compute and print the average.

```rust
fn main() {
    let product = ("Pen", 13.5, 100);
    println!("Product: {}, Price: {}, Stock: {}", product.0, product.1, product.2);

    let scores = [90, 92, 87, 89, 94];
    let sum: i32 = scores.iter().sum();
    let avg = sum as f64 / scores.len() as f64;
    println!("Average score: {}", avg);
}
```

## 8. String vs \&str

**Task:**
Write a function that takes a `&str` parameter, converts it to a `String`, and returns the string in uppercase.

```rust
fn shout(text: &str) -> String {
    text.to_uppercase()
}

fn main() {
    let greeting = "hello, rust!";
    let loud = shout(greeting);
    println!("{}", loud);
}
```

## 9. Vector Manipulation

**Task:**
Read numbers from the user until they input 0, store them in a vector, then print the sum and all even numbers.

```rust
use std::io;

fn main() {
    let mut numbers = Vec::new();
    loop {
        println!("Enter a number (0 to finish):");
        let mut input = String::new();
        io::stdin().read_line(&mut input).unwrap();
        let n: i32 = input.trim().parse().unwrap();
        if n == 0 { break; }
        numbers.push(n);
    }

    let sum: i32 = numbers.iter().sum();
    let even: Vec<i32> = numbers.iter().cloned().filter(|x| x % 2 == 0).collect();
    println!("Sum: {}", sum);
    println!("Even numbers: {:?}", even);
}
```

## 10. Type Annotations and Inference

**Task:**
Create an empty vector with an explicit type annotation and fill it with squares of the first ten natural numbers.

```rust
fn main() {
    let mut squares: Vec<u32> = Vec::new();
    for i in 1..=10 {
        squares.push(i * i);
    }
    println!("Squares: {:?}", squares);
}
```

## Tips for Practice

* **Modify and extend** each program; e.g., add more error-handling, different data types, new print statements.
* Try **refactoring**: turn repeated code sections into functions.
* **Comment out** lines and observe compilation errors to learn about type rules and borrowing.
* Use the **Rust Playground** online to test programs instantly (play.rust-lang.org).

These exercises reflect authentic, real-world programming in Rust and reinforce the safety, performance, and expressiveness that set it apart from other languages.
