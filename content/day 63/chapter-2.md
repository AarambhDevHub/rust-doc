---
title: "Inter-Process Communication (IPC) in Rust"
description: "Explore message passing, shared memory, and other IPC mechanisms for efficient communication between processes."
date: 2025-11-05
weight: 2
---

# Inter-Process Communication (IPC) in Rust: A Beginner’s Guide

Inter-process communication (IPC) is how independent programs (“processes”) running on the same machine exchange data and coordinate work. Unlike threads within a single process, processes are isolated by the operating system, so you must use OS-level techniques to share information.

## Summary of IPC Mechanisms

Here are common IPC options in Rust, each with its strengths and limitations:


| Mechanism | Best for | Rust Support (as of 2025) |
| :-- | :-- | :-- |
| Message Passing | Small, independent messages | Pipes, Unix sockets, TCP/UDP, `interprocess` crate |
| Shared Memory | Large, fast data transfer | `memmap2`, `shmem`, `ipc-channel` crates |
| Files / Tempfiles | Persistence, audit, simplicity | Standard library |
| Named Pipes (FIFOs) | Cross-language, simple streaming | `interprocess` crate, OS APIs |
| Sockets (TCP/UDP) | Local or network communication | Standard library (`std::net`), crates |


***

## 1. Message Passing: Pipes and Sockets

### Example 1: Parent/Child Process with Std IO Pipes

You can use Rust’s standard library to spawn a child process and send/receive data via its standard input/output (just like Bash pipes):

```rust
use std::io::{Write, BufRead, BufReader};
use std::process::{Command, Stdio};

fn main() {
    // Start a child process (replace "sort" with any CLI command)
    let mut child = Command::new("sort")
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()
        .unwrap();

    // Send lines to the child's stdin
    let mut stdin = child.stdin.take().unwrap();
    write!(stdin, "banana\napple\ncarrot\n").unwrap();
    drop(stdin); // Close input so child can finish

    // Read lines from child's stdout
    let stdout = child.stdout.take().unwrap();
    let reader = BufReader::new(stdout);
    for line in reader.lines() {
        println!("sorted: {}", line.unwrap());
    }
}
```

This technique is great for quick-and-easy parent/child IPC.[^1]

### Example 2: Sockets (Local or Network)

A minimal TCP socket server and client for IPC:

#### Server

```rust
use std::net::{TcpListener, TcpStream};
use std::io::{Read, Write};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:4000").unwrap();
    for stream in listener.incoming() {
        let mut stream = stream.unwrap();
        let mut buf = [0; 128];
        let n = stream.read(&mut buf).unwrap();
        println!("Received: {}", String::from_utf8_lossy(&buf[..n]));
        stream.write_all(b"ack") .unwrap(); // Respond to client
    }
}
```


#### Client

```rust
use std::net::TcpStream;
use std::io::{Read, Write};

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:4000").unwrap();
    stream.write_all(b"ping from client").unwrap();
    let mut buf = [0; 128];
    let n = stream.read(&mut buf).unwrap();
    println!("Server says: {}", String::from_utf8_lossy(&buf[..n]));
}
```

Works across the local machine or over the network for multi-host scenarios.[^2]

***

## 2. Shared Memory

With shared memory, two or more processes map a region of RAM to their own address space; reading/writing to this shared space is as fast as normal memory access. **You must synchronize access with mutexes or semaphores to prevent race conditions**.[^3]

### Example: POSIX Shared Memory (Linux/Unix)

Popular Rust crates for shared memory IPC:

- `memmap2`
- `ipc-channel`
- `interprocess`

You can use the [`interprocess`](https://github.com/kotauskas/interprocess) crate for true Rust cross-platform shared memory:

```rust
use interprocess::shared_memory::SharedMemCastMut;
use interprocess::shared_memory::ShmemConf;

fn main() -> std::io::Result<()> {
    // Create or open a shared memory region named "my_shm"
    let shmem = ShmemConf::new().size(4096).os_id("my_shm").create()?;
    let slice: &mut [u8] = shmem.as_slice();

    // Write to shared memory
    slice[0..5].copy_from_slice(b"Hello");
    Ok(())
}
```

This lets two processes cooperate by both opening "my_shm" and working with its contents (safely, using extra synchronization).[^4][^5]

***

## 3. Advanced IPC: Message Queues, Named Pipes, and ZeroMQ

- **Named Pipes/FIFOs:** Create a named queue/file in the filesystem that processes can write/read from (`interprocess::os::unix::pipe`).
- **ZeroMQ/nanomsg:** Use `zmq` or `nanomsg` crates for high-speed message passing in large, distributed systems.[^6]
- **HTTP/REST:** Run a mini HTTP server (`hyper`, `tiny_http`) for maximum flexibility and language interoperability.[^2]

***

## Quick Comparison: Which IPC for What?

| **Scenario** | **Best IPC Mechanism** |
| :-- | :-- |
| Parent/child process, simple text | STDIN/STDOUT pipes |
| Large or binary data, high speed | Shared memory + semaphores |
| Many tiny messages | Named pipes, message queue |
| Local network, multiple languages | TCP/UDP socket |
| Distributed system | ZeroMQ/nanomsg, HTTP |


***

## Key Points and Best Practices

- **Message passing** is simple and safe—ideal for small data and simple communication patterns.
- **Shared memory** is much faster for large data, but requires careful synchronization (mutexes, semaphores) to avoid bugs.
- Leverage crates like `interprocess`, `memmap2`, `ipc-channel`, or standard sockets.
- Synchronize shared memory access to prevent collisions.
- Properly clean up any OS resources (named pipes, shared memory) when finished.

By understanding and applying these Rust IPC mechanisms, you can efficiently connect independent processes and services, building more modular, scalable systems for both local and distributed use.[^5][^1][^3]

1. https://stackoverflow.com/questions/27683266/how-do-you-do-interprocess-communication-ipc-in-rust
2. https://users.rust-lang.org/t/interconnect-between-programs/52718
3. https://www.geeksforgeeks.org/operating-systems/ipc-shared-memory/
4. https://users.rust-lang.org/t/shared-memory-for-interprocess-communication/92408
5. https://3tilley.github.io/posts/simple-ipc-ping-pong/
6. https://www.dvlv.co.uk/how-to-do-inter-process-communication-ipc-w-python-and-rust.html
7. https://www.reddit.com/r/rust/comments/1diofg1/i_explored_some_different_methods_for/
8. https://v2.tauri.app/concept/inter-process-communication/
9. https://www.youtube.com/watch?v=Y2mDwW2pMv4
10. https://vadosware.io/post/yet-another-ipc-in-rust-experiment/
