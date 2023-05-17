# `async-lazy`

[![Build Status](https://github.com/Jules-Bertholet/async-lazy/actions/workflows/actions.yml/badge.svg)](https://github.com/Jules-Bertholet/async-lazy/actions)
[![Crates.io](https://img.shields.io/crates/v/async-lazy.svg)](https://crates.io/crates/async-lazy)
[![API reference](https://docs.rs/async-lazy/badge.svg)](https://docs.rs/async-lazy/)
[![License](https://img.shields.io/crates/l/async-lazy.svg)](https://github.com/Jules-Bertholet/async-lazy#license)

An `async` version of [`once_cell::sync::Lazy`](https://docs.rs/once_cell/latest/once_cell/sync/struct.Lazy.html), [`std::sync::OnceLock`](https://doc.rust-lang.org/nightly/std/sync/struct.OnceLock.html) or [`lazy_static`](https://crates.io/crates/lazy_static). Uses [`tokio`](https://github.com/tokio-rs/tokio)'s sychronization primitives.

## Crate features

- `parking_lot`: Enables the corresponding feature in `tokio`, and ssallows constructing `Lazy`s in `const` contexts.
- `nightly`: Uses nightly Rust features to implement [`IntoFuture`](https://doc.rust-lang.org/std/future/trait.IntoFuture.html) for references to `Lazy`s, obviating the need to call `force()`.

## Example

```rust
use std::time::Duration;
use async_lazy::Lazy;

async fn some_computation() -> u32 {
    tokio::time::sleep(Duration::from_secs(1)).await;
    1 + 1
}

static LAZY : Lazy<u32> = Lazy::const_new(|| Box::pin(async { some_computation().await }));

#[tokio::main]
async fn main() {
    let result = tokio::spawn(async {
        *LAZY.force().await
    }).await.unwrap();

    assert_eq!(result, 2);
}
```
