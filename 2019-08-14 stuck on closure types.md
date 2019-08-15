As rust-postgres-rest is getting finished, I have one last major hurdle to tackle before I can make a preliminary release,
and that is how to allow the creation of a TLS socket via `Config`.

Ideally I'd like either:
1. The user can pass their own TLS connection to `Config`, where it is cloned/used on every database connection made.
1. The user passes in a closure (or some way) to generate a TLS connection whenever a database connection is made.

I don't think #1 is feasible, as it sounds not-thread-safe. How do you pass an active socket between threads?

Which leaves #2. I've tried:

```rust
use tokio_postgres::{tls::MakeTlsConnect, NoTls};

struct A<S, T, F> where
T: MakeTlsConnect<S> + Clone + Send + Sync,
F: Fn() -> T {
    tls: F
}

fn main() {
    let test = A { tls: || NoTls };
}
```

but that leads to the error:
```plaintext
error[E0392]: parameter `S` is never used
 --> src/main.rs:3:10
  |
3 | struct A<S, T, F> where
  |          ^ unused parameter
  |
  = help: consider removing `S` or using a marker such as `std::marker::PhantomData`
```

Following the suggestion and adding `std::marker::PhantomData` leads to this code:

```rust
use tokio_postgres::{tls::{MakeTlsConnect, NoTlsStream}, NoTls};
use std::marker::PhantomData;

struct A<S, T, F> where
T: MakeTlsConnect<S> + Clone + Send + Sync,
F: Fn() -> T {
    tls: F,
    sp: PhantomData<S>
}

fn main() {
    let test = A { tls: || NoTls, sp: PhantomData };
}
```

which leads to this error:

```plaintext
error[E0282]: type annotations needed for `A<S, tokio_postgres::tls::NoTls, [closure@src/main.rs:12:25: 12:33]>`
  --> src/main.rs:12:16
   |
12 |     let test = A { tls: || NoTls, sp: PhantomData };
   |         ----   ^ cannot infer type for `S`
   |         |
   |         consider giving `test` the explicit type `A<S, tokio_postgres::tls::NoTls, [closure@src/main.rs:12:25: 12:33]>`, where the type parameter `S` is specified
```

Now I need to research how to use the correct syntax to proceed with the suggestion in the compiler error.

**EDIT**: Yay for type inference! This compiles:

```rust
use std::marker::PhantomData;
use tokio_postgres::{
    tls::{MakeTlsConnect, NoTlsStream},
    NoTls,
};

struct A<S, T, F>
where
    T: MakeTlsConnect<S> + Clone + Send + Sync,
    F: Fn() -> T,
{
    tls: F,
    sp: PhantomData<S>,
}

fn main() {
    let test: A<NoTlsStream, NoTls, _> = A {
        tls: || NoTls,
        sp: PhantomData,
    };
}
```

Now I need to figure out how I can call `test.tls`, as `test.tls()` gives me a `No method named `tls` found for type A` compiler error.

EDIT: Found this gem that gave me the info I needed: https://github.com/rust-lang/rust/issues/18343

This compiles and runs:

```rust
use std::marker::PhantomData;
use tokio_postgres::{
    tls::{MakeTlsConnect, NoTlsStream},
    NoTls,
};

struct A<S, T, F>
where
    T: MakeTlsConnect<S> + Clone + Send + Sync,
    F: Fn() -> T,
{
    tls: F,
    sp: PhantomData<S>,
}

fn main() {
    let test: A<NoTlsStream, NoTls, _> = A {
        tls: || NoTls,
        sp: PhantomData,
    };

    dbg!((test.tls)());
    // prints:
    // [src/main.rs:22] (test.tls)() = NoTls
}
```
