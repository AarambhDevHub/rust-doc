---
title: "Foreign Function Interface - Calling Rust from C"
description: "Understand how to expose Rust functions and libraries to be called from C."
date: 2025-10-15
weight: 3
---

# Foreign Function Interface (FFI): Calling Rust from C

Learning to expose Rust code to be called from C via the Foreign Function Interface (FFI) is like being a **professional translator who prepares a delegate for a speech at the United Nations**. Your Rust delegate is modern and has many powerful, high-level ways of speaking (like `String`, `Vec`, and enums with data). However, the UN's main floor (the C ecosystem) only understands a very specific, simple, and rigid C-style of communication. Your job is to create a "C-compatible" version of your Rust delegate's speech—a safe, stable interface that C can understand and use without causing an international incident (a program crash).

## The UN Translator Analogy: Preparing a Speech for a C Audience 🌐

- **Rust Library**: Your modern, safety-conscious delegate who speaks idiomatic Rust.
- **C Application**: The established, experienced audience that only understands the C language and its data types.
- **FFI Boundary**: The podium where your Rust delegate will speak. Everything said at this podium must be in a C-compatible format.
- **Your Job**: To take your delegate's rich, expressive Rust ideas and translate them into a simple, C-style script that can be delivered at the podium.

***

### Why Expose Rust to C?

1. **Gradual Modernization**: You can rewrite critical parts of a legacy C/C++ application in Rust for safety and performance, without having to rewrite the entire application at once.
2. **Extending Other Languages**: Many high-level languages (like Python, Ruby, and Node.js) have C FFI interfaces. By exposing your Rust library as a C library, you can create high-performance extensions for them.
3. **Building Cross-Platform Libraries**: You can write your core logic in Rust and expose a stable C API that can be used by C, C++, C\#, Java (via JNI), Swift, and many other languages.

## The Three Steps of Exposing Rust to C

1. **Create a C-Compatible Rust Function**: Write a Rust function using special annotations (`extern "C"`, `#[no_mangle]`) to make it callable from C.
2. **Compile the Rust Code as a C Library**: Configure `cargo` to produce a static (`.a`) or dynamic (`.so`, `.dylib`, `.dll`) library.
3. **Call the Rust Function from C**: Write a C program that includes a header file for your Rust library and calls the exported function.

Let's walk through a complete example.

### Step 1: Create a C-Compatible Rust Library

We'll create a Rust library that provides a function to add two numbers.

**1. Set up the Cargo project:**

```bash
cargo new rust_math_lib --lib
cd rust_math_lib
```

**2. Configure `Cargo.toml` for a C library:**
To tell Cargo to build a C-compatible static or dynamic library, we add a `[lib]` section.

**`Cargo.toml`**

```toml
[package]
name = "rust_math_lib"
version = "0.1.0"
edition = "2021"

[lib]
name = "rust_math" # The base name of the output library file
crate-type = ["cdylib"] # Or "staticlib"

# "cdylib" -> creates a dynamic library (e.g., lib<name>.so)
# "staticlib" -> creates a static library (e.g., lib<name>.a)
```

**3. Write the FFI-safe Rust function:**
**`src/lib.rs`**

```rust
use std::os::raw::c_int;

/// This is a public function, but it is intended to be called from C.
/// It must have annotations to make it C-compatible.
#[no_mangle]
pub extern "C" fn add(a: c_int, b: c_int) -> c_int {
    // We can call safe Rust code from within this FFI function.
    a + b
}
```

**Key Annotations:**

- `pub`: The function must be public to be exported.
- `extern "C"`: This tells the compiler to use the standard C Application Binary Interface (ABI) for this function. This ensures that the way it expects arguments and returns values is understood by C code.[^2]
- `#[no_mangle]`: By default, the Rust compiler "mangles" function names to include extra information. This annotation tells the compiler to use the exact function name (`add`) in the compiled library, so that C can find it by that name.[^1]


### Step 2: Compile the Rust Library and Generate a Header

Now, build the Rust library.

```bash
cargo build --release
```

This will create a dynamic library file at `target/release/librust_math.so` (or `.dylib`/`.dll`).

For C code to use this library, it needs a **header file** (`.h`) that declares the functions available. The `cbindgen` tool is perfect for automatically generating this.

**1. Install `cbindgen`:**

```bash
cargo install cbindgen
```

**2. Generate the header:**

```bash
cbindgen --crate rust_math_lib --output rust_math.h
```

This will create a `rust_math.h` file in your project root with the following content:
**`rust_math.h`**

```c
#include <stdarg.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdlib.h>

/**
 * This is a public function, but it is intended to be called from C.
 * It must have annotations to make it C-compatible.
 */
int32_t add(int32_t a, int32_t b);
```


### Step 3: Call the Rust Function from a C Program

Finally, let's write a C program that uses our new library.

**File: `main.c`** (place this in the project root)

```c
#include <stdio.h>
#include "rust_math.h" // Include our generated header

int main() {
    int32_t a = 15;
    int32_t b = 25;

    // Call the function from our Rust library!
    int32_t result = add(a, b);

    printf("%d + %d = %d (calculated in Rust!)\n", a, b, result);

    return 0;
}
```

**Compile and Run the C Program:**

```bash
# Compile the C code, linking it against our Rust library.
# -I. tells the compiler to look for headers in the current directory.
# -L./target/release tells the linker where to find the library.
# -lrust_math tells the linker to link the "rust_math" library.
gcc main.c -I. -L./target/release -lrust_math -o main_app

# Run the final C application.
# (You might need to set LD_LIBRARY_PATH on Linux)
./main_app
```

**Expected Output:**

```
15 + 25 = 40 (calculated in Rust!)
```


## Handling Complex Data: Opaque Structs and Memory Management

Passing numbers is easy. The real challenge is managing complex Rust types (like `structs` or `Vecs`) from C. You cannot expose a Rust `struct` directly to C if it doesn't have a C-compatible memory layout.

The solution is the **opaque pointer** pattern (also known as a "handle").

1. Rust creates an instance of a complex `struct` and allocates it on the heap using `Box`.
2. Rust gives C a **raw pointer** to this heap-allocated data. From C's perspective, this is just an opaque "handle" – it doesn't know what's inside.
3. C can pass this handle back to other Rust functions to operate on the data.
4. Finally, C must pass the handle back to a special Rust function to **free the memory**, preventing a memory leak.

**Example: A "Counter" Object**

**`src/lib.rs`**:

```rust
use std::os::raw::c_int;

// Our complex Rust object
pub struct Counter {
    count: c_int,
}

#[no_mangle]
pub extern "C" fn counter_new() -> *mut Counter {
    // Allocate the Counter on the heap and return a raw pointer to it.
    Box::into_raw(Box::new(Counter { count: 0 }))
}

#[no_mangle]
pub extern "C" fn counter_increment(ptr: *mut Counter) {
    if ptr.is_null() { return; }
    // Convert the raw pointer back into a mutable reference to work with it safely.
    let counter = unsafe { &mut *ptr };
    counter.count += 1;
}

#[no_mangle]
pub extern "C" fn counter_get_value(ptr: *const Counter) -> c_int {
    if ptr.is_null() { return -1; }
    let counter = unsafe { &*ptr };
    counter.count
}

#[no_mangle]
pub extern "C" fn counter_free(ptr: *mut Counter) {
    if ptr.is_null() { return; }
    // Convert the raw pointer back into a Box and let it drop, freeing the memory.
    unsafe { Box::from_raw(ptr) };
}
```

**C Usage (`main.c`):**

```c
#include <stdio.h>
#include "rust_math.h// Use the new header from cbindgen

int main() {
    // Get an opaque handle to our Rust object
    struct Counter* my_counter = counter_new();

    counter_increment(my_counter);
    counter_increment(my_counter);

    int32_t value = counter_get_value(my_counter);
    printf("Counter value from Rust: %d\n", value);

    // Crucially, we must tell Rust to free the memory
    counter_free(my_counter);

    return 0;
}
```

This pattern is the foundation for building robust, safe FFI APIs. It keeps the complex logic and ownership inside Rust, exposing only simple, C-compatible handles to the outside world.

1. https://doc.rust-lang.org/rust-by-example/std_misc/ffi.html
2. https://doc.rust-lang.org/nomicon/ffi.html
3. https://opensource.com/article/22/11/rust-calls-c-library-functions
4. https://www.reddit.com/r/rust/comments/18vnsf2/rusts_ffi_with_c/
5. https://masteringbackend.com/hubs/advanced-rust/ffi-and-interoperability-in-rust
6. https://labex.io/tutorials/rust-foreign-function-interface-99280
7. https://github.com/vanjacosic/rust-ffi-to-c
8. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/ffi.html
