+++
title = "Day 5: Practice & Review"
description = "Building small Rust programs, doing code reviews, answering common questions, and tackling a realistic weekend CLI assignment to reinforce concepts."
date = 2025-08-01
draft = false
weight = 1
+++

# Day 5 (Friday): Practice & Review

# Rust Learning: Building Small Programs, Code Review, Q\&A, and Weekend Assignment (In Detail)

You've built a solid foundation in Rust's syntax and major concepts. The logical next step is to move from exercises to **building small programs, practicing code review, addressing real coding questions, and tackling a structured weekend assignment**. Here’s a detailed guide for each aspect:

## 1. Building Small Programs

### General Approach

* **Start Simple:** Target programs you understand end-to-end (calculator, todo-list, mini blog, quiz, alarm clock, etc.).
* **Incremental Development:** Write in small steps; compile and run after each addition.
* **Leverage Cargo:** Always use `cargo new` for project setup—gain experience with Rust’s project workflow.

### Example Mini-Projects

#### **A. Tip Calculator**

```rust
use std::io;

fn main() {
    println!("Enter the bill amount:");
    let mut bill = String::new();
    io::stdin().read_line(&mut bill).unwrap();
    let bill: f64 = bill.trim().parse().unwrap();

    println!("Enter desired tip %:");
    let mut tip = String::new();
    io::stdin().read_line(&mut tip).unwrap();
    let tip: f64 = tip.trim().parse().unwrap();

    let tip_amount = bill * tip / 100.0;
    let total = bill + tip_amount;

    println!("Tip: ₹{:.2}, Total: ₹{:.2}", tip_amount, total);
}
```

#### **B. Word Counter**

```rust
fn main() {
    let text = "rust is safe fast and memory efficient";
    let count = text.split_whitespace().count();
    println!("There are {} words.", count);
}
```

#### **C. Simple To-Do List**

* Store tasks in a `Vec<String>`, loop input, print, mark as done, and quit on command.

## 2. Code Review Sessions

### How To Do A Review

* **Read the code line by line:** Try to explain it in your own words.
* **Check for idiomatic Rust:** Prefer slices, iterators, safe error handling, immutable-by-default.
* **Comment on:** Variable naming, function decomposition, repetition, and possible panics.
* **Ask:** Could this be written more simply? Are there unchecked unwraps? Is input handling robust?
* **Example Feedback:**
  * “Consider using `if let Some(x) = ...` instead of `unwrap()` to handle empty cases.”
  * "Move repeated parsing logic into a helper function."

### Sample Code Review Table

| Line(s) | Feedback                                                          |
| ------- | ----------------------------------------------------------------- |
| 5-7     | `unwrap()` on parse: Ok for demo, but use `match` for production. |
| 12-15   | The logic for summing can use `.iter().sum()` for conciseness.    |

**Practice:**
Review code you wrote earlier (say, the quiz, or vector exercises). Try to find:

* At least one possible panic or error case
* One place for clearer names or less repetition
* Anything you’d do differently with your new knowledge

## 3. Q\&A and Problem-Solving

### Common Coding Questions

* **“Why does Rust require explicit type annotations here?”**
  * Because type inference needs context—like empty collections or parse ops.
* **“What does ‘borrow’ mean in a compiler error?”**
  * You're passing a reference to data. The compiler ensures you don’t have simultaneous mutable and immutable references for safety.
* **“How do I print all even numbers from a vector in one line?”**

```rust
let nums = vec![1, 2, 3, 4, 5, 6];
let evens: Vec<_> = nums.iter().filter(|x| **x % 2 == 0).collect();
println!("{:?}", evens);
```

* **“How can I clean up repeated code for input parsing?”**
  * Write a helper function that reads a line and parses it into any type with generics.

### Problem-Solving Session

Try to *verbalize your confusion*. For example:

* “I get a ‘move occurs’ error—I tried to use my String after passing it to a function.”
* “My code panics on invalid input—how do I prevent this?”

**Approach:**

* Reproduce the bug/error.
* Check compiler error messages closely—they usually tell you where the problem is.
* Look up terms ("move", "borrow", "lifetime") in the Rust Book.
* Reduce your problem to the smallest example that triggers it.

## 4. Weekend Assignment

### Suggested Structure

> **Goal:** Reinforce skills in reading, writing, and refactoring code + deepen understanding of Rust patterns.

**Assignment Prompt:**

> Build a *Personal Expense Tracker* CLI tool.**Bonus:** Persist data to a file (JSON/CSV), optionally re-loads on startup.

**Features to Practice:**

* Struct Definitions
* Vec Operations
* User Input Parsing
* Command Loops (add, list, total, filter, quit)
* Data Formatting and Printing
* Error Handling

### Sample CLI Flow

```
Welcome to Expense Tracker
Commands:
a - Add new expense
l - List all expenses
f - Filter by min/max
t - Show total
q - Quit

a
> Description? Taxi
> Amount? 180
> Date? 2025-08-01
Expense added!

t
Total spent: ₹180

l
1. [2025-08-01] Taxi: ₹180

q
Goodbye!
```

### Beginner Steps

1. Start with basic structures:

```rust
struct Expense {
    desc: String,
    amount: f64,
    date: String,
}
```

1. Use a vector:
   `let mut expenses: Vec<Expense> = Vec::new();`
2. Prompt for user input, parse, push to vector.
3. Implement "list", "total", "filter" commands, printing formatted output.

## Tips for All Sections

* **Always test with invalid and missing input.**
* **Read error messages fully—Rust’s are verbose but extremely helpful.**
* **Use the&#x20;****[Rust Playground](https://play.rust-lang.org/)****&#x20;to share or experiment.**
* **Write comments about what you’re trying or where you’re unsure.**
* **Seek code review from a peer, mentor, or on forums.**

## Recap Table

| Activity             | What You’ll Practice                                                |
| -------------------- | ------------------------------------------------------------------- |
| Small Programs       | Project structure, function & variable use, control flow, ownership |
| Code Review Session  | Readability, idioms, bug detection, safe code, teamwork             |
| Q\&A/Problem-Solving | Debugging, compiler errors, deeper concept exploration              |
| Weekend Assignment   | Integration of all skills, tackling realistic Rust projects         |

**This structure mirrors real developer workflows.** Don’t hesitate to ask for help on any block or concept—Rust’s learning curve is steep, but your foundation is now strong!
