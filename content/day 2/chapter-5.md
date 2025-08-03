+++
title = "Basic Input/Output"
description = "A hands-on guide to using Rust's standard input and output: println!, stdin, Read, Write, and common patterns for building interactive CLI tools."
date = 2025-08-01
draft = false
weight = 5
+++

# Basic input/output

Rust’s standard library offers a **minimal-but-powerful I/O toolkit** centered on three ideas:

* the `println!`/`print!` family of *macros* for formatted **output**
* the `std::io` module’s `stdin()` handle (plus `Read`/`BufRead` traits) for **input**
* the `Write` trait for low-level control of any destination that can accept bytes, including `stdout()` and `stderr()`

Below is a thorough tour of the essentials, plus idiomatic patterns you’ll meet every day.

## 1. Printing to the console

1. `println!` adds a newline; `print!` does not.`println!` ultimately writes to `io::stdout()` via the `Write` trait[8](https://doc.rust-lang.org/stable/std/io/index.html).
   ```rust
   println!("Hello, {}!", name);   // trailing '\n'
   print!("Processing...");         // needs manual flush
   std::io::stdout().flush()?;      // force characters out

   ```
2. Debug vs. Display formatting
   * `{}` uses the `Display` trait – implement for user-facing text
   * `{:?}` or `{:#?}` use the `Debug` trait – convenient for structs/enums
   ```rust
   println!("Point = {:?}", point);      // compressed
   println!("Point = {:#?}", point);     // pretty-printed

   ```
3. Other convenience macros
   * `eprint!` / `eprintln!` write to `stderr` (useful for progress bars, errors)
   * `format!` builds a `String` in memory instead of printing.

## 2. Reading from standard input

1. One-line interactive read`read_line` appends bytes to `buffer` and stops at `\n` or EOF[1](https://doc.rust-lang.org/std/io/index.html).
   ```rust
   use std::io;

   let mut buffer = String::new();
   io::stdin().read_line(&mut buffer)?;          // returns Result<usize>[1]
   let trimmed = buffer.trim();                  // removes trailing '\n'

   ```
2. Parsing numbers (compile-time safety)
   ```rust
   let n: i32 = trimmed.parse()                 // &str → i32
                      .expect("Please type a number!");

   ```
3. Locked, buffered reads (efficient loops)The `lock()` call avoids repeated internal mutex traffic on every read[2](https://doc.rust-lang.org/std/io/fn.stdin.html).
   ```rust
   use std::io::{self, BufRead};

   let stdin = io::stdin();          // Grab handle
   for line in stdin.lock().lines()  // BufRead::lines iterator[2]
   {
       let line = line?;             // io::Result<String>
       println!("Echo: {}", line);
   }

   ```
4. Reading all STDIN at once
   ```rust
   use std::io::{self, Read};

   let mut data = String::new();
   io::stdin().read_to_string(&mut data)?;       // whole stream[7]

   ```

## 3. Low-level writing with `Write`

If you need more control than the convenience macros provide:

```rust
use std::io::{self, Write};

let mut out = io::stdout();          // Stdout implements Write[8]
out.write_all(b"raw bytes")?;        // no UTF-8 check
out.write_fmt(format_args!(" value = {}\n", n))?;

```

Any type that implements `Write`—files, sockets, in-memory buffers—works with `write!`, `writeln!`, or manual `write_all`.

## 4. Common error-handling patterns

* Use `?` to bubble up I/O errors; make `main` return `io::Result<()>`.

```rust
fn main() -> std::io::Result<()> {
    // I/O calls with ? operator
    Ok(())
}

```

* In quick demos, `unwrap()`/`expect()` is acceptable but panics on error.

## 5. Putting it together ─ tiny calculator

```rust
use std::io::{self, Write};

fn main() -> io::Result<()> {
    print!("Enter two integers: ");
    io::stdout().flush()?;          // ensure prompt is visible

    let mut line = String::new();
    io::stdin().read_line(&mut line)?;

    let mut nums = line.split_whitespace()
                       .map(|s| s.parse::<i32>()
                                  .expect("Need integers"));
    let a = nums.next().unwrap();
    let b = nums.next().unwrap();

    println!("{} + {} = {}", a, b, a + b);
    Ok(())
}

```

## 6. Best practices checklist

* **Use&#x20;****`println!`****/****`print!`****&#x20;for quick output; switch to&#x20;****`Write`****&#x20;when you need explicit flushing or are writing somewhere other than the terminal.**
* **Always trim the newline after&#x20;****`read_line`****&#x20;and perform a&#x20;****`parse()`****&#x20;for numeric input.**
* **Lock&#x20;****`stdin`****&#x20;or&#x20;****`stdout`****&#x20;inside tight loops for performance.**
* **Return&#x20;****`io::Result`****&#x20;from&#x20;****`main`****&#x20;to keep&#x20;****`?`****&#x20;available everywhere.**

With these patterns you can build robust command-line utilities while staying squarely in safe, idiomatic Rust.

1. https://doc.rust-lang.org/std/io/index.html
2. https://doc.rust-lang.org/std/io/fn.stdin.html
3. https://www.geeksforgeeks.org/rust/standard-i-o-in-rust/
4. https://www.tutorialspoint.com/rust/rust\_input\_output.htm
5. https://www.cs.brandeis.edu/\~cs146a/rust/doc-02-21-2015/book/standard-input.html
6. https://dev.to/nikhilraojl/taking-an-input-and-printing-stuff-in-rust-jnl
7. https://dev.to/jkreeftmeijer/testing-input-and-output-in-rust-command-line-applications-56p5
8. https://doc.rust-lang.org/stable/std/io/index.html
9. https://alexeden.github.io/learning-rust/programming\_rust/18\_input\_and\_output.html
10. https://doc.rust-lang.org/std/io/struct.Stdin.html
