---
title: "File and Directory Manipulation in Rust"
description: "Learn how to create, read, write, and delete files and directories using Rust’s standard library and crates."
date: 2025-11-02
weight: 1
---

# File and Directory Manipulation in Rust: A Beginner's Guide

Rust's standard library (`std::fs`) provides powerful, cross-platform tools for interacting with the filesystem—creating, reading, writing, and deleting files and directories. These are essential tasks for all kinds of applications: from CLI utilities to servers. Let's explore how to perform these operations, with clear, beginner-friendly examples.

***

## 1. Creating and Writing to a File

The typical way to create and write to a new file is via `std::fs::File` and the `write_all` method.

```rust
use std::fs::File;
use std::io::Write;

fn main() {
    // Create a file (or truncate if it exists)
    let mut file = File::create("data.txt").expect("creation failed");
    // Write text to it as bytes
    file.write_all(b"Hello, Rust filesystem!").expect("write failed");

    println!("Created and wrote to data.txt");
}
```

- If the file doesn't exist, it's created.
- If it does exist, it is **truncated** (emptied).
- Always handle errors using `expect()` or `match`.

***

## 2. Reading a File

You can read the content of a text file into a string with `std::fs::read_to_string`.

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("data.txt").expect("read failed");
    println!("File content: {}", content);
}
```

If you want to process the file line-by-line:

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("data.txt").unwrap();
    for line in content.lines() {
        println!("Line: {}", line);
    }
}
```


***

## 3. Appending to a File

To add data at the end of an existing file (rather than overwriting), use `std::fs::OpenOptions`:

```rust
use std::fs::OpenOptions;
use std::io::Write;

fn main() {
    let mut file = OpenOptions::new()
        .append(true)
        .open("data.txt")
        .expect("cannot open file for appending");

    file.write_all(b"\nAppended line!").expect("append failed");
}
```


***

## 4. Creating and Listing Directories

Create a new directory with `fs::create_dir` and list directory entries with `fs::read_dir`:

```rust
use std::fs;

fn main() {
    fs::create_dir("my_folder").expect("creation failed");

    // List entries in the current directory
    for entry in fs::read_dir(".").expect("read dir failed") {
        let entry = entry.expect("entry failed");
        let path = entry.path();
        if path.is_dir() {
            println!("{:?} is a directory", path);
        } else {
            println!("{:?} is a file", path);
        }
    }
}
```


***

## 5. Deleting Files and Directories

Remove a file with `fs::remove_file`, and a directory with `fs::remove_dir` (for empty dirs) or `fs::remove_dir_all` (for non-empty).

```rust
use std::fs;

fn main() {
    fs::remove_file("data.txt").expect("file remove failed");
    fs::remove_dir_all("my_folder").expect("dir remove failed");

    println!("Cleaned up files and directories.");
}
```


***

## 6. Checking File Existence

You can use `std::path::Path` to check if a file or directory exists.

```rust
use std::path::Path;

fn main() {
    let path = Path::new("data.txt");
    if path.exists() {
        println!("data.txt exists!");
    } else {
        println!("data.txt NOT found");
    }
}
```


***

## Key Points and Gotchas

- All file operations may fail (e.g., permissions, disk full, file not found), so error handling is essential.
- File operations are buffered and relatively efficient, but for very large files or advanced patterns, consider `BufReader` and `BufWriter`.

***

## Summary Table

| Operation | Method Example | Notes |
| :-- | :-- | :-- |
| Create/write file | `File::create`, `.write_all` | Truncates file if exists |
| Read whole file to string | `fs::read_to_string` | For small-to-medium-sized text files |
| Read line by line | `for line in content.lines()` | `BufReader` for large files |
| Append to file | `OpenOptions::new().append(true)` | File must exist |
| Create directory | `fs::create_dir`, `fs::create_dir_all` | Recursively make folders with `dir_all` |
| List directory entries | `fs::read_dir` | Iterates content as `DirEntry`s |
| File/directory exists | `Path::exists()` | Checks file or folder |
| Delete file | `fs::remove_file` | File must exist |
| Delete dir (all) | `fs::remove_dir_all` | Removes non-empty folders |

With these examples and patterns, you're ready to manage files and folders with idiomatic Rust code!

1. https://www.youtube.com/watch?v=0H3pg_pjyRE
2. https://codesignal.com/learn/courses/fundamentals-of-text-data-manipulation-in-rust/lessons/reading-text-files-in-rust-a-beginners-guide
3. https://doc.rust-lang.org/rust-by-example/std_misc/fs.html
4. https://www.programiz.com/rust/file-handling
5. https://mkaz.blog/working-with-rust/files-and-dirs
6. https://doc.rust-lang.org/std/fs/struct.File.html
7. https://labex.io/tutorials/rust-filesystem-operations-in-rust-99277
8. https://www.geeksforgeeks.org/rust/file-i-o-in-rust/
9. https://www.petergirnus.com/blog/rust-parsing-directories-and-file-entries-with-direntry
10. https://blog.stackademic.com/working-with-files-in-rust-creating-writing-reading-and-deleting-79ae584104e9
