---
title: "Exit Codes and Error Handling in CLI"
description: "Learn best practices for returning exit codes and handling errors in Rust CLI programs."
date: 2025-11-03
weight: 4
---

# Exit Codes and Error Handling in Rust CLI Applications: Best Practices

Error handling and exit codes are fundamental for building robust command-line (CLI) tools in Rust. This ensures users and scripts can reliably detect, respond to, and debug problems. Rust’s idioms help you write programs that return clear exit status codes—using `Result`, custom error enums, and strategic use of crates for rich error reporting.

## The Practical Analogy: Running a Restaurant Kitchen

- **A successful meal**: The kitchen hands back a perfect dish. In CLI, this means an exit code of **0**: success.
- **A minor mistake**: Maybe you’re out of an ingredient, but the order can be retried or given a clear error message. Exit code **1** or another specific code.
- **A disaster**: The kitchen catches on fire! In CLI, this might be a panic or crash (Rust sets an exit code of 101 if the program panics).[^1]

***

## Foundation: Exit Codes

- **Exit code 0**: Standard for success.
- **Non-zero exit code**: Any failure or error, with `1` for general errors. Other codes can communicate particular failure reasons.
- **Rust panics**: By default, set exit code **101**.
- **On Unix and Windows**: Only 0–255 are portable exit codes.[^1]


### How to Set Exit Codes in Rust

1. **Returning `Result` from `main()`** (Rust 1.26+):

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() -> Result<(), io::Error> {
    let path = "example.txt";
    let contents = read_file(path)?;
    println!("File contents:\n{}", contents);
    Ok(())
}
```

    - If `main` returns `Ok(())`, exit code is 0.
    - If it returns `Err(..)`, Rust prints the error, exit code is 1.[^2]
2. **Custom exit codes with `std::process::exit()`**:

```rust
use std::process;

fn main() {
    if let Err(e) = do_work() {
        eprintln!("Error: {}", e);
        process::exit(2); // Custom error code
    }
}
```

3. **Returning rich exit codes with crates**:
    - Use the [`exitcode`](https://docs.rs/exitcode/) crate for commonly used, standardized exit codes (e.g., config error, not found, usage error).[^1]

```rust
use exitcode;

fn main() {
    // ...
    match run_cli() {
        Ok(_) => std::process::exit(exitcode::OK),
        Err(MyCliError::Config(_)) => std::process::exit(exitcode::CONFIG),
        Err(_) => std::process::exit(exitcode::DATAERR),
    }
}
```


***

## Idiomatic Error Handling in Rust CLI Programs

### 1. **Favor `Result<T, E>` over `panic!`**

- **Use panics only for unrecoverable bugs**, not for user-facing errors or input failures.[^3][^4]
- Propagate errors up to `main()` and act on them—print a message, set exit code.


### 2. **Define Custom Error Types**

- Use enums with variants for each error kind, and derive `Debug`, `Display`, and (optionally) `thiserror::Error` for good printouts.[^4]

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum CliError {
    #[error("Config error: {0}")]
    Config(String),
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Other error: {0}")]
    Other(String),
}
```


### 3. **Return Errors from a Library Layer**

- Structure CLI apps so `main()` just calls a function like `run() -> Result<(), CliError>`. This function does all the work, letting you handle errors and exit codes in one place.[^5][^4]


### 4. **Clear and Actionable Error Messages**

- Use `eprintln!()` for errors, and make messages user-friendly.
- Consider colorful errors with crates like [`color-eyre`] or tracking error chains with [`anyhow`].[^5]

***

## Example: A Complete, Idiomatic CLI with Exit Codes

```rust
use std::{fs::File, io::{self, Read}, process::exit};
use thiserror::Error;

#[derive(Error, Debug)]
enum CliError {
    #[error("Could not read config file: {0}")]
    Config(#[from] io::Error),
    #[error("Config is empty or invalid")]
    EmptyConfig,
}

fn read_config(path: &str) -> Result<String, CliError> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    if contents.trim().is_empty() {
        return Err(CliError::EmptyConfig);
    }
    Ok(contents)
}

fn run() -> Result<(), CliError> {
    let contents = read_config("config.toml")?;
    println!("Config loaded: {}", contents);
    Ok(())
}

fn main() {
    match run() {
        Ok(()) => exit(0),
        Err(CliError::Config(_)) => exit(2),
        Err(CliError::EmptyConfig) => {
            eprintln!("Configuration file is empty or not valid");
            exit(3);
        }
    }
}
```

- This structure allows you to test `run()` independently and control error printing and exit codes precisely.

***

## Summary Checklist

- Use exit code **0** for success, **non-zero** for errors (use custom codes as needed).[^1]
- Prefer returning `Result<(), ErrorType>` from `main()` and bubbling up errors.
- Print clear error messages to users via `eprintln!`.
- Avoid `panic!()` except for true programming bugs.
- Use crates like `thiserror`/`anyhow` for ergonomic, rich error handling.
- For more standardized exit codes, use the `exitcode` crate.[^1]

By following these conventions, you’ll write robust CLI tools that behave predictably for users, scripts, and system administrators.[^4][^5][^1]

1. https://rust-cli.github.io/book/in-depth/exit-code.html
2. https://rust-cli.github.io/book/tutorial/errors.html
3. https://doc.rust-lang.org/book/ch09-00-error-handling.html
4. https://technorely.com/insights/effective-error-handling-in-rust-cli-apps-best-practices-examples-and-advanced-techniques
5. https://www.reddit.com/r/learnrust/comments/117f4j4/error_handling_in_a_cli_app/
6. https://users.rust-lang.org/t/best-error-handing-practices-when-using-std-command/42259
7. https://stackoverflow.com/questions/30569408/best-practices-on-setting-exit-status-codes
8. https://www.reddit.com/r/rust/comments/eii16i/pattern_for_rich_cli_exit_codes/
9. https://stackoverflow.com/questions/73848357/what-is-the-best-way-of-handling-error-in-rust-cli-with-clap
10. https://users.rust-lang.org/t/how-to-handle-exit-status-properly-for-piped-commands/109150
