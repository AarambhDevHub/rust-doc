+++
title = "String Slices"
description = "Dive into Rust's string slices (&str), how they work, and how to use them effectively for safe and efficient string handling."
date = 2025-08-03
draft = false
weight = 1
+++

# String Slices (&str) in Rust: Comprehensive Documentation

Building on your understanding of borrowing rules and reference lifetimes, let's explore **string slices** (`&str`) - one of Rust's most fundamental and frequently used types. String slices are immutable references to sequences of UTF-8 bytes and are central to efficient string handling in Rust.

## What are String Slices?

A **string slice** (`&str`) is an **immutable reference** to a contiguous sequence of UTF-8 bytes stored somewhere in memory. Unlike `String`, which owns its data on the heap, `&str` is a **borrowed view** into existing string data that can live in various memory locations.

### Core Characteristics

- **Immutable reference** - Cannot modify the underlying string data
- **UTF-8 encoded** - Guarantees valid Unicode text
- **Zero-cost** - No heap allocation, just a pointer and length
- **Versatile** - Can reference data in binary, heap, or stack memory
- **Sized at runtime** - Length determined when the slice is created

```rust
fn main() {
    let owned = String::from("Hello, world!");  // String owns heap data
    let slice = &owned[0..5];                   // &str references part of String
    let literal = "Hello, world!";              // &str pointing to binary data

    println!("Slice: {}", slice);    // "Hello"
    println!("Literal: {}", literal); // "Hello, world!"
}
```

## Memory Layout of String Slices

A string slice is a **fat pointer** consisting of two components:

```rust
// Conceptual representation of &str
struct StrSlice {
    ptr: *const u8,  // Pointer to the first byte
    len: usize,      // Length in bytes (not characters!)
}
```

### Visual Memory Layout

```
String in heap:     H  e  l  l  o  ,     w  o  r  l  d  !
                    ^                    ^
                    |                    |
&str slice:      [ptr]---------------[len: 5]
                 Points to 'H'        5 bytes long

String literal:     H  e  l  l  o  \0  (in binary/static memory)
                    ^
                    |
&str:            [ptr]--------[len: 5]
```

### Memory Layout Example

```rust
fn main() {
    let text = String::from("Hello, Rust!");
    let slice = &text[0..5];  // "Hello"

    println!("String address: {:p}", text.as_ptr());
    println!("Slice address: {:p}", slice.as_ptr());
    println!("Slice length: {}", slice.len());
    println!("Same address: {}", text.as_ptr() == slice.as_ptr());

    // Size comparison
    println!("Size of String: {}", std::mem::size_of::());     // 24 bytes
    println!("Size of &str: {}", std::mem::size_of::());         // 16 bytes
}
```

## Types of String Slices

### 1. String Literals (`&'static str`)

String literals are embedded in the program binary and have a `'static` lifetime:

```rust
fn main() {
    let literal: &'static str = "Hello, world!";  // Lives for entire program
    let another = "This is also a string literal"; // Type inferred as &str

    // String literals are stored in read-only memory
    println!("Literal: {}", literal);
    println!("Address: {:p}", literal.as_ptr());

    // Can be used anywhere a &str is expected
    print_str(literal);
    print_str("Direct literal");
}

fn print_str(s: &str) {
    println!("Got string: {}", s);
}
```

### 2. Slices from String

Creating slices from owned `String` data:

```rust
fn main() {
    let owned = String::from("Hello, beautiful world!");

    // Various slicing operations
    let full_slice = &owned[..];        // Entire string
    let hello = &owned[0..5];           // "Hello"
    let beautiful = &owned[7..16];      // "beautiful"
    let world = &owned[17..22];         // "world"
    let from_index = &owned[7..];       // "beautiful world!"
    let to_index = &owned[..16];        // "Hello, beautiful"

    println!("Full: '{}'", full_slice);
    println!("Hello: '{}'", hello);
    println!("Beautiful: '{}'", beautiful);
    println!("World: '{}'", world);
    println!("From 7: '{}'", from_index);
    println!("To 16: '{}'", to_index);
}
```

### 3. Slices from Arrays and Vectors

String slices can reference data in various collection types:

```rust
fn main() {
    // From byte array (if valid UTF-8)
    let bytes = [72, 101, 108, 108, 111]; // "Hello" in ASCII
    let from_bytes = std::str::from_utf8(&bytes).unwrap();
    println!("From bytes: {}", from_bytes);

    // From Vec (if valid UTF-8)
    let vec_bytes = vec![87, 111, 114, 108, 100]; // "World"
    let from_vec = std::str::from_utf8(&vec_bytes).unwrap();
    println!("From vec: {}", from_vec);

    // From string in vector
    let strings = vec![String::from("first"), String::from("second")];
    let first_slice = &strings[0][..];  // &str from String in Vec
    println!("First slice: {}", first_slice);
}
```

## String Slice Operations

### Basic Operations

```rust
fn main() {
    let text = "Hello, Rust programming!";

    // Length operations
    println!("Length in bytes: {}", text.len());           // 23
    println!("Length in chars: {}", text.chars().count()); // 23
    println!("Is empty: {}", text.is_empty());             // false

    // Character operations
    println!("First char: {:?}", text.chars().next());     // Some('H')
    println!("Last char: {:?}", text.chars().last());      // Some('!')

    // Searching
    println!("Contains 'Rust': {}", text.contains("Rust"));          // true
    println!("Starts with 'Hello': {}", text.starts_with("Hello")); // true
    println!("Ends with '!': {}", text.ends_with("!"));             // true

    // Finding positions
    println!("Position of 'Rust': {:?}", text.find("Rust"));        // Some(7)
    println!("Position of 'Python': {:?}", text.find("Python"));    // None
}
```

### String Slice Iteration

```rust
fn main() {
    let text = "Hello, 🦀!";

    // Iterate over characters
    println!("Characters:");
    for (i, ch) in text.chars().enumerate() {
        println!("  {}: {} (Unicode: U+{:04X})", i, ch, ch as u32);
    }

    // Iterate over bytes
    println!("Bytes:");
    for (i, byte) in text.bytes().enumerate() {
        println!("  {}: {} (0x{:02X})", i, byte, byte);
    }

    // Iterate over char indices (byte positions)
    println!("Char indices:");
    for (byte_index, ch) in text.char_indices() {
        println!("  Byte {}: '{}'", byte_index, ch);
    }
}
```

### String Processing

```rust
fn main() {
    let text = "  Hello, World!  ";

    // Trimming
    let trimmed = text.trim();
    let left_trimmed = text.trim_start();
    let right_trimmed = text.trim_end();

    println!("Original: '{}'", text);         // "  Hello, World!  "
    println!("Trimmed: '{}'", trimmed);       // "Hello, World!"
    println!("Left: '{}'", left_trimmed);     // "Hello, World!  "
    println!("Right: '{}'", right_trimmed);   // "  Hello, World!"

    // Case operations (return new String, not &str)
    let lower = text.to_lowercase();
    let upper = text.to_uppercase();

    println!("Lowercase: {}", lower);
    println!("Uppercase: {}", upper);

    // Splitting
    let sentence = "apple,banana,cherry,date";
    let fruits: Vec = sentence.split(',').collect();
    println!("Fruits: {:?}", fruits);

    // Replacing (returns new String)
    let replaced = text.replace("World", "Rust");
    println!("Replaced: {}", replaced);
}
```

## Unicode and UTF-8 Handling

### Character vs Byte Indexing

**Important**: String indexing in Rust is **byte-based**, not character-based, due to UTF-8 encoding:

```rust
fn main() {
    let text = "Hello, 🦀!";  // The crab emoji is 4 bytes

    println!("Text: {}", text);
    println!("Byte length: {}", text.len());              // 10 bytes
    println!("Character count: {}", text.chars().count()); // 8 characters

    // Safe slicing (must be on character boundaries)
    let hello = &text[0..5];     // "Hello" - safe
    let comma = &text[5..7];     // ", " - safe
    let crab = &text[7..11];     // "🦀" - safe (4 bytes)

    println!("Hello: '{}'", hello);
    println!("Comma: '{}'", comma);
    println!("Crab: '{}'", crab);

    // This would panic - not on character boundary:
    // let invalid = &text[0..8];  // Panic: byte index 8 not a char boundary
}
```

### Safe Character Operations

```rust
fn main() {
    let text = "Héllo, 世界! 🦀";

    // Get nth character safely
    fn get_nth_char(s: &str, n: usize) -> Option {
        s.chars().nth(n)
    }

    // Get substring by character position safely
    fn get_char_substring(s: &str, start: usize, len: usize) -> String {
        s.chars().skip(start).take(len).collect()
    }

    println!("3rd character: {:?}", get_nth_char(text, 2));  // Some('l')
    println!("Characters 7-9: '{}'", get_char_substring(text, 7, 2)); // "世界"

    // Safe iteration
    for (char_index, ch) in text.chars().enumerate() {
        println!("Character {}: '{}' (bytes: {})",
                 char_index, ch, ch.len_utf8());
    }
}
```

### Validation and Conversion

```rust
fn main() {
    // Valid UTF-8 from bytes
    let valid_bytes = [72, 101, 108, 108, 111]; // "Hello"
    match std::str::from_utf8(&valid_bytes) {
        Ok(s) => println!("Valid UTF-8: {}", s),
        Err(e) => println!("Invalid UTF-8: {}", e),
    }

    // Invalid UTF-8 from bytes
    let invalid_bytes = [0xFF, 0xFE]; // Invalid UTF-8 sequence
    match std::str::from_utf8(&invalid_bytes) {
        Ok(s) => println!("Valid UTF-8: {}", s),
        Err(e) => println!("Invalid UTF-8: {}", e),
    }

    // Lossy conversion (replaces invalid sequences with �)
    let lossy = String::from_utf8_lossy(&invalid_bytes);
    println!("Lossy conversion: {}", lossy);
}
```

## String Slice vs String Conversion

### Converting &str to String

```rust
fn main() {
    let slice = "Hello, world!";

    // Multiple ways to convert &str to String
    let string1 = slice.to_string();           // Most common
    let string2 = slice.to_owned();            // Explicit ownership
    let string3 = String::from(slice);         // Explicit constructor
    let string4 = slice.into();               // Using Into trait

    println!("All are equivalent:");
    println!("string1: {}", string1);
    println!("string2: {}", string2);
    println!("string3: {}", string3);
    println!("string4: {}", string4);
}
```

### Converting String to &str

Rust provides automatic **deref coercion** from `String` to `&str`:

```rust
fn takes_str_slice(s: &str) {
    println!("Received: {}", s);
}

fn main() {
    let owned = String::from("Hello, world!");

    // Automatic coercion
    takes_str_slice(&owned);           // &String -> &str
    takes_str_slice(owned.as_str());   // Explicit conversion
    takes_str_slice(&owned[..]);       // Explicit slicing

    // String is still owned and accessible
    println!("Still owned: {}", owned);
}
```

## String Slice Lifetimes

### Lifetime Dependencies

String slices must not outlive the data they reference:

```rust
fn main() {
    let slice: &str;

    {
        let owned = String::from("temporary");
        slice = &owned[..];  // Error: owned doesn't live long enough
    }

    // println!("{}", slice);  // Would be dangling reference
}
```

**Solution**: Ensure proper lifetime relationships:

```rust
fn main() {
    let owned = String::from("long-lived");
    let slice = &owned[..];  // slice lifetime tied to owned

    println!("Slice: {}", slice);  // OK: both in same scope
}
```

### Static Lifetime Slices

String literals have `'static` lifetime:

```rust
fn get_greeting() -> &'static str {
    "Hello, world!"  // String literal lives for entire program
}

fn main() {
    let greeting = get_greeting();
    println!("{}", greeting);  // Always valid
}
```

### Function Parameters and Lifetimes

```rust
// Lifetime elision - compiler infers lifetimes
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(index) => &s[0..index],
        None => s,
    }
}

// Explicit lifetime annotation
fn longer_string(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() { s1 } else { s2 }
}

fn main() {
    let sentence = "Hello beautiful world";
    let first = first_word(sentence);
    println!("First word: {}", first);

    let s1 = "short";
    let s2 = "much longer string";
    let longer = longer_string(s1, s2);
    println!("Longer: {}", longer);
}
```

## Pattern Matching with String Slices

### Basic Pattern Matching

```rust
fn classify_input(input: &str) -> &'static str {
    match input {
        "" => "empty",
        "quit" | "exit" | "q" => "quit command",
        s if s.starts_with("hello") => "greeting",
        s if s.parse::().is_ok() => "number",
        _ => "unknown",
    }
}

fn main() {
    let inputs = ["", "quit", "hello world", "42", "random"];

    for input in &inputs {
        println!("'{}' -> {}", input, classify_input(input));
    }
}
```

### Advanced Pattern Matching

```rust
fn analyze_command(command: &str) -> String {
    let parts: Vec = command.split_whitespace().collect();

    match parts.as_slice() {
        [] => "empty command".to_string(),
        ["help"] => "showing help".to_string(),
        ["echo", rest @ ..] => format!("echoing: {}", rest.join(" ")),
        ["set", key, value] => format!("setting {} = {}", key, value),
        ["get", key] => format!("getting value for {}", key),
        [cmd, ..] => format!("unknown command: {}", cmd),
    }
}

fn main() {
    let commands = [
        "",
        "help",
        "echo hello world",
        "set name Alice",
        "get name",
        "unknown command with args",
    ];

    for cmd in &commands {
        println!("'{}' -> {}", cmd, analyze_command(cmd));
    }
}
```

## Common String Slice Patterns

### The Parser Pattern

```rust
struct Parser {
    input: &'a str,
    position: usize,
}

impl Parser {
    fn new(input: &'a str) -> Self {
        Parser { input, position: 0 }
    }

    fn peek(&self) -> Option {
        self.input[self.position..].chars().next()
    }

    fn advance(&mut self) -> Option {
        let mut chars = self.input[self.position..].chars();
        if let Some(ch) = chars.next() {
            self.position += ch.len_utf8();
            Some(ch)
        } else {
            None
        }
    }

    fn parse_word(&mut self) -> Option {
        let start = self.position;

        while let Some(ch) = self.peek() {
            if ch.is_alphabetic() {
                self.advance();
            } else {
                break;
            }
        }

        if self.position > start {
            Some(&self.input[start..self.position])
        } else {
            None
        }
    }

    fn skip_whitespace(&mut self) {
        while let Some(ch) = self.peek() {
            if ch.is_whitespace() {
                self.advance();
            } else {
                break;
            }
        }
    }
}

fn main() {
    let mut parser = Parser::new("hello world rust programming");

    while parser.peek().is_some() {
        parser.skip_whitespace();
        if let Some(word) = parser.parse_word() {
            println!("Word: {}", word);
        }
    }
}
```

### The Configuration Parser Pattern

```rust
fn parse_config(content: &str) -> std::collections::HashMap {
    let mut config = std::collections::HashMap::new();

    for line in content.lines() {
        let line = line.trim();

        // Skip empty lines and comments
        if line.is_empty() || line.starts_with('#') {
            continue;
        }

        // Parse key=value pairs
        if let Some(eq_pos) = line.find('=') {
            let key = line[..eq_pos].trim();
            let value = line[eq_pos + 1..].trim();
            config.insert(key, value);
        }
    }

    config
}

fn main() {
    let config_content = r#"
# Database configuration
host=localhost
port=5432
database=myapp

# Cache settings
cache_size=1000
cache_ttl=3600
"#;

    let config = parse_config(config_content);

    for (key, value) in &config {
        println!("{} = {}", key, value);
    }
}
```

### The Template Processing Pattern

```rust
fn process_template(template: &str, vars: &std::collections::HashMap) -> String {
    let mut result = String::with_capacity(template.len());
    let mut chars = template.chars().peekable();

    while let Some(ch) = chars.next() {
        if ch == '{' && chars.peek() == Some(&'{') {
            chars.next(); // consume second '{'

            // Find the closing '}}'
            let mut var_name = String::new();
            let mut found_close = false;

            while let Some(ch) = chars.next() {
                if ch == '}' && chars.peek() == Some(&'}') {
                    chars.next(); // consume second '}'
                    found_close = true;
                    break;
                }
                var_name.push(ch);
            }

            if found_close {
                if let Some(value) = vars.get(var_name.as_str()) {
                    result.push_str(value);
                } else {
                    result.push_str(&format!("{{{{{}}}}}", var_name)); // Keep original
                }
            } else {
                result.push_str("{{");
                result.push_str(&var_name);
            }
        } else {
            result.push(ch);
        }
    }

    result
}

fn main() {
    let template = "Hello {{name}}, welcome to {{app}}! Your score is {{score}}.";

    let mut vars = std::collections::HashMap::new();
    vars.insert("name", "Alice");
    vars.insert("app", "MyApp");
    vars.insert("score", "95");

    let result = process_template(template, &vars);
    println!("Result: {}", result);
}
```

## Performance Considerations

### Zero-Copy String Processing

```rust
use std::time::Instant;

fn process_with_copies(text: &str) -> Vec {
    text.split(',')
        .map(|s| s.trim().to_string())  // Creates owned Strings
        .collect()
}

fn process_with_slices(text: &str) -> Vec {
    text.split(',')
        .map(|s| s.trim())  // Returns string slices
        .collect()
}

fn main() {
    let data = "apple, banana, cherry, date, elderberry, fig, grape";

    // Benchmark copying approach
    let start = Instant::now();
    for _ in 0..100_000 {
        let _result = process_with_copies(data);
    }
    let copy_time = start.elapsed();

    // Benchmark slice approach
    let start = Instant::now();
    for _ in 0..100_000 {
        let _result = process_with_slices(data);
    }
    let slice_time = start.elapsed();

    println!("Copy approach: {:?}", copy_time);
    println!("Slice approach: {:?}", slice_time);
    println!("Slice is {:.1}x faster",
             copy_time.as_nanos() as f64 / slice_time.as_nanos() as f64);
}
```

### Memory Usage Comparison

```rust
fn main() {
    let large_text = "word ".repeat(100_000);  // ~500KB string

    // Memory-efficient: slice processing
    let word_count = large_text.split_whitespace().count();
    println!("Word count: {}", word_count);

    // Memory-expensive: collecting all words
    let all_words: Vec = large_text
        .split_whitespace()
        .map(|s| s.to_string())
        .collect();

    println!("Collected {} words", all_words.len());

    // Show memory usage difference
    let slice_size = std::mem::size_of_val(&large_text);
    let vec_size = std::mem::size_of_val(&all_words) +
                   all_words.iter().map(|s| s.capacity()).sum::();

    println!("Original string: {} bytes", slice_size);
    println!("Collected words: {} bytes", vec_size);
    println!("Memory overhead: {:.1}x", vec_size as f64 / slice_size as f64);
}
```

## Best Practices for String Slices

### API Design Guidelines

```rust
// Good: Accept &str for maximum flexibility
fn process_text(text: &str) -> usize {
    text.split_whitespace().count()
}

// Less flexible: Accepts only String
fn process_string(text: String) -> usize {
    text.split_whitespace().count()
}

// Good: Return &str when referencing input data
fn extract_domain(email: &str) -> Option {
    email.find('@').map(|pos| &email[pos + 1..])
}

// Good: Return String when creating new data
fn format_greeting(name: &str) -> String {
    format!("Hello, {}!", name)
}

fn main() {
    let text = String::from("hello world rust");
    let literal = "hello world rust";

    // Both work with &str parameter
    println!("String: {}", process_text(&text));
    println!("Literal: {}", process_text(literal));

    let email = "user@example.com";
    if let Some(domain) = extract_domain(email) {
        println!("Domain: {}", domain);
    }

    let greeting = format_greeting("Alice");
    println!("{}", greeting);
}
```

### Error Handling with String Slices

```rust
use std::str::Utf8Error;

fn safe_slice(s: &str, start: usize, end: usize) -> Result {
    if start > s.len() || end > s.len() || start > end {
        return Err("Invalid slice bounds");
    }

    // Check that we're on character boundaries
    if !s.is_char_boundary(start) || !s.is_char_boundary(end) {
        return Err("Slice bounds not on character boundaries");
    }

    Ok(&s[start..end])
}

fn parse_utf8_safely(bytes: &[u8]) -> Result {
    std::str::from_utf8(bytes)
}

fn main() {
    let text = "Hello, 🦀 world!";

    match safe_slice(text, 0, 5) {
        Ok(slice) => println!("Safe slice: '{}'", slice),
        Err(e) => println!("Error: {}", e),
    }

    // This would error due to character boundary
    match safe_slice(text, 0, 8) {  // Cuts through emoji
        Ok(slice) => println!("Safe slice: '{}'", slice),
        Err(e) => println!("Error: {}", e),
    }

    let bytes = b"Hello, world!";
    match parse_utf8_safely(bytes) {
        Ok(s) => println!("Valid UTF-8: {}", s),
        Err(e) => println!("Invalid UTF-8: {}", e),
    }
}
```

### Performance Optimization

```rust
// Efficient string processing with slices
fn count_words_in_lines(text: &str) -> Vec {
    text.lines()                    // Iterator over &str
        .map(|line| {
            line.split_whitespace() // Iterator over &str
                .count()            // Count without allocation
        })
        .collect()                  // Only collect the counts
}

// Efficient filtering with slices
fn filter_long_words(text: &str, min_length: usize) -> Vec {
    text.split_whitespace()
        .filter(|word| word.len() >= min_length)
        .collect()
}

// Efficient text transformation
fn process_csv_line(line: &str) -> Vec {
    line.split(',')
        .map(|field| field.trim())
        .collect()
}

fn main() {
    let text = "Hello world\nRust programming\nString slices are efficient";
    let word_counts = count_words_in_lines(text);
    println!("Word counts per line: {:?}", word_counts);

    let long_words = filter_long_words(text, 5);
    println!("Long words: {:?}", long_words);

    let csv = "name, age, city, country";
    let fields = process_csv_line(csv);
    println!("CSV fields: {:?}", fields);
}
```

## Common Pitfalls and Solutions

### String Indexing Panics

```rust
fn main() {
    let text = "Hello, 🦀!";

    // Dangerous - can panic on invalid boundaries
    // let slice = &text[0..8];  // Panic: not a char boundary

    // Safe alternatives

    // 1. Use character-based operations
    let first_n_chars: String = text.chars().take(7).collect();
    println!("First 7 chars: {}", first_n_chars);

    // 2. Check boundaries
    fn safe_slice_to(s: &str, end: usize) -> &str {
        if end  0 && !s.is_char_boundary(valid_end) {
                valid_end -= 1;
            }
            &s[..valid_end]
        }
    }

    let safe = safe_slice_to(text, 8);
    println!("Safe slice: '{}'", safe);

    // 3. Use get() for bounds checking
    if let Some(slice) = text.get(0..7) {
        println!("Get slice: '{}'", slice);
    }
}
```

### Lifetime Issues with String Slices

```rust
// This won't compile - returning reference to local data
// fn create_slice() -> &str {
//     let s = String::from("local");
//     &s[..]  // Error: s dropped at end of function
// }

// Solutions:

// 1. Return owned String
fn create_owned() -> String {
    String::from("owned")
}

// 2. Use string literals for static data
fn get_static_slice() -> &'static str {
    "static data"
}

// 3. Accept and return slices with same lifetime
fn process_slice(input: &str) -> &str {
    input.trim()  // Returns slice with same lifetime as input
}

fn main() {
    let owned = create_owned();
    let static_slice = get_static_slice();
    let processed = process_slice("  hello  ");

    println!("Owned: {}", owned);
    println!("Static: {}", static_slice);
    println!("Processed: '{}'", processed);
}
```

### Mixing String and &str

```rust
fn main() {
    // Problem: inefficient mixing of String and &str
    fn inefficient_concat(parts: Vec) -> String {
        let mut result = String::new();
        for part in parts {
            result = result + part;  // Creates new String each time
        }
        result
    }

    // Solution: use efficient string building
    fn efficient_concat(parts: &[&str]) -> String {
        parts.join("")  // Single allocation
    }

    // Or use format! for simple cases
    fn format_name(first: &str, last: &str) -> String {
        format!("{} {}", first, last)  // Efficient formatting
    }

    let parts = vec!["Hello", ", ", "world", "!"];
    let result1 = inefficient_concat(parts.clone());
    let result2 = efficient_concat(&parts);
    let name = format_name("Alice", "Smith");

    println!("Result 1: {}", result1);
    println!("Result 2: {}", result2);
    println!("Name: {}", name);
}
```

## Summary and Key Takeaways

### **Core Concepts**

**String slices** (`&str`) provide:
- **Immutable references** to UTF-8 encoded text data
- **Zero-cost abstractions** with no heap allocation
- **Flexible data sources** - can reference literals, heap data, or other sources
- **Memory safety** through Rust's borrowing system

### **Key Characteristics**

- **Fat pointer** containing pointer and length
- **UTF-8 guarantees** ensure valid Unicode text
- **Byte-indexed** but must respect character boundaries
- **Lifetime-dependent** on the data they reference

### **Best Practices**

```rust
// Good: Accept &str for flexibility
fn process(text: &str) -> usize { text.len() }

// Good: Return &str when referencing input
fn get_extension(filename: &str) -> Option {
    filename.rfind('.').map(|pos| &filename[pos + 1..])
}

// Good: Return String when creating new data
fn create_greeting(name: &str) -> String {
    format!("Hello, {}!", name)
}

// Avoid: Unnecessary String parameters
fn less_flexible(text: String) -> usize { text.len() }
```

### **Performance Benefits**

- **Zero-copy operations** for most string processing
- **Memory efficiency** - no unnecessary allocations
- **Cache-friendly** - contiguous memory access
- **Optimal for parsing** and text analysis tasks

### **Memory Safety Guarantees**

- **No buffer overflows** - bounds checking prevents out-of-bounds access
- **No dangling pointers** - lifetime system ensures valid references
- **UTF-8 validation** - guarantees valid Unicode text
- **Thread-safe sharing** - immutable data can be safely shared

Understanding string slices is fundamental to effective Rust programming. **They enable efficient, safe text processing while maintaining zero-cost abstractions**. Use string slices for most text processing tasks, and convert to owned `String` only when you need to modify the data or transfer ownership. The combination of performance, safety, and expressiveness makes string slices one of Rust's most powerful features for text manipulation.
