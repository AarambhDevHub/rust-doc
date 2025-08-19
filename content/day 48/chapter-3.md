---
title: "Async I/O - HTTP Requests with Reqwest"
description: "Use the Reqwest crate to make asynchronous HTTP requests and handle responses efficiently."
date: 2025-10-01
weight: 3
---

# Async I/O with Reqwest: A Beginner's Guide to HTTP Requests in Rust

Understanding asynchronous HTTP requests in Rust is like learning to **manage a team of waiters in a busy restaurant who are taking phone orders**. You have two choices: a blocking approach or an async approach.

- **The Blocking (Synchronous) Waiter**: This waiter calls a customer for their order and then **stands by the phone, doing nothing else**, until the customer has finished ordering. If the customer is slow, this waiter is completely stuck, and no other work gets done. This is inefficient.
- **The Async (Non-Blocking) Waiter**: This waiter calls a customer and, while waiting for them to decide, **puts the call on hold and moves on to another task**, like taking an order from another customer or clearing a table. When the first customer is ready, the phone system "notifies" the waiter, who can then switch back to complete the order. This waiter can handle dozens of orders at once without getting stuck.

The `reqwest` crate, combined with an async runtime like `tokio`, allows you to write these efficient, "non-blocking" waiters in Rust, making it perfect for applications that need to handle many network requests concurrently.[^1]

## What is `reqwest`?

`reqwest` is a high-level, user-friendly HTTP client for Rust. It is built on top of lower-level libraries like `hyper` and provides a convenient API for making all kinds of HTTP requests. It's the most popular choice for this task in the Rust ecosystem because it handles many complexities for you, such as HTTPS, cookies, and connection pooling.[^2][^3]

## Project Setup

To get started, you'll need to add `reqwest` and `tokio` to your `Cargo.toml`. `tokio` is the async runtime that will execute our asynchronous code.

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] } # "json" feature is for easy JSON handling
tokio = { version = "1", features = ["full"] }      # "full" feature enables the async runtime and macros
serde = { version = "1.0", features = ["derive"] } # Optional: for deserializing JSON into structs
serde_json = "1.0"                                  # Optional: for working with JSON values
```


## Making Your First Async GET Request

Let's make a simple GET request to `https://httpbin.org/get`, a service for testing HTTP requests.

**File: `src/main.rs`**

```rust
use reqwest::Error;

// The `#[tokio::main]` macro transforms our async main function
// into a regular one by setting up the Tokio runtime.
#[tokio::main]
async fn main() -> Result<(), Error> {
    // 1. Make the GET request.
    //    `.await` pauses the function until the request is sent and a response
    //    is received, without blocking the thread.
    let response = reqwest::get("https://httpbin.org/get").await?;

    // 2. Check the status of the response.
    println!("Status: {}", response.status());

    // 3. Get the response body as text.
    //    `.text()` also returns a future, so we must `.await` it.
    let body = response.text().await?;
    println!("Body:\n{}", body);

    Ok(())
}
```


### How It Works: A Step-by-Step Breakdown

1. **`#[tokio::main]`**: This macro is our entry point. It sets up and starts the `tokio` async runtime. The runtime is responsible for "juggling" all the async tasks, running them on a small pool of OS threads.
2. **`async fn main()`**: We declare `main` as `async`, which allows us to use the `.await` keyword inside it.
3. **`reqwest::get(...)`**: This is a simple helper function in `reqwest` for making a GET request. It returns a `Future`, which is a value that represents a computation that hasn't finished yet.
4. **`.await`**: This is the magic of async Rust. When the code hits `.await`, it tells the `tokio` runtime, "I'm waiting for this network request to complete. While I'm waiting, you can go run some other tasks." This frees up the thread to do other work, making the program non-blocking. Once the network response arrives, the runtime will "wake up" this function and continue from where it left off.[^4]
5. **Error Handling (`?`)**: The `?` operator works seamlessly in `async` functions that return a `Result`, automatically propagating any errors that occur during the request.

## Working with JSON Responses

A very common task is to make a request to an API and parse the JSON response. `reqwest` makes this incredibly easy with its `json()` method.

Let's fetch information about the `reqwest` crate itself from the crates.io API.

```rust
use reqwest::Error;
use serde::Deserialize;

// Define a struct that matches the structure of the expected JSON response.
// We only care about a few fields, so we'll only define those.
#[derive(Deserialize, Debug)]
struct CrateInfo {
    #[serde(rename = "crate")]
    krate: CrateDetails,
}

#[derive(Deserialize, Debug)]
struct CrateDetails {
    id: String,
    name: String,
    description: String,
    max_version: String,
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    let url = "https://crates.io/api/v1/crates/reqwest";

    // Make the request and use `.json()` to automatically deserialize the response.
    // The type `CrateInfo` is inferred from the variable type annotation.
    let crate_info: CrateInfo = reqwest::get(url)
        .await?
        .json()
        .await?;

    println!("--- Crate Information for 'reqwest' ---");
    println!("ID: {}", crate_info.krate.id);
    println!("Name: {}", crate_info.krate.name);
    println!("Latest Version: {}", crate_info.krate.max_version);
    println!("Description: {}", crate_info.krate.description);

    Ok(())
}
```


## Making a POST Request

Sending data, such as with a POST request, is also straightforward. You can build a request using the `reqwest::Client`. It's a best practice to create a `Client` and reuse it, as it manages a connection pool internally, making subsequent requests faster.

```rust
use reqwest::{Client, Error};
use serde::Serialize;
use std::collections::HashMap;

// Define a struct for our request body. `Serialize` will convert it to JSON.
#[derive(Serialize)]
struct NewDish {
    name: String,
    ingredients: Vec<String>,
    price: f64,
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    // 1. Create a client. It's good practice to reuse this.
    let client = Client::new();

    // 2. Create the data we want to send.
    let dish = NewDish {
        name: "Async Paella".to_string(),
        ingredients: vec!["rice".to_string(), "saffron".to_string(), "seafood".to_string()],
        price: 25.50,
    };

    // 3. Make the POST request.
    let response = client
        .post("https://httpbin.org/post")
        .json(&dish) // Automatically serializes `dish` to JSON and sets the Content-Type header
        .send()
        .await?;

    // 4. Handle the response.
    if response.status().is_success() {
        // httpbin.org/post echoes the request body back to us in the 'json' field.
        let response_json: serde_json::Value = response.json().await?;
        println!("Server received our dish data:\n{:#?}", response_json["json"]);
    } else {
        println!("POST request failed with status: {}", response.status());
    }

    Ok(())
}
```


## Running Multiple Requests Concurrently

The true power of `async` is unlocked when you run many I/O-bound tasks at the same time. `tokio::join!` is a simple way to run a few futures concurrently, and `futures::future::join_all` is great for a dynamic number of futures.[^5]

```rust
use reqwest::Error;

async fn fetch_url(url: &str) -> Result<String, Error> {
    let body = reqwest::get(url).await?.text().await?;
    Ok(format!("URL: {}, Length: {}", url, body.len()))
}

#[tokio::main]
async fn main() {
    let urls = [
        "https://www.rust-lang.org",
        "https://tokio.rs",
        "https://docs.rs/reqwest",
    ];

    // --- Using tokio::join! for a fixed number of futures ---
    println!("--- Running with tokio::join! ---");
    let (res1, res2, res3) = tokio::join!(
        fetch_url(urls),
        fetch_url(urls[^9]),
        fetch_url(urls[^10])
    );
    println!("{}\n{}\n{}", res1.unwrap(), res2.unwrap(), res3.unwrap());


    // --- Using join_all for a dynamic number of futures ---
    println!("\n--- Running with join_all ---");
    let futures = urls.iter().map(|&url| fetch_url(url));
    let results = futures::future::join_all(futures).await;

    for res in results {
        match res {
            Ok(summary) => println!("{}", summary),
            Err(e) => eprintln!("Error fetching URL: {}", e),
        }
    }
}
```

*Note: You'll need `futures = "0.3"` in your `Cargo.toml` for `join_all`.*

## Best Practices for `reqwest`

- **Reuse the `Client`**: Creating a `Client` is not free. Create it once and share it (e.g., using `Arc`) for all your requests to take advantage of connection pooling.
- **Set Timeouts**: Network requests can hang. Always configure timeouts on your client to prevent your application from waiting forever.

```rust
let client = reqwest::Client::builder()
    .timeout(std::time::Duration::from_secs(10))
    .build()?;
```

- **Handle Errors Gracefully**: Network I/O is unreliable. Always handle the `Result` returned by `reqwest` functions. Don't just `.unwrap()`.
- **Check Status Codes**: A successful request doesn't mean the operation succeeded. Always check the HTTP status code (e.g., `200 OK` vs. `404 Not Found`).

By using `reqwest` with `tokio`, you can write highly efficient, concurrent network applications in Rust that can handle thousands of I/O-bound tasks with ease, just like an expert chef managing a busy kitchen.
<span style="display:none">[^6][^7][^8]</span>

1. https://blog.logrocket.com/making-http-requests-rust-reqwest/
2. https://docs.rs/reqwest/
3. https://www.petergirnus.com/blog/how-to-make-http-requests-in-rust
4. https://rust-lang-nursery.github.io/rust-cookbook/web/clients/requests.html
5. https://stackoverflow.com/questions/51044467/how-can-i-perform-parallel-asynchronous-http-get-requests-with-reqwest
6. https://www.youtube.com/watch?v=E2qnMh3W7TM
7. https://qxf2.com/blog/http-requests-using-rust-reqwest/
8. https://github.com/seanmonstar/reqwest
9. https://doc.rust-lang.org/book/ch10-01-syntax.html
10. https://rustc-dev-guide.rust-lang.org/backend/monomorph.html
