---
title: "Performance Monitoring with Rust"
description: "Learn techniques and libraries for profiling, benchmarking, and monitoring system performance in Rust applications."
date: 2025-11-05
weight: 5
---

# Performance Monitoring with Rust: Profiling, Benchmarking, and Monitoring

Rust provides powerful tools and libraries for keeping your code fast and efficient. Whether you want to find "hot spots" that drain performance, compare two different algorithms, or monitor your app in production, Rust's ecosystem makes it possible—even for beginners. Let’s break down each step.

***

## 1. **Benchmarking in Rust**

**Benchmarking** is the process of measuring how long a particular operation or function takes, ideally with statistical robustness and reproducibility.

### A. Using Criterion.rs (Recommended)

**Criterion** is the most popular Rust benchmarking crate, far superior to the outdated built-in benchmarking feature. It provides advanced statistical analysis, nice reports, and easy comparison of results.[^1]

**Setup**:

1. **Add to Cargo.toml**

```toml
[dev-dependencies]
criterion = "0.5"
```

2. **Create a benchmark file** in `benches/bench.rs`:

```rust
use criterion::{criterion_group, criterion_main, Criterion};

fn my_bench_fn(c: &mut Criterion) {
    c.bench_function("sum 1 to 1000", |b| {
        b.iter(|| (1..=1000).sum::<u64>())
    });
}

criterion_group!(benches, my_bench_fn);
criterion_main!(benches);
```

3. **Run your benchmarks:**

```sh
cargo bench
```


Criterion will produce a detailed report, including **mean**, **variance**, and **regression plots**.

***

## 2. **Profiling Rust Applications**

**Profiling** helps you see *where* time is spent or *how* resources are used (CPU, memory, etc.).

### A. Native (External) Profilers

- **Linux:** [`perf`](https://nnethercote.github.io/perf-book/profiling.html), **FlameGraph**, Callgrind/Cachegrind, [heaptrack/bytehound](https://github.com/koute/bytehound)
- **MacOS:** Instruments, DTrace
- **Windows:** Intel VTune, AMD μProf

**Basic Example with Linux `perf`**:

1. **Build in release mode with debug info**:

```toml
[profile.release]
debug = true
```

```sh
cargo build --release
```

2. **Record profiling data**:

```sh
perf record -g ./target/release/my_binary
```

3. **Generate a flamegraph**:
Download [Flamegraph](https://github.com/brendangregg/FlameGraph):

```sh
perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > flamegraph.svg
```

Open the SVG to explore visually which functions take the most time.[^2][^3]

### B. Rust-Native Profiling Libraries

**[pprof-rs](https://github.com/tikv/pprof-rs)**: Easy-integrate CPU profiler for Rust; can generate flamegraphs during test runs or at runtime.[^4]

**[measureme](https://github.com/rust-lang/measureme)**: Used for profiling the Rust compiler itself.

**[pyroscope](https://pyroscope.io/docs/rust/)**: Continuous profiling \& monitoring (like Datadog/Google Cloud Profiler but open source).

**Example: Using `pprof` in Code**

```rust
use pprof::ProfilerGuard;

fn main() {
    let guard = ProfilerGuard::new(100).unwrap(); // samples per second

    my_expensive_function();

    if let Ok(report) = guard.report().build() {
        let file = std::fs::File::create("flamegraph.svg").unwrap();
        report.flamegraph(file).unwrap();
    }
}
```

After running, open `flamegraph.svg` in your browser.

***

## 3. **Monitoring in Production**

To keep track of live system metrics (CPU, memory, custom metrics) in deployed Rust apps, use:

- **[metrics](https://github.com/metrics-rs/metrics):** Expose counters and histograms in a Prometheus-friendly fashion.
- **[prometheus](https://github.com/tikv/rust-prometheus):** Native Prometheus client for Rust.
- **[opentelemetry](https://github.com/open-telemetry/opentelemetry-rust):** Industry-standard distributed tracing and metrics.

**Example: Tracking HTTP requests with Prometheus**

```rust
use prometheus::{IntCounter, register_int_counter};

static HTTP_REQUESTS: IntCounter = register_int_counter!("http_requests_total", "Total requests").unwrap();

fn handle_request() {
    HTTP_REQUESTS.inc();
}
```

You can then expose these metrics on an HTTP endpoint to let Prometheus scrape and display them in dashboards.[^5]

***

## 4. **Best Practices \& Tips**

- **Always profile and benchmark** after building in **release mode.**
- **Micro-benchmarking** is best for isolated functions; use system profilers for end-to-end and real-world performance.
- **Small changes in code** can have hidden costs—let data, not intuition, guide optimizations!
- Start with **Criterion** for easy benchmarking; use **perf/flamegraph** or **pprof** for robust profiling.

***

## Summary Table

| Goal | Tool/Library | How to Use/Where to Learn |
| :-- | :-- | :-- |
| Benchmarking | criterion | Add as dev-dep, use `cargo bench` |
| Profiling | perf, FlameGraph | Build with debug symbols, run with system profiler |
| Profiling | pprof-rs | Integrate in code, generate flamegraphs |
| Monitoring | metrics/prometheus | Expose metrics in code, scrape with Prometheus/Grafana |
| Memory/Heap | bytehound, heaptrack | Install and run on your binary |

With these patterns and tools, you can confidently monitor, measure, and optimize the performance of any Rust application, from small utilities to large production systems.[^3][^2][^1]

1. https://www.worthe-it.co.za/blog/2021-06-19-rust-performance-optimization-tools.html
2. https://nnethercote.github.io/perf-book/profiling.html
3. https://codedamn.com/news/rust/benchmarking-profiling-rust-applications-optimal-performance
4. https://github.com/tikv/pprof-rs
5. https://lib.rs/development-tools/profiling
6. https://users.rust-lang.org/t/profiling-a-library-project/99778
7. https://www.intel.com/content/www/us/en/developer/articles/guide/enhancing-rust-performance-profiling-with-vtune.html
8. https://www.infoq.com/articles/benchmark-profile-ebpf-code/
9. https://kobzol.github.io/rust/rustc/2023/08/18/rustc-benchmark-suite.html
10. https://github.com/rust-unofficial/awesome-rust
