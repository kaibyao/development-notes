Before I can release `rust-postgres-rest`, I need to make sure that my implementation of TLS is working correctly.
To do that, I need to set up TLS on my test database and attempt to connect to it with TLS configured in `rust-postgres-rest`.

A few steps to doing that:

1. Set up TLS certificates/keys.
1. Start the database with TLS.
1. Attempt to connect to the database using `psql` (with TLS).
1. Set up the same config in `rust-postgres-rest` tests.

## Set up TLS certificates/keys

Ran the following:

```bash
$ openssl req -new -x509 -nodes -text -out server.crt -keyout server.key -subj "/CN=localhost"
$ chmod og-rwx server.key
```
Notes about arguments:
- `x509` means self-signing certificate.
- `nodes` means no password (no "des" [data encryption standard], I think)
- `-text` ?
- `-out` output cert to a file
- `-keyout` output key to a file
- `-subj` answer questions on the same command instead of in a prompt

### Resources

- https://www.howtoforge.com/postgresql-ssl-certificates
- https://stackoverflow.com/questions/55072221/deploying-postgresql-docker-with-ssl-certificate-and-key-with-volumes
- https://www.postgresql.org/docs/current/ssl-tcp.html

## Start the database with TLS

Configuring docker-compose w/ the new keys + commands to run:

```yaml
# docker-compose.yaml

version: '3.1'

services:
  db:
    image: postgres
    command: -c ssl=on -c ssl_cert_file=/var/lib/postgresql/server.crt -c ssl_key_file=/var/lib/postgresql/server.key
    restart: always
    environment:
      POSTGRES_PASSWORD: example # user = postgres
    ports:
      - 5433:5432
    volumes:
      - ./rust-postgres-rest/tests/server.crt:/var/lib/postgresql/server.crt
      - ./rust-postgres-rest/tests/server.key:/var/lib/postgresql/server.key
```

Had to run `docker-compose down` and `docker-compose up` to restart the postgres container with the new settings.

### Resources

- https://gist.github.com/mrw34/c97bb03ea1054afb551886ffc8b63c3b

## Attempt to connect to the database using `psql` (with TLS)

From the root folder of `rust-postgres-rest`:

```bash
$ psql "host=localhost port=5433 user=postgres password=example dbname=postgres sslmode=verify-full sslrootcert=./rust-postgres-rest/tests/server.crt sslkey=./rust-postgres-rest/tests/server.key"
```

### Resources

- https://stackoverflow.com/questions/14021998/using-psql-to-connect-to-postgresql-in-ssl-mode
- https://www.postgresql.org/docs/9.5/libpq-envars.html

## Set up the same config in `rust-postgres-rest` tests

Had to look at the `server.crt` file and extract the following to make the `server.pem` file:

- The certificate portion of `server.crt` (`BEGIN CERTIFICATE`).

Then I created the following unit test:
```rust
use native_tls::{Certificate, TlsConnector};
use postgres_native_tls::MakeTlsConnector;
use rust_postgres_rest::Config;
use std::fs;

#[test]
fn native_tls() {
    let cert_str = fs::read("./tests/server.pem").unwrap();
    let cert = Certificate::from_pem(&cert_str).unwrap();
    let tls_connector = TlsConnector::builder()
        .add_root_certificate(cert)
        .build()
        .unwrap();
    let postgres_tls_connector = MakeTlsConnector::new(tls_connector);

    let mut cfg = Config::new("host=localhost port=5433 user=postgres password=example dbname=postgres sslmode=require");
    cfg.set_tls(postgres_tls_connector);

    cfg.connect().wait().unwrap();
}
```
