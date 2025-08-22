---
title: "Configuration Files in CLI Tools"
description: "Explore how to load and manage configuration files (TOML, YAML, JSON) for CLI apps in Rust."
date: 2025-11-03
weight: 3
---

# Configuration Files in CLI Tools: Loading and Managing TOML, YAML, and JSON in Rust

Managing configuration files is a core skill for robust CLI tool development. Think of your CLI app as a restaurant: every time it runs, you want it to behave according to the current "menu" (the config file), which may specify which features to offer, defaults, network addresses, or user options.

Rust provides powerful, ergonomic libraries for loading configuration from TOML, YAML, and JSON, using the same approach: **deserialize** config files into plain Rust structs.

***

## Why Configuration Files?

- **Separation of logic and configuration:** Avoid hard-coding settings in your source code.
- **Easier deployment:** Settings can be tweaked per environment or user.
- **Flexibility:** CLI users can easily override or extend behavior via simple edits.

***

## Common Approaches in Rust

### 1. Use a Serde-Compatible Struct

No matter the file format (TOML, YAML, JSON), you define your config as a plain Rust struct. Use the `serde::Deserialize` trait for loading:

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Config {
    server: String,
    port: u16,
    debug: bool,
}
```


***

### 2. Pick a File Format and Library

**TOML**: Used by Cargo and many Rust projects, supported by the `toml` crate.
**YAML**: Human-friendly for complex/nested configs, use the `serde_yaml` crate.
**JSON**: Widely supported and machine friendly, use `serde_json`.

Most libraries use `serde` "under the hood" for deserialization, so switching formats requires very little code change.

***

### 3. Reading and Parsing Configuration Files

#### Example: Parsing a JSON Config File

```rust
use serde::Deserialize;
use std::fs;

#[derive(Deserialize, Debug)]
struct Config {
    server: String,
    port: u16,
    debug: bool,
}

fn load_config(filename: &str) -> Result<Config, Box<dyn std::error::Error>> {
    let contents = fs::read_to_string(filename)?;
    let config = serde_json::from_str(&contents)?;
    Ok(config)
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let config = load_config("config.json")?;
    println!("Loaded config: {:?}", config);
    Ok(())
}
```

- `serde_json::from_str` will map the file into your struct and error on mismatch.[^1][^2]

***

#### Example: Parsing a TOML Config File (Recommended for CLI Tools)

Add dependencies in `Cargo.toml`:

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
toml = "0.7"
```

```rust
use serde::Deserialize;
use std::fs;

#[derive(Deserialize, Debug)]
struct Config {
    server: String,
    port: u16,
    debug: bool,
}

fn load_toml_config(filename: &str) -> Result<Config, Box<dyn std::error::Error>> {
    let contents = fs::read_to_string(filename)?;
    let config = toml::from_str(&contents)?;
    Ok(config)
}

fn main() {
    match load_toml_config("config.toml") {
        Ok(config) => println!("Config loaded: {:?}", config),
        Err(e) => eprintln!("Error reading config: {e}"),
    }
}
```

**`config.toml`:**

```toml
server = "localhost"
port = 8080
debug = true
```


***

#### Example: Parsing a YAML Config File

Add in `Cargo.toml`:

```toml
serde_yaml = "0.9"
```

And load with:

```rust
let contents = fs::read_to_string("config.yaml")?;
let config: Config = serde_yaml::from_str(&contents)?;
```


***

### 4. Managing Multiple File Formats

Use libraries like [`config`](https://github.com/rust-cli/config-rs) or [`config-file`](https://docs.rs/config-file), which let you easily support JSON, YAML, TOML, and even layered configs and environment variables:[^3][^4]

```rust
use config::{Config, File};

fn main() {
    // Loads settings from config.toml, config.yaml, or config.json if present
    let mut settings = Config::builder()
        .add_source(File::with_name("config"))
        .build()
        .unwrap();

    // Access settings as typed values
    let server: String = settings.get("server").unwrap();
    let port: u16 = settings.get("port").unwrap();
}
```


***

### 5. Error Handling

- **Always handle parse errors**, e.g., missing fields or bad types, to show clear messages to users.
- You can use Rust’s strong error type system (`Result`, `Option`), and provide helpful fallback defaults if needed.[^2]

***

### Tips and Best Practices

- Set sane defaults in your config struct and override with file/CLI/env.
- Document your config format (generate a sample config file).
- Consider supporting overrides from environment variables for Docker/cloud use cases (`config` crate supports this).
- Use `clap` CLI parser's support for loading config files to bake-in user-override logic.

***

## Summary Table

| Format | Crate | Read Example | Comment |
| :-- | :-- | :-- | :-- |
| TOML | `toml` | `toml::from_str` | Cargo's default, readable |
| YAML | `serde_yaml` | `serde_yaml::from_str` | Human-friendly, lenient |
| JSON | `serde_json` | `serde_json::from_str` | Widely used, tools support |
| Any | `config` | Layered, multi-format support | Advanced, layering, ENV merge |

By leveraging these patterns, your CLI tool becomes highly configurable, user-friendly, and ready for production deployments.[^4][^1][^2]

1. https://towardsdatascience.com/get-started-with-rust-installation-your-first-cli-tool-a-beginners-guide/
2. https://www.w3resource.com/rust/error_handling/rust-error-propagation-exercise-4.php
3. https://github.com/rust-cli/config-rs
4. https://docs.rs/config-file
5. https://www.reddit.com/r/rust/comments/zmicie/how_do_you_manage_configuration_in_rust/
6. https://www.youtube.com/watch?v=4EmKgrzHfv4
7. https://tarquin-the-brave.github.io/blog/posts/generating-config-reference-rust-cli/
8. https://doc.rust-lang.org/cargo/reference/config.html
9. https://stackoverflow.com/questions/59013919/rust-how-to-read-a-config-file-at-runtime-and-store-in-a-global-struct-accessi
10. https://crates.io/crates/clap-config-file
