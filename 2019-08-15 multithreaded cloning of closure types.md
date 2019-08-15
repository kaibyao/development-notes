Continuing off of yesterday's findings, I had to install the `derivative` crate in order to
properly clone my object containing the TLS-generating closure.

I had to:
1. Change the `Fn() -> T` to just a closure type (didn't know I could do this): `fn() -> T`.
1. Change `#[derive(Clone)]` to `#[derive(Derivative)] #[derivative(Clone)]`.

```rust
use derivative::Derivative;
use std::marker::PhantomData;
use tokio_postgres::{
    tls::{MakeTlsConnect, NoTlsStream},
    NoTls,
};

#[derive(Derivative)]
#[derivative(Clone)]
struct A<S, T>
where
    T: MakeTlsConnect<S> + Clone + Send + Sync,
{
    tls: fn() -> T,
    sp: PhantomData<S>,
}

fn main() {
    let test: A<NoTlsStream, NoTls> = A {
        tls: || NoTls,
        sp: PhantomData,
    };

    dbg!((test.tls)());
    dbg!((test.clone().tls)());
}
```
