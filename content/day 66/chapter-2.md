---
title: "Profiling Rust Programs with perf and valgrind"
description: "Understand how to profile Rust programs with tools like perf and valgrind for CPU and memory insights."
date: 2025-11-10
weight: 2
---

# Profiling Rust Programs: A Beginner's Guide with `perf` and `valgrind`

Understanding *why* your Rust program is slow or uses too much memory is the first step to making it faster and more efficient. Profiling is the art of gathering this data. This guide will walk beginners through using two powerful, classic Linux tools—`perf` for CPU profiling and `valgrind` for memory profiling—to gain deep insights into their Rust applications.

## The "Car Mechanic" Analogy: Looking Under the Hood 🚗

- **Your Rust Program**: A high-performance car. It runs, but you want it to be faster and more fuel-efficient.
- **Benchmarking**: Taking the car to a racetrack to get its lap time. It tells you *how fast* it is but not *why*.
- **Profiling**: Putting the car on a diagnostic machine. It tells you exactly which parts of the engine are using the most fuel (CPU) or which components are too heavy (memory).

Profiling lets you stop guessing and start making data-driven optimizations.

***

## Part 1: Preparing Your Rust Program for Profiling

Before you can get useful data, you need to compile your program correctly. You want the speed of a `release` build but the debugging information of a `dev` build.

**Step 1: Configure `Cargo.toml`**

Add the following lines to your `Cargo.toml` file. This tells the Rust compiler to include debug symbols (like function names) in your optimized release binary, which is essential for profilers to make sense of the data.[^1]

```toml
[profile.release]
debug = true
```

**Step 2: Build Your Program**

Now, build your application in release mode as you normally would:

```bash
cargo build --release
```

Your final executable will be in `target/release/your_binary_name`.

***

## Part 2: CPU Profiling with `perf` and FlameGraphs

`perf` is a powerful Linux tool that samples your program as it runs to see which functions are "hot" (i.e., using the most CPU time). A **flamegraph** is a popular way to visualize this data.[^2]

### Step-by-Step Guide

**1. Install `perf` and FlameGraph scripts:**

`perf` is usually available in a package like `linux-tools-common`. FlameGraph is a set of Perl scripts you need to download from GitHub.

```bash
# On Debian/Ubuntu
sudo apt-get install -y linux-tools-common linux-tools-generic linux-tools-`uname -r`

# Download FlameGraph
git clone https://github.com/brendangregg/FlameGraph.git
cd FlameGraph
```

**2. Record Profiling Data:**

Run your program under `perf record`. The `-g` flag is crucial as it records the **call graph**, showing not just which functions are slow, but what call chains led to them.[^1]

```bash
# From your project's root directory
perf record -g ./target/release/your_binary_name
```

This will run your program and create a `perf.data` file in the current directory.

**3. Generate the FlameGraph:**

Now, use the `perf` and FlameGraph scripts to process the `perf.data` file into an interactive SVG image.

```bash
# Make sure you are in the directory with the FlameGraph scripts
perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > ../my_flamegraph.svg
```

**4. Analyze the FlameGraph:**

Open `my_flamegraph.svg` in your web browser.

- **What you're seeing**: A visual representation of all the function calls in your program.
- **The Y-axis** represents the call stack (what calls what). `main` is at the bottom.
- **The X-axis** represents the percentage of CPU time spent. **Wider bars mean more time spent in that function and all the functions it calls.**
- **How to use it**: Look for the widest plateaus at the top of the graph. These are the functions that are themselves taking up the most CPU time. This is where you should focus your optimization efforts.

***

## Part 3: Memory Profiling with `valgrind`

While Rust prevents most memory *safety* errors, it doesn't prevent memory *inefficiency* (like allocating too much or holding onto memory for too long). `valgrind` is a suite of tools that can help diagnose these issues. We'll focus on its `massif` tool.[^3]

### Step-by-Step Guide

**1. Install `valgrind`:**

```bash
# On Debian/Ubuntu
sudo apt-get install -y valgrind
```

**2. Profile Heap Memory with `massif`:**

Run your program under `valgrind --tool=massif`. This will record every heap allocation (`Box`, `Vec`, `String`, etc.) your program makes.

```bash
valgrind --tool=massif ./target/release/your_binary_name
```

This will run your program (much more slowly) and create a `massif.out.<PID>` file.

**3. Analyze the `massif` output:**

You can analyze the text output with `ms_print`, but it's much better to visualize it.

```bash
# Print a graph of memory usage over time
ms_print massif.out.<PID>
```

For a graphical view, install `massif-visualizer`:

```bash
sudo apt-get install -y massif-visualizer
massif-visualizer massif.out.<PID>
```

**What you're looking for**:

- **High Water Mark**: The peak memory usage of your program. Is it higher than you expect?
- **Memory Leaks**: In a long-running program, is the memory usage constantly climbing without ever going down? `massif` can help you spot this trend.
- **Allocation Hotspots**: The visualizer can show you which function calls are responsible for the largest allocations. This can help you find places where you might be allocating memory unnecessarily in a loop.


### Alternative Tool: `heaptrack`

For a more modern and often easier-to-use memory profiling experience, consider `heaptrack`. It provides a GUI that combines the features of `massif` with flamegraphs for allocations, making it very powerful.[^4]

```bash
# Run with heaptrack
heaptrack ./target/release/your_binary_name

# Analyze the results
heaptrack_gui heaptrack.your_binary_name.<PID>.zst
```


## Best Practices and Final Advice

1. **Profile What You Measure**: Don't guess where your program is slow. The results are often surprising.
2. **Profile Realistic Workloads**: Profile your application using data and usage patterns that are representative of how it will be used in the real world.
3. **Focus on the Biggest Wins**: Use the 80/20 rule. Focus on the widest bars in your flamegraph or the biggest spikes in your memory graph. Small optimizations in cold code paths are not worth the effort.
4. **Iterate**: Make one change, then re-profile to confirm it had the intended effect. Sometimes, an "optimization" can make things worse.
5. **Consider Rust-Native Tools**: For more integrated solutions, explore crates like `criterion` for benchmarking and `pprof-rs` for programmatic profiling, which can be easier to set up and run in CI environments.[^4]

By systematically using tools like `perf` and `valgrind`, you can move from "writing code that works" to "writing code that works efficiently," a crucial step in becoming a proficient systems programmer in Rust.

1. https://rustc-dev-guide.rust-lang.org/profiling/with_perf.html
2. https://gist.github.com/KodrAus/97c92c07a90b1fdd6853654357fd557a
3. https://valgrind.org/docs/manual/quick-start.html
4. https://www.youtube.com/watch?v=X6Xz4CRd6kw
5. https://llogiq.github.io/2015/07/15/profiling.html
6. https://users.rust-lang.org/t/issues-in-profiling-a-rust-application/70306
7. https://hackmd.io/@Yl0VNGYRR6aeDrHHQNhuaA/ryL5xm2x0
8. https://www.justanotherdot.com/posts/profiling-with-perf-and-dhat-on-rust-code-in-linux.html
9. https://rust-lang.github.io/packed_simd/perf-guide/prof/linux.html
