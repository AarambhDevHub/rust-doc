+++
title = "Function Syntax and Parameters in Rust"
description = "A complete guide to defining and using functions in Rust, including parameters, return values, visibility, and idiomatic patterns with lifetimes, generics, and references."
date = 2025-08-01
draft = false
weight = 1
tags = ["rust", "functions", "syntax", "parameters", "generics", "lifetimes"]
+++

# Function syntax and parameters

Functions in Rust are declared with the fn keyword, list their (typed) parameters in parentheses, and—if they return a value—specify its type after an arrow (->). Inside the curly braces lies the function body, whose final expression (no semicolon) becomes the return value[1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html)[4](https://doc.rust-lang.org/rust-by-example/fn.html).

## Basic syntax

```rust
fn function_name(param1: Type1, param2: Type2) -> ReturnType {
    // body
    expression_without_semicolon   // implicitly returned
}

```

* **snake\_case names** are idiomatic for functions and variables[1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html).
* Omitting -> … makes the return type the unit type () (void)[4](https://doc.rust-lang.org/rust-by-example/fn.html).

## Parameters

1. **Mandatory type annotations**
   &#x20;Every parameter is written as name: Type; Rust never infers parameter types[4](https://doc.rust-lang.org/rust-by-example/fn.html).
2. **Pattern parameters**
   &#x20;You can destructure tuples, structs, or references directly:

```rust
fn swap((a, b): (i32, i32)) -> (i32, i32) { (b, a) }
```

1. **Borrowed and mutable references**
   &#x20;Passing \&T (or \&mut T) lets a function read (or mutate) data without taking ownership:

```rust
fn push(v: &mut Vec<i32>, value: i32) { v.push(value); }
```

*(Rust’s borrowing rules apply; only one mutable borrow at a time.)*

1. **Generic parameters and trait bounds**

```rust
fn max<T: PartialOrd>(a: T, b: T) -> T { if a > b { a } else { b } }
```

Angle-bracket syntax declares type parameters; T: PartialOrd is a **trait bound** restricting what T must implement.

1. **Lifetime parameters**
   &#x20;Needed when returned references could come from multiple inputs:

```rust
fn first<'a>(s: &'a str, _t: &'a str) -> &'a str { s }
```

1. **Variadic parameters**
   &#x20;Rust itself has no native variadics, but you can declare `extern "C"` interfaces to C vararg functions (e.g., printf).

## Return values

* The last expression is returned implicitly; add a semicolon to suppress return.
* Use `return expr;` anywhere for early exit[4](https://doc.rust-lang.org/rust-by-example/fn.html).
* Multiple values can be returned with tuples.

```rust
fn stats(data: &[i32]) -> (i32, i32) { (*data.iter().min().unwrap(), *data.iter().max().unwrap()) }
```

## Function visibility & placement

* Add **pub** in front of fn to make a function visible outside the current module.
* Functions can be declared in any order; the compiler doesn’t require forward declarations[11](https://doc.rust-lang.org/stable/book/ch03-03-how-functions-work.html).

## Limitations compared with some languages

* No default parameter values—use Option\<T> or builder patterns instead.
* No function overloading by signature; use different names or traits.

## Idiomatic patterns

* Use short helper functions for readable pipelines:

```rust
fn main() {
    let number = "42";
    let n = parse(number);
    println!("{}", double(n));
}

fn parse(s: &str) -> i32 { s.parse().unwrap() }
fn double(x: i32) -> i32 { x * 2 }

```

* Return `Result<T, E>` for fallible operations to integrate with the `?` operator.

```rust
fn read_file(path: &str) -> std::io::Result<String> {
    std::fs::read_to_string(path)
}

```

Understanding these rules gives you full control over Rust’s function system—from simple helpers to generic, lifetime-parameterised APIs.

1. https://doc.rust-lang.org/book/ch03-03-how-functions-work.html
2. https://www.programiz.com/rust/function
3. https://www.w3schools.com/rust/rust\_functions.php
4. https://doc.rust-lang.org/rust-by-example/fn.html
5. https://itsfoss.com/rust-functions/
6. https://www.codecademy.com/learn/rust-for-programmers/modules/functions-rust/cheatsheet
7. https://doc.rust-lang.org/stable/book/functions.html
8. https://web.mit.edu/rust-lang\_v1.25/arch/amd64\_ubuntu1404/share/doc/rust/html/book/second-edition/ch03-03-how-functions-work.html
9. https://web.mit.edu/rust-lang\_v1.25/arch/amd64\_ubuntu1404/share/doc/rust/html/book/first-edition/functions.html
10. https://rust-book.cs.brown.edu/ch03-03-how-functions-work.html
11. https://doc.rust-lang.org/stable/book/ch03-03-how-functions-work.html
