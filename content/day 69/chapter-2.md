---
title: "Link-Time Optimization (LTO) in Rust"
description: "Learn how to use Link-Time Optimization (LTO) in Rust to improve runtime performance."
date: 2025-11-13
weight: 2
---

# Link-Time Optimization (LTO) in Rust: A Beginner's Guide to Better Performance

Understanding Link-Time Optimization (LTO) in Rust is like discovering a way for your entire kitchen staff to collaborate on a "whole-restaurant" level just before serving a meal. Normally, each chef optimizes their own dish in isolation. With LTO, the head chef (the **linker**) gets to look at *all* the finished dishes together and perform final, powerful optimizations that cross individual dish boundaries—like sharing a common garnish or noticing that two separate recipes can be combined into one more efficient process. This "whole-program" view can lead to significant improvements in both the speed and size of your final application.

## How Rust Usually Compiles (Without LTO)

To understand LTO, you first need to know how Rust compiles code normally.

1. **Individual Compilation**: The Rust compiler takes each of your source code files (and the libraries you depend on) and compiles them into separate "object files." During this step, it performs many optimizations *within* each file.
2. **Linking**: A separate program called the **linker** takes all these pre-compiled object files and "links" them together into a single executable file.

The problem with this approach is that when the compiler is working on `file_A.o`, it has no knowledge of the code inside `file_B.o`. It can't perform optimizations that span across these boundaries, such as **inlining** a function from one file into another.[^1]

## What is Link-Time Optimization (LTO)?

LTO changes this process. Instead of giving the linker fully compiled object files, the compiler gives it object files containing **Intermediate Representation (IR)**—a more abstract, not-yet-finalized version of the code.[^2][^3]

This allows the linker, guided by the compiler, to look at the entire program's IR all at once. With this "whole-program" view, it can perform powerful new optimizations that were previously impossible, such as :[^4]

- **Cross-Crate Inlining**: A function from an external library (crate) can be directly embedded into your code, eliminating the overhead of a function call.
- **Better Dead Code Elimination**: If a function in a library is never actually used by your final program, the linker can completely remove it.
- **More Aggressive Optimizations**: With a global view, the compiler can make more intelligent decisions about how to rearrange code and optimize data flow across the entire application.

The result is often a smaller, faster binary. The trade-off is that this process is more complex and significantly **increases compile times**. Therefore, LTO is typically only enabled for `release` builds.

## How to Enable LTO in Rust

You can enable LTO in your project by adding a few lines to your `Cargo.toml` file under the `[profile.release]` section. Rust offers several levels of LTO.[^4]

### 1. `lto = "thin"` (The Recommended Default)

ThinLTO is a more modern, incremental approach to LTO. It allows for some parallelism and is a great balance between compilation time and performance benefits. It's an excellent default choice for most projects.

**`Cargo.toml`**

```toml
[profile.release]
lto = "thin"
```


### 2. `lto = "fat"` or `lto = true` (The "Max Power" Option)

FatLTO is the traditional, more aggressive form of LTO. It performs a full analysis of the entire program, which can sometimes yield slightly better performance or a smaller binary than "thin" LTO, but at the cost of much longer compile times and higher memory usage during the build. It is not parallel and can be a bottleneck in the compilation process.

**`Cargo.toml`**

```toml
[profile.release]
lto = "fat" # or `lto = true`
```


### 3. `codegen-units = 1` (A Related and Important Setting)

By default, Rust compiles your crate in multiple "codegen units" in parallel to speed up compilation. However, this can prevent certain optimizations that can only be done when a whole crate is considered as a single unit.

For maximum performance, it's often recommended to combine LTO with `codegen-units = 1`. This tells the compiler to treat your entire crate as a single compilation unit, which can unlock even more optimization opportunities, especially when combined with LTO.

**The "Ultimate Performance" Configuration:**

```toml
[profile.release]
lto = "fat"
codegen-units = 1
```

**Warning**: This configuration will result in the **slowest compile times**. Always measure to see if the performance benefit is worth the wait.

## A Practical Example: When Does LTO Help?

Imagine you have a small utility function in one module that is called frequently in a tight loop in another module.

**`src/utils.rs`**

```rust
// We don't mark this with #[inline]
pub fn is_special_number(n: u32) -> bool {
    n == 42
}
```

**`src/main.rs`**

```rust
mod utils;

fn main() {
    let mut special_count = 0;
    // This is a "hot loop" that runs many times.
    for i in 0..100_000_000 {
        if utils::is_special_number(i) {
            special_count += 1;
        }
    }
    println!("Found {} special numbers.", special_count);
}
```

- **Without LTO**: The compiler will generate a real function call to `is_special_number` for every iteration of the loop. This function call has overhead (setting up the stack, jumping, returning).
- **With LTO Enabled**: The linker will see the entire program, notice that `is_special_number` is a very small function, and will likely **inline** it directly into the loop. The compiled code will look more like this:

```rust
// What the code becomes after LTO inlining
for i in 0..100_000_000 {
    if i == 42 { // The function call is gone!
        special_count += 1;
    }
}
```


This elimination of the function call overhead in a hot loop can lead to a significant runtime performance improvement.

## Summary: When to Use LTO

| **LTO Setting** | **When to Use** | **Pros** | **Cons** |
| :-- | :-- | :-- | :-- |
| **`lto = "off"` (Default for debug)** | During development, for the fastest possible compile times. | - Fastest compile times. | - Slower runtime performance.<br>- Larger binary size. |
| **`lto = "thin"`** | **The recommended default for release builds.** A great balance of performance and compile time. | - Good runtime performance improvement.<br>- Good binary size reduction. | - Slower compile times than "off". |
| **`lto = "fat"`** | For final release builds where you want to squeeze out every last bit of performance and are willing to wait much longer for compilation. | - Potentially the best runtime performance and smallest binary size. | - Very slow compile times.<br>- High memory usage during build. |

**The Golden Rule**: Always **benchmark your application** before and after enabling LTO to see if the trade-off is worth it for your specific use case. For many applications, the performance gain from LTO can be substantial, making it a crucial tool in the Rust performance toolbox.

1. https://gist.github.com/jFransham/369a86eff00e5f280ed25121454acec1
2. https://benw.is/posts/how-i-improved-my-rust-compile-times-by-seventy-five-percent
3. https://developers.redhat.com/blog/2020/03/18/cross-language-link-time-optimization-using-red-hat-developer-tools
4. https://nnethercote.github.io/perf-book/build-configuration.html
5. https://convolv.es/guides/lto/
6. https://stackoverflow.com/questions/77469480/enabling-link-time-optimization-via-makefile
7. https://qformic.com/en-US/posts/rust-c-lto
8. https://users.rust-lang.org/t/rust-static-library-parts-in-c-lto-cross-language-optimization/85722
