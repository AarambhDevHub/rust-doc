---
title: "Criterion Benchmarking in Rust"
description: "Learn how to measure and compare performance of Rust functions using Criterion benchmarking library."
date: 2025-11-10
weight: 1
---

# Criterion Benchmarking in Rust: A Beginner's Guide

Criterion.rs is a powerful benchmarking library for Rust that helps you measure your code's performance with statistical rigor. It allows you to:

- Detect performance regressions.
- Compare different implementations of a function.
- Generate detailed reports to visualize performance.

This guide will walk you through setting up and using Criterion, even if you're a beginner.

## Why Use Criterion?

While Rust has a built-in benchmark runner, it's unstable and only works on nightly Rust. Criterion, on the other hand, is a stable and more powerful alternative that provides:

- **Statistical analysis**: It runs your code many times to get a reliable measurement, accounting for system noise and other variations.
- **Regression detection**: It can compare the results of the current benchmark run to the previous one, telling you if your changes made the code faster or slower.
- **Detailed reports**: It can generate HTML reports with interactive graphs, making it easy to visualize and analyze performance data.[^1]


## Getting Started: A Step-by-Step Example

Let's say we want to compare two ways of calculating the sum of numbers from 1 to `n`: a simple loop versus the arithmetic formula `n * (n + 1) / 2`.

### Step 1: Add Criterion to Your Project

Criterion should be added as a **`dev-dependency`** because you only need it for testing and development, not for your final production binary.

In your `Cargo.toml` file, add the following:

```toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "my_benchmark"
harness = false
```

- `features = ["html_reports"]`: This enables the generation of graphical reports.
- `[[bench]]`: This section tells Cargo about our benchmark suite. `harness = false` is crucial; it tells Cargo to let Criterion manage the benchmarks instead of using the default Rust test harness.[^2][^1]


### Step 2: Write the Functions to Benchmark

Create a `src/lib.rs` file (if you don't have one) and add the two functions we want to compare.
**`src/lib.rs`**

```rust
/// Calculates the sum using a simple iteration.
pub fn sum_iterative(n: u64) -> u64 {
    (1..=n).sum()
}

/// Calculates the sum using the arithmetic formula.
pub fn sum_formula(n: u64) -> u64 {
    n * (n + 1) / 2
}
```


### Step 3: Create the Benchmark File

Cargo expects benchmarks to live in the `benches` directory. Create a new file named `benches/my_benchmark.rs`. The filename must match the `name` you specified in `Cargo.toml`.

**`benches/my_benchmark.rs`**

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};
// Import the functions from your library
use your_crate_name::{sum_iterative, sum_formula};

// This is the function that defines our benchmark group.
fn bench_sums(c: &mut Criterion) {
    let input_value = 1_000_000u64;

    // Create a new benchmark group to compare the two functions.
    let mut group = c.benchmark_group("Sum to a Million");

    // Add the first function to the group.
    group.bench_function("Iterative", |b| {
        // The `iter` method runs the code inside the closure many times.
        b.iter(|| sum_iterative(black_box(input_value)))
    });

    // Add the second function to the group.
    group.bench_function("Formula", |b| {
        b.iter(|| sum_formula(black_box(input_value)))
    });

    // End the group. This is where reports are finalized.
    group.finish();
}

// These two macros are the entry point for the benchmark suite.
criterion_group!(benches, bench_sums);
criterion_main!(benches);
```


### Step 4: Run the Benchmark

Now, run the following command in your terminal:

```bash
cargo bench
```


### Understanding the Code

- `criterion_group!(benches, bench_sums)`: This macro creates a benchmark group. You can list multiple benchmark functions here (e.g., `criterion_group!(benches, bench_sums, another_bench)`).
- `criterion_main!(benches)`: This macro creates the `main` function for your benchmark executable.[^3]
- `c.benchmark_group("...")`: Grouping benchmarks is a great way to compare different implementations of the same task. The results will be plotted on the same graph in the HTML report.
- `b.iter(|| ...)`: This is the core of the benchmark. Criterion's `Bencher` (`b`) will run the closure (`|| ...`) many times until it has collected enough data to be statistically confident about the performance [^4].
- `black_box(...)`: This is a crucial function. It acts as an opaque barrier to the compiler's optimizer. Without it, the compiler might be so clever that it pre-calculates the result of `sum_formula(1_000_000)` at compile time, leading to a benchmark time of zero. `black_box` ensures that the code is actually executed at runtime.[^1]


## Analyzing the Results

After running `cargo bench`, you will see output in your terminal summarizing the results. More importantly, Criterion will create a `target/criterion` directory. Inside it, you can find the HTML reports.

Navigate to `target/criterion/Sum to a Million/report/index.html` (the path will match your group name) and open it in a browser. You will see:

- **An overview** of the benchmarks in the group.
- **Detailed statistics**: Mean time, standard deviation, etc.
- **Interactive graphs**: Violin plots and probability density functions that visualize the distribution of execution times.

In our example, you will see that the `Formula` version is orders of magnitude faster than the `Iterative` version, as expected.

## Advanced Usage: Benchmarking with Different Inputs

You often want to see how an algorithm's performance changes as the input size grows. You can do this by creating a benchmark that iterates over a range of inputs.

```rust
use criterion::{BenchmarkId, Criterion};
// ... (imports and functions)

fn bench_fibonacci(c: &mut Criterion) {
    let mut group = c.benchmark_group("Fibonacci");

    // Benchmark the same function with different inputs.
    for i in [10u64, 20, 30].iter() {
        // `BenchmarkId` creates a unique name for each benchmark run.
        group.bench_with_input(BenchmarkId::from_parameter(i), i, |b, i| {
            b.iter(|| fibonacci(black_box(*i)))
        });
    }
    group.finish();
}

// A recursive fibonacci function (intentionally slow for demonstration)
fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

This will generate a report showing how the performance of the `fibonacci` function degrades as `n` increases, which is a classic sign of an exponential time complexity algorithm.

By integrating Criterion into your development workflow, you can make informed, data-driven decisions about performance, ensuring that your Rust applications are not only safe and correct but also fast and efficient.[^5][^2]

1. https://engineering.deptagency.com/benchmarking-rust-code-using-criterion-rs
2. https://www.rustfinity.com/blog/rust-benchmarking-with-criterion
3. https://www.w3resource.com/rust-tutorial/rust-criterion.php
4. https://bencher.dev/learn/benchmarking/rust/criterion/
5. https://github.com/bheisler/criterion.rs
6. https://bheisler.github.io/criterion.rs/book/
7. https://towardsdatascience.com/benchmarking-rust-compiler-settings-with-criterion-62db50cd62fb/
8. https://www.youtube.com/watch?v=Ue0a032ZARA
9. https://bheisler.github.io/criterion.rs/book/getting_started.html
10. https://nnethercote.github.io/perf-book/benchmarking.html
