---
title: "Foreign Function Interface - Error Handling in FFI"
description: "Learn best practices for handling and propagating errors across Rust and C FFI boundaries."
date: 2025-10-15
weight: 5
---

# Foreign Function Interface (FFI): Best Practices for Error Handling Across Rust and C Boundaries

Handling errors across Rust and C FFI boundaries is like working as an international interpreter between two agencies with very different ways of reporting problems. Rust favors rich error types (`Option`, `Result`), while C typically uses simple conventions like returning `NULL`, special values, or error codes. Your job as the interpreter is to ensure that mistakes, failures, and exceptions are reliably communicated without getting lost in translation or causing unpredictable bugs.

## The International Interpreter Analogy: Reporting Problems Between Agencies 🌐

- **Rust Side**: Uses explicit error types such as `Option<T>`, `Result<T, E>`, and panics for reporting errors.
- **C Side**: Uses return values like `NULL`, `-1`, or separate out-parameters to indicate errors.
- **FFI Bridge**: You, the interpreter, must map Rust errors to C-style conventions and translate back if needed.

***

## Key Challenges

1. **Rust errors must be translated into C-understandable signals** (e.g., `NULL` pointer, integer error codes).
2. **C errors must be safely handled in Rust without causing UB** (undefined behavior).
3. **Memory ownership and lifetime issues** must be handled with care, especially when returning pointers or allocating resources.

***

## Common Patterns for Error Handling in FFI

### Pattern 1: Returning a `NULL` Pointer on Error

**Best for:** Functions that return pointers (e.g., for newly allocated objects or data buffers).

**Rust Side:**

```rust
#[no_mangle]
pub extern "C" fn my_struct_new() -> *mut MyStruct {
    match MyStruct::try_new() {
        Ok(obj) => Box::into_raw(Box::new(obj)), // Success: Return pointer
        Err(_) => std::ptr::null_mut(),         // Error: Return NULL
    }
}
```

**C Side:**

```c
MyStruct* obj = my_struct_new();
if (!obj) {
    // Error occurred, handle gracefully
}
```

This maps Rust's rich error into a simple C signal.

### Pattern 2: Returning an Error Code and Using Out-Parameters

**Best for:** Functions that need to indicate multiple error types or return non-pointer results.

**Rust Side:**

```rust
#[no_mangle]
pub extern "C" fn compute_sum(a: i32, b: i32, out_sum: *mut i32) -> i32 {
    if out_sum.is_null() { return -1; }
    // Simulate error (e.g., check bounds or logic)
    if a < 0 || b < 0 { return 1; } // Invalid argument error code
    unsafe { *out_sum = a + b; }
    0 // Zero means success
}
```

**C Side:**

```c
int result;
int error = compute_sum(5, 7, &result);
if (error == 0) {
    printf("Sum is %d\n", result);
} else {
    // Error code interpretation
}
```

This gives more fine-grained control for different error cases.

### Pattern 3: FFI-Friendly Structs for Richer Error Info

**Best for:** When you need to return more than just a pointer or basic error code, use a C-compatible struct.

**Rust Side:**

```rust
#[repr(C)]
pub struct MyResult {
    pub code: i32,      // 0 for success, otherwise error
    pub value: i32,     // Only valid if code == 0
}

#[no_mangle]
pub extern "C" fn safe_divide(a: i32, b: i32) -> MyResult {
    if b == 0 {
        MyResult { code: 1, value: 0 } // Divide-by-zero error
    } else {
        MyResult { code: 0, value: a / b }
    }
}
```

**C Side:**

```c
MyResult res = safe_divide(10, 0);
if (res.code == 0) {
    printf("Result is %d\n", res.value);
} else {
    printf("Error occurred: code %d\n", res.code);
}
```

A struct can also carry a pointer for an error message or additional info if needed.

### Pattern 4: Managing Memory for Error Messages

Passing dynamically-allocated error messages from Rust to C means you must provide freeing functions to avoid leaks.

**Rust Side:**

```rust
use std::ffi::CString;

#[no_mangle]
pub extern "C" fn divide_with_error(a: i32, b: i32, out: *mut i32) -> *mut libc::c_char {
    if b == 0 {
        let msg = CString::new("Division by zero!").unwrap();
        msg.into_raw()
    } else {
        unsafe { *out = a / b; }
        std::ptr::null_mut()
    }
}

#[no_mangle]
pub extern "C" fn free_error_string(s: *mut libc::c_char) {
    if !s.is_null() {
        unsafe { let _ = CString::from_raw(s); }
    }
}
```

**C Side:**

```c
int result;
char* err = divide_with_error(10, 0, &result);
if (err) {
    printf("Error: %s\n", err);
    free_error_string(err);
} else {
    printf("Result: %d\n", result);
}
```

Here, responsibility for freeing memory is clear.

***

## Critical Best Practices

1. **Never Panic Across FFI**
Panics must never cross FFI boundaries. Use `std::panic::catch_unwind` if you have Rust code that could panic:
```rust
#[no_mangle]
pub extern "C" fn dangerous_fn() -> i32 {
    let result = std::panic::catch_unwind(|| {
        // potentially panicking logic
        42
    });
    match result {
        Ok(val) => val,
        Err(_) => -1, // Error signal for panic
    }
}
```

2. **Use `#[repr(C)]` for Shared Structs**
Any struct or enum used across the boundary must be marked with `#[repr(C)]` to guarantee layout.
3. **Validate Input and Output Pointers**
Check all pointer arguments for null before use. This avoids UB and crashes.
4. **Favor Simple Error Codes When Possible**
The more complicated the error-handling contract, the harder it is to maintain. Stick to codes and access out-parameters for complex data.
5. **Free All Dynamically Allocated Memory**
If Rust returns heap-allocated memory to C, always provide a freeing function.
6. **Document Error Handling Contracts Clearly**
Write C header comments and Rust docstrings explaining what each error code or return value means.

***

By following these patterns, you ensure safe, predictable, and maintainable error communication between Rust and C, as well as robust cleanup and memory management. This makes your Rust FFI both a reliable bridge and a durable part of the systems software ecosystem.[^1][^2][^3]

1. https://users.rust-lang.org/t/best-practices-for-error-reporting-from-rust-to-c/18345
2. https://www.ralphminderhoud.com/blog/rust-ffi-wrong-way/
3. http://kmdouglass.github.io/posts/complex-data-types-and-the-rust-ffi/
4. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/error-handling.html
5. https://www.reddit.com/r/rust/comments/1ffz3fn/rust_error_handling_is_perfect_actually/
6. https://users.rust-lang.org/t/catching-exception-of-ffi/51052
7. https://blog.rust-lang.org/inside-rust/2020/11/23/What-the-error-handling-project-group-is-working-on.html
8. https://news.ycombinator.com/item?id=9545647
