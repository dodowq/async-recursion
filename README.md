# async-recursion macro

![Build Status](https://github.com/dcchut/async-recursion/workflows/Push%20action/badge.svg?branch=master)

Procedural macro for recursive async functions.

* [Documentation](https://docs.rs/async-recursion/)
* Cargo package: [async-recursion](https://crates.io/crates/async-recursion)

## Motivation
Consider the following recursive implementation of the fibonacci numbers:

```rust,ignore
async fn fib(n : u32) -> u64 {
   match n {
       0     => panic!("zero is not a valid argument to fib()!"),
       1 | 2 => 1,
       3     => 2,
       _ => fib(n-1).await + fib(n-2).await
   }
}
```

The compiler helpfully tells us that:

```console
error[E0733]: recursion in an `async fn` requires boxing
--> src/main.rs:1:26
  |
1 | async fn fib(n : u32) -> u64 {
  |                          ^^^ recursive `async fn`
  |
 = note: a recursive `async fn` must be rewritten to return a boxed `dyn Future`.
 = note: consider using the `async_recursion` crate: https://crates.io/crates/async_recursion
```

This crate provides an attribute macro to automatically convert an async function 
to one returning a boxed `Future`.

## Example

```rust
use async_recursion::async_recursion;

#[async_recursion]
async fn fib(n : u32) -> u64 {
   match n {
       0     => panic!("zero is not a valid argument to fib()!"),
       1 | 2 => 1,
       3     => 2,
       _ => fib(n-1).await + fib(n-2).await
   }
}
```

## ?Send Option

The returned future has a `Send` bound to make sure it can be sent between threads.
If this is undesirable you can mark that the bound should be left out like so:

```rust
#[async_recursion(?Send)]
async fn example() {
   // ...
}
```

In detail:

- `#[async_recursion]` modifies your function to return a [`BoxFuture`], and
- `#[async_recursion(?Send)]` modifies your function to return a [`LocalBoxFuture`].

[`BoxFuture`]: https://docs.rs/futures/0.3.19/futures/future/type.BoxFuture.html
[`LocalBoxFuture`]: https://docs.rs/futures/0.3.19/futures/future/type.LocalBoxFuture.html

### License

Licensed under either of
 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.
