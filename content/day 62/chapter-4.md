---
title: "Watching File Changes with Rust"
description: "Learn how to watch for file changes in real time using notify and similar crates in Rust."
date: 2025-11-02
weight: 4
---

# Watching File Changes with Rust: Real-Time Monitoring Using the `notify` Crate

Want to build a tool that reloads config files, live-refreshes a web app, or runs code when files change? In Rust, the `notify` crate lets you **watch the filesystem for changes in real time**—no polling or inefficient hacks required. It works much like a professional building security system: you install sensors (watchers) on files or folders, and they *instantly* notify (send events to) your program whenever something changes.

## The Building Security System Analogy 🕵️♂️

- **The Files/Folders**: The sensitive rooms in your building.
- **The Watcher (notify crate)**: The security sensors installed at each room.
- **Your Rust Code (the callback/channel)**: The security control room that receives a message as soon as a door is opened, closed, or tampered with—allowing immediate action.

***

## Step-by-Step Guide: Watching Files with the `notify` Crate

### Step 1: Add Dependencies

Add these to your `Cargo.toml`:

```toml
[dependencies]
notify = "6"
log = "0.4"
env_logger = "0.10"   # For logging (optional, helps see events)
```


### Step 2: Basic Example—Watch a File or Folder for Changes

Below is a minimal Rust program that watches a given path for changes and prints events as they happen.

```rust
use notify::{Watcher, RecommendedWatcher, RecursiveMode, Config, Result, Event};
use std::sync::mpsc::channel;
use std::path::Path;

fn main() -> Result<()> {
    // Setup logging so you can see output
    env_logger::init();

    // The path you want to watch (file or folder)
    let path = std::env::args().nth(1).expect("Usage: watch <PATH>");

    // Channel to receive file system events
    let (tx, rx) = channel();

    // Create default watcher (cross-platform)
    let mut watcher = RecommendedWatcher::new(tx, Config::default())?;

    // Start watching the specified path recursively (for folders)
    watcher.watch(Path::new(&path), RecursiveMode::Recursive)?;

    println!("Watching {:?} for changes...", path);

    // Listen for events
    for res in rx {
        match res {
            Ok(event) => println!("Change detected: {:?}", event),
            Err(e) => println!("Watch error: {:?}", e),
        }
    }
    Ok(())
}
```

Save this as `main.rs` and run with:

```
cargo run -- <path_to_watch>
```

Now, edit, create, or delete files in the target — changes are printed out instantly!

***

### Step 3: Handling Specific Events

You can match on `event.kind` to act only on certain changes:

```rust
match event.kind {
    notify::EventKind::Create(_) => println!("File created: {:?}", event.paths),
    notify::EventKind::Modify(_) => println!("File modified: {:?}", event.paths),
    notify::EventKind::Remove(_) => println!("File removed: {:?}", event.paths),
    _ => (),
}
```

This is useful for building file synchronizers, daemons, or live-reloading applications.[^1]

***

### Step 4: Tuning Responsiveness

The watcher can use debouncing (bundling rapid changes into one event). Set debounce in `Config`:

```rust
let mut watcher = RecommendedWatcher::new(tx, Config::default().with_poll_interval(std::time::Duration::from_millis(100)))?;
```

Shorter intervals improve responsiveness for real-time needs.[^2]

***

### Step 5: Recap \& Best Practices

- The `notify` crate automatically chooses the best backend: inotify (Linux), FSEvents (macOS), or ReadDirectoryChangesW (Windows).[^3]
- You can watch files, directories, or entire directory trees.[^4]
- Always check event details (`event.paths`, `event.kind`) for more precise handling.
- Use logging or print statements for debugging your watch logic.
- For larger projects, handle errors (e.g., permission denied) and be careful about cross-platform quirks (like editors that rewrite files instead of editing in place).[^5]

***

By following these steps, you can efficiently monitor file or directory changes in a robust and cross-platform way using modern Rust tools—perfect for developers who need immediate reactions to filesystem events!

1. https://users.rust-lang.org/t/details-from-the-events-using-the-notify-crate/108830
2. https://users.rust-lang.org/t/how-to-make-good-usage-of-the-notify-crate-for-responsive-events/55891
3. https://github.com/notify-rs/notify
4. https://stackoverflow.com/questions/55440289/how-do-i-recursively-watch-file-changes-in-rust
5. https://docs.rs/notify
6. https://www.reddit.com/r/rust/comments/1g76noh/how_to_watch_a_file_for_changes/
7. https://www.youtube.com/watch?v=1SDiYadU5to
8. https://users.rust-lang.org/t/problem-with-notify-crate-v6-1/99877
9. https://crates.io/crates/notify/range/^4.0
