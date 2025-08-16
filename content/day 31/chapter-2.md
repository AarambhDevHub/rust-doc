+++
title = "Capturing Environment"
description = "Learn how closures capture variables from their surrounding environment in Rust."
date = "2025-08-13"
weight = 2
+++

# (Capturing Environment) Learn how closures capture variables from their surrounding environment in Rust. for documentation in details with examples and how it works explain in details for beginner

Closures in Rust automatically **capture variables from the scope where they are defined**; how a variable is captured depends on what the closure does with it and determines which of the three closure traits—**Fn, FnMut, FnOnce**—the compiler implements for that closure.[^1][^3]

- **Immutable borrow (Fn)** – if the closure only *reads* the value, it borrows it immutably:

```rust
fn main() {
    let list = vec![1, 2, 3];
    let print = || println!("From closure: {:?}", list);   // &list
    print();            // can call many times
    println!("Outside: {:?}", list);  // still usable
}
```

The closure implements **Fn**, **FnMut**, and **FnOnce** because an immutable borrow imposes no extra restrictions.

- **Mutable borrow (FnMut)** – if the body *modifies* the captured value, the borrow must be mutable:

```rust
fn main() {
    let mut counter = 0;
    let mut tick = || {            // &mut counter
        counter += 1;
        println!("Counter = {counter}");
    };
    tick();  tick();               // may call repeatedly
    println!("Final = {counter}");
}
```

Such a closure implements **FnMut** and **FnOnce** (but not Fn) because a mutable borrow forbids simultaneous immutable borrows.

- **Move / take ownership (FnOnce)** – if the closure *moves* the value (e.g., drops it or transfers it to another thread) it owns the captured variable, so it can be invoked **at most once**:

```rust
fn main() {
    let secret = String::from("shhh");
    let reveal = move || {          // secret moved here
        println!("The secret is {secret}");
    };
    reveal();                       // OK
    // reveal();                   // ERROR: use after move
}
```

The compiler now generates only **FnOnce** because the environment is consumed.[^7]

### How the compiler decides

1. It starts with an immutable borrow; if the body needs more permission it escalates to a mutable borrow, and finally to a move (ownership transfer).[^3]
2. The chosen capture mode is invisible in the source code unless you force a move with the `move` keyword (common when sending closures to other threads).[^1]

### Why three traits?

- **Fn** (\&T)  – can be called many times, never mutates or moves captures.
- **FnMut** (\&mut T) – can mutate captures, still callable many times.
- **FnOnce** (T) – may consume captures; guaranteed callable only once.
A closure automatically implements *all* the traits that its capture mode permits; e.g., an immutable-borrow closure implements all three.


### Forcing specific capture behaviour

- Add `move` before the vertical bar to capture **everything by value** (ownership).
- Add explicit borrows in the parameter list if you need to pass references into the closure.

```rust
let data = vec![10, 20, 30];
std::thread::spawn(move || println!("Thread got {data:?}"));  // data moved
```


### Returning or storing closures

When you store a closure in a struct or return it from a function, use a trait object whose bound matches the strongest requirement:

```rust
fn make_printer(msg: String) -> impl Fn() {
    move || println!("{msg}")      // FnOnce + immutable borrow → Fn
}

struct Handler {
    f: Box<dyn FnMut(usize) + Send>,
}
```


### Common beginner pitfalls

1. **Borrow checker errors** – try adding `mut` to the closure variable (`let mut c = || { … }`) or insert `move`.
2. **Trying to call an FnOnce twice** – store its result or redesign to borrow instead of move.
3. **Capturing large data unintentionally** – use references (`&data`) or slice parameters to avoid cloning big collections.

### Take-away rules

1. **Read-only → Fn**, **read + write → FnMut**, **move/consume → FnOnce**.[^7]
2. The compiler escalates capture permissions automatically, so start simple.
3. Use `move` for thread safety or when you must outlive the original scope.
4. Understand capture modes to avoid lifetime or ownership surprises.

With these principles you can confidently use Rust closures that interact with their environment while keeping ownership and borrowing rules intact.

1. https://doc.rust-lang.org/book/ch13-01-closures.html
2. https://www.reddit.com/r/rust/comments/1cdn357/confused_on_what_rust_book_means_for_closures/
3. https://doc.rust-lang.org/rust-by-example/fn/closures/capture.html
4. https://rust-book.cs.brown.edu/ch13-01-closures.html
5. https://www.geeksforgeeks.org/rust/rust-concept-of-capturing-variables-with-closers/
6. https://dev.to/francescoxx/closures-in-rust-2kcm
7. https://leapcell.io/blog/understanding-rust-closures
8. https://stackoverflow.com/questions/36511141/returning-closures-which-capture-outer-variables-in-rust
9. https://blog.logrocket.com/optimizing-rust-code-with-closures-environment-capturing/
