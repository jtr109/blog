---
title: "How to Log in Rust"
date: 2020-06-17T15:35:25+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
  - log
---

## Using `log` Module

[Log module](https://docs.rs/log/) does not show any log without a logging implementation[^1].

```rust
use log::{debug, trace, info, warn, error};

fn main() {
    debug!("debug");
    trace!("trace");
    info!("info");
    warn!("warn");
    error!("error");
}
```

## Using `env_logger`

[The `env_logger` module](https://docs.rs/env_logger/0.7.1/env_logger/) is a logging implementation we can choose. But even if we init it. The output is not shown as expected.

```rust
use log::{debug, trace, info, warn, error};

fn main() {
    env_logger::init();

    debug!("debug");
    trace!("trace");
    info!("info");
    warn!("warn");
    error!("error");
}
```

The output only contains error message, because the default logging level is _error_.

```
[2020-06-17T09:56:01Z ERROR playground] error
```

The document has [explained](https://docs.rs/env_logger/0.7.1/env_logger/#enabling-logging).

> Log levels are controlled on a per-module basis, and by default all logging is disabled except for `error!`. Logging is controlled via the `RUST_LOG` environment variable.

## Specifying Logging Level

If we want to set the logging level, the environment variable `RUST_LOG` is required. Such as below.

```shell
RUST_LOG=info cargo run
```

## Setting Default Logging Level

But we can also change the default logging level explicitly.

```rust
use log::{debug, trace, info, warn, error};
use env_logger::Env;

fn main() {
    env_logger::from_env(Env::default().default_filter_or("warn")).init();

    debug!("debug");
    trace!("trace");
    info!("info");
    warn!("warn");
    error!("error");
}
```

Now you can [see](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=ff9ceff45cee38a23e818f60c3557184) all logs higher than specific level.

```
[2020-06-17T10:33:10Z WARN  playground] warn
[2020-06-17T10:33:10Z ERROR playground] error
```

## References

* [`log` module document](https://docs.rs/log/0.4.8/log/index.html)

* [`env_logger` document](https://docs.rs/env_logger/0.7.1/env_logger/index.html)
* [Configure Loggin in Rust Cookbook](https://rust-lang-nursery.github.io/rust-cookbook/development_tools/debugging/config_log.html)

-----

[^1]: All avaliable implementations are listed [here](https://docs.rs/log/0.4.8/log/#available-logging-implementations).