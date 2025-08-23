---
title: "Monitoring and Logging"
description: "Implement monitoring and logging in Rust applications to track performance and debug issues."
date: 2025-11-11
weight: 2
---

# Monitoring and Logging in Rust: A Beginner's Guide

Effective monitoring and logging are essential for building robust, reliable, and debuggable Rust applications. They are your eyes and ears, telling you what your application is doing in real-time, helping you track performance, and allowing you to diagnose issues when they arise. This guide will walk you through the standard tools and best practices for implementing both in your Rust projects.

## The "Flight Recorder" Analogy: Understanding Your Application ✈️

- **Logging**: This is your application's **cockpit voice recorder and flight data recorder**. It captures a detailed, chronological stream of events, warnings, errors, and other messages. When something goes wrong, you can "play back the tape" to understand what happened step-by-step.[^1]
- **Monitoring**: This is the **live instrument panel** in the cockpit. It gives you a real-time, high-level view of critical metrics: altitude (memory usage), speed (requests per second), engine temperature (CPU load), etc. It helps you spot problems as they are happening.

A good system needs both: the live dashboard for immediate awareness and the detailed logs for deep analysis.

## Part 1: Logging in Rust

The Rust ecosystem has a powerful and flexible logging system built around a "facade" pattern. This means that libraries and applications don't need to choose a specific logging implementation; they just use a standard API, and the final application decides how to handle the logs.[^1]

- **The `log` Crate**: This is the facade. It provides the standard logging macros (`info!`, `warn!`, `error!`, `debug!`, `trace!`) that everyone uses.
- **A Logging Backend**: This is the implementation that actually writes the logs to the console, a file, or a remote service. Popular choices include `env_logger`, `fern`, and `tracing-subscriber`.


### A Simple and Practical Setup: `log` + `env_logger`

This is the most common and straightforward way to get started with logging.

#### Step 1: Add Dependencies to `Cargo.toml`

```toml
[dependencies]
log = "0.4"
env_logger = "0.10"
```


#### Step 2: Initialize the Logger and Add Log Messages

In your `main.rs`, you initialize `env_logger` once at the very beginning. Then, you can use the macros from the `log` crate anywhere in your application.

```rust
use log::{info, warn, error, debug};

fn main() {
    // Initialize the logger.
    env_logger::init();

    info!("Application is starting up...");

    let result = perform_task(10, 5);
    info!("Task finished with result: {}", result);

    let error_result = perform_task(10, 0);
    if error_result == -1 {
        error!("Task failed with an division-by-zero error!");
    }

    debug!("This is a detailed debug message that won't show up by default.");

    info!("Application shutting down.");
}

fn perform_task(a: i32, b: i32) -> i32 {
    if b == 0 {
        warn!("Division by zero attempted. This may lead to an error.");
        return -1;
    }
    a / b
}
```


#### Step 3: Control Log Verbosity with an Environment Variable

`env_logger` allows you to control which log levels are visible using the `RUST_LOG` environment variable. This is incredibly useful for switching between concise production logs and verbose development logs without changing your code.[^2]

- **Running with default (error) level**: `cargo run`
- **Running with `info` level and above**: `RUST_LOG=info cargo run`

```
[2025-08-23T14:30:00Z INFO  my_app] Application is starting up...
[2025-08-23T14:30:00Z INFO  my_app] Task finished with result: 2
[2025-08-23T14:30:00Z WARN  my_app::main] Division by zero attempted. This may lead to an error.
[2025-08-23T14:30:00Z ERROR my_app] Task failed with an division-by-zero error!
[2025-08-23T14:30:00Z INFO  my_app] Application shutting down.
```

- **Running with `debug` level (shows everything)**: `RUST_LOG=debug cargo run`


### Advanced and Structured Logging with `tracing`

For more complex applications, especially `async` ones, the `tracing` crate is the modern standard. It provides **structured logging** (attaching key-value data to logs) and is designed for high-performance, asynchronous environments.[^3]

- **Why `tracing`?**: It integrates seamlessly with `async/await`, can trace the flow of a single request across multiple functions and threads, and produces structured logs (like JSON) that are easy for machines to parse. This is what you would use for production-grade, distributed systems.[^4][^5]


## Part 2: Monitoring in Rust

Monitoring is about collecting time-series data (metrics) about your application's performance and health. The standard approach is to expose these metrics on an HTTP endpoint, which a monitoring system like **Prometheus** can then scrape.

### A Simple Setup: The `prometheus` Crate

Let's create an application that exposes a simple counter for how many HTTP requests it has received.

#### Step 1: Add Dependencies to `Cargo.toml`

```toml
[dependencies]
prometheus = { version = "0.13", features = ["process"] }
lazy_static = "1.4.0"
```


#### Step 2: Create and Expose a Metric

We'll create a static, global counter that can be incremented from anywhere in our app.

```rust
use prometheus::{Encoder, IntCounter, TextEncoder, opts};
use lazy_static::lazy_static;
use std::net::TcpListener;
use std::io::Write;

// Use lazy_static to create a global, static metric.
lazy_static! {
    pub static ref HTTP_REQUESTS_TOTAL: IntCounter =
        IntCounter::with_opts(opts!("http_requests_total", "Total number of HTTP requests received")).unwrap();
}

// A simple function to simulate handling a web request.
fn handle_request() {
    println!("Handling a request...");
    // Increment the counter every time this function is called.
    HTTP_REQUESTS_TOTAL.inc();
}

// A simple server to expose the metrics on port 9898.
fn start_metrics_server() {
    let listener = TcpListener::bind("127.0.0.1:9898").unwrap();
    println!("Metrics server listening on http://127.0.0.1:9898/metrics");
    for stream in listener.incoming() {
        let mut stream = stream.unwrap();

        let encoder = TextEncoder::new();
        let metric_families = prometheus::gather();
        let mut buffer = Vec::new();
        encoder.encode(&metric_families, &mut buffer).unwrap();

        let response = format!(
            "HTTP/1.1 200 OK\r\nContent-Type: {}\r\nContent-Length: {}\r\n\r\n{}",
            encoder.format_type(),
            buffer.len(),
            String::from_utf8(buffer).unwrap()
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}

fn main() {
    // Register the metric with the default registry.
    prometheus::register(Box::new(HTTP_REQUESTS_TOTAL.clone())).unwrap();

    // Start the metrics server in a separate thread.
    std::thread::spawn(start_metrics_server);

    // Simulate receiving some requests in our main application.
    for _ in 0..5 {
        handle_request();
        std::thread::sleep(std::time::Duration::from_secs(1));
    }
}
```

**How to check it**:

1. Run the application: `cargo run`.
2. While it's running, open your web browser or use `curl` to access `http://127.0.0.1:9898/metrics`.
3. You will see the Prometheus-formatted metrics, including our counter:

```
# HELP http_requests_total Total number of HTTP requests received
# TYPE http_requests_total counter
http_requests_total 5
```


This endpoint can now be added to a Prometheus server, which will scrape it periodically and allow you to build graphs and alerts in a tool like **Grafana**.

## Best Practices for Logging and Monitoring

- **Log What Matters**: Log key business events, errors, and significant state changes. Don't log so much that it becomes noise.
- **Use Structured Logging in Production**: For services, logging in a machine-readable format like JSON is essential for aggregation and analysis tools (like Datadog, Splunk, or an ELK stack).
- **Don't Log Sensitive Data**: Be extremely careful not to log passwords, API keys, or personally identifiable information (PII).
- **Combine Logs and Metrics**: Your metrics can tell you *that* there's a spike in errors. Your logs can tell you *why*. They work together to provide full observability.
- **Use a Tiered Configuration**: Use environment variables to set different log levels for different environments (e.g., `debug` in development, `info` in production).

By implementing these patterns, you can build Rust applications that are not only fast and safe, but also transparent, manageable, and easy to debug in any environment.

1. https://notes.kodekloud.com/docs/Rust-Programming/Debugging-in-Rust/Using-println-and-logging
2. https://notes.kodekloud.com/docs/Rust-Programming/Collections-Error-Handling/Logging-Libraries
3. https://docs.aws.amazon.com/lambda/latest/dg/rust-logging.html
4. https://blog.nashtechglobal.com/from-println-hell-to-production-ready-logging-a-rust-developers-journey/
5. https://users.rust-lang.org/t/logging-framework-for-a-distributed-system/115034
6. https://rustlogs.com/getting-started/index.html
7. https://www.datadoghq.com/blog/monitor-rust-otel/
8. https://www.youtube.com/watch?v=2Eg_Keqt1I0
9. https://www.mongodb.com/docs/drivers/rust/v3.0/fundamentals/tracing-logging/
10. https://www.linkedin.com/learning/rust-llmops/monitoring-and-logging
11. https://www.reddit.com/r/rust/comments/unbyue/tutorial_on_monitoring_your_rust_application_with/
12. https://www.mongodb.com/docs/drivers/rust/v2.8/fundamentals/tracing-logging/
