---
layout: default
title: Rust
description: Rust tips
tags: rust cargo coding asynchronous
---

## Rust asynchronous

Write the dumbest version first, optimise later (for multithreading - tokio, performance, optimal structures, cache etc). Clone data, don't bother with a reference (ignore lifetime issues). Waste RAM to get it working, worry later. Improvements are easy once you got it to compile the first time. Small changes easier.

Think of speed later, use for safety and type system.

- `cargo test`
- criterion (benchmarking)
- compile times slow, use cargo check before to minimize issues on compile

tokio - most widely used async library, for network functions (need to wait for I/O)
