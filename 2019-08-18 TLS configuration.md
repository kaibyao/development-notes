Before I can release `rust-postgres-rest`, I need to make sure that my implementation of TLS is working correctly.
To do that, I need to set up TLS on my test database and attempt to connect to it with TLS configured in `rust-postgres-rest`.

A few steps to doing that:

1. Set up TLS certificates/keys
1. Start the database with TLS.
1. Attempt to connect to the database using `psql` (with TLS).
1. Set up the same config in `rust-postgres-rest` tests.
1. Attempt to connect to the database using `rust-postgres-rest`.

