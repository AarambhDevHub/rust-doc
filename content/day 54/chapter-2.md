---
title: "Environment Variables"
description: "Understand how to use and manage environment variables inside build scripts for flexible builds."
date: 2025-10-08
weight: 2
---

# Using Environment Variables in Rust Build Scripts: A Beginner's Guide

Understanding how to use environment variables in Rust build scripts (`build.rs`) is like learning to **give special, last-minute instructions to your kitchen staff before they start cooking**. Imagine you need to build a C library as part of your Rust project. You might need to tell the C compiler where to find a specific tool or what optimization level to use. Instead of hardcoding this information, you can use environment variables to make your build process flexible, configurable, and adaptable to different systems.

## The Professional Kitchen Build Analogy 🍽️

### Imagine You're a Head Chef Preparing for a Big Service

- **Your Rust Crate**: The final, complete dish you will serve.
- **The Build Process (`cargo build`)**: The entire process of preparing and cooking the dish.
- **A Build Script (`build.rs`)**: A special prep-chef who performs crucial tasks *before* the main cooking begins (e.g., grinding spices, finding a special tool, or pre-cooking an ingredient).
- **Environment Variables**: A note you leave for the prep-chef with specific instructions for today's service (e.g., `SPICE_LEVEL=ExtraHot` or `TOOL_PATH=/usr/local/bin/special_whisk`).

Without these notes, the prep-chef would have to guess or use default settings, which might not be right for every situation. Environment variables allow you to control the build process from the outside.

## What is a Build Script?

A build script is a special Rust file named `build.rs` that you place in the root of your crate. Cargo will compile and run this script *before* it compiles your main crate code.[^2]

**Common uses for build scripts:**

- Compiling and linking against third-party C/C++ libraries.
- Generating Rust code from a specification (e.g., from a Protobuf file).
- Discovering system information or dependencies at build time.
- Setting configuration flags based on the target platform or enabled features.


## Two-Way Communication: Environment Variables in Build Scripts

Environment variables are the primary way to communicate with and from your build script. The communication flows in two directions:

1. **Cargo to Build Script**: Cargo sets several environment variables to provide your `build.rs` with context about the build process.
2. **Build Script to Cargo/Crate**: Your `build.rs` can print special instructions to `stdout` to pass information back to Cargo, which can then set environment variables for your main crate's compilation.

### 1. Information from Cargo to the Build Script

When Cargo runs your `build.rs`, it sets many useful environment variables. You can access these using `std::env::var()` inside your build script.[^1][^6]

**Key Environment Variables Provided by Cargo:**


| Variable | Description | Example Use Case |
| :-- | :-- | :-- |
| **`OUT_DIR`** | The path to a directory where your script can place generated files. This is the **only** place you should write to [^7]. | Storing a compiled C library or a generated Rust module. |
| **`TARGET`** | The "target triple" for the build (e.g., `x86_64-unknown-linux-gnu`). | Choosing the correct C compiler or settings for cross-compilation. |
| **`PROFILE`** | The current build profile (e.g., `debug` or `release`). | Applying higher optimization levels to a C library in a `release` build. |
| **`CARGO_MANIFEST_DIR`** | The path to your crate's root directory (where `Cargo.toml` is). | Finding local assets or configuration files needed for the build. |
| **`CARGO_FEATURE_<NAME>`** | Set for each enabled feature (e.g., `CARGO_FEATURE_MY_FEATURE`). | Conditionally compiling code based on which features are active. |

**Example `build.rs`: Using `OUT_DIR` and `PROFILE`**

Let's imagine we're compiling a C library and want to enable optimizations only for release builds.

```rust
// build.rs
use std::env;

fn main() {
    // Get the build profile
    let profile = env::var("PROFILE").unwrap();

    let mut builder = cc::Build::new();
    builder.file("src/my_c_code.c");

    // Conditionally set the optimization level
    if profile == "release" {
        println!("Build script: Applying release optimizations to C code.");
        builder.opt_level(3);
    } else {
        println!("Build script: Building C code in debug mode.");
        builder.opt_level(0);
    }

    builder.compile("my_c_library");

    // Tell Cargo to rerun this script only if our C code changes.
    println!("cargo:rerun-if-changed=src/my_c_code.c");
}
```


### 2. Information from the Build Script back to Your Crate

This is where things get powerful. Your build script can communicate back to Cargo by printing specially formatted strings to standard output. The most common instruction is `cargo:rustc-env=VAR=VALUE`, which tells Cargo to set an environment variable for the compilation of your *actual crate code*.[^2]

**Key Instructions for Cargo:**


| Instruction | Purpose |
| :-- | :-- |
| **`cargo:rustc-env=VAR=VALUE`** | Sets an environment variable `VAR` to `VALUE` for your crate's compilation. |
| **`cargo:rustc-link-lib=NAME`** | Tells Cargo to link against a specific native library (e.g., `crypto` for `libcrypto`). |
| **`cargo:rustc-link-search=PATH`** | Adds a path for the linker to search for libraries. |
| **`cargo:rerun-if-changed=PATH`** | Tells Cargo to re-run the build script only if the specified file or directory changes. This is crucial for performance. |
| **`cargo:rerun-if-env-changed=VAR`** | Tells Cargo to re-run the build script if an external environment variable changes [^9]. |

**Accessing the Variable in Your Crate**

Once your build script sets a variable with `rustc-env`, you can access it in your crate code using the `env!` macro. This macro reads the environment variable at **compile time** and embeds its value directly into your binary.

**Example: Embedding a Build-Time Timestamp**

Let's create a build script that records the build timestamp and makes it available to our application.

**`build.rs`**

```rust
// build.rs
use std::time::{SystemTime, UNIX_EPOCH};

fn main() {
    // Get the current Unix timestamp
    let timestamp = SystemTime::now()
        .duration_since(UNIX_EPOCH)
        .unwrap()
        .as_secs();

    // Set the BUILD_TIMESTAMP environment variable for our crate
    println!("cargo:rustc-env=BUILD_TIMESTAMP={}", timestamp);
}
```

**`src/main.rs`**

```rust
// src/main.rs

// The `env!` macro reads the environment variable at compile time.
// The value of `BUILD_TIMESTAMP` will be hardcoded into our final executable.
const BUILD_TIME: &str = env!("BUILD_TIMESTAMP");

fn main() {
    println!("This program was built at Unix timestamp: {}", BUILD_TIME);

    // You can parse it if needed
    let build_timestamp: u64 = BUILD_TIME.parse().unwrap();
    println!("That's {} seconds since the epoch.", build_timestamp);
}
```

**How It Works:**

1. `cargo build` first runs `build.rs`.
2. The build script calculates the timestamp and prints `cargo:rustc-env=BUILD_TIMESTAMP=1660920000` (or whatever the current time is) to its stdout.
3. Cargo sees this line and sets the `BUILD_TIMESTAMP` environment variable for the *next step*.
4. Cargo compiles `src/main.rs`. When the compiler encounters `env!("BUILD_TIMESTAMP")`, it replaces it with the literal string `"1660920000"`.
5. The final binary contains this hardcoded value.

## Reading External Environment Variables

Your build script can also react to environment variables set by the user *before* they run `cargo build`. This is perfect for allowing users to configure your build process.

**Example: Optional Feature Controlled by an Environment Variable**

Let's allow the user to enable a "deluxe" feature by setting `DELUXE_MODE=1` in their shell.

**`build.rs`**

```rust
// build.rs
use std::env;

fn main() {
    // Check if the user has set the DELUXE_MODE variable.
    // `env::var` returns a `Result`, so we use `is_ok()` to check for presence.
    if env::var("DELUXE_MODE").is_ok() {
        // If it's set, we tell Cargo to enable a specific `cfg` flag for our crate.
        println!("cargo:rustc-cfg=deluxe_mode_enabled");
    }

    // Tell Cargo to re-run this script if this environment variable changes.
    println!("cargo:rerun-if-env-changed=DELUXE_MODE");
}
```

**`src/main.rs`**

```rust
// src/main.rs

fn main() {
    println!("Welcome to our application!");

    // Use the `#[cfg(...)]` attribute to conditionally compile code.
    // This block will only be included in the final binary if the build script
    // emitted the `deluxe_mode_enabled` flag.
    #[cfg(deluxe_mode_enabled)]
    {
        println!("You are running in DELUXE MODE!");
    }
}
```

**How a user would run this:**

```bash
# Standard build (no deluxe mode)
cargo build

# Build with deluxe mode enabled
DELUXE_MODE=1 cargo build
```

This pattern is extremely powerful for creating flexible and configurable builds without cluttering your `Cargo.toml` with dozens of features.
<span style="display:none">[^10][^3][^4][^5][^8]</span>

1. https://doc.rust-lang.org/cargo/reference/environment-variables.html
2. https://doc.rust-lang.org/cargo/reference/build-scripts.html
3. https://stackoverflow.com/questions/57017066/how-do-i-set-environment-variables-with-cargo
4. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/cargo/reference/build-scripts.html
5. https://users.rust-lang.org/t/std-set-var-in-build-rs-not-setting-environment-variable/34924
6. https://carols10cents.github.io/cargo/environment-variables.html
7. https://doc.rust-lang.org/cargo/reference/build-script-examples.html
8. https://www.reddit.com/r/rust/comments/12te0wp/how_to_pass_env_var_to_dependency_build_script/
9. https://github.com/rust-lang/cargo/issues/12403
10. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/cargo/reference/environment-variables.html
