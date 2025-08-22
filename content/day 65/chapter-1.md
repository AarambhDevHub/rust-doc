---
title: "CLI Project in Rust"
description: "Build a feature-rich command line tool in Rust. Learn configuration management, plugin architecture, and performance optimization techniques to create efficient and extensible CLI applications."
date: 2025-11-07
weight: 1
---

# Building a Feature-Rich Command-Line Tool in Rust

This guide provides a comprehensive walkthrough for building a feature-rich, extensible command-line (CLI) tool in Rust. We will cover:

- **Argument Parsing**: Using the `clap` crate for robust command and argument handling.
- **Configuration Management**: Sourcing configuration from files and command-line arguments.
- **Plugin Architecture**: Designing a system to extend the tool's functionality dynamically.
- **Performance**: Leveraging Rust's strengths for efficient execution.


## The "Modular Multi-Tool" Analogy 🛠️

Think of the CLI tool we're building as a high-tech, modular multi-tool, like a Leatherman or a Swiss Army Knife.

- **The Main Application**: The sturdy frame of the multi-tool. It provides the core structure and the mechanism to attach new tools.
- **CLI Arguments (`clap`)**: The user manual that explains how to open and use each tool.
- **Configuration (`config` crate)**: A set of user-defined presets for how the tools should behave (e.g., "always use the Phillips head screwdriver in high-torque mode").
- **Plugins**: The individual tools themselves (the pliers, the knife, the can opener). You can add or remove them, and each one performs a specific job.

Our goal is to build the frame and a system for adding new tools, making our CLI powerful and extensible.

## Core Components and Libraries

| **Component** | **Crate(s)** | **Purpose** |
| :-- | :-- | :-- |
| **CLI Parsing** | `clap` | The de facto standard for parsing command-line arguments, subcommands, and flags. |
| **Configuration** | `config`, `serde` | For loading configuration from files (e.g., TOML, JSON) and the environment. |
| **Plugin System** | `libloading` (for dynamic) | Enables loading external libraries as plugins at runtime. |
| **Asynchronous Runtime** | `tokio` | For high-performance I/O operations and concurrent tasks. |


***

## Part 1: Setting Up the Project and CLI Arguments

Let's start by defining our CLI's structure using `clap`. Our tool, `my-cli`, will have two subcommands: `greet` and `config`.

**`Cargo.toml`**

```toml
[package]
name = "my_cli"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
config = "0.13"
serde = { version = "1.0", features = ["derive"] }
```

**`src/main.rs`**

```rust
use clap::{Parser, Subcommand};

#[derive(Parser, Debug)]
#[command(name = "my-cli", version = "1.0", about = "A feature-rich CLI tool")]
struct Cli {
    /// Path to a custom configuration file
    #[arg(short, long, global = true)]
    config: Option<String>,

    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Prints a greeting
    Greet {
        /// The name to greet
        name: String,
        /// How many times to repeat the greeting
        #[arg(short, long, default_value_t = 1)]
        times: u8,
    },
    /// Shows the current configuration
    Config {
        /// The configuration key to display
        key: Option<String>,
    },
}

fn main() {
    let cli = Cli::parse();
    println!("CLI args parsed: {:?}", cli);

    // We'll add logic here later
    match cli.command {
        Commands::Greet { name, times } => {
            for _ in 0..times {
                println!("Hello, {}!", name);
            }
        }
        Commands::Config { key } => {
            // Configuration logic will go here
            println!("Configuration command (key: {:?})", key);
        }
    }
}
```

**Run it:**

- `cargo run -- greet World` -> Prints "Hello, World!"
- `cargo run -- greet World -t 3` -> Prints "Hello, World!" three times.
- `cargo run -- --help` -> `clap` automatically generates a detailed help message.

***

## Part 2: Configuration Management

A good CLI tool allows users to configure its behavior. We'll use the `config` crate to merge settings from a default file, a user-provided file, and environment variables.

1. **Create a default config file:**
**`config/default.toml`**

```toml
greeting_prefix = "Hi"
```

2. **Define a `Settings` struct:**
**`src/config.rs`** (a new file)

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
pub struct Settings {
    pub greeting_prefix: String,
}

impl Settings {
    pub fn new(config_path: Option<&String>) -> Result<Self, config::ConfigError> {
        let mut builder = config::Config::builder()
            // 1. Add default settings
            .add_source(config::File::with_name("config/default"));

        // 2. Add user-provided settings file (optional)
        if let Some(path) = config_path {
            builder = builder.add_source(config::File::with_name(path).required(true));
        }

        // 3. Add environment variables (e.g., `MY_CLI_GREETING_PREFIX="Yo"`)
        builder = builder.add_source(config::Environment::with_prefix("MY_CLI"));

        builder.build()?.try_deserialize()
    }
}
```

3. **Integrate it into `main.rs`:**

```rust
// In src/main.rs
mod config;
use config::Settings;
// ...

fn main() {
    let cli = Cli::parse();
    let settings = Settings::new(cli.config.as_ref()).expect("Failed to load settings");

    match cli.command {
        Commands::Greet { name, times } => {
            for _ in 0..times {
                // Use the prefix from our settings!
                println!("{}, {}!", settings.greeting_prefix, name);
            }
        }
        Commands::Config { key } => {
            println!("Current settings: {:?}", settings);
        }
    }
}
```


Now you can override the greeting prefix with a custom file or an environment variable!

***

## Part 3: A Dynamic Plugin Architecture

A plugin architecture makes a CLI truly extensible. The most flexible approach is to load dynamic libraries (`.so`, `.dll`, `.dylib`) at runtime. This allows users to drop new plugins into a folder without recompiling the main application.

**The Strategy:**

1. **Define a `Plugin` trait**: A common interface that all plugins must implement.
2. **Use `libloading`**: A crate to safely load dynamic libraries.
3. **The Plugin Crate**: Each plugin is a separate Rust project compiled as a `cdylib`.

### 1. The Core Application (`my-cli`)

**`src/plugin.rs`**

```rust
// A trait that all plugins must implement.
pub trait Plugin {
    fn name(&self) -> &'static str;
    fn execute(&self);
}

// A macro to declare the plugin entry point in the plugin crate.
#[macro_export]
macro_rules! declare_plugin {
    ($plugin_type:ty, $constructor:path) => {
        #[no_mangle]
        pub extern "C" fn _plugin_create() -> *mut dyn Plugin {
            let constructor: fn() -> $plugin_type = $constructor;
            let object = constructor();
            let boxed: Box<dyn Plugin> = Box::new(object);
            Box::into_raw(boxed)
        }
    };
}
```

**Modify `main.rs` to load plugins:**

```rust
// In src/main.rs
mod plugin;
use libloading::{Library, Symbol};
// ...

fn load_plugins() {
    // Assume plugins are in a `plugins` directory
    let plugin_dir = "plugins";
    if let Ok(entries) = std::fs::read_dir(plugin_dir) {
        for entry in entries.flatten() {
            if entry.path().extension().map_or(false, |e| e == "so" || e == "dll" || e == "dylib") {
                unsafe {
                    let lib = Library::new(entry.path()).unwrap();
                    let constructor: Symbol<unsafe extern "C" fn() -> *mut dyn plugin::Plugin> = lib.get(b"_plugin_create").unwrap();
                    let plugin_ptr = constructor();
                    let plugin = Box::from_raw(plugin_ptr);

                    println!("Loaded plugin: {}", plugin.name());
                    plugin.execute();

                    // Important: We must forget the library, otherwise it unloads
                    // when `lib` goes out of scope, invalidating the plugin.
                    std::mem::forget(lib);
                }
            }
        }
    }
}

fn main() {
    // ... (parsing and config logic)
    load_plugins();
    // ... (match statement)
}
```


### 2. A Sample Plugin (`hello_plugin`)

1. **Create a new library crate**: `cargo new --lib hello_plugin`
2. **Configure `Cargo.toml` to build a dynamic library**:
**`hello_plugin/Cargo.toml`**

```toml
[package]
name = "hello_plugin"
# ...

[lib]
crate-type = ["cdylib"] # Crucial!

[dependencies]
my_cli = { path = "../" } # Depend on the main crate to get the Plugin trait
```

3. **Implement the plugin**:
**`hello_plugin/src/lib.rs`**

```rust
use my_cli::{declare_plugin, plugin::Plugin};

struct HelloPlugin;

impl Plugin for HelloPlugin {
    fn name(&self) -> &'static str { "Hello Plugin" }
    fn execute(&self) {
        println!("This message is from the Hello Plugin!");
    }
}

// Expose the plugin's constructor.
declare_plugin!(HelloPlugin, HelloPlugin::new);

impl HelloPlugin {
    fn new() -> Self { Self }
}
```


**To use it:**

1. `cd hello_plugin && cargo build`
2. `mkdir ../plugins`
3. Copy the compiled library (e.g., `target/debug/libhello_plugin.so`) into the `plugins` directory.
4. Run `cargo run` from the main `my-cli` directory. It will find and load the plugin!

***

## Part 4: Performance Optimization

Rust is fast by default, but for a high-performance CLI, consider these techniques:

1. **Use `tokio` for I/O**: For commands that perform file or network operations, use `tokio`'s asynchronous APIs to prevent blocking the main thread.
2. **Parallelize with `rayon`**: For CPU-intensive tasks (like searching through many files), use `rayon`'s parallel iterators to leverage all available CPU cores.
3. **Minimize Allocations**: In hot loops, avoid creating new `String`s or `Vec`s repeatedly. Reuse buffers where possible.
4. **Profile Your Code**: Use tools like `perf` and `flamegraph` to find performance bottlenecks. Don't optimize what you haven't measured.

By combining `clap` for a great user interface, `config` for flexibility, and a dynamic plugin architecture for extensibility, you can build powerful, professional-grade CLI tools in Rust that are both efficient and a joy to use.

1. https://rust-cli.github.io/book/index.html
2. https://www.reddit.com/r/rust/comments/sboyb2/designing_a_rust_rust_plugin_system/
3. https://betterprogramming.pub/building-cli-apps-in-rust-what-you-should-consider-99cdcc67710c
4. https://users.rust-lang.org/t/plugin-architecture-with-rust/23334
5. https://zicklag.github.io/rust-tutorials/rust-plugins.html
6. https://dev.to/mineichen/plugin-based-architecture-in-rust-4om7
7. https://www.youtube.com/watch?v=D_76TcfzDTU
8. https://www.linkedin.com/pulse/developing-cli-applications-rust-comprehensive-guide-clap-burlakov-8saae
9. https://code.visualstudio.com/docs/languages/rust
10. https://stackoverflow.com/questions/2768104/how-to-create-a-flexible-plug-in-architecture
11. https://www.arroyo.dev/blog/rust-plugin-systems/
12. https://nullderef.com/blog/plugin-tech/
