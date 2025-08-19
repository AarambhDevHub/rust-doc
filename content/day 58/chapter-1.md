---
title: "Foreign Function Interface - Calling C Code from Rust"
description: "Learn how to call C functions safely and efficiently from Rust using FFI."
date: 2025-10-15
weight: 1
---

# Foreign Function Interface (FFI): Calling C Code from Rust

Learning to use Rust's Foreign Function Interface (FFI) is like learning to be a **professional translator at the United Nations**. You have a Rust-speaking delegate who needs to communicate with a C-speaking delegate. Your job is to facilitate this conversation, ensuring that messages are passed back and forth correctly, that data types are translated properly, and that both parties understand each other's customs (calling conventions). FFI is the bridge that allows your safe, modern Rust code to leverage the vast ecosystem of existing C libraries.

## The UN Translator Analogy: Bridging Two Languages 🌐

- **Rust Code**: Your modern, safety-conscious delegate who speaks Rust.
- **C Library**: The established, experienced delegate who speaks C.
- **FFI**: You, the translator, who understands the rules, vocabulary, and customs (calling conventions) of both languages.
- `unsafe` **Block**: The official, recorded part of the conversation where you, the translator, take full responsibility for the accuracy of the translation. Any misinterpretation is on you.

***

### Why Use FFI?

1. **Leveraging Existing Code**: There are millions of lines of high-quality, battle-tested C code in the world (e.g., system libraries, compression algorithms, physics engines). FFI allows you to use them directly without reinventing the wheel.[^6]
2. **Performance**: For certain low-level tasks, a highly optimized C library might be the fastest option available.
3. **OS Interoperability**: Interacting directly with the operating system often means calling C APIs (e.g., POSIX on Linux, Win32 on Windows).

## The Three Steps of Calling C from Rust

The process can be broken down into three main steps:

1. **The C Code**: Identify or create the C library you want to call.
2. **The Rust Declaration**: Declare the C function's signature in your Rust code.
3. **The Unsafe Call**: Call the C function from within an `unsafe` block.

Let's walk through a complete, simple example.

### Step 1: Create a C Library

First, we need a C function to call. Let's create a simple C library that adds two numbers.

**File: `math_lib/sum.c`**

```c
#include <stdint.h>

// A simple function that adds two 32-bit integers.
int32_t add(int32_t a, int32_t b) {
    return a + b;
}
```

Now, we need to compile this C code into a **shared library** (`.so` on Linux, `.dylib` on macOS, `.dll` on Windows) so our Rust program can link to it at runtime.

**Compilation (on Linux/macOS):**

```bash
# Create a directory for our C library
mkdir math_lib
# (Place sum.c inside it)

# Compile the C code into a shared library named libsum.so
gcc -shared -o math_lib/libsum.so math_lib/sum.c
```


### Step 2: Declare the Foreign Function in Rust

Now, in our Rust project, we need to tell the Rust compiler about the `add` function. We do this inside an `extern "C"` block. This block lists the functions from the foreign library that we want to use.[^2]

**File: `src/main.rs`**

```rust
// We use types from the `libc` crate to ensure our integer types
// match C's representation exactly. Add `libc = "0.2"` to Cargo.toml.
use libc::c_int;

// This block tells Rust about the external C library we want to link to.
// The name "sum" corresponds to the filename "libsum.so".
#[link(name = "sum")]
extern "C" {
    // This is the function signature of our C function.
    // It must match the C header file exactly.
    fn add(a: c_int, b: c_int) -> c_int;
}

fn main() {
    // We'll call the function in the next step.
}
```

**Key Components:**

- `#[link(name = "sum")]`: This attribute tells the linker to look for a library named `sum` (e.g., `libsum.so`).
- `extern "C"`: This specifies the **Application Binary Interface (ABI)**. It tells Rust to use the standard C calling convention, which ensures that Rust and C agree on how to pass arguments and return values.[^1]
- `fn add(...)`: This is the function declaration. Note that we don't provide a function body; we are just declaring its existence and signature.


### Step 3: Make the Unsafe Call

Calling any foreign function is considered `unsafe` because the Rust compiler cannot analyze the C code to guarantee it's memory-safe. You, the programmer, must take on that responsibility.[^3]

**File: `src/main.rs` (Completed)**

```rust
use libc::c_int;

#[link(name = "sum")]
extern "C" {
    fn add(a: c_int, b: c_int) -> c_int;
}

fn main() {
    let a = 10;
    let b = 20;

    // All FFI calls must be wrapped in an `unsafe` block.
    let result = unsafe {
        // SAFETY: We are calling a simple C function that we wrote.
        // We know it doesn't have any dangerous side effects, takes two
        // 32-bit integers, and returns a 32-bit integer. The types match,
        // and the operation is safe.
        add(a, b)
    };

    println!("{} + {} = {} (calculated in C!)", a, b, result);
}
```


### Running the Project

To run this, you need to tell the dynamic linker where to find your C library.

**On Linux/macOS:**

```bash
# Add the directory containing your .so/.dylib to the linker path
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$(pwd)/math_lib

# Now, run the Rust program
cargo run
```

**Expected Output:**

```
10 + 20 = 30 (calculated in C!)
```


## Handling More Complex Data: Strings and Pointers

Passing simple numbers is easy. Things get more complex when dealing with data types that don't have the same memory layout in Rust and C, like strings.

- **Rust `String`**: A "fat pointer" containing a pointer to the data, a length, and a capacity. It is guaranteed to be UTF-8 and is not null-terminated.
- **C `char*`**: A "thin pointer" to a block of memory. It is assumed to be null-terminated and has no defined encoding.

You cannot pass a Rust `String` directly to C. You must first convert it into a C-compatible representation.

**Example: Passing a String to C**

**C Code (`string_lib/print_str.c`):**

```c
#include <stdio.h>

void print_string(const char* s) {
    printf("C received the string: %s\n", s);
}
```

Compile it: `gcc -shared -o string_lib/libprint_str.so string_lib/print_str.c`

**Rust Code (`src/main.rs`):**

```rust
use std::ffi::CString;
use libc::c_char;

#[link(name = "print_str")]
extern "C" {
    fn print_string(s: *const c_char);
}

fn main() {
    let rust_string = "Hello from Rust!";

    // 1. Convert the Rust &str into a C-compatible string.
    // CString ensures the string is null-terminated.
    let c_string = CString::new(rust_string).unwrap();

    unsafe {
        // 2. Pass a pointer to the C-compatible string.
        // .as_ptr() gives us a `*const c_char`.
        print_string(c_string.as_ptr());
    }
}
```

**Running it:**
`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$(pwd)/string_lib && cargo run`

## Best Practices for Safe FFI

FFI is inherently `unsafe`, but you can mitigate the risks by following best practices.

1. **Create Safe Wrappers**: The most important practice. Don't sprinkle `unsafe` blocks throughout your codebase. Create a safe Rust function that encapsulates the `unsafe` FFI call and handles all the data conversion and validation.[^2]

```rust
// Safe wrapper around the unsafe C function
pub fn add_safely(a: i32, b: i32) -> i32 {
    unsafe { add(a, b) }
}
```

2. **Use the `libc` Crate**: Always use the type definitions from `libc` (like `c_int`, `c_char`, `c_void`) to ensure your types match C's exactly across different platforms.
3. **Handle Pointers Carefully**: Pointers coming from C could be `NULL`. Always check them with `.is_null()` before using them.
4. **Manage Memory Manually**: Rust's memory management does not extend to C. If a C function allocates memory and returns a pointer to it, you are responsible for eventually calling the corresponding C `free` function to prevent memory leaks.
5. **Use `bindgen` for Large APIs**: Manually writing `extern` blocks for large C libraries is tedious and error-prone. The `bindgen` tool can automatically generate the Rust FFI declarations from C header files, saving you a huge amount of time and effort.[^5]

By acting as a careful and diligent "translator," you can safely and efficiently integrate the power of the C ecosystem into your modern, memory-safe Rust programs.

1. https://doc.rust-lang.org/nomicon/ffi.html
2. https://doc.rust-lang.org/rust-by-example/std_misc/ffi.html
3. https://github.com/vanjacosic/rust-ffi-to-c
4. https://labex.io/tutorials/rust-foreign-function-interface-99280
5. https://blog.theembeddedrustacean.com/rust-ffi-and-bindgen-integrating-embedded-c-code-in-rust
6. https://locka99.gitbooks.io/a-guide-to-porting-c-to-rust/content/features_of_rust/ffi.html
7. https://www.reddit.com/r/rust/comments/18vnsf2/rusts_ffi_with_c/
8. https://opensource.com/article/22/11/rust-calls-c-library-functions
9. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/ffi.html
