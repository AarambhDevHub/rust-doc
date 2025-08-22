---
title: "System Calls in Rust"
description: "Understand how to make direct system calls in Rust for low-level control over operating system functionality."
date: 2025-11-05
weight: 3
---

# System Calls in Rust: Gaining Low-Level OS Control

Direct system calls are like sending messages straight to a country's government without using an embassy, consultant, or translator. In normal Rust (and most languages), you work through libraries (`std`, `libc`), which abstract these raw operating system interactions. For **maximum control**, **learning**, or **no-std/embedded** contexts, you may want to send system calls *yourself*.

Below, you'll learn what a system call is, see how to make them directly in Rust, and understand the best practices—all in a way accessible to beginners.

***

## What is a System Call?

A **system call** is the fundamental interface between a running program and the operating system kernel. It lets you ask the OS to do work "on your behalf"—open files, allocate memory, communicate over the network, etc.

- In C: you often use functions like `write()`, `open()`, `read()`, which are *wrappers* around the actual system call.
- In Rust: the standard library (`std::fs::File`, `std::io::Write`, etc.) and `libc` crate wrap system calls for safe, cross-platform, and ergonomic usage.


## Why Make Direct System Calls?

- **No standard library**: In embedded, bootloaders, or kernel code, you may not have `std` available.
- **Performance/Customization**: Sometimes you want the *bare* minimum, with zero overhead or features stripped by abstraction layers.
- **Learning and Debugging**: Seeing how abstractions work under the hood.[^1][^2]

***

## How to Make a Direct System Call in Rust

### The x86_64 Linux Example

On Linux (x86_64), system calls use the `syscall` CPU instruction, with arguments placed in specific registers. For example, to call `write()` (which prints bytes to a file descriptor), you use syscall number 1 (`write`), and pass the arguments:

- `rdi`: file descriptor
- `rsi`: pointer to bytes
- `rdx`: number of bytes


#### Minimal Example: Print "Hello world" to STDOUT

```rust
#[cfg(target_arch = "x86_64")]
fn main() {
    let s = b"Hello world\n";
    let syscall_number = 1; // write
    let fd = 1; // standard output
    let ret: isize;

    unsafe {
        // Use inline assembly to trigger the syscall
        std::arch::asm!(
            "syscall",
            in("rax") syscall_number,    // syscall number (write)
            in("rdi") fd,                // file descriptor (stdout)
            in("rsi") s.as_ptr(),        // pointer to data
            in("rdx") s.len(),           // number of bytes
            lateout("rax") ret           // output: return value is placed in rax
        );
    }

    assert!(ret > 0, "System call failed!");
}
```

- `std::arch::asm!` lets you inject raw assembly—here, the Linux `syscall` instruction.[^3][^4]
- You must know syscall numbers and the calling convention (argument locations).


#### How Do You Know Which Number and Arguments?

- Linux syscall table: [syscall_64.tbl list](https://github.com/torvalds/linux/blob/master/arch/x86/entry/syscalls/syscall_64.tbl)
- Definitions and meanings: [syscalls.h](https://github.com/torvalds/linux/blob/master/include/linux/syscalls.h)
- For `write`, the arguments are:
    - fd: `rdi`
    - buf: `rsi`
    - count: `rdx`

***

### Abstracting System Calls

To avoid copy-pasting assembly, it's common to wrap this in functions:

```rust
/// Make a system call with three arguments
unsafe fn syscall_3(num: u64, arg1: u64, arg2: u64, arg3: u64) -> i64 {
    let res: i64;
    std::arch::asm!(
        "syscall",
        in("rax") num,
        in("rdi") arg1,
        in("rsi") arg2,
        in("rdx") arg3,
        lateout("rax") res,
    );
    res
}

// Use enum or consts for syscall numbers for readability
const SYS_WRITE: u64 = 1;

/// Write bytes to a file descriptor (fd)
fn sys_write(fd: u64, data: &[u8]) -> i64 {
    unsafe { syscall_3(SYS_WRITE, fd, data.as_ptr() as u64, data.len() as u64) }
}

fn main() {
    let bytes = b"Direct syscall!\n";
    let written = sys_write(1, bytes); // 1=STDOUT
    assert!(written as usize == bytes.len());
}
```

With this helper, you could add wrappers for `open` and `read` similarly.[^3]

***

### Notes and Best Practices

- **ALWAYS wrap direct system calls in `unsafe`**: The compiler cannot check what you do; responsibility is 100% on you.
- Use standard library or `libc` for "real" projects, unless you need direct syscalls.
- For portability, prefer library functions—system call numbers, conventions, and semantics differ across OSes and architectures.
- Investigate higher-level wrappers like [`linux-syscall`](https://docs.rs/linux-syscall) for more convenient/safer use.[^5]
- When in doubt, study the source of Rust's standard library or `libc` on your platform.

***

## Conclusion

Direct system calls in Rust give you **full, low-level access** to your operating system, but require an expert's attention and care—useful mainly for advanced, constrained, or minimal environments, or for educational purposes. For most Rust code, the standard and `libc` libraries offer all the power with none of the danger!

1. https://www.reddit.com/r/rust/comments/1bg458s/linux_system_calls_in_rust/
2. https://users.rust-lang.org/t/help-understanding-os-system-calls-and-how-rust-does-this-versus-other-languages/13270
3. https://phip1611.de/blog/direct-systemcalls-to-linux-from-rust-code-x86_64/
4. https://github.com/phip1611/direct-syscalls-linux-from-rust
5. https://docs.rs/linux-syscall
6. https://users.rust-lang.org/t/how-could-a-syscall-be-built-in-rust/88113
7. https://stackoverflow.com/questions/13219501/accessing-a-system-call-directly-from-user-program
8. https://dominuscarnufex.github.io/cours/rs-kernel/en.html
9. https://fluxsec.red/implementing-syscall-hooking-rust
10. https://www.reddit.com/r/rust/comments/1lyxyoa/practicing_linux_syscalls_with_rust_and_x86_64/
