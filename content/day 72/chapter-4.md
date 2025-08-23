---
title: "Health Checks"
description: "Add health checks to your Rust services to ensure they are running properly in production."
date: 2025-11-11
weight: 4
---

# Adding Health Checks to Your Rust Services

Implementing health checks in your Rust services is a critical practice for building reliable, production-ready applications. A health check is a simple, dedicated endpoint (like `/health`) that external systems can query to determine if your application is running correctly. This is essential for automated systems like load balancers, container orchestrators (like Kubernetes), and monitoring services.

## The "Restaurant Inspection" Analogy: Checking for Health 👩🍳

Think of your running application as a professional restaurant kitchen.

- **A Health Check Endpoint**: A visit from a health inspector.
- **A Basic "Liveness" Check**: The inspector just peeks through the door to see if the lights are on and someone is inside. This confirms the restaurant (your process) is running.
- **A "Readiness" Check**: The inspector comes inside and checks if the ovens are hot, the ingredients are prepped, and the staff is ready to cook. This confirms the restaurant is ready to take new orders (handle traffic).

Orchestration systems use these checks to make decisions:

- If a **liveness** probe fails, the system assumes the application has crashed and will restart it.
- If a **readiness** probe fails, the system assumes the application is temporarily busy or not yet initialized. It will stop sending new traffic to it until the probe succeeds again.[^1]


## Implementing a Basic Health Check

The simplest health check is an HTTP endpoint that always returns a `200 OK` status. This serves as a basic **liveness** probe. Let's implement one using the popular `actix-web` framework.

### Step 1: Project Setup

First, add `actix-web` to your `Cargo.toml`:

```toml
[dependencies]
actix-web = "4"
```


### Step 2: Create the Health Check Handler

In your `src/main.rs`, create a simple asynchronous function that will handle requests to your health endpoint.

```rust
use actix_web::{get, App, HttpResponse, HttpServer, Responder};

/// This is our health check handler.
/// It is decorated with `#[get("/health")]` to tell Actix to route
/// GET requests for the "/health" path to this function.
#[get("/health")]
async fn health_check() -> impl Responder {
    // Return a simple 200 OK response.
    // The body is optional, but it's good practice to return a simple confirmation.
    HttpResponse::Ok().body("OK")
}
```

This function is incredibly simple: it takes no arguments and returns an `HttpResponse` with a status code of `200 OK`.[^2]

### Step 3: Create and Run the Web Server

Now, we need a `main` function to create the server, register our health check handler, and run it.

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    println!("🚀 Server starting on http://127.0.0.1:8080");

    HttpServer::new(|| {
        // Create a new App instance and register our health_check service.
        App::new().service(health_check)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```


### Step 4: Test It

Run your application with `cargo run`. In another terminal, use `curl` to test the endpoint:

```bash
curl -i http://127.0.0.1:8080/health
```

You should see a response like this, confirming your service is live:

```
HTTP/1.1 200 OK
content-length: 2
content-type: text/plain; charset=utf-8
date: ...

OK
```


## Advanced Health Checks: Checking Dependencies

A simple "I'm alive" check is good, but a great health check also confirms that the service is actually *ready to do work*. This usually means checking its connections to critical dependencies, like a database. This is a **readiness** probe.[^1]

Let's extend our example to check the status of a PostgreSQL database connection using `sqlx`.

### Step 1: Update Dependencies

Add `sqlx` and `serde` (for JSON responses) to your `Cargo.toml`:

```toml
[dependencies]
actix-web = "4"
serde = { version = "1.0", features = ["derive"] }
sqlx = { version = "0.6", features = ["runtime-actix-native-tls", "postgres"] }
```


### Step 2: Create Shared Application State

We need a way to share the database connection pool with our request handlers. A common pattern is to create an `AppState` struct.

```rust
use sqlx::PgPool;

// This struct will hold our database connection pool.
struct AppState {
    db: PgPool,
}
```


### Step 3: Implement the Advanced Health Check Handler

This handler will now take the application state as an argument, perform a simple query to check the database connection, and return a detailed JSON response.

```rust
use actix_web::{web, get, HttpResponse, Responder};
use serde::Serialize;

#[derive(Serialize)]
struct HealthResponse {
    status: String,
    database: String,
}

#[get("/health")]
async fn detailed_health_check(data: web::Data<AppState>) -> impl Responder {
    // Try to run a simple, lightweight query against the database.
    // `SELECT 1` is a common choice as it's very fast.
    let db_status = match sqlx::query("SELECT 1").execute(&data.db).await {
        Ok(_) => "up".to_string(),
        Err(e) => {
            // Log the actual error for debugging
            eprintln!("Database health check failed: {}", e);
            "down".to_string()
        }
    };

    let is_healthy = db_status == "up";

    let response = HealthResponse {
        status: if is_healthy { "up" } else { "down" }.to_string(),
        database: db_status,
    };

    if is_healthy {
        HttpResponse::Ok().json(response)
    } else {
        // If a dependency is down, return a 503 Service Unavailable status.
        HttpResponse::ServiceUnavailable().json(response)
    }
}
```


### Step 4: Update the Server to Manage State

Finally, update your `main` function to establish the database connection and pass the state to the `App`.

```rust
use actix_web::{web, App, HttpServer};
use sqlx::postgres::PgPoolOptions;

// ... (AppState struct and detailed_health_check function) ...

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Make sure to set the DATABASE_URL environment variable.
    let database_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");

    // Create the database connection pool.
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(&database_url)
        .await
        .expect("Failed to create database pool.");

    println!("🚀 Server starting on http://127.0.0.1:8080");

    HttpServer::new(move || {
        App::new()
            // Store the connection pool in the app data.
            .app_data(web::Data::new(AppState { db: pool.clone() }))
            // Register the health check handler.
            .service(detailed_health_check)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Now, when you run your application:

- If the database is connected, `GET /health` will return a `200 OK` with a JSON body like `{"status":"up","database":"up"}`.
- If the database is down, it will return a `503 Service Unavailable` with `{"status":"down","database":"down"}`.

This gives your orchestration system a clear signal about whether your service is just alive or truly ready to handle requests. By implementing both basic and dependency-aware health checks, you can build robust, resilient, and production-ready services in Rust.

1. https://itnext.io/implementing-the-health-check-api-pattern-with-rust-eaef04cb4d2d
2. https://www.lpalmieri.com/posts/2020-08-09-zero-to-production-3-how-to-bootstrap-a-new-rust-web-api-from-scratch/
3. https://bcnrust.github.io/devbcn-workshop/backend/13_health_check.html
4. https://www.reddit.com/r/rust/comments/qgzat5/implementing_the_health_check_api_pattern_with/
5. https://www.linkedin.com/learning/web-apis-in-rust/creating-a-health-check-endpoint
6. https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks
7. https://blog.logrocket.com/async-crud-web-service-rust-warp/
8. https://community.fly.io/t/health-check-not-passing-even-though-local-request-to-the-url-returns-an-200-ok-http-response/11595
9. https://stackoverflow.com/questions/53873153/should-health-checks-call-other-app-health-checks
10. https://www.apollographql.com/docs/apollo-server/monitoring/health-checks
11. https://dev.to/priyanshuverma/a-masochists-journey-to-building-an-http-server-from-scratch-1272
12. https://stackoverflow.com/questions/65259144/http-health-check-get-or-head-and-200-or-204-response
13. https://marcobacis.com/blog/load-balancer-rust-3/
14. https://www.zupzup.org/rust-webapp/index.html
15. https://betterprogramming.pub/creating-a-web-server-using-rust-rocket-1e4939e582df
16. https://brandur.org/rust-web
17. https://bcnrust.github.io/devbcn-workshop/backend/15_testing.html
18. https://dev.to/sirneij/full-stack-authentication-system-using-rust-actix-web-and-sveltekit-1cc6
19. https://github.com/wpcodevo/simple-api-actix-web
