---
title: "Cache-Friendly Programming in Rust"
description: "Improve performance by writing cache-aware code and memory access patterns in Rust."
date: 2025-11-11
weight: 3
---

# Writing Cache-Friendly Code in Rust for Maximum Performance

To write high-performance Rust code, you need to think like a computer architect. Modern CPUs are incredibly fast, but they are often bottlenecked by a much slower component: main memory (RAM). To bridge this speed gap, CPUs use small, extremely fast memory caches (L1, L2, L3) that store recently used data. **Cache-friendly programming** is the art of arranging your data and writing your code in a way that maximizes the chances of finding data in these fast caches (a "cache hit") and minimizes the times the CPU has to wait for data from slow RAM (a "cache miss").[^1]

## The "Chef's Spice Rack" Analogy 🧑🍳

Imagine a chef working in a large kitchen.

- **Main Memory (RAM)**: A huge pantry at the far end of the kitchen. It holds every possible ingredient but is slow to get to.
- **CPU Cache**: A small spice rack right next to the chef's cooking station. It's tiny but incredibly fast to access.
- **A Cache Miss**: The chef needs a spice that isn't on the rack. They have to stop everything, walk all the way to the pantry, find the spice, and bring it back. This is very slow.
- **A Cache Hit**: The spice is already on the rack. The chef grabs it instantly and continues working without delay.

When the chef fetches a spice (e.g., salt) from the pantry, they don't just bring back one grain. They bring back the whole shaker. This is a **cache line**—a contiguous block of memory. If the next ingredient they need is right next to the salt in the pantry (e.g., pepper), it will already be on their spice rack, resulting in a cache hit.

This leads to the two fundamental principles of cache-friendly code.

## The Two Pillars of Cache Locality

### 1. Spatial Locality: "Things used together should be stored together."

This is the most important principle. If you access a piece of data, it's highly likely you will need to access data that is physically close to it in memory soon. By keeping related data tightly packed, you ensure that a single memory fetch (bringing one "spice shaker" to your rack) pulls in lots of useful data at once.[^1]

**How to Achieve Spatial Locality in Rust:**

- **Prefer `Vec<T>` and arrays over `LinkedList<T>`**: `Vec` stores its elements in a single, contiguous block of memory. This is the most cache-friendly data structure. A `LinkedList`, on the other hand, scatters its nodes all over memory, leading to a cache miss for every single node you visit (pointer chasing).[^2]
- **Store data in a "Struct of Arrays" (SoA) instead of an "Array of Structs" (AoS)**, if your access patterns warrant it.

**Example: AoS vs. SoA**

Imagine we have a game with many particles, each with a position and velocity.

**Array of Structs (AoS) - The Standard Approach (Often Less Cache-Friendly)**

```rust
struct Particle {
    pos_x: f32,
    pos_y: f32,
    vel_x: f32,
    vel_y: f32,
}

let mut particles: Vec<Particle> = //... a million particles

// To update all positions, we have to skip over the velocity data for each particle.
for p in &mut particles {
    p.pos_x += p.vel_x;
    p.pos_y += p.vel_y;
}
```

The memory layout looks like: `[pos_x, pos_y, vel_x, vel_y, pos_x, pos_y, vel_x, vel_y, ...]`. When updating positions, the CPU loads `vel_x` and `vel_y` into the cache even though it doesn't need them for this loop, wasting precious cache space.

**Struct of Arrays (SoA) - The Cache-Friendly Approach**

```rust
struct Particles {
    pos_x: Vec<f32>,
    pos_y: Vec<f32>,
    vel_x: Vec<f32>,
    vel_y: Vec<f32>,
}

let mut particles: Particles = //...

// Now, all the `pos_x` values are contiguous in memory.
for i in 0..particles.pos_x.len() {
    particles.pos_x[i] += particles.vel_x[i];
    particles.pos_y[i] += particles.vel_y[i];
}
```

When this loop runs, the CPU can fetch a large block of `pos_x` values, then a large block of `vel_x` values, leading to far better cache utilization.

### 2. Temporal Locality: "Things used recently will likely be used again soon."

If you just used an ingredient, keep it close because you'll probably need it again. Once data is in the cache, try to perform all the work you need on it before it gets "evicted" to make room for other data.[^1]

**How to Achieve Temporal Locality:**

- **Loop Order Matters**: When iterating over a 2D array (which is stored row-by-row), always make your inner loop iterate over the columns. This touches elements that are next to each other in memory (good spatial locality) and keeps the same row in the cache for the duration of the inner loop (good temporal locality).

**Example: Matrix Traversal**

Let's assume a 2D matrix stored in a flat `Vec` in row-major order.

```rust
const N: usize = 1024;
let matrix: Vec<u8> = vec![0; N * N];

// Cache-UNFRIENDLY: Iterating column by column
let mut sum = 0;
for col in 0..N {
    for row in 0..N {
        // This jumps N elements in memory on every single access!
        sum += matrix[row * N + col];
    }
}

// Cache-FRIENDLY: Iterating row by row
let mut sum = 0;
for row in 0..N {
    for col in 0..N {
        // Accesses are sequential in memory. This is ideal.
        sum += matrix[row * N + col];
    }
}
```

On a modern machine, the cache-friendly version can be **orders of magnitude faster** than the unfriendly one, even though they perform the exact same number of additions.

## Practical Tips for Writing Cache-Friendly Rust

1. **Use the Right Data Structures**: Default to `Vec<T>` for collections. Be very critical of linked data structures unless their flexibility is absolutely necessary.
2. **Think About Data Layout**: Consider the SoA pattern if you have hot loops that only access a subset of a struct's fields.
3. **Optimize Your Loops**: Ensure your data access patterns are sequential. For very large datasets, look into techniques like **loop tiling** (or cache blocking), where you process data in chunks that are sized to fit perfectly in the CPU cache.[^3]
4. **Mind Your `enum` Sizes**: An `enum` in Rust will be sized to hold its largest variant. If you have a `Vec` of enums where one variant is much larger than the others, it can waste a lot of space and harm cache locality. Consider boxing the data of the large variant (`MyEnum::LargeVariant(Box<LargeData>)`).
5. **Profile, Profile, Profile**: You cannot know if your changes are effective without measuring. Use profiling tools like `perf` and `flamegraph` to identify cache misses and confirm that your "cache-friendly" changes are actually improving performance.

By keeping the principles of spatial and temporal locality in mind, you can write Rust code that works *with* the hardware, not against it, unlocking significant performance gains that go beyond simple algorithmic optimization.

1. https://www.baeldung.com/cs/cache-friendly-code
2. https://www.reddit.com/r/rust/comments/2be61t/how_do_i_write_cache_friendly_code_in_rust/
3. https://stackoverflow.com/questions/16699247/what-does-it-mean-for-code-to-be-cache-friendly
4. https://users.rust-lang.org/t/tutorial-on-rust-memory-performance-latency-locality-access-allocate-cache-lines/130718
5. https://velog.io/@migorithm/Writing-Cache-friendly-programs-Rust-example
6. https://blog.devgenius.io/a-simple-cache-system-in-rust-7cf3d2489607
7. https://dev.to/seanchen1991/implementing-an-lru-cache-in-rust-33pp
8. https://www.youtube.com/watch?v=n-AsCx0142Y
9. https://gist.github.com/kvark/f067ba974446f7c5ce5bd544fe370186
