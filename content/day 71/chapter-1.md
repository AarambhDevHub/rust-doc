---
title: "Publishing to crates.io"
description: "Learn how to publish your Rust library to crates.io, including setting up accounts, metadata, and versioning."
date: 2025-11-10
weight: 1
---

# Publishing Your Library to crates.io: A Beginner's Guide

Publishing your Rust library to `crates.io`, the official Rust package registry, is a straightforward process that allows you to share your code with the entire Rust community. This guide will walk you through every step, from setting up your account to managing versions of your crate.

## The "Public Library" Analogy 📚

Think of `crates.io` as a massive public library.

- **Your Crate**: A book you have written.
- **Publishing**: The process of getting your book accepted, cataloged, and placed on the library's shelves for everyone to read and use.
- **`Cargo.toml`**: The cover and inside flap of your book. It contains the title, author, a brief summary, and other essential information.
- **Versioning**: Releasing new editions of your book with updates and corrections.

Just like a library has rules to ensure quality and organization, `crates.io` has a set of steps to ensure that every crate is well-documented and easy for others to use.

***

### Step 1: Create Your `crates.io` Account

Before you can publish anything, you need to become a registered "author" at the library.

1. **Visit `crates.io`**: Go to the official [crates.io](https://crates.io) website.[^1]
2. **Log in with GitHub**: The site uses GitHub for authentication. You will need to authorize `crates.io` to access your GitHub account information.
3. **Verify Your Email**: In your `crates.io` Account Settings, make sure you have verified your email address. This is a required step.
4. **Generate an API Token**: In your Account Settings, create a new API token. **This token is a secret, like a password.** Copy it immediately, as you will not be able to see it again.[^2]

### Step 2: Log in with Cargo on Your Machine

Now, you need to introduce yourself to Cargo, the Rust package manager, so it knows who you are when you try to publish.

- Run the `cargo login` command in your terminal and paste the API token you just copied:

```bash
cargo login <your_copied_api_token>
```

- This command will save your credentials securely in a file at `~/.cargo/credentials.toml`. You only need to do this once per computer.[^3]

***

### Step 3: Prepare Your Crate's Metadata (`Cargo.toml`)

This is the most important step for making your crate professional and usable. Your `Cargo.toml` file must contain specific metadata. If this information is missing, `crates.io` will reject your publish attempt.[^4]

Here is a well-formed `[package]` section:

```toml
[package]
name = "my-awesome-utility"  # Must be unique on crates.io
version = "0.1.0"             # Follow Semantic Versioning (SemVer)
authors = ["Your Name <your.email@example.com>"]
edition = "2021"

# Required for publishing
license = "MIT OR Apache-2.0" # Use a standard SPDX license identifier
description = "A short, clear description of what your crate does."

# Highly recommended for discoverability
readme = "README.md"          # Points to your README file
repository = "https://github.com/your-username/my-awesome-utility"
homepage = "https://github.com/your-username/my-awesome-utility"
keywords = ["cli", "utility", "data-processing"] # Helps people find your crate
categories = ["command-line-utilities"]         # Standardized categories
```

**Key Points:**

- **`name`**: Must be unique. Search on `crates.io` first to make sure your desired name isn't already taken.[^5]
- **`license`**: You must provide a valid license identifier from the [SPDX list](https://spdx.org/licenses/). `MIT OR Apache-2.0` is a common and flexible choice for Rust projects.
- **`description`**: This is a required field. Make it descriptive and helpful.
- **`repository` and `homepage`**: These links are crucial for helping users find your source code and documentation.

***

### Step 4: Final Checks Before Publishing (The Dry Run)

Before you publish permanently, Cargo gives you a chance to check your work.

1. **Check which files will be included**:

```bash
cargo package --list
```

Cargo automatically respects your `.gitignore` file, but if you need to exclude other files (like large test assets), you can add an `exclude` field in `Cargo.toml`.
2. **Perform a dry run**:

```bash
cargo publish --dry-run
```

This command simulates the entire publishing process without actually uploading anything. It will compile your code, run checks, and package your crate, reporting any warnings or errors that would cause the real publish to fail. **Always do a dry run first**.[^2]

***

### Step 5: Publish!

Once your metadata is complete and the dry run passes without errors, you are ready to publish.

- Run the publish command:

```bash
cargo publish
```

- Cargo will package your crate into a `.crate` file, upload it to `crates.io`, and add it to the registry.

**A publish is permanent.** You can never delete a specific version of a crate, though you can "yank" it to prevent new projects from depending on it while allowing existing ones to continue working.[^4]

***

### Step 6: Publishing New Versions

To update your crate, you need to publish a new version.

1. **Update the `version` number** in your `Cargo.toml`. Follow the rules of **Semantic Versioning (SemVer)**:
    - **Patch release (0.1.0 -> 0.1.1)**: For bug fixes and other small, backwards-compatible changes.
    - **Minor release (0.1.0 -> 0.2.0)**: For new features that are backwards-compatible.
    - **Major release (0.1.0 -> 1.0.0)**: For breaking changes that are not backwards-compatible.
2. **Update your documentation** and `CHANGELOG.md` file to reflect the changes.
3. **Run `cargo publish` again**.

Congratulations! You have successfully shared your work with the Rust community. By providing good documentation, clear metadata, and responsible versioning, you contribute to the vibrant and reliable ecosystem that makes Rust a great language to work with.

1. https://crates.io
2. https://doc.rust-lang.org/cargo/reference/publishing.html
3. https://rust-cli.github.io/book/tutorial/packaging.html
4. https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html
5. https://dev.to/theoforger/my-first-publish-to-cratesio-and-cross-compilation-3a6o
6. https://www.youtube.com/watch?v=8HlMJUYTsTQ
7. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch14-02-publishing-to-crates-io.html
8. https://www.youtube.com/watch?v=4TI153PIEDQ
9. https://www.printlnhello.world/blog/publishing-to-crates-io/
10. https://hamatti.org/posts/learning-rust-3-crates-io-publishing-your-package/
11. https://www.reddit.com/r/rust/comments/qw5ea1/how_to_publish_your_first_rust_crate/
12. https://www.youtube.com/watch?v=FzbxAhTqK9s
13. https://users.rust-lang.org/t/blog-post-crate-publishing-tips-and-guidelines/39391
