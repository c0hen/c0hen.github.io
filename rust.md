---
layout: default
title: Rust
description: Rust tips
tags: rust cargo coding development asynchronous publishing apps threads io-bound
---

## Rust publishing and distribution

[Install Rust and cargo](https://doc.rust-lang.org/nightly/cargo/getting-started/installation.html)

### Maintenance and common operations
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
```
Other local documentation - books etc.
```sh
rustup doc --path --toolchain stable --clippy
rustup doc fn
```

### Packages

Rust code is [packaged](https://doc.rust-lang.org/cargo/appendix/glossary.html#package) as a collection of [crates](https://doc.rust-lang.org/cargo/appendix/glossary.html#crate).
It's possible to create private package [registries](https://doc.rust-lang.org/cargo/reference/registries.html).

Cargo honor locked versions, do a reproducible all binaries install.
```sh
cargo install --bins --locked
```
```sh
cargo install --list
```

Show package information with [cargo info](https://doc.rust-lang.org/nightly/cargo/commands/cargo-info.html)

## Fearless Rust

Write the dumbest version first, optimise later (for multithreading - tokio, performance, optimal structures, cache etc). Clone data, don't bother with a reference (ignore lifetime issues). Waste RAM to get it working, worry later. Improvements are easy once you got it to compile the first time. Small changes easier.

Think of speed later, use for safety and type system.

- `cargo test`
- criterion (benchmarking)
- compile times slow, use `cargo check` before to minimize issues on compile. Alternatively, run [bacon clippy](/useful-commands/#bacon).

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

`std::sync::Arc` - [Atomically Reference Counted](https://doc.rust-lang.org/std/sync/struct.Arc.html)

[smol](https://docs.rs/smol/latest/smol/) is a small and fast async runtime.

[rayon](https://docs.rs/rayon/latest/rayon/) - converts iterators into parallel iterators. Rayon guarantees you that using Rayon APIs will not introduce data races.

## Rust (mobile) app development

### [Tauri](https://v2.tauri.app/start/)

Tauri uses web technologies, that means that virtually any frontend framework is compatible with Tauri.

## Rust [web development](https://www.arewewebyet.org/)

## Rust [game development](https://arewegameyet.rs/)

Godot has [Rust bindings](https://godot-rust.github.io/).
