---
title: "Foreign Function Interface - bindgen for C Bindings"
description: "Explore how bindgen can automatically generate Rust FFI bindings from C headers."
date: 2025-10-15
weight: 2
---

# Foreign Function Interface: Automating C Bindings with `bindgen`

After learning how to manually call C from Rust, you might be thinking, "That's great for one or two functions, but what if I need to interface with a massive C library that has hundreds of functions and dozens of structs?" Manually writing the `extern "C"` blocks for a large API would be incredibly tedious, error-prone, and a nightmare to maintain.

This is the exact problem that `bindgen` solves.

## The UN Translator Analogy: From Human to Machine Translation 🌐

If manual FFI is like being a **human translator** carefully translating a document sentence by sentence, then `bindgen` is like using a **sophisticated, AI-powered machine translation service**.

- **Manual FFI**: You, the human, read the C header file and manually write the equivalent Rust `extern` block. It's precise but slow, and you might make mistakes with complex types.
- **`bindgen`**: You feed the entire C header file into the `bindgen` machine. It instantly analyzes all the functions, structs, enums, and type definitions and automatically generates the complete, accurate Rust FFI code for you.

`bindgen` is a build-time tool that automatically creates Rust FFI bindings from C (and some C++) header files, saving you immense time and preventing manual errors.[^2][^3]

## How `bindgen` Works: The `build.rs` Script

`bindgen` operates as part of Rust's build process via a special file called `build.rs`. This is a script that Cargo executes *before* compiling your main crate. It can be used to compile C code, generate code, or perform other build-time tasks.[^1]

The typical workflow with `bindgen` is:

1. You create a `build.rs` script in your project root.
2. In `build.rs`, you tell the `cc` crate how to compile your C source code into a static library.
3. You then tell the `bindgen` crate to read your C header file.
4. `bindgen` generates a `bindings.rs` file containing all the necessary Rust FFI declarations.
5. Your main Rust code then simply includes and uses the auto-generated `bindings.rs` file.

## Step-by-Step Example: Using `bindgen` for a Custom C Library

Let's build a slightly more complex C library than before—one with a `struct`—to see `bindgen`'s power. We'll create a library that calculates the distance between two 2D points.

### Step 1: The C Library

Create a directory for our C library.
**File: `point_lib/point.h`**

```c
#include <stdint.h>

// A C struct representing a point in 2D space.
typedef struct {
    double x;
    double y;
} Point;

// A function that calculates the distance between two points.
double calculate_distance(Point p1, Point p2);
```

**File: `point_lib/point.c`**

```c
#include "point.h"
#include <math.h> // We need this for sqrt()

double calculate_distance(Point p1, Point p2) {
    double dx = p1.x - p2.x;
    double dy = p1.y - p2.y;
    return sqrt(dx*dx + dy*dy);
}
```


### Step 2: The Rust Project Setup

Create a new Rust project: `cargo new bindgen_example`.

**`Cargo.toml`**
This setup is crucial. We need `bindgen` and `cc` as **build dependencies**, meaning they are only used during the build process and are not part of the final application's dependencies.[^4]

```toml
[package]
name = "bindgen_example"
version = "0.1.0"
edition = "2021"

[dependencies]

[build-dependencies]
bindgen = "0.69" # Or the latest version
cc = "1.0"
```


### Step 3: The Build Script (`build.rs`)

This is where the magic happens. Create `build.rs` in the root of your `bindgen_example` project.

**File: `build.rs`**

```rust
use std::env;
use std::path::PathBuf;

fn main() {
    // 1. COMPILE THE C LIBRARY
    // Tell Cargo to re-run this script if any of our C code changes.
    println!("cargo:rerun-if-changed=point_lib/point.c");
    println!("cargo:rerun-if-changed=point_lib/point.h");

    // Use the `cc` crate to compile our C code into a static library.
    cc::Build::new()
        .file("point_lib/point.c")
        .compile("point"); // Outputs `libpoint.a`

    // Tell Cargo to link our Rust code with the C static library.
    println!("cargo:rustc-link-lib=static=point");
    // Also link against the standard math library, since our C code uses `sqrt`.
    println!("cargo:rustc-link-lib=m");


    // 2. GENERATE THE RUST BINDINGS
    // Create a `bindgen` builder.
    let bindings = bindgen::Builder::default()
        // The header file is the input to `bindgen`.
        .header("point_lib/point.h")
        // Tell cargo to invalidate the built crate whenever any of the
        // included header files changed.
        .parse_callbacks(Box::new(bindgen::CargoCallbacks))
        // Finish the builder and generate the bindings.
        .generate()
        // `unwrap` if binding generation fails.
        .expect("Unable to generate bindings");

    // Write the bindings to a file in the `OUT_DIR`.
    // `OUT_DIR` is a directory inside `target` where build artifacts are placed.
    let out_path = PathBuf::from(env::var("OUT_DIR").unwrap());
    bindings
        .write_to_file(out_path.join("bindings.rs"))
        .expect("Couldn't write bindings!");
}
```


### Step 4: Using the Generated Bindings in Rust

Now, our `main.rs` can use the bindings that `build.rs` will generate for us.

**File: `src/main.rs`**

```rust
// This `include!` macro will paste the contents of the generated `bindings.rs`
// file directly into our code. The file is located in the `OUT_DIR`.
include!(concat!(env!("OUT_DIR"), "/bindings.rs"));

// Let's create a safe wrapper around our unsafe FFI function.
// This is a best practice to contain the `unsafe` logic.
fn distance_safe(p1: Point, p2: Point) -> f64 {
    // The generated function is unsafe because bindgen cannot guarantee
    // the C code is safe.
    unsafe { calculate_distance(p1, p2) }
}

fn main() {
    println!("Calling C code from Rust using bindgen!");

    // We can use the `Point` struct directly, as bindgen generated
    // a Rust-compatible version for us.
    let point1 = Point { x: 1.0, y: 1.0 };
    let point2 = Point { x: 4.0, y: 5.0 };

    // Call our safe wrapper function.
    let distance = distance_safe(point1, point2);

    println!("The C library calculated the distance as: {}", distance);
    assert_eq!(distance, 5.0);
}
```

*Note: You will need a C compiler like `gcc` or `clang` installed on your system for the `cc` crate to work.*

Now, just run `cargo run`. Cargo will first execute `build.rs`, which compiles the C code and generates `bindings.rs`. Then, it will compile `main.rs`, linking everything together into a single executable.

## Why `bindgen` is a Game-Changer

1. **Accuracy and Completeness**: `bindgen` uses `libclang` to parse C code, so it understands complex C features like structs, unions, enums, function pointers, and even macros. It generates correct and complete bindings automatically.
2. **Greatly Reduced Effort**: For a library with hundreds of functions, `bindgen` turns days or weeks of tedious, error-prone manual work into a few lines in a build script.
3. **Maintainability**: When the C library is updated, you simply re-run `cargo build`. `bindgen` will regenerate the bindings, immediately highlighting any breaking changes in the C API as compile-time errors in your Rust code.
4. **Customization**: The `bindgen::Builder` is highly configurable. You can choose to generate bindings only for specific functions or types (`allowlist`), block others (`blocklist`), map C types to custom Rust types, and much more.[^5]

By automating the most tedious part of FFI, `bindgen` makes it feasible and safe to integrate Rust with the massive world of existing C libraries, allowing you to build on decades of existing work while leveraging Rust's safety and performance.

1. https://blog.theembeddedrustacean.com/rust-ffi-and-bindgen-integrating-embedded-c-code-in-rust
2. https://rust-lang.github.io/rust-bindgen/
3. https://github.com/rust-lang/rust-bindgen
4. https://www.youtube.com/watch?v=KWrfxKUBIuo
5. https://n8henrie.com/2021/11/the-simplest-rust-c-ffi-example/
6. https://www.w3resource.com/rust-tutorial/rust-ffi-guide.php
7. https://doc.rust-lang.org/nomicon/ffi.html
8. https://www.youtube.com/watch?v=pePqWoTnSmQ
9. https://www.reddit.com/r/rust/comments/18vnsf2/rusts_ffi_with_c/
10. https://rust-lang.github.io/rust-bindgen/tutorial-6.html
