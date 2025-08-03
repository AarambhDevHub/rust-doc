+++
title = "Best Practices (Advanced Ownership)"
description = "Learn best practices for managing complex ownership scenarios in Rust, including effective use of smart pointers, borrowing, and lifetimes."
date = 2025-08-03
draft = false
weight = 5
+++

# Advanced Ownership Best Practices in Rust: Comprehensive Documentation

Building on the foundational concepts of ownership, references, and memory management, this guide presents **advanced best practices** for mastering Rust's ownership system. These practices will help you write more efficient, safer, and more maintainable Rust code.

## Best Practices for the Clone Trait

### **1. Use Clone Judiciously - Prefer Borrowing When Possible**

**Anti-pattern: Unnecessary Cloning**
```rust
// Inefficient - clones when borrowing would suffice
fn bad_count_words(text: String) -> usize {
    let words = text.clone(); // Unnecessary heap allocation
    words.split_whitespace().count()
}

// Better - borrow instead of cloning
fn good_count_words(text: &str) -> usize {
    text.split_whitespace().count()
}

fn main() {
    let document = String::from("The quick brown fox jumps over the lazy dog");

    // Inefficient: forces caller to clone or move
    // let count = bad_count_words(document.clone());

    // Efficient: allows reuse without cloning
    let count = good_count_words(&document);
    println!("Word count: {}", count);
    println!("Original still available: {}", document);
}
```

### **2. Implement Clone Efficiently with `clone_from`**

```rust
use std::time::Instant;

#[derive(Clone)]
struct LargeData {
    buffer: Vec,
    metadata: String,
}

impl LargeData {
    fn new(size: usize, info: &str) -> Self {
        LargeData {
            buffer: vec![0; size],
            metadata: info.to_string(),
        }
    }
}

fn demonstrate_clone_efficiency() {
    let original = LargeData::new(1_000_000, "large dataset");
    let mut target = LargeData::new(10, "small");

    // Inefficient: always allocates new memory
    let start = Instant::now();
    target = original.clone();
    let clone_time = start.elapsed();

    // Efficient: reuses existing allocation when possible
    let mut target2 = LargeData::new(1_000_000, "reusable");
    let start = Instant::now();
    target2.clone_from(&original);
    let clone_from_time = start.elapsed();

    println!("clone(): {:?}", clone_time);
    println!("clone_from(): {:?}", clone_from_time);
}
```

### **3. Design APIs to Minimize Clone Requirements**

```rust
// Good: Flexible API that works with borrowed or owned data
fn process_config(config: &Configuration) -> Result {
    // Process without taking ownership
    Ok(Summary::from_config(config))
}

// Good: Builder pattern that moves ownership efficiently
struct ConfigBuilder {
    inner: Configuration,
}

impl ConfigBuilder {
    fn add_feature(mut self, feature: Feature) -> Self {
        self.inner.features.push(feature); // Move feature, no clone needed
        self
    }

    fn build(self) -> Configuration {
        self.inner // Move ownership to caller
    }
}

// Advanced: Smart cloning with copy-on-write semantics
use std::borrow::Cow;

fn smart_transform(data: Cow) -> Cow {
    if data.contains("special") {
        // Only clone when modification is needed
        Cow::Owned(data.replace("special", "SPECIAL"))
    } else {
        data // Return borrowed data unchanged
    }
}

fn main() {
    let text = "normal text";
    let result = smart_transform(Cow::Borrowed(text));
    // No allocation if no transformation needed

    let special_text = "special text";
    let result2 = smart_transform(Cow::Borrowed(special_text));
    // Only allocates when transformation occurs
}
```

### **4. Document Clone Costs in Public APIs**

```rust
/// Configuration manager for application settings.
///
/// # Performance Notes
/// - `clone()` performs a deep copy of all settings (~O(n) where n = number of settings)
/// - Consider using `&Config` for read-only operations
/// - Use `clone_from()` for repeated cloning to reuse allocations
#[derive(Clone)]
pub struct Config {
    settings: std::collections::HashMap,
    cached_values: Vec,
}

impl Config {
    /// Clones configuration efficiently, reusing target's allocations when possible.
    ///
    /// This is more efficient than `target = source.clone()` when target
    /// already has allocated capacity.
    pub fn clone_into(&self, target: &mut Self) {
        target.clone_from(self);
    }

    /// Returns a borrowed view of settings for read-only access.
    ///
    /// Prefer this over cloning when you only need to read configuration.
    pub fn settings(&self) -> &std::collections::HashMap {
        &self.settings
    }
}
```

## Best Practices for the Drop Trait

### **1. Keep Drop Implementation Simple and Fast**

```rust
// Good: Simple, fast cleanup
struct GoodResource {
    connection: TcpConnection,
    buffer: Vec,
}

impl Drop for GoodResource {
    fn drop(&mut self) {
        // Fast cleanup operations only
        if let Err(e) = self.connection.close() {
            // Log but don't panic
            eprintln!("Warning: Failed to close connection: {}", e);
        }
        // buffer automatically dropped
    }
}

// Avoid: Expensive operations in Drop
struct ProblematicResource {
    data: Vec,
}

impl Drop for ProblematicResource {
    fn drop(&mut self) {
        // DON'T DO THIS: expensive operations in Drop
        // self.perform_complex_analysis(); // Slow!
        // self.send_network_request();      // Can fail!
        // self.write_large_file();          // I/O intensive!

        // DO THIS: simple cleanup only
        println!("Resource cleaned up");
    }
}
```

### **2. Handle Errors Gracefully - Never Panic in Drop**

```rust
use std::fs::File;
use std::io::Write;

struct SafeFileWriter {
    file: Option,
    temp_path: Option,
    final_path: std::path::PathBuf,
}

impl SafeFileWriter {
    fn new(path: std::path::PathBuf) -> std::io::Result {
        let temp_path = path.with_extension("tmp");
        let file = File::create(&temp_path)?;

        Ok(SafeFileWriter {
            file: Some(file),
            temp_path: Some(temp_path),
            final_path: path,
        })
    }

    fn write_data(&mut self, data: &[u8]) -> std::io::Result {
        if let Some(ref mut file) = self.file {
            file.write_all(data)
        } else {
            Err(std::io::Error::new(
                std::io::ErrorKind::Other,
                "File already finalized"
            ))
        }
    }

    fn finalize(mut self) -> std::io::Result {
        if let (Some(mut file), Some(temp_path)) = (self.file.take(), self.temp_path.take()) {
            file.flush()?;
            drop(file); // Ensure file is closed
            std::fs::rename(temp_path, &self.final_path)?;
        }
        Ok(())
    }
}

impl Drop for SafeFileWriter {
    fn drop(&mut self) {
        // Cleanup temporary files on Drop - never panic
        if let Some(temp_path) = &self.temp_path {
            if temp_path.exists() {
                if let Err(e) = std::fs::remove_file(temp_path) {
                    eprintln!("Warning: Failed to clean up temp file {:?}: {}", temp_path, e);
                }
            }
        }

        // Always log what happened
        if self.file.is_some() {
            eprintln!("Warning: SafeFileWriter dropped without finalization");
        }
    }
}
```

### **3. Make Drop Operations Idempotent**

```rust
struct IdempotentResource {
    connection: Option,
    cleanup_performed: bool,
}

impl IdempotentResource {
    fn new() -> Self {
        IdempotentResource {
            connection: Some(DatabaseConnection::new()),
            cleanup_performed: false,
        }
    }

    fn close(&mut self) -> Result {
        if !self.cleanup_performed {
            if let Some(conn) = self.connection.take() {
                conn.close()?;
                self.cleanup_performed = true;
                println!("Resource explicitly closed");
            }
        }
        Ok(())
    }
}

impl Drop for IdempotentResource {
    fn drop(&mut self) {
        // Safe to call multiple times
        if !self.cleanup_performed {
            if let Some(conn) = self.connection.take() {
                if let Err(e) = conn.close() {
                    eprintln!("Error during automatic cleanup: {}", e);
                } else {
                    println!("Resource automatically closed");
                }
                self.cleanup_performed = true;
            }
        }
    }
}
```

### **4. Use Drop for Resource Management, Not Business Logic**

```rust
// Good: Drop for resource cleanup
struct FileHandler {
    file: File,
    lock: FileLock,
}

impl Drop for FileHandler {
    fn drop(&mut self) {
        // Resource cleanup only
        self.lock.release();
        // file automatically closed
    }
}

// Avoid: Business logic in Drop
struct BadEventProcessor {
    events: Vec,
    processor: EventProcessor,
}

impl Drop for BadEventProcessor {
    fn drop(&mut self) {
        // DON'T DO THIS: business logic in Drop
        // for event in &self.events {
        //     self.processor.handle_critical_event(event); // Business logic!
        // }

        // DO THIS: resource cleanup only
        if !self.events.is_empty() {
            eprintln!("Warning: {} unprocessed events", self.events.len());
        }
    }
}

// Better: Explicit finalization for business logic
impl BadEventProcessor {
    fn finalize(mut self) -> Result {
        // Business logic in explicit method
        for event in &self.events {
            self.processor.handle_critical_event(event)?;
        }
        self.events.clear(); // Clear after processing
        Ok(())
    }
}
```

## Advanced RAII Pattern Best Practices

### **1. Design for Automatic Resource Management**

```rust
// Excellent RAII design - resources tied to object lifetime
struct DatabaseTransaction {
    connection: DatabaseConnection,
    transaction_id: TransactionId,
    committed: bool,
}

impl DatabaseTransaction {
    fn begin(connection: DatabaseConnection) -> Result {
        let transaction_id = connection.begin_transaction()?;
        println!("Transaction {} started", transaction_id);

        Ok(DatabaseTransaction {
            connection,
            transaction_id,
            committed: false,
        })
    }

    fn commit(mut self) -> Result {
        self.connection.commit(self.transaction_id)?;
        self.committed = true;
        println!("Transaction {} committed", self.transaction_id);
        Ok(())
    }

    fn execute_query(&mut self, query: &str) -> Result {
        if self.committed {
            return Err(DatabaseError::TransactionClosed);
        }
        self.connection.execute_in_transaction(self.transaction_id, query)
    }
}

impl Drop for DatabaseTransaction {
    fn drop(&mut self) {
        if !self.committed {
            if let Err(e) = self.connection.rollback(self.transaction_id) {
                eprintln!("Warning: Failed to rollback transaction {}: {}",
                         self.transaction_id, e);
            } else {
                println!("Transaction {} automatically rolled back", self.transaction_id);
            }
        }
    }
}

// Usage demonstrates automatic safety
fn safe_database_operation() -> Result {
    let connection = DatabaseConnection::connect("localhost:5432")?;
    let mut transaction = DatabaseTransaction::begin(connection)?;

    transaction.execute_query("INSERT INTO users (name) VALUES ('Alice')")?;
    transaction.execute_query("UPDATE settings SET last_user = 'Alice'")?;

    // If any error occurs, transaction automatically rolls back via Drop
    transaction.commit()?; // Only commits if we reach this point

    Ok(())
}
```

### **2. Layer RAII Resources for Complex Management**

```rust
// Layered RAII for complex resource management
struct ApplicationContext {
    config: Configuration,
    logger: Logger,
    metrics: MetricsCollector,
}

struct RequestProcessor {
    context: ApplicationContext,
    request_id: RequestId,
    start_time: std::time::Instant,
}

struct DatabaseSession {
    processor: RequestProcessor,
    connection: DatabaseConnection,
    session_token: SessionToken,
}

impl ApplicationContext {
    fn new() -> Result {
        let config = Configuration::load()?;
        let logger = Logger::new(&config.log_path)?;
        let metrics = MetricsCollector::new(&config.metrics_endpoint)?;

        println!("Application context initialized");
        Ok(ApplicationContext { config, logger, metrics })
    }
}

impl Drop for ApplicationContext {
    fn drop(&mut self) {
        println!("Shutting down application context");
        self.metrics.flush();
        // logger and config automatically cleaned up
    }
}

impl RequestProcessor {
    fn new(context: ApplicationContext, request_id: RequestId) -> Self {
        println!("Processing request {}", request_id);
        RequestProcessor {
            context,
            request_id,
            start_time: std::time::Instant::now(),
        }
    }
}

impl Drop for RequestProcessor {
    fn drop(&mut self) {
        let duration = self.start_time.elapsed();
        println!("Request {} completed in {:?}", self.request_id, duration);
        self.context.metrics.record_request_duration(duration);
        // context automatically cleaned up after metrics recorded
    }
}

impl DatabaseSession {
    fn new(processor: RequestProcessor) -> Result {
        let connection = DatabaseConnection::new(&processor.context.config)?;
        let session_token = connection.create_session()?;

        Ok(DatabaseSession {
            processor,
            connection,
            session_token,
        })
    }
}

impl Drop for DatabaseSession {
    fn drop(&mut self) {
        if let Err(e) = self.connection.end_session(self.session_token) {
            eprintln!("Warning: Failed to end database session: {}", e);
        }
        // processor and connection automatically cleaned up
    }
}

// Usage shows automatic layered cleanup
fn handle_request(request_id: RequestId) -> Result {
    let context = ApplicationContext::new()?;
    let processor = RequestProcessor::new(context, request_id);
    let session = DatabaseSession::new(processor)?;

    // Perform database operations
    let data = session.connection.query("SELECT * FROM data")?;

    Ok(data)
    // Automatic cleanup in reverse order:
    // 1. DatabaseSession (ends session, closes connection)
    // 2. RequestProcessor (records metrics)
    // 3. ApplicationContext (flushes metrics, closes logger)
}
```

### **3. Use RAII Guards for Scope-Based Behavior**

```rust
use std::sync::{Mutex, MutexGuard};

// Performance timing guard
struct TimingGuard {
    operation: String,
    start: std::time::Instant,
}

impl TimingGuard {
    fn new(operation: String) -> Self {
        println!("Starting operation: {}", operation);
        TimingGuard {
            operation,
            start: std::time::Instant::now(),
        }
    }
}

impl Drop for TimingGuard {
    fn drop(&mut self) {
        let duration = self.start.elapsed();
        println!("Operation '{}' completed in {:?}", self.operation, duration);
    }
}

// Scope-based temporary directory
struct TempDir {
    path: std::path::PathBuf,
    cleanup_on_drop: bool,
}

impl TempDir {
    fn new() -> std::io::Result {
        let path = std::env::temp_dir().join(format!("rust_temp_{}",
                                                    std::process::id()));
        std::fs::create_dir_all(&path)?;
        println!("Created temporary directory: {:?}", path);

        Ok(TempDir {
            path,
            cleanup_on_drop: true,
        })
    }

    fn path(&self) -> &std::path::Path {
        &self.path
    }

    fn persist(mut self) -> std::path::PathBuf {
        self.cleanup_on_drop = false;
        self.path.clone()
    }
}

impl Drop for TempDir {
    fn drop(&mut self) {
        if self.cleanup_on_drop {
            if let Err(e) = std::fs::remove_dir_all(&self.path) {
                eprintln!("Warning: Failed to cleanup temp dir {:?}: {}", self.path, e);
            } else {
                println!("Cleaned up temporary directory: {:?}", self.path);
            }
        }
    }
}

// Usage demonstrates scope-based management
fn process_files() -> std::io::Result {
    let _timer = TimingGuard::new("file_processing".to_string());
    let temp_dir = TempDir::new()?;

    // Create temporary files
    let temp_file = temp_dir.path().join("data.txt");
    std::fs::write(&temp_file, "temporary data")?;

    // Process files...

    // Option 1: Let temp directory clean up automatically
    Ok(())

    // Option 2: Persist the directory
    // let persistent_path = temp_dir.persist();
    // println!("Directory persisted at: {:?}", persistent_path);
    // Ok(())
}
```

## Advanced Memory Management Best Practices

### **1. Choose the Right Smart Pointer**

```rust
use std::rc::Rc;
use std::sync::Arc;
use std::cell::RefCell;
use std::sync::Mutex;

// Decision matrix for smart pointers
fn demonstrate_smart_pointer_usage() {
    // Single-threaded, single owner: Box
    let boxed_data = Box::new(LargeStruct::new());

    // Single-threaded, multiple owners: Rc
    let shared_data = Rc::new(SharedData::new());
    let reference1 = shared_data.clone();
    let reference2 = shared_data.clone();

    // Single-threaded, interior mutability: Rc>
    let mutable_shared = Rc::new(RefCell::new(MutableData::new()));
    {
        let mut borrowed = mutable_shared.borrow_mut();
        borrowed.update();
    } // Borrow released automatically

    // Multi-threaded, multiple owners: Arc
    let thread_safe_data = Arc::new(ThreadSafeData::new());
    let handle = {
        let data_clone = thread_safe_data.clone();
        std::thread::spawn(move || {
            data_clone.process();
        })
    };

    // Multi-threaded, interior mutability: Arc>
    let thread_safe_mutable = Arc::new(Mutex::new(MutableData::new()));
    let handles: Vec = (0..3).map(|i| {
        let data_clone = thread_safe_mutable.clone();
        std::thread::spawn(move || {
            let mut guard = data_clone.lock().unwrap();
            guard.process_with_id(i);
        })
    }).collect();

    // Wait for threads
    handle.join().unwrap();
    for h in handles {
        h.join().unwrap();
    }
}
```

### **2. Optimize Memory Layout and Access Patterns**

```rust
// Memory-efficient struct layout
#[repr(C)]
struct OptimizedStruct {
    // Group fields by size to minimize padding
    large_field: u64,      // 8 bytes
    medium_field: u32,     // 4 bytes
    small_field: u16,      // 2 bytes
    tiny_field: u8,        // 1 byte
    another_tiny: u8,      // 1 byte - packed with previous
    // Total: 16 bytes (vs 24 with poor ordering)
}

// Cache-friendly data structures
struct CacheFriendlyProcessor {
    // Hot data (frequently accessed)
    counters: [u64; 8],
    flags: u64,

    // Cold data (infrequently accessed)
    configuration: Box,
    debug_info: Option>,
}

impl CacheFriendlyProcessor {
    fn process_hot_path(&mut self, index: usize) {
        // Only accesses hot data - good cache locality
        self.counters[index] += 1;
        self.flags |= 1  {
    available: Vec>,
    in_use: usize,
}

impl MemoryPool {
    fn new(initial_capacity: usize) -> Self {
        let mut available = Vec::with_capacity(initial_capacity);
        for _ in 0..initial_capacity {
            available.push(Box::new(T::default()));
        }

        MemoryPool {
            available,
            in_use: 0,
        }
    }

    fn acquire(&mut self) -> Option> {
        if let Some(item) = self.available.pop() {
            self.in_use += 1;
            Some(item)
        } else {
            // Pool exhausted - could allocate new or return None
            None
        }
    }

    fn release(&mut self, mut item: Box) {
        // Reset item state
        *item = T::default();
        self.available.push(item);
        self.in_use = self.in_use.saturating_sub(1);
    }
}
```

### **3. Design for Zero-Copy Operations**

```rust
use std::borrow::Cow;

// Zero-copy string processing
fn process_text(input: &'a str) -> Cow {
    if input.chars().any(|c| c.is_uppercase()) {
        // Only allocate when transformation is needed
        Cow::Owned(input.to_lowercase())
    } else {
        // Return borrowed data unchanged
        Cow::Borrowed(input)
    }
}

// Zero-copy slice operations
fn find_pattern(haystack: &[T], needle: &[T]) -> Option {
    haystack
        .windows(needle.len())
        .find(|window| *window == needle)
}

// Efficient data transformation pipeline
struct DataProcessor {
    input: &'a [u8],
    position: usize,
}

impl DataProcessor {
    fn new(input: &'a [u8]) -> Self {
        DataProcessor { input, position: 0 }
    }

    fn next_chunk(&mut self, size: usize) -> Option {
        if self.position + size  &'a [u8] {
        &self.input[self.position..]
    }
}

// Usage shows zero-copy efficiency
fn efficient_data_processing(data: &[u8]) -> Vec {
    let mut processor = DataProcessor::new(data);
    let mut results = Vec::new();

    while let Some(chunk) = processor.next_chunk(1024) {
        // Process chunk without copying data
        results.push(ProcessedChunk::from_slice(chunk));
    }

    // Handle remaining data
    let remainder = processor.remaining();
    if !remainder.is_empty() {
        results.push(ProcessedChunk::from_slice(remainder));
    }

    results
}
```

### **4. Implement Custom Allocators for Specialized Use Cases**

```rust
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering};

// Custom allocator with tracking
struct TrackedAllocator {
    inner: System,
    allocated: AtomicUsize,
    deallocated: AtomicUsize,
}

impl TrackedAllocator {
    const fn new() -> Self {
        TrackedAllocator {
            inner: System,
            allocated: AtomicUsize::new(0),
            deallocated: AtomicUsize::new(0),
        }
    }

    fn allocated_bytes(&self) -> usize {
        self.allocated.load(Ordering::Relaxed)
    }

    fn deallocated_bytes(&self) -> usize {
        self.deallocated.load(Ordering::Relaxed)
    }

    fn current_usage(&self) -> usize {
        self.allocated_bytes() - self.deallocated_bytes()
    }
}

unsafe impl GlobalAlloc for TrackedAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        let ptr = self.inner.alloc(layout);
        if !ptr.is_null() {
            self.allocated.fetch_add(layout.size(), Ordering::Relaxed);
        }
        ptr
    }

    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        self.inner.dealloc(ptr, layout);
        self.deallocated.fetch_add(layout.size(), Ordering::Relaxed);
    }
}

// Usage with custom allocator
#[global_allocator]
static TRACKED_ALLOCATOR: TrackedAllocator = TrackedAllocator::new();

fn memory_usage_demo() {
    println!("Initial usage: {} bytes", TRACKED_ALLOCATOR.current_usage());

    {
        let large_vec = vec![0u8; 1_000_000];
        println!("After allocation: {} bytes", TRACKED_ALLOCATOR.current_usage());
    }

    println!("After deallocation: {} bytes", TRACKED_ALLOCATOR.current_usage());
}
```

## Integration Best Practices

### **1. Combining All Patterns Effectively**

```rust
// Comprehensive example combining all advanced ownership patterns
use std::sync::Arc;
use std::sync::Mutex;

pub struct AdvancedResourceManager {
    // RAII-managed resources
    database_pool: DatabasePool,
    file_cache: FileCache,

    // Shared state with appropriate smart pointers
    metrics: Arc>,
    configuration: Arc,

    // Cleanup tracking
    cleanup_handlers: Vec>,
}

impl AdvancedResourceManager {
    pub fn new(config: Configuration) -> Result {
        let _timer = TimingGuard::new("ResourceManager::new".to_string());

        // Initialize RAII resources
        let database_pool = DatabasePool::new(&config.database_url)?;
        let file_cache = FileCache::new(&config.cache_directory)?;

        // Setup shared resources
        let metrics = Arc::new(Mutex::new(MetricsCollector::new()));
        let configuration = Arc::new(config);

        Ok(AdvancedResourceManager {
            database_pool,
            file_cache,
            metrics,
            configuration,
            cleanup_handlers: Vec::new(),
        })
    }

    pub fn register_cleanup(&mut self, cleanup: F)
    where
        F: Fn() + Send + 'static
    {
        self.cleanup_handlers.push(Box::new(cleanup));
    }

    pub fn create_worker(&self) -> Worker {
        Worker {
            database: self.database_pool.get_connection(),
            cache: self.file_cache.clone(),
            metrics: self.metrics.clone(),
            config: self.configuration.clone(),
        }
    }
}

impl Drop for AdvancedResourceManager {
    fn drop(&mut self) {
        // Run custom cleanup handlers first
        for cleanup in &self.cleanup_handlers {
            cleanup();
        }

        // Flush metrics before shutdown
        if let Ok(metrics) = self.metrics.lock() {
            metrics.flush();
        }

        // RAII resources automatically cleaned up after this
        println!("AdvancedResourceManager cleanup completed");
    }
}

pub struct Worker {
    database: DatabaseConnection,
    cache: FileCache,
    metrics: Arc>,
    config: Arc,
}

impl Worker {
    pub fn process_request(&mut self, request: Request) -> Result {
        let _timer = TimingGuard::new(format!("process_request_{}", request.id));

        // Try cache first (zero-copy when possible)
        if let Some(cached) = self.cache.get(&request.key) {
            return Ok(Response::from_cache(cached));
        }

        // Fallback to database
        let data = self.database.query(&request.query)?;

        // Update cache and metrics
        self.cache.insert(request.key, data.clone());

        if let Ok(mut metrics) = self.metrics.lock() {
            metrics.record_cache_miss();
        }

        Ok(Response::from_data(data))
    }
}

impl Drop for Worker {
    fn drop(&mut self) {
        // Ensure any pending operations are completed
        if let Err(e) = self.database.flush() {
            eprintln!("Warning: Failed to flush database connection: {}", e);
        }

        println!("Worker cleanup completed");
    }
}
```

### **2. Error Handling Integration**

```rust
use std::error::Error;
use std::fmt;

// Custom error types that work well with RAII
#[derive(Debug)]
pub enum ResourceError {
    InitializationFailed(String),
    OperationFailed { operation: String, cause: Box },
    ResourceExhausted(String),
}

impl fmt::Display for ResourceError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            ResourceError::InitializationFailed(msg) => {
                write!(f, "Resource initialization failed: {}", msg)
            }
            ResourceError::OperationFailed { operation, cause } => {
                write!(f, "Operation '{}' failed: {}", operation, cause)
            }
            ResourceError::ResourceExhausted(resource) => {
                write!(f, "Resource exhausted: {}", resource)
            }
        }
    }
}

impl Error for ResourceError {}

// RAII guard that handles errors properly
pub struct ErrorHandlingGuard {
    operation: String,
    cleanup_on_error: Option>,
    success: bool,
}

impl ErrorHandlingGuard {
    pub fn new(operation: String, cleanup_on_error: F) -> Self
    where
        F: FnOnce() + 'static
    {
        ErrorHandlingGuard {
            operation,
            cleanup_on_error: Some(Box::new(cleanup_on_error)),
            success: false,
        }
    }

    pub fn mark_success(mut self) {
        self.success = true;
    }
}

impl Drop for ErrorHandlingGuard {
    fn drop(&mut self) {
        if !self.success {
            if let Some(cleanup) = self.cleanup_on_error.take() {
                cleanup();
            }
            println!("Operation '{}' failed, cleanup performed", self.operation);
        } else {
            println!("Operation '{}' completed successfully", self.operation);
        }
    }
}
```

## Summary and Key Principles

### **Master These Advanced Patterns**

1. **Clone Strategically**
   - Prefer borrowing over cloning
   - Use `clone_from` for repeated cloning
   - Document clone costs in public APIs
   - Design APIs to minimize clone requirements

2. **Drop Safely and Efficiently**
   - Keep Drop implementations simple and fast
   - Never panic in Drop implementations
   - Make Drop operations idempotent
   - Use Drop for resource cleanup only, not business logic

3. **Apply RAII Systematically**
   - Design for automatic resource management
   - Layer RAII resources for complex scenarios
   - Use RAII guards for scope-based behavior
   - Combine with ownership system for maximum safety

4. **Optimize Memory Management**
   - Choose appropriate smart pointers
   - Design for cache-friendly access patterns
   - Implement zero-copy operations when possible
   - Consider custom allocators for specialized needs

### **Performance and Safety Goals**

- **Zero-cost abstractions** - High-level patterns compile to efficient code
- **Memory safety** - Prevent entire classes of bugs at compile time
- **Resource safety** - Guarantee proper cleanup of all resources
- **Predictable performance** - Deterministic resource management timing

These advanced ownership patterns, when applied correctly, enable you to build **high-performance, memory-safe systems** that leverage Rust's unique advantages while maintaining code clarity and maintainability. The key is understanding when and how to apply each pattern for maximum benefit.
