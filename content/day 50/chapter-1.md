---
title: "Async Project - HTTP Server, Database, WebSocket & Performance"
description: "Build a complete async project in Rust covering HTTP server implementation, async database operations, WebSocket handling, and performance optimization."
date: 2025-10-03
weight: 1
---

# Async Project: Building a Real-Time Visitor Counter

This project provides a comprehensive, step-by-step guide to building a complete asynchronous application in Rust from the ground up, without relying on high-level web frameworks. We will build a real-time web application that tracks and displays the number of active visitors.

**This guide will cover:**

1. **Async HTTP Server Implementation**: Building a simple HTTP server to serve a web page.
2. **Async Database Operations**: Connecting to a database to store and retrieve data asynchronously.
3. **WebSocket Handling**: Using WebSockets for real-time communication with clients.
4. **Performance \& Concurrency**: Managing shared state and understanding why `async` is a powerful choice.

## The Professional Restaurant Operations Analogy 🍽️

Imagine you are managing a high-tech restaurant. This project is like building the restaurant's entire operational system:

- **The HTTP Server**: The **front door and menu**. It greets customers and gives them a place to sit (serves the HTML page).
- **The Database**: The **restaurant's official guestbook**. It keeps a persistent record of every visitor.
- **WebSockets**: The **live announcement system**. When a new customer walks in, an announcement is broadcast to a screen at every table, showing the new total number of guests.
- **Async Runtime (`tokio`)**: The **head chef and kitchen manager**. This highly efficient manager can juggle hundreds of tasks at once—taking new orders, checking on cooking dishes, and sending them out—without getting overwhelmed. They manage the entire flow of the kitchen.


## Project Overview: The Live Visitor Counter

**What we will build**:

1. An HTTP server that serves a simple HTML page.
2. When the page loads, it will connect to our server via a WebSocket.
3. Upon connection, the server will increment a visitor count in a database.
4. The server will then **broadcast** the new total visitor count to **all** connected clients via their WebSocket connections.

This demonstrates a full-stack, real-time application using core `async` concepts.

## Project Setup

First, let's set up our `Cargo.toml`. We are intentionally avoiding web frameworks like Axum or Actix, but we will use foundational libraries that handle the low-level protocols for us.

**`Cargo.toml`**

```toml
[package]
name = "async_project"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1", features = ["full"] }
tokio-tungstenite = "0.21"
futures-util = "0.3"
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres"] }
dotenv = "0.15"
```

- `tokio`: The asynchronous runtime that will drive our entire application.
- `tokio-tungstenite`: A library for handling the WebSocket protocol on top of `tokio`.
- `futures-util`: Provides useful utilities for working with async streams.
- `sqlx`: A modern, async-safe SQL toolkit for Rust.
- `dotenv`: For managing database connection strings from a `.env` file.

You will also need to have **PostgreSQL** installed and running. Create a `.env` file in your project root:
**`.env`**

```
DATABASE_URL=postgres://your_user:your_password@localhost/your_db
```


## Part 1: The Core Application State

To manage our application, we need a way to share state—like the database connection pool and the list of connected clients—safely across many concurrent tasks. We'll use `Arc<Mutex<T>>`.

```rust
use std::sync::{Arc, Mutex};
use tokio::sync::mpsc;
use sqlx::PgPool;
use futures_util::stream::SplitSink;
use tokio_tungstenite::tungstenite::Message;

// A type alias for our WebSocket write-half, which we'll use to send messages.
type WsTx = SplitSink<tokio_tungstenite::WebSocketStream<tokio::net::TcpStream>, Message>;

// Our shared application state.
struct AppState {
    db_pool: PgPool,
    // A list of senders for each connected client. This is how we'll broadcast messages.
    clients: Vec<mpsc::Sender<String>>,
}

// We wrap our state in Arc<Mutex<...>> to share it safely across threads/tasks.
type SharedState = Arc<Mutex<AppState>>;
```


## Part 2: Async Database Operations

We'll use `sqlx` to handle our database operations asynchronously. This ensures that waiting for the database doesn't block our entire server.

**Database Setup SQL**:

```sql
CREATE TABLE IF NOT EXISTS visitor_count (
    id SERIAL PRIMARY KEY,
    count INTEGER NOT NULL
);
-- Initialize with one row if it doesn't exist
INSERT INTO visitor_count (count)
SELECT 0 WHERE NOT EXISTS (SELECT 1 FROM visitor_count);
```

**Async Database Functions**:

```rust
use sqlx::{PgPool, Row};

/// Fetches the current visitor count from the database.
async fn get_visitor_count(pool: &PgPool) -> Result<i32, sqlx::Error> {
    let row = sqlx::query("SELECT count FROM visitor_count")
        .fetch_one(pool)
        .await?;
    Ok(row.get("count"))
}

/// Increments the visitor count and returns the new count.
async fn increment_visitor_count(pool: &PgPool) -> Result<i32, sqlx::Error> {
    let mut tx = pool.begin().await?; // Start a transaction
    let row = sqlx::query("UPDATE visitor_count SET count = count + 1 RETURNING count")
        .fetch_one(&mut *tx)
        .await?;
    tx.commit().await?; // Commit the transaction
    Ok(row.get("count"))
}
```

These functions are marked `async` and use `.await`, so they can be run concurrently without blocking.

## Part 3: The Async HTTP Server (Without a Framework)

Here, we build a raw HTTP server using `tokio`. It will listen for TCP connections and handle basic HTTP `GET` requests.

```rust
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

/// Handles an individual incoming TCP connection.
async fn handle_http_connection(mut stream: TcpStream, state: SharedState) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).await.unwrap();

    let request = String::from_utf8_lossy(&buffer[..]);

    // Rudimentary HTTP parsing
    if request.starts_with("GET / ") {
        // Serve the main HTML page
        let html_content = tokio::fs::read_to_string("index.html").await.unwrap();
        let response = format!("HTTP/1.1 200 OK\r\nContent-Length: {}\r\n\r\n{}", html_content.len(), html_content);
        stream.write_all(response.as_bytes()).await.unwrap();
    } else if request.starts_with("GET /ws ") {
        // This is a WebSocket upgrade request. Hand it off to the WebSocket handler.
        handle_websocket_connection(stream, state).await;
    } else {
        // Handle 404 Not Found
        let response = "HTTP/1.1 404 NOT FOUND\r\n\r\nNot Found";
        stream.write_all(response.as_bytes()).await.unwrap();
    }
}
```

We also need a simple `index.html` file in our project root.
**`index.html`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Live Visitor Counter</title>
</head>
<body>
    <h1>Live Visitor Count</h1>
    <p>Current Visitors: <strong id="visitor-count">Connecting...</strong></p>
    <script>
        const visitorCountElement = document.getElementById('visitor-count');
        const ws = new WebSocket('ws://' + window.location.host + '/ws');

        ws.onopen = () => {
            console.log('WebSocket connection established.');
        };

        ws.onmessage = (event) => {
            console.log('Received message:', event.data);
            visitorCountElement.textContent = event.data;
        };

        ws.onclose = () => {
            console.log('WebSocket connection closed.');
            visitorCountElement.textContent = 'Disconnected';
        };
    </script>
</body>
</html>
```


## Part 4: WebSocket Handling

When a client tries to connect to `/ws`, we "upgrade" their HTTP connection to a WebSocket connection.

```rust
use tokio_tungstenite::accept_async;
use futures_util::{StreamExt, SinkExt};

/// Handles a WebSocket connection after the initial HTTP handshake.
async fn handle_websocket_connection(stream: TcpStream, state: SharedState) {
    let ws_stream = accept_async(stream).await.expect("Failed to accept WebSocket connection");
    println!("New WebSocket connection established.");

    // Increment the count in the database and get the new total.
    let new_count = {
        let app_state = state.lock().unwrap();
        increment_visitor_count(&app_state.db_pool).await.unwrap_or_else(|e| {
            eprintln!("Database error: {}", e);
            -1 // Return an error indicator
        })
    };
    println!("New visitor count: {}", new_count);

    // Broadcast the new count to all connected clients.
    broadcast_message(state.clone(), new_count.to_string()).await;

    // We can also handle incoming messages from this client if needed,
    // but for this project, we'll just keep the connection open.
    let (_write, mut read) = ws_stream.split();
    while let Some(_) = read.next().await {
        // Ignore incoming messages for this simple example.
    }

    // When the client disconnects, the loop will end.
    println!("WebSocket connection closed.");
}

/// Sends a message to all connected WebSocket clients.
async fn broadcast_message(state: SharedState, message: String) {
    let mut app_state = state.lock().unwrap();
    let mut dead_clients = Vec::new();

    // Iterate over all client channels and try to send the message.
    for (i, client_tx) in app_state.clients.iter().enumerate() {
        if client_tx.send(message.clone()).await.is_err() {
            // If sending fails, the client has disconnected. Mark for removal.
            dead_clients.push(i);
        }
    }

    // Remove disconnected clients. We iterate in reverse to preserve indices.
    for i in dead_clients.iter().rev() {
        app_state.clients.remove(*i);
    }
}
```

Wait, we need to adapt our `handle_websocket_connection` to work with the `broadcast_message` function. We need to create a channel *for each client* so we can send them messages.

**Revised `handle_websocket_connection`**:

```rust
async fn handle_websocket_connection(stream: TcpStream, state: SharedState) {
    let ws_stream = accept_async(stream).await.expect("Failed to accept WebSocket connection");
    println!("New WebSocket connection established.");

    let (mut ws_tx, mut ws_rx) = ws_stream.split();

    // Create a new channel for this specific client.
    let (mpsc_tx, mut mpsc_rx) = mpsc::channel::<String>(100);

    // Add this client's sender to our shared list of clients.
    state.lock().unwrap().clients.push(mpsc_tx);

    // Increment count and broadcast
    let new_count = increment_visitor_count(&state.lock().unwrap().db_pool).await.unwrap();
    broadcast_message(state.clone(), new_count.to_string()).await;

    // This loop listens for messages from our broadcast system and forwards
    // them to the actual WebSocket client.
    loop {
        tokio::select! {
            // Forward a broadcast message to the client
            Some(msg) = mpsc_rx.recv() => {
                if ws_tx.send(Message::Text(msg)).await.is_err() {
                    break; // Client disconnected
                }
            },
            // Handle incoming messages from the client (e.g., ping/pong)
            Some(_) = ws_rx.next() => {
                // For this example, we just ignore incoming messages.
            },
            else => break,
        }
    }

    println!("WebSocket connection closed.");
    // The client's `mpsc_tx` will be dropped when this function ends,
    // and `broadcast_message` will clean it up on the next broadcast.
}
```


## Part 5: Putting It All Together and Performance Optimization

Now, let's create the `main` function to tie everything together.

**`main.rs` (Full Code)**

```rust
use std::env;
use std::sync::{Arc, Mutex};
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use tokio::sync::mpsc;
use dotenv::dotenv;
use sqlx::{PgPool, Row, postgres::PgPoolOptions};
use tokio_tungstenite::accept_async;
use futures_util::{stream::SplitSink, StreamExt, SinkExt};
use tokio_tungstenite::tungstenite::Message;

// Type aliases and Structs (as defined above)
type WsTx = SplitSink<tokio_tungstenite::WebSocketStream<TcpStream>, Message>;

struct AppState {
    db_pool: PgPool,
    clients: Vec<mpsc::Sender<String>>,
}

type SharedState = Arc<Mutex<AppState>>;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    dotenv().ok(); // Load .env file

    // --- Database Setup ---
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await?;
    println!("Database connection established.");

    // --- Application State ---
    let state = Arc::new(Mutex::new(AppState {
        db_pool: pool,
        clients: Vec::new(),
    }));

    // --- Start HTTP Server ---
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server listening on http://127.0.0.1:8080");

    while let Ok((stream, _)) = listener.accept().await {
        let state_clone = Arc::clone(&state);
        tokio::spawn(handle_http_connection(stream, state_clone));
    }

    Ok(())
}

// All other functions (handle_http_connection, handle_websocket_connection, etc.) go here...
// (Paste the functions from the previous sections here)
async fn get_visitor_count(pool: &PgPool) -> Result<i32, sqlx::Error> {
    let row = sqlx::query("SELECT count FROM visitor_count").fetch_one(pool).await?;
    Ok(row.get("count"))
}

async fn increment_visitor_count(pool: &PgPool) -> Result<i32, sqlx::Error> {
    let mut tx = pool.begin().await?;
    sqlx::query("UPDATE visitor_count SET count = count + 1").execute(&mut *tx).await?;
    let row = sqlx::query("SELECT count FROM visitor_count").fetch_one(&mut *tx).await?;
    tx.commit().await?;
    Ok(row.get("count"))
}

async fn broadcast_message(state: SharedState, message: String) {
    let mut app_state = state.lock().unwrap();
    let mut dead_clients = Vec::new();
    for (i, client_tx) in app_state.clients.iter().enumerate() {
        if client_tx.send(message.clone()).await.is_err() {
            dead_clients.push(i);
        }
    }
    for i in dead_clients.iter().rev() {
        app_state.clients.remove(*i);
    }
}

async fn handle_http_connection(mut stream: TcpStream, state: SharedState) {
    let mut buffer = vec![0; 1024];
    if stream.read(&mut buffer).await.is_err() { return; }

    let request = String::from_utf8_lossy(&buffer);

    if request.starts_with("GET /ws ") && request.contains("Upgrade: websocket") {
        handle_websocket_connection(stream, state).await;
    } else if request.starts_with("GET / ") {
        let html_content_result = tokio::fs::read_to_string("index.html").await;
        let (status, content) = match html_content_result {
            Ok(c) => ("HTTP/1.1 200 OK", c),
            Err(_) => ("HTTP/1.1 404 NOT FOUND", "Not Found".to_string()),
        };
        let response = format!("{}\r\nContent-Length: {}\r\n\r\n{}", status, content.len(), content);
        if stream.write_all(response.as_bytes()).await.is_err() { return; }
    } else {
        let response = "HTTP/1.1 404 NOT FOUND\r\n\r\nNot Found";
        if stream.write_all(response.as_bytes()).await.is_err() { return; }
    }
}

async fn handle_websocket_connection(stream: TcpStream, state: SharedState) {
    match accept_async(stream).await {
        Ok(ws_stream) => {
            println!("New WebSocket connection established.");
            let (mut ws_tx, mut ws_rx) = ws_stream.split();
            let (mpsc_tx, mut mpsc_rx) = mpsc::channel::<String>(100);

            state.lock().unwrap().clients.push(mpsc_tx);

            let initial_count = get_visitor_count(&state.lock().unwrap().db_pool).await.unwrap_or(0);
            broadcast_message(state.clone(), (initial_count + 1).to_string()).await;

            if let Err(_) = increment_visitor_count(&state.lock().unwrap().db_pool).await {
                eprintln!("Failed to increment visitor count");
            }

            loop {
                tokio::select! {
                    Some(msg) = mpsc_rx.recv() => {
                        if ws_tx.send(Message::Text(msg)).await.is_err() { break; }
                    },
                    Some(res) = ws_rx.next() => {
                        if res.is_err() { break; }
                    },
                    else => break,
                }
            }
            println!("WebSocket connection closed.");
        },
        Err(e) => {
            eprintln!("WebSocket handshake error: {}", e);
        }
    }
}
```


### Performance Optimization and Discussion

- **Why `async` is perfect for this**: Our application is **I/O-bound**. It spends most of its time waiting for network requests (HTTP, WebSocket) and database responses. `async` allows a small number of threads to handle thousands of these waiting connections concurrently, which is vastly more memory-efficient than spawning a thread per connection.
- **Database Connection Pooling**: `sqlx::PgPool` is an async-aware connection pool. It manages a set of database connections, handing them out to tasks as needed. This avoids the expensive overhead of creating a new database connection for every single query.
- **Race Condition Prevention**: We have two critical shared resources: the database and the list of connected clients.
    * **Database**: Race conditions are handled at the database level using a `TRANSACTION`. When we `UPDATE ... RETURNING`, this is an atomic operation on the database side, ensuring the count is always correct.
    * **Client List**: The `Vec<mpsc::Sender<String>>` is protected by `Arc<Mutex<...>>`. The `Mutex` ensures that only one task can add or remove a client from the list at a time, preventing data races. The lock is held for a very short duration, minimizing contention.
- **Broadcasting Efficiency**: Our broadcast mechanism iterates through clients and sends messages. If a send fails (because the client disconnected), we mark it for removal. This self-healing mechanism keeps our client list clean. A more advanced implementation might use a `tokio::sync::broadcast` channel for even more efficient broadcasting.
<span style="display:none">[^1][^10][^11][^12][^2][^3][^4][^5][^6][^7][^8][^9]</span>

1. https://users.rust-lang.org/t/basic-http-server-implementation/104368
2. https://www.reddit.com/r/rust/comments/1jtl7f9/built_my_own_http_server_in_rust_from_scratch/
3. https://rust-lang.github.io/async-book/09_example/00_intro.html
4. https://stackoverflow.com/questions/14154753/how-do-i-make-an-http-request-from-rust
5. https://www.petergirnus.com/blog/how-to-make-http-requests-in-rust
6. https://www.videosdk.live/developer-hub/websocket/rust-websocket
7. https://stackoverflow.com/questions/76237191/trying-to-understand-how-to-create-an-async-function-in-rust-that-returns-a-mysq
8. https://bitskingdom.com/blog/web-apps-rust-performance-optimization/
9. https://dev.to/priyanshuverma/a-masochists-journey-to-building-an-http-server-from-scratch-1272
10. https://dev.to/campbellgoe/websocket-starter-in-rust-with-client-and-server-example-4ahj
11. https://blog.stackademic.com/rust-dissel-lambda-907ae8209af6
12. https://blog.yoshuawuyts.com/why-async-rust/
