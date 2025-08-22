---
title: "Axum Framework Basics"
description: "Learn the fundamentals of Axum, a lightweight and type-safe web framework for Rust."
date: 2025-11-03
weight: 1
---

# Axum Framework Basics: A Beginner's Guide

Learning Axum is like being handed the keys to a **brand-new, professional-grade restaurant kitchen**. It's modern, clean, and incredibly well-organized. Everything is designed to be ergonomic and modular. You get a high-performance stove (`hyper`), an efficient kitchen management system (`tokio`), and a set of standardized tools and stations (`tower`). Axum is the framework that brings all these powerful components together, giving you a simple, intuitive layout to build high-performance web applications in Rust.

As of late 2025, the latest version of Axum is **v0.8.x**. This guide will use syntax compatible with this version.[^1][^2]

## What is Axum?

Axum is a web application framework for Rust that focuses on:

* **Ergonomics**: It's designed to be easy and pleasant to use.
* **Modularity**: It's built on top of best-in-class libraries like `tokio` for its async runtime, `hyper` for its HTTP server, and `tower` for middleware services. You can easily use components from these ecosystems.[^3]
* **Type Safety**: It leverages Rust's powerful type system to prevent common web development bugs at compile time.

Unlike some other frameworks, Axum uses a "macro-free" API for its core routing, which can make the code feel more like standard Rust.[^4]

## Setting Up Your First Axum Project

1. **Create a new Rust project:**

```bash
cargo new axum_basics
cd axum_basics
```

2. **Add the necessary dependencies to `Cargo.toml`:**

```toml
[dependencies]
axum = "0.8.4"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

    * `axum`: The framework itself.
    * `tokio`: The asynchronous runtime needed to run the server.
    * `serde` and `serde_json`: For handling JSON data, which is essential for almost any web API.

## "Hello, World!": Your First Axum Server

Let's create the simplest possible web server.

**`src/main.rs`**

```rust
use axum::{
    routing::get,
    Router,
};
use std::net::SocketAddr;

#[tokio::main]
async fn main() {
    // 1. Create the Router
    // A router is where you define all the "routes" or paths your application will handle.
    let app = Router::new()
        // We define a route for the root path ("/") that handles GET requests.
        // When a request hits this route, it calls the `root_handler`.
        .route("/", get(root_handler));

    // 2. Define the server address
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await.unwrap();
    println!("Listening on http://127.0.0.1:3000");

    // 3. Start the Server
    // `axum::serve` takes the router and an address and starts the web server.
    axum::serve(listener, app)
        .await
        .unwrap();
}

// This is a "handler" function.
// Handlers are async functions that take some arguments and return something
// that can be converted into an HTTP response.
async fn root_handler() -> &'static str {
    "Hello, World!"
}
```

Run this with `cargo run`. Now, if you open `http://127.0.0.1:3000` in your browser, you will see "Hello, World!".

## Core Concepts Explained

### 1. Routing

The `Router` is the heart of your application. It maps URL paths and HTTP methods to your handler functions.[^4]

```rust
// You can chain .route() calls to define your entire application.
let app = Router::new()
    .route("/", get(root_handler))
    .route("/users", get(list_users).post(create_user)) // Handle multiple methods on one path
    .route("/users/:id", get(get_user_by_id));
```

* `get(...)`, `post(...)`, etc., are shorthand for defining which HTTP method a handler responds to.
* `/users/:id` defines a path with a **dynamic parameter**. Axum can extract this `id` for you.


### 2. Handlers

A handler is just an `async` function that contains your application logic. Its power comes from what it can accept as arguments and what it can return.[^3]

### 3. Extractors: The Magic of Axum

Extractors are how you get data from an incoming request. They are function arguments that implement a special `FromRequest` trait. Axum sees them and automatically "extracts" the data for you.[^4]

#### a. `Path`: Extracting from the URL Path

To get the `:id` from `/users/:id`, you use the `Path` extractor.

```rust
use axum::extract::Path;

// This handler will be called for a request to `/users/123`,
// and `user_id` will automatically be `123`.
async fn get_user_by_id(Path(user_id): Path<u32>) -> String {
    format!("Fetching user with ID: {}", user_id)
}
```


#### b. `Query`: Extracting from Query Parameters

To get data from a URL like `/search?q=rust&lang=en`, you use the `Query` extractor.

```rust
use axum::extract::Query;
use serde::Deserialize;

// Define a struct for your query parameters.
// The field names must match the query keys.
#[derive(Deserialize)]
struct SearchParams {
    q: String,
    lang: Option<String>, // Use Option for optional parameters
}

// For a request to `/search?q=axum`, `params.q` will be "axum".
async fn search_handler(Query(params): Query<SearchParams>) -> String {
    format!("Searching for '{}' in language '{}'", params.q, params.lang.unwrap_or_else(|| "any".to_string()))
}
```


#### c. `Json`: Deserializing a JSON Body

For `POST` or `PUT` requests, you often need to read a JSON body. The `Json` extractor does this automatically.

```rust
use axum::Json;
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct CreateUserPayload {
    username: String,
    email: String,
}

// Axum will read the request body, deserialize it from JSON into
// our `CreateUserPayload` struct, and pass it to the handler.
// If deserialization fails, it automatically returns a 400 Bad Request error.
async fn create_user_handler(Json(payload): Json<CreateUserPayload>) {
    println!("Creating user: {}", payload.username);
    // ... logic to save the user ...
}
```


### 4. Responses: Returning Data to the Client

Just as extractors simplify getting data *in*, `IntoResponse` simplifies getting data *out*. Many common types can be returned directly from a handler.

#### a. Simple Text

`&'static str`, `String`

```rust
async fn greet() -> String {
    "Hello, Axum user!".to_string()
}
```


#### b. JSON Responses

Wrap your serializable struct in `Json`. Axum will automatically serialize it and set the `Content-Type` header to `application/json`.

```rust
#[derive(Serialize)]
struct User {
    id: u32,
    username: String,
}

async fn get_user() -> Json<User> {
    let user = User {
        id: 1,
        username: "axum-fan".to_string(),
    };
    Json(user)
}
```


#### c. Custom Responses

For full control over the status code and headers, you can return an `impl IntoResponse`.

```rust
use axum::http::StatusCode;
use axum::response::{IntoResponse, Response};

async fn create_something() -> impl IntoResponse {
    (StatusCode::CREATED, "Resource created successfully")
}

// Or build a response manually
async fn custom_response() -> Response {
    Response::builder()
        .status(StatusCode::OK)
        .header("X-Custom-Header", "my-value")
        .body("This is a custom response".into())
        .unwrap()
}
```


## Putting It All Together: A Mini API

This example combines everything we've learned into a simple, in-memory API.

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::IntoResponse,
    routing::{get, post},
    Json, Router,
};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::net::SocketAddr;
use std::sync::{Arc, Mutex};

// Our application's shared state.
// We use Arc<Mutex<...>> to share it safely across threads.
type Db = Arc<Mutex<HashMap<u32, User>>>;

#[derive(Debug, Serialize, Clone)]
struct User {
    id: u32,
    username: String,
}

#[derive(Debug, Deserialize)]
struct CreateUser {
    username: String,
}

#[tokio::main]
async fn main() {
    // Initialize our in-memory database.
    let db = Db::default();

    let app = Router::new()
        .route("/users", get(list_users).post(create_user))
        .route("/users/:id", get(get_user))
        // The .with_state() call makes our `Db` available to all handlers.
        .with_state(db);

    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    println!("Server listening on http://{}", addr);
    axum::serve(tokio::net::TcpListener::bind(addr).await.unwrap(), app)
        .await
        .unwrap();
}

// --- Handlers ---

// `State` is an extractor that gives us access to our shared application state.
async fn list_users(State(db): State<Db>) -> Json<Vec<User>> {
    let users = db.lock().unwrap().values().cloned().collect();
    Json(users)
}

async fn create_user(
    State(db): State<Db>,
    Json(input): Json<CreateUser>,
) -> impl IntoResponse {
    let mut db = db.lock().unwrap();
    let id = db.keys().max().unwrap_or(&0) + 1;
    let user = User {
        id,
        username: input.username,
    };
    db.insert(id, user.clone());
    (StatusCode::CREATED, Json(user))
}

async fn get_user(
    State(db): State<Db>,
    Path(id): Path<u32>,
) -> Result<Json<User>, StatusCode> {
    let db = db.lock().unwrap();
    if let Some(user) = db.get(&id).cloned() {
        Ok(Json(user))
    } else {
        Err(StatusCode::NOT_FOUND)
    }
}
```

This example shows how Axum's simple but powerful building blocks—the `Router`, `handlers`, `extractors`, and `responses`—come together to create a clean, type-safe, and highly performant web API.

1. https://github.com/tokio-rs/axum/releases
2. https://tokio.rs/blog/2025-01-01-announcing-axum-0-8-0
3. https://www.twilio.com/en-us/blog/developers/community/build-high-performance-rest-apis-rust-axum
4. https://docs.rs/axum/latest/axum/
5. https://github.com/tokio-rs/axum
6. https://masteringbackend.com/posts/axum-framework
7. https://www.reddit.com/r/rust/comments/1hr0m5h/announcing_axum_080/
8. https://www.shuttle.dev/blog/2023/12/06/using-axum-rust
9. https://crates.io/crates/axum-server
10. https://www.youtube.com/watch?v=TTDdHZAyQN8
