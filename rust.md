---
layout: default
title: Rust
description: Rust tips
tags: rust cargo coding development asynchronous publishing apps threads io-bound types macros
---

## Rust basics

Rust toolchain is tightly integrated with documentation and development with `cargo`.

### Rust features

#### Algebraic Type System

A trait defines the functionality a particular type has and can share with other types. Struct and Enum are user defined types in Rust. The algebraic type system allows modeling with types and traits that makes code harder to misuse. Think about valid states when modeling.

- "OR" (sum, Enum) type

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

- "AND" (product, Struct) type

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

#### Macros

Macros are Rust code that writes other Rust code (metaprogramming). Allows
- creating new language features,
- keeping the language clean and clear.

Macros are created using the `macro_rules!` macro.

```rust
macro_rules! say_hello {
    // `()` indicates that the macro takes no argument.
    () => {
        // The macro will expand into the contents of this block.
        println!("Hello!")
    };
}

fn main() {
    // This call will expand into `println!("Hello!")`
    say_hello!()
}
```

#### Perfect backwards compatibility

[Crater](https://crater.rust-lang.org) [compiles and tests](https://github.com/rust-lang/crater) every crate. It is used to verify that new versions of Rust are compatible with all crates.

#### [Ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

Compiler `rustc` checks the rules governing memory ownership.

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

Function `std::mem::drop` takes ownership of a value and the value is dropped automatically before it returns. The whole function definition is

```rust
pub fn drop<T>(_x: T) {}
```

Memory stack is last in first out (LIFO) and faster than the heap. Data with an unknown size at compile time or a size that might change must be stored on the heap.

#### No garbage collection

No garbage collection means no "stop the world" pauses in code execution.

#### Variables have lifetimes

Let the compiler teach usage of lifetimes.

#### Unsafe system

Allows operations that Rust can not guarantee the safety of and limits where to search for memory bugs.

### Rust development, publishing and distribution

[Install Rust and cargo](https://doc.rust-lang.org/stable/cargo/getting-started/installation.html)

#### Maintenance and common operations

```sh
rustup update
```
```sh
cargo clean --help
```
Remove a crate with
```sh
cargo uninstall
```
Toolchains
```sh
rustup toolchain list --help
rustup component list -h
rustup component add rust-analyzer
```
Other local documentation - books etc.
```sh
rustup doc --path --toolchain stable --clippy
rustup doc --toolchain beta fn
rustup doc --reference
rustup man rustc
rustc --explain E0425
```

#### Packages

Rust code is [packaged](https://doc.rust-lang.org/cargo/appendix/glossary.html#package) as a collection of [crates](https://doc.rust-lang.org/cargo/appendix/glossary.html#crate).
It's possible to create private package [registries](https://doc.rust-lang.org/cargo/reference/registries.html).

```sh
cargo search mask
```

Cargo honor locked versions, do a reproducible all binaries install.
```sh
cargo install --bins --locked mask
```
```sh
cargo install --list
```

Show package information with [cargo info](https://doc.rust-lang.org/stable/cargo/commands/cargo-info.html)

## Fearless Rust

Write the dumbest version first, optimise later (for multithreading - tokio, performance, optimal structures, cache etc). Clone data, don't bother with a reference (ignore lifetime issues). Waste RAM to get it working, worry later. Improvements are easy once you got it to compile the first time. Small changes easier.

Think of speed later, use for safety and type system.

- `cargo test`
- criterion (benchmarking)
- compile times slow, use `cargo check` before to minimize issues on compile. Alternatively, run [bacon clippy](/useful-commands/#bacon).

```sh
cargo clippy --fix -- -W clippy::pedantic -W clippy::nursery -W clippy::unwrap_used -W clippy::expect_used
```

## Rust By Example

```sh
rustup doc --rust-by-example
```

### Intro

```sh
cargo install --locked rustlings manrs
```
Rustlings exercises and the Rust book (The Rust Programming Language, `rustup doc --book`) are interlinked, Rust By Example is more loosely associated with them.

`manrs` allows reading the docs without a web browser. Show only examples of I/O
```
manrs -e std::io
```

### Language topics

#### Rustup toolchain

A toolchain may become outdated when crates have become a component between updates.

```sh
rustup toolchain uninstall stable
rustup toolchain install stable
```

Profile is a grouping of components.
```sh
rustup show profile
rustup install --profile minimal beta
```

The default [target architecture](https://rust-lang.github.io/rustup/cross-compilation.html) for cargo and rustc is the host toolchain's platform.

Toolchain may be [overridden on a project basis](https://rust-lang.github.io/rustup/overrides.html#the-toolchain-file).

```toml
# project_directory/rust-toolchain.toml
[toolchain]
channel = "nightly-2020-07-10"
components = [ "rustfmt", "rustc-dev" ]
targets = [ "arm-linux-androideabi", "thumbv2-none-eabi" ]
profile = "minimal"
```

#### More cargo, project dependency updates

The yank command removes a previously published crate's version from the server's index. This command does not delete any data, and the crate will still be available for download via the registry's download link.

Cargo will not use a yanked version for any new project or checkout without a pre-existing lockfile, and will generate an error if there are no longer any compatible versions for your crate.

Suppose `bitstream-io` version used depends on a yanked crate (deprecated core2), requires a version update.

```sh
cargo update bitstream-io
cargo test
```

#### [Crates.io data access](https://crates.io/data-access)

`cargo search` shows first 100 results.
```sh
xh user-agent:contact@example.com https://crates.io/api/openapi.json
```
Complete database dump [db-dump.tar.gz](https://static.crates.io/db-dump.tar.gz), updated every 24 hours.

#### [Documentation comments](https://doc.rust-lang.org/rust-by-example/hello/comment.html#documentation-comments-doc-comments-which-are-parsed-into-html-library-documentation)

#### Using [match](https://doc.rust-lang.org/rust-by-example/flow_control/match.html) instead of [if](https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html) makes compiler errors more helpful

`if` can do anything but `match` is type safe, compiler can determine what cases may be missing.

#### Box

`<T>` - Type

A pointer type `Box<T>` for heap allocation.

```sh
rustup doc std::boxed
```

#### Option type

```sh
rustup doc std::option::Option
```

Type Option represents an optional value: every Option is either Some and contains a value, or None, and does not.

There are no "null" references. Instead, Rust has optional pointers, like the optional owned box, `Option<Box<T>>`.

#### Infallible and ! never type

`core::convert::Infallible` is the error type for errors that can never happen, hence the result is always Ok. Deprecation in favour of `!` is planned.

`!` is a type with no values, representing the result of computations that never complete. Currently unstable.

```rust
fn foo() -> ! {
    panic!("This call never returns.");
}
```

#### Borrow checker

Borrow checker rules.

1. data has one owner
1. data may have multiple readers or one writer

Trying to use data that has been passed for writing or moved results in error `use of moved value`.

### Threads and borrowing references

Avoid immortal ([static](https://doc.rust-lang.org/std/keyword.static.html)) references when writing asynchronous code. Allows borrowing references.

Wait for [threads](https://doc.rust-lang.org/book/ch16-01-threads.html) to complete, this happens at the end of a scope. [Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html) is a major goal of Rust, along with safety.

```
# Cargo.toml
[dependencies]
minreq = { version >= "2.14.1" }
```

```rust
// scope blocks until threads are done, therefore
// rustc has a guarantee that all threads will finish (join).

fn expensive_scope(url: &String) {
  std::thread::scope(|s| {
    s.spawn(|| minreq::get(url));
  })
  dbg!("url: {}", url);
}
```

### Rust parallel asynchronous

Better not to use async unless threads is not enough.

[tokio](https://docs.rs/tokio/latest/tokio/) - most widely used async library, for network functions (need to wait for I/O). Muddies code, better to do without in the beginning. Consider a [scoped tokio runtime](https://docs.rs/tokio-scoped/latest/tokio_scoped/).

`std::sync::Arc` - [Atomically Reference Counted](https://doc.rust-lang.org/std/sync/struct.Arc.html), thread-safe reference-counting pointer.

[smol](https://docs.rs/smol/latest/smol/) is a small and fast async runtime.

[rayon](https://docs.rs/rayon/latest/rayon/) - converts iterators into parallel iterators. Rayon guarantees you that using Rayon APIs will not introduce data races.

### Rust other language tools

#### sqlx

Async SQL toolkit for Rust, with compile time SQL validation.

#### poem-openapi

Uses procedural macros to generate a lot of boilerplate code so your code is a clear and clean implementation of needs. Complies with the OpenAPIv3 specification.

#### serde

Serialize, deserialize data types.

#### rstml

Backend html.

#### yew

Frontend html.

#### reqwest

Convenient, higher-level HTTP client.

#### tracing

Async native (tokio) logging.

#### color\_eyre

Colourful, consistent, well formatted error reports.

#### chrono

Date and time library.

#### [irust](https://docs.rs/crate/irust/1.76.2)

Interactive Rust read-eval-print-loop (REPL), debug, asm inspection.
```
:help
:quit
```

## Rust (mobile) app development

### [Tauri](https://v2.tauri.app/start/)

Tauri uses web technologies, that means that virtually any frontend framework is compatible with Tauri.

## Rust [web development](https://www.arewewebyet.org/)

## Rust [game development](https://arewegameyet.rs/)

Godot has [Rust bindings](https://godot-rust.github.io/).
