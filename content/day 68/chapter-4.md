---
title: "Memory Mapping in Rust"
description: "Discover how to use memory-mapped files in Rust for efficient file I/O and memory usage."
date: 2025-11-12
weight: 4
---

# Memory Mapping in Rust: An Efficient I/O Technique

Understanding memory-mapped files in Rust is like discovering a secret passage between your kitchen and the pantry. Instead of making individual trips to fetch ingredients (traditional file I/O), you open a portal that makes the pantry's shelves appear as if they were right in your kitchen. You can then grab or replace ingredients instantly, as if they were just another part of your local workspace. This "portal" is a memory map, and it's a powerful technique for high-performance file I/O.

## What is a Memory-Mapped File?

A memory-mapped file is a mechanism where a file on disk is mapped directly into a process's virtual memory address space. The operating system handles the magic behind the scenes. Instead of you calling `read()` or `write()` and copying data from the OS into your application's buffers, you can access the file's contents as if it were a simple slice or array in memory (`&[u8]`).[^1]

### Why is This So Efficient?

1. **Lazy Loading**: The entire file is not loaded into RAM at once. The OS only loads pages (small, fixed-size blocks of memory) from the file into physical RAM as your program accesses them. For a 10 GB file, if you only touch 1 MB of data, only that 1 MB is loaded.[^1]
2. **Zero-Copy I/O (in theory)**: Data doesn't need to be copied from the kernel's page cache into your application's memory. Your application works directly with the kernel's cache, reducing overhead.
3. **Random Access is Cheap**: Jumping to the middle of a huge file is just as fast as reading the beginning. You don't need to `seek()` and then `read()`; you just access the slice at the desired index (e.g., `mmap[1_000_000]`).
4. **Shared Memory**: Multiple processes can map the same file into their memory, making it an efficient way to share large amounts of data for Inter-Process Communication (IPC).[^1]

## Using Memory-Mapped Files in Rust with `memmap2`

While Rust's standard library doesn't have a built-in memory mapping API, the `memmap2` crate is the de facto standard. It provides a safe, cross-platform interface.

**Setup**: Add `memmap2` to your `Cargo.toml`:

```toml
[dependencies]
memmap2 = "0.9"
```


### Example 1: Reading from a Memory-Mapped File

Let's say we have a large log file, `app.log`, and we want to efficiently read parts of it.

1. **Create a sample file:**

```rust
use std::fs::File;
use std::io::Write;

fn create_sample_file() -> std::io::Result<()> {
    let mut file = File::create("app.log")?;
    file.write_all(b"INFO: Application starting up.\n")?;
    file.write_all(b"DEBUG: Configuration loaded.\n")?;
    file.write_all(b"ERROR: Failed to connect to database.\n")?;
    Ok(())
}
```

2. **Memory-map it and read from it:**

```rust
use memmap2::Mmap;
use std::fs::File;

fn read_from_mmap() -> std::io::Result<()> {
    let file = File::open("app.log")?;

    // Memory-map the file. This is an `unsafe` operation because another
    // process could modify the file while we're reading it, violating
    // Rust's aliasing rules.
    let mmap = unsafe { Mmap::map(&file)? };

    // Now, we can treat `mmap` just like a byte slice!
    // Let's find the line containing "ERROR".
    let error_line = mmap.split(|&b| b == b'\n')
                       .find(|s| s.starts_with(b"ERROR"));

    if let Some(line) = error_line {
        println!("Found error: {}", String::from_utf8_lossy(line));
    }

    Ok(())
}
```


In this example, we never explicitly called `read()`. The OS handled loading the necessary parts of `app.log` into memory as our code iterated through it.

### Example 2: Writing to a Memory-Mapped File

Writing works similarly, but we need to open the file with write permissions and use `MmapMut`.

1. **Setup the file and the mapping:**

```rust
use memmap2::MmapMut;
use std::fs::OpenOptions;
use std::io::{Write, Result};

fn write_to_mmap() -> Result<()> {
    let file = OpenOptions::new()
        .read(true)
        .write(true)
        .create(true)
        .open("data.bin")?;

    // The file must have a size before we can map it.
    // Let's set it to 1KB.
    file.set_len(1024)?;

    // Create a mutable memory map.
    let mut mmap = unsafe { MmapMut::map_mut(&file)? };

    // 2. Write to the map like it's a mutable slice.
    let message = b"Hello, memory-mapped world!";
    mmap[0..message.len()].copy_from_slice(message);

    // 3. Flush the changes to disk.
    // This guarantees the data is written back to the file.
    mmap.flush()?;

    println!("Data written and flushed to data.bin");
    Ok(())
}
```


**Crucial Point**: When using a mutable map (`MmapMut`), you must call `.flush()` to ensure your changes are written back to the underlying file. The operating system may otherwise delay writing the changes indefinitely.

## The `unsafe` Contract: Why is `mmap` Unsafe?

Creating a memory map is `unsafe` for a fundamental reason: it breaks Rust's core promise that if you have a reference (`&T`), the data it points to will not change unexpectedly.

When you map a file, **another process could modify that file on disk at any time**. This would change the contents of your memory slice "from under your feet," which is a data race and constitutes Undefined Behavior.[^2][^3]

When you write `unsafe { Mmap::map(&file)? }`, you are making a promise to the compiler:
> "I understand that this memory is backed by a file that could change. I will write my code in a way that is robust to these potential changes, or I am operating in an environment where I can guarantee no other process will interfere with this file."

## When to Use Memory-Mapped Files

- **Large Files**: When you need to process files that are too large to fit comfortably in RAM.
- **Random Access**: When you need to frequently jump between different parts of a file (e.g., reading metadata from the end, then jumping to the middle for data). This is common in database and index implementations.
- **High-Performance I/O**: For applications where the overhead of `read`/`write` system calls and memory copies is a bottleneck.
- **Inter-Process Communication (IPC)**: When you need two or more processes on the same machine to share and modify a large block of data efficiently.

By leveraging memory mapping, you can write highly efficient Rust code that handles large-scale data with the simplicity of in-memory operations, all while being mindful of the safety contracts you undertake when working this close to the operating system.

1. https://blog.devgenius.io/memory-mapped-i-o-in-rust-c6e039132df9
2. https://www.reddit.com/r/rust/comments/10u4anm/how_to_use_mmap_safely_in_rust/
3. https://news.ycombinator.com/item?id=26446495
4. https://stackoverflow.com/questions/69789530/writing-to-a-memory-mapped-file-from-multiple-threads-in-rust
5. https://docs.rs/memmap/latest/memmap/struct.Mmap.html
6. https://tonisagrista.com/blog/2021/rust-memorymap/
7. https://users.rust-lang.org/t/towards-a-more-perfect-rustio/18570?page=2
8. https://www.youtube.com/watch?v=zdZdtg1f9lA
9. https://stackoverflow.com/questions/28516996/how-to-create-and-write-to-memory-mapped-files
10. https://docs.rs/memmap2/latest/memmap2/struct.MmapOptions.html
11. https://www.reddit.com/r/rust/comments/sn4zl4/memory_mapped_files_in_rust/
12. https://www.youtube.com/watch?v=0H3pg_pjyRE
13. https://www.youtube.com/watch?v=-6cnnNlAvNk
14. https://dev.to/gritmax/memory-allocations-in-rust-3m7l
15. https://www.youtube.com/watch?v=7_o-YRxf_cc
16. https://users.rust-lang.org/t/memory-mapping-soc-registers/31976
17. https://doc.rust-lang.org/rust-by-example/
