---
title: "build.rs Scripts"
description: "Learn how build.rs scripts work to automate tasks during the build process in Rust."
date: 2025-10-08
weight: 1
---

# `build.rs` Scripts in Rust: A Beginner's Guide to Build-Time Automation

Understanding `build.rs` scripts in Rust is like learning to **give your kitchen staff a prep list to complete *before* you start cooking the main dishes**. Instead of manually setting up everything during the main cooking process, you have a dedicated, automated step that prepares special ingredients, fetches tools from another workshop, or even builds a custom piece of equipment. `build.rs` is that automated prep list for your Rust crate, allowing you to perform tasks and generate code *before* your main crate is compiled.

## The Professional Kitchen Prep List Analogy 🍽️

### Imagine You're a Head Chef Preparing for a Busy Night

- **Your Rust Crate (`src` directory)**: The main recipes you will be cooking tonight.
- **A `build.rs` Script**: A special, detailed prep list you give to your assistants.
- **Cargo**: Your kitchen manager, who ensures the prep list is completed before the main cooking starts.
- **External Dependencies (like a C library)**: A specialized ingredient or tool you need to get from another supplier or workshop.

***

### The Problem without a Prep List

Without a dedicated prep step, you'd have to do everything during the main cooking process. Imagine having to stop in the middle of sautéing to go forge a new knife or grind a special spice blend from raw ingredients. This would be slow, inefficient, and mix concerns.

### The Power of `build.rs`

A `build.rs` script allows you to handle these preparatory tasks separately and automatically :[^7]

- "Go to the metal workshop (a C library) and build me a custom meat grinder (`cc` crate)."
- "Look at this old family recipe book (a C header file) and translate it into our modern recipe format (Rust FFI bindings using `bindgen`)."
- "Based on today's date, generate a 'special of the day' recipe card and place it in the kitchen (`OUT_DIR`)."

This makes the main cooking process cleaner, faster, and more focused.

## How `build.rs` Works

1. **Placement**: You create a file named `build.rs` in the **root directory** of your crate (alongside `Cargo.toml` and `src`).[^2]
2. **Detection**: When you run `cargo build`, Cargo sees the `build.rs` file.
3. **Execution**: *Before* compiling your main crate code (in `src`), Cargo compiles and runs your `build.rs` script as a standalone Rust program.
4. **Communication**: The build script can communicate instructions back to Cargo by printing special formatted lines to standard output (e.g., `println!("cargo:...")`).[^6]
5. **Compilation**: After the build script finishes successfully, Cargo proceeds to compile your main crate, taking into account any instructions it received from the script.

## Common Use Cases and Examples

### 1. Generating Rust Code at Build Time

This is one of the simplest yet most powerful use cases. The build script can generate a `.rs` file, which your main crate code can then include using the `include!` macro.

**Scenario**: You want to include the current git commit hash in your application's version string.

**Project Setup**:

```
my_app/
├── Cargo.toml
├── build.rs
└── src/
    └── main.rs
```

**`build.rs`**

```rust
use std::process::Command;
use std::env;
use std::path::Path;
use std::fs;

fn main() {
    // 1. Get the git commit hash
    let output = Command::new("git")
        .args(&["rev-parse", "HEAD"])
        .output()
        .expect("Failed to execute git command");
    let git_hash = String::from_utf8(output.stdout).unwrap();

    // 2. Find the output directory designated by Cargo
    let out_dir = env::var("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("commit_hash.rs");

    // 3. Write the generated code to a file
    fs::write(
        &dest_path,
        format!("pub const GIT_HASH: &str = \"{}\";", git_hash.trim())
    ).unwrap();

    // 4. Tell Cargo to rerun this script if build.rs changes
    println!("cargo:rerun-if-changed=build.rs");
}
```

**`src/main.rs`**

```rust
// Use the include! macro to bring the generated code into this scope.
// The env!("OUT_DIR") macro gives us the path to the output directory.
include!(concat!(env!("OUT_DIR"), "/commit_hash.rs"));

fn main() {
    println!("Application Version: 0.1.0");
    println!("Built from git commit: {}", GIT_HASH);
}
```

**How it works**: The build script runs `git`, gets the hash, and writes a file like `commit_hash.rs` containing `pub const GIT_HASH: &str = "a1b2c3d...";` into a special directory managed by Cargo (`OUT_DIR`). The `main.rs` file then uses `include!` to pull that generated code directly into its compilation, making the `GIT_HASH` constant available.

### 2. Compiling and Linking to Native C/C++ Libraries

This is a very common use case for crates that provide bindings to existing native libraries (often called `-sys` crates).[^1]

**Scenario**: Your Rust application needs to call a simple function written in C.

**Project Setup**:

```
hello_from_c/
├── Cargo.toml
├── build.rs
└── src/
    ├── hello.c
    └── main.rs
```

**Add a build dependency in `Cargo.toml`**:

```toml
[build-dependencies]
cc = "1.0" # A helper crate for compiling C/C++/asm
```

**`src/hello.c`**

```c
#include <stdio.h>

void hello_from_c() {
    printf("Hello from the C world!\n");
}
```

**`build.rs`**

```rust
fn main() {
    // 1. Use the `cc` crate to compile our C file.
    cc::Build::new()
        .file("src/hello.c")
        .compile("libhello.a"); // The output will be a static library

    // 2. Tell Cargo to rerun this script if the C file changes.
    println!("cargo:rerun-if-changed=src/hello.c");
}
```

**`src/main.rs`**

```rust
// Declare the C function's signature using `extern "C"`
extern "C" {
    fn hello_from_c();
}

fn main() {
    println!("Hello from the Rust world!");
    unsafe {
        // Calling C functions is unsafe because the Rust compiler
        // cannot guarantee their memory safety.
        hello_from_c();
    }
}
```

**How it works**: The `build.rs` script uses the `cc` crate to invoke the system's C compiler (like `gcc` or `clang`), compile `hello.c`, and create a static library (`libhello.a`). The `cc` crate automatically handles printing the necessary `cargo:rustc-link-lib` and `cargo:rustc-link-search` instructions so that Cargo knows how to link this new library to your final Rust executable.

### 3. Generating FFI Bindings with `bindgen`

Manually writing `extern "C"` blocks for large C libraries is tedious and error-prone. The `bindgen` crate can automate this by reading a C header file and generating the corresponding Rust FFI bindings.

**Scenario**: You want to use the `bzip2` compression library from Rust.

**`build.rs` (Conceptual Example)**:

```rust
// (Requires adding `bindgen` as a build-dependency)
use std::env;
use std::path::PathBuf;

fn main() {
    // Tell Cargo to link to the system's bzip2 library
    println!("cargo:rustc-link-lib=bz2");

    // Generate the bindings
    let bindings = bindgen::Builder::default()
        .header("wrapper.h") // A simple C header that includes bzlib.h
        .generate()
        .expect("Unable to generate bindings");

    // Write the bindings to a file in OUT_DIR
    let out_path = PathBuf::from(env::var("OUT_DIR").unwrap());
    bindings
        .write_to_file(out_path.join("bindings.rs"))
        .expect("Couldn't write bindings!");

    println!("cargo:rerun-if-changed=wrapper.h");
}
```

This is a very powerful pattern that makes it much easier to interface with the vast ecosystem of existing C libraries.[^5]

## Key `cargo:` Instructions

The build script communicates with Cargo via `stdout`. Here are the most important instructions :[^2]

- `cargo:rustc-link-lib=[KIND=]NAME`: Tells Cargo to link to a native library. `KIND` can be `static`, `dylib` (default), or `framework`.
- `cargo:rustc-link-search=[KIND=]PATH`: Adds a path to the library search path.
- `cargo:rerun-if-changed=PATH`: Tells Cargo to re-run the build script only if the file at `PATH` has changed. This is crucial for performance, as it avoids re-running the script on every build.
- `cargo:rerun-if-env-changed=VAR`: Re-runs the script if an environment variable has changed.
- `cargo:rustc-cfg=KEY[="VALUE"]`: Enables conditional compilation via `#[cfg(KEY)]` or `#[cfg(KEY = "VALUE")]` in your crate code.

By mastering `build.rs`, you gain the ability to automate complex build-time tasks, seamlessly integrate with non-Rust code, and generate code dynamically, making your Rust projects more powerful and versatile.
<span style="display:none">[^10][^3][^4][^8][^9]</span>

1. https://doc.rust-lang.org/cargo/reference/build-script-examples.html
2. https://doc.rust-lang.org/cargo/reference/build-scripts.html
3. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/cargo/reference/build-scripts.html
4. https://www.geeksforgeeks.org/rust/rust-build-scripts/
5. https://rust-lang.github.io/rust-bindgen/tutorial-3.html
6. https://dev.to/deciduously/automatically-generate-rust-modules-with-cargo-build-scripts-157h
7. https://doc.rust-lang.org/rust-by-example/cargo/build_scripts.html
8. https://www.reddit.com/r/rust/comments/6b40pb/how_to_write_buildrs_scripts_properly/
9. https://kazlauskas.me/entries/writing-proper-buildrs-scripts
10. https://stackoverflow.com/questions/74049386/is-it-possible-to-have-example-specific-build-rs-file
