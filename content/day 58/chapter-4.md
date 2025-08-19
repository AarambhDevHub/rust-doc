---
title: "Foreign Function Interface - String Handling Across FFI"
description: "Handle Rust and C strings across the FFI boundary while avoiding memory issues."
date: 2025-10-15
weight: 4
---

# FFI String Handling in Rust: Safe and Practical Approaches

Handling strings between Rust and C is a common task in Foreign Function Interface (FFI), but it's also a potential source of bugs if not done correctly. In Rust, strings are UTF-8 encoded and either owned (`String`) or borrowed (`&str`), while in C, strings are simply pointers to bytes ending in a null (`'\0'`) byte, and might not be valid UTF-8. Crossing the FFI boundary means converting between these representations safely—without leaking memory, causing undefined behavior, or corrupting your data.

## Key Differences: Rust Strings vs. C Strings

| Aspect | Rust | C |
| :-- | :-- | :-- |
| Encoding | UTF-8 | Arbitrary bytes (often ASCII, not always UTF-8) |
| Ownership | Owned (`String`), borrowed (`&str`) | Raw pointer (`char *`, often borrowed) |
| Null bytes | May not contain nulls (`\0`) | Must end with null (`\0`) |
| Length | Explicit (stored separately) | Implicit (calculated by searching for null) |
| Safety | Enforced by Rust | Must be validated manually |

## Sending Strings from Rust to C

Rust provides the `CString` type for sending strings to C code. This type guarantees that your Rust string is null-terminated and free of any interior nulls (since an interior null would truncate the C string).

**Example: Passing a Rust String to C**

```rust
use std::ffi::CString;
use std::os::raw::c_char;

extern "C" {
    fn print_message(msg: *const c_char);
}

fn main() {
    // Create a Rust string
    let my_msg = "Hello from Rust!";
    // Convert to a C-compatible, null-terminated string
    let c_msg = CString::new(my_msg).expect("CString::new failed");
    // Pass to C function (unsafe!)
    unsafe {
        print_message(c_msg.as_ptr());
    }
}
```

**Important**:

- Use `CString` only for strings that **do not contain embedded nulls** (`\0`). If your data might, consider sending bytes and their length separately.
- The pointer from `as_ptr()` stays valid as long as the `CString` is kept alive (not dropped).


## Receiving Strings from C in Rust

For strings received from C (as `*const c_char`), Rust offers the `CStr` type, which lets you "borrow" the memory location of a C string and safely work with it.

**Example: Converting a C String to a Rust String**

```rust
use std::ffi::CStr;
use std::os::raw::c_char;

extern "C" {
    fn get_message() -> *const c_char;
}

fn main() {
    unsafe {
        let c_msg = get_message();
        if !c_msg.is_null() {
            // Make a safe Rust view
            let rust_str = CStr::from_ptr(c_msg)
                .to_str()
                .expect("C string was not valid UTF-8");
            println!("Received from C: {}", rust_str);
        } else {
            println!("C returned null pointer!");
        }
    }
}
```

**Best Practices:**

- Always check for null pointers before using `CStr::from_ptr`.
- `.to_str()` checks for valid UTF-8; if you can't guarantee UTF-8, use `.to_bytes()` for raw byte access.
- If the C string's memory is owned by C, do **not** free it in Rust unless specifically instructed.


## Handling Memory Safely

- **Who owns the data?** If C allocates a string and expects Rust to free it, you'll need to call the corresponding C `free` function using FFI. If Rust allocates (`CString`), Rust manages the memory.
- Avoid passing pointers from data that Rust will drop—dangling pointers will cause undefined behavior.


## Full Roundtrip Example

Let's combine everything by round-tripping a string from Rust to C and back.

**C Code:**

```c
// C file: message.c
#include <string.h>
#include <stdio.h>

void print_message(const char* msg) {
    printf("Message from Rust: %s\n", msg);
}

const char* get_message() {
    return "Hello from C!";
}
```

**Rust Code:**

```rust
use std::ffi::{CString, CStr};
use std::os::raw::c_char;

extern "C" {
    fn print_message(msg: *const c_char);
    fn get_message() -> *const c_char;
}

fn main() {
    // Send message from Rust to C
    let rust_msg = CString::new("Hello from Rust!").unwrap();
    unsafe { print_message(rust_msg.as_ptr()); }

    // Receive a message from C to Rust
    unsafe {
        let c_ptr = get_message();
        if !c_ptr.is_null() {
            let rust_string = CStr::from_ptr(c_ptr).to_str().unwrap();
            println!("Received from C: {}", rust_string);
        }
    }
}
```


## Additional Tips

- Use `CString` and `CStr` as much as possible—these types prevent most string mistakes automatically.
- Document your safety guarantees for any `unsafe` FFI code you write.
- When passing strings in either direction, clarify which side is responsible for allocation and deallocation (memory ownership).
- Prefer passing string *lengths* explicitly if your API might need to transfer arbitrary bytes, including embedded nulls (for binary data).
- For complex APIs, consider using `bindgen` to automate FFI bindings and reduce errors.


## Summary Table

| Direction | Rust Type | C Type | Conversion | Memory Safety |
| :--: | :--: | :--: | :--: | :--: |
| Rust → C | `&str` | `const char*` | `CString::new(str).as_ptr()` | Prefer lifetime in Rust |
| C → Rust | `const char*` | `&str`/`String` | `CStr::from_ptr(ptr).to_str()` | Check for null, UTF-8 validity |

Handling strings safely across FFI requires understanding how memory and encoding differ between Rust and C. By leaning on Rust's safe abstractions like `CString` and `CStr`, and following best practices, you can write reliable, memory-safe bindings without surprises.[^1][^2][^3][^4]
<span style="display:none">[^5][^6][^7][^8][^9]</span>

1. https://doc.rust-lang.org/std/ffi/struct.CString.html
2. https://rust-unofficial.github.io/patterns/idioms/ffi/accepting-strings.html
3. https://webreference.com/rust/systems/ffi-c-interop/
4. https://doc.rust-lang.org/std/ffi/struct.CStr.html
5. https://www.reddit.com/r/rust/comments/pw9cga/how_do_you_handle_strings_in_ffi/
6. https://doc.rust-lang.org/nomicon/ffi.html
7. https://www.ralphminderhoud.com/blog/rust-ffi-wrong-way/
8. https://stackoverflow.com/questions/73501840/rust-ffi-lib-share-string-free-from-both
9. https://dev.to/kgrech/7-ways-to-pass-a-string-between-rust-and-c-4ieb
