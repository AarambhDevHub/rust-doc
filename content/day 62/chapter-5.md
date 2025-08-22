---
title: "Working with Archives in Rust"
description: "Explore how to create and extract archive formats like ZIP and TAR using Rust libraries."
date: 2025-11-02
weight: 5
---

# Working with Archives in Rust: Creating and Extracting ZIP and TAR Files

Rust makes it possible to handle common archive formats like ZIP and TAR (including compressed variants such as `.tar.gz`) using well-established crates. Let's explore how to **create** and **extract** these archives through clear, hands-on examples suitable for beginners.

***

## Getting Started: What Are Archives?

- A **TAR archive** bundles many files and directories into a single file (no compression by itself, often compressed with gzip as `.tar.gz`).
- A **ZIP archive** both bundles and compresses files, and supports random-access extraction.

Think of TAR as a "container" and ZIP as a "container that also compresses its contents and has an index".

***

## 1. Creating and Extracting TAR Archives

### Key Crates

- [`tar`](https://docs.rs/tar): For reading/writing TAR archives.
- [`flate2`](https://docs.rs/flate2): For compression (gzip).


### Example: Creating a `.tar.gz` Archive

Let's compress a directory (e.g., `logs/`) into `archive.tar.gz`:

```rust
use std::fs::File;
use flate2::Compression;
use flate2::write::GzEncoder;
use tar::Builder;

fn main() -> std::io::Result<()> {
    // Create the output file
    let tar_gz = File::create("archive.tar.gz")?;
    let enc = GzEncoder::new(tar_gz, Compression::default());
    let mut tar = Builder::new(enc);

    // Recursively add the directory's contents
    tar.append_dir_all("backup/logs", "logs")?;

    tar.finish()?; // Finalize the archive
    Ok(())
}
```

- `tar.append_dir_all("backup/logs", "logs")?;` adds all the `logs/` directory contents under the path `backup/logs` in the archive.[^1]
- `GzEncoder` wraps the TAR "stream" to add gzip compression.


### Example: Extracting a `.tar.gz` Archive

Let's extract `archive.tar.gz` into the current directory.

```rust
use std::fs::File;
use flate2::read::GzDecoder;
use tar::Archive;

fn main() -> std::io::Result<()> {
    let tar_gz = File::open("archive.tar.gz")?;
    let tar = GzDecoder::new(tar_gz); // Decompress file
    let mut archive = Archive::new(tar);
    archive.unpack(".")?; // Unpacks all files
    Ok(())
}
```

- `archive.unpack(".")?;` extracts all files into the current dir.[^1]

***

## 2. Creating and Extracting ZIP Archives

### Key Crate

- [`zip`](https://docs.rs/zip): For reading/writing ZIP files.


### Example: Creating a ZIP Archive

Let's add a couple of files to a new `archive.zip`:

```rust
use std::fs::File;
use zip::{ZipWriter, write::FileOptions};
use std::io::{Write, Seek};

fn main() -> zip::result::ZipResult<()> {
    let path = std::path::Path::new("archive.zip");
    let file = File::create(&path)?;
    let mut zip = ZipWriter::new(file);

    let options = FileOptions::default();

    // Add a file called "hello.txt" with text content
    zip.start_file("hello.txt", options)?;
    zip.write_all(b"Hello, world!")?;

    // Add an existing file as "backup/config.toml"
    let mut f = File::open("config.toml")?;
    zip.start_file("backup/config.toml", options)?;
    std::io::copy(&mut f, &mut zip)?;

    zip.finish()?;
    Ok(())
}
```

- `zip.start_file(...)?;` starts a new file in the archive; `write_all` provides content.


### Example: Extracting a ZIP Archive

To extract all files from `archive.zip`:

```rust
use std::fs::File;
use zip::read::ZipArchive;

fn main() -> zip::result::ZipResult<()> {
    let file = File::open("archive.zip")?;
    let mut archive = ZipArchive::new(file)?;

    for i in 0..archive.len() {
        let mut file = archive.by_index(i)?;
        let outpath = file.enclosed_name().unwrap();

        if file.is_dir() {
            std::fs::create_dir_all(&outpath)?;
        } else {
            if let Some(p) = outpath.parent() {
                if !p.exists() {
                    std::fs::create_dir_all(p)?;
                }
            }
            let mut outfile = File::create(&outpath)?;
            std::io::copy(&mut file, &mut outfile)?;
        }
    }
    Ok(())
}
```

This script recreates the ZIP'd files and folders in your working directory.

***

## Additional Tips and Learnings

- **To extract or manipulate a specific file within an archive (without saving to disk)**, use the uncompressed file's reader returned by the library and read its contents as bytes or string.[^2]
- **For archives compressed with other algorithms (e.g., `.tar.bz2`, `.tar.xz`), use appropriate codecs from other crates in combination with `tar`.**

***

## Summary Table: Crates for Archiving

| **Format** | **Crate** | **Create** | **Extract** | **Compression** |
| :-- | :-- | :-- | :-- | :-- |
| TAR | `tar` | Yes | Yes | Use `flate2` for gzip |
| TAR.GZ | `tar`, `flate2` | Yes | Yes | gzip (GzEncoder/Decoder) |
| ZIP | `zip` | Yes | Yes | Built-in |


***

## How to Choose

- Use **tar + flate2** for `tar`/`tar.gz` (great for POSIX/Unix-style).
- Use **zip** for Windows compatibility or cases where random file access in the archive matters.
- Always handle errors (`Result`) and check the crate documentation for more advanced features (like encryption or seeking).

Armed with these patterns, you can create backup tools, installers, and deliver self-contained app packages in Rust with just a few lines of code!.[^3]

1. https://rust-lang-nursery.github.io/rust-cookbook/compression/tar.html
2. https://users.rust-lang.org/t/extract-contents-of-single-file-from-a-tar-archive/69046
3. https://users.rust-lang.org/t/which-library-is-most-commonly-used-to-read-and-write-to-archive-files/129644
4. https://rust.code-maven.com/unzip-file
5. https://stackoverflow.com/questions/73375518/how-to-create-an-archive-tar-7z-zip-on-the-fly
6. https://www.warp.dev/terminus/unzip-a-tar-gz-file
7. https://docs.rs/tar
8. https://docs.rs/zarchive
9. https://www.reddit.com/r/rust/comments/k4dpd7/is_there_a_rust_library_like_the_7zip_suite_that/
