---
title: "Flash"
weight: 8031
menu:
  book:
    parent: "middlewares"
---

Middleware that provides the functionality of Flash Message.

`FlashStore` provides access to data. `CookieStore` stores data in `Cookie`. While `SessionStore` stores data in `Session`, `SessionStore` must be used with the `session` middleware.

## Config Cargo.toml

```toml
salvo = { version = "*", features = ["flash"] }
```

## Sample Code

```rust
use std::fmt::Write;

use salvo::prelude::*;
use salvo::flash::{CookieStore, FlashDepotExt};

#[handler]
pub async fn set_flash(depot: &mut Depot, res: &mut Response) {
    let flash = depot.outgoing_flash_mut();
    flash.info("Hey there!").debug("How is it going?");
    res.render(Redirect::other("/get"));
}

#[handler]
pub async fn get_flash(depot: &mut Depot, _res: &mut Response) -> String {
    let mut body = String::new();
    if let Some(flash) = depot.incoming_flash() {
        for message in flash.iter() {
            writeln!(body, "{} - {}", message.value, message.level).unwrap();
        }
    }
    body
}

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt().init();

    tracing::info!("Listening on http://127.0.0.1:7878");
    let router = Router::new()
        .hoop(CookieStore::new().into_handler())
        .push(Router::with_path("get").get(get_flash))
        .push(Router::with_path("set").get(set_flash));
    Server::new(TcpListener::bind("127.0.0.1:7878")).serve(router).await;
}
```