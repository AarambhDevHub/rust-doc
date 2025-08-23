---
title: "Dependency Auditing"
description: "Learn how to audit dependencies in Rust projects using tools like cargo-audit."
date: 2025-11-12
weight: 3
---

# Dependency Auditing in Rust with `cargo-audit`

In modern software development, your project's security is not just about the code you write; it's also about the code you *depend on*. The Rust ecosystem on `crates.io` is vast and powerful, but this reliance on third-party libraries means you are also inheriting their potential security vulnerabilities. This is where dependency auditing becomes a critical practice.

**Dependency auditing** is the process of scanning your project's dependencies to check for known security vulnerabilities. For Rust developers, the essential tool for this job is `cargo-audit`.

## The "Building Inspection" Analogy 🏢

Think of your Rust application as a skyscraper you've just built.

- **Your Code**: The floors and rooms you designed and constructed yourself.
- **Your Dependencies**: The foundational materials, electrical wiring, and plumbing systems supplied by other companies.

You might be a fantastic architect, but if the steel beams from a third-party supplier have a known structural flaw, your entire building is at risk. A **dependency audit** is like bringing in a certified inspector (`cargo-audit`) who has a master list of all known defects in construction materials (the **RustSec Advisory Database**). The inspector checks your building's manifest (`Cargo.lock`) against their list to ensure you haven't used any faulty parts.

## What is `cargo-audit`?

`cargo-audit` is a command-line extension for Cargo that checks your project's dependency tree for crates with known security vulnerabilities. It works by comparing the exact versions of the crates listed in your `Cargo.lock` file against the [RustSec Advisory Database](https://rustsec.org), a community-curated database of security advisories affecting the Rust ecosystem.[^1][^2]

## How to Use `cargo-audit`: A Step-by-Step Guide

Using `cargo-audit` is straightforward and can be easily integrated into any developer's workflow.

### Step 1: Installation

First, you need to install the tool using `cargo`. It's a one-time setup.

```bash
cargo install cargo-audit
```

This command installs `cargo-audit` globally on your system, making it available as a Cargo subcommand.[^3]

### Step 2: Running an Audit

Navigate to the root directory of your Rust project in your terminal and run the following command:

```bash
cargo audit
```

That's it! `cargo-audit` will then perform the following steps :[^4]

1. **Fetch the latest advisories**: It downloads the up-to-date list of vulnerabilities from the RustSec Advisory Database.
2. **Scan `Cargo.lock`**: It reads your `Cargo.lock` file to get the exact versions of every direct and transitive dependency in your project.
3. **Compare and Report**: It compares your dependency list against the vulnerability database and reports any matches.

### Step 3: Interpreting the Output

The output of `cargo-audit` is designed to be clear and actionable.

**If no vulnerabilities are found**, you'll see a reassuring message:

```
Fetching advisory database...
    Updating 'https://github.com/rustsec/advisory-db.git' index
    Loading advisories...
    Scanning Cargo.lock...
Success! No vulnerable packages found
```

**If a vulnerability is found**, the output will be much more detailed:

```
Fetching advisory database...
    Updating 'https://github.com/rustsec/advisory-db.git' index
    Loading advisories...
    Scanning Cargo.lock...

Vulnerability Found!
      Crate: regex
    Version: 1.5.4
      Title: regex has exponential backtracking complexity in its default configuration
         ID: RUSTSEC-2022-0013
        URL: https://rustsec.org/advisories/RUSTSEC-2022-0013
   Solution: Upgrade to >=1.5.5
Dependency tree:
regex 1.5.4
└── my-app 0.1.0 (root)
```

This report gives you all the information you need:

- **Crate**: The name of the vulnerable dependency.
- **Version**: The exact version you are using.
- **Title**: A brief description of the vulnerability.
- **URL**: A link to the full advisory on the RustSec website, which provides detailed information about the vulnerability and its impact.
- **Solution**: The recommended action, which is almost always to upgrade to a newer version of the crate.
- **Dependency Tree**: Shows you how the vulnerable crate was pulled into your project. This is especially useful for transitive dependencies.


### Step 4: Fixing Vulnerabilities

Once a vulnerability is identified, fixing it is usually straightforward.

1. **Upgrade the Crate**: Run `cargo update -p <crate_name> --precise <new_version>` to update the specific crate to the patched version. For the example above:

```bash
cargo update -p regex --precise 1.5.5
```

2. **Re-run `cargo audit`**: Run the audit again to confirm that the vulnerability has been resolved.

For more automated fixing, `cargo-audit` has an experimental `fix` subcommand. You can install it with `cargo install cargo-audit --features=fix` and then run `cargo audit fix` to have it attempt to automatically update your `Cargo.toml` file.[^5]

## Integrating Auditing into Your Workflow

Manually running `cargo audit` is good, but automating it is even better. This ensures that no vulnerable code ever makes it into your main branch or production builds.

- **Continuous Integration (CI)**: The best practice is to add `cargo audit` as a step in your CI pipeline (e.g., GitHub Actions, GitLab CI). If the command finds any vulnerabilities, it will exit with a non-zero status code, failing the build and preventing the code from being merged.[^2]
- **Pre-commit Hooks**: You can also set up a git pre-commit hook that runs `cargo audit` before you're allowed to commit code locally.


## The Limitations of `cargo-audit`

It's important to understand what `cargo-audit` does and does not do.

- It only checks for **publicly known and reported** vulnerabilities. It cannot find zero-day vulnerabilities.
- It audits your dependencies, not **your own application code**. You are still responsible for writing secure code yourself.
- It primarily checks for memory safety issues, but might not catch other types of bugs, like logic flaws.

Therefore, `cargo-audit` should be seen as one essential layer in a comprehensive security strategy, which should also include secure coding practices, code reviews, and potentially other tools like fuzzing (`cargo-fuzz`) or static analysis.

By regularly using `cargo-audit`, you can leverage the strength of the `crates.io` ecosystem with confidence, building more secure and robust applications in Rust.

1. https://blog.rust-lang.org/inside-rust/2019/10/03/Keeping-secure-with-cargo-audit-0.9.html
2. https://rustsec.org
3. https://www.youtube.com/watch?v=w2Co88TzrsQ
4. https://blog.rust-lang.org/inside-rust/2023/09/04/keeping-secure-with-cargo-audit-0.18.html
5. https://blog.rust-lang.org/inside-rust/2020/01/23/Introducing-cargo-audit-fix-and-more.html
6. https://github.com/rust-secure-code/cargo-auditable
7. https://zerotomastery.io/blog/top-cargo-subcommands-for-rust-development/
8. https://web.cs.ucdavis.edu/~cdstanford/doc/2024/CargoScan-draft.pdf
9. https://www.youtube.com/watch?v=MHtn1bLjlNY
10. https://www.pointguardai.com/integrations/cargo-audit
11. https://mozilla.github.io/cargo-vet/performing-audits.html
12. https://crates.io/crates/cargo-audit
13. https://mozilla.github.io/cargo-vet/audit-criteria.html
14. https://docs.rs/cargo-audit/latest/cargo_audit/config/index.html
15. https://internals.rust-lang.org/t/how-to-audit-and-improve-rust-crates-eco-system-for-security-in-general/16699
16. https://www.reddit.com/r/rust/comments/1hci8nb/is_there_a_security_audit_of_the_rust_toolchain/
17. https://luk6xff.github.io/other/safe_secure_rust_book/rust_ecosystem/security_features.html
18. https://corgea.com/Learn/rust-security-best-practices-2025
19. https://www.kodemsecurity.com/resources/addressing-rust-security-vulnerabilities
