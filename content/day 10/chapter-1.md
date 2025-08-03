+++
title = "Complex Ownership Scenarios"
description = "Understand complex ownership patterns in Rust, including multiple owners, interior mutability, and shared vs. exclusive access through smart pointers."
date = 2025-08-03
draft = false
weight = 1
+++

# Complex Ownership Scenarios in Rust: Comprehensive Documentation

Complex ownership scenarios in Rust go beyond basic ownership rules and involve intricate interactions between different language features. These scenarios challenge developers to think deeply about data flow, lifetimes, and resource management. This comprehensive guide explores advanced ownership patterns that occur in real-world Rust applications.

## 1. Ownership in Structs and Enums

### Complex Struct Ownership Patterns

#### Partial Moves from Structs

One of the most common complex scenarios involves **partial moves** where individual fields are moved out of a struct, leaving the struct in a partially initialized state[1]:

```rust
struct ComplexData {
    name: String,
    values: Vec,
    metadata: Option,
    id: u64,
}

fn demonstrate_partial_moves() {
    let data = ComplexData {
        name: String::from("dataset"),
        values: vec![1, 2, 3, 4, 5],
        metadata: Some(String::from("important")),
        id: 42,
    };

    // Partial move: extract name and values
    let extracted_name = data.name;       // Move name field
    let extracted_values = data.values;   // Move values field

    // These fields are still accessible (Copy types or unchanged)
    println!("ID: {}", data.id);
    println!("Metadata: {:?}", data.metadata);

    // ERROR: Cannot access moved fields
    // println!("Name: {}", data.name);      // Compile error!
    // println!("Values: {:?}", data.values); // Compile error!

    // ERROR: Cannot use data as a whole after partial move
    // let full_data = data; // Compile error!
}
```

#### Ownership Transfer in Nested Structures

Complex nested structures create intricate ownership relationships:

```rust
#[derive(Debug)]
struct Document {
    title: String,
    sections: Vec,
    metadata: DocumentMetadata,
}

#[derive(Debug)]
struct Section {
    heading: String,
    content: String,
    subsections: Vec, // Recursive structure
}

#[derive(Debug)]
struct DocumentMetadata {
    author: String,
    created: chrono::DateTime,
    tags: Vec,
}

impl Document {
    // Method that takes ownership of the entire document
    fn into_archive(self) -> Archive {
        Archive {
            original_title: self.title,
            sections_count: self.sections.len(),
            archive_data: self.serialize(),
        }
    }

    // Method that extracts specific sections by moving them
    fn extract_sections_by_pattern(mut self, pattern: &str) -> (Vec, Document) {
        let mut extracted = Vec::new();
        let mut remaining = Vec::new();

        for section in self.sections {
            if section.heading.contains(pattern) {
                extracted.push(section); // Move matching sections
            } else {
                remaining.push(section); // Keep non-matching sections
            }
        }

        self.sections = remaining;
        (extracted, self)
    }

    fn serialize(&self) -> Vec {
        // Simplified serialization
        format!("{:?}", self).into_bytes()
    }
}

struct Archive {
    original_title: String,
    sections_count: usize,
    archive_data: Vec,
}

fn complex_document_ownership() {
    let document = Document {
        title: String::from("Rust Programming Guide"),
        sections: vec![
            Section {
                heading: String::from("Introduction"),
                content: String::from("Welcome to Rust"),
                subsections: vec![],
            },
            Section {
                heading: String::from("Advanced Ownership"),
                content: String::from("Complex scenarios"),
                subsections: vec![
                    Section {
                        heading: String::from("Partial Moves"),
                        content: String::from("When fields are moved individually"),
                        subsections: vec![],
                    }
                ],
            },
        ],
        metadata: DocumentMetadata {
            author: String::from("Rust Developer"),
            created: chrono::Utc::now(),
            tags: vec![String::from("programming"), String::from("systems")],
        },
    };

    // Complex ownership transfer: extract specific sections
    let (advanced_sections, remaining_doc) = document.extract_sections_by_pattern("Advanced");

    println!("Extracted {} advanced sections", advanced_sections.len());
    println!("Remaining document has {} sections", remaining_doc.sections.len());

    // Convert remaining document to archive (consumes ownership)
    let archive = remaining_doc.into_archive();
    println!("Archived document: {}", archive.original_title);
}
```

### Enum Ownership Complexities

#### Pattern Matching and Ownership Transfer

Enums present unique ownership challenges, especially when pattern matching involves moving data[2]:

```rust
#[derive(Debug)]
enum Message {
    Text(String),
    Image { data: Vec, format: String },
    Video { data: Vec, duration: u32, codec: String },
    Compound(Vec), // Recursive enum
}

impl Message {
    // Complex pattern matching with ownership transfer
    fn process_message(self) -> ProcessingResult {
        match self {
            // Simple move for Text
            Message::Text(content) => {
                ProcessingResult::TextProcessed {
                    word_count: content.split_whitespace().count(),
                    content, // content moved into result
                }
            }

            // Destructuring with partial moves
            Message::Image { data, format } => {
                let size = data.len();
                ProcessingResult::ImageProcessed {
                    size,
                    format, // format moved
                    thumbnail: Self::generate_thumbnail(data), // data moved
                }
            }

            // Complex nested processing
            Message::Video { data, duration, codec } => {
                let processed_data = Self::compress_video(data, &codec);
                ProcessingResult::VideoProcessed {
                    original_size: processed_data.len(),
                    duration,
                    codec, // codec moved
                    compressed_data: processed_data,
                }
            }

            // Recursive processing with ownership transfer
            Message::Compound(messages) => {
                let mut results = Vec::new();
                for message in messages { // Each message moved in iteration
                    results.push(message.process_message());
                }
                ProcessingResult::CompoundProcessed(results)
            }
        }
    }

    fn generate_thumbnail(data: Vec) -> Vec {
        // Simulate thumbnail generation
        data.into_iter().take(100).collect()
    }

    fn compress_video(data: Vec, codec: &str) -> Vec {
        // Simulate video compression
        match codec {
            "h264" => data.into_iter().take(data.len() / 2).collect(),
            _ => data,
        }
    }
}

#[derive(Debug)]
enum ProcessingResult {
    TextProcessed { word_count: usize, content: String },
    ImageProcessed { size: usize, format: String, thumbnail: Vec },
    VideoProcessed { original_size: usize, duration: u32, codec: String, compressed_data: Vec },
    CompoundProcessed(Vec),
}

fn complex_enum_ownership() {
    let messages = vec![
        Message::Text(String::from("Hello, world!")),
        Message::Image {
            data: vec![0u8; 1000],
            format: String::from("PNG"),
        },
        Message::Compound(vec![
            Message::Text(String::from("Nested message")),
            Message::Video {
                data: vec![0u8; 5000],
                duration: 120,
                codec: String::from("h264"),
            },
        ]),
    ];

    for message in messages { // Each message moved in iteration
        let result = message.process_message();
        println!("Processed: {:?}", result);
    }
}
```

## 2. Ownership with Pattern Matching

### Advanced Destructuring Patterns

Pattern matching can create complex ownership scenarios when destructuring data[3]:

```rust
struct Configuration {
    database_url: String,
    cache_settings: CacheConfig,
    feature_flags: Vec,
    debug_mode: bool,
}

struct CacheConfig {
    max_size: usize,
    ttl: u64,
    storage_type: StorageType,
}

enum StorageType {
    Memory,
    Disk { path: String },
    Distributed { nodes: Vec },
}

fn complex_pattern_matching_ownership() {
    let config = Configuration {
        database_url: String::from("postgresql://localhost:5432/mydb"),
        cache_settings: CacheConfig {
            max_size: 1000,
            ttl: 3600,
            storage_type: StorageType::Distributed {
                nodes: vec![
                    String::from("node1:6379"),
                    String::from("node2:6379"),
                    String::from("node3:6379"),
                ],
            },
        },
        feature_flags: vec![
            String::from("new_ui"),
            String::from("experimental_api"),
        ],
        debug_mode: true,
    };

    // Complex destructuring with ownership transfer
    let Configuration {
        database_url,    // Moved
        cache_settings: CacheConfig {
            max_size,    // Copied (primitive)
            ttl,         // Copied (primitive)
            storage_type, // Moved
        },
        feature_flags,   // Moved
        debug_mode,      // Copied (primitive)
    } = config;

    // config is now invalid due to partial moves

    // Pattern match on moved storage_type
    match storage_type {
        StorageType::Memory => {
            println!("Using memory storage");
        }
        StorageType::Disk { path } => {
            println!("Using disk storage at: {}", path);
        }
        StorageType::Distributed { nodes } => {
            println!("Using distributed storage with {} nodes", nodes.len());
            for (i, node) in nodes.into_iter().enumerate() {
                println!("  Node {}: {}", i + 1, node);
            }
        }
    }

    println!("Database: {}", database_url);
    println!("Cache max size: {}", max_size);
    println!("Feature flags: {:?}", feature_flags);
    println!("Debug mode: {}", debug_mode);
}

// Alternative approach using references to avoid moves
fn pattern_matching_with_references() {
    let config = Configuration {
        database_url: String::from("postgresql://localhost:5432/mydb"),
        cache_settings: CacheConfig {
            max_size: 1000,
            ttl: 3600,
            storage_type: StorageType::Memory,
        },
        feature_flags: vec![String::from("feature1")],
        debug_mode: false,
    };

    // Pattern match with references to avoid moving
    match &config.cache_settings.storage_type {
        StorageType::Memory => {
            println!("Memory storage configured");
        }
        StorageType::Disk { path } => {
            println!("Disk storage at: {}", path);
        }
        StorageType::Distributed { nodes } => {
            println!("Distributed storage with {} nodes", nodes.len());
        }
    }

    // config is still valid because we used references
    println!("Config still available: {}", config.database_url);
}
```

## 3. Closure Ownership Scenarios

### Capture Modes and Ownership Transfer

Closures create complex ownership scenarios through their capture behavior[4][5]:

```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn complex_closure_ownership() {
    let data = vec![1, 2, 3, 4, 5];
    let multiplier = 10;
    let counter = Arc::new(Mutex::new(0));

    // Different capture modes demonstrate ownership complexity

    // 1. Closure capturing by reference (borrowing)
    let borrow_closure = || {
        println!("Data length: {}", data.len()); // Borrows data
        println!("Multiplier: {}", multiplier);  // Copies multiplier (i32 is Copy)
    };

    borrow_closure();
    println!("Data still available: {:?}", data); // data still owned by main

    // 2. Closure capturing by move
    let move_closure = move || {
        println!("Moved data: {:?}", data);       // Takes ownership of data
        println!("Moved multiplier: {}", multiplier); // Copies multiplier

        // Can modify counter through Arc
        let mut count = counter.lock().unwrap();
        *count += 1;
    };

    // data is no longer accessible here due to move
    // println!("Data: {:?}", data); // Compile error!

    move_closure();
    println!("Counter value: {}", *counter.lock().unwrap());
}

// Complex closure scenario with thread spawning
fn closure_with_threads() {
    let shared_data = Arc::new(Mutex::new(vec![0; 10]));
    let processor_config = ProcessorConfig {
        batch_size: 5,
        timeout_ms: 1000,
    };

    let handles: Vec = (0..3)
        .map(|thread_id| {
            let data_clone = shared_data.clone();
            let config = processor_config.clone(); // Assume Clone is implemented

            thread::spawn(move || {
                // Complex ownership: data_clone and config moved into thread
                let processor = DataProcessor::new(thread_id, config);

                for i in 0..5 {
                    let mut data = data_clone.lock().unwrap();
                    processor.process_item(&mut data, i);
                }

                println!("Thread {} completed", thread_id);
            })
        })
        .collect();

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final data: {:?}", *shared_data.lock().unwrap());
}

#[derive(Clone)]
struct ProcessorConfig {
    batch_size: usize,
    timeout_ms: u64,
}

struct DataProcessor {
    thread_id: usize,
    config: ProcessorConfig,
}

impl DataProcessor {
    fn new(thread_id: usize, config: ProcessorConfig) -> Self {
        DataProcessor { thread_id, config }
    }

    fn process_item(&self, data: &mut Vec, item: usize) {
        if item  {
    name: String,
    transformer: Box T + Send + Sync>,
}

impl DataTransformer {
    fn new(name: String, transformer: F) -> Self
    where
        F: Fn(T) -> T + Send + Sync + 'static,
    {
        DataTransformer {
            name,
            transformer: Box::new(transformer),
        }
    }

    fn transform(&self, value: T) -> T {
        (self.transformer)(value)
    }
}

fn higher_order_ownership() {
    let mut transformers: Vec> = Vec::new();

    // Create transformers with different capture patterns
    let multiplier = 5;
    let base_value = 100;

    // Transformer that captures by copy
    transformers.push(DataTransformer::new(
        "multiply".to_string(),
        move |x| x * multiplier, // multiplier copied into closure
    ));

    // Transformer that captures by move
    let bonus_map = HashMap::from([
        (1, 10),
        (2, 20),
        (3, 30),
    ]);

    transformers.push(DataTransformer::new(
        "bonus".to_string(),
        move |x| {
            let bonus = bonus_map.get(&x).unwrap_or(&0);
            x + bonus + base_value // bonus_map and base_value moved into closure
        },
    ));

    // Complex processing pipeline
    let values = vec![1, 2, 3, 4, 5];

    for value in values {
        let mut result = value;
        for transformer in &transformers {
            result = transformer.transform(result);
            println!("After {}: {}", transformer.name, result);
        }
        println!("Final result for {}: {}", value, result);
    }
}
```

## 4. Self-Referential Structures and Ownership

### Pin and Self-References

Self-referential structures create some of the most complex ownership scenarios[6]:

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

struct SelfReferential {
    data: String,
    pointer: *const String, // Raw pointer to avoid borrow checker issues
    _pin: PhantomPinned,
}

impl SelfReferential {
    fn new(data: String) -> Pin> {
        let mut boxed = Box::pin(SelfReferential {
            data,
            pointer: std::ptr::null(),
            _pin: PhantomPinned,
        });

        unsafe {
            let mut_ref: Pin = Pin::as_mut(&mut boxed);
            let ptr = &mut_ref.data as *const String;
            Pin::get_unchecked_mut(mut_ref).pointer = ptr;
        }

        boxed
    }

    fn get_data(&self) -> &str {
        &self.data
    }

    fn get_pointer_data(&self) -> &str {
        unsafe { &*self.pointer }
    }
}

// Complex scenario: Linked list with self-references
struct Node {
    value: i32,
    next: Option>,
}

struct LinkedList {
    head: Option>,
    current: *mut Node, // Raw pointer for current position
}

impl LinkedList {
    fn new() -> Self {
        LinkedList {
            head: None,
            current: std::ptr::null_mut(),
        }
    }

    fn push(&mut self, value: i32) {
        let new_node = Box::new(Node {
            value,
            next: self.head.take(),
        });

        let new_node_ptr = &*new_node as *const Node as *mut Node;
        self.head = Some(new_node);
        self.current = new_node_ptr;
    }

    fn current_value(&self) -> Option {
        if self.current.is_null() {
            None
        } else {
            unsafe { Some((*self.current).value) }
        }
    }
}

fn self_referential_ownership() {
    // Pin-based self-reference
    let pinned = SelfReferential::new("Hello, self-reference!".to_string());
    println!("Data: {}", pinned.get_data());
    println!("Pointer data: {}", pinned.get_pointer_data());

    // Linked list with raw pointers
    let mut list = LinkedList::new();
    list.push(1);
    list.push(2);
    list.push(3);

    println!("Current value: {:?}", list.current_value());
}
```

## 5. Ownership Across Thread Boundaries

### Send and Sync Constraints

Complex ownership scenarios emerge when transferring data across threads[7]:

```rust
use std::thread;
use std::sync::{Arc, Mutex, mpsc};
use std::time::Duration;

// Complex data structure with mixed Send/Sync requirements
struct ComplexProcessor {
    id: usize,
    data: Arc>>,
    sender: mpsc::Sender,
    config: ProcessingConfig,
}

#[derive(Clone)]
struct ProcessingConfig {
    batch_size: usize,
    delay_ms: u64,
    error_threshold: f64,
}

struct ProcessingItem {
    id: usize,
    data: Vec,
    priority: Priority,
}

#[derive(Debug, Clone)]
enum Priority {
    Low,
    Medium,
    High,
    Critical,
}

#[derive(Debug)]
struct ProcessingResult {
    processor_id: usize,
    item_id: usize,
    success: bool,
    duration_ms: u128,
}

impl ComplexProcessor {
    fn new(
        id: usize,
        data: Arc>>,
        sender: mpsc::Sender,
        config: ProcessingConfig,
    ) -> Self {
        ComplexProcessor {
            id,
            data,
            sender,
            config,
        }
    }

    fn run(self) -> thread::JoinHandle {
        thread::spawn(move || {
            loop {
                let item = {
                    let mut data = self.data.lock().unwrap();
                    data.pop()
                };

                match item {
                    Some(item) => {
                        let start = std::time::Instant::now();
                        let success = self.process_item(&item);
                        let duration = start.elapsed();

                        let result = ProcessingResult {
                            processor_id: self.id,
                            item_id: item.id,
                            success,
                            duration_ms: duration.as_millis(),
                        };

                        if self.sender.send(result).is_err() {
                            break; // Receiver dropped
                        }

                        thread::sleep(Duration::from_millis(self.config.delay_ms));
                    }
                    None => {
                        thread::sleep(Duration::from_millis(100));
                    }
                }
            }
        })
    }

    fn process_item(&self, item: &ProcessingItem) -> bool {
        // Simulate processing based on priority
        match item.priority {
            Priority::Critical => true,
            Priority::High => item.data.len()  item.data.len()  item.data.len()  Priority::Low,
                    1 => Priority::Medium,
                    2 => Priority::High,
                    3 => Priority::Critical,
                    _ => unreachable!(),
                },
            });
        }
    }

    // Spawn multiple processors
    let config = ProcessingConfig {
        batch_size: 5,
        delay_ms: 50,
        error_threshold: 0.1,
    };

    let handles: Vec = (0..3)
        .map(|id| {
            let processor = ComplexProcessor::new(
                id,
                data.clone(),
                sender.clone(),
                config.clone(),
            );
            processor.run()
        })
        .collect();

    // Drop the original sender so receivers know when to stop
    drop(sender);

    // Collect results
    let mut results = Vec::new();
    while let Ok(result) = receiver.recv() {
        results.push(result);
    }

    // Wait for all threads to complete
    for handle in handles {
        handle.join().unwrap();
    }

    println!("Processed {} items", results.len());
    for result in results {
        println!("Processor {}: Item {} - Success: {} ({}ms)",
                 result.processor_id, result.item_id, result.success, result.duration_ms);
    }
}
```

## 6. Trait Objects and Dynamic Ownership

### Complex Trait Object Scenarios

Trait objects introduce dynamic dispatch and complex ownership patterns:

```rust
use std::any::Any;

trait Processor: Send + Sync {
    fn process(&self, data: &[u8]) -> Vec;
    fn name(&self) -> &str;
    fn as_any(&self) -> &dyn Any;
}

struct ImageProcessor {
    quality: u8,
    format: String,
}

struct VideoProcessor {
    codec: String,
    bitrate: u32,
}

struct AudioProcessor {
    sample_rate: u32,
    channels: u8,
}

impl Processor for ImageProcessor {
    fn process(&self, data: &[u8]) -> Vec {
        // Simulate image processing
        data.iter().map(|&b| b.saturating_mul(self.quality)).collect()
    }

    fn name(&self) -> &str {
        "ImageProcessor"
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

impl Processor for VideoProcessor {
    fn process(&self, data: &[u8]) -> Vec {
        // Simulate video processing
        data.chunks(4)
            .flat_map(|chunk| chunk.iter().cloned())
            .take(data.len() / 2)
            .collect()
    }

    fn name(&self) -> &str {
        "VideoProcessor"
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

impl Processor for AudioProcessor {
    fn process(&self, data: &[u8]) -> Vec {
        // Simulate audio processing
        data.iter()
            .enumerate()
            .filter(|(i, _)| i % (self.channels as usize) == 0)
            .map(|(_, &b)| b)
            .collect()
    }

    fn name(&self) -> &str {
        "AudioProcessor"
    }

    fn as_any(&self) -> &dyn Any {
        self
    }
}

// Complex ownership with trait objects
struct ProcessingPipeline {
    processors: Vec>,
    intermediate_results: Vec>,
}

impl ProcessingPipeline {
    fn new() -> Self {
        ProcessingPipeline {
            processors: Vec::new(),
            intermediate_results: Vec::new(),
        }
    }

    fn add_processor(mut self, processor: Box) -> Self {
        self.processors.push(processor);
        self
    }

    fn process_data(mut self, input: Vec) -> Vec {
        let mut current_data = input;
        self.intermediate_results.clear();

        for processor in &self.processors {
            println!("Processing with {}", processor.name());
            self.intermediate_results.push(current_data.clone());
            current_data = processor.process(&current_data);
        }

        current_data
    }

    fn get_processor_by_type(&self) -> Option {
        self.processors
            .iter()
            .find_map(|processor| {
                processor.as_any().downcast_ref::()
            })
    }
}

fn complex_trait_object_ownership() {
    let pipeline = ProcessingPipeline::new()
        .add_processor(Box::new(ImageProcessor {
            quality: 80,
            format: "JPEG".to_string(),
        }))
        .add_processor(Box::new(VideoProcessor {
            codec: "H264".to_string(),
            bitrate: 1000,
        }))
        .add_processor(Box::new(AudioProcessor {
            sample_rate: 44100,
            channels: 2,
        }));

    let input_data = vec![100u8; 1000];

    // Try to access specific processor type
    if let Some(image_proc) = pipeline.get_processor_by_type::() {
        println!("Found image processor with quality: {}", image_proc.quality);
    }

    let result = pipeline.process_data(input_data);
    println!("Final result size: {}", result.len());
}
```

## 7. Async Ownership Scenarios

### Complex Async Ownership Patterns

Async code introduces additional complexity to ownership scenarios:

```rust
use std::future::Future;
use std::pin::Pin;
use std::sync::Arc;
use tokio::sync::{Mutex, RwLock};
use tokio::time::{sleep, Duration};

struct AsyncProcessor {
    id: usize,
    state: Arc>,
    work_queue: Arc>>,
}

#[derive(Debug, Clone)]
struct ProcessorState {
    is_running: bool,
    processed_count: usize,
    error_count: usize,
}

#[derive(Debug)]
struct WorkItem {
    id: usize,
    data: Vec,
    priority: u8,
}

impl AsyncProcessor {
    fn new(id: usize, work_queue: Arc>>) -> Self {
        AsyncProcessor {
            id,
            state: Arc::new(RwLock::new(ProcessorState {
                is_running: false,
                processed_count: 0,
                error_count: 0,
            })),
            work_queue,
        }
    }

    async fn start(&self) -> Result {
        {
            let mut state = self.state.write().await;
            if state.is_running {
                return Err(ProcessingError::AlreadyRunning);
            }
            state.is_running = true;
        }

        self.processing_loop().await
    }

    async fn processing_loop(&self) -> Result {
        loop {
            let work_item = {
                let mut queue = self.work_queue.lock().await;
                queue.pop()
            };

            match work_item {
                Some(item) => {
                    match self.process_item(item).await {
                        Ok(_) => {
                            let mut state = self.state.write().await;
                            state.processed_count += 1;
                        }
                        Err(_) => {
                            let mut state = self.state.write().await;
                            state.error_count += 1;
                        }
                    }
                }
                None => {
                    // Check if we should continue running
                    let state = self.state.read().await;
                    if !state.is_running {
                        break;
                    }
                    drop(state); // Release read lock before sleeping
                    sleep(Duration::from_millis(100)).await;
                }
            }
        }

        Ok(())
    }

    async fn process_item(&self, item: WorkItem) -> Result, ProcessingError> {
        // Simulate async processing
        sleep(Duration::from_millis(item.priority as u64 * 10)).await;

        if item.data.len() % 13 == 0 {
            return Err(ProcessingError::ProcessingFailed);
        }

        Ok(item.data.into_iter().map(|b| b.wrapping_mul(2)).collect())
    }

    async fn stop(&self) -> ProcessorState {
        let mut state = self.state.write().await;
        state.is_running = false;
        state.clone()
    }

    async fn get_state(&self) -> ProcessorState {
        let state = self.state.read().await;
        state.clone()
    }
}

#[derive(Debug)]
enum ProcessingError {
    AlreadyRunning,
    ProcessingFailed,
}

// Complex async ownership with multiple processors
async fn complex_async_ownership() {
    let work_queue = Arc::new(Mutex::new(Vec::new()));

    // Populate work queue
    {
        let mut queue = work_queue.lock().await;
        for i in 0..20 {
            queue.push(WorkItem {
                id: i,
                data: vec![(i % 256) as u8; (i * 10) % 100 + 1],
                priority: (i % 5) as u8 + 1,
            });
        }
    }

    // Create multiple processors
    let processors: Vec = (0..3)
        .map(|id| AsyncProcessor::new(id, work_queue.clone()))
        .collect();

    // Start all processors concurrently
    let processor_handles: Vec = processors
        .iter()
        .map(|processor| {
            let processor_clone = processor.clone(); // This won't work directly
            tokio::spawn(async move {
                // This creates a complex ownership scenario
                // processor_clone.start().await
            })
        })
        .collect();

    // Simplified version using Arc
    let processor_arcs: Vec> = (0..3)
        .map(|id| Arc::new(AsyncProcessor::new(id, work_queue.clone())))
        .collect();

    let handles: Vec = processor_arcs
        .into_iter()
        .map(|processor| {
            tokio::spawn(async move {
                if let Err(e) = processor.start().await {
                    println!("Processor {} error: {:?}", processor.id, e);
                }
            })
        })
        .collect();

    // Let processors run for a while
    sleep(Duration::from_secs(2)).await;

    // Wait for all tasks to complete
    for handle in handles {
        handle.await.unwrap();
    }
}

// We need to implement Clone for AsyncProcessor to use it in async contexts
impl Clone for AsyncProcessor {
    fn clone(&self) -> Self {
        AsyncProcessor {
            id: self.id,
            state: self.state.clone(),
            work_queue: self.work_queue.clone(),
        }
    }
}
```

## 8. Generic Types and Ownership Constraints

### Complex Generic Ownership Patterns

Generic types can create intricate ownership scenarios with lifetime and trait bound interactions:

```rust
use std::marker::PhantomData;
use std::collections::HashMap;

// Complex generic structure with ownership constraints
struct Repository
where
    T: Clone + Send + Sync,
    K: Clone + Eq + std::hash::Hash + Send + Sync,
{
    storage: HashMap,
    cache: Option>>,
    _phantom: PhantomData,
}

trait Cache: Send + Sync
where
    T: Clone,
    K: Clone + Eq + std::hash::Hash,
{
    fn get(&self, key: &K) -> Option;
    fn put(&mut self, key: K, value: T);
    fn invalidate(&mut self, key: &K);
}

struct LruCache
where
    T: Clone,
    K: Clone + Eq + std::hash::Hash,
{
    capacity: usize,
    storage: HashMap,
    access_order: Vec,
}

impl Cache for LruCache
where
    T: Clone,
    K: Clone + Eq + std::hash::Hash,
{
    fn get(&self, key: &K) -> Option {
        self.storage.get(key).cloned()
    }

    fn put(&mut self, key: K, value: T) {
        if self.storage.len() >= self.capacity {
            if let Some(oldest_key) = self.access_order.first().cloned() {
                self.storage.remove(&oldest_key);
                self.access_order.retain(|k| k != &oldest_key);
            }
        }

        self.storage.insert(key.clone(), value);
        self.access_order.push(key);
    }

    fn invalidate(&mut self, key: &K) {
        self.storage.remove(key);
        self.access_order.retain(|k| k != key);
    }
}

impl Repository
where
    T: Clone + Send + Sync,
    K: Clone + Eq + std::hash::Hash + Send + Sync,
{
    fn new() -> Self {
        Repository {
            storage: HashMap::new(),
            cache: None,
            _phantom: PhantomData,
        }
    }

    fn with_cache(mut self, cache: Box>) -> Self {
        self.cache = Some(cache);
        self
    }

    fn store(&mut self, key: K, value: T) -> Option {
        let old_value = self.storage.insert(key.clone(), value.clone());

        if let Some(ref mut cache) = self.cache {
            cache.put(key, value);
        }

        old_value
    }

    fn retrieve(&self, key: &K) -> Option {
        // Try cache first
        if let Some(ref cache) = self.cache {
            if let Some(value) = cache.get(key) {
                return Some(value);
            }
        }

        // Fallback to storage
        self.storage.get(key).cloned()
    }

    fn remove(&mut self, key: &K) -> Option {
        if let Some(ref mut cache) = self.cache {
            cache.invalidate(key);
        }

        self.storage.remove(key)
    }
}

// Complex ownership scenario with multiple generic parameters and lifetimes
struct ComplexProcessor
where
    T: Clone + Send + 'a,
    F: Fn(&T) -> R + Send + Sync + 'a,
    R: Clone + Send,
{
    name: String,
    processor_fn: F,
    input_data: &'a [T],
    processed_results: Vec,
    _phantom: PhantomData,
}

impl ComplexProcessor
where
    T: Clone + Send + 'a,
    F: Fn(&T) -> R + Send + Sync + 'a,
    R: Clone + Send,
{
    fn new(name: String, processor_fn: F, input_data: &'a [T]) -> Self {
        ComplexProcessor {
            name,
            processor_fn,
            input_data,
            processed_results: Vec::new(),
            _phantom: PhantomData,
        }
    }

    fn process_all(mut self) -> Vec {
        self.processed_results = self.input_data
            .iter()
            .map(|item| (self.processor_fn)(item))
            .collect();

        self.processed_results
    }

    fn process_parallel(mut self) -> Vec
    where
        T: Sync,
        R: Send,
    {
        use std::sync::Arc;
        use std::thread;

        let processor_fn = Arc::new(self.processor_fn);
        let chunk_size = (self.input_data.len() + 3) / 4; // 4 threads

        let handles: Vec = self.input_data
            .chunks(chunk_size)
            .map(|chunk| {
                let processor_fn_clone = processor_fn.clone();
                let chunk_vec = chunk.to_vec(); // Clone chunk for thread ownership

                thread::spawn(move || {
                    chunk_vec
                        .iter()
                        .map(|item| processor_fn_clone(item))
                        .collect::>()
                })
            })
            .collect();

        let mut results = Vec::new();
        for handle in handles {
            results.extend(handle.join().unwrap());
        }

        results
    }
}

fn complex_generic_ownership() {
    // Repository with cache
    let cache = Box::new(LruCache:: {
        capacity: 100,
        storage: HashMap::new(),
        access_order: Vec::new(),
    });

    let mut repo = Repository::::new()
        .with_cache(cache);

    repo.store(1, "first".to_string());
    repo.store(2, "second".to_string());
    repo.store(3, "third".to_string());

    println!("Retrieved: {:?}", repo.retrieve(&2));

    // Complex processor with lifetime constraints
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let processor = ComplexProcessor::new(
        "square_processor".to_string(),
        |x: &i32| x * x,
        &data,
    );

    let results = processor.process_parallel();
    println!("Processed results: {:?}", results);
}
```

## Summary and Key Patterns

### **Complex Ownership Scenario Categories**

1. **Structural Complexity**
   - Partial moves from structs and enums[1]
   - Nested ownership in recursive data structures
   - Pattern matching with ownership transfer[2]

2. **Functional Complexity**
   - Closure capture modes and thread ownership[4][5]
   - Higher-order functions with trait bounds
   - Async ownership across await points

3. **Concurrency Complexity**
   - Send/Sync constraints with shared data[7]
   - Arc/Mutex patterns for thread-safe ownership
   - Channel-based ownership transfer

4. **Type System Complexity**
   - Trait objects with dynamic dispatch
   - Generic types with lifetime constraints
   - Self-referential structures with Pin

### **Best Practices for Complex Scenarios**

```rust
// 1. Use clear ownership documentation
/// Takes ownership of the processor and returns processed results.
/// The processor cannot be used after this call.
fn process_and_consume(processor: DataProcessor) -> Results {
    processor.into_results()
}

// 2. Design APIs to minimize complexity
pub struct Builder {
    inner: T,
}

impl Builder {
    pub fn with_config(mut self, config: Config) -> Self {
        self.inner.configure(config);
        self
    }

    pub fn build(self) -> T {
        self.inner // Clear ownership transfer
    }
}

// 3. Use type aliases for complex generic constraints
type ProcessorResult = Result;
type SharedData = Arc>;
type ProcessorFn = Box ProcessorResult + Send + Sync>;
```

### **Key Principles**

- **Explicit over implicit** - Make ownership transfers obvious in your API design
- **Minimize complexity** - Use references and borrowing when ownership transfer isn't necessary
- **Document ownership semantics** - Clearly indicate when functions take ownership
- **Use appropriate smart pointers** - Choose Arc, Rc, Box based on your threading and sharing needs
- **Consider Pin for self-references** - Use Pin> for self-referential structures
- **Test ownership boundaries** - Verify that your complex ownership patterns work correctly under all conditions

Complex ownership scenarios in Rust require careful consideration of data flow, lifetime relationships, and the interaction between different language features. **Understanding these patterns enables you to build sophisticated, safe, and efficient systems** while maintaining Rust's memory safety guarantees.

1. https://www.rustfinity.com/learn/rust/structs/structs-and-ownership
2. https://www.reddit.com/r/learnrust/comments/1acu9c8/need_help_understanding_match_expression/
3. https://doc.rust-lang.org/book/ch19-03-pattern-syntax.html
4. https://stackoverflow.com/questions/62125100/why-does-rust-closure-take-ownership-before-being-called
5. https://www.reddit.com/r/rust/comments/1azprsh/ownership_and_closure/
6. https://www.kth.se/files/view/svardl/5c1448c383da5d22bbd2f4e2/ownership-in-rust.pdf
7. https://www.javacodegeeks.com/2024/10/mastering-ownership-and-lifetimes-in-rust-advanced-strategies.html
8. https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
9. https://www.reddit.com/r/rust/comments/yibdpi/ownership_as_explained_in_the_rust_book/
10. https://www.programiz.com/rust/ownership
11. https://dev.to/leapcell/rust-ownership-and-borrowing-explained-22l6
12. https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/
13. https://www.risein.com/courses/rust-programming/ownership-concept
14. https://www.w3schools.com/rust/rust_ownership.php
15. https://masteringbackend.com/hubs/advanced-rust/advanced-ownership-in-rust
16. https://dev.to/ajtech0001/enums-and-pattern-matching-in-rust-a-deep-dive-into-type-safety-and-control-flow-1gjp
17. https://dev.to/lymah/week-3-of-learning-rust-match-patterns-and-the-power-of-methods-3ed4
18. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/closures.html
19. https://users.rust-lang.org/t/modeling-a-multiple-ownership-scenario/62744
20. https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html
21. https://blog.devgenius.io/rust-structs-and-enums-under-the-hood-960a049e5019
22. https://rust-book.cs.brown.edu/ch04-01-what-is-ownership.html
23. https://earthly.dev/blog/rust-lifetimes-ownership-burrowing/
24. https://dev.to/francescoxx/ownership-in-rust-57j2
25. https://betterprogramming.pub/understanding-rust-ownership-model-by-example-5d586ec5e8e4
26. https://www.andy-pearce.com/blog/posts/2019/Jun/uncovering-rust-references-and-ownership/
27. https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/second-edition/ch04-01-what-is-ownership.html
28. https://fitech101.aalto.fi/programming-languages/rust/11-structs-and-enums/
29. https://stackoverflow.com/questions/64761331/rust-take-ownership-of-enum-value-without-another-pattern-matching
30. https://www.risein.com/courses/rust-programming/implementation-of-generics-using-structs-and-enums-in-rust
