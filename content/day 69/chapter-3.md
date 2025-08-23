---
title: "Profile-Guided Optimization in Rust"
description: "Explore Profile-Guided Optimization (PGO) in Rust to fine-tune binary performance."
date: 2025-11-13
weight: 3
---

# Profile-Guided Optimization (PGO) in Rust: A Beginner's Guide

Profile-Guided Optimization (PGO) is an advanced compiler technique that can fine-tune your Rust application's performance beyond what a standard release build can achieve. Think of it as **training a race car driver on the actual track before the main event**. A standard compiler is like a driver who has studied the map of the track but has never driven on it. They make smart, educated guesses about the best way to take each corner. PGO is like letting that driver do practice laps. By observing which paths are fastest and which corners are taken most often, the driver (the compiler) can devise a perfect, optimized strategy for the final race (your finished binary).

## What is Profile-Guided Optimization?

Normally, when you compile your code in release mode (`cargo build --release`), the compiler makes many optimizations based on static analysis and heuristics. It guesses which branches of an `if` statement are more likely, which functions are worth inlining, and how to best arrange the machine code. These guesses are good, but they're still just guesses.[^1]

PGO replaces these guesses with **real data**. The core idea is a two-phase process :[^2]

1. **Gather a Profile**: You first compile your program with special "instrumentation." This instrumented version is slightly slower, but as it runs, it collects data about its own execution—which functions are called most often, which branches are taken, which loops run many times.
2. **Optimize with the Profile**: You then feed this collected data back into the compiler. During a second compilation, the compiler uses this profile to make much smarter optimization decisions, tailored to how your application is *actually* used.

### What Does PGO Actually Optimize?

Using the profile data, the compiler can make more informed decisions about :[^3][^1]

- **Smarter Inlining**: It will more aggressively inline functions that are on "hot paths" (called very frequently), eliminating function call overhead where it matters most.
- **Optimized Branching**: For an `if/else` block, the compiler can arrange the machine code so that the most likely branch is executed with maximum efficiency (e.g., without a CPU jump instruction).
- **Better Code Layout**: It can place frequently executed code blocks close together in the final binary. This improves instruction cache locality, meaning the CPU spends less time waiting for instructions to be fetched from memory.
- **Improved Register Allocation**: It can prioritize keeping variables from hot paths in fast CPU registers.

The result is often a binary that is **5–15% faster** on typical workloads, with no changes to the source code.[^3]

## The PGO Workflow: A Step-by-Step Guide

There are two ways to use PGO: the manual method using `rustc` and `llvm-profdata`, and the much easier automated method using `cargo-pgo`. We'll cover both.

### The Easy Way: Using `cargo-pgo` (Recommended for Beginners)

The `cargo-pgo` tool automates the entire PGO workflow into a few simple commands.

#### Step 1: Install `cargo-pgo` and `llvm-tools`

```bash
# Install the cargo subcommand
cargo install cargo-pgo

# The PGO process needs tools from LLVM (the backend Rust uses).
# You can install them with rustup.
rustup component add llvm-tools-preview
```


#### Step 2: The Three `cargo-pgo` Commands

Let's assume you have a simple project with a `main.rs`.

```rust
// src/main.rs
fn hot_function() {
    // This function does a lot of work
}

fn cold_function() {
    // This function is rarely called
}

fn main() {
    for i in 0..1000 {
        // In a real app, this would be based on user input
        if i % 100 != 0 {
            hot_function();
        } else {
            cold_function();
        }
    }
}
```

Here is the entire PGO process using `cargo-pgo`:

1. **Build the instrumented binary:**

```bash
cargo pgo build
```

This compiles your program with instrumentation enabled.
2. **Run the binary to gather data:**

```bash
cargo pgo run
```

This runs your executable. It's crucial that you run it with a **representative workload**—that is, use it in a way that reflects how it will be used in production. The data gathered here will generate `.profraw` files.
3. **Build the final optimized binary:**

```bash
cargo pgo optimize
```

This command takes the profile data, processes it, and re-compiles your program using that data to guide optimizations. Your final, faster binary will be in the `target/release` directory.

That's it! `cargo-pgo` handles all the intermediate steps for you.[^2]

### The Manual Way: `rustc` and `llvm-profdata`

Understanding the manual process is useful for advanced use cases or troubleshooting. It involves four distinct steps.[^1][^2]

1. **Step 1: Compile with Instrumentation**
First, compile your program using the `-Cprofile-generate` flag. This creates the instrumented binary.

```bash
# Create a directory for the profile data
mkdir -p /tmp/pgo-data

# Compile the program
rustc -Cprofile-generate=/tmp/pgo-data -O src/main.rs -o my_app_instrumented
```

2. **Step 2: Run the Instrumented Program**
Run the binary you just created. This will generate a `default.profraw` file in the directory you specified.

```bash
./my_app_instrumented
```

You should run the program multiple times with different typical inputs to get a rich profile.
3. **Step 3: Merge the Profile Data**
The `.profraw` files are raw data. They need to be merged into a single `.profdata` file that the compiler can understand. This is done with `llvm-profdata`, which you installed with `llvm-tools-preview`.

```bash
# Find the path to your llvm-profdata tool
LLVM_PROFDATA=$(rustc --print target-libdir)/../bin/llvm-profdata

# Merge the raw profiles
$LLVM_PROFDATA merge -o /tmp/pgo-data/merged.profdata /tmp/pgo-data
```

4. **Step 4: Re-compile with the Profile**
Finally, compile your program again, but this time use the `-Cprofile-use` flag, pointing it to your merged profile data.

```bash
rustc -Cprofile-use=/tmp/pgo-data/merged.profdata -O -Zunstable-options --out-dir target/release src/main.rs
```

The resulting binary in `target/release` is now PGO-optimized.

## When Should You Use PGO?

Profile-Guided Optimization is not a magic bullet for all applications. It is most effective when:

- Your application has **hot and cold paths**—that is, some code paths are executed far more frequently than others. Web servers, databases, compilers, and video games are excellent candidates.
- The application's workload is **predictable and repeatable**. The quality of the optimization depends entirely on the quality of the profile data you gather.
- You have already performed other optimizations, and you are looking to squeeze out the last **5-15% of performance**.

PGO is a powerful, advanced tool in the Rust performance ecosystem. While the manual process is complex, `cargo-pgo` makes it accessible to all developers, allowing you to build highly-tuned applications based on real-world usage data.

1. https://rustc-dev-guide.rust-lang.org/profile-guided-optimization.html
2. https://doc.rust-lang.org/rustc/profile-guided-optimization.html
3. https://www.youtube.com/watch?v=1RmNfO84z_M
4. https://kobzol.github.io/rust/cargo/2023/07/28/rust-cargo-pgo.html
5. https://israelo.io/blog/pgo/
6. https://www.ibm.com/docs/en/osfroa/1.84?topic=started-profile-guided-optimization-pgo
7. https://www.youtube.com/watch?v=_EpALMNXM24
8. https://www.datocms-assets.com/98516/1734435430-zaitsau_2024.pdf
9. https://www.youtube.com/watch?v=VPcH63xrysY
