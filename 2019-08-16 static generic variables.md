Another new thing that stumped me as I continue trying to get TLS functionality into `rust-postgres-rest`:

```rust
use actix::Addr;
use tokio_postgres::{tls::MakeTlsConnect, Socket};
use crate::Config;
use std::sync::{atomic::AtomicBool, Mutex};

/// Contains the Table Stats cache.
pub(crate) struct StatsCache<T: MakeTlsConnect<Socket> + Clone + Send + Sync + 'static> {
    /// Postgres-formatted URL.
    config: Config<T>,
    /// Multi-threaded access to the table stats cache.
    cache: Arc<RwLock<Option<HashMap<String, TableStats>>>>,
    /// Whether the cache is currently being fetched/reset.
    is_fetching: Arc<AtomicBool>,
}

lazy_static! {
    static ref STATS_CACHE_MUTEX: Mutex<Option<Addr<StatsCache<T>>>> = Mutex::new(None);
}
```

The idea being that I want a static mutex that contains the Actix actor containing
our cache (which contains the result of an expensive and frequently used database query).

Unfortunately it doesn't compile:

```plaintext
error[E0412]: cannot find type `T` in this scope
   --> rust-postgres-rest/src/stats_cache.rs:154:64
    |
    |     static ref STATS_CACHE_MUTEX: Mutex<Option<Addr<StatsCache<T>>>> = Mutex::new(None);
    |                                                                ^ not found in this scope
```

Apparently you can't declare generic static variables. According to Fenrir on the Rust Discord (beginners channel),
> Generics create a new version of a struct for each type they're instantiated with,
> which conflicts with the idea of a single static item

So either I'd have to:
1. declare it with a concrete type, or
1. find another way to store a cache, or
1. eliminate the need for a generic.

UPDATE: I ended up going with #3. This required me to move the non-generic part that I needed from `Config` out of `Config` in the `StatsCache` struct.
